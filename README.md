# Techify Portal — Local Backend + Frontend

This small project provides a simple Express backend (SQLite) with JWT authentication and role-based access control, and a frontend that can use the backend or fall back to localStorage when the server isn't running.

Quick start (Windows PowerShell):

1. Install dependencies

```powershell
cd "c:\Users\LOQ\Desktop\Padmavyuh 4.0 Hackathon Round 2\App Developement\sol 1"
npm install
```

2. Start the server

```powershell
npm start
# Server runs on http://localhost:3000
```

3. Open `index.html` in a browser (double-click file or serve it from a static server). The frontend will try to reach `http://localhost:3000/api` and will fall back to localStorage if the API is not available.

Notes:
- A seeded admin user is created on first run: username `admin`, password `adminpass` (change secret in `server.js` for production).
- The database file `techify.db` will be created in the same folder.

## ER-based API endpoints

- `POST /api/tvshows` — create tv show { Title, Description }
- `GET /api/tvshows` — list tv shows
- `GET /api/tvshows/:id` — get tv show by id
- `PUT /api/tvshows/:id` — update tv show
- `DELETE /api/tvshows/:id` — delete tv show (Admin only)

- `POST /api/seasons` — create season { SeasonNumber, SeasonDescription, DateStarted, DateEnded, Title, TVShowID }
- `GET /api/seasons/:id` — get season

- `POST /api/episodes` — create episode { Episodetitle, EpisodeDescription, Rating, DatePublished, SeasonID }
- `GET /api/episodes/:id` — get episode

- `POST /api/actors` — create actor/cast { ActorsFirstName, ActorsLastName }
- `GET /api/actors/:id` — get actor

- `POST /api/crew` — create crew { FirstName, LastName, PersonDefination }
- `GET /api/crew/:id` — get crew

- `POST /api/screentime` — create screentime record { ActorID, EpisodeNumber, StartTime, EndTime, RoleName, RoleType }

All endpoints require authentication (Bearer token). Server enforces referential integrity for Season -> TVShow, Episode -> Season, ScreenTime -> Episode.

Role-based access control (RBAC)
- **Admin**: Full CRUD on core content (TV shows, seasons, episodes, crew, actors, screentime management). Admin-only endpoints include creating, updating and deleting TV shows, seasons, episodes, crew and actors as well as editing/deleting screentime records.
- **User**: Can authenticate, view resources, and create personal resources where applicable (items and screentime creation is allowed for authenticated users). Users can update/delete their own items; Admin can update/delete any item.

Examples (Admin-only):
- `POST /api/tvshows` (create)
- `PUT /api/tvshows/:id` (update)
- `DELETE /api/tvshows/:id` (delete)
- `POST /api/seasons`, `PUT /api/seasons/:id`, `DELETE /api/seasons/:id`
- `POST /api/episodes`, `PUT /api/episodes/:id`, `DELETE /api/episodes/:id`
- `POST /api/actors`, `PUT /api/actors/:id`, `DELETE /api/actors/:id`
- `POST /api/crew`, `PUT /api/crew/:id`, `DELETE /api/crew/:id`
- `PUT /api/screentime/:id`, `DELETE /api/screentime/:id`

Fetching users Excel from Google Drive

Two modes are supported for importing the `techify_registered_users` Excel file from Google Drive.

1) Recommended: Service account (server-side, secure)
- Steps:
	1. Create a Google Cloud project and enable the Google Drive API.
	2. Create a Service Account and generate a JSON key file.
	3. Share your Google Sheet / Excel file with the service account's email address (or make it available to the service account). Alternatively, put the file in a Drive location accessible to the service account.
	4. In the server environment, set either `GOOGLE_SERVICE_ACCOUNT_KEY` (contents of the JSON key) or `GOOGLE_SERVICE_ACCOUNT_KEY_PATH` (absolute path to the JSON key file).
	5. Set the file ID in `USERSHEET_FILE_ID` environment variable. You can get the file ID from the Drive share link (the long id in the URL).
	6. Start the server. Admins can fetch the JSON-parsed sheet at the Admin-only endpoint:

```
GET /api/external/usersheet
Authorization: Bearer <admin-token>
```

2) Simpler: Public URL (not recommended for sensitive data)
- Make the file shareable publicly (Anyone with link) and obtain a direct download URL.
- Set `USERSHEET_PUBLIC_URL` to that direct file URL in the server environment.
- Admins can then call the same endpoint above to retrieve parsed JSON.

Notes:
- The server uses `xlsx` to parse the Excel contents and returns the first sheet as JSON.
- The endpoint is Admin-only and will return an error if configuration is missing.

