# Bussola

App desktop **portable per Windows**, senza installazione e senza permessi di amministratore, per la gestione di un piccolo ufficio: **appuntamenti condivisi**, **task per i dipendenti**, **utenti con password e permessi atomici**. 15-20 utenti al massimo, pochi dati.

Lo sviluppo avviene su **macOS** (VS Code + Claude Code). La build per Windows la fa **GitHub Actions** su un runner Windows.

---

## Regole di lavoro per Claude Code (leggere per prime)

1. **Resta nella cartella corrente.** Questa cartella è la radice del progetto. Non leggere, scrivere o modificare nulla fuori da essa. Non usare `cd` verso percorsi esterni, non toccare la home dell'utente, i dotfile o altri progetti. Il progetto va creato **direttamente qui**, non in una sottocartella.
2. **Nessun `sudo`.** Mai.
3. **Strumenti globali: chiedi prima.** Dipendenze di progetto (`npm install`, `cargo fetch`) le installi tu. Strumenti globali (Rust, Node, GitHub CLI) mancanti: dimmi cosa manca e il comando consigliato, e aspetta il mio ok prima di installarli.
4. **Lingua:** parla con me in italiano. Interfaccia dell'app in italiano. Codice e commenti tecnici in inglese.
   - **Messaggi di commit in inglese**, Conventional Commits con scope quando utile (`feat(sync): ...`, `ci(windows): ...`, `docs(readme): ...`): oggetto all'imperativo e minuscolo, senza punto finale; corpo a righe di ~78 colonne che spiega cosa cambia e perché (omesso solo per modifiche banali); trailer `Co-Authored-By` in fondo. Identità git globale, commit non firmati.
5. **Anonimato dei contenuti (requisito forte).** Riguarda i contenuti potenzialmente sensibili dell'app, non l'identità di chi committa (autore ed email dei commit sono quelli globali dell'utente). Nessun riferimento a enti, reparti, strutture, nomi di persone, percorsi di rete reali, indirizzi o loghi: né nel codice, né nei commenti, né nei dati di esempio, né nei messaggi di commit, né nei metadati dell'eseguibile. Dati di test sempre generici e inventati ("Utente A", "Task di prova").
6. **Nessuna funzione di rete esterna.** Niente telemetria, updater, chiamate HTTP, font o script da CDN, plugin Tauri che usano la rete. Content Security Policy rigida, tutto il contenuto della UI è locale.
7. **Procedi per fasi** (vedi sotto). A fine fase: test verdi, commit, riassunto in poche righe, poi **aspetta il mio via libera**. Non anticipare le fasi successive.
8. Prima di scrivere codice che dipende da API di Tauri v2, plugin o crate, **verifica la documentazione attuale**: le API cambiano.
9. **GitHub Actions solo su mia richiesta.** Non avviare run della CI (né build né test) senza chiedermelo: niente `gh workflow run`, e niente `git push` senza il mio ok esplicito, perché ogni push o PR su `main` fa partire il job di test (minuti Windows ×2). Commit locali liberi.

---

## Bootstrap (la prima volta che leggi questo file)

Esegui solo questi passi, poi fermati e riferisci.

1. **Verifica gli strumenti:** `git`, `node` e `npm`, `rustc` e `cargo`, `gh` (opzionale). Su macOS gli strumenti di build di Xcode (`xcode-select -p`). Se manca qualcosa, spiegamelo e aspetta il mio ok.
   - **Ambiente locale attuale (deciso, da non cambiare):** Node 26 e Rust stable installati con **Homebrew** (niente rustup: il Rust di Homebrew è condiviso con altri progetti), `gh` non installato (il repo GitHub si crea da web). Non proporre di cambiare questi strumenti. Senza rustup non si aggiungono target o toolchain in locale: la build Windows la fa solo la CI.
   - **In CI** si usa invece una Node **LTS** (`actions/setup-node`) e Rust stable via `dtolnay/rust-toolchain`.
2. **Git:** se non esiste, `git init -b main`. Crea `.gitignore` (`node_modules/`, `dist/`, `build/`, `.svelte-kit/`, `src-tauri/target/`, `.DS_Store`, `.env*`, `*.log`) e `.gitattributes` (`* text=auto eol=lf`).
3. **Identità dei commit:** si usa l'identità git **globale** già configurata (nome ed email reali vanno bene). Nessuna configurazione locale, nessun indirizzo `noreply`.
4. **Scaffold Tauri v2 + Svelte + TypeScript** direttamente in questa cartella, con `npm create tauri-app@latest` (controlla `--help` per le opzioni non interattive). Se la cartella non è vuota (c'è questo file) e il tool rifiuta, genera in una sottocartella temporanea, sposta il contenuto qui e rimuovi la temporanea, **senza sovrascrivere `CLAUDE.md`**.
   - Nome prodotto: `Bussola`. Package/crate: `bussola`. Identifier: `local.bussola.app`.
   - Nei metadati (`Cargo.toml`, `package.json`, `tauri.conf.json`): campo autori vuoto, descrizione generica ("Gestione attività d'ufficio"), nessun nome di società.
   - Finestra principale con titolo `Bussola`; la UI è una SPA statica (adapter static / SPA mode se si usa SvelteKit).
5. **Scarica le dipendenze:** `npm install`, poi `cargo fetch` (o `cargo check`) in `src-tauri/`. Aggiungi già i crate che servono al core (verifica le versioni attuali): `serde`, `serde_json`, `serde_jcs`, `ed25519-dalek`, `argon2`, `chacha20poly1305`, `rand`, `uuid` (v7), `sha2`, `thiserror`, `zeroize`. Aggiungi `tempfile` come dev-dependency.
6. **Configurazione di release in `src-tauri/Cargo.toml`:** `[profile.release]` con `strip = true`, `lto = true`, `codegen-units = 1`, `opt-level = "s"`.
7. **CI:** crea `.github/workflows/build-windows.yml` secondo la sezione "Build su GitHub Actions".
8. **Verifica:** `npm run build` e `cargo check` devono passare. Se riesci, avvia `npm run tauri dev` per vedere che la finestra si apre sul Mac, poi chiudila.
9. **Primo commit** (`chore: scaffold iniziale`), senza fare push.
10. **Spiegami come creare il repo su GitHub** (sezione sotto), adattando le istruzioni a ciò che hai trovato (es. se `gh` è già installato e autenticato). **Non creare il repo e non fare push senza la mia conferma esplicita.**
11. Riassumi lo stato in poche righe e chiedi se passare alla Fase 1.

---

## Creazione del repository GitHub (istruzioni da darmi a fine bootstrap)

Il repo deve essere **privato** e senza riferimenti all'ente. Proponi le due strade:

**A. Con GitHub CLI**
```
brew install gh            # se manca
gh auth login              # una tantum, browser o token
gh repo create bussola --private --source=. --remote=origin --push
```

**B. Da web**
1. Su github.com: *New repository* → nome `bussola` → **Private** → **senza** README, `.gitignore` o licenza (il repo deve nascere vuoto).
2. Poi in questa cartella:
```
git remote add origin git@github.com:<utente>/bussola.git    # oppure URL https
git push -u origin main
```

Poi ricordami di:
- attivare la **2FA** sull'account GitHub;
- controllare che **Actions** sia abilitato (Settings → Actions);
- lanciare la prima build da *Actions → build-windows → Run workflow* e scaricare l'`.exe` dagli **Artifacts** del run;
- tenere presente che i runner Windows consumano i minuti gratuiti del piano a velocità doppia (per questo uso, di norma, non è un problema).

---

## Build su GitHub Actions

File `.github/workflows/build-windows.yml`:
- trigger: `workflow_dispatch` per la build (**solo manuale**, lanciata dall'utente dal web: nessuna build automatica, nemmeno sui tag), e `pull_request`/push su `main` per i soli test;
- job di test su `ubuntu-latest` o `windows-latest`: `cargo test` in `src-tauri/`;
- job di build su **`windows-latest`**: checkout, `actions/setup-node` (con cache npm), `dtolnay/rust-toolchain@stable`, `swatinem/rust-cache` (workspace `src-tauri`), `npm ci`, poi `npm run tauri build -- --no-bundle` (produce solo l'`.exe`, niente installer; verifica che il flag esista ancora nella CLI attuale);
- variabile d'ambiente `RUSTFLAGS: -C target-feature=+crt-static --remap-path-prefix=${{ github.workspace }}=.` (CRT statico: non serve il runtime Visual C++ sui PC; il remap evita che percorsi del runner finiscano nell'exe). Nota: `RUSTFLAGS` da ambiente sostituisce quelli di `.cargo/config.toml`, quindi mettili tutti qui;
- `actions/upload-artifact` dell'exe prodotto in `src-tauri/target/release/`.

Niente firma del codice per ora. L'exe non firmato può essere bloccato da SmartScreen o AppLocker: è un tema da concordare con chi gestisce i PC.

---

## Architettura

### Principio: un solo scrittore per file
La cartella di lavoro sta su una condivisione SMB. **I lock di filesystem su SMB non sono affidabili e SQLite non va usato su rete.** Quindi nessun file condiviso è mai scritto da due processi:

```
<cartella-di-lavoro>/
  genesis.json            creato una sola volta, in modo esclusivo (create_new)
  keys/<username>.key     chiave privata cifrata di ogni utente
  events/<deviceId>.jsonl un file per ogni INSTALLAZIONE (non per utente), solo append
```

- **`deviceId`**: identificatore casuale generato al primo avvio di ogni installazione, salvato nella config locale. Così due PC dello stesso utente non scrivono mai nello stesso file.
- **Scrittura evento:** apri il proprio file in append, scrivi **una riga JSON completa terminata da `\n` con una sola chiamata di write**, poi flush/sync. Mai riscritture o troncamenti del proprio file.
- **Lettura incrementale:** per ogni file in `events/` ricorda l'offset di byte già letto; leggi solo fino all'**ultimo `\n`** (una riga finale incompleta = scrittura in corso, si riprova al giro dopo). Se un file risulta più corto dell'offset salvato, rileggilo da zero. Polling ogni 5-10 secondi; **non usare notifiche di filesystem su SMB**.
- **Stato = replay deterministico** di tutti gli eventi, in memoria, ordinati per `(hlc, deviceId, seq)`. Con questi volumi si può ricalcolare tutto da zero senza problemi; una cache locale (in `%LOCALAPPDATA%`) è un'ottimizzazione successiva. Gli snapshot condivisi **non** sono in scope finché non servono.
- Qualunque scrittura su file condivisi diversi da `events/<deviceId>.jsonl` (config admin, chiavi) avviene con **file temporaneo + rename atomico**.

### Formato evento (una riga JSONL)
Campi: `v` (versione formato), `id` (UUID v7), `device`, `seq` (contatore monotono per device, serve a rilevare buchi), `hlc`, `author` (username), `type`, `payload`, `sig`.
- Firma **Ed25519** sulla serializzazione canonica (JCS, RFC 8785) di tutti i campi tranne `sig`.
- **HLC** (Hybrid Logical Clock) al posto dell'ora di sistema, perché gli orologi dei PC non sono allineati. Il client rifiuta di scrivere se il proprio orologio si discosta troppo (es. > 5 minuti) dal massimo HLC osservato, e lo segnala in UI.
- Eventi duplicati (stesso `id`) ignorati. Righe malformate o con firma non valida: ignorate e registrate nel log diagnostico locale, mai fatali.
- Conflitti sullo stesso oggetto: **last-write-wins per campo** secondo l'ordine deterministico.
- Specifica completa del formato da scrivere in `docs/FORMATO.md` durante la Fase 1 e tenere aggiornata. Il campo `v` serve per le migrazioni future.

### Utenti, password, chiavi
- **Evento genesi:** `genesis.json` contiene `archiveId`, nome e chiave pubblica del primo admin, ed è autofirmato. Si crea con `create_new`: se esiste già, l'inizializzazione rifiuta. È la **radice di fiducia**.
- Al primo collegamento a una cartella esistente il client mostra l'**impronta (fingerprint) della chiave admin** e la fissa nella config locale insieme ad `archiveId` (trust on first use); se poi cambia, avvisa e blocca.
- Ogni utente ha una coppia **Ed25519**. La chiave privata è cifrata con **XChaCha20-Poly1305** usando una chiave derivata dalla password con **Argon2id**, e salvata in `keys/<username>.key` (JSON con parametri KDF, salt, nonce, ciphertext, chiave pubblica). **Il login è la decifratura della chiave**: nessun hash di password in giro.
- Creazione utente (evento `user.create` firmato da un admin): l'admin genera la coppia di chiavi e una password temporanea. **Al primo login l'utente genera una nuova coppia**, pubblica `user.key_rotated` firmato con la vecchia chiave e sostituisce il file chiave: così l'admin non conosce mai la chiave definitiva.
- Cambio password: ricifratura del file chiave con rename atomico (stessa chiave).
- Password dimenticata: **non recuperabile**; l'admin crea un nuovo utente/chiave (evento `user.key_reset`). Va scritto chiaramente anche nella UI.
- Zeroizza in memoria password e chiave privata appena possibile. Blocco automatico dopo inattività (configurabile).

### Permessi atomici
Stringhe granulari, assegnate a **ruoli** (insiemi di permessi, definiti da eventi `role.*`) e con **grant/deny per singolo utente** (`user.permissions`). Il deny prevale. Elenco iniziale:
`appointment.view`, `appointment.create`, `appointment.edit_own`, `appointment.edit_any`, `appointment.delete_own`, `appointment.delete_any`, `task.view_own`, `task.view_all`, `task.create`, `task.assign`, `task.edit_own`, `task.edit_any`, `task.comment`, `user.view`, `user.manage`, `role.manage`, `audit.view`, `backup.run`.

**Applicazione dei permessi in lettura:** durante il replay, ogni evento viene applicato solo se l'autore aveva il permesso **nello stato ricostruito fino a quel punto**; altrimenti viene scartato (e compare nel log diagnostico). Gli eventi `user.*`, `role.*`, `user.permissions` richiedono admin. Regole fisse: l'ultimo admin non può essere disabilitato o declassato; la firma deve corrispondere alla chiave pubblica che l'utente aveva in quel momento; un utente disabilitato non produce più eventi validi dopo la disabilitazione.

### Dominio
- **Appuntamenti:** titolo, inizio/fine (istante UTC + flag "tutto il giorno", visualizzati nel fuso locale), luogo, note, partecipanti, stato (attivo/annullato). Eventi: `appointment.create|update|cancel`. Ricorrenze: **fuori dalla v1**, da decidere con me.
- **Task:** titolo, descrizione, assegnatario, stato (da fare / in corso / completato), priorità, scadenza, commenti. Eventi: `task.create|update|assign|set_status|comment`.
- **Audit log:** nasce gratis dagli eventi (chi, cosa, quando); vista dedicata con permesso `audit.view`.

### Struttura del codice
```
src-tauri/src/
  lib.rs / main.rs
  core/
    hlc.rs        orologio logico ibrido
    event.rs      struttura, serializzazione canonica, firma e verifica
    store.rs      append, lettura incrementale, scansione della cartella
    state.rs      replay deterministico, applicazione eventi
    perms.rs      ruoli e permessi
    crypto.rs     Argon2id, XChaCha20-Poly1305, Ed25519
    config.rs     configurazione locale
    backup.rs     esportazione/backup
  commands.rs     comandi Tauri esposti alla UI
src/              UI Svelte + TypeScript
tools/stress/     test di stress multi-processo
docs/FORMATO.md   specifica del formato dati
```
**Tutto l'I/O su file avviene in Rust.** La UI non accede mai direttamente al filesystem: usa solo i comandi Tauri. Capabilities Tauri al minimo indispensabile (solo selezione cartella tramite dialog e i comandi dell'app).

### Portabilità e configurazione locale
- Al primo avvio l'app chiede la **cartella di lavoro** e la salva in una config locale (`%LOCALAPPDATA%\Bussola\config.json`). Modalità "portable vero": se accanto all'exe c'è un file marker `bussola.portable`, la config sta accanto all'exe.
- Cartella vuota → l'app propone di **inizializzare un nuovo archivio** e di creare il primo admin. Cartella già popolata → si collega e basta.
- **Dati di WebView2 in `%LOCALAPPDATA%\Bussola\webview`** (imposta `WEBVIEW2_USER_DATA_FOLDER` all'avvio, prima di creare la finestra): di default finirebbero accanto all'exe e l'app non partirebbe da una cartella non scrivibile.
- Se la share non è raggiungibile: l'app resta in sola lettura con banner ben visibile; le scritture non vanno perse (outbox locale riprovata in automatico, Fase 5).
- Backup: comando admin "esporta copia dell'archivio" in una cartella scelta (permesso `backup.run`).

---

## Fasi di sviluppo

0. **Bootstrap** (sopra).
1. **Core di sincronizzazione senza UI:** HLC, formato evento, append, lettura incrementale con tolleranza della riga finale incompleta, replay deterministico. Test unitari ed eseguibile `tools/stress` che lancia ~20 processi che scrivono e leggono insieme e verifica che tutti convergano allo stesso stato. Scrivi `docs/FORMATO.md`.
2. **Crittografia, utenti e permessi:** genesi, chiavi, login, rotazione, ruoli, permessi atomici, regole fisse, scarto degli eventi non autorizzati. Test: evento falsificato, evento di utente senza permesso, utente disabilitato, ultimo admin, replay in ordine diverso → stesso stato.
3. **Dominio:** appuntamenti e task con relativi comandi Tauri.
4. **UI Svelte (in italiano):** setup iniziale/collegamento cartella, login, calendario (giorno/settimana/mese), lista e dettaglio task, gestione utenti e ruoli, audit log, indicatore di sincronizzazione e di stato rete/share.
5. **Robustezza:** outbox locale, cache locale, backup, blocco per inattività, messaggi d'errore chiari, eventuale aggiornamento dell'app da una cartella della share (senza rete esterna).
6. **Prove su SMB reale:** share Samba in un container Docker montata da una VM Windows (UTM o Parallels), poi prova sul campo con pochi utenti.

## Limiti da tenere presenti (e da dichiarare nella documentazione)
- Chi ha accesso in scrittura alla cartella può **cancellare o corrompere** i file, anche senza poter falsificare eventi: servono **backup periodici** e permessi NTFS adeguati sulla cartella (competenza di chi gestisce il fileserver).
- I permessi sono verificati da ogni client al replay: proteggono da eventi falsificati o non autorizzati, non da chi distrugge i dati.
- Le password dimenticate non sono recuperabili.
- L'eseguibile non firmato può essere bloccato dalle policy del PC.
