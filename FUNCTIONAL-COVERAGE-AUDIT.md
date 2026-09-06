# NeoScreen Hub — Verifica di copertura funzionale per la release 1.0.11

Data verifica: **2026-09-04**

## 1. Esito

La build tecnica `1.0.11` è riproducibile, self-contained `win-x64`, completa
del runtime .NET 10 e accompagnata da installer conservativo, checksum, SBOM,
manuale e gate automatici. Il pacchetto generato in questo workspace è un
**artifact tecnico di collaudo**, non una release autorizzata per dati sanitari
reali, perché:

- proviene da un worktree con modifiche non ancora committate;
- non è firmato Authenticode;
- la candidate clinica è ancora `VALIDATED/SHADOW_DISABLED`;
- Golden Case medici, UAT istituzionale, restore su un secondo computer e
  pilota offline non sono ancora attestati.

Il sistema resta fail-closed rispetto all'attivazione clinica: l'assenza
dell'attestazione medica impedisce alla candidate di diventare operativa.

## 2. Funzioni implementate e verificate automaticamente

| Area | Copertura verificata |
|---|---|
| Foundation desktop | .NET 10/Avalonia `win-x64`, Clean Architecture, i18n IT/EN, Medical Clean Glass, logging redatto |
| Database e sicurezza | SQLCipher Community, fixed-point INTEGER, DPAPI/MRK, Recovery Set autenticato, migrazioni fino a v16 |
| Autenticazione | Cold Unlock Admin/password e Technologist/PIN, RBAC, throttling, gestione utenti e step-up Admin |
| Ingestion primario | CSV 104 colonne Windows-1252/RFC 4180, streaming read-only, Skip & Warn, peso, identità HMAC v2, idempotenza e 53 rapporti |
| Query | AST parametrizzato, paginazione keyset, ricerca paziente, timeline, filtri salvati e benchmark da un milione di righe |
| QC | classificazione LOW/HIGH, kit versionati, import JSON/XLSX/manuale, Mean ±3 SD, trend e gate pre-commit |
| Backup/restore | `.nshbackup` bounded, provider locale e infrastruttura Drive OAuth/PKCE, restore in staging, audit e quote |
| Second Tier | parser MMA/IVA/HCY/MSUD, matching paziente, code I/R/C, interprete phase-aware e casi fail-closed |
| Motore clinico | AST bounded, tri-state, release prospective-only, shadow evaluation, routing Standard/2TT/Misto, TPN, Golden Runner e Visual Rule Builder |
| Workflow | Ciclo del Rosso, Retest, Richiami, auto-routing Reflex 2TT, azioni Daily Review in RowDetails frozen, CheckIn in code/TPN/storia, code/aging, visual evidence, refertazione e clipboard audit |
| Release engineering | restore locked, dipendenze FOSS, build/test/format, privacy gate, publish self-contained, SBOM, checksum e prova install→update→uninstall |

## 3. Funzioni software ancora incomplete

Questi elementi sono nel piano ma non sono conclusi e non devono essere
descritti come completati:

1. **Registro Patologici 7B source-aware completo.** La pagina base è reale,
   paginata, filtrabile per periodo ed esportabile in Excel con gate Admin;
   mancano il dettaglio tipizzato Richiamo/2TT/TPN, evidenze e timeline
   completa, eventi compensativi e i relativi test anti-leak/performance.
2. **Gestione Admin completa dei cutoff e delle formule.** Schema, sigilli,
   candidate clinica e lettura dei cutoff sono presenti; manca il workflow UI
   generale di import/diff/revisione `DRAFT → VALIDATED → ACTIVE → RETIRED`
   equivalente al Rule Builder.
3. **Hardening sessione.** Il lock manuale e lo step-up esistono; timeout
   automatico di inattività/durata massima e cambio password Admin non sono
   ancora implementati.
4. **Mapper visuale centralizzato.** I colori clinici sono operativi e
   accessibili, ma la conversione disposizione→token è ancora distribuita nei
   ViewModel/stili anziché essere un singolo mapper tipizzato.
5. **Pseudo-localizzazione e collaudo accessibilità su hardware reale.**
   Parità IT/EN e scanner sono automatici; restano high contrast, scaling,
   reduced motion, screen reader e testi espansi sul parco Windows del
   laboratorio.
6. **Controlli opzionali di coerenza dei rapporti.** Il calcolo autoritativo e
   gli stati `NOT_COMPUTABLE` sono presenti; eventuali confronti diagnostici
   aggiuntivi fra rapporti correlati non sono stati definiti né implementati.

## 4. Decisioni cliniche/operative ancora richieste

- Golden Case ufficiali firmati e attestazione del Dirigente Medico;
- UAT clinica su cutoff inclusivi, fasce d'età, multi-regola, TPN, Richiami e
  Second Tier;
- limiti/sentinel definitivi per età e altri campi essenziali;
- decisione se la correzione del peso debba includere valori mancanti o
  sintatticamente non interi;
- workflow di rettifica di un evento già acquisito che ricompare con stessa
  chiave ma contenuto differente;
- policy istituzionali per retention di backup/audit, clipboard, export e
  destinazioni protette.

## 5. Procedure pratiche prima del pilota

1. chiudere o accettare formalmente i backlog software della sezione 3;
2. eseguire commit revisionato, tag `v1.0.11` e build da worktree pulito;
3. procurare certificato Authenticode istituzionale e timestamp authority;
4. eseguire `New-Release.ps1 -RequireSignature`;
5. pubblicare ZIP, `.sha256`, SBOM e note di migrazione sulla GitHub Release;
6. provare backup e restore con MRK su un secondo PC/profilo Windows;
7. collaudare Google Drive con Client ID, cartella e consenso istituzionali;
8. eseguire UAT medica e conservare l'attestazione;
9. eseguire pilota offline su hardware reale, chiudere i difetti critici/alti
   e autorizzare formalmente l'uso con dati reali.

Fino alla chiusura di questi gate, il verdetto è:
**GO per collaudo tecnico controllato; NO-GO per uso clinico reale.**
