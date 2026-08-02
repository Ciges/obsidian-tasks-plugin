# Code Study: Obsidian Tasks v8.2.1

Analysis of the `main.js` bundle (pretty-printed, ~880 KB).

---

## Basic Information

- **ID:** `obsidian-tasks-plugin`
- **Name:** Tasks
- **Version:** 8.2.1
- **Minimum Obsidian version:** 1.8.7
- **Original authors:** Clare Macrae, Ilyas Landikov (created by Martin Schenck)
- **Official repo:** https://github.com/obsidian-tasks-group/obsidian-tasks

---

## Main Functionality

Manages tasks defined as checkboxes `- [ ]` across the entire vault. Extracts metadata from each task (dates, priority, status, dependencies) and allows filtering and grouping them via a custom query language in code blocks.

---

## Main Components

| Class (bundled name) | Role |
|---|---|
| `bf` (Plugin) | Main class, manages the lifecycle |
| `ws` (TaskCache) | Keeps all tasks in memory |
| `lf` (QueryRenderer) | Renders ` ```tasks ``` ` code blocks |
| `ef` (InlineRenderer) | Renders tasks inline in the editor |
| `jd` (TasksEvents) | Custom event system for the plugin |
| `mf` (SettingTab) | Settings panel |
| `at` (StatusRegistry) | Task status registry |

---

## Initialization Sequence (onload)

1. Register console logger
2. Load settings
3. Initialize local storage
4. Create event manager (`TasksEvents`)
5. Register settings panel
6. Initialize i18n with Obsidian metadata
7. Load custom task statuses
8. Create task cache with file listeners
9. Create inline and query renderers
10. Configure Obsidian properties types
11. Register editor extension (syntax highlighting)
12. Register editor suggest (autocomplete)
13. Register commands

---

## Events Listened To

```javascript
this.app.metadataCache.on("changed", ...)  // File metadata changes
this.app.vault.on("rename", ...)           // File renames
```

---

## Registered Commands

| ID | Name | Type |
|---|---|---|
| `edit-task` | Create or edit task | Editor |
| `toggle-done` | Toggle task done | Editor |
| `add-query-file-defaults-properties` | Add all Query File Defaults properties | Global |

Also generates dynamic commands for each registered custom status.

---

## Supported Task Metadata

- Dates: due, start, scheduled, created, done
- Recurrence (via rrule.js)
- Priority: highest, high, medium, low, normal
- Task dependencies
- Customizable statuses (symbol, name, type)
- Tags in the description
- Urgency calculated automatically

---

## Query Language

` ```tasks ``` ` blocks accept filters, sorting and grouping:

- Filters by date, status, priority, tags, file path...
- `filterByFunction`, `sortByFunction`, `groupByFunction` for custom JavaScript logic
- Global filter applicable to the entire vault from settings

---

## Task Statuses

Four base types: `TODO`, `IN_PROGRESS`, `DONE`, `CANCELLED`.

Each status has a checkbox symbol (e.g. `x` for done), a name and a type. Custom statuses can be created with CSS.

The status flow can be visualized as a Mermaid diagram from the settings panel.

---

## Public API (v1)

Accessible from other plugins via `plugin.apiV1()`:

```javascript
{
    createTaskLineModal()              // Opens modal to create a task
    editTaskLineModal(line)            // Opens modal to edit a task line
    executeToggleTaskDoneCommand(line, options)  // Toggles task status
}
```

---

## Libraries Included in the Bundle

| Library | Purpose |
|---|---|
| rrule.js | Recurring date calculation |
| EventEmitter2 | Event system |
| mustache.js | Templates for i18n |

---

## Installation Files

| File | Description |
|---|---|
| `main.js` | Full code bundle (pretty-printed) |
| `manifest.json` | Plugin metadata (ID, version, author...) |
| `styles.css` | CSS styles (~27 KB) |
| `data.json` | User configuration |
| `ui-strings.json` | Modal UI strings per language *(added in this fork)* |

---

## Fork Modifications

### 1. UI strings in a separate file with i18n support

**Branch:** `strings-in-a-separate-file` (merged into `main`)

The text strings in the "Create or edit task" modal were hardcoded in `main.js`. They have been moved to `ui-strings.json` with support for multiple languages.

**Mechanism:**

- Module-level variable `_taskUiAllStrings` loaded in `onload()` by reading `ui-strings.json` via `this.app.vault.adapter.read()`
- Helper function `_ui(key, fallback)` that selects the language using `RO()` (the same function used internally by i18next) on first call
- Automatic fallback to English if the language is not present in the file

**Externalized strings:**

| Key | Description |
|---|---|
| `title` | Modal title |
| `description` | Description field label |
| `descriptionPlaceholder` | Textarea placeholder |
| `priority` | Priority label |
| `priorityOptions.*` | Priority level labels |
| `recurs` | Recurs label |
| `recurrencePlaceholder` | Recurrence field placeholder |
| `notRecurring` | Text shown when there is no recurrence rule |
| `dateLabels.due/scheduled/start/done` | Date field labels |
| `datePlaceholder` | Date input placeholder |
| `onlyFutureDates` | Future dates checkbox label |
| `status` | Status label |
| `apply` / `cancel` | Modal buttons |
| `taskNote` | Label for the source note link |

**Languages included:** `en` (English), `es` (Spanish)

---

### 2. Link to the task's source note

**Branch:** `show-source-file`

An element is added at the bottom of the modal (before the Apply/Cancel buttons) showing the name of the note where the task is defined, as a clickable link.

**Implementation** (in `Yi.onOpen()`, after the Svelte component renders):

```javascript
const leaf = this.app.workspace.getLeaf("tab");
const file = this.app.vault.getAbstractFileByPath(this.task.path);
if (file) {
    await leaf.openFile(file);
    this.app.workspace.setActiveLeaf(leaf, { focus: true });
}
this.close();
```

- Uses `querySelector(".tasks-modal-button-section")` to insert the div before the buttons
- Opens the note in a new tab and focuses it
- Only shown when `task.path` is not empty (tasks that already exist in a note)

---

### 3. Time in the done date (`completion`)

Adds an optional hour/minute to the done date, in both supported task formats (Tasks emoji `✅` and Dataview `[completion:: ...]`), controlled by a new setting **"Add time also when task is completed"** (Settings → Dates, right after "Set done date"; default: **on**).

**New setting:**

- `addTimeToDoneDate: true` added to the settings defaults object (next to `setDoneDate`)
- Persisted/merged the same way as every other setting (no migration needed — missing key in an old `data.json` falls back to the default)

**Parsing (`gs.deserialize`)**

- Both `doneDateRegex` definitions (Tasks-emoji `fa("✅")` and Dataview `completion:: *(\d{4}-\d{2}-\d{2})`) had their capture group extended to `(\d{4}-\d{2}-\d{2}(?:T\d{2}:\d{2})?)` — the time suffix is optional, so existing date-only completions still match unchanged. Only the `doneDate` regex was touched; the other five date fields (`due`, `scheduled`, `start`, `created`, `cancelled`) keep the original date-only regex.
- New method `extractDoneDateField` (sibling of the generic `extractDateField`) parses the capture with `window.moment(value, [dateFormat, "YYYY-MM-DDTHH:mm"])` — moment picks whichever format leaves no unmatched characters. The generic `extractDateField` used by the other five fields is untouched.
- `deserialize()` calls `extractDoneDateField` only for the `doneDate` regex; every other field still calls `extractDateField`.

**Serialization (`gs.componentToString`, case `"doneDate"`)**

- Formats with `t.doneDate.format(X().addTimeToDoneDate ? "YYYY-MM-DDTHH:mm" : dateFormat)` instead of always using the shared `pa()` helper (which is still used, unmodified, by every other date field).
- The Dataview serializer (`Dc`) inherits this via `super.componentToString()`, so both formats are covered by this single change.

**Capture time already happens today**: `Task.handleNewStatus(status, now = window.moment())` already stores the full-precision `now` as `doneDate` when a task transitions to `DONE` — this was never truncated to midnight. This fork modification only changes what gets *written to* and *read from* the markdown line; no change was needed to how the date is captured at completion time.

**Caveat:** a task completed before this change (date-only `doneDate`) will render as `...T00:00` if it gets re-serialized (e.g. edited via the "Create or edit task" modal) after this fork modification, since the stored moment defaults to midnight when no time was parsed. It is not rewritten proactively — only on next serialization of that specific line.
