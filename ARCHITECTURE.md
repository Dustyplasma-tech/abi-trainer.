# Abi-Trainer: Architecture and upgrade guide

## Goal
New functions and new questions must never cost her learning progress. Everything below serves that rule.

## Layers (all inside one `index.html`)
1. **Content** (`data.js` part): topics, knowledge cards, random question generators `GEN[topicId]()`, video channels.
2. **State**: one JSON object `S` in `localStorage['abitrainer.v2']` (the key name is historical and must never change). Fields: `v` (data version), `t` topics, `c` cards, `h` days, `x` XP, `b` badges, `gd` goal days, `log` every answer, `ext` space for add-ons, `meta`.
3. **App logic**: answering, mastery, spaced repetition, XP, badges.
4. **UI**: views are registered tabs; actions are registered handlers.
5. **Extension API** `window.ABI`.
6. **Access gate**: password check before anything is shown.

## Storage rules (do not break)
- The key `abitrainer.v2` stays forever. Progress lives under this origin (the web address). Keep the GitHub Pages address stable.
- Never delete or rename a field. Only add fields.
- Data layout changes go into the migration chain `MIGR`: `MIGR[n]` upgrades version n to n+1 and only adds fields. Raise `STATE_V` by one for each new step.
- Before a migration runs, the old raw data is copied to `abitrainer.backup.v<old>`. Unreadable data is copied to `abitrainer.corrupt.<time>` and never overwritten.
- The progress file (`abi-trainer-progress`) is the second safety net: readers ignore unknown fields and keep them on write. See `Abi-Trainer-Fortschrittsformat.md`.

## Extension points (`ABI`)
- `ABI.registerTab({id,label,icon,view,order,show})`: new screen in the bottom bar. `view()` returns HTML.
- `ABI.registerAction(name, fn)`: handler for `data-act="name"` buttons.
- `ABI.on('boot'|'save'|'answer'|'session', fn)` and `ABI.emit`.
- `ABI.ext(name)`: private persistent storage for an add-on inside `S.ext[name]`. It is saved and exported automatically.
- `ABI.registerTopic(topic)` / `ABI.registerGenerator(id, fn)`: new topics in code.
- `ABI.registerPack(pack)`: data-only content (below).

## Question packs (no code)
A `.json` file loaded under Ich, Fragenpakete. It is validated, sanitized (only `<sup> <sub> <b> <i> <br>` survive) and stored in the browser. Pack topics get their own ids and cannot replace the built-in ones. Example: `Beispiel-Fragenpaket.json`.

## Release procedure
1. Change `data.js` / `app4.js` (or later versions). If the data layout changes: add a `MIGR` step, raise `STATE_V`.
2. Raise `APPV`.
3. Run `python3 build4.py "<password>"` (writes `standalone/index.html`), then `python3 t4.py` (continuity: an older build saves progress, the new one must read it unchanged).
4. Replace `index.html` in the GitHub repo. Keep the same repo name and file location.

## Password
Only a salted, 20000-round SHA-256 hash is in the file, not the password. The check runs in the browser, so it keeps out casual users but is not real security. Case and surrounding spaces are ignored.
