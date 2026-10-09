# Gather — Campus Event Management

Gather is a responsive, full-stack campus event platform built for the ACM Student Chapter SRMIST recruitment task. Students can discover events, search and filter the lineup, create an account, reserve a place and manage their registrations. Organizers can create and remove events from their dashboard.

## Features

- Responsive event discovery with category, text and date filters
- Event details with schedule, venue, host and live remaining capacity
- Student accounts with scrypt-hashed passwords and seven-day HTTP-only sessions
- Registration and cancellation with duplicate and capacity checks
- Organizer event creation and deletion via role-protected endpoints
- SQLite persistence for users, events, registrations and sessions
- Seeded sample event lineup on the first run
- Built-in Node.js server; no third-party runtime packages required

## Requirements

- Node.js 22 or newer (uses the built-in `node:sqlite` module)

## Run locally

```bash
git clone <repository-url>
cd campus-events
node server.js
```

Open [http://localhost:3000](http://localhost:3000). The SQLite database is created at `data/campus-events.db` on first launch. Set `PORT` to change the port and `DATA_DIR` to choose a persistent database directory.

## Organizer setup

Set `ORGANIZER_EMAIL` and `ORGANIZER_PASSWORD` in the server environment. Then send a one-time POST request to `/api/admin/seed-organizer` with JSON `{ "key": "<the configured ORGANIZER_PASSWORD>" }`. Sign into the app using that email and password. Choose a strong password and keep these values private. The endpoint is disabled unless both environment variables are set.

Example (PowerShell):

```powershell
$env:ORGANIZER_EMAIL = 'organizer@example.edu'
$env:ORGANIZER_PASSWORD = 'use-a-long-private-password'
node server.js
```

In a second terminal, create the organizer account:

```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:3000/api/admin/seed-organizer -ContentType 'application/json' -Body '{"key":"use-a-long-private-password"}'
```

## API overview

| Method | Route | Purpose |
| --- | --- | --- |
| GET | `/api/events` | List upcoming events with registration counts |
| GET | `/api/events/:id` | Read one event and its current capacity |
| POST | `/api/register` | Create a student account |
| POST | `/api/login` | Sign in and start a session |
| POST | `/api/logout` | End the current session |
| GET | `/api/me` | Read the signed-in profile and registrations |
| POST | `/api/events/:id/register` | Reserve a place (authenticated) |
| DELETE | `/api/events/:id/register` | Cancel a reservation (authenticated) |
| POST | `/api/admin/events` | Create an event (organizer only) |
| PUT / DELETE | `/api/admin/events/:id` | Update or remove an event (organizer only) |
| POST | `/api/admin/seed-organizer` | One-time organizer setup (environment gated) |

## Project layout

```text
server.js          HTTP server, SQLite schema, authentication and API
public/index.html  Page structure
public/styles.css  Responsive visual system
public/app.js      Event browsing and account interactions
data/              Local SQLite database (created at runtime, git-ignored)
```

## Notes and next steps

Sample events use dates in October and November 2026. Replace the sample content with current campus events before launch. For a public deployment, use HTTPS, configure a durable `DATA_DIR`, set a private organizer credential, add rate limiting and CSRF protection, and consider moving sessions to secure expiring tokens or a dedicated session store. The server uses SQLite transaction boundaries and a unique registration constraint so simultaneous requests cannot overbook or duplicate a user's reservation.

## License

MIT — see [LICENSE](LICENSE).
