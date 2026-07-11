---
layout: post
title: Hunt for DevOps Roles, Day 3 - Tables, Constraints, and Getting Lost in the Sauce
subtitle: Writing SQL by Hand, Then Watching Claude Code Try to Rewrite My Whole App
cover-img: /assets/img/hunt-for-devops-day-3-cover.jpg
thumbnail-img: /assets/img/hunt-for-devops-day-3-thumb.jpg
share-img: /assets/img/hunt-for-devops-day-3-cover.jpg
tags:
  [
    devops,
    postgresql,
    sql,
    docker,
    psql,
    claude-code,
    ai-assisted-development,
    learning-in-public,
    tutorial-hell,
    python,
    containers,
    project-devlog,
  ]
author: Mark Truong
---

Hmmm, I think I want to keep watching until creating tables, this is good stuff. Setting up PostgreSQL from scratch, CLI `psql` commands, seeing the tables, how to create a database. All took me 40 min.

## Where Does the Connection String Even Go?

I asked Claude Code where to put the connection string in the codebase. There are datatypes when creating a table.

```sql
CREATE TABLE caption_history (
    id int,
    image_key VARCHAR(50),
    caption VARCHAR(255),
    tag VARCHAR(255),
    created_at TIMESTAMP,
);
```

I kept it simple, I was tempted to use AI, but writing code longhand style is pretty darn cool nowadays. Have `id` as int, then spam varchar, variable character, which can be any length of character up to 50 or 255.

Now, I'm just a bit confused about how the `image_key` works. What if there are more images than the number of permutations in `image_xxx`? Let's ask Claude.

## Getting Corrected on My Schema

```sql
CREATE TABLE caption_history (
    id          SERIAL PRIMARY KEY,
    image_key   VARCHAR(50) UNIQUE NOT NULL,
    caption     TEXT NOT NULL,
    tag         VARCHAR(255),
    created_at  TIMESTAMPTZ DEFAULT now()
);
```

![psql \d caption_history output showing id, image_key, caption, tag, and created_at columns with their constraints](/assets/img/hunt-for-devops-day-3-table-created.png)

Looks like I trolled and this is the right way to create it, and this is my todo on what to learn:

**What is a primary key?**
In this case, the `id` is what uniquely identifies this caption in this table.

**What is a constraint?**
Before someone inserts a new record in the table it must satisfy this constraint. `NOT NULL` means you should specify these columns, or else what's the point.

`image_key` should be unique, `caption` should be `TEXT` because the caption can run a little long, so 255 would be a cap that may overflow. `created_at` should be timezone aware with `DEFAULT now()`.

## Writing Data In

Alright, let's now start writing the db in. So how do you insert records into tables? The notation for this is:

```sql
INSERT INTO caption_history (
    image_key, caption, tag, created_at
) VALUES (
    %s, %s, %s, %s
)
```

So it doesn't look like the rest of this video will help. We need to figure out how to grab the caption from the LLM and have the LLM push the data into the database. Remember, I already connected the db to my codebase: how do you insert a row of data with Python into a PostgreSQL db?

## Handing It to Claude Code

Alright, so now I'm a little lost without a tutorial. Now we're going to use Claude Code to explain how they would implement this db into the code, and try to implement it myself, writing all the code.

I asked Claude Code:

> How would you implement a PostgreSQL database in this codebase for a caption history, timestamp, user supplied tag — this will have an image key and an ID — and have the Anthropic API grab the caption and input it into the database every time it's run to write a meme.

Claude Code's plan:

- **Backend (`db.py`)** — replace the connection-test stub with a real module: connection pool via `psycopg2`, reading `DATABASE_URL` from env, `init_db()` to create the captions table (id, image_key, raw_caption, meme_caption, tag, created_at), and `insert_caption()` / `get_history()` functions.
- **`docker-compose.yml`** — parameterize the postgres service via `.env` (db name/user/password) instead of hardcoded example, add a named volume for persistence, and expose port 5432 so the locally-run backend can reach it.
- **`main.py`** — call `db.init_db()` on startup, accept an optional tag field on both endpoints, hash the image bytes for `image_key`, and insert a row after each caption/memeify run.
- **`App.jsx`** — add a small tag input so users can actually supply the tag.

I'll add `psycopg2-binary` to `requirements.txt`. Let me check the existing `.env` to match variable naming conventions.

## Oh Boy…

So a couple main ideas: we gotta swap the PostgreSQL to Docker and run there, but the idea is once it's running, we can just connect it with `psycopg2`.

The idea here is as a software engineer, you really gotta dive deep, find videos, resources, anything: [https://www.youtube.com/watch?v=Q8iYj2ypWss&t=954s](https://www.youtube.com/watch?v=Q8iYj2ypWss&t=954s)

## Pulling Back to Actual DevOps Work

You know what, I think I'm getting lost in the sauce, this isn't exactly DevOps work, so I'm going to get the app to have PostgreSQL and get to the DevOps work. Maybe I'll have it use Postgres.app, and try and convert it into Docker myself, and learn what containerize means.

Also, the MinIO S3 bucket is probably on the table along with the Redis cache, but the idea here is to keep motivation high and be able to explain what I'm building.
