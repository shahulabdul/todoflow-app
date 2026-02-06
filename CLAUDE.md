# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start Vite dev server
- `npm run build` — Production build (vite build)
- `npm run lint` — Run ESLint
- `npm run preview` — Preview production build

## Architecture

This is a **TodoFlow** app — a single-page React + TypeScript todo manager styled with Tailwind CSS, built with Vite.

### Key layers

- **`src/types/index.ts`** — `Todo` interface (id, text, completed, createdAt, updatedAt) and `FilterType` union
- **`src/hooks/useTodos.ts`** — All todo state and logic lives in a single custom hook (`useTodos`). It manages CRUD, filtering, toggle-all, clear-completed, and computed stats. Todos are persisted to localStorage on every change.
- **`src/utils/storage.ts`** — localStorage read/write helpers (key: `todos-app-data`). Handles Date serialization/deserialization.
- **`src/components/`** — Presentational components (`TodoForm`, `TodoItem`, `TodoFilter`, `TodoStats`). They receive data and callbacks from `useTodos` via `App.tsx`.
- **`src/App.tsx`** — Composes everything; the only consumer of `useTodos`.

## Rules

- **Always ask for explicit user permission before deleting any file.**

### Design conventions

- Use **lucide-react** for icons — do not add other icon/UI libraries unless necessary.
- Styling is Tailwind utility classes only (no custom CSS beyond Tailwind directives in `index.css`).
- `lucide-react` is excluded from Vite's dependency optimization (`vite.config.ts`).
