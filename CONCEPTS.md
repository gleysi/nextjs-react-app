# NodeJS & React

## The Core Definition

* **React** is a UI **library**. It handles the "view" layer but leaves decisions like routing and rendering strategies up to the developer. A JavaScript library for building user interfaces. It´s a client-side library.
* **Next.js** is a **framework** built on top of React. It comes with "opinions" and pre-configured tools to automatically solve common problems like performance, image optimization, and SEO.

# React vs. Next.js: Library or Framework?

| Feature | React (Library) | Next.js (Framework) |
| :--- | :--- | :--- |
| **Role** 🏗️ | **The engine.** The core library for building user interfaces. | **The whole car.** A complete "out-of-the-box" solution. |
| **Routing** 🛣️ | You must install a separate library (like `react-router`). | **Built-in.** File-system based. Just add a file to the folder to create a page. |
| **Rendering** 🚀 | **Client-Side (CSR):** The browser does the work. This can lead to slower initial loads. | **Server-Side (SSR) & Static:** The server prepares the page first. It’s much faster for the user. |
| **SEO** 🔍 | **Harder.** Search engines see an "empty" page before JavaScript runs. | **Excellent.** Search engines see the full content immediately. |
| **Backend** ⚙️ | Needs a separate backend (Node, Python, etc.) for APIs. | **Full-stack:** You can write backend API routes directly inside the project. |
