---
title: A Small Go API Challenge
description: My notes on building a quick REST API using the net/http library in the Go programming language.
date: 2024-03-19
tags:
    - stdAPI
    - go
---

<span style="font-size: 1.25em; font-weight: bold;">Building APIs from scratch </span>isn't easy in many programming languages, but Go is one of my favorites for being able to build almost any kind of MVP using almost entirely standard library tools.

Commonly, we see the following packages:
- `net/http`
- `html/template`
- `time`
- `sql`

and the ever so used `encodings` package.


So, to get the ball rolling, I want to challenge myself to build a RESTful API that represents a bookshelf library using the Go 1.22 standard library to play around with some of the new features available in the `net/http` package.

### What will this API include?
- JSON Responses
- CRUD for Books
- CRUD for Created Groups of Books
- Some form of local database (either SQLite or Postgres depending on how I'm feeling)

### Are you going to be using any third-party libraries?
Yes, namely `sqlc` which counts less as a library and more as a code generation tool, because I don't want to transfer my SQL queries into Go code by hand.

I've been wanting to try it for a while, and I also want to focus more on the API bits than the database connection bits in a project for once.