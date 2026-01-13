- [NextJS & React](#nextjs-&-react)
  - [The Core Definition](#the-core-definition)
  - [React vs. Next.js: Library or Framework?](#react-vs-nextjs-library-or-framework)
  - [Next.js Reserved Filenames (App Router)](#nextjs-reserved-filenames-app-router-)
    - [🛠️ Reserved Filenames Reference](#reserved-filenames-reference)
    - [💡 Visualizing the Hierarchy](#-visualizing-the-hierarchy)

# NextJS & React

## The Core Definition

* **React** is a UI **library**. It handles the "view" layer but leaves decisions like routing and rendering strategies up to the developer. A JavaScript library for building user interfaces. It´s a client-side library.
* **Next.js** is a **framework** built on top of React. It comes with "opinions" and pre-configured tools to automatically solve common problems like performance, image optimization, and SEO.

## React vs. Next.js: Library or Framework?

| Feature | React (Library) | Next.js (Framework) |
| :--- | :--- | :--- |
| **Role** 🏗️ | **The engine.** The core library for building user interfaces. | **The whole car.** A complete "out-of-the-box" solution. |
| **Routing** 🛣️ | You must install a separate library (like `react-router`). | **Built-in.** File-system based. Just add a file to the folder to create a page. |
| **Rendering** 🚀 | **Client-Side (CSR):** The browser does the work. This can lead to slower initial loads. | **Server-Side (SSR) & Static:** The server prepares the page first. It’s much faster for the user. |
| **SEO** 🔍 | **Harder.** Search engines see an "empty" page before JavaScript runs. | **Excellent.** Search engines see the full content immediately. |
| **Backend** ⚙️ | Needs a separate backend (Node, Python, etc.) for APIs. | **Full-stack:** You can write backend API routes directly inside the project. |

## Next.js Reserved Filenames (App Router) 📁

When working inside the `app/` directory (or any of its subfolders), Next.js uses a set of **reserved filenames**. These files have special meanings and define how your routes, UI, and error states behave.

> **Important:** These filenames are only reserved when created inside the `app/` folder. Outside of it, they are treated as regular files.

---

### 🛠️ Reserved Filenames Reference

| Filename | Purpose | Description |
| :--- | :--- | :--- |
| **`page.js`** | **New Page** | The core file for a route. For example, `app/about/page.js` creates the `/about` route. |
| **`layout.js`** | **Persistent UI** | Defines a layout that wraps sibling and nested pages. It preserves state and doesn't re-render on navigation. |
| **`not-found.js`** | **404 Fallback** | The UI shown when the `notFound()` function is triggered or a route doesn't exist. |
| **`error.js`** | **Error Boundary** | A fallback UI to handle runtime errors for a specific segment and its children. |
| **`loading.js`** | **Loading State** | Shown automatically while a route segment is fetching data (uses React Suspense internally). |
| **`route.js`** | **API Route** | Allows you to create an API endpoint that returns data (JSON) instead of JSX/HTML. |

---

### 💡 Visualizing the Hierarchy

Next.js uses a nested folder structure to determine how these files wrap around your content. For a single route, the UI is organized in this specific order:



1.  **`layout.js`** (The outermost wrapper)
2.  **`error.js`** (React Error Boundary)
3.  **`loading.js`** (React Suspense boundary)
4.  **`page.js`** (The actual content)
