<img src="https://socialify.git.ci/siyamthandagwamanda/Library-Management-System-PostgreSQL/image?language=1&owner=1&name=1&stargazers=1&theme=Light" alt="Library-Management-System-PostgreSQL" width="640" height="320" />

# LibraryDB — PostgreSQL Project

A relational database project built with **PostgreSQL 18** and managed through **pgAdmin4**, modeling a simple library system with authors, books, and patrons.

## Tech Stack

- **Database:** PostgreSQL 18
- **Admin Tool:** pgAdmin4

## Getting Started

### 1. Connect to the Server

Open pgAdmin4 and connect to your PostgreSQL server:

- Server: `PostgreSQL 18`
- Enter the password for the `postgres` user when prompted.

> **Note:** If you see *"The application has lost the database connection"*, this usually means the connection was idle and forcibly disconnected, the server was restarted, or the session timed out. Choose to establish a new session and reconnect.

### 2. Create the Database

In the Browser panel:

1. Right-click **Databases** → **Create** → **Database** (or use the shortcut `Alt + Shift + N`)
2. Name it `LibraryDB`
3. Click **Save**

### 3. Create the Tables

Right-click **LibraryDB** → **Query Tool**, then run the script below.

Tables must be created in this order — `authors` first, since `books` references it via foreign key:

```sql
-- 1. Authors table
CREATE TABLE IF NOT EXISTS authors (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    nationality VARCHAR(100),
    birth_year INT,
    death_year INT
);

-- 2. Books table (FK -> authors.id, ON DELETE CASCADE)
-- Deleting an author also deletes their books
CREATE TABLE IF NOT EXISTS books (
    id INT PRIMARY KEY,
    title VARCHAR(100) UNIQUE,
    genres TEXT[],
    published_year INT,
    available BOOL,
    author_id INT REFERENCES authors(id) ON DELETE CASCADE
);

-- 3. Patrons table
CREATE TABLE IF NOT EXISTS patrons (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    borrowed_books INT[]
);
```

Run the script using the **Execute (▶)** button. A successful run returns:

```
Query returned successfully.
```

**Why `ON DELETE CASCADE` matters:** without it, deleting an author who still has books linked to them raises:

```
ERROR:  update or delete on table "authors" violates foreign key constraint "books_author_id_fkey" on table "books"
Key (id)=(4) is still referenced from table "books".
```

Adding `ON DELETE CASCADE` to the `author_id` foreign key ensures a book's author can be deleted cleanly, and its books are removed along with it.

---

## Sprint 1 — Project Setup

- Created `LibraryDB` database
- Created `authors`, `books`, and `patrons` tables with the schema above

## Sprint 2 — Insert Data

Data must be inserted in dependency order: **authors before books**, since books reference `author_id`. (Books before patrons isn't enforced by a constraint, but keeps `borrowed_books` references meaningful.)

```sql
INSERT INTO authors (id, name, nationality, birth_year, death_year) VALUES
(1, 'George Orwell', 'British', 1903, 1950),
(2, 'Harper Lee', 'American', 1926, 2016),
(3, 'F. Scott Fitzgerald', 'American', 1896, 1940),
(4, 'Aldous Huxley', 'British', 1894, 1963),
(5, 'J.D. Salinger', 'American', 1919, 2010),
(6, 'Herman Melville', 'American', 1819, 1891),
(7, 'Jane Austen', 'British', 1775, 1817),
(8, 'Leo Tolstoy', 'Russian', 1828, 1910),
(9, 'Fyodor Dostoevsky', 'Russian', 1821, 1881),
(10, 'J.R.R. Tolkien', 'British', 1892, 1973);

INSERT INTO books (id, title, author_id, genres, published_year, available) VALUES
(1, '1984', 1, ARRAY['Dystopian', 'Political Fiction'], 1949, TRUE),
(2, 'To Kill a Mockingbird', 2, ARRAY['Southern Gothic', 'Bildungsroman'], 1960, TRUE),
(3, 'The Great Gatsby', 3, ARRAY['Tragedy'], 1925, TRUE),
(4, 'Brave New World', 4, ARRAY['Dystopian', 'Science Fiction'], 1932, TRUE),
(5, 'The Catcher in the Rye', 5, ARRAY['Realist Novel', 'Bildungsroman'], 1951, TRUE),
(6, 'Moby-Dick', 6, ARRAY['Adventure Fiction'], 1851, TRUE),
(7, 'Pride and Prejudice', 7, ARRAY['Romantic Novel'], 1813, TRUE),
(8, 'War and Peace', 8, ARRAY['Historical Novel'], 1869, TRUE),
(9, 'Crime and Punishment', 9, ARRAY['Philosophical Novel'], 1866, TRUE),
(10, 'The Hobbit', 10, ARRAY['Fantasy'], 1937, TRUE);
```

> **Note:** Only run the `INSERT INTO books` statement once — running it twice fails on the `UNIQUE` constraint on `title` (and the `id` primary key).

```sql
INSERT INTO patrons (id, name, email, borrowed_books) VALUES
(1, 'Alice Johnson', 'alice@example.com', ARRAY[]::INT[]),
(2, 'Bob Smith', 'bob@example.com', ARRAY[1, 2]),
(3, 'Carol White', 'carol@example.com', ARRAY[]::INT[]),
(4, 'David Brown', 'david@example.com', ARRAY[3]),
(5, 'Eve Davis', 'eve@example.com', ARRAY[]::INT[]),
(6, 'Frank Moore', 'frank@example.com', ARRAY[4, 5]),
(7, 'Grace Miller', 'grace@example.com', ARRAY[]::INT[]),
(8, 'Hank Wilson', 'hank@example.com', ARRAY[6]),
(9, 'Ivy Taylor', 'ivy@example.com', ARRAY[]::INT[]),
(10, 'Jack Anderson', 'jack@example.com', ARRAY[7, 8]);
```

## Sprint 3 — Read Operations

**Get all books**
```sql
SELECT * FROM books;
```

**Get a book by title**
```sql
SELECT * FROM books
WHERE title = 'The Catcher in the Rye';
```

**Get all books by a specific author**
```sql
SELECT * FROM books
WHERE author_id = 8;
```

**Get all available books**
```sql
SELECT * FROM books
WHERE available = true;
```

## Sprint 4 — Update Operations

**Mark a book as borrowed**
```sql
UPDATE books
SET available = false
WHERE id = 4;
```

**Add a new genre to an existing book** (appends to the end of the array)
```sql
UPDATE books
SET genres = array_append(genres, 'Fiction')
WHERE id = 3;
```

**Add a genre at the start of the array instead** (left-to-right / prepend logic)
```sql
UPDATE books
SET genres = array_prepend('Fiction', genres)
WHERE id = 3;
```

**Add a borrowed book to a patron's record**
```sql
UPDATE patrons
SET borrowed_books = array_append(borrowed_books, 4)
WHERE id = 2;
```

## Sprint 5 — Delete Operations

**Delete a book by title**
```sql
DELETE FROM books
WHERE title = 'Moby-Dick';
```

**Delete an author by ID** — thanks to `ON DELETE CASCADE`, this also removes all of that author's books:
```sql
DELETE FROM authors
WHERE id = 5;
```

## Sprint 6 — Advanced Queries

**Find books published after 1950**
```sql
SELECT * FROM books
WHERE published_year > 1950;
```

**Find all American authors**
```sql
SELECT * FROM authors
WHERE nationality = 'American';
```

**Set all books as available**
```sql
UPDATE books
SET available = true;
```

**Find all books that are available AND published after 1930**
```sql
SELECT * FROM books
WHERE available = true AND published_year > 1930;
```

**Find authors whose names contain "George"** (case-insensitive partial match)
```sql
SELECT * FROM authors
WHERE name ILIKE '%George%';
```

**Increment a book's published year by 1** (e.g. the book published in 1869)
```sql
UPDATE books
SET published_year = published_year + 1
WHERE published_year = 1869;
```


## Notes / Fixes Applied

While consolidating the working notes into this README, a few small corrections were made to match each query's stated intent:
- "Find books published after 1950" now filters on `> 1950` (was `> 1930`).
- "Find all American authors" now filters on `nationality = 'American'` (was `'British'`).
- "Find authors whose names contain George" now uses `'%George%'` (was `'%Aldous%'`).
- The duplicated `INSERT INTO books` block in Sprint 2 was removed to avoid a unique-constraint violation.
- Added the missing SQL for "Increment the published year 1869 by 1."
- Added the missing `INSERT INTO patrons` statement to Sprint 2.