---
title: Using sqlc instead of an ORM on small projects
description: My thoughts on using the sqlc code generation tool for golang on my small API project instead of a fully-featured ORM.
date: 2024-05-01
tags:
    - stdAPI
    - go
    - sql
---

<span style="font-size: 1.25em; font-weight: bold;">I've tried various ORMs</span>  in web development such as `Prisma`, `typeorm`, and `gorm` as they were very easy to get started with.
However, their goal of abstracting away SQL code into methods that you can chain results in really frustrating ways to interact with your database of choice, and it ends up being easier to write raw SQL instead of actually adapting to the library. At that point, where's the benefit of using a fully-featured ORM when you don't need most of the features?

In comes [`sqlc`](https://docs.sqlc.dev/en/latest/), which follows 3 simple steps:

1. You write SQL queries

2. You run `sqlc` to generate Go code that presents type-safe interfaces to those queries

3. You write application code that calls the methods `sqlc` generated

`sqlc` is a fairly interesting tool by concept. 
It lets developers generate code based on their SQL queries, and works very well for people who don't want to touch Object-Relational Mappers but also want some convenience in the typing and results of their queries.

For my API example, I had a fairly simple schema:
```sql
CREATE TABLE Books
(
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    title         TEXT NOT NULL,
    author        TEXT NOT NULL,
    publishDate   DATETIME,
    pageCount     INTEGER,
    readStatus    INTEGER,
    collection_id INTEGER,
    FOREIGN KEY (collection_id) REFERENCES Collections (id)
);

CREATE TABLE Collections
(
    id    INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL
);

CREATE TABLE Genres
(
    id   INTEGER PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);

CREATE TABLE BookGenres
(
    book_id  INTEGER,
    genre_id INTEGER,
    FOREIGN KEY (book_id) REFERENCES Books (id),
    FOREIGN KEY (genre_id) REFERENCES Genres (id),
    PRIMARY KEY (book_id, genre_id)
)
```

`sqlc` then used this schema to generate the following code:

```go
type Book struct {
	ID           int64
	Title        string
	Author       string
	Publishdate  sql.NullTime
	Pagecount    sql.NullInt64
	Readstatus   sql.NullInt64
	CollectionID sql.NullInt64
}

type BookGenre struct {
	BookID  sql.NullInt64
	GenreID sql.NullInt64
}

type Collection struct {
	ID    int64
	Title string
}

type Genre struct {
	ID   int64
	Name string
}
```

And when writing my queries, they will automatically be return typed to those structs.

For example, a simple query such as marking a book as **read** or **unread**:
```sql
-- name: MarkBookRead :exec
-- Mark a book as read
UPDATE Books
SET readStatus = 1 -- Assuming 1 represents "read" status, adjust if needed
WHERE id = ?;
```
```go
const markBookRead = `-- name: MarkBookRead :exec
UPDATE Books
SET readStatus = 1 -- Assuming 1 represents "read" status, adjust if needed
WHERE id = ?
`

// Mark a book as read
func (q *Queries) MarkBookRead(ctx context.Context, id int64) error {
	_, err := q.db.ExecContext(ctx, markBookRead, id)
	return err
}

```

However, `sqlc` can also generate structs to pass in information to, perhaps, perform an update operation on a table row:
```sql
-- name: UpdateBook :exec
-- Update an existing book
UPDATE Books
SET title         = ?,
    author        = ?,
    publishDate   = ?,
    pageCount     = ?,
    readStatus    = ?,
    collection_id = ?
WHERE id = ?;
```
```go
const updateBook = `-- name: UpdateBook :exec
UPDATE Books
SET title         = ?,
    author        = ?,
    publishDate   = ?,
    pageCount     = ?,
    readStatus    = ?,
    collection_id = ?
WHERE id = ?
`

type UpdateBookParams struct {
	Title        string
	Author       string
	Publishdate  sql.NullTime
	Pagecount    sql.NullInt64
	Readstatus   sql.NullInt64
	CollectionID sql.NullInt64
	ID           int64
}

// Update an existing book
func (q *Queries) UpdateBook(ctx context.Context, arg UpdateBookParams) error {
	_, err := q.db.ExecContext(ctx, updateBook,
		arg.Title,
		arg.Author,
		arg.Publishdate,
		arg.Pagecount,
		arg.Readstatus,
		arg.CollectionID,
		arg.ID,
	)
	return err
}
```
This is where I see `sqlc` shine, as now my parameters are separate from my actual struct, and I didn't have to write them myself, they were automatically generated for me.

I'm no expert in SQL, but I find it reasonable to be able to skip this kind of boilerplate code which just executes the sql query I end up having to write regardless.

This is a strange kind of code to work with, as it's not what I've written but I think it's a fun choice to try on this project. Hopefully I'll have more insight towards the end of this series of blog posts though, so tune in for that in a month or so.