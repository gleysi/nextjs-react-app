- [Routing & File-Based Structure](#1-routing--file-based-structure)
    - [Reserved File Names](https://github.com/gleysi/nextjs-react-app/blob/main/ROUTING.md#reserved-file-names)
    - [Advanced Routing Patterns](https://github.com/gleysi/nextjs-react-app/blob/main/ROUTING.md#advanced-routing-patterns)
        - Dynamic Routes
        - Route Groups
        - Parallel Routes
        - Intercepting Routes
- [Component Architecture](#2-component-architecture)
    - [Server vs. Client Components](https://github.com/gleysi/nextjs-react-app/blob/main/ROUTING.md#server-vs-client-components) 
- [Data Fetching & Caching](#3-data-fetching--caching)
    - [Caching and Streaming](https://github.com/gleysi/nextjs-react-app/blob/main/ROUTING.md#caching-and-streaming)
        - fetch
        - suspense 
- [Data Mutation (Server Actions)](#4-data-mutation-server-actions)
    - [1. The Server Action (lib/actions.js)](#1-the-server-action-libactionsjs)
    - [The Form Component (Using Hooks)](#2-the-form-component-using-hooks)
    - [Core Hooks for Actions useFormStatus, useActionState](#core-hooks-for-actions)
    - [How Data Mutation Works here:](#how-data-mutation-works-here)

## 1. Routing & File-Based Structure
Next.js uses a directory-based router where folders define the URL segments.

### Reserved File Names
* `page.tsx`: The UI unique to a route.
* `layout.tsx`: Shared UI for a segment and its children (remains interactive/maintains state).
* `loading.tsx`: An automated loading UI using React Suspense.
* `error.tsx`: An error boundary for a specific segment.
* `not-found.tsx`: UI for 404 cases within a segment.

### Advanced Routing Patterns
* **Dynamic Routes `[slug]`**: Captures variable path segments (e.g., `/blog/my-post`).

Dynamic routes allow you to create pages based on external data (like a slug or ID). You wrap the folder name in square brackets: [id].

**File Structure:** - app/blog/[slug]/page.js

```tsx
    // app/blog/[slug]/page.js
    export default function BlogPost({ params }) {
    const { slug } = params; // e.g., /blog/hello-world -> slug = 'hello-world'
  
    return <h1>Reading: {slug}</h1>;
    }
```
* **Route Groups `(folderName)`**: Used to organize files (e.g., `(admin)`) without affecting the URL path.

    **File Structure:**

    - app/(auth)/login/page.js -> Accessible at /login
    - app/(auth)/register/page.js -> Accessible at /register

  ```tsx
    // app/(auth)/layout.js
    // This layout only applies to login and register
    export default function AuthLayout({ children }) {
    return (
        <div className="auth-container">
        <nav>Auth Header</nav>
        {children}
        </div>
    );
    }
  ```


* **Parallel Routes `@folder`**: Allows rendering multiple pages in the same layout simultaneously. You define them using the `@slot` convention.

    **File Structure:**

    - app/dashboard/layout.js
    - app/dashboard/@team/page.js
    - app/dashboard/@analytics/page.js

    ```js
    // app/dashboard/layout.js
    export default function DashboardLayout({ children, team, analytics }) {
    return (
        <>
        {children} {/* Renders app/dashboard/page.js */}
        <div className="grid">
            <section>{team}</section>      {/* Renders @team/page.js */}
            <section>{analytics}</section> {/* Renders @analytics/page.js */}
        </div>
        </>
        );
    }
    ```
    - **Use Case:** Complex dashboards where different sections (analytics, team feed, chat) load independently.

* **Intercepting Routes `(.)folder`**: Allows you to load a route from another part of your app inside the current layout (e.g., a modal overlaying the background gallery).
You use `(.)` to match segments on the same level or `(..)` for one level above.

    **File Structure:**

    - app/feed/page.js (List of photos)
    - app/photo/[id]/page.js (Full page for a photo)
    - app/feed/@modal/(.)photo/[id]/page.js (The intercepted modal)

    ```js
    // app/feed/@modal/(.)photo/[id]/page.js
    export default function PhotoModal({ params }) {
    return (
        <div className="modal-overlay">
        <div className="modal-content">
            <h2>Photo {params.id}</h2>
            <p>This is intercepted! Refreshing the page takes you to the full route.</p>
        </div>
        </div>
    );
    }
    ```
---

## 2. Component Architecture
Next.js utilizes **React Server Components (RSC)** by default.

### Server vs. Client Components
* **Server Components**: Fetch data directly, keep large dependencies on the server, and improve SEO.
* **Client Components**: Use the `"use client"` directive. Required for hooks (`useState`, `useEffect`) and browser APIs (event listeners).

> **Rule of Thumb**: Fetch data in Server Components and pass it down to Client Components for interactivity.

---

## 3. Data Fetching & Caching
In the App Router, you fetch data directly inside Server Components using `async/await`.

### Caching and Streaming
* **Fetch**: Next.js extends the native `fetch` to include caching.
* **Suspense**: Use `<Suspense>` to provide granular loading states for specific parts of a page rather than the whole screen.

```tsx
// app/products/page.tsx
import { Suspense } from 'react';

async function ProductList() {
  const res = await fetch('https://api.example.com/products', { 
    next: { revalidate: 3600 } // Cache for 1 hour
  });
  const data = await res.json();
  
  return <ul>{data.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}

export default function Page() {
  return (
    <section>
      <h1>Our Products</h1>
      <Suspense fallback={<p>Loading items...</p>}>
        <ProductList />
      </Suspense>
    </section>
  );
}
```
## 4. Data Mutation (Server Actions)

**Data Mutation** refers to the process of changing data on the server (Creating, Updating, or Deleting) and then updating the UI to reflect those changes.

S**erver Actions** are the bridge. They allow you to write a function that runs on the server but can be called directly from your client-side form.

### 1. The Server Action (`lib/actions.js`)
This is where the actual mutation happens. We use useActionState to handle the response from this function.

```typescript
'use server'
import { saveMeal } from './meals';
import { revalidatePath } from 'next/cache';

export async function shareMeal(prevState, formData) {
  const meal = {
    title: formData.get('title'),
    creator: formData.get('name'),
  };

  // Validation logic
  if (!meal.title || meal.title.length < 5) {
    return { message: 'Invalid title. Must be at least 5 characters.' };
  }

  try {
    await saveMeal(meal);
    // Tell Next.js to clear the cache and fetch fresh data
    revalidatePath('/meals'); 
    return { message: 'Success!' };
  } catch (error) {
    return { message: 'Database error occurred.' };
  }
}
```

### 2. The Form Component (Using Hooks)
To use `useActionState` and `useFormStatus`, your component must be a **Client Component.**

**A. The Submit Button** (`components/submit-button.js`)

`useFormStatus` only works if it is called in a component inside a `<form>`. We usually split the button into its own small component.

```typescript
'use client'
import { useFormStatus } from 'react-dom';

export function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Share Meal'}
    </button>
  );
}
```

**B. The Main Form** (`app/meals/share/page.js`)

`useActionState` (formerly `useFormState`) tracks the result of the action (like error messages) and the loading state.

```typescript
'use client'
import { useActionState } from 'react';
import { shareMeal } from '@/lib/actions';
import { SubmitButton } from '@/components/submit-button';

export default function ShareMealPage() {
  // state: the object returned from the action
  // formAction: the enhanced function to put in your form's action prop
  const [state, formAction] = useActionState(shareMeal, { message: null });

  return (
    <form action={formAction}>
      <p>
        <label htmlFor="name">Your Name</label>
        <input type="text" id="name" name="name" required />
      </p>
      <p>
        <label htmlFor="title">Meal Title</label>
        <input type="text" id="title" name="title" required />
      </p>

      {/* Display error/success messages from the server */}
      {state.message && <p className="status">{state.message}</p>}

      <SubmitButton />
    </form>
  );
}
```

### Core Hooks for Actions
* **`useFormStatus`**: Provides the pending state of a form submission (e.g., `pending: true`). **Note:** This must be used in a component that is a **child** of the `<form>` element.
* **`useActionState`** (formerly `useFormState`): Used to handle the state returned from the server, such as validation errors, success messages, or the previous form data.

### How Data Mutation Works here:
1. **User submits**: `useFormStatus` sets `pending` to true, disabling the button.

2. **Action runs:** The `shareMeal` function executes on the server, talking to your database.

3. **Revalidation:** `revalidatePath` tells Next.js the data has changed, so the list of meals is updated.

4. **State update:** The server returns a message, and `useActionState` updates the UI with the result.
