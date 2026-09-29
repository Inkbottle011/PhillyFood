# PhillyFood

A restaurant discovery web app for Philadelphia, built by a team of 5 at a hackathon. Users can find nearby restaurants on an interactive map and sign up or log in through a JWT-secured API.

**Stack:** React · Google Maps Places API · Node.js · Express · PostgreSQL (Neon) · JWT · bcrypt

> **Status: hackathon prototype.** The map search and the authentication API work end to end on their own. The review screen is still front-end only, and the browser-to-API connection needs CORS configuration (see [What's next](#whats-next)).

## What's in it

**Frontend** (`Frontend/src/main/webapp`)
- Home screen with a Google Map that shows nearby restaurants as markers, with details on hover and click
- Login/register, profile, review, and rewards screens, built with React (loaded in the browser from a CDN)
- An API helper that keeps the access token in memory and retries once through `/refresh` when it expires

**Backend** (`Backend/src/server.js`)
- Registration and login with bcrypt-hashed passwords
- Short-lived JWT access tokens (15 minutes by default)
- Refresh tokens stored in PostgreSQL and sent as `httpOnly`, `sameSite=strict` cookies; logout deletes the stored token
- A protected route that requires a valid access token

### API

| Method | Route | Auth | Purpose |
|---|---|---|---|
| POST | `/users` | none | Register (`username`, `email`, `password`) |
| POST | `/login` | none | Log in; returns an access token and sets the refresh cookie |
| POST | `/refresh` | refresh cookie | Issue a new access token |
| POST | `/logout` | refresh cookie | Delete the refresh token and clear the cookie |
| GET | `/users` | access token | List usernames and emails |
| GET | `/db-test` | none | Check the database connection |

## Getting started

**Prerequisites:** Node.js 18+ and a PostgreSQL database (a free [Neon](https://neon.tech) project works).

### 1. Backend

```bash
cd Backend
npm install
cp .env.example .env     # then fill in DATABASE_URL and JWT_SECRET
```

Create the two tables the API expects:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL
);

CREATE TABLE refresh_tokens (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token TEXT UNIQUE NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);
```

Start the API on port 3000, then confirm it works at `http://127.0.0.1:3000/db-test`:

```bash
npm run dev
```

### 2. Frontend

The frontend is a set of static files in `Frontend/src/main/webapp`. Serve that folder with any static server, for example the VS Code Live Server extension (port 5500) or `npx serve`.

The map needs a Google Maps API key. Create one in Google Cloud with the **Maps JavaScript API** and **Places API** enabled, restrict it to your site's address, and put it in the `<script src="https://maps.googleapis.com/maps/api/js?key=...">` tag near the bottom of `index.html`.

## Project structure

```
Backend/
  src/server.js               API routes, auth, PostgreSQL access
  src/middleware/             Token verification
  .env.example                Environment variables to set
Frontend/src/main/webapp/
  index.html                  App shell and Google Maps loader
  _React_CGF/                 Home, Login, Profile, Review, Rewards screens
  _JS_reusable/api.js         API client with automatic token refresh
  docs/DB_proposal.pdf        Database design proposal
```

The `Frontend/src/main/java` folder is an unused Spring Boot scaffold with no build file; the app itself is the static files above.

## What's next

**Finish the core features**
- Store restaurants and reviews in PostgreSQL and add the API routes, then connect the review screen to them
- Configure CORS on the API so the frontend can call it from another origin (the `CORS_ORIGIN` setting is reserved for this)
- Rotate the refresh token each time it's used

**Security and cleanup**
- Stop logging request bodies, since they can contain passwords
- Load the Google Maps key from configuration instead of hardcoding it in `index.html`
- Remove the unused Spring Boot scaffold

**Quality and deployment**
- Add automated tests for the auth routes
- Deploy a hosted demo
