# TodoFlow MongoDB Migration Plan

## Context

The TodoFlow application currently uses browser localStorage to persist todos. While this works for a single-user, single-device scenario, it has limitations: data is browser-specific, no server-side validation, and no multi-device sync capability. This migration moves to a proper database architecture using MongoDB (which you already have installed locally) with a Node.js backend API. This provides a foundation for future enhancements like multi-user support, real-time sync, and deployment to production.

**Goal**: Migrate from localStorage to MongoDB while maintaining all existing functionality and preparing for future scalability.

## Architecture Overview

**Current**: React SPA → localStorage
**Target**: React SPA → Express API → MongoDB

### Tech Stack
- **Backend**: Node.js + Express + Mongoose (MongoDB ODM)
- **Database**: MongoDB (local instance at mongodb://localhost:27017)
- **Frontend**: React + TypeScript (existing Vite setup, minimal changes)
- **Project Structure**: Monorepo (client/, server/, shared/)

## Task Summary (21 Tasks)

Each task scoped to ~1-5% of context tokens (~2,000-10,000 tokens):

**Phase 1: Project Restructuring** (4 tasks)
- 1.1: Create monorepo structure
- 1.2: Move frontend to client directory
- 1.3: Create shared types
- 1.4: Initialize backend package

**Phase 2: Backend API** (7 tasks)
- 2.1: Create MongoDB connection
- 2.2: Create Mongoose Todo model
- 2.3: Create API controllers (CRUD)
- 2.4: Create API controllers (bulk ops)
- 2.5: Create Express routes
- 2.6: Create middleware (error & validation)
- 2.7: Create Express server entry point

**Phase 3: Frontend Migration** (7 tasks)
- 3.1: Update shared types for MongoDB
- 3.2: Create API client (fetch methods)
- 3.3: Create API client (delete & bulk methods)
- 3.4: Refactor useTodos (async state & fetch)
- 3.5: Refactor useTodos (convert CRUD to async)
- 3.6: Update components to use _id
- 3.7: Configure Vite proxy & delete storage

**Phase 4: Development Workflow** (1 task)
- 4.1: Setup concurrent development

**Phase 5: Testing** (4 tasks)
- 5.1: Test basic CRUD operations
- 5.2: Test filtering & bulk operations
- 5.3: Test error handling
- 5.4: Verify MongoDB data

---

## Implementation Plan - Granular Tasks

### Phase 1: Project Restructuring & Backend Setup

#### Task 1.1: Create Monorepo Structure (Scope: ~3-5% context)
**Files to create/modify**: 4 files, 0 reads needed

1. Create three directories: `client/`, `server/`, `shared/`
2. Create root `package.json` with monorepo scripts
3. Create `.env` file with MongoDB connection string
4. Document the new structure

**Deliverables**:
- Empty directory structure
- Root package.json with concurrently scripts
- .env with MONGODB_URI and PORT

**Verification**: `ls` shows client/, server/, shared/ directories

---

#### Task 1.2: Move Frontend to Client Directory (Scope: ~2-3% context)
**Files to modify**: Moving existing files only

1. Move `src/` → `client/src/`
2. Move `vite.config.ts` → `client/`
3. Move `index.html` → `client/`
4. Move `public/` → `client/`
5. Move `package.json` → `client/package.json`
6. Move `tsconfig.json` → `client/`

**Verification**: `npm run dev` from client/ directory still works

---

#### Task 1.3: Create Shared Types (Scope: ~1-2% context)
**Files to create**: 1 file

1. Copy `client/src/types/index.ts` → `shared/types.ts`
2. Add `Stats` interface to shared types
3. Update Todo interface documentation

**Deliverables**: `shared/types.ts` with Todo, FilterType, Stats

---

#### Task 1.4: Initialize Backend Package (Scope: ~2-3% context)
**Files to create**: 2 files

1. Create `server/package.json` with dependencies:
   - express, mongoose, cors, dotenv, express-validator
   - Dev: @types/*, ts-node-dev, typescript
2. Create `server/tsconfig.json` for TypeScript configuration
3. Run `npm install` in server directory

**Deliverables**: Backend package.json and tsconfig.json ready for development

### Phase 2: Backend API Implementation

#### Task 2.1: Create MongoDB Connection (Scope: ~2-3% context)
**Files to create**: 1 file

1. Create `server/src/config/database.ts`
2. Implement `connectDB()` function with error handling
3. Use environment variable for MongoDB URI
4. Add connection success/failure logging

**Deliverables**: Database connection utility
**Verification**: Server logs "MongoDB Connected" on startup

---

#### Task 2.2: Create Mongoose Todo Model (Scope: ~2-3% context)
**Files to create**: 1 file (`server/src/models/Todo.ts`)

1. Define ITodo interface extending Document
2. Create TodoSchema with validation:
   - text: required, trimmed, 1-500 chars
   - completed: boolean, default false
3. Enable timestamps (auto createdAt/updatedAt)
4. Export Todo model

**Key Decision**: Use `_id` (MongoDB native) instead of UUID

**Deliverables**: `server/src/models/Todo.ts` with schema and model

#### Task 2.3: Create API Controllers - CRUD Operations (Scope: ~4-5% context)
**Files to create**: 1 file (`server/src/controllers/todoController.ts`)

Implement 4 basic CRUD controllers:
1. `getTodos(req, res, next)` - Fetch all with optional filter query param
2. `createTodo(req, res, next)` - Create from req.body.text
3. `updateTodo(req, res, next)` - Update by ID with partial data
4. `deleteTodo(req, res, next)` - Delete by ID

**Response Format**: `{ success: true, data: any }` or `{ success: false, error: any }`

**Deliverables**: 4 controller functions with error handling

---

#### Task 2.4: Create API Controllers - Bulk Operations (Scope: ~2-3% context)
**Files to modify**: 1 file (`server/src/controllers/todoController.ts`)

Add 2 bulk operation controllers:
1. `toggleAll(req, res, next)` - Update all todos' completed status
2. `clearCompleted(req, res, next)` - Delete all completed todos

**Deliverables**: 2 additional controller functions

---

#### Task 2.5: Create Express Routes (Scope: ~3-4% context)
**Files to create**: 1 file (`server/src/routes/todoRoutes.ts`)

1. Create Express router
2. Define 6 routes with validation middleware:
   - GET / → getTodos
   - POST / → createTodo (validate text)
   - PATCH /:id → updateTodo (validate text/completed)
   - DELETE /:id → deleteTodo
   - POST /toggle-all → toggleAll (validate completed)
   - DELETE /completed → clearCompleted
3. Export router

**Deliverables**: Complete routing configuration

#### Task 2.6: Create Middleware (Scope: ~2-3% context)
**Files to create**: 2 files

1. Create `server/src/middleware/errorHandler.ts`:
   - Central error handling middleware
   - Format errors as `{ success: false, error: { message, code, details } }`
   - Log errors to console

2. Create `server/src/middleware/validate.ts`:
   - Validation middleware using express-validator
   - Return 400 with validation errors if invalid

**Deliverables**: Error handling and validation middleware

---

#### Task 2.7: Create Express Server Entry Point (Scope: ~3-4% context)
**Files to create**: 1 file (`server/src/server.ts`)

1. Import dependencies (express, cors, dotenv, etc.)
2. Configure middleware: CORS (allow localhost:5173), JSON parsing
3. Mount routes at `/api/todos`
4. Add error handler middleware
5. Connect to MongoDB and start server on PORT from .env

**Deliverables**: Complete Express server setup
**Verification**: `npm run dev` starts server on port 5000

### Phase 3: Frontend Migration

#### Task 3.1: Update Shared Types for MongoDB (Scope: ~1-2% context)
**Files to modify**: 1 file (`shared/types.ts`)

1. Change `id: string` → `_id: string` in Todo interface
2. Add Stats interface (total, active, completed)
3. Add JSDoc comments for clarity

**Key Change**: MongoDB uses `_id` instead of `id`

**Deliverables**: Updated shared/types.ts

#### Task 3.2: Create API Client - Fetch Methods (Scope: ~3-4% context)
**Files to create**: 1 file (`client/src/api/todosApi.ts`)

Create todosApi object with 3 fetch methods:
1. `fetchTodos(filter?)` - GET /api/todos with optional filter
2. `createTodo(text)` - POST /api/todos
3. `updateTodo(id, updates)` - PATCH /api/todos/:id

**Deliverables**: API client with fetch methods
**Pattern**: All methods use fetch API, throw on error, return parsed JSON

---

#### Task 3.3: Create API Client - Delete & Bulk Methods (Scope: ~2-3% context)
**Files to modify**: 1 file (`client/src/api/todosApi.ts`)

Add 3 more methods:
1. `deleteTodo(id)` - DELETE /api/todos/:id
2. `toggleAll(completed)` - POST /api/todos/toggle-all
3. `clearCompleted()` - DELETE /api/todos/completed

**Deliverables**: Complete API client with all 6 methods

#### Task 3.4: Refactor useTodos - Add Async State & Initial Fetch (Scope: ~4-5% context)
**Files to modify**: 1 file (`client/src/hooks/useTodos.ts`)

1. Add new state: `loading` (boolean), `error` (string | null), `stats` (Stats)
2. Remove: `saveTodos()` useEffect, localStorage imports
3. Replace initial load with async fetch:
   - useEffect on mount
   - Call todosApi.fetchTodos()
   - Set todos, stats, loading, error
4. Update return value to include loading, error

**Key Change**: From sync localStorage to async API fetch

**Deliverables**: Updated hook with async initial fetch

---

#### Task 3.5: Refactor useTodos - Convert CRUD to Async (Scope: ~4-5% context)
**Files to modify**: 1 file (`client/src/hooks/useTodos.ts`)

Convert 6 operations to async with error handling:
1. `addTodo` - Call todosApi.createTodo, update local state
2. `updateTodo` - Call todosApi.updateTodo
3. `deleteTodo` - Call todosApi.deleteTodo
4. `toggleTodo` - Call todosApi.updateTodo with completed toggle
5. `toggleAll` - Call todosApi.toggleAll
6. `clearCompleted` - Call todosApi.clearCompleted

**Pattern**: Try-catch with optimistic updates where applicable

**Deliverables**: All CRUD operations async

#### Task 3.6: Update Components to Use _id (Scope: ~2-3% context)
**Files to modify**: 2 files

1. `client/src/components/TodoItem.tsx`:
   - Change all `todo.id` → `todo._id`
   - Update prop types if needed

2. `client/src/App.tsx`:
   - Add loading/error UI handling
   - Update any `todo.id` references to `todo._id`

**Deliverables**: Components using MongoDB _id field

---

#### Task 3.7: Configure Vite Proxy & Delete Storage Utils (Scope: ~1-2% context)
**Files to modify**: 1 file, 1 file to delete

1. Update `client/vite.config.ts`:
   - Add server.proxy configuration for /api → http://localhost:5000

2. Delete `client/src/utils/storage.ts` (no longer needed)

**Deliverables**: Vite proxy configured, old storage removed

### Phase 4: Development Workflow

#### Task 4.1: Setup Concurrent Development (Scope: ~2-3% context)
**Files to modify**: 1 file (root `package.json`)

1. Verify root package.json has concurrently scripts
2. Install root dependencies: `npm install`
3. Install client dependencies: `cd client && npm install`
4. Install server dependencies: `cd server && npm install`
5. Test concurrent startup: `npm run dev`

**Deliverables**: Both servers running concurrently
**Verification**:
- Frontend on http://localhost:5173
- Backend on http://localhost:5000
- MongoDB connected

### Phase 5: Testing & Verification

#### Task 5.1: Test Basic CRUD Operations (Scope: ~3-4% context)
**Manual testing checklist**:

1. Start MongoDB (mongod or MongoDB Compass)
2. Start servers: `npm run dev`
3. Test in browser:
   - Create new todo → verify appears
   - Toggle completion → verify updates
   - Edit todo text → verify saves
   - Delete todo → verify removes
   - Refresh page → verify data persists

**Verification**: All basic operations work

---

#### Task 5.2: Test Filtering & Bulk Operations (Scope: ~2-3% context)
**Manual testing checklist**:

1. Filter todos:
   - Click "All" → shows all todos
   - Click "Active" → shows only incomplete
   - Click "Completed" → shows only complete
2. Bulk operations:
   - Toggle all → all become same state
   - Clear completed → removes all completed
3. Stats verification:
   - Create/delete todos → stats update correctly

**Verification**: Filtering and bulk ops work

---

#### Task 5.3: Test Error Handling (Scope: ~2-3% context)
**Manual testing checklist**:

1. Stop backend server
2. Try to create todo → error message shown
3. Restart backend → app recovers
4. Test with invalid data (empty text)
5. Check browser console for errors

**Verification**: Errors handled gracefully

---

#### Task 5.4: Verify MongoDB Data (Scope: ~1-2% context)
**Database verification**:

```bash
# Connect to MongoDB
mongosh

# Switch to todoflow database
use todoflow

# View all todos
db.todos.find().pretty()

# Check completed count
db.todos.countDocuments({ completed: true })
```

**Verification**: Data correctly stored in MongoDB

## Critical Files Reference

**Backend (9 new files)**:
- `server/src/models/Todo.ts` - Mongoose schema
- `server/src/controllers/todoController.ts` - Business logic (6 controllers)
- `server/src/routes/todoRoutes.ts` - Express routes
- `server/src/middleware/errorHandler.ts` - Error handling
- `server/src/middleware/validate.ts` - Validation
- `server/src/config/database.ts` - MongoDB connection
- `server/src/server.ts` - Express entry point
- `server/package.json`, `server/tsconfig.json` - Config

**Frontend (2 new, 4 modified, 1 deleted)**:
- NEW: `client/src/api/todosApi.ts` - API client (6 methods)
- NEW: `shared/types.ts` - Shared types (Todo with `_id`)
- MODIFY: `client/src/hooks/useTodos.ts` - Async operations
- MODIFY: `client/src/App.tsx` - Loading/error UI
- MODIFY: `client/src/components/TodoItem.tsx` - Use `_id`
- MODIFY: `client/vite.config.ts` - Add proxy
- DELETE: `client/src/utils/storage.ts` - Replaced by API client

**Root**:
- `package.json` - Monorepo scripts with concurrently
- `.env` - MongoDB URI and PORT

## Quick Reference

### Starting the Application
```bash
# One-time setup
npm run install:all

# Start both servers
npm run dev
```
- Frontend: http://localhost:5173
- Backend: http://localhost:5000
- MongoDB: mongodb://localhost:27017/todoflow

### Verifying MongoDB Data
```bash
mongosh
use todoflow
db.todos.find().pretty()
```

### Testing API Directly (Optional)
```bash
curl http://localhost:5000/api/todos
```

## Key Decisions & Notes

**Architecture Choices**:
- **Monorepo**: Simpler dependency management for PoC
- **Mongoose**: Schema validation, auto-timestamps, better TypeScript
- **MongoDB _id**: Use native `_id` instead of UUID for simplicity
- **Optimistic Updates**: Instant UI feedback with rollback on error
- **Backend Stats**: Single source of truth for counts
- **Backend Filtering**: Query params instead of client-side filtering

**Estimated Timeline**: 1-2 days (21 small, focused tasks)
