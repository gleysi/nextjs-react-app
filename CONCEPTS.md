- [NextJS & React](#nextjs--react)
  - [The Core Definition](#the-core-definition)
  - [React vs. Next.js: Library or Framework?](#react-vs-nextjs-library-or-framework)
  - [Next.js Reserved Filenames (App Router)](#nextjs-reserved-filenames-app-router-)
    - [🛠️ Reserved Filenames Reference](#%EF%B8%8F-reserved-filenames-reference)
    - [💡 Visualizing the Hierarchy](#-visualizing-the-hierarchy)
  - [Metadata](#metadata)

# NextJS & React

## The Core Definition

* **React** is a UI **library**. It handles the "view" layer but leaves decisions like routing and rendering strategies up to the developer. A JavaScript library for building user interfaces. It´s a client-side library.
* **Next.js** is a **framework** built on top of React. It comes with "opinions" and pre-configured tools to automatically solve common problems like performance, image optimization, and SEO.
* 
---

## React vs. Next.js: Library or Framework?

| Feature | React (Library) | Next.js (Framework) |
| :--- | :--- | :--- |
| **Role** 🏗️ | **The engine.** The core library for building user interfaces. | **The whole car.** A complete "out-of-the-box" solution. |
| **Routing** 🛣️ | You must install a separate library (like `react-router`). | **Built-in.** File-system based. Just add a file to the folder to create a page. |
| **Rendering** 🚀 | **Client-Side (CSR):** The browser does the work. This can lead to slower initial loads. | **Server-Side (SSR) & Static:** The server prepares the page first. It’s much faster for the user. |
| **SEO** 🔍 | **Harder.** Search engines see an "empty" page before JavaScript runs. | **Excellent.** Search engines see the full content immediately. |
| **Backend** ⚙️ | Needs a separate backend (Node, Python, etc.) for APIs. | **Full-stack:** You can write backend API routes directly inside the project. |

---

## Next.js Reserved Filenames (App Router) 📁

When working inside the `app/` directory (or any of its subfolders), Next.js uses a set of **reserved filenames**. These files have special meanings and define how your routes, UI, and error states behave.

> **Important:** These filenames are only reserved when created inside the `app/` folder. Outside of it, they are treated as regular files.

 ### 🛠️ Reserved Filenames Reference

| Filename | Purpose | Description |
| :--- | :--- | :--- |
| **`page.js`** | **New Page** | The core file for a route. For example, `app/about/page.js` creates the `/about` route. |
| **`layout.js`** | **Persistent UI** | Defines a layout that wraps sibling and nested pages. It preserves state and doesn't re-render on navigation. |
| **`not-found.js`** | **404 Fallback** | The UI shown when the `notFound()` function is triggered or a route doesn't exist. |
| **`error.js`** | **Error Boundary** | A fallback UI to handle runtime errors for a specific segment and its children. |
| **`loading.js`** | **Loading State** | Shown automatically while a route segment is fetching data (uses React Suspense internally). |
| **`route.js`** | **API Route** | Allows you to create an API endpoint that returns data (JSON) instead of JSX/HTML. |

 ### 💡 Visualizing the Hierarchy

Next.js uses a nested folder structure to determine how these files wrap around your content. For a single route, the UI is organized in this specific order:

## Metadata

In Next.js, metadata is used to define the `<head>` elements of your page (like `title`, `description`, and Open Graph tags for social media). You can define it **statically** for fixed pages or **dynamically** for pages that depend on data (like a blog post).

### 1. Static Metadata
For pages where the content doesn't change based on parameters, you export a `metadata` object.

```js
// app/about/page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About Us | My Next.js App',
  description: 'Learn more about our team and mission.',
};

export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

### 2. Dynamic Metadata
For dynamic routes (e.g., `app/posts/[id]/page.tsx`), you use the `generateMetadata` function. This allows you to fetch data before the page renders to set the title and description specifically for that item.

```js
// app/posts/[slug]/page.tsx
import { Metadata } from 'next';
import { getPostBySlug } from '@/lib/posts';

type Props = {
  params: Promise<{ slug: string }>;
};

// Next.js will call this function automatically
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPostBySlug(slug);

  if (!post) {
    return { title: 'Post Not Found' };
  }

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.coverImage],
    },
  };
}

export default async function PostPage({ params }: Props) {
  const { slug } = await params;
  // ... render post content
}
````

### 3. Layout Metadata (The Template)
You can define a "base" metadata in your root `layout.tsx` This can include a `title.template` which automatically appends your site name to every sub-page title.

```js
// app/layout.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | My Brand Name', // The %s is replaced by the page title
    default: 'Welcome to My Brand Name', // Used if a page has no title
  },
  description: 'The best app for learning Next.js',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

#### Key Rules for Metadata
- **Server Side Only:** Metadata can only be exported from **Server Components.** You cannot use it in a file that has `"use client"` at the top.

- **Ordering:** Next.js starts at the page and moves up to the root layout. It merges the metadata, with the page-level metadata overriding the layout-level metadata if they share the same keys.

- **No Manual Tags:** Avoid using `<head>` or `<meta>` tags manually in your TSX/JSX; the Metadata API handles performance optimizations (like deduplication) for you.

1.  **`layout.js`** (The outermost wrapper)
2.  **`error.js`** (React Error Boundary)
3.  **`loading.js`** (React Suspense boundary)
4.  **`page.js`** (The actual content)
