# itm-green-mobility
Project for Benin

## Structure

- `index.html` — the public marketing site and the "Platforms & Apps" hub page.
- `apps/<name>/` — one folder per mobile app simulation (interactive phone-frame demo):
  `rider`, `driver`, `station-operator`, `rental`.
- `portals/<name>/` — one folder per web portal/dashboard:
  `management`, `corporate`, `fleet`, `charging`.

Each app/portal is a self-contained static page (own `index.html`), linked from the
Platforms & Apps hub once it's built.

## Workflow

Each app/portal is built on its own branch (`feature/<name>-app` or `feature/<name>-portal`),
verified locally, then merged to `main` and deployed. `main` is always the deployable state.
