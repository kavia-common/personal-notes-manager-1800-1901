# Nuxt Notes Manager Frontend

A minimalistic, responsive web-based notes manager built with Nuxt 3.

## Features

- CRUD operations: Create, Read, Update, and Delete notes
- Responsive single-page layout
- Sidebar for navigation, main area for editing/reading notes
- Light theme, minimal UI, custom color palette:
  - Primary: `#4a90e2`
  - Secondary: `#22313f`
  - Accent: `#f5a623`
- Data persistence using browser localStorage (no backend required)

## Run Locally

Install dependencies:

```bash
npm install
# or pnpm install / yarn install / bun install
```

Start the development server (http://localhost:3000):

```bash
npm run dev
```

## Build for Production

```bash
npm run build
npm run preview
```

## Environment Variables

This app does not currently require remote API/configuration, but if you add API integration, use the `.env` file for secrets/config.

## Usage

- "New" (`+` button): Create a new note.
- "Save": Save current edits.
- "🗑": Delete selected note.
- All changes are saved in your browser.

Check out the [Nuxt documentation](https://nuxt.com/docs/getting-started/introduction) for framework details.
