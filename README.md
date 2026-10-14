# Anagram

Anagram is a full-stack application for browsing a dictionary, finding words
that are anagrams of a given word, and managing the word list. The backend
loads and normalizes dictionary words into PostgreSQL and provides the API;
this Angular frontend provides the interface for searching and managing them.

The frontend is in this repository. The Spring Boot backend is maintained
separately in
[Solomon-Moyo756/anagram](https://github.com/Solomon-Moyo756/anagram).

## Features

- Loads the word list automatically when the frontend opens.
- Searches and paginates the word list.
- Looks up anagrams for a selected word.
- Displays statistics grouped by word length.
- Adds and deletes words through the backend API.
- Serves the production Angular application with Nginx in Docker.
- Proxies API requests through Nginx so the browser can use the frontend's
  origin for both the UI and API.

## How it works

At backend startup, the service reads its provided `dictionary.txt`, normalizes
the words, and seeds the database. Normalization establishes a consistent word
format for processing and lookup. The backend also builds an in-memory
hash-map index for anagram lookup; hash-map access is average O(1), while
returning matches still depends on the number of results.

The backend uses Liquibase to track database changes with versioned change
sets. This adds migration overhead for a small application, but gives database
changes a repeatable, auditable history.

The backend caches word-list and statistics results to avoid repeatedly
querying or recalculating them. Word changes invalidate the affected cached
results so subsequent requests can return up-to-date data.

## Technologies

| Area | Technologies |
| --- | --- |
| Frontend | Angular 15.2, Angular Material, TypeScript, RxJS |
| Backend | Java, Spring Boot, Spring Data JPA |
| Database and migrations | PostgreSQL, Liquibase |
| Testing | Mockito (backend tests) |
| Containerization | Docker, Docker Compose, Nginx |

## Requirements

For local development:

- Node.js 18.10 or later in the Node 18 line
- npm 9 or later
- A running backend API and PostgreSQL database

For the containerized setup:

- Docker with the Compose plugin
- A strong database password configured as `DB_PASSWORD`

Angular 15 is no longer supported upstream and does not receive security fixes.
This project uses it for compatibility with the existing application.

## Run locally

Install the frontend dependencies and start the development server:

```bash
npm ci
npm start
```

Open [http://localhost:4200](http://localhost:4200). The Angular development
server proxies `/anagrams` requests to `http://localhost:8080` using
[`proxy.conf.json`](./proxy.conf.json). Start the Spring Boot backend first;
its API must be reachable at that address.

To create a production build:

```bash
npm run build
```

The generated files are written to `dist/anagram-frontend`. The frontend
currently has no `test` script configured in `package.json`.

## API

The frontend calls the following endpoints under `/anagrams`:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/anagrams/words` | Retrieve all words for the word list. |
| `GET` | `/anagrams/stats` | Retrieve statistics grouped by word length. |
| `GET` | `/anagrams/anagram-for-word?word={word}` | Find anagrams for the supplied word. |
| `POST` | `/anagrams/add-word?value={word}` | Add a word to the dictionary. |
| `DELETE` | `/anagrams/delete-word?value={word}` | Delete a word from the dictionary. |

The read endpoints return data in the backend's response format, which
includes `data` and `executionTimeMs`. Adding or deleting a word changes the
database and therefore can change later word-list, statistics, and anagram
results.

## Docker

The frontend image is built from the [`Dockerfile`](./Dockerfile). It compiles
the Angular application and serves the result with Nginx. The Nginx
configuration serves the Angular routes and forwards `/anagrams` requests to
the Compose service named `backend` on port 8080.

To build the image locally from this repository:

```bash
docker build -t solomon756/anagram-frontend:latest .
```

The backend repository contains the Compose setup for running the frontend,
backend, and PostgreSQL together. From that repository, configure `DB_PASSWORD`
in the environment (or its `.env` file) and run:

```bash
docker compose up --build -d
```

The Compose setup uses the following images:

- `solomon756/anagram-frontend:latest`
- `solomon756/anagram-backend:latest`
- `postgres:18`

When the frontend image is not already available to Docker on the host, build
it locally or publish it to Docker Hub before starting Compose. Once running,
the frontend is available at [http://localhost](http://localhost); the API is
also exposed on port 8080 by the Compose configuration.

Useful Compose commands (run from the backend repository) include:

```bash
docker compose logs -f
docker compose down
```

`docker compose down` leaves the PostgreSQL named volume intact. Do not put a
real database password in a committed file.

## Repository notes

This repository contains the Angular frontend and its Docker build files. The
backend repository contains the Spring Boot application, dictionary input,
Liquibase change sets, and backend Docker Compose configuration. Refer to the
[backend repository](https://github.com/Solomon-Moyo756/anagram) for its
backend-specific setup and implementation details.
