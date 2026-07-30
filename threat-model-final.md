# NeoScreen Hub — Threat model di rilascio

Stato: **baseline tecnica Fase 8**. Il documento non sostituisce un penetration
test indipendente né l'autorizzazione istituzionale.

## Asset protetti

- dati anagrafici e risultati clinici nel database SQLCipher;
- DEK, MRK, device secret DPAPI e buste utente;
- configurazioni cutoff/regole e relative versioni;
- audit append-only e workflow clinici;
- Recovery Set e backup `.nshbackup`;
- token OAuth Google Drive;
- integrità dei binari e del pacchetto di release.

## Confini di fiducia

1. processo Desktop autenticato;
2. Data Root `%LOCALAPPDATA%\NeoScreenHub\Data\`;
3. profilo Windows e DPAPI CurrentUser;
4. CSV sorgente esterno in sola lettura;
5. destinazioni backup locali/rimovibili;
6. Google Drive opt-in tramite OAuth/PKCE;
7. GitHub API consultata esclusivamente su azione manuale;
8. pacchetto di installazione e canale di distribuzione.

## Minacce e mitigazioni

| Minaccia | Mitigazione principale | Rischio residuo |
|---|---|---|
| Furto del database | SQLCipher, DEK casuale, Argon2id + device secret DPAPI, MRK separata | malware attivo nella sessione sbloccata |
| PIN Technologist offline | PIN combinato con segreto dispositivo non esportabile; envelope individuale | compromissione simultanea profilo Windows e PIN |
| Alterazione Recovery Set | manifest autenticato, hash envelope, staging, quote e `integrity_check` | perdita completa senza backup separato |
| CSV malevolo | parser streaming bounded, contratti rigidi, Skip & Warn circoscritto, nessuna formula eseguita | nuovi formati richiedono nuova revisione |
| SQL injection | AST/field catalog e soli parametri ADO.NET | difetti futuri fuori dal compilatore centralizzato |
| PII nei log | logging deny-by-default, eccezioni redatte, scanner automatico Fase 8 | dump di memoria/strumenti amministrativi esterni |
| Workflow duplicati | hash/HMAC versionati, chiavi uniche, idempotenza e Unit of Work auditata | rettifiche cliniche richiedono processo umano |
| Regola clinica errata | lifecycle immutabile, Sandbox, Golden Case, attestazione medica e step-up Admin | errore nel protocollo medico approvato |
| QC fuori controllo | gate pre-commit bloccante, override nominativo permanente | rischio esplicitamente assunto dall'operatore |
| Aggiornamento manomesso | SHA-256 del pacchetto, SBOM, hook Authenticode | build non firmata non è approvabile per il pilota |
| Perdita dati durante update | binari e Data Root disgiunti, staging, precedente versione conservata, test sentinella | rollback schema richiede Recovery Set pre-update |
| Esfiltrazione cloud | Drive opt-in `drive.file`, PKCE/state, token DPAPI, backup già cifrato | metadata tecnici visibili al provider |

## Assunzioni operative obbligatorie

- BitLocker attivo sul volume;
- account Windows nominativi, blocco schermo ed EDR istituzionale;
- backup cifrati su almeno una destinazione separata;
- MRK custodita fuori dal computer e sottoposta a procedura di accesso;
- nessun utente opera con privilegi amministrativi Windows non necessari;
- firma Authenticode istituzionale prima del pilota;
- monitoraggio e revisione periodica di utenti, regole, cutoff e backup.

## Finding residui prima del pilota

1. certificato Authenticode e procedura di firma non sono forniti nel
   repository;
2. Golden Case ufficiali, UAT e firma del Dirigente Medico non sono ancora
   disponibili;
3. prova restore su macchina/profilo Windows distinto deve essere eseguita e
   verbalizzata;
4. collaudo hardware/accessibilità e pilota offline devono essere eseguiti;
5. policy istituzionali di retention, clipboard/export e gestione incidenti
   devono essere approvate;
6. timeout automatico di sessione e cambio password Admin restano backlog di
   hardening noto.

Fino alla chiusura dei finding 1–4 lo stato è **GO tecnico al packaging e alla
UAT, NO-GO al pilota clinico reale**.
