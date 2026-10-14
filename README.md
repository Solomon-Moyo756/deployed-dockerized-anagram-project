# Anagram

Anagram is a full-stack application for browsing a dictionary, finding words
that are anagrams of a given word, and managing the word list. The Spring Boot
backend loads and normalizes dictionary words into PostgreSQL and provides the
API; the Angular frontend provides the interface for searching and managing
them.

This repository contains the Docker Compose configuration for running the
published frontend and backend images with PostgreSQL. The application source
code is maintained separately in the
[backend repository](https://github.com/Solomon-Moyo756/anagram) and the
[frontend repository](https://github.com/Solomon-Moyo756/anagram-frontend).

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

To run the application images with Docker Compose:

- Docker Engine with the Compose plugin
- A strong database password configured as `DB_PASSWORD`

To develop the applications from source, also see the requirements in each
application repository. The frontend currently uses Node.js 18.10 or later in
the Node 18 line and npm 9 or later; the backend requires Java 17.

Angular 15 is no longer supported upstream and does not receive security fixes.
The frontend uses it for compatibility with the existing application.

## Run locally with Docker Compose

The simplest way to run the complete application is to use the prebuilt images
defined in this repository's [`docker-compose.yaml`](./docker-compose.yaml).
You do not need to clone the backend or frontend repositories or install Java,
Node.js, or npm for this option.

Set `DB_PASSWORD` in your shell environment or in a local `.env` file beside
`docker-compose.yaml`. Keep the password private and do not commit it. Then run
these commands from this repository:

```bash
docker compose up -d
```

Compose starts the frontend and backend images with PostgreSQL. If an image is
not already available locally, Compose pulls it from the registry named by its
image reference. This Compose file uses prebuilt images; it does not build the
applications from source. When the services are ready, open
[http://localhost](http://localhost). The frontend is served on port 80, and
the backend API is exposed on port 8080.

To fetch the latest published images before starting, run
`docker compose pull` first. If you built images locally but have not published
them, make sure their tags match those in `docker-compose.yaml` and skip the
pull command.

To check service logs or stop the application:

```bash
docker compose logs -f
docker compose down
```

`docker compose down` leaves the PostgreSQL named volume intact. To also delete
the database data, run `docker compose down --volumes`.

## Develop the applications from source

To change or run the Angular and Spring Boot applications from their source
code, clone the
[backend repository](https://github.com/Solomon-Moyo756/anagram) and the
[frontend repository](https://github.com/Solomon-Moyo756/anagram-frontend)
separately. The backend also needs PostgreSQL. Its
[`docker-compose.yml`](https://github.com/Solomon-Moyo756/anagram/blob/master/docker-compose.yml)
starts the database service; run that Compose file from the backend repository.

Start the backend from its repository with the included Maven wrapper:

```bash
./mvnw spring-boot:run
```

On Windows, use `.\mvnw.cmd spring-boot:run` instead. Once the API is running
on port 8080, install the frontend dependencies and start its development
server from the frontend repository:

```bash
npm ci
npm start
```

Open [http://localhost:4200](http://localhost:4200). The Angular development
server proxies `/anagrams` requests to the backend at `http://localhost:8080`.
For source-specific setup and build details, refer to the READMEs in the
[backend](https://github.com/Solomon-Moyo756/anagram) and
[frontend](https://github.com/Solomon-Moyo756/anagram-frontend) repositories.

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

## Repository notes

This repository contains the Compose configuration for the frontend, backend,
and database services. The application source, build files, and development
instructions are in the separate
[frontend](https://github.com/Solomon-Moyo756/anagram-frontend) and
[backend](https://github.com/Solomon-Moyo756/anagram) repositories.
