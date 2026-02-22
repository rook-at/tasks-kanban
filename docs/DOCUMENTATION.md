# Tasks Kanban - Complete Documentation

This document contains the full reference documentation for Tasks Kanban.

---

## Table of Contents

- [Board Types](#board-types)
- [Templates](#templates)
- [Filtering](#filtering)
- [Swimlanes](#swimlanes)
- [Column Rules & Actions](#column-rules--actions)
- [Archive](#archive)
- [Keyboard Navigation](#keyboard-navigation)
- [Bulk Operations](#bulk-operations)
- [Webhooks](#webhooks)
- [Context Menu & Quick Actions](#context-menu--quick-actions)
- [Board Options](#board-options)
- [Card Sorting & Colors](#card-sorting--colors)
- [Status Characters](#status-characters)
- [Supported Task Formats](#supported-task-formats)
- [Commands](#commands)
- [Plugin Settings](#plugin-settings)
- [Architecture](#architecture)

---

## Board Types

### Task-Level Boards (Default)
Each card represents a single task. This is the default behavior.

### File-Level Boards
Visualize entire files as cards instead of individual tasks. Perfect for managing projects, content pipelines, or any file-based workflow.

````yaml
```tasks-kanban
name: Project Overview
viewType: files
source:
  folders: ["Projects/"]
  exclude: ["Projects/Archive/"]
columns:
  - name: Planning
    match:
      frontmatter: { status: planning }
  - name: Active
    match:
      frontmatter: { status: active }
    onDrop:
      setFrontmatter: { status: active }
  - name: Completed
    match:
      completion: { min: 100 }
```
````

Each file card shows:
- File name
- Task completion progress (e.g., "5/12 tasks")
- Earliest due date across all tasks
- Highest priority

---

## Templates

### One-Click Board Templates
Create boards from 8 built-in productivity frameworks:

**Command:** `Tasks Kanban: Insert task board from template`

| Template | Category | Description |
|----------|----------|-------------|
| Simple Kanban | Workflow | Basic To Do → Doing → Done |
| GTD | Workflow | Inbox, Next Actions, Waiting, Someday |
| Eisenhower Matrix | Prioritization | Urgent × Important quadrants |
| PARA | Lifecycle | Projects, Areas, Resources, Archives |
| MoSCoW | Prioritization | Must, Should, Could, Won't |
| OKR Tracker | Agile | Objectives by confidence level |
| Sprint Board | Agile | Backlog → Sprint → Done |
| Content Pipeline | Workflow | Ideas → Drafts → Review → Published |

### Per-Column Templates
Define default text appended to new tasks in specific columns:

```yaml
columns:
  - name: Today
    match: { status: [" "] }
    template: "#today {{today}}"
```

**Available variables:**
| Variable | Output |
|----------|--------|
| `{{today}}` | Today's date (YYYY-MM-DD) |
| `{{tomorrow}}` | Tomorrow's date |
| `{{nextWeek}}` | Date 7 days from now |
| `{{column}}` | Column name |
| `{{file}}` | Board file name (without .md) |

---

## Filtering

### Board-Level Filters
Filter which tasks appear on the board:

```yaml
filter:
  tags: ["#project"]
  paths: ["Work/"]
  excludeTags: ["#archived"]
  excludePaths: ["Templates/"]
```

### Advanced Filters
Interactive filter bar above your board for dynamic filtering:

**Priority Filters:**
```yaml
filter:
  priority: [high, medium]
  excludePriority: [low, none]
```

**Due Date Filters:**
- Presets: `today`, `this-week`, `next-week`, `overdue`, `has-date`, `no-date`
- Custom ranges: `dueRange: { from: "2024-01-01", to: "2024-12-31" }`

**Status Filters:**
```yaml
filter:
  status: [" ", "/"]  # Only incomplete and in-progress
```

All filters persist to your board's YAML configuration automatically.

---

## Swimlanes

Group tasks within columns by priority, due date, tags, or custom criteria:

```yaml
columns:
  - name: To Do
    match: { status: [" "] }
    swimlanes:
      groupBy: priority  # Auto-generates priority lanes
  - name: In Progress
    match: { status: ["/"] }
    swimlanes:
      groupBy: tags
      groupTags: ["#frontend", "#backend", "#docs"]
```

**Built-in groupBy presets:**
| Preset | Lanes Generated |
|--------|-----------------|
| `priority` | Highest, High, Medium, Low, No Priority |
| `dueDate` | Overdue, Today, This Week, Next Week, Later, No Date |
| `tags` | One lane per tag in `groupTags` array |

**Custom lanes:**
```yaml
swimlanes:
  lanes:
    - name: High Priority
      match: { priority: [highest, high] }
    - name: Normal
      match: { priority: [medium, low, lowest] }
    - name: Other
      catchAll: true
```

**Drag-drop between lanes:**
Moving tasks between swimlanes automatically updates their metadata:
- Priority lanes → updates task priority
- Tag lanes → removes old tag, adds new tag
- Supports hierarchical tags (`#project/client` satisfies `#project` lane)

Control the behavior with `onDrop`:
```yaml
swimlanes:
  onDrop: update   # (default) Automatically update metadata
  # onDrop: prompt # Ask for confirmation before changes
  # onDrop: none   # Disable lane-to-lane drops
```

---

## Column Rules & Actions

### Match Rules
Columns match tasks by:
| Rule | Description | Example |
|------|-------------|---------|
| `status` | Checkbox character | `[" ", "/"]` |
| `tags` | Tasks with specific tags | `["#work", "#urgent"]` |
| `excludeTags` | Exclude tasks with tags | `["#archived"]` |
| `priority` | Task priority level | `["high", "highest"]` |
| `due` | Due date preset | `"today"`, `"overdue"`, `"this-week"` |
| `files` | File path prefixes | `["Projects/", "Work/"]` |
| `folder` | Files in folder (file-level) | `"Projects/Active/"` |
| `frontmatter` | YAML frontmatter fields | `{ status: active }` |
| `completion` | Task completion % (file-level) | `{ min: 100 }` |
| `catchAll` | Match remaining tasks | `true` |
| `limit` | WIP limit (max cards) | `3` |

### OnDrop Actions
Define what happens when a card is dropped into a column:

```yaml
columns:
  - name: In Progress
    match: { status: ["/"] }
    onDrop:
      setStatus: "/"
      addTags: ["#wip"]

  - name: Done
    match: { status: ["x"] }
    onDrop:
      setStatus: "x"
      removeTags: ["#wip", "#urgent"]

  - name: Archive
    match: { folder: "Archive/" }
    onDrop:
      moveToFolder: "Archive/"

  - name: Backend Team
    match: { tags: ["#backend"] }
    onDrop:
      swapTags: { from: "#frontend", to: "#backend" }
```

**Available mutations:**
| Mutation | Description |
|----------|-------------|
| `setStatus` | Change checkbox status |
| `addTags` | Add tags to task |
| `removeTags` | Remove tags from task |
| `swapTags` | Replace one tag with another |
| `moveToFile` | Move task to a different file |
| `setFrontmatter` | Update frontmatter fields (file-level) |
| `moveToFolder` | Move file to folder (file-level) |

**Smart defaults:** If `onDrop` is omitted, it's inferred from `match`. Status-matched columns update status, tag-matched columns swap tags, etc.

---

## Archive

Automatically manage completed tasks:

```yaml
options:
  archiveEnabled: true
  archiveDaysDelay: 7        # Archive tasks completed >7 days ago
  autoArchiveColumn: "Done"  # Archive when dropped in this column
  archiveTag: "archived"     # Tag added to archived tasks
  archiveHideTasks: true     # Hide archived tasks from board
  addCompletionDate: true    # Add ✅ YYYY-MM-DD when completing
```

**Manual archiving:** Create an Archive column that matches the archive tag:
```yaml
columns:
  - name: Archive
    match: { tags: ["#archived"] }
```

Dragging tasks out of the Archive column removes the archive tag.

---

## Keyboard Navigation

Navigate and manage your board without a mouse:

| Key | Action |
|-----|--------|
| `↑` / `↓` | Navigate between cards in column |
| `←` / `→` | Navigate between columns |
| `Home` / `End` | Jump to first/last card in column |
| `Ctrl+Home` / `Ctrl+End` | Jump to first/last card on board |
| `Enter` | Edit selected card |
| `Delete` / `Backspace` | Cancel task (soft delete) |
| `Ctrl+Delete` | Permanently delete task |
| `Escape` | Deselect card / Clear selection |
| `n` | Quick-add new task in current column |
| `Ctrl+A` | Select all cards in column |

The board uses ARIA attributes for screen reader accessibility.

---

## Bulk Operations

Select multiple cards and perform actions in bulk:

**Selection:**
| Action | Shortcut |
|--------|----------|
| Toggle selection | `Ctrl+Click` |
| Range select | `Shift+Click` |
| Select all in column | `Ctrl+A` |
| Clear selection | `Escape` |

**Bulk Actions** (floating bar appears when 2+ cards selected):
- **Move to...** - Move selected cards to another column
- **Status...** - Change status (To Do, In Progress, Done, Cancelled)
- **Send to file...** - Dispatch all selected tasks to another file
- **Cancel** - Soft delete (mark as cancelled)
- **Delete** - Permanently remove from files

---

## Webhooks

Integrate your board with external automation tools (n8n, Make, Zapier):

```yaml
webhooks:
  - url: "https://your-n8n.com/webhook/abc123"
    events: ["task.moved", "task.created"]
    secret: "my-webhook-secret"  # References plugin settings
```

**Events:**
| Event | Trigger |
|-------|---------|
| `task.created` | New task added via the board |
| `task.moved` | Card dragged to new column |

**Payload includes:** task text, status, tags, source file, line number, column names, and mutations applied.

Configure secrets in Plugin Settings to keep API keys out of your vault files.

---

## Context Menu & Quick Actions

### Context Menu
Right-click any task card for quick actions:
- **Edit** - Edit the task text inline
- **Send to file...** - Move task (with subtasks) to another file
- **Cancel task** - Mark as cancelled (soft delete)
- **Delete permanently** - Remove from file completely

### Send to File
Dispatch tasks from an inbox to the appropriate project file:
1. Right-click a task card
2. Select "Send to file..."
3. Choose from smart suggestions or search any file

**Smart suggestions prioritize:**
- Recently used destinations
- Files matching task's tags
- Files linked in the task text
- Files in the same folder

The task and all its subtasks move to the end of the destination file.

### Quick Add
Click the `+` button on any column header to add a new task. Tasks are created in your configured task file with appropriate status and tags for that column.

### GUI Settings Modal
Click the gear icon on any board to:
- Rename the board
- Add, remove, and reorder columns
- Configure column match rules (status, tags)
- Set WIP limits per column
- Configure filters with autocomplete

---

## Board Options

| Option | Default | Description |
|--------|---------|-------------|
| `viewType` | `tasks` | `tasks` for individual cards, `files` for file aggregation |
| `taskFile` | - | Default file for new tasks |
| `showSubtasks` | `true` | Display subtask progress on cards |
| `showSubtasksAsCards` | `true` | Show subtasks as separate cards on the board |
| `showCompleted` | `true` | Show tasks with `[x]` status |
| `showEmptyColumns` | `true` | Display columns with no tasks |
| `cardPreview` | `false` | Show content preview below task title |
| `fullWidth` | `false` | Distribute columns evenly across board width |
| `readOnly` | `false` | Disable all editing and dragging (dashboard mode) |
| `clickableCheckboxes` | `true` | Allow completing tasks by clicking checkbox |
| `inheritFrontmatterTags` | `false` | Apply file's frontmatter tags to its tasks |
| `addCompletionDate` | `false` | Add ✅ YYYY-MM-DD when completing tasks |
| `archiveEnabled` | `false` | Enable auto-archiving of old completed tasks |
| `archiveDaysDelay` | `30` | Days after completion before auto-archive |
| `archiveTag` | `archived` | Tag added to archived tasks |
| `autoArchiveColumn` | - | Column name that triggers archive on drop |
| `archiveHideTasks` | `false` | Hide archived tasks from all columns |
| `sort` | - | Sort by: `priority`, `dueDate`, `createdDate`, `alphabetical`, `manual` |
| `sortDirection` | `asc` | Sort direction: `asc` or `desc` |
| `colors` | - | Card color configuration (see Card Colors) |

---

## Card Sorting & Colors

### Sorting
Sort tasks within columns:

```yaml
options:
  sort: priority
  sortDirection: asc
```

**Per-column override:**
```yaml
columns:
  - name: To Do
    match: { status: [" "] }
    sort: dueDate
    sortDirection: desc
```

### Card Colors
Add color stripes to cards based on priority or tags:

**Color by priority:**
```yaml
options:
  colors:
    by: priority
    priorityColors:
      highest: "#dc2626"
      high: "#ea580c"
      medium: "#ca8a04"
```

**Color by tags:**
```yaml
options:
  colors:
    by: tags
    tagColors:
      "#frontend": "#3b82f6"
      "#backend": "#10b981"
      "#urgent": "#ef4444"
```

---

## Status Characters

| Status | Meaning | Example |
|--------|---------|---------|
| `" "` | Todo (unchecked) | `- [ ] Task` |
| `"x"` | Done (checked) | `- [x] Task` |
| `"/"` | In Progress | `- [/] Task` |
| `"-"` | Cancelled | `- [-] Task` |
| `"?"` | Needs Input | `- [?] Task` |

---

## Supported Task Formats

### Tasks Plugin (emoji format)
```markdown
- [ ] Buy groceries 📅 2024-12-01
- [ ] Call mom ⏳ 2024-11-30
- [x] Finish report ✅ 2024-11-29
```

### Dataview (inline fields)
```markdown
- [ ] Buy groceries [due:: 2024-12-01]
- [ ] Call mom [scheduled:: 2024-11-30]
```

### Supported Metadata
| Field | Tasks Plugin | Dataview |
|-------|--------------|----------|
| Due Date | 📅 YYYY-MM-DD | [due:: YYYY-MM-DD] |
| Scheduled | ⏳ YYYY-MM-DD | [scheduled:: YYYY-MM-DD] |
| Start Date | 🛫 YYYY-MM-DD | - |
| Created | ➕ YYYY-MM-DD | - |
| Done | ✅ YYYY-MM-DD | - |
| Priority | ⏫🔼🔸🔽⏬ | - |
| Tags | #tag | #tag |
| Recurrence | 🔁 pattern | - |

*Note: Start date, priority, and recurrence are only parsed from Tasks plugin emoji format, not Dataview inline fields.*

---

## Commands

| Command | Description |
|---------|-------------|
| `Tasks Kanban: Insert task board from template` | Insert a board using a productivity template |
| `Tasks Kanban: Insert task board` | Insert a blank board code block |
| `Tasks Kanban: Create task board file` | Create a new file with a board |
| `Tasks Kanban: Refresh Task Index` | Force rebuild the task index from all vault files |

---

## Plugin Settings

| Setting | Description |
|---------|-------------|
| Default inbox file | File to add new tasks to when a board doesn't specify `taskFile` |
| Auto-refresh | Update boards when files change |
| Refresh interval | Debounce time for file change events (ms) |
| Webhook secrets | Named secrets for webhook authentication |

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Obsidian                          │
│  ┌─────────────────────────────────────────────┐    │
│  │             Tasks Kanban Plugin               │    │
│  │  ┌───────────┐  ┌───────────┐               │    │
│  │  │ TaskIndex │  │FileWatcher│               │    │
│  │  └─────┬─────┘  └─────┬─────┘               │    │
│  │        │              │                      │    │
│  │  ┌─────▼──────────────▼─────┐               │    │
│  │  │    Code Block Processor   │               │    │
│  │  │  ┌─────────────────────┐ │               │    │
│  │  │  │  FileBasedBoard.tsx │ │               │    │
│  │  │  │  ┌────┐ ┌────┐ ┌───┐│ │               │    │
│  │  │  │  │Col │ │Col │ │Col││ │               │    │
│  │  │  │  └────┘ └────┘ └───┘│ │               │    │
│  │  │  └─────────────────────┘ │               │    │
│  │  └──────────────────────────┘               │    │
│  └─────────────────────────────────────────────┘    │
│                         │                            │
│                         ▼                            │
│              vault.process() ──► Markdown Files      │
└─────────────────────────────────────────────────────┘
```

**Key Design Decisions:**
- **File-based boards**: Boards defined in markdown, not plugin settings
- **Preact** for lightweight, fast UI
- **TaskIndex**: Central cache with per-file parsing and real-time updates
- **vault.process()** for atomic file operations
- **Content-based matching** to find task lines reliably
- **Optimistic UI** for instant visual feedback

---

## Acknowledgments

Inspired by:
- [CardBoard](https://github.com/roovo/obsidian-card-board) plugin
- [Tasks](https://github.com/obsidian-tasks-group/obsidian-tasks) plugin
- [Dataview](https://github.com/blacksmithgu/obsidian-dataview) plugin
