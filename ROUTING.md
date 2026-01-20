- [Routing & File-Based Structure](#1-routing--file-based-structure)
- [Component Architecture](#2-component-architecture)
- [Data Fetching & Caching](#3-data-fetching--caching)

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
