# Virbooks

## Educational Purpose

This project was created primarily for **educational and learning purposes**.  
While it is well-structured and could technically be used in production, it is **not intended for commercialization**.  
The main goal is to explore and demonstrate best practices, patterns, and technologies in software development.

## Description

**Virbooks** is a full-stack bookstore management application that lets users catalog, browse, and manage a personal book collection through a clean, responsive web interface.

### What it does

The application provides a single-page interface where users can add new books to their collection by filling out a form with the book's title, author, genre, description, and a cover image URL. Every book added is immediately persisted to a MongoDB database and displayed on the main page alongside the rest of the collection. Books can be removed at any time via a delete action directly from the book card.

The collection can be filtered by genre: the interface exposes a filter menu populated dynamically from the genres already present in the database, so the list of available filters always reflects the actual state of the collection without any manual configuration.

## Technologies used

FrontEnd:

1. React JS
2. Typescript
3. CSS3
4. HTML5
5. Vite

BackEnd:

1. Python -> Flask

Deploy:

1. Docker
2. Nginx
3. Gunicorn

Database:

1. MongoDB -> PyMongo

## Libraries used

### Frontend

#### Dependencies

```
"react": "^19.2.4"
"react-dom": "^19.2.4"
"react-icons": "^4.4.0"
```

#### devDependencies

```
"@eslint/js": "^9.0.0"
"@testing-library/dom": "^10.4.0"
"@testing-library/jest-dom": "^6.6.3"
"@testing-library/react": "^16.0.1"
"@testing-library/user-event": "^14.5.2"
"@types/jest": "^30.0.0"
"@types/node": "^22.0.0"
"@types/react": "^19.2.14"
"@types/react-dom": "^19.2.3"
"@vitejs/plugin-react": "^5.0.2"
"eslint": "^9.0.0"
"eslint-config-prettier": "^9.0.0"
"eslint-plugin-prettier": "^5.5.5"
"eslint-plugin-react-hooks": "^5.0.0"
"eslint-plugin-react-refresh": "^0.4.0"
"globals": "^15.0.0"
"husky": "^9.0.0"
"jest": "^30.3.0"
"jest-environment-jsdom": "^30.3.0"
"lint-staged": "^15.0.0"
"prettier": "^3.0.0"
"ts-jest": "^29.4.6"
"typescript": "^5.2.2"
"typescript-eslint": "^8.0.0"
"vite": "^7.1.6"
```

### Backend

#### Requirements.txt

```
flask==3.1.3
pymongo==4.16.0
pydantic==2.11.9
gunicorn==23.0.0
```

#### Requirements.dev.txt

```
pre-commit==4.3.0
pip-audit==2.7.3
ruff==0.11.12
```

#### Requirements.test.txt

```
pytest==8.4.2
pytest-env==1.1.5
pytest-cov==4.1.0
pytest-timeout==2.3.1
pytest-xdist==3.5.0
```

## Getting Started

With the stack and dependencies in mind, here's how to bring the project up locally.

1. Clone the repository with `git clone "repository link"`
2. Join to `virbooks-app` folder and execute: `npm install` or `yarn install` in the terminal
3. Copy each `.env.example` to `.env` (root, `virbooks-app/`, and `virbooks-api/`) and fill in the values described in the [Env Keys](#env-keys) section. Without this step the containers will not pick up the right configuration.
4. Go to the previous folder and execute: `docker-compose -f dev.docker-compose.yml build --no-cache` in the terminal
5. Once built, you must execute the command: `docker-compose -f dev.docker-compose.yml up --force-recreate` in the terminal

NOTE: You have to be standing in the folder containing the: `dev.docker-compose.yml` and you need to install `Docker Desktop` if you are in Windows.

### Pre-Commit for Development (Python)

NOTE: Install **pre-commit** inside: `virbooks-api` folder.

1. Once you're inside the virtual environment, let's install the hooks specified in the pre-commit. Execute: `pre-commit install`
2. Now every time you try to commit, the pre-commit lint will run. If you want to do it manually, you can run the command: `pre-commit run --all-files`

## Env Keys

The setup steps above reference these variables. Each one is consumed either by the frontend (Vite) or the backend (Flask + PyMongo) at startup.

1. `TZ`: Refers to the timezone setting for the container.
2. `VITE_API_URL`: Refers to the base URL of the backend API the frontend consumes.
3. `MONGO_HOST`: Specifies the hostname or address where the MongoDB server is located. In this case, `host.docker.internal` allows a Docker container to connect to the host machine.
4. `MONGO_PORT`: Defines the port on which the MongoDB server is listening for connections. The default MongoDB port is `27017`.
5. `MONGO_USER`: Indicates the username for authenticating with the MongoDB database.
6. `MONGO_PASS`: Contains the password associated with the user specified in `MONGO_USER` for authentication.
7. `MONGO_DB_NAME`: Specifies the name of the database to which the application will connect within the MongoDB server.
8. `MONGO_AUTH_SOURCE`: Defines the database where the user credentials will be verified. Typically set to `admin` when the credentials were created in that database.
9. `HOST`: Refers to the network interface where the backend API listens (e.g., 0.0.0.0 to allow external connections).
10. `PORT`: Refers to the port on which the backend API is exposed.

```ts
# Frontend Envs
TZ=America/Argentina/Buenos_Aires

VITE_API_URL=http://host.docker.internal:5050

# Backend Envs
TZ=America/Argentina/Buenos_Aires

MONGO_HOST=host.docker.internal
MONGO_PORT=27017
MONGO_USER=admin
MONGO_PASS=secret123
MONGO_DB_NAME=books
MONGO_AUTH_SOURCE=admin

HOST=0.0.0.0
PORT=5050
```

## Architecture & Design Patterns

With the project running and configured, this is how the pieces fit together internally.

The frontend is a React 19 + TypeScript SPA bundled with Vite. It communicates exclusively with the backend through a versioned REST API (`/api/v1/books/`). In development, Vite's proxy forwards those requests to the Flask server, so no CORS configuration is needed. In production, Nginx serves the static build and acts as a reverse proxy to the Gunicorn/Flask process.

The backend is a Flask 3 application structured in clear layers: blueprints handle routing, controllers parse requests and build responses, services contain the business logic (including duplicate-book detection before insertion), and data access objects (DAOs) execute all PyMongo queries. Input validation is handled by Pydantic v2 models, and a centralized decorator catches both Pydantic `ValidationError` and PyMongo errors, mapping them to structured JSON error responses with appropriate HTTP status codes.

Data is stored in MongoDB. Each book document holds title, author, genre, description, and image. The DAO layer serializes MongoDB's `ObjectId` to a plain string before returning data to the client, keeping the API transport format clean and predictable.

The entire stack runs in Docker via a single Compose file for development. A separate test Compose file spins up an isolated MongoDB instance on a different port exclusively for the test suite, ensuring the development database is never touched during testing.

### Virbooks Endpoints API

---

- **Endpoint Name**: Get Books
- **Endpoint Method**: GET
- **Endpoint Prefix**: /api/v1/books/
- **Endpoint Fn**: This endpoint obtains all the Books
- **Endpoint Params**: None

---

- **Endpoint Name**: Get Books by Genre
- **Endpoint Method**: GET
- **Endpoint Prefix**: /api/v1/books/:genre
- **Endpoint Fn**: This endpoint obtains all the Books
- **Endpoint Params**: 

```ts
{
  genre: string;
}
```

---

- **Endpoint Name**: Get Books All Genres
- **Endpoint Method**: GET
- **Endpoint Prefix**: /api/v1/books/genres
- **Endpoint Fn**: This endpoint obtains all the Genres
- **Endpoint Params**:  None

---

- **Endpoint Name**: Create Book
- **Endpoint Method**: POST
- **Endpoint Prefix**: /api/v1/books/
- **Endpoint Fn**: This endpoint create a new Book
- **Endpoint Body**:

```ts
{
    title: string;
    description: string;
    author: string;
    image: string;
    genre: string;
}
```

---

- **Endpoint Name**: Delete Book
- **Endpoint Method**: DELETE
- **Endpoint Prefix**: /api/v1/books/:id
- **Endpoint Fn**: This endpoint deletes a Book by id
- **Endpoint Params**: 

```ts
{
  id: string;
}
```

---

## Testing

### Frontend

1. Navigate to the project folder
2. Execute: `npm test`

For coverage report:

```bash
npm run test:coverage
```

### Backend

1. Join to the correct path of the clone and join to: `virbooks-api`
2. Execute: `python -m venv venv`
3. Execute in Windows: `venv\Scripts\activate`
4. Execute: `pip install -r requirements.txt`
5. Execute: `pip install -r requirements.test.txt`
6. Execute: `pytest --log-cli-level=INFO`

## Security Audit

Beyond running the test suite, both sides of the stack ship with tools to scan dependencies for known vulnerabilities.

### Backend (Python)

You can check your dependencies for known vulnerabilities using **pip-audit**.

1. Go to the repository folder
2. Activate your virtual environment
3. Execute: `pip install -r requirements.dev.txt`
4. Execute: `pip-audit -r requirements.txt`

### Frontend

#### npm audit

Check for vulnerabilities in dependencies:

```bash
npm audit
```

#### React Doctor

Run a health check on the project (security, performance, dead code, architecture):

```bash
npm run doctor
```

Use `--verbose` to see specific files and line numbers:

```bash
npm run doctor -- --verbose
```

## Known Issues

None at the moment.

## Portfolio Link

```ts
APP VERSION: 0.0.1
README UPDATED: 10/05/2026
AUTHOR: Diego Libonati
```

[`https://www.diegolibonati.com.ar/#/project/virbooks`](https://www.diegolibonati.com.ar/#/project/virbooks)
