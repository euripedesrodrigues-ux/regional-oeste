# Regional Oeste — notes for agents

## What this project is
A **single-file static dashboard** (`index.html.html`, ~570 KB) written in plain HTML/CSS/JS
(lang `pt-BR`). There is no package manager, no build step, no backend and no test suite —
all markup, styles and application logic live in that one file.

## Running it
```bash
docker compose -f docker-compose.base44.yml up -d --build
```
Serves the repo root on host port **3000** via `live-server` (bind-mounted source, auto-reload
on change to `index.html.html`).

Verify it is really up (and serving the live file, not a cached bundle):
```bash
curl -s localhost:3000/ | grep -c 'Regional Oeste'
docker compose -f docker-compose.base44.yml ps
```

## Non-obvious gotchas
- **The file name has a double extension**: `index.html.html`. A default static host will not
  map `/` to it, so the dev server is started with `--entry-file=index.html.html`. Don't rename
  the file casually (the repo has no other entry point) — if you ever do, update the compose
  command's `--entry-file` to match.
- **All third-party libraries load from public CDNs at runtime** (Tabler icons webfont,
  Chart.js 4.4.1, SheetJS/xlsx 0.18.5, Firebase JS SDK 10.12.2 from `gstatic.com`). The page
  therefore needs network access from the *browser*; nothing is vendored locally.
- **Firebase is hardcoded in the HTML** (`index.html.html`, the `firebaseConfig` object near the
  top): project `painelregionaloeste`, Realtime Database `regionalOeste` / `corretores` nodes.
  The RTDB is publicly readable (verified with a plain `GET` to
  `https://painelregionaloeste-default-rtdb.firebaseio.com/regionalOeste.json` → HTTP 200), so no
  credentials have to be supplied to boot. No environment variables are used anywhere.
  Because these values are literal in the file, they are **not** delivered through
  `/run/base44/app.env`; if the user ever rotates or changes the Firebase project, edit the
  `firebaseConfig` object directly.
- **The page still renders without Firebase**: the `configured` flag plus a `try/catch` fall back
  to built-in seed data (`SEED`) and `localStorage` keys (`regionalOesteV2`, `corretores_extra`,
  `okr_ov`, `okr_ex`). The header badge shows "Firebase conectado" / "Firebase erro" / "Modo local",
  which is the quickest way to see whether the remote database was reached.
- **Data entry is two-way**: several screens (`inserir`, `editar`, `converter`, `okrs`, `corretores`)
  write to Firebase via the `window._fbSave*` helpers. Changes made in the preview therefore persist
  in the *real* remote database and are visible to other devices — be careful with anything that
  looks like production data.
- The "Limpar cache do navegador" button wipes localStorage **and** writes an empty object to the
  Firebase `regionalOeste` node; avoid clicking it while exploring.

## Editing style
Keep the single-file, vanilla-JS style (plain `<script>` blocks, `window.*` globals, string-built
HTML). There is no bundler, so no imports/build tooling should be introduced for small changes.
