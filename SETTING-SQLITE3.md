- [Setting up better-sqlite3]()
- [1. Installation]()
- [2. Initialize the Database]()
- [3. Creating Data Access Logic]()
- [4. Using Data in a Server Component]()

# Setting up better-sqlite3

## 1. Installation
Install the package via npm or yarn.

`npm install better-sqlite3`

## 2. Initialize the Database
Create a separate file (e.g., lib/db.js) to initialize the database and prevent multiple connections from being created during development.

```js
// lib/db.js
import sqlite from 'better-sqlite3';

// Creates or opens the 'database.db' file in your project root
const db = sqlite('database.db');

// Basic setup: Create a table if it doesn't exist
db.prepare(`
  CREATE TABLE IF NOT EXISTS meals (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    slug TEXT NOT NULL UNIQUE,
    title TEXT NOT NULL,
    image TEXT NOT NULL,
    summary TEXT NOT NULL,
    instructions TEXT NOT NULL,
    creator TEXT NOT NULL,
    creator_email TEXT NOT NULL
  )
`).run();

export default db;
```

## 3. Creating Data Access Logic
It is best practice to keep your SQL queries in a separate service file. This keeps your Server Components clean.

```js
// lib/meals.js
import db from './db';

// Fetching all data
export function getMeals() {
  return db.prepare('SELECT * FROM meals').all();
}

// Fetching a single item by slug
export function getMeal(slug) {
  return db.prepare('SELECT * FROM meals WHERE slug = ?').get(slug);
}

// Inserting data (usually called by a Server Action)
export function saveMeal(meal) {
  return db.prepare(`
    INSERT INTO meals 
      (title, summary, instructions, creator, creator_email, image, slug)
    VALUES (
      @title,
      @summary,
      @instructions,
      @creator,
      @creator_email,
      @image,
      @slug
    )
  `).run(meal);
}
```

## 4. Using Data in a Server Component
Because Server Components are `async`, you can call these functions directly.

```js
// app/meals/page.js
import { getMeals } from '@/lib/meals';

export default async function MealsPage() {
  const meals = getMeals(); // better-sqlite3 is synchronous!

  return (
    <main>
      <h1>Check out our meals</h1>
      <ul>
        {meals.map((meal) => (
          <li key={meal.id}>{meal.title}</li>
        ))}
      </ul>
    </main>
  );
}
```
