# Bussola

Gestione attività d'ufficio: appuntamenti condivisi, task e utenti con permessi.
App desktop portable per Windows (Tauri v2 + SvelteKit + TypeScript).

## Sviluppo (macOS)

```
npm install
npm run tauri dev
```

Test del core: `cargo test` in `src-tauri/`.

La build Windows (`.exe` senza installer) è prodotta da GitHub Actions
(`.github/workflows/build-windows.yml`).
