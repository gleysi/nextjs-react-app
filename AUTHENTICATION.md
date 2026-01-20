- [Sources](#sources)

# Authentication

## 1. The Action: Logging In
Server Actions used to handle the user login and signup logic

### Core Functionality
- `signup` **Action:** Handles the creation of new user accounts. It extracts the email and password from the `formData`, performs basic validation (e.g., checking for valid email format and password length), hashes the password (usually using `bcryptjs`), and saves the new user to the database (SQLite in this course).

- `login` **Action:** Manages existing user authentication. It verifies the submitted credentials against the database, compares the hashed passwords, and establishes a session.

- **Session Management:** It typically uses the `lucia` authentication library or a custom cookie-based system (depending on the specific lecture version) to set a session cookie that keeps the user logged in across requests.

- **Error Handling:** It returns state objects (often used with React's `useActionState` or `useFormState`) that contain error messages to be displayed on the frontend if validation or authentication fails.

**File:** `lib/auth-actions.js`

```javascript
'use server';
import { redirect } from 'next/navigation';

import { hashUserPassword, verifyPassword } from '@/lib/hash';
import { createUser, getUserByEmail } from '@/lib/user';
import { createAuthSession } from '@/lib/auth';

export async function signup(prevState, formData) {
  const email = formData.get('email');
  const password = formData.get('password');

  let errors = {};

  if (!email.includes('@')) {
    errors.email = 'Please enter a valid email address.';
  }

  if (password.trim().length < 8) {
    errors.password = 'Password must be at least 8 characters long.';
  }

  if (Object.keys(errors).length > 0) {
    return {
      errors,
    };
  }

  const hashedPassword = hashUserPassword(password);
  try {
    const id = createUser(email, hashedPassword);
    await createAuthSession(id);
    redirect('/training');
  } catch (error) {
    if (error.code === 'SQLITE_CONSTRAINT_UNIQUE') {
      return {
        errors: {
          email:
            'It seems like an account for the chosen email already exists.',
        },
      };
    }
    throw error;
  }
}

export async function login(prevState, formData) {
  const email = formData.get('email');
  const password = formData.get('password');

  const existingUser = getUserByEmail(email);

  if (!existingUser) {
    return {
      errors: {
        email: 'Could not authenticate user, please check your credentials.',
      },
    };
  }

  const isValidPassword = verifyPassword(existingUser.password, password);

  if (!isValidPassword) {
    return {
      errors: {
        password: 'Could not authenticate user, please check your credentials.',
      },
    };
  }

  await createAuthSession(existingUser.id);
  redirect('/training');
}

export async function auth(mode, prevState, formData) {
  if (mode === 'login') {
    return login(prevState, formData);
  }
  return signup(prevState, formData);
}
```

## 2. The Configuration (lib/auth.ts)
**Central configuration file for the authentication library**

Lucia Auth (a flexible authentication library) to demonstrate how sessions are managed manually, rather than using a "magic" box like NextAuth. 

### Primary Roles:

* **Database Adapter Setup**: It imports the database connection (usually from your `db.js` or similar) and creates an "adapter" (e.g., `BetterSqlite3Adapter`). This tells the auth library exactly where and how to store user sessions.
* **Lucia Instance Initialization**: It creates the `lucia` object, which provides the main methods for:
    * Creating new sessions.
    * Validating session cookies.
    * Deleting sessions (logging out).
* **Cookie Configuration**: It defines how the session cookie should behave (e.g., setting `secure: true` in production, defining the cookie name, and managing expiration).
* **TypeScript Integration**: It often contains a `declare module "lucia"` block to ensure that the user object includes custom fields (like email or username) throughout the rest of your app.
* **Session Validation Helper**: It usually exports a function (like `verifyAuth`) that checks the request headers for a valid cookie and returns the current user if they are logged in.

```javascript
import { cookies } from 'next/headers';
import { Lucia } from 'lucia';
import { BetterSqlite3Adapter } from '@lucia-auth/adapter-sqlite';

import db from './db';

const adapter = new BetterSqlite3Adapter(db, {
  user: 'users',
  session: 'sessions',
});

const lucia = new Lucia(adapter, {
  sessionCookie: {
    expires: false,
    attributes: {
      secure: process.env.NODE_ENV === 'production',
    },
  },
});

export async function createAuthSession(userId) {
  const session = await lucia.createSession(userId, {});
  const sessionCookie = lucia.createSessionCookie(session.id);
  cookies().set(
    sessionCookie.name,
    sessionCookie.value,
    sessionCookie.attributes
  );
}

export async function verifyAuth() {
  const sessionCookie = cookies().get(lucia.sessionCookieName);

  if (!sessionCookie) {
    return {
      user: null,
      session: null,
    };
  }

  const sessionId = sessionCookie.value;

  if (!sessionId) {
    return {
      user: null,
      session: null,
    };
  }

  const result = await lucia.validateSession(sessionId);

  try {
    if (result.session && result.session.fresh) {
      const sessionCookie = lucia.createSessionCookie(result.session.id);
      cookies().set(
        sessionCookie.name,
        sessionCookie.value,
        sessionCookie.attributes
      );
    }
    if (!result.session) {
      const sessionCookie = lucia.createBlankSessionCookie();
      cookies().set(
        sessionCookie.name,
        sessionCookie.value,
        sessionCookie.attributes
      );
    }
  } catch {}

  return result;
}
```

## 3. Checking the Session (UI)
In a Server Component, you can check if the user is logged in using the auth() function.

**File:** `components/auth-form.js`

```javascript
'use client';
import Link from 'next/link';
import { useFormState } from 'react-dom';

import { auth } from '@/actions/auth-actions';

export default function AuthForm({ mode }) {
  const [formState, formAction] = useFormState(auth.bind(null, mode), {});
  return (
    <form id="auth-form" action={formAction}>
      <div>
        <img src="/images/auth-icon.jpg" alt="A lock icon" />
      </div>
      <p>
        <label htmlFor="email">Email</label>
        <input type="email" name="email" id="email" />
      </p>
      <p>
        <label htmlFor="password">Password</label>
        <input type="password" name="password" id="password" />
      </p>
      {formState.errors && (
        <ul id="form-errors">
          {Object.keys(formState.errors).map((error) => (
            <li key={error}>{formState.errors[error]}</li>
          ))}
        </ul>
      )}
      <p>
        <button type="submit">
          {mode === 'login' ? 'Login' : 'Create Account'}
        </button>
      </p>
      <p>
        {mode === 'login' && (
          <Link href="/?mode=signup">Create an account.</Link>
        )}
        {mode === 'signup' && (
          <Link href="/?mode=login">Login with existing account.</Link>
        )}
      </p>
    </form>
  );
}
```

## 4 Database Helper `(lib/user.js)`

 To connect NextAuth.js to your actual SQLite database, we need to replace the hardcoded user with a database query. We will use the better-sqlite3 library to look up the user by their email.

First, create a function to find a user in your SQLite table.

```javascript
import db from './db';

export function createUser(email, password) {
  const result = db
    .prepare('INSERT INTO users (email, password) VALUES (?, ?)')
    .run(email, password);
  return result.lastInsertRowid;
}

export function getUserByEmail(email) {
  return db.prepare('SELECT * FROM users WHERE email = ?').get(email)
}
```

### Sources
<a href="https://github.com/mschwarzmueller/nextjs-complete-guide-course-resources/blob/main/code/08-authentication/08-user-login/components/auth-form.js">Link User Login</a>
