# TodoFlow Application - Mermaid Diagrams

## 1. High-Level Architecture

```mermaid
graph TB
    subgraph Browser["🌐 Browser Environment"]
        subgraph UI["User Interface Layer"]
            App[App.tsx<br/>Main Coordinator]
            TForm[TodoForm<br/>Add New Tasks]
            TItem[TodoItem<br/>Display Each Task]
            TFilter[TodoFilter<br/>Filter Controls]
            TStats[TodoStats<br/>Progress Display]
        end

        subgraph Logic["Business Logic Layer"]
            Hook[useTodos Hook<br/>🧠 The Brain<br/>All State & Logic]
        end

        subgraph Storage["Storage Layer"]
            Utils[storage.ts<br/>Save/Load Helpers]
            LS[(localStorage<br/>💾 Browser Database<br/>Key: todos-app-data)]
        end

        subgraph Types["Type Definitions"]
            TypeDef[types/index.ts<br/>Todo Interface<br/>FilterType]
        end
    end

    App --> Hook
    Hook --> TForm
    Hook --> TItem
    Hook --> TFilter
    Hook --> TStats
    Hook <--> Utils
    Utils <--> LS
    TypeDef -.defines.-> Hook
    TypeDef -.defines.-> TForm
    TypeDef -.defines.-> TItem

    style Hook fill:#4CAF50,color:#fff
    style LS fill:#2196F3,color:#fff
    style App fill:#9C27B0,color:#fff
```

## 2. Complete Data Flow - Adding a Todo

```mermaid
sequenceDiagram
    participant User
    participant TodoForm
    participant App
    participant useTodos
    participant storage
    participant localStorage
    participant Screen

    User->>TodoForm: 1. Types "Buy groceries"<br/>and clicks + button
    TodoForm->>TodoForm: 2. Validates input<br/>(text.trim())
    TodoForm->>App: 3. Calls onAddTodo("Buy groceries")
    App->>useTodos: 4. Executes addTodo("Buy groceries")

    useTodos->>useTodos: 5. Creates new todo object:<br/>{id: crypto.randomUUID(),<br/>text: "Buy groceries",<br/>completed: false,<br/>createdAt: new Date(),<br/>updatedAt: new Date()}

    useTodos->>useTodos: 6. Updates state:<br/>setTodos([newTodo, ...prev])

    Note over useTodos,localStorage: Automatic Save Triggered

    useTodos->>storage: 7. Auto-triggers useEffect<br/>saveTodos(todos)
    storage->>storage: 8. Converts to JSON:<br/>JSON.stringify(todos)
    storage->>localStorage: 9. Saves to browser:<br/>localStorage.setItem(<br/>'todos-app-data', json)
    localStorage-->>storage: 10. ✓ Saved successfully

    Note over useTodos,Screen: React Re-render Triggered

    useTodos-->>App: 11. Returns updated todos[]
    App-->>TodoForm: 12. TodoForm clears input
    App-->>Screen: 13. TodoItem renders new task
    App-->>Screen: 14. TodoStats updates (1 of 1)
    Screen-->>User: 15. ✓ User sees new task on screen
```

## 3. Complete Data Flow - Toggling Todo (Mark Complete)

```mermaid
sequenceDiagram
    participant User
    participant TodoItem
    participant App
    participant useTodos
    participant storage
    participant localStorage
    participant Screen

    User->>TodoItem: 1. Clicks checkbox circle
    TodoItem->>App: 2. Calls onToggle(todo.id)
    App->>useTodos: 3. Executes toggleTodo(id)

    useTodos->>useTodos: 4. Maps through todos array
    useTodos->>useTodos: 5. Finds matching todo by id
    useTodos->>useTodos: 6. Creates updated todo:<br/>{...todo,<br/>completed: !todo.completed,<br/>updatedAt: new Date()}

    useTodos->>useTodos: 7. Updates state with<br/>modified todos array

    Note over useTodos,localStorage: Automatic Save

    useTodos->>storage: 8. Auto-triggers save
    storage->>localStorage: 9. Overwrites data:<br/>localStorage.setItem(...)
    localStorage-->>storage: 10. ✓ Persisted

    Note over useTodos,Screen: Visual Update

    useTodos-->>App: 11. Returns updated todos
    App-->>TodoItem: 12. TodoItem re-renders
    TodoItem->>Screen: 13. Circle turns green ✓
    TodoItem->>Screen: 14. Text gets strikethrough
    App-->>Screen: 15. TodoStats updates progress
    Screen-->>User: 16. ✓ User sees completed task
```

## 4. Complete Data Flow - Filtering Todos

```mermaid
sequenceDiagram
    participant User
    participant TodoFilter
    participant App
    participant useTodos
    participant Screen

    User->>TodoFilter: 1. Clicks "Active" button
    TodoFilter->>App: 2. Calls onFilterChange("active")
    App->>useTodos: 3. Executes setFilter("active")

    useTodos->>useTodos: 4. Updates filter state
    useTodos->>useTodos: 5. Runs filter logic:<br/>todos.filter(todo => {<br/>  if (filter === "active")<br/>    return !todo.completed<br/>})

    Note over useTodos: Original todos in memory:<br/>✓ Buy groceries (done)<br/>○ Walk dog (active)<br/>○ Call mom (active)

    useTodos->>useTodos: 6. Creates filtered array:<br/>[Walk dog, Call mom]

    useTodos-->>App: 7. Returns filteredTodos
    App-->>Screen: 8. Re-renders TodoItem<br/>components (only 2 items)
    App-->>TodoFilter: 9. Updates button state<br/>("Active" highlighted)
    Screen-->>User: 10. ✓ Shows only active tasks

    Note over User,Screen: Note: No localStorage interaction<br/>Filter is UI state only
```

## 5. Application Startup - Loading Data

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant App
    participant useTodos
    participant storage
    participant localStorage
    participant Screen

    User->>Browser: 1. Opens TodoFlow app
    Browser->>App: 2. Loads App.tsx
    App->>useTodos: 3. Initializes useTodos hook

    Note over useTodos,localStorage: Load Saved Data

    useTodos->>useTodos: 4. useEffect runs on mount
    useTodos->>storage: 5. Calls loadTodos()
    storage->>localStorage: 6. Reads data:<br/>localStorage.getItem(<br/>'todos-app-data')

    alt Data exists
        localStorage-->>storage: 7a. Returns JSON string
        storage->>storage: 8a. Parses JSON:<br/>JSON.parse(stored)
        storage->>storage: 9a. Converts dates:<br/>new Date(todo.createdAt)
        storage-->>useTodos: 10a. Returns Todo[]
        useTodos->>useTodos: 11a. setTodos(loadedTodos)
    else No data found
        localStorage-->>storage: 7b. Returns null
        storage-->>useTodos: 8b. Returns empty array []
        useTodos->>useTodos: 9b. setTodos([])
    end

    useTodos-->>App: 12. Provides todos & functions
    App-->>Screen: 13. Renders all components
    Screen-->>User: 14. ✓ App ready with<br/>saved todos displayed
```

## 6. localStorage Database Schema

```mermaid
erDiagram
    LOCALSTORAGE ||--|| TODOS_DATA : stores

    LOCALSTORAGE {
        string key "todos-app-data"
        string value "JSON stringified array"
    }

    TODOS_DATA {
        array todos "Array of Todo objects"
    }

    TODO {
        string id "Unique UUID"
        string text "Task description"
        boolean completed "Completion status"
        datetime createdAt "Creation timestamp"
        datetime updatedAt "Last modified timestamp"
    }

    TODOS_DATA ||--o{ TODO : contains
```

## 7. Component Hierarchy & Data Flow

```mermaid
graph TD
    subgraph Browser["Browser Window"]
        subgraph Main["Main Application"]
            App["App.tsx<br/>Main Container"]
        end

        subgraph Brain["State Management"]
            Hook["useTodos Hook<br/>---<br/>State: todos, filter<br/>---<br/>Functions:<br/>addTodo, toggleTodo<br/>updateTodo, deleteTodo<br/>clearCompleted, toggleAll<br/>---<br/>Computed:<br/>filteredTodos, stats"]
        end

        subgraph Components["UI Components"]
            Form["TodoForm<br/>Input + Add Button"]
            Stats["TodoStats<br/>Progress Bar & Actions"]
            Filter["TodoFilter<br/>All - Active - Done"]
            Item1["TodoItem 1<br/>Task Display"]
            Item2["TodoItem 2<br/>Task Display"]
            Item3["TodoItem 3<br/>Task Display"]
        end

        subgraph Storage["Persistence"]
            Utils["storage.ts<br/>Save/Load Logic"]
            LS[("localStorage<br/>Browser DB<br/>todos-app-data")]
        end
    end

    App -->|uses| Hook
    Hook -->|data + callbacks| App

    App -->|addTodo fn| Form
    App -->|todos + fns| Item1
    App -->|todos + fns| Item2
    App -->|todos + fns| Item3
    App -->|filter + setFilter| Filter
    App -->|stats + fns| Stats

    Form -.->|user adds| Hook
    Item1 -.->|toggle/edit/delete| Hook
    Item2 -.->|toggle/edit/delete| Hook
    Item3 -.->|toggle/edit/delete| Hook
    Filter -.->|change filter| Hook
    Stats -.->|bulk actions| Hook

    Hook <-->|auto sync| Utils
    Utils <-->|read/write| LS

    style Hook fill:#4CAF50,color:#fff,stroke:#2E7D32,stroke-width:3px
    style LS fill:#2196F3,color:#fff,stroke:#1565C0,stroke-width:3px
    style App fill:#9C27B0,color:#fff,stroke:#6A1B9A,stroke-width:3px
```

## 8. State Management Flow

```mermaid
stateDiagram-v2
    [*] --> AppLoads

    AppLoads --> LoadFromStorage: useEffect on mount
    LoadFromStorage --> EmptyState: No data found
    LoadFromStorage --> PopulatedState: Data exists

    state PopulatedState {
        [*] --> DisplayingAll
        DisplayingAll --> DisplayingActive: Filter Active
        DisplayingActive --> DisplayingCompleted: Filter Completed
        DisplayingCompleted --> DisplayingAll: Filter All
        DisplayingActive --> DisplayingAll: Filter All
        DisplayingCompleted --> DisplayingActive: Filter Active
    }

    state EmptyState {
        [*] --> NoTodos
        NoTodos --> FirstTodo: User adds todo
    }

    FirstTodo --> PopulatedState

    PopulatedState --> AddTodo: Add new
    PopulatedState --> ToggleTodo: Mark complete
    PopulatedState --> UpdateTodo: Edit text
    PopulatedState --> DeleteTodo: Remove
    PopulatedState --> ClearCompleted: Bulk delete
    PopulatedState --> ToggleAll: Bulk toggle

    AddTodo --> SaveToStorage
    ToggleTodo --> SaveToStorage
    UpdateTodo --> SaveToStorage
    DeleteTodo --> SaveToStorage
    ClearCompleted --> SaveToStorage
    ToggleAll --> SaveToStorage

    SaveToStorage --> PopulatedState: Update UI
    PopulatedState --> EmptyState: Last todo removed
```

## 9. User Interaction Flow Map

```mermaid
flowchart TD
    Start([👤 User Opens App])
    Start --> Load{localStorage<br/>has data?}

    Load -->|Yes| ShowTodos[Display Saved Todos]
    Load -->|No| ShowEmpty[Show Empty State]

    ShowEmpty --> WaitAction[Wait for User Action]
    ShowTodos --> WaitAction

    WaitAction --> Action{What does<br/>user do?}

    Action -->|Types in form| AddFlow[Add Todo Flow]
    Action -->|Clicks checkbox| ToggleFlow[Toggle Todo Flow]
    Action -->|Clicks edit| EditFlow[Edit Todo Flow]
    Action -->|Clicks delete| DeleteFlow[Delete Todo Flow]
    Action -->|Clicks filter| FilterFlow[Filter Flow]
    Action -->|Clicks toggle all| ToggleAllFlow[Toggle All Flow]
    Action -->|Clicks clear done| ClearFlow[Clear Completed Flow]

    AddFlow --> Validate{Input<br/>valid?}
    Validate -->|No| WaitAction
    Validate -->|Yes| CreateTodo[Create Todo Object<br/>with UUID]
    CreateTodo --> UpdateState[Update State]

    ToggleFlow --> FindTodo[Find Todo by ID]
    FindTodo --> FlipStatus[Flip completed status]
    FlipStatus --> UpdateState

    EditFlow --> ShowInput[Show Inline Editor]
    ShowInput --> SaveEdit{User saves?}
    SaveEdit -->|Yes| UpdateText[Update todo.text]
    SaveEdit -->|No| CancelEdit[Revert Changes]
    CancelEdit --> WaitAction
    UpdateText --> UpdateState

    DeleteFlow --> RemoveTodo[Remove from Array]
    RemoveTodo --> UpdateState

    FilterFlow --> ChangeFilter[Set Filter State]
    ChangeFilter --> Rerender[Re-render Filtered List]
    Rerender --> WaitAction

    ToggleAllFlow --> CheckAll{All<br/>completed?}
    CheckAll -->|Yes| UncheckAll[Set all to incomplete]
    CheckAll -->|No| CheckAll2[Set all to complete]
    UncheckAll --> UpdateState
    CheckAll2 --> UpdateState

    ClearFlow --> FilterCompleted[Filter completed todos]
    FilterCompleted --> RemoveAll[Remove all completed]
    RemoveAll --> UpdateState

    UpdateState --> SaveLS[Save to localStorage]
    SaveLS --> UpdateUI[Update Screen]
    UpdateUI --> WaitAction

    style Start fill:#9C27B0,color:#fff
    style UpdateState fill:#4CAF50,color:#fff
    style SaveLS fill:#2196F3,color:#fff
    style WaitAction fill:#FF9800,color:#fff
```

## 10. localStorage Interaction Details

```mermaid
sequenceDiagram
    autonumber
    participant App as Application
    participant Hook as useTodos Hook
    participant Utils as storage.ts
    participant LS as localStorage
    participant Browser as Browser Memory

    Note over App,Browser: SAVE OPERATION

    App->>Hook: User action triggers state change
    Hook->>Hook: setTodos(newTodosArray)
    Hook->>Hook: useEffect detects change
    Hook->>Utils: saveTodos(todos)
    Utils->>Utils: JSON.stringify(todos)
    Utils->>LS: setItem('todos-app-data', json)
    LS->>Browser: Write to disk
    Browser-->>LS: Success
    LS-->>Utils: Saved
    Utils-->>Hook: Complete

    Note over App,Browser: LOAD OPERATION

    App->>Hook: Component mounts
    Hook->>Utils: loadTodos()
    Utils->>LS: getItem('todos-app-data')
    LS->>Browser: Read from disk
    Browser-->>LS: JSON string or null
    LS-->>Utils: Return data

    alt Data exists
        Utils->>Utils: JSON.parse(data)
        Utils->>Utils: Convert date strings to Date objects
        Utils-->>Hook: Return Todo[]
        Hook->>Hook: setTodos(loadedTodos)
    else No data
        Utils-->>Hook: Return []
        Hook->>Hook: setTodos([])
    end

    Note over App,Browser: ERROR HANDLING

    alt Save fails
        Utils->>LS: setItem throws error
        LS-->>Utils: Error
        Utils->>Utils: console.error('Failed to save')
        Utils-->>Hook: Silently fail
    else Load fails
        Utils->>LS: getItem throws error
        LS-->>Utils: Error
        Utils->>Utils: console.error('Failed to load')
        Utils-->>Hook: Return []
    end
```

## 11. Todo Object Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Created: User adds todo

    state Created {
        [*] --> GenerateID: crypto.randomUUID()
        GenerateID --> SetProperties
        state SetProperties {
            id: UUID
            text: User_Input
            completed: false
            createdAt: new_Date()
            updatedAt: new_Date()
        }
        SetProperties --> AddToArray
    }

    AddToArray --> Active: completed = false

    state Active {
        [*] --> Displayed
        Displayed --> BeingEdited: User clicks edit
        BeingEdited --> Displayed: Save/Cancel
    }

    Active --> Completed: User toggles checkbox
    Completed --> Active: User toggles again

    state Completed {
        [*] --> StrikethroughDisplay
        StrikethroughDisplay --> AwaitingClear
    }

    Active --> Deleted: User clicks delete
    Completed --> Deleted: User clicks delete<br/>OR<br/>Clear completed

    state Deleted {
        [*] --> RemovedFromArray
        RemovedFromArray --> UpdatedStorage
    }

    Deleted --> [*]

    note right of Created
        Every state change
        triggers:
        1. State update
        2. localStorage save
        3. UI re-render
    end note
```

## 12. Complete System Overview

```mermaid
graph TB
    subgraph External["User Interactions"]
        U1[Type text]
        U2[Click buttons]
        U3[Edit tasks]
        U4[Filter view]
    end

    subgraph Presentation["Presentation Layer"]
        TF[TodoForm Component]
        TI[TodoItem Component]
        TS[TodoStats Component]
        TFL[TodoFilter Component]
    end

    subgraph Container["Container Layer"]
        APP["App.tsx<br/>Orchestrator"]
    end

    subgraph State["State Management"]
        HOOK["useTodos Hook<br/>---<br/>Central State Manager"]
        S1["State: todos"]
        S2["State: filter"]
        F1[addTodo Function]
        F2[toggleTodo Function]
        F3[updateTodo Function]
        F4[deleteTodo Function]
        F5[clearCompleted Function]
        F6[toggleAll Function]
        F7[setFilter Function]
        C1[Computed: filteredTodos]
        C2[Computed: stats]
    end

    subgraph Data["Data Layer"]
        UTILS["storage.ts<br/>Persistence Utils"]
        TYPES["types/index.ts<br/>Type Definitions"]
    end

    subgraph Persistence["Persistence Layer"]
        LS[("localStorage<br/>Browser Database<br/>Key: todos-app-data<br/>Value: JSON string")]
    end

    U1 --> TF
    U2 --> TI
    U3 --> TS
    U4 --> TFL

    TF <--> APP
    TI <--> APP
    TS <--> APP
    TFL <--> APP

    APP <--> HOOK

    HOOK --> S1
    HOOK --> S2
    HOOK --> F1
    HOOK --> F2
    HOOK --> F3
    HOOK --> F4
    HOOK --> F5
    HOOK --> F6
    HOOK --> F7
    HOOK --> C1
    HOOK --> C2

    S1 --> F1
    S1 --> F2
    S1 --> C1
    S1 --> C2
    S2 --> C1

    F1 --> S1
    F2 --> S1
    F3 --> S1
    F4 --> S1
    F5 --> S1
    F6 --> S1
    F7 --> S2

    HOOK <--> UTILS
    UTILS <--> LS
    TYPES -.-> HOOK
    TYPES -.-> TF
    TYPES -.-> TI

    style HOOK fill:#4CAF50,color:#fff,stroke:#2E7D32,stroke-width:4px
    style LS fill:#2196F3,color:#fff,stroke:#1565C0,stroke-width:4px
    style APP fill:#9C27B0,color:#fff,stroke:#6A1B9A,stroke-width:4px
    style External fill:#FF9800,color:#fff
```

## Summary

These diagrams show:

1. **Architecture** - How all files are organized
2. **Add Todo Flow** - Complete sequence from user input to screen update
3. **Toggle Todo Flow** - How checking off items works
4. **Filter Flow** - How filtering doesn't affect storage
5. **Startup Flow** - How data loads from localStorage
6. **Database Schema** - What's stored in localStorage
7. **Component Hierarchy** - Who talks to whom
8. **State Management** - Different states of the app
9. **User Interaction Map** - All possible user actions
10. **localStorage Details** - Save/Load operations in detail
11. **Todo Lifecycle** - Birth to death of a todo item
12. **Complete System** - Everything together

Key Insights:
- **useTodos Hook** is the single source of truth (the brain)
- **localStorage** is the only database (automatic sync)
- **Components** are "dumb" - they just display and trigger actions
- **Every state change** automatically saves to localStorage
- **Filter is UI-only** - doesn't affect stored data
