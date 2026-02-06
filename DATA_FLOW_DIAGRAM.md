# TodoFlow Data Flow Diagrams

## 1. Overall Application Structure

```
┌─────────────────────────────────────────────────────────────┐
│                         Browser                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    App.tsx                            │  │
│  │              (Main Coordinator)                       │  │
│  │                                                       │  │
│  │  ┌─────────────────────────────────────────────┐    │  │
│  │  │         useTodos Hook                       │    │  │
│  │  │    (The Brain - All Logic Lives Here)      │    │  │
│  │  │                                             │    │  │
│  │  │  • Stores all todos in memory              │    │  │
│  │  │  • Handles add/edit/delete/toggle          │    │  │
│  │  │  • Filters todos (all/active/completed)    │    │  │
│  │  │  • Calculates statistics                    │    │  │
│  │  │  • Syncs with localStorage                 │    │  │
│  │  └─────────────────────────────────────────────┘    │  │
│  │                       │                              │  │
│  │                       ↓                              │  │
│  │         Data flows to components:                    │  │
│  │                                                       │  │
│  │  ┌──────────────┐  ┌──────────────┐                │  │
│  │  │  TodoForm    │  │  TodoStats   │                │  │
│  │  │ (Add tasks) │  │  (Progress)  │                │  │
│  │  └──────────────┘  └──────────────┘                │  │
│  │                                                       │  │
│  │  ┌──────────────┐  ┌──────────────┐                │  │
│  │  │ TodoFilter   │  │  TodoItem    │                │  │
│  │  │ (All/Active) │  │ (Each task)  │                │  │
│  │  └──────────────┘  └──────────────┘                │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              localStorage                             │  │
│  │         (Browser's Filing Cabinet)                    │  │
│  │    Saves: todos-app-data (all your tasks)            │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 2. Adding a New Todo - Step by Step

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: User Types and Clicks Add                          │
└─────────────────────────────────────────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │      TodoForm           │
              │  "Buy groceries" [+]    │
              └─────────────────────────┘
                           │
                           ↓ User clicks +
              ┌─────────────────────────┐
              │  Calls: onAddTodo()     │
              └─────────────────────────┘
                           │
┌──────────────────────────┴────────────────────────────────┐
│ STEP 2: Data Goes to the Brain                            │
└────────────────────────────────────────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   useTodos Hook         │
              │   addTodo() function    │
              └─────────────────────────┘
                           │
                           ↓ Creates new todo object
              ┌─────────────────────────┐
              │  {                      │
              │    id: "abc-123"        │
              │    text: "Buy groceries"│
              │    completed: false     │
              │    createdAt: [now]     │
              │    updatedAt: [now]     │
              │  }                      │
              └─────────────────────────┘
                           │
                           ↓ Adds to list
              ┌─────────────────────────┐
              │  todos = [newTodo,      │
              │           ...oldTodos]  │
              └─────────────────────────┘
                           │
┌──────────────────────────┴────────────────────────────────┐
│ STEP 3: Automatically Saves                               │
└────────────────────────────────────────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   storage.ts            │
              │   saveTodos()           │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   localStorage          │
              │   (Saved to browser!)   │
              └─────────────────────────┘
                           │
┌──────────────────────────┴────────────────────────────────┐
│ STEP 4: Screen Updates                                    │
└────────────────────────────────────────────────────────────┘
                           │
                           ↓ React re-renders
              ┌─────────────────────────┐
              │  TodoItem components    │
              │  show updated list      │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │  TodoStats updates      │
              │  (1 of 1 tasks)         │
              └─────────────────────────┘
```

## 3. Checking Off a Todo

```
User clicks the circle → TodoItem → onToggle(id)
                                        │
                                        ↓
                            ┌───────────────────────┐
                            │   useTodos Hook       │
                            │   toggleTodo(id)      │
                            └───────────────────────┘
                                        │
                     ┌──────────────────┴──────────────────┐
                     ↓                                     ↓
        Find todo with matching id              Update that todo
                     │                                     │
                     ↓                                     ↓
        { id: "abc-123", ...}              { completed: true,
                                             updatedAt: [now] }
                                                          │
                                                          ↓
                                            ┌─────────────────────┐
                                            │  Save to localStorage│
                                            └─────────────────────┘
                                                          │
                                                          ↓
                                            ┌─────────────────────┐
                                            │  Screen updates:     │
                                            │  • Circle turns green│
                                            │  • Text crossed out  │
                                            │  • Stats update      │
                                            └─────────────────────┘
```

## 4. Filtering Todos

```
┌────────────────────────────────────────────────────────────┐
│  User clicks "Active" filter button                        │
└────────────────────────────────────────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   TodoFilter            │
              │   onFilterChange()      │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   useTodos Hook         │
              │   setFilter("active")   │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────────────────────┐
              │  Filter logic runs:                     │
              │                                         │
              │  All todos in memory:                   │
              │  ✓ Buy groceries (completed)            │
              │  ○ Walk dog (not completed)             │
              │  ○ Call mom (not completed)             │
              │                                         │
              │  Filter: "active" → only incomplete     │
              │                                         │
              │  Result shown on screen:                │
              │  ○ Walk dog                             │
              │  ○ Call mom                             │
              └─────────────────────────────────────────┘
```

## 5. Data Persistence (Saving & Loading)

```
┌────────────────────────────────────────────────────────────┐
│              WHEN APP FIRST OPENS                          │
└────────────────────────────────────────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   App.tsx loads         │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   useTodos Hook         │
              │   (runs on startup)     │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   storage.ts            │
              │   loadTodos()           │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   localStorage          │
              │   Read: "todos-app-data"│
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   Parse saved data      │
              │   Convert dates         │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   Display your old todos│
              │   on screen!            │
              └─────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│         WHENEVER TODOS CHANGE (ANY ACTION)                 │
└────────────────────────────────────────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │  useTodos detects change│
              │  (automatic)            │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   storage.ts            │
              │   saveTodos()           │
              └─────────────────────────┘
                           │
                           ↓
              ┌─────────────────────────┐
              │   localStorage          │
              │   Write: "todos-app-data"│
              └─────────────────────────┘
```

## 6. Component Communication Map

```
                    ┌─────────────────┐
                    │    App.tsx      │
                    │  (Coordinator)  │
                    └────────┬────────┘
                             │
                             │ Gets everything from:
                             ↓
                    ┌─────────────────┐
                    │  useTodos Hook  │
                    │   (The Brain)   │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ↓                    ↓                    ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   todos[]     │   │  functions    │   │    stats      │
│   filter      │   │  - addTodo()  │   │  - total: 5   │
│               │   │  - toggleTodo()│   │  - active: 2  │
│               │   │  - deleteTodo()│   │  - done: 3    │
└───────────────┘   └───────────────┘   └───────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                             │ Passed down to:
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ↓                    ↓                    ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   TodoForm    │   │  TodoFilter   │   │  TodoStats    │
│ receives:     │   │ receives:     │   │ receives:     │
│ - addTodo()   │   │ - filter      │   │ - stats       │
│               │   │ - setFilter() │   │ - clearDone() │
└───────────────┘   └───────────────┘   └───────────────┘
                             │
                             ↓
                    ┌───────────────┐
                    │   TodoItem    │
                    │ receives:     │
                    │ - todo data   │
                    │ - toggle()    │
                    │ - update()    │
                    │ - delete()    │
                    └───────────────┘
```

## 7. Simple Summary

```
┌──────────────────────────────────────────────────────────┐
│                    Think of it like:                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  useTodos Hook = Your brain (remembers everything,       │
│                  makes all decisions)                     │
│                                                           │
│  Components = Your hands (show things, react to clicks)  │
│                                                           │
│  localStorage = Your notebook (writes things down so     │
│                 you remember later)                       │
│                                                           │
│  App.tsx = Your body (connects brain to hands)           │
│                                                           │
└──────────────────────────────────────────────────────────┘

Flow: User clicks → Component tells Brain → Brain updates
      → Brain writes to Notebook → Brain tells Hands to update screen
```
