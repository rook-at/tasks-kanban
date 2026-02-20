# Tasks Kanban

A Kanban-style task board plugin for Obsidian that displays your vault's tasks in a visual board layout.

**Your tasks stay in your markdown files. The board is a dynamic view - all changes sync back to your actual notes.**

---

## Features

- **File-Based Boards** - Define boards in markdown code blocks, no plugin settings needed
- **Drag-and-Drop** - Move tasks between columns; status updates automatically in your files
- **Mobile Optimized** - Touch drag-and-drop, swipe navigation, responsive layout
- **Instant Feedback** - Optimistic UI updates for immediate visual response
- **Inline Editing** - Double-click any task to edit directly on the board
- **Rich Metadata** - Due dates, priorities, recurrence, tags, subtask progress
- **Swimlanes** - Group tasks by priority, due date, tags, or custom criteria
- **Bulk Operations** - Select multiple cards for batch actions
- **Webhooks** - Integrate with n8n, Make, Zapier for automation
- **Keyboard Navigation** - Full keyboard support with ARIA accessibility

---

## Quick Start

Create a new note and add a `task-board` code block:

````yaml
```task-board
name: My Tasks
columns:
  - name: To Do
    match: { status: [" "] }
  - name: Doing
    match: { status: ["/"] }
    limit: 3
  - name: Done
    match: { status: ["x"] }
```
````

Or use a template: Open the command palette and run **Tasks Kanban: Insert task board from template**.

---

## Built-in Templates

Get started quickly with 8 productivity frameworks. Run the command palette (`Ctrl/Cmd+P`) and select **Tasks Kanban: Insert task board from template**.

| Template | Description |
|----------|-------------|
| **Simple Kanban** | Classic To Do → Doing → Done workflow |
| **GTD** | Getting Things Done: Inbox, Next Actions, Waiting For, Someday |
| **Eisenhower Matrix** | Urgent × Important priority quadrants |
| **PARA** | Projects, Areas, Resources, Archives (file-level) |
| **MoSCoW** | Must, Should, Could, Won't prioritization |
| **OKR Tracker** | Track objectives by confidence level |
| **Sprint Board** | Backlog → Sprint → In Progress → Review → Done |
| **Content Pipeline** | Ideas → Drafting → Editing → Published (file-level) |

Each template is customizable - set your source folder, WIP limits, and board name before inserting.

---

## AI-Assisted Board Creation

Use your favorite AI assistant (ChatGPT, Claude, etc.) to generate custom boards. Copy the prompt template below, fill in your requirements, and paste the whole thing into your AI chat.

### Prompt Template

````
I want a Tasks Kanban board for [DESCRIBE YOUR WORKFLOW HERE].

Example: "a 4-column software dev board with Backlog, To Do, In Progress (limit 3),
and Done. Group In Progress by priority swimlanes. Color cards by tags #frontend,
#backend, #devops."

Please generate a valid task-board YAML code block using this schema reference:

```yaml
# TASKS KANBAN - YAML SCHEMA REFERENCE

# ═══════════════════════════════════════════════════════════════════════════════
# TOP-LEVEL FIELDS
# ═══════════════════════════════════════════════════════════════════════════════

name: string                    # Board display name (required)
viewType: tasks | files         # "tasks" = individual task cards (default)
                                # "files" = entire files as cards

taskFile: string                # Default file path for new tasks created on board
                                # Example: "Inbox.md" or "Projects/Tasks.md"

# ═══════════════════════════════════════════════════════════════════════════════
# FILTER - Which tasks appear on the board
# ═══════════════════════════════════════════════════════════════════════════════

filter:
  tags: [string]                # Include only tasks with these tags
                                # Example: ["#work", "#project"]

  paths: [string]               # Include only tasks from these folder paths
                                # Example: ["Projects/", "Work/"]

  excludeTags: [string]         # Exclude tasks with these tags
                                # Example: ["#archived", "#template"]

  excludePaths: [string]        # Exclude tasks from these folder paths
                                # Example: ["Templates/", "Archive/"]

  priority: [string]            # Include only these priority levels
                                # Values: highest, high, medium, low, lowest, none

  excludePriority: [string]     # Exclude these priority levels

  due: string                   # Due date preset filter
                                # Values: today, this-week, next-week, overdue, has-date, no-date

  dueRange:                     # Custom due date range
    from: "YYYY-MM-DD"
    to: "YYYY-MM-DD"

  status: [string]              # Include only these checkbox statuses
                                # Values: " " (todo), "x" (done), "/" (in-progress), "-" (cancelled), "?" (needs-input)

# ═══════════════════════════════════════════════════════════════════════════════
# SOURCE - For file-level boards (viewType: files)
# ═══════════════════════════════════════════════════════════════════════════════

source:
  folders: [string]             # Folders to scan for files
                                # Example: ["Projects/", "Areas/"]

  exclude: [string]             # Folders to exclude
                                # Example: ["Projects/Archive/"]

# ═══════════════════════════════════════════════════════════════════════════════
# COLUMNS - Board columns (required, array)
# ═══════════════════════════════════════════════════════════════════════════════

columns:
  - name: string                # Column display name (required)

    # ─────────────────────────────────────────────────────────────────────────
    # MATCH - Which tasks belong in this column
    # ─────────────────────────────────────────────────────────────────────────

    match:
      status: [string]          # Checkbox characters to match
                                # Values: " ", "x", "/", "-", "?"
                                # Example: [" ", "/"] for todo and in-progress

      tags: [string]            # Tasks must have ALL these tags
                                # Example: ["#urgent", "#work"]

      excludeTags: [string]     # Exclude tasks with these tags

      priority: [string]        # Match priority levels
                                # Values: highest, high, medium, low, lowest, none

      due: string               # Due date preset
                                # Values: today, overdue, this-week, next-week, has-date, no-date

      files: [string]           # Match tasks from these file paths
                                # Example: ["Projects/Alpha/", "Inbox.md"]

      folder: string            # For file-level boards: files in this folder
                                # Example: "Projects/Active/"

      frontmatter:              # Match YAML frontmatter fields (file-level boards)
        key: value              # Example: { status: active, type: project }

      completion:               # For file-level boards: task completion percentage
        min: number             # Minimum completion % (0-100)
        max: number             # Maximum completion % (0-100)

      catchAll: boolean         # true = match all remaining unmatched tasks
                                # Place this column last as a fallback

    # ─────────────────────────────────────────────────────────────────────────
    # ONDROP - Actions when a card is dropped into this column
    # ─────────────────────────────────────────────────────────────────────────

    onDrop:
      setStatus: string         # Change checkbox status
                                # Values: " ", "x", "/", "-", "?"

      addTags: [string]         # Add these tags to the task
                                # Example: ["#wip", "#review"]

      removeTags: [string]      # Remove these tags from the task
                                # Example: ["#inbox", "#todo"]

      swapTags:                 # Replace one tag with another
        from: string            # Tag to remove (e.g., "#draft")
        to: string              # Tag to add (e.g., "#published")

      moveToFile: string        # Move task to a different file
                                # Example: "Archive/Completed.md"

      setFrontmatter:           # Update frontmatter (file-level boards)
        key: value              # Example: { status: active, reviewed: true }

      moveToFolder: string      # Move file to folder (file-level boards)
                                # Example: "Archive/"

    # ─────────────────────────────────────────────────────────────────────────
    # COLUMN OPTIONS
    # ─────────────────────────────────────────────────────────────────────────

    limit: number               # WIP limit - max cards allowed in column
                                # Cards over limit are visually indicated

    sort: string                # Sort order for this column
                                # Values: priority, dueDate, createdDate, alphabetical, manual

    sortDirection: asc | desc   # Sort direction

    template: string            # Text appended to new tasks in this column
                                # Variables: {{today}}, {{tomorrow}}, {{nextWeek}}, {{column}}, {{file}}
                                # Example: "#today 📅 {{today}}"

    # ─────────────────────────────────────────────────────────────────────────
    # SWIMLANES - Group tasks within a column
    # ─────────────────────────────────────────────────────────────────────────

    swimlanes:
      groupBy: string           # Auto-generate lanes by grouping
                                # Values: priority, dueDate, tags

      groupTags: [string]       # For groupBy: tags - which tags to create lanes for
                                # Example: ["#frontend", "#backend", "#design"]

      onDrop: string            # Behavior when dragging between swimlanes
                                # Values: update (default), prompt, none

      lanes:                    # Custom lane definitions (instead of groupBy)
        - name: string          # Lane display name
          match:                # Same match rules as columns
            priority: [string]
            tags: [string]
            due: string
            # ... any match rule
          catchAll: boolean     # true = catch remaining tasks

# ═══════════════════════════════════════════════════════════════════════════════
# OPTIONS - Board-wide settings
# ═══════════════════════════════════════════════════════════════════════════════

options:
  showSubtasks: boolean         # Show subtask progress on cards (default: true)
  showSubtasksAsCards: boolean  # Show subtasks as separate cards (default: true)
  showCompleted: boolean        # Show completed tasks (default: true)
  showEmptyColumns: boolean     # Show columns with no tasks (default: true)
  cardPreview: boolean          # Show content preview below task title (default: false)
  fullWidth: boolean            # Distribute columns evenly (default: false)
  readOnly: boolean             # Disable editing/dragging (default: false)
  clickableCheckboxes: boolean  # Allow completing via checkbox click (default: true)
  inheritFrontmatterTags: boolean # Apply file's frontmatter tags (default: false)
  addCompletionDate: boolean    # Add completion date when marking done (default: false)

  # Sorting
  sort: string                  # Default sort for all columns
                                # Values: priority, dueDate, createdDate, alphabetical, manual
  sortDirection: asc | desc

  # Archive settings
  archiveEnabled: boolean       # Enable auto-archiving (default: false)
  archiveDaysDelay: number      # Days after completion before archive (default: 30)
  archiveTag: string            # Tag added to archived tasks (default: "archived")
  autoArchiveColumn: string     # Column name that triggers archive on drop
  archiveHideTasks: boolean     # Hide archived tasks from board (default: false)

  # Card colors
  colors:
    by: priority | tags         # Color cards by priority or tags

    priorityColors:             # Custom priority colors (hex codes)
      highest: "#dc2626"
      high: "#ea580c"
      medium: "#ca8a04"
      low: "#65a30d"
      lowest: "#0891b2"

    tagColors:                  # Custom tag colors (hex codes)
      "#frontend": "#3b82f6"
      "#backend": "#10b981"
      "#urgent": "#ef4444"

# ═══════════════════════════════════════════════════════════════════════════════
# WEBHOOKS - External integrations
# ═══════════════════════════════════════════════════════════════════════════════

webhooks:
  - url: string                 # Webhook endpoint URL (required)
                                # Example: "https://your-n8n.com/webhook/abc123"

    events: [string]            # Events that trigger the webhook
                                # Values: task.created, task.moved

    secret: string              # Name of secret in plugin settings
                                # Keeps API keys out of vault files

    headers:                    # Custom HTTP headers
      key: value                # Example: { "X-Custom-Header": "value" }
```
````

### Example Requests

Here are some example descriptions you can put in the `[DESCRIBE YOUR WORKFLOW HERE]` section:

- "a 4-column board with Backlog, To Do, In Progress (limit 3), and Done"
- "a GTD board with Inbox, Next Actions, Waiting For, and Someday/Maybe - filter to only show tasks from my Projects folder"
- "a content pipeline with Ideas, Research, Writing, Editing, Published - color cards by tags #tutorial, #opinion, #review"
- "a sprint board with swimlanes grouping tasks by priority in the In Progress column"

---

## Installation

### Manual Installation
1. Download the latest release from [GitHub](https://github.com/rook-at/tasks-kanban/releases)
2. Extract to `.obsidian/plugins/tasks-kanban/`
3. Enable the plugin in Settings → Community Plugins

### From Source
```bash
git clone https://github.com/rook-at/tasks-kanban
cd tasks-kanban
npm install
npm run build
```
Copy `main.js`, `manifest.json`, and `styles.css` to your vault's plugin folder.

### From Obsidian Community Plugins
*Coming soon* - the plugin is pending submission to the community plugins directory.

---

## Documentation

For complete documentation including all features, options, and advanced configurations:

**[View Full Documentation](docs/DOCUMENTATION.md)**

---

## Feedback

Found a bug or have a feature request? [Open an issue](https://github.com/rook-at/tasks-kanban/issues) - feedback is welcome!

---

## License

AGPL-3.0 - see [LICENSE](LICENSE) file
