# Obsidian Tasks Fork

Original Obsidian Tasks version: **8.2.1**

Local fork of the [Obsidian Tasks](https://github.com/obsidian-tasks-group/obsidian-tasks) plugin (built files, not source code) to create a custom version. Made by modifying the JavaScript code.

The following evolutions have been made:
- Task user interface is now in English and Spanish, with strings in separate json file
- A link to the note of the task is added
- The done date can optionally also record the time (`YYYY-MM-DDTHH:mm`), toggled by a new setting "Add time also when task is completed" (on by default)
- Recurring tasks always create the next occurrence as plain `[ ]` (empty/Todo), ignoring the custom status chain
- Recurring tasks: `start` no longer shifts on recurrence, `scheduled` always advances by the recurrence interval, and `[reminder:: ...]` advances along with it

