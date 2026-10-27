# M0 · Foundations — how to work these tasks

Spec: `.docs/specs/cubu-dash-foundation.md` (pt-BR). Project rules: `RULES.md`. Engine: Godot 4.7, GL Compatibility.
All work is done **locally** (Cursor/Codex on the dev machine). No cloud agents.

## Order
0. `0.0-editor-setup.md` first.
1. `0.1-router.md`, `0.2-content-classes.md`, `0.4-save-schema.md` (independent; 0.2 installs GUT, so do 0.2 first if another task needs tests before it exists).
2. `0.3-content-repository.md` (needs 0.2), `0.5-run-contracts.md` (needs 0.1), `0.6-auth-session.md` (needs 0.4), `0.7-theme-service.md` (needs 0.3).

## Loop (one task at a time)
1. Pick the first task with `Status: todo` whose dependencies are `done`.
2. Set `Status: in-progress` in the task file.
3. Work on `main` (Hil's decision, 27/10: no PRs, no feature branches).
4. Implement only the Scope. Anything in "Out of scope" stays untouched.
5. Run tests (GUT, headless):
   ```
   godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://tests -ginclude_subdirs -gexit
   ```
   Also boot the game once headless to catch parse errors: `godot --headless --path . --quit`.
6. When a logical part is finished and its tests pass, commit it to `main` and push. One commit per logical part, English message (`feat(router): add Router autoload with back stack`). Never commit with failing tests.
7. When the task is complete, set `Status: done` and fill the `## Log` section (what changed, commit SHAs, test output summary).
8. No PRs and no merge step.

## Rules
- Coding standard, in order: constructor pattern + adapter (Port + Adapter + Factory), clean code, semantic code readable by humans.
- Identifiers, code, comments, commits and UI text in English.
- No secrets, tokens or passwords anywhere in the repo.
- Keep existing behaviour working (menu → select → run → pause → menu; dev lab; control settings).
- Do not edit `.godot/` or `.uid` files by hand; let the editor generate them.
- Never install software on the dev machine on your own; write files and instructions only.
- Anything touching secrets, credentials, `.env` or signing keys → stop and ask. Secrets live only in `.env` (ignored).
- **Ambiguity → stop and ask** (write the question in the task Log, set `Status: blocked`). Items marked HYPOTHESIS: implement as written, behind a constant/flag, and do not extend them.

## Status values
`todo` · `in-progress` · `blocked` · `done`.
