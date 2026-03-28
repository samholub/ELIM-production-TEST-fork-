# ELIM Production

A Progressive Web App (PWA) for managing weekly AV equipment setup and teardown for a church volunteer team.

## Architecture

- **Type:** Single-file static HTML app (zero-build architecture)
- **Frontend:** React 18 loaded via CDN, JSX transpiled in-browser by Babel Standalone
- **Database:** Firebase Realtime Database (with localStorage fallback for offline support)
- **Styling:** Inline JavaScript styles + base CSS in the HTML file

## Project Layout

- `index.html` — The entire application (HTML, CSS, React components, Firebase config)
- `index` — Original file from GitHub (copy of index.html)
- `PROJECT-BRIEF.md` — Project purpose, technical constraints, and roadmap
- `README.md` — Basic project identification
- `.cursorrules` — Architecture constraints for AI-assisted development

## Running the App

The app is served using the `serve` npm package:

```bash
npm start
# Serves on port 5000
```

## Key Features

- Real-time collaboration via Firebase Realtime Database
- Offline-first with localStorage + in-memory fallback
- Role-based task views for ~16 church volunteers
- PWA support (Add to Home Screen)
- Mobile-first design for on-site AV setup

## Deployment

Configured as a static site deployment (no build step needed).
