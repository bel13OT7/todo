# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately minimal todo app: plain HTML, CSS and vanilla JavaScript in a single file, [index.html](index.html). No framework, no build step, no package manager, no tests, no git repo. Keep it that way: do not add dependencies or split it into multiple files unless asked.

Run it by opening `index.html` in a browser.

## Architecture (all in index.html)

- **State**: a `todos` array of `{id, text, category, due, done}`. `due` is `'YYYY-MM-DD'` or `''`. `nextId` is derived from the max existing id on load.
- **`render()`** is the only thing that touches the list DOM. It calls `save()` first, then clears and rebuilds the `<ul>`. Every mutation (add, toggle, edit, delete, filter) changes `todos` or a view variable, then calls `render()`. Use `textContent`, never `innerHTML`, for user text.
- **Display order**, in `byDue` inside `render()`: undone before done, then earliest due date first (no date last), then oldest id. The category filter is applied after sorting.
- **Edit mode** is a single `editingId`. The row for that id renders text, category and date inputs instead of the meta line. Enter saves and Escape cancels.
- **Categories** are fixed in the `CATEGORIES` constant (Work, Personal, Others) and feed the add form, the edit select and the filter. `load()` folds any unknown legacy category into `'Others'`.
- **Due indicator** (the `.meta` line): "No due date", or "Due <date>" with an overdue, today or tomorrow note. Those notes are hidden for done items.
- **Theme**: CSS variables on `:root`, overridden by `:root[data-theme="dark"]`. The initial theme comes from `localStorage`, otherwise `prefers-color-scheme`. The header button is an ARIA `role="switch"` with a sliding knob.

## Persistence

`localStorage` keys: `todo-items` (the array) and `todo-theme` (`'light'` or `'dark'`). All access is wrapped in try/catch because storage is blocked in some contexts, such as a `data:` URL or a sandboxed iframe. The app then keeps working in memory. Do not change the key names or the item shape without a migration in `load()`, because users' saved items would be lost.

## mockup.html is generated

[mockup.html](mockup.html) shows the app in iPhone and Samsung Galaxy frames. Each frame is an iframe whose `srcdoc` is a copy of `index.html` with a fake `localStorage` shim injected, so sample items appear and the real saved list is untouched. Do not hand-edit it. It is produced by a PowerShell script that lives in the session scratchpad, not in this repo, so it may be gone. If it is, regenerate it by reading `index.html`, injecting the shim after `<head>`, HTML-escaping `&` and `"` for `srcdoc`, and rebuilding the frames. After any change to `index.html`, the mockup is stale until regenerated.
