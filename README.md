# Interview AI

An AI-powered interview preparation platform. Upload your résumé, describe yourself,
paste a job description, and get back a tailored interview report: a match score,
model technical and behavioral questions (with the interviewer's intent and how to
answer), a prioritized list of skill gaps, and a day-by-day preparation plan. You can
also generate an ATS-friendly résumé PDF rewritten for the target role.

The report is produced by Google Gemini with a strict JSON schema, persisted to
MongoDB, and rendered in a React single-page app.

---

## Features

- **Auth** — register / login / logout with JWT stored in an HTTP-only cookie; a
  token blacklist invalidates cookies on logout.
- **Interview report generation** — `résumé PDF + self description + job description`
  → structured report:
  - `matchScore` (0–100)
  - `technicalQuestions` / `behavioralQuestions` — each with `question`, `intention`, `answer`
  - `skillGaps` — each with a `low` / `medium` / `high` severity
  - `preparationPlan` — day-wise `focus` + `tasks`
- **Report history** — list every report you've generated, open any one by id.
- **Résumé PDF builder** — generates a tailored, ATS-friendly résumé as HTML and
  renders it to PDF with Puppeteer.

---

## Tech stack

| Layer     | Stack |
|-----------|-------|
| Frontend  | React 19, Vite 7, React Router 7, Axios, Sass |
| Backend   | Node.js, Express 5, Mongoose 9 (MongoDB) |
| Auth      | `jsonwebtoken`, `bcryptjs`, `cookie-parser` |
| AI        | `@google/genai` (Gemini), `zod` + `zod-to-json-schema` for structured output |
| Files     | `multer` (in-memory upload), `pdf-parse` (résumé text extraction), `puppeteer` (HTML → PDF) |

---

## Project structure

```
.
├── Backend/
│   ├── server.js                 # entry point — connects DB, listens on :3000
│   └── src/
│       ├── app.js                # express app, CORS, routes
│       ├── config/database.js    # mongoose connection
│       ├── controllers/          # auth + interview request handlers
│       ├── middlewares/          # auth (JWT cookie), file upload (multer)
│       ├── models/               # user, interviewReport, blacklist
│       ├── routes/               # /api/auth, /api/interview
│       └── services/ai.service.js# Gemini calls + PDF generation
└── Frontend/
    └── src/
        ├── app.routes.jsx        # route table
        ├── features/
        │   ├── auth/             # context, hooks, pages, api
        │   └── interview/        # context, hooks, pages, api
        └── main.jsx
```

---

## Getting started

### Prerequisites

- Node.js 18+
- A MongoDB database (local or Atlas)
- A Google Gemini API key — https://aistudio.google.com/apikey

### 1. Backend

```bash
cd Backend
npm install
```

Create `Backend/.env`:

```env
MONGO_URI=mongodb://localhost:27017/interview-ai
JWT_SECRET=replace-with-a-long-random-string
GOOGLE_GENAI_API_KEY=your-gemini-api-key
```

Run it:

```bash
npm run dev        # nodemon server.js → http://localhost:3000
```

### 2. Frontend

```bash
cd Frontend
npm install
npm run dev        # vite → http://localhost:5173
```

The frontend expects the API at `http://localhost:3000` and the backend allows CORS
from `http://localhost:5173`. If you change either port, update `Backend/src/app.js`
(CORS `origin`) and the `baseURL` in `Frontend/src/features/*/services/*.api.js`.

---

## API reference

Base URL: `http://localhost:3000`. Authenticated routes require the `token` cookie
set at login.

### Auth — `/api/auth`

| Method | Path        | Access  | Body | Description |
|--------|-------------|---------|------|-------------|
| POST   | `/register` | public  | `{ username, email, password }` | Create account, set auth cookie |
| POST   | `/login`    | public  | `{ email, password }` | Log in, set auth cookie |
| GET    | `/logout`   | public  | — | Clear cookie, blacklist token |
| GET    | `/get-me`   | private | — | Current user details |

### Interview — `/api/interview`

| Method | Path                          | Access  | Body | Description |
|--------|-------------------------------|---------|------|-------------|
| POST   | `/`                           | private | `multipart/form-data`: `resume` (PDF ≤ 3 MB), `selfDescription`, `jobDescription` | Generate + store an interview report |
| GET    | `/`                           | private | — | List the current user's reports (summary fields) |
| GET    | `/report/:interviewId`        | private | — | Full report by id |
| POST   | `/resume/pdf/:interviewReportId` | private | — | Generate a tailored résumé PDF (`application/pdf`) |

---

## Notes & limitations

- The Gemini model id is set in `Backend/src/services/ai.service.js`
  (`gemini-3-flash-preview`) — change it there if needed.
- The auth cookie is not currently flagged `httpOnly` / `secure` / `sameSite` in
  code; harden this before any non-local deployment.
- There is no root-level `package.json`; the backend and frontend are installed and
  run independently.
- `puppeteer` downloads a bundled Chromium on install — the first `npm install` in
  `Backend/` may take a while.

---

## License

ISC (see `Backend/package.json`).
