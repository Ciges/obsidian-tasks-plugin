# Obsidian Tasks Fork

Original Obsidian Tasks version: **8.2.1** (fork version **8.2.1.8**)

Local fork of the [Obsidian Tasks](https://github.com/obsidian-tasks-group/obsidian-tasks) plugin (built files, not source code) to create a custom version. Made by modifying the JavaScript code.

The following evolutions have been made:
- Task user interface is now in English and Spanish, with strings in separate json file
- A link to the note of the task is added
- The done date can optionally also record the time (`YYYY-MM-DDTHH:mm`), toggled by a new setting "Add time also when task is completed" (on by default)
- Recurring tasks always create the next occurrence as plain `[ ]` (empty/Todo), ignoring the custom status chain
- Recurring tasks: `start` no longer shifts on recurrence, `scheduled` always advances by the recurrence interval, and `[reminder:: ...]` advances along with it
- Two extra priorities, `normal` and `next-month`, with priority symbols configurable in `data.json`; tasks sort from highest to lowest priority with "no priority" last, and grouping by priority shows the text from `ui-strings.json` plus the emoji from `data.json`
- `[context:: ...]` inline fields are rendered in the task UI (styles in `styles.css`)
- Completed task dates are shown as `YYYY-MM-DD HH:mm`
