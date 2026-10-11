# ClimateRisk frontend

The six-screen React interface for [ClimateRisk Observatory](../README.md). See the root README for dark-mode screenshots, architecture, Docker setup, model requirements, API documentation, and troubleshooting.

## Run locally

Requires Node.js 24. From this directory:

```sh
npm ci
cp .env.example .env.local
npm run dev
```

Open **http://localhost:3000**. The default `VITE_APP_MODE=demo` uses labelled sample records and makes no backend/API/socket connections. Image selection previews locally; analysis is disabled in demo mode.

For the local backend, edit `.env.local` and restart Vite:

```dotenv
VITE_APP_MODE=connected
VITE_API_URL=http://localhost:8000/api/v1
VITE_SOCKET_URL=http://localhost:8000
```

Only public configuration belongs in `VITE_` variables. Never place Gemini, database, or other credentials in this file.

## UI and state

- React 18, TypeScript, Vite 7, Tailwind 4, Leaflet, Recharts, and Framer Motion.
- Zustand persists theme, compact layout, refresh, and notification preferences.
- The header theme toggle works on the home page and every route, including mobile. Settings uses the same preference.
- TanStack Query handles reports and metrics; polling remains sufficient when Socket.IO disconnects.
- Connected failures stay failures. Demo records never become fallback results.
- Local image replacement/reset aborts stale requests; report details and PDFs preserve saved narrative and provenance.

## Check and build

```sh
npm run lint
npm test
npm run build
```

The build writes `dist`. `vercel.json` explicitly selects demo mode and rewrites only the application routes. See the [root deployment guide](../README.md#frontend-deployment-on-vercel) for project settings.
