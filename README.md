🚀 NeoScreen Hub v1.2.1

NeoScreen Hub è una soluzione desktop professionale dedicata alla gestione,
all'analisi e alla refertazione dei dati provenienti dallo Screening Neonatale
Esteso.

Sviluppato per operare esclusivamente in ambiente locale, il software funge da
centro nevralgico per l'importazione di flussi CSV giornalieri, garantendo la
massima sicurezza del dato sanitario e fornendo un supporto decisionale avanzato
attraverso un motore di regole cliniche dinamiche.

🌟 Caratteristiche Principali

🔬 Precisione e Analisi Clinica

  - Motore Cutoff v2: Gestione di 128 limiti clinici con precisione al millesimo
    (\times 1000).
  - Ratio Engine Autoritativo: Calcolo automatico di 53 rapporti biochimici
    (incluso Acs/Cit) con logica deterministica.
  - Valutazione Multi-Fase: Cutoff differenziati per fase analisi (Iniziale,
    Retest, Richiamo) e fasce d'età neonatali (0-360h e >360h).
  - Reflex Testing (2TT): Gestione integrata di test di secondo livello (MMA,
    IVA, HCY, MSUD) con matching automatico dei pazienti.

🛡️ Sicurezza e Privacy (Privacy-by-Design)

  - Cifratura Totale: Database locale protetto da SQLCipher Community con
    standard AES-256.
  - Accesso Multi-Utente: Login differenziato per Admin (Password) e
    Technologist (PIN a 4 cifre), blindato all'hardware tramite Windows DPAPI.
  - Zero Cloud Dependency: Tutti i dati sensibili risiedono esclusivamente sul
    PC del laboratorio.
  - Master Recovery Key (MRK): Sistema di recupero d'emergenza tramite chiave
    a 24 parole per il ripristino su nuove postazioni.
  - Audit Trail: Registro permanente e inalterabile di ogni operazione clinica o
    amministrativa.

⚙️ Workflow Operativo (Medical-to-Lab)

  - Ingestion Resiliente (Skip & Warn): Importazione intelligente dei CSV che
    isola gli errori di riga senza bloccare l'intero caricamento.
  - Ciclo del Rosso: Gestione automatizzata di Retest, Richiami Ospedalieri e
    Registro Patologici.
  - TPN Safety Interlock: Protocollo di sicurezza dedicato per i pazienti in
    nutrizione parenterale (Visualizzazione Viola).
  - Refertazione Rapida: Generazione di diagnosi editabili e pronti per il
    "Copia & Incolla" nei sistemi ospedalieri.

🎨 Interfaccia "High-End"

  - Medical Clean Glass: Estetica moderna basata su effetti Mica/Acrylic
    (Windows 11/10).
  - Wide Grid: Visualizzazione simultanea di oltre 100 parametri con "righello"
    dei cutoff sincronizzato.
  - Focus Mode: Modalità a schermo intero per l'analisi di precisione delle
    singole tabelle.

🛠️ Stack Tecnologico

  - Framework: .NET 10 LTS (C# 14)
  - UI: Avalonia UI (Desktop nativo)
  - Database: SQLite + SQLCipher Community 4.17.0
  - Cloud (Optional): Google Drive API v3 (Backup cifrati)
  - Architettura: Clean Architecture & MVVM

📦 Installazione e Avvio

NeoScreen Hub è distribuito in modalità Self-Contained: il runtime .NET è
incluso nel pacchetto e non richiede installazioni aggiuntive sul PC host.

1.  Scarica l'ultimo archivio NeoScreenHub-1.2.1-win-x64-self-contained.zip
    dalla sezione Releases.
2.  Estrai il contenuto in una cartella locale (es. C:\Programmi\NeoScreenHub).
3.  Lancia il file Installa-NeoScreenHub.cmd per configurare i collegamenti.
4.  Al primo avvio, segui la procedura di Onboarding per configurare l'account
    Amministratore e generare la tua Master Recovery Key.

🚦 Stato del Progetto

Il software è attualmente in fase di Pilot/Alpha.

  - Validazione Tecnica: Superata (511 test automatici).
  - Validazione Clinica: In corso (richiede attivazione della release RC2 da
    parte dell'Admin istituzionale).

📄 Licenza e Riservatezza

  - Licenza: Software distribuito secondo le licenze FOSS incluse nel pacchetto
    (MIT/Apache 2.0).
  - Dati: Questo repository non contiene dati sanitari reali. Tutti i file di
    esempio sono sintetici e anonimizzati.

NeoScreen Hub: Precisione clinica, Sicurezza garantita.
