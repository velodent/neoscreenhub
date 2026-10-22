# NeoScreen Hub

NeoScreen Hub è un'applicazione desktop locale per Windows dedicata all'importazione, alla conservazione cifrata, alla ricerca e all'analisi di dati provenienti dallo screening neonatale.

Il progetto nasce per offrire un hub professionale, rapido e verificabile per CSV giornalieri a struttura costante. Elaborazione e database operativo restano locali; l'unica integrazione cloud prevista è il backup opzionale di Recovery Set già cifrati e autorizzati. L'applicazione non sostituisce il giudizio clinico: applica regole configurate e tracciabili per facilitare l'individuazione e la revisione dei risultati.

La versione applicativa corrente è **1.3.0**, schema Database **v19**.

La bonifica P1 di ottobre è descritta in [ADR-067](docs/adr/067-october-remediation-operational-safety.md). I risultati dei test e i collaudi ancora necessari sono distinti nel [rapporto di remediation](docs/security/october-2026-remediation-1.3.0.md): una build tecnica non equivale a un'approvazione clinica.

## Stato del progetto

- **Fase 0A — Fondazioni architetturali:** completata.
- **Fase 0B — Specificazione clinica e operativa enterprise:** completata. Il protocollo cutoff ufficiale v2 usa Fascia A `0..360 h` e Fascia B `>360 h`, quattro fasi operative e Numeric Policy v2 a tre decimali; formula `Acs/Cit` e baseline DBS QC sono approvate da ADR-061/ADR-062. Le 24 regole cliniche primarie, con routing Standard o Reflex 2TT, sono consolidate in `regole_patologie.md` e diventano operative soltanto mediante una release clinica revisionata e attivata.
- **Fase 1 — Fondazioni tecniche:** completata il 2026-07-20; runtime e branding stabilizzati e riverificati il 2026-07-21.
- **Fase 2 — Database cifrato:** completata, bonificata e security-accepted il 2026-07-21 con SQLCipher Community, doppio envelope DPAPI/MRK, schema v2 `STRICT`, HMAC identità v2 e Unit of Work auditata.
- **Fase 2B — Backup & Recovery:** nucleo completato, bonificato e security-accepted il 2026-07-21: manifest attivo v2 vincolato all'envelope, `.nshbackup` bounded, restore/migrazione in staging e Google Drive OAuth/PKCE fail-closed. Il collaudo contro il tenant Drive istituzionale richiede Client ID, cartella e consenso reali, mai conservati nel repository.
- **Fase 3 — Ingestion Engine:** completata e verificata il 2026-07-21 con 112/112 test Release; parser streaming Windows-1252/RFC 4180, Skip & Warn, correzione peso, identità HMAC v2, deduplicazione, episodi A/B, 53 rapporti materializzati e pagina Importa sono operativi sul vault SQLCipher schema v3.
- **Fase 4 — Query, filtri e nucleo Operatori:** completata e verificata il 2026-07-21. Lo schema SQLCipher v4 rende persistenti operatore/sessione corrente e filtri salvati; Ricerca Paziente, Archivio, AST AND/OR parametrizzato, paginazione keyset e timeline con Media A/B sono collegati ai servizi reali. Il benchmark cifrato da 1.000.000 record ha misurato `P50 14,55 ms` e `P95 21,70 ms` sulla prima pagina da 100 righe.
- **Fase 5 — Archivio e UI High-End:** implementazione completata il 2026-07-22. Oltre a Overview, Filter Builder, DataGrid virtualizzata e drawer Acrylic, sono operative le pagine Controlli Qualità, Backup e Registro Patologici con export Excel protetto. Il gate automatico delle transizioni UI resta `P95 < 100 ms`; collaudo visuale/accessibilità sul parco Windows del laboratorio e approvazione UAT restano attività di rilascio.
- **Fase 6A — Infrastruttura Second Tier Tests:** completata. I quattro parser MMA/IVA/HCY/MSUD usano ora Numeric Policy v2 a tre decimali, matching HMAC/fallback `Code + DateOfBirth` rivalidato nella transazione di commit, richieste indipendenti per tipo, QNS, record tecnici MSUD, risultati spontanei, idempotenza HMAC v2, quinta coda e sezione 2TT nel drawer.
- **Motore clinico e attivazione istituzionale:** le specifiche Q1–Q10, la matrice Reflex/TPN e le fasi 6B.1–6B.8 sono implementate. Lo schema SQLCipher v19 comprende Numeric Policy v2, il protocollo cutoff phase-aware e la correzione versionata della scala QC: la candidate `NEOSCREEN-CLINICAL-RC2` contiene 460 righe soglia per 115 combinazioni campo/fascia, equivalenti a 128 cutoff sorgente per fase (77 analiti + 51 rapporti) e 127 boundary runtime dopo la riconciliazione dell’unico duplicato semantico concordante. L’Admin e il Dirigente Medico possono visionare integralmente cutoff e 24 regole, clonare una bozza prospective-only, modificarla ed eseguire i controlli interni. Nome, qualifica e approvazione esplicita vengono acquisiti nell’app; Config-Hash, verbale definitivo, attestazione append-only e attivazione sono sigillati in un’unica transazione dopo lo step-up Admin. Non sono richiesti JSON o PowerShell. L’approvazione reale e la UAT istituzionale restano obbligatorie prima dell’uso clinico. Dettagli: [RFC regole dinamiche](docs/rfc/dynamic-clinical-rules.md) e [ADR-065](docs/adr/065-clinical-release-candidate-and-golden-uat.md).
- **Fase 7 — Workflow e refertazione:** implementazione tecnica completata e
  verificata il 2026-07-28 sul vault v14: gate QC pre-commit, evidenze/colori,
  Ciclo del Rosso persistente, TPN, code/aging, Richiami e diagnosi/clipboard
  audit sono operativi. La pagina Patologici source-aware completa resta il
  perimetro separato 7B.
- **Fase 8 — Hardening e packaging:** toolchain tecnica completata il
  2026-07-28: publish self-contained, pacchetto installabile per utente, SBOM,
  SHA-256, scanner privacy e prova automatica di preservazione del Data Root.
  Il responsabile ha autorizzato il pilota senza Authenticode, accettando
  proceduralmente SmartScreen (ADR-067). Restore su una seconda postazione,
  UAT medica e pilota offline restano collaudi esterni; non si dichiara
  l'artifact corrente una release clinica approvata.
- **Gate sicurezza Fase 2/2B:** chiuso con GO dopo 107 test e due contro-audit indipendenti; tutte le scritture della Fase 3 passano dalla Unit of Work auditata approvata.
- La baseline ufficiale `screening-sample-v1` ha 104 colonne, con `DateOfBirth` in posizione 104 e senza identificativo paziente esterno.

## Stack tecnologico approvato

- .NET SDK `10.0.302`, target `net10.0-windows10.0.19041.0` e C# 14;
- Avalonia `12.1.0` con backend espliciti Win32, Skia e HarfBuzz per l'interfaccia desktop nativa;
- SQLCipher Community `4.17.0`, compilato `win-x64` con OpenSSL `3.6.3` statico e caricato tramite `SQLitePCLRaw.provider.dynamic_cdecl` `3.0.3`;
- Windows DPAPI per la protezione trasparente della chiave operativa;
- Master Recovery Key per il recupero controllato del database;
- Google Drive API v3 opzionale per backup/restore cifrato, con OAuth 2.0 Desktop + PKCE e token protetti da DPAPI;
- target esclusivo Windows 10/11 `win-x64`.

Tutto lo stack deve rimanere Open Source/Free. SDK e dipendenze sono bloccati esattamente tramite `global.json`, Central Package Management e cinque `packages.lock.json`; il grafo corrente comprende 54 pacchetti FOSS diretti/transitivi verificati dall'allowlist secondo [AGENTS.md](AGENTS.md).

## Funzioni implementate e pianificate

- importazione CSV tramite selezione file o drag & drop;
- lettura in streaming e sola lettura, senza copiare, modificare o eliminare il file originale;
- validazione di schema, tipi, dominio e coerenza;
- importazione incrementale **Skip & Warn**: le righe Patient incomplete/corrotte vengono scartate con avviso, mentre le altre righe valide proseguono e sono committed insieme in modo transazionale;
- controllo di plausibilità del peso Patient nell'intervallo inclusivo 400–7000 g, con correzione manuale auditata o scarto esplicito della sola riga;
- idempotenza del file tramite SHA-256 e degli eventi Patient/QC tramite HMAC canonici: i duplicati sono segnalati e mostrati dai dati già presenti, senza creare nuovi test o workflow;
- classificazione automatica di campioni paziente, controlli qualità e blank/internal standard;
- conservazione individuale di tutti i test reali per l'audit trail;
- generazione casuale di `Internal_Patient_ID` opachi e immutabili; candidate matching sulla tripla Nome + Cognome + Data di Nascita e identità definitiva univoca sulla quadrupla che include `Mother_Name`;
- storia longitudinale e associazione dei test tramite `Internal_Patient_ID`;
- Giornata Operativa determinata dalla data di importazione del CSV;
- identificazione deterministica degli episodi A/B tramite stesso Import Batch + `patient_id` risolto + `Barcode`, con media limitata ai due record del medesimo CSV;
- Ratio Engine dichiarativo come unica fonte clinica: tutti i rapporti vengono calcolati e materializzati dal software; quelli pre-calcolati dal macchinario sono ignorati;
- importazione e versionamento dei cutoff ufficiali descritti in [cutoff.md](cutoff.md): 128 limiti sorgente per fase, quattro fasi operative, due fasce d’età e unità esclusiva `µmol/L` per gli analiti;
- `WARNING` universale quando qualunque analita o rapporto configurato raggiunge/supera un limite superiore (`>=`) oppure raggiunge/scende sotto un limite inferiore (`<=`), anche se quel campo non appartiene a una regola patologica; una regola clinica approvata composta da 1–6 evidenze cutoff produce invece `STANDARD_ALERT` con routing Standard oppure `SECOND_TIER_REFLEX` con routing 2TT;
- workflow Medical-to-Lab “Ciclo del Rosso” con code `Errati`, `Retest Singolo`, `Retest Doppio`, `Richiami` e `Second Tier Test`;
- ingestion specialistica dei quattro contratti 2TT MMA/IVA/HCY/MSUD, con `Code` come Barcode primario, matching HMAC o fallback univoco Barcode+Data di Nascita, risultati spontanei in revisione e separazione dei record tecnici `Role = 2`;
- specifica clinica 2TT phase-aware a token esatti: MMA/HCY progrediscono `I→R→C`, IVA/MSUD positivi compatibili conducono direttamente al Registro Patologici, token inattesi e spontanei restano fail-closed; matching per sottostringa e Media A/B sono vietati;
- matching automatico dei nuovi CSV, doppio spontaneo e decisioni operative auditabili;
- promozione automatica da Retest Singolo a Retest Doppio quando i dati reali contengono A/B;
- Richiamo con nuovo barcode in doppio, esito determinato dalla Media e Registro Patologici;
- refertazione rapida editabile soltanto dopo un doppio retest ancora rosso, con copia negli appunti;
- anomalie tecniche di integrità e duplicazione separate dagli stati clinici;
- catalogo controllato di 124 campi filtrabili: 11 dati record/anagrafici, 60 analiti e 53 rapporti software autoritativi;
- filtri multi-parametro come AST tipizzato con gruppi AND/OR e operatori `=`, `≠`, `>`, `≥`, `<`, `≤`, `Tra`, `Contiene` e null; i valori arrivano a SQLCipher soltanto come parametri ADO.NET;
- query asincrone annullabili, ordinamento stabile e paginazione keyset lato database senza caricare l'archivio completo in memoria;
- filtri Archivio salvabili per operatore, versionati e auditati;
- ricerca paziente per ID interno, Nome/Cognome o Nome Madre, disambiguazione omonimi e timeline oldest-first con test individuali e Media A/B materializzata;
- operatori locali reali con predefinito opzionale, sessione corrente persistente, operatore visibile nell'Header e Audit Log attribuito tramite identità stabile; selezionare un operatore sblocca la conferma dell'importazione preparata;
- backup manuale locale/rimovibile, backup automatico Google Drive, registro esiti e restore verificato in staging;
- shell Medical Clean Glass compatta con sidebar ridotta, header operativo monoriga, logo 80×80, pagina separata Controlli qualità e cinque contatori Errati/Retest/Richiami/Second Tier con aging;
- avvio della finestra principale massimizzato, mantenendo barra del titolo, taskbar e controlli Windows;
- pagina Impostazioni con nome e versione applicativa, scelta lingua e controllo manuale della release GitHub più recente;
- attivazione tramite `Enter/Invio` della singola azione primaria associata ai campi Password, PIN e testo; nei pannelli ambigui, nei campi multilinea e nelle griglie l'azione resta esplicita;
- i18n completo senza testi hardcoded: italiano predefinito e inglese secondario.

## Principi del dominio dati

### Classificazione dei record

L'ordine delle regole è deterministico:

1. se `Barcode` contiene `IS`, il record è `BLANK_INTERNAL_STANDARD`;
2. altrimenti, se `Barcode` termina con `-1` o `-2`, è `QUALITY_CONTROL`;
3. negli altri casi è `PATIENT_SAMPLE`.

QC e blank sono esclusi integralmente dalle viste paziente. Sono consultabili soltanto nella sezione dedicata **Controlli qualità**, con tabella, filtri e grafici di andamento. Per i soli `QUALITY_CONTROL`, `-1` identifica il livello LOW, `-2` il livello HIGH e il barcode normalizzato privato del suffisso identifica stabilmente il kit fisico per i suoi 2–3 mesi di utilizzo. La baseline certificata `QC_DBS_BASELINE_V1` (matrice DBS, `µmol/L`) considera un valore `IN_CONTROL` nell'intervallo inclusivo Mean ± 3 SD. Un nuovo barcode kit produce `QC_REFERENCE_REQUIRED` finché l'operatore non registra il relativo set; non esistono fallback per data o “set più recente”. Il Gestore kit consente import `.xlsx`, import JSON dichiarativo `nsh-qc-reference-v1` e inserimento/modifica manuale nello stesso draft, mappando gli analiti per codice e non per ordine. Blank/Internal Standard non eredita questi range.

### Identità, ripetizioni e medie

Il `Barcode` identifica il campione fisico, ma non la persona. Il CSV non fornisce un ID paziente: l'app usa la tripla normalizzata Nome + Cognome + Data di Nascita come ricerca preliminare HMAC e genera un `Internal_Patient_ID` casuale, opaco e immutabile. La quadrupla Nome + Cognome + Data di Nascita + Nome Madre è la chiave anagrafica definitiva; un match esatto identifica lo stesso paziente, perché l'omonimia perfetta sui quattro campi è dichiarata impossibile.

Alla prima corrispondenza la preview propone il paziente esistente. Se la tripla corrisponde a più pazienti, il commit della riga richiede una scelta esplicita usando `Mother_Name`; prima di creare il secondo membro della stessa tripla, il nome della madre deve essere valorizzato per tutti i pazienti del gruppo e ogni quadrupla deve essere completa e univoca. `Mother_Name` resta nullable finché non serve a disambiguare, può essere inserito o modificato con audit e non modifica l'ID stabile. Una tripla incompleta non viene conservata come identità irrisolta: la sola riga viene scartata con avviso.

- Ogni test distinto viene salvato come riga autonoma.
- Il reimport dello stesso file, riconosciuto dal SHA-256, mostra “File già importato” e ricarica dal database i risultati del batch originario, senza creare record o workflow.
- Una riga Patient ricomparsa in un nuovo file con lo stesso paziente risolto, `Barcode` e `CheckIn`, oppure una riga tecnica con stessi `Barcode`, `Well` e data tecnica effettiva, viene segnalata come già analizzata e mostrata tramite il record canonico esistente, senza duplicarlo. Per QC/Blank la data effettiva è `Data CheckIn` quando presente, altrimenti la Giornata Operativa; il fallback è tracciato ma non produce warning. Il record riusato conserva la Giornata Operativa originaria e non incrementa gli esami del giorno corrente.
- Due o più test distinti dello stesso paziente restano visibili individualmente.
- Due righe formano un episodio A/B soltanto quando provengono dallo stesso file/Import Batch e coincidono `patient_id` risolto e `Barcode` normalizzato. La media riguarda esattamente quei due record, resta separata dai test originali e non fonde mai batch diversi.
- Lo stesso paziente/barcode in file differenti rappresenta test sequenziali e non viene mai mediato, anche se `Data CheckIn` coincide. La data resta un metadato clinico/temporale, non una chiave dell'episodio.
- La Giornata Operativa è la data locale del commit riuscito dell'importazione; è indipendente da `Data CheckIn`.

### Stato clinico e integrità tecnica

Cutoff e regole cliniche non sono incorporati nel codice. Il motore produce `WARNING` quando un valore raggiunge il limite superiore (`>=`) o quello inferiore (`<=`) attivo: l'uguaglianza è sempre gialla. Una regola versionata composta da una a sei evidenze cutoff produce `STANDARD_ALERT` oppure `SECOND_TIER_REFLEX` in base al routing. Il solo Reflex non è un Alert rosso; un Alert standard è un sospetto da revisionare, non una diagnosi automatica.

Tutti i rapporti clinici sono divisioni dichiarative calcolate una volta all'import dagli analiti e materializzate con versione `CALCULATED_BY_NEOSCREEN`. Alias e canali compositi sono definiti in [cutoff.md](cutoff.md); i rapporti macchina sono `MACHINE_RAW_IGNORED` e non possono alimentare UI, filtri o motori clinici. La Media A/B calcola il rapporto A, il rapporto B e poi la loro media; calcolo, persistenza e visualizzazione usano tre decimali secondo `numeric-policy-v2` (INTEGER ×1000). Un valore non calcolabile resta tecnico/`NOT_COMPUTABLE`, mai zero o fallback.

Per i Patient, un dato essenziale mancante o corrotto scarta soltanto la riga e produce il messaggio localizzato “Probabile lettura errata del codice a barre alla riga [X] (CheckIn [Y])”; le altre righe valide continuano. Un peso intero fuori dal range inclusivo 400–7000 g sospende la sola riga e apre una correzione manuale: l'operatore inserisce un valore valido oppure sceglie di scartarla. Sorgente, correzione e attore restano tracciati nel database cifrato, ma il CSV originale non viene modificato. Stato tecnico e stato clinico rimangono dimensioni indipendenti.

### Workflow Medical-to-Lab: il Ciclo del Rosso

Un `STANDARD_ALERT` isolato, oppure il ramo standard di un caso `MIXED_STANDARD_AND_SECOND_TIER`, entra al primo riscontro nella finestra **Sospetti Patologia**, senza mostrare ancora alcuna diagnosi. L'operatore deve scegliere una sola delle tre azioni; un caso `SECOND_TIER_ONLY` segue invece l'apertura automatica descritta più avanti e non presenta queste scelte:

- **Errato:** campione inadeguato; la notifica resta attiva finché un import successivo porta un barcode nuovo per lo stesso paziente;
- **Retest Singolo:** la notifica si risolve quando lo stesso barcode ricompare per lo stesso paziente in un CSV successivo;
- **Retest Doppio:** la notifica si risolve quando un singolo CSV successivo contiene due record distinti con lo stesso `patient_id` e lo stesso `Barcode`.

Se il retest non contiene né `STANDARD_ALERT` né `SECOND_TIER_REFLEX`, il record resta nella tabella principale e la cella `CheckIn` diventa verde per indicare il successo operativo; eventuali evidenze Warning inclusive restano gialle. Un esito Solo 2TT resta invece grigio e apre la richiesta specialistica. Se un Retest Singolo torna con Standard Alert, rientra nella finestra Sospetti e riparte dalle tre scelte; se è Misto, resta arancione e apre anche i 2TT. Se un singolo nuovo CSV contiene due righe dello stesso paziente/barcode, il Singolo viene promosso automaticamente a Doppio. Se un Retest Doppio torna con Standard Alert, la vista mostra A, B, Media e il campo Diagnosi: l'operatore sceglie **Richiamo**, **Retest Doppio** oppure **Errato**, mentre eventuali lineage 2TT restano indipendenti.

Un doppio rilevato senza alcuna notifica pendente è trattato automaticamente come Retest Doppio spontaneo. Le code `Errati`, `Retest Singolo`, `Retest Doppio`, `Richiami` e `Second Tier Test` sono ordinate dalla più vecchia alla più recente; il click apre la storia completa del paziente. La rimozione da una coda è soltanto logica: l'audit append-only conserva ogni passaggio.

Quando una regola dichiara `SECOND_TIER_REQUIRED(type)` senza Alert standard concomitanti, il record resta nella finestra superiore in grigio e la richiesta MMA/IVA/HCY/MSUD viene aperta automaticamente. Se coesiste un Alert standard, il record è arancione nella finestra inferiore e vengono creati atomicamente sia la revisione standard sia le richieste 2TT; i rami restano indipendenti. La richiesta viene soddisfatta soltanto da un risultato dello stesso paziente, tipo e fase attesa; richieste di tipi differenti possono coesistere. Tutti i 2TT sono singoli. MMA/HCY positivi avanzano automaticamente da Initial a Retest e poi Recall, confermando soltanto al positivo Recall; IVA/MSUD positivi compatibili confermano direttamente. La quinta coda mostra tipo e fase e include tutte le attese I/R/C. Un risultato senza richiesta resta Test Extra/Review senza automatismi. `QNS` HCY chiude l'attesa ma resta anomalia tecnica, mentre i sentinel MSUD dei record `Role = 2` restano controlli tecnici e non entrano nel workflow paziente.

Nel primario, `TPN = Sì` prevale: gli analiti alterati e il CheckIn sono viola, le evidenze cutoff restano visibili ma regole e richieste automatiche sono sospese. Il primo riscontro offre **Nessuna Azione** o **Retest Doppio**; se il doppio è ancora TPN, l'operatore sceglie **Normale** oppure **Patologico**, seleziona una patologia censita e conferma una diagnosi editabile prima dell'inserimento nel Registro.

La coda **Richiami** è l'eccezione operativa autorizzata nella quale Nome, Cognome e Data di Nascita sono mostrati in chiaro. Si chiude quando un CSV successivo contiene lo stesso paziente, un barcode nuovo e un doppio completo. A, B e Media restano accoppiati anche quando discordanti:

- Media rossa → finestra Rossi, nessuna diagnosi automatica e registrazione idempotente nel `pathological_registry`;
- Media senza Standard Alert → scelta **Normale** oppure **Ripetere**; il CheckIn è verde soltanto senza Reflex, altrimenti resta grigio con 2TT pendente;
- **Normale** chiude definitivamente; **Ripetere** riapre Richiami e richiede un ulteriore barcode nuovo in doppio.

Il testo diagnostico è disponibile solo nella revisione di un doppio interno ancora rosso, è editabile e copiabile, e usa il formato:

```text
Patologia: [Nome] | Analiti: [Valori] | Nota: [Testo della regola]
```

Quando sono soddisfatte più patologie, ciascuna produce un blocco nello stesso campo, separato chiaramente dagli altri; un solo pulsante copia l'intero contenuto.

## Operatori, ricerca e navigazione

La Fase 2C collega ogni operatore a un account locale autenticato. Admin usa una password di almeno 8 caratteri; i Technologist usano un PIN di 4 cifre. Entrambe le credenziali eseguono il Cold Unlock della DEK tramite Argon2id, una busta personale AES-256-GCM e un segreto dispositivo protetto da DPAPI. Il nome autenticato resta visibile nell'Header; Cambia operatore blocca il Database e torna al Login. Creazione, disabilitazione, reset PIN, login operativo e scritture di dominio mantengono ID stabile, sessione e audit.

La Sidebar, ridotta del 15% rispetto alla prima Foundation, comprende **Overview**, **Importa**, **Ricerca Paziente**, **Archivio**, **Sospetti Patologia**, **Code Operative**, **Patologici**, **Controlli qualità**, **Regole Cliniche**, **Backup**, **Operatori**, **Audit** e **Impostazioni**. Il logo 80×80 occupa il quadrante superiore sinistro all'intersezione con l'Header.

La **Overview** interroga il vault e mostra Importati oggi, Batch, anomalie tecniche e pazienti archiviati senza simulare dati clinici non disponibili. L'**Archivio** usa una DataGrid virtualizzata con paginazione keyset, colonne selezionabili/riordinabili/ridimensionabili, filtro visuale AND/OR rappresentato da chip e drawer laterale Acrylic. Il drawer mantiene test individuali e Media A/B distinti e mostra lo snapshot dell'operatore quando disponibile. Skeleton, empty state ed error state sono localizzati; nessuno stato — inclusi verde, giallo, rosso, grigio, arancione, viola TPN e indaco tecnico — è mai comunicato dal solo colore.

Dopo un import primario committed, la pagina **Importa** mantiene consultabile la Revisione Risultati dell'ultimo batch anche se l'utente naviga altrove; lo stato viene sostituito soltanto all'avvio effettivo di un nuovo CSV. Le due finestre usano una griglia analitica Wide con colonne iniziali congelate, catalogo completo di analiti e rapporti software e di ogni ulteriore campo dotato di cutoff attivo, righe dati compatte da 44 px e scrollbar orizzontale e verticale da 16 px sempre visibili. Il doppio righello mostra sulla prima riga la fase `NEW_TEST` per Fascia A `0..360 h` e sulla seconda le sole differenze per Fascia B `>360 h`; il suo `Offset.X` scorre bidirezionalmente, al pixel, con il `PART_HorizontalScrollbar` della DataGrid anche in Focus. Il tooltip di ogni valore espone fase e limite realmente applicati al record. Uno splitter realmente collegato alle due righe ridimensionabili distribuisce liberamente lo spazio verticale; ogni finestra dispone inoltre della modalità Focus totale. La singola cella viene colorata tramite l'evidenza cutoff persistita: una violazione isolata rimane gialla anche quando il campo non compare nelle 24 regole; il CheckIn applica la priorità operativa Rosso > Grigio > Giallo. Nel Riquadro Rossi la riga principale contiene soltanto dati clinici; ogni caso azionabile espone i comandi nel `RowDetails` congelato. La stessa griglia è presente nelle Revisioni TPN. Il Drawer paziente contiene inoltre la storia analitica completa.

La pagina **Audit** è una scatola nera legale read-only: elenca sessioni/login ed eventi auditati con timestamp, operatore, azione, entità ed esito, senza riportare PII cliniche. Il banner interroga lo stato reale nel Database: una release `ACTIVE` mostra **Database Operativo — Protezione Clinica Attiva**; in sua assenza chiarisce che lo storico corrente riguarda soltanto operazioni tecniche e amministrative.

**Controlli qualità** è separata dall'archivio pazienti. Mostra QC e Blank/Internal Standard, consente la selezione dell'intervallo temporale e dell'analita e visualizza l'andamento in tabella e grafico. Il Gestore kit elenca barcode rilevati, stato configurazione e revisioni dei riferimenti; accetta template `.xlsx`, JSON `nsh-qc-reference-v1` oppure compilazione manuale e consente correzioni mirate nella medesima anteprima prima dell'attivazione auditata. I soli QC vengono confrontati con la revisione ADR-062 associata esattamente al barcode base del kit, conservando Mean, SD, limiti e versione realmente usati; lo stato tecnico resta separato da Warning/Alert Patient.

Il **Registro Patologici** è una vista read-only paginata e filtrabile per periodo. L'export Excel è riservato all'Admin, richiede conferma esplicita della destinazione protetta ed è auditato; finché la Fase 7 non completa il Ciclo del Rosso, la pagina mostra correttamente lo stato vuoto senza dati simulati.

Ricerca Paziente accetta `Internal_Patient_ID`, Nome/Cognome e, come dato ausiliario, Nome della Madre. Gli omonimi non vengono mai uniti automaticamente: l'utente sceglie dalla lista disambiguata tramite ID interno, Data di Nascita, `Mother_Name` e ultimo test. Scheda e ricerca consentono di inserire/modificare `Mother_Name` con audit anche in assenza di omonimia. Il dettaglio mostra dalla data più vecchia alla più recente test individuali, episodi A/B, Medie, valutazioni, esiti, transizioni e chi ha convalidato ogni passaggio. Patologici consente filtri per anno specifico o intervallo della data di conferma.

L'Header è una sola riga compatta con cinque pulsanti glass per **Errati**, **Retest Singolo**, **Retest Doppio**, **Richiami** e **Second Tier Test**: ogni pulsante mostra icona, label e numero. A destra restano distinti il nome dell'operatore e il pulsante **Cambia operatore**. Lo stato importazione è contestuale alla sola pagina **Importa**. I numeri Retest e Second Tier diventano rossi se esiste una voce pendente da più di 48 ore e il numero Richiami oltre 5 giorni; il rosso indica anzianità operativa, non severità clinica. La coda 2TT viene aggiornata all'apertura, dopo ogni commit 2TT e automaticamente ogni minuto, interrompendo il ciclo al logout/cambio schermata. La scelta lingua `it-IT/en` è nella pagina **Impostazioni**, non nell'Header, e resta applicabile a runtime.

La finestra principale parte massimizzata, senza trasformarsi in una modalità kiosk. La pagina **Impostazioni** mostra **NeoScreen Hub 1.3.0** e offre un controllo aggiornamenti avviato soltanto dall'utente. La richiesta consulta esclusivamente l'endpoint pubblico HTTPS della release GitHub più recente. Come ogni connessione HTTPS, comunica a GitHub l'indirizzo di rete e i metadati tecnici standard, inclusa la versione dell'applicazione; NeoScreen Hub non invia PII, dati clinici, operatore o identificativi del Database e non scarica né installa automaticamente alcun file.

## Backup e ripristino

Il backup produce un unico artifact cifrato e verificabile contenente il Recovery Set; non esporta CSV o dati sanitari in chiaro e non include MRK, DEK o token OAuth. Sono disponibili:

- backup manuale verso cartella locale o supporto rimovibile;
- backup Google Drive opt-in verso una cartella condivisa autorizzata;
- trigger automatici dopo import committed, prima delle migrazioni e giornalieri se il database è cambiato;
- registro cronologico di esito, destinazione tecnica, hash/dimensione e attore.

Il link Google Drive identifica la cartella ma non concede accesso: l'utente deve autorizzare nel browser un account con i permessi necessari. I token vengono protetti da DPAPI e devono essere riacquisiti dopo reinstallazione/cambio profilo quando non più utilizzabili. La MRK serve a decifrare il Recovery Set, non sostituisce OAuth e non viene caricata nel cloud.

L'Admin registra il Client ID OAuth Desktop istituzionale nella sezione **Integrazione Google Cloud** della pagina Impostazioni; il valore viene validato, salvato atomicamente come configurazione pubblica di deployment e applicato subito al servizio Backup. La variabile `NEOSCREENHUB_GOOGLE_CLIENT_ID` resta un fallback tecnico per deployment amministrati. Dopo il salvataggio del Client ID e di un link cartella valido, **Collega Google Drive** abilita il consenso OAuth. Se la configurazione manca o OAuth/Drive rifiutano una fase, la pagina mostra un messaggio operativo localizzato e un codice diagnostico invariabile; i log conservano soltanto quel codice allowlisted, mai token, link cartella o dati sanitari.

La pagina Backup accetta un link HTTPS `drive.google.com/drive/folders/...` oppure il solo ID della cartella, ne persiste esclusivamente l'identificatore tecnico e avvia OAuth Desktop con PKCE. Mostra inoltre il comando **Esegui backup ora**, il backup locale tramite selettore cartella e lo storico degli esiti con testo e colore accessibile. Il Client ID OAuth istituzionale rimane configurazione di deployment, mai un segreto incorporato nei dati o nel repository.

Al primo avvio o dopo reinstallazione la schermata iniziale offre **Nuova Installazione** oppure **Ripristino da Backup**. Per Drive il restore richiede link cartella, autorizzazione Google e MRK; il sistema individua il backup completo e compatibile più recente, lo scarica in staging, verifica manifest, hash, envelope e SQLCipher, quindi lo promuove atomicamente e rigenera l'envelope DPAPI. Un Data Root esistente non viene mai sovrascritto direttamente.

## Internazionalizzazione

La Foundation implementa risorse `.resx`, italiano `it-IT` predefinito, inglese `en`, fallback deterministico e cambio lingua a runtime sulle viste aperte. La sola preferenza non sensibile del codice cultura viene salvata atomicamente in `%LOCALAPPDATA%\NeoScreenHub\Settings\ui-culture-v1.txt` e ripristinata al riavvio; un file assente, corrotto o non supportato ricade su `it-IT`. Tutti i testi user-facing, inclusi accessibilità, errori e futuri template clinici, devono continuare a passare dal servizio i18n. Codici di dominio, parsing CSV, HMAC e dati persistiti restano indipendenti dalla lingua selezionata.

## Sicurezza e custodia delle chiavi

Al primo avvio l'applicazione:

1. richiede la password dell'account Admin;
2. genera la DEK del database e la Master Recovery Key;
3. mostra la MRK una sola volta e ne richiede la reimmissione;
4. crea il Recovery Set SQLCipher in staging;
5. genera il segreto dispositivo DPAPI e la busta Admin;
6. finalizza il database solo dopo il round-trip di MRK, busta locale e integrity check.

Il database SQLCipher è la fonte di verità persistente dell'applicazione. Dati sanitari, chiavi e valori identificativi non devono essere esposti nei log o in fixture versionate; i backup esterni restano copie cifrate e autenticate del Recovery Set, non repository operativi condivisi. Il provider di logging deny-by-default elimina categoria, messaggio, stato, identificativo arbitrario e contenuto delle eccezioni; l'API strutturata accetta soltanto enum e factory tipizzate con range controllati. Gli errori non gestiti di Dispatcher, AppDomain e Task vengono tradotti in codici invarianti senza messaggio, stack o oggetto eccezione. Le future scritture di dominio devono usare la Unit of Work SQLCipher: mutazione, audit e outbox condividono la stessa transazione e le primitive capaci di aprire direttamente il vault sono interne e protette dal gate architetturale.

I dati sono separati dai binari: il Data Root predefinito è `%LOCALAPPDATA%\NeoScreenHub\Data\`, oppure un percorso istituzionale configurabile, e non viene rimosso da aggiornamenti o disinstallazioni ordinarie. Database cifrato, recovery envelope MRK e manifest costituiscono il Recovery Set da conservare e sottoporre a backup come unità atomica.

Il manifest attivo v2 autentica anche lo SHA-256 dell'intero recovery envelope. Un mismatch viene rilevato all'avvio, apre la schermata di anomalia e blocca la creazione di nuovi backup. Restore locale e cloud applicano ceiling non disabilitabili, counting stream e preflight ZIP a memoria costante prima dell'estrazione.

La Master Recovery Key è l'unica credenziale segreta necessaria per ricollegare un Recovery Set esistente e integro dopo reinstallazione dell'applicazione o cambio del profilo Windows; una volta aperto l'archivio, viene creato un nuovo envelope DPAPI per il profilo corrente. La MRK non ricrea però file cancellati o corrotti: dopo una reinstallazione del sistema operativo, il Data Root deve essere sopravvissuto su un volume non cancellato oppure deve essere ripristinato da un backup cifrato verificato.

## Architettura corrente

```text
NeoScreenHub.Desktop         UI Avalonia, composizione e adapter i18n `.resx`
NeoScreenHub.Application     casi d'uso, filtri e orchestrazione
NeoScreenHub.Domain          episodi, evidenze, regole e state machine pure
NeoScreenHub.Infrastructure  SQLCipher, DPAPI, filesystem, import e backup
NeoScreenHub.Tests           suite consolidata unit/integration/headless UI
```

La Fase 2 ha sostituito gli stub di database, backup e audit con implementazioni reali. La Fase 2C ha sostituito l'apertura DPAPI trasparente con il keyring multiutente: password/PIN e segreto dispositivo partecipano entrambi al decrypt della DEK, mentre la MRK resta indipendente e portabile per il recovery. Lo schema corrente è `neoscreen-vault-v19`; autenticazione e RBAC baseline sono reali. La v19 aggiunge revoche append-only, checkpoint e vincoli di routing A/B; la migrazione v17 converte le materializzazioni autoritative a ×1000 senza usare `REAL`, registra il contesto numerico di ogni valutazione e introduce cutoff per fase; la v18 completa la migrazione del template QC ADR-062 e preserva come storico immutabile ogni revisione già usata. La Fase 6B.6 interpreta i Result Code 2TT tramite allowlist versionata, la 6B.7 rende l’amministrazione clinica accessibile tramite builder dichiarativo, la 6B.8 prepara candidate e prove Golden UAT, la Fase 7 completa proiezioni e workflow e la Clinical Activation Suite applica il gate medico guidato prima di qualunque effetto clinico reale. Il flusso clinico complessivo è:

```text
CSV originale (read-only stream)
  → riconoscimento formato e validazione contratto
  → parsing e normalizzazione deterministica
  → classificazione, Skip & Warn e correzione manuale del peso
  → SHA-256/HMAC, riuso visuale di file e righe già analizzati
  → candidate matching anagrafico e identità definitiva con Nome Madre
  → Ratio Engine e materializzazione valori derivati
  → transazione atomica dei soli record accettati nel database SQLCipher
  → episodio e media A/B
  → cutoff Warning su analiti/rapporti e regole diagnostiche Alert
  → matching code Medical-to-Lab e Ciclo del Rosso
  → Richiamo ospedaliero · valutazione Media · Registro Patologici
  → audit append-only
  → query parametrizzate e UI virtualizzata
```

## Documentazione di progetto

- [AGENTS.md](AGENTS.md): leggi operative, sicurezza, dependency policy e Style Guide.
- [piano_fasi.md](piano_fasi.md): piano attuativo, architettura e criteri di accettazione.
- [csv.md](csv.md): contratto del CSV, colonne, righe logiche e regole di validazione.
- [cutoff.md](cutoff.md): specchio clinico validato di cutoff, fasce, operatori, formule software e input esterni residui.
- [regole_patologie.md](regole_patologie.md): specifica clinica completa delle 24 regole cliniche primarie, routing 2TT phase-aware, Result Code e decisioni Q1–Q10.
- [progress.md](progress.md): memoria operativa e stato reale dello sviluppo.
- [docs/adr](docs/adr/README.md): decisioni architetturali finali e distinzione tra Foundation implementata e fasi future.
- [docs/rfc/dynamic-clinical-rules.md](docs/rfc/dynamic-clinical-rules.md): proposta tecnica per regole patologiche dinamiche, Visual Rule Builder, DSL dichiarativo, validazione, versionamento e attivazione auditata.
- [docs/security/recovery-set.md](docs/security/recovery-set.md): formato del Recovery Set, custodia chiavi, restore fail-closed e Google Drive.
- [docs/security/phase-2-2b-remediation-audit.md](docs/security/phase-2-2b-remediation-audit.md): chiusura dei finding, evidenze avversariali e confini del verdetto GO.
- [docs/security/threat-model-final.md](docs/security/threat-model-final.md):
  threat model di rilascio e rischi residui.
- [docs/release/INSTALLAZIONE-E-RIPRISTINO.md](docs/release/INSTALLAZIONE-E-RIPRISTINO.md):
  installazione, aggiornamento conservativo e recupero tramite MRK.
- [docs/release/RELEASE-CHECKLIST.md](docs/release/RELEASE-CHECKLIST.md):
  gate tecnici, Authenticode, recovery, UAT e pubblicazione GitHub.
- [docs/release/FUNCTIONAL-COVERAGE-AUDIT.md](docs/release/FUNCTIONAL-COVERAGE-AUDIT.md):
  matrice di copertura e attività ancora necessarie prima del pilota.

## Runtime, avvio e build

Il pacchetto finale è **self-contained**: include .NET Runtime 10.0.10 e tutte
le DLL/native library applicative richieste da Avalonia. Sul PC del laboratorio
non è necessario installare separatamente .NET o Windows Desktop Runtime.

Per **sviluppare, compilare o usare `dotnet run`** serve invece il .NET SDK x64 `10.0.302`, esattamente quello dichiarato in `global.json`; il solo runtime non contiene compiler, MSBuild o comandi SDK. L'SDK include già i runtime necessari, quindi sul computer di sviluppo non occorre installare anche il Desktop Runtime separatamente.

Verifica rapida dopo l'installazione:

```powershell
dotnet --list-runtimes
dotnet --list-sdks
dotnet --version
```

Eseguito dalla root del repository, `dotnet --version` deve restituire `10.0.302`.

```powershell
dotnet restore NeoScreenHub.sln --runtime win-x64 --locked-mode
powershell -NoProfile -File eng/Verify-Dependencies.ps1
powershell -NoProfile -File eng/Verify-Architecture.ps1
powershell -NoProfile -File eng/Verify-Localization.ps1
powershell -NoProfile -File eng/Verify-Privacy.ps1
dotnet build NeoScreenHub.sln --configuration Release --no-restore
dotnet test NeoScreenHub.sln --configuration Release --no-build --no-restore
dotnet format NeoScreenHub.sln --verify-no-changes --no-restore --severity warn
dotnet publish src/NeoScreenHub.Desktop/NeoScreenHub.Desktop.csproj --configuration Release --runtime win-x64 --self-contained true --no-restore --output artifacts/NeoScreenHub_Final_Build -p:PublishProfile=win-x64-self-contained -p:PublishSingleFile=false
```

Il comando maestro, che esegue anche test, SBOM, package, checksum e prova di
aggiornamento/disinstallazione senza alterare il Data Root, è:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\eng\New-Release.ps1
```

Per impostazione predefinita il comando rifiuta un worktree Git non pulito.
`-AllowDirtyWorktree` è riservato ai soli pacchetti tecnici di collaudo.
Il pilota senza firma è autorizzato dal responsabile (ADR-067); se viene
configurato un certificato istituzionale, `-RequireSignature` impone la firma
Authenticode. Nessun pacchetto non firmato viene dichiarato firmato.

Gli artifact vengono generati in:

```text
artifacts\NeoScreenHub_Final_Build\
artifacts\NeoScreenHub_Installer_Package\
artifacts\NeoScreenHub-1.3.0-win-x64-self-contained.zip
```

### Installazione, aggiornamento e protezione del database

Il pacchetto installa i binari in
`%LOCALAPPDATA%\Programs\NeoScreenHub\`; il database e il Recovery Set
rimangono in `%LOCALAPPDATA%\NeoScreenHub\Data\`. I percorsi sono disgiunti per
contratto e lo script rifiuta configurazioni sovrapposte.

Durante un aggiornamento:

1. i nuovi binari vengono copiati in staging e verificati tramite SHA-256;
2. i binari correnti vengono conservati come `NeoScreenHub.previous`;
3. la nuova directory viene promossa;
4. il Data Root non viene aperto, copiato, spostato o cancellato
   dall'installer;
5. al primo avvio l'applicazione crea un Recovery Set cifrato interno prima
   delle migrazioni SQLCipher versionate e compatibili; se il checkpoint
   fallisce, la migrazione non viene avviata.

I checkpoint interni sono separati dal Data Root, in
`%LOCALAPPDATA%\NeoScreenHub\RecoveryCheckpoints\`: `Automatic` contiene gli
ultimi 10 checkpoint automatici registrati e verificati per hash; `Pinned`
conserva quelli pre-migrazione e pre-attivazione senza rotazione. File non
riconosciuti o alterati non vengono cancellati dalla rotazione. I backup
manuali conservano sempre la destinazione scelta dall'Admin e non vengono
eliminati automaticamente. Una copia sullo stesso disco non protegge dalla
perdita del disco: mantenere anche una copia verificata su supporto separato.

Anche la disinstallazione rimuove esclusivamente binari e collegamento Start.
Prima di ogni aggiornamento resta obbligatorio creare un `.nshbackup` verificato
e custodire la MRK separatamente: un rollback dopo migrazione deve ripristinare
insieme versione applicativa e Recovery Set pre-aggiornamento.

Per l'installazione e il ripristino completi consultare
[Installazione e ripristino](docs/release/INSTALLAZIONE-E-RIPRISTINO.md).

### Identità visuale

`assets/icona.png` è la sorgente canonica. `assets/NeoScreenHub.ico` incorpora dieci risoluzioni da 16 a 256 px ed è configurato come icona dell'eseguibile e della finestra. La shell mostra il logo 80×80 in un host trasparente 96×96 nel quadrante superiore sinistro di intersezione tra Header e Sidebar, senza superficie o bordo da pulsante, affiancato dalla label localizzata **NeoScreen Hub** con una lieve ombra tipografica. Hash e provenienza sono registrati in [assets/README.md](assets/README.md).

### Verifiche correnti

- build `Release`: zero warning e zero errori con nullable, analizzatori e `TreatWarningsAsErrors=true`;
- 546 test automatici superati nella bonifica 1.3.0, inclusi SQLCipher reale, MRK/DPAPI,
  manifest/envelope corrotti, restore, audit/outbox atomici, HMAC v2, Numeric
  Policy, OAuth/PKCE/Drive ostili, Ingestion, QC, Query, 2TT, Ciclo del Rosso,
  packaging e shell Avalonia headless;
- 883 risorse con parità esatta italiano/inglese, terminologia user-facing
  “Database” e nessun attributo UI letterale rilevato;
- 54 pacchetti FOSS a versione esatta in cinque lock file;
- publish tecnico 1.3.0 self-contained verificato, runtime 10.0.10 incluso;
  la pipeline completa del nuovo ZIP resta da eseguire dal proprietario;
  le precedenti prove installer con sentinella non sono un collaudo del nuovo ZIP;
- controllo aggiornamenti fissato a
  `https://api.github.com/repos/velodent/neoscreenhub/releases/latest`.

## Riservatezza

Il repository non deve contenere dati sanitari reali non anonimizzati. I file di esempio e le fixture di test dovranno essere sintetici o adeguatamente anonimizzati e gestiti secondo le regole definite in `AGENTS.md`.
