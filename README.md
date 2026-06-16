# Framework Operativo per lo Sviluppo Web in Vibe Coding (OpenCode)
**Progetto di Riferimento:** Todo App / Lista della Spesa (Vue 3 SPA + Supabase + Google Stitch via MCP)
**Data Ultimo Aggiornamento:** 20 Maggio 2026

Questo documento definisce il workflow standard standardizzato basato sulla separazione rigida del contesto e sull'uso di agenti custom e sub-agenti specialistici isolati. Impedisce il degrado delle performance dell'LLM (sopra i 100k token) e mantiene il progetto deterministico e scalabile.

---

## Mappa dei Flussi e Relazioni di Contesto

```
[FASE 1: Setup Globale] ──> Configurazione opencode.json (MCP Servers Attivi)
                                     │
[FASE 2: Progettazione] ──> Orchestratore Senior Architect (Genera prd.md, architecture.md, DESIGN.md)
                                     │
[FASE 3: Pianificazione] ──> Evocazione TASK MANAGER (Genera roadmap_2026-05-20.md con flag Parallel e current_state.md)
                                     │
[FASE 4: Team Setup] ────> Comando /init nella root ──> Generazione file AGENTS.md nella root del progetto
                                     │
[FASE 5: Esecuzione] ────> Chat Isolate per singoli Task ──> Comando /checkpoint ──> /clear (Reset Token)
```

## Architettura del Team di Agenti (Modello IBM)

Nel nostro ecosistema, un compito complesso viene risolto da un team di specialisti suddivisi in 4 macro-aree cognitive:
1. **Thinking & Planning (Pensiero):** L'Agente Custom `Architect` progetta, il sub-agente `Task-Manager` pianifica e isola le dipendenze.
2. **Doing & Operating (Azione):** I sub-agenti `Vue-Frontend-Dev` e `Supabase-DB-Dev` scrivono il codice. Usano i Tool MCP (`google-stitch`) come estensioni operative.
3. **Feedback & Critiquing (Controllo):** Il sub-agente `QA-Critico` testa il codice e contrasta le allucinazioni usando `context7`.
4. **Supervising & Presenting (Chiusura):** Il protocollo `/checkpoint` supervisiona i blocchi logici e l'agente finale assume il ruolo di `Presenter` per riassumere l'output all'utente umano.

```
                  ┌────────────────────────────────────────┐
                  │  AGENTE CUSTOM PERMANENTE: Architect  │ (Thinking)
                  └───────────────────┬────────────────────┘
                                      ▼
                  ┌────────────────────────────────────────┐
                  │       Sub-Agente: Task-Manager         │ (Planning)
                  └───────────────────┬────────────────────┘
                                      ▼
             ┌────────────────────────┴────────────────────────┐
             ▼                                                 ▼
┌──────────────────────────┐                      ┌──────────────────────────┐
│ Sub-Agente: Vue-Frontend │ (Doing)              │  Sub-Agente: Supabase-DB  │ (Doing)
└────────────┬─────────────┘                      └────────────┬─────────────┘
             │ (Usa MCP Stitch)                                │ (Usa SQL Skills)
             └────────────────────────┬────────────────────────┘
                                      ▼
                  ┌────────────────────────────────────────┐
                  │        Sub-Agente: QA-Critico          │ (Feedback/Critic)
                  └───────────────────┬────────────────────┘
                                      ▼
                  ┌────────────────────────────────────────┐
                  │       Fase Custom: Presenter           │ (Communication)
                  └────────────────────────────────────────┘
```

---

## STEP 1: Preparazione Ambiente e Configurazione MCP Globale

1. Apri il terminale del tuo sistema operativo, crea la cartella del progetto e spostati al suo interno:
   ```bash
   mkdir spesa-vue && cd spesa-vue
   
```

2. Apri (o crea se non esiste) il file di configurazione globale di OpenCode sul tuo computer situato in `~/.config/opencode/opencode.json` e assicurati che i server MCP siano registrati esattamente così:
   ```json
   {
     "mcpServers": {
       "google-stitch": {
         "command": "npx",
         "args": ["-y", "@stitch/mcp-server", "--api-key", "STITCH_SECRET_API_KEY_2026"]
       },
       "context7": {
         "command": "npx",
         "args": ["-y", "@context7/mcp-server"]
       }
     }
   }
   
```

---

## STEP 2: Sessione Blueprint (Creazione Agente Custom "Architect", PRD & DESIGN.md)

Avvia OpenCode nel terminale dentro la cartella `spesa-vue`. Seleziona il modello LLM e prima di procedere alla scrittura dei file, configuriamo l'interfaccia creando un **Agente Custom permanente** in OpenCode (Orchestratore), in modo da non dover reinviare il System Prompt a ogni apertura di sessione.

### Nota di Configurazione dell'Agente Custom
> **Nome Agente:** `Architect`  
> **Capabilities abilitate di default:** `Plan`, `Build` (Nessun'altra modalità attiva per preservare i token).  
> **Permessi:** Lettura e scrittura completi sul filesystem del workspace.  
> **System Prompt dell'Agente Custom:** *"Sei un Senior Web Architect con 20 anni di esperienza nello sviluppo di Single Page Application e sistemi cloud distribuiti. Il tuo obiettivo è la coerenza tecnica, la minimizzazione assoluta del debito tecnico e l'isolamento del contesto. Non tolleri codice scritto senza una pianificazione preventiva. Ti esprimi e progetti esclusivamente tramite file Markdown."*

Seleziona l'agente custom `Architect` appena creato e invia il primo prompt

**Prompt 1.1**:


```markdown
I tuoi task ora sono:
1. Crea una cartella chiamata `.opencode` nella root del progetto.
2. All'interno di essa, crea il file `.opencode/prd.md` che descriva il Product Requirement Document per una Todo App (Lista della spesa) avanzata: deve supportare liste multiple, condivisione della lista tra utenti in tempo reale e persistenza offline locale con sincronizzazione automatica.
3. Crea il file `.opencode/architecture.md` per impostare lo stack: Vue 3 (Vite, Pinia), Supabase (PostgreSQL, Row Level Security, Realtime).
4. Esegui un'azione di lettura sul mio file di skill globale situato in `~/.opencode/skills/supabase-crud.ms`. Estrai le linee guida architetturali di quel file e integrale come "Regole Architetturali Obbligatorie" all'interno di `.opencode/architecture.md`.

Non scrivere codice applicativo adesso. Genera solo la struttura delle cartelle e i due file Markdown richiesti usando i tuoi tool di scrittura file. Conferma l'avvenuta creazione.
```

### Prompt 1.2: Estrazione Token Grafici e Collegamento Architetturale
Subito dopo la conferma della creazione dei file, invia questo prompt nella stessa sessione mantenendo l'agente `Architect`:

```markdown
Usa lo strumento MCP `google-stitch` per ispezionare il progetto all'indirizzo del design system ufficiale: [INSERISCI_URL_DI_GOOGLE_STITCH].

Estrai:
1. I design token relativi alla palette colori, tipografia e spaziature, salvando queste informazioni strutturate in un nuovo file chiamato `.opencode/DESIGN.md` (Praticamente uno standard per AI-Generated UI: https://www.youtube.com/watch?v=W1gWIQp9k1Y).
2. La struttura visiva e gli stati dei componenti della lista della spesa (campo di input, card dell'item, checkbox di spunta, bottone di eliminazione).

Una volta estratti, aggiorna il file `.opencode/architecture.md` inserendo una sezione dedicata al "Mappaggio dei Componenti Grafici", definendo chiaramente quale componente Vue dovrà ereditare e applicare i token rilevati da Stitch.
```

---

## STEP 3: Sessione Pianificazione e Analisi del Parallelismo (Task Manager)

Svuota completamente la memoria della chat per eliminare i token accumulati durante la fase di progettazione. Digita nel terminale di OpenCode:

```bash
/clear
```
*(Oppure clicca su "New Session" nell'interfaccia grafica).*

In questa nuova sessione pulita, evochiamo il sub-agente **Task Manager**. Passiamo solo l'output dello Step 2. Incolla questo prompt esatto:

```markdown
Sei un esperto Task Manager e Project Management Officer (PMO) specializzato in flussi di lavoro ad agenti. Il tuo unico compito è prendere i documenti tecnici prodotti dall'Architect ed elaborare una pianificazione atomica ottimizzata per il parallelismo.

I tuoi task ora sono:
1. Leggi esclusivamente i file `.opencode/prd.md`, `.opencode/architecture.md` e `.opencode/DESIGN.md`.
2. Crea una sottocartella chiamata `.opencode/plan/`.
3. Crea al suo interno il file datato `.opencode/plan/roadmap_YYYY-MM-DD.md`. Dividi lo sviluppo del software in Batch logici sequenziali (Es. Batch 1: Setup & DB, Batch 2: UI e Componenti, Batch 3: Integrazione Logica & Realtime).
4. Fornisci per ogni Batch i singoli Task in modo atomico. Ogni Task deve contenere tassativamente: 
   - ID: [E.g., T1.1]
   - Descrizione chiara del task.
   - Criteri di Accettazione (Acceptance Criteria) stringenti.
   - Context Files Pointer: L'elenco dei soli file che il sub-agente dovrà leggere (es. solo DESIGN.md e il file .vue specifico).
   - Parallel: [True/False] (Specifica SE questo task può essere eseguito in contemporanea ad altri task dello stesso Batch in sessioni separate).
5. Crea il file di puntamento volatile `.opencode/plan/current_state.md`. Questo file deve contenere unicamente:
   - Il link al file della roadmap attiva (`roadmap_YYYY-MM-DD.md`).
   - L'ID del task attualmente in corso.
   - Il nome del sub-agente attualmente attivo.

Genera la struttura dei file sul filesystem usando i tuoi tool. Non scrivere codice.
```

NOTA: Utiliziamo nel nostro workflow la memorizzazione dei plan, in questo caso lo abbiamo chiesto via chat ma Gemini e Claude code hanno dei settings specifici per abilitarlo (https://geminicli.com/docs/cli/plan-mode/#custom-plan-directory-and-policies e https://code.claude.com/docs/en/settings). Inoltre ricorda di creare un ".geminiignore" o l'ecquivalente (se esiste) per lo strumento che usi dove ci metterai file e cartelle da non far passare all'llm (praticamente deve essere una copia del .gitignore (https://geminicli.com/docs/cli/gemini-ignore/#how-it-works) )

---

## STEP 4: Definizione del Team di Sviluppo (AGENTS.md nella Root)

Senza cambiare sessione, andiamo a inizializzare il file dei sub-agenti operativi. Per farlo, sfruttiamo il comando nativo di OpenCode che genera il file direttamente nella radice del workspace. Invia questo prompt:

```markdown
Esegui il comando `/init` per generare il file `AGENTS.md` direttamente nella root del progetto (fuori dalla cartella `.opencode`).

All'interno del file `AGENTS.md` appena generato, configura e codifica i System Prompt e i vincoli operativi dei 3 sub-agenti di sviluppo che evocheremo durante le sessioni isolate:

1. **Vue-Frontend-Dev**:
   - Competenze: Vue 3 (Composition API), Pinia, CSS basato strettamente sui token definiti in `DESIGN.md`.
   - Vincoli: Non conosce la struttura del database Supabase, né le sue tabelle. Lavora isolato sui file `.vue` nella cartella `src/`. Comunica con i servizi dati solo tramite contratti di interfaccia astratti (funzioni stub / API mock).

2. **Supabase-DB-Dev**:
   - Competenze: PostgreSQL, Migrazioni Supabase, Row Level Security (RLS), Realtime replication configuration.
   - Vincoli: Lavora esclusivamente dentro la cartella `supabase/migrations/`. Deve seguire pedissecamente le regole della skill `supabase-crud.ms` copiata in architettura.

3. **QA-Controller alia QA-Critico**:
   - Competenze: Code Review rigorosa, Static Analysis, verifica dei requisiti di sicurezza e performance.
   - Vincoli: Non scrive codice applicativo. Legge il codice prodotto dagli altri agenti e usa lo strumento MCP `context7` per verificare che la sintassi di Supabase e Vue sia aggiornata alla documentazione ufficiale dell'anno corrente. Blocca il rilascio se rileva funzioni deprecate o violazioni del PRD.

Configura e scrivi il file `AGENTS.md` secondo queste specifiche.
```

---

## STEP 5: Iniezione del Protocollo di Stato e Creazione Comandi Custom

Per fare in modo che il sistema mantenga permanentemente la memoria dello stato di avanzamento tra una chiusura e una riapertura della chat, impostiamo le regole del Checkpoint. Invia questo prompt finale:

```markdown
Aggiorna il file `.opencode/plan/current_state.md` aggiungendo in calce la specifica del protocollo di persistenza della sessione che utilizzeremo rigorosamente da ora in avanti. Definisci le seguenti regole operative:

### Regola per il comando testuale "/checkpoint":
Quando digiterò "/checkpoint", l'agente attivo deve sovrascrivere interamente il file `.opencode/plan/current_state.md` (mantenendolo leggero, sotto le 30 righe) aggiornando solo:
1. Timestamp del salvataggio.
2. ID del Task della roadmap corrente appena completato.
3. ID del prossimo Task da eseguire e nome del sub-agente da evocare preso da `AGENTS.md` nella root.
4. File Modificati Pointer: Elenco dei file fisici toccati nell'ultima sessione che dovranno essere ereditati.
5. Context Delta: Eventuali variazioni impreviste o problemi bloccanti emersi.
Se una nuova feature richiede una deviazione importante dal piano, l'agente genererà un nuovo file di roadmap sussidiario (es. `.opencode/plan/roadmap_feature_x.md`) e sposterà il puntatore di `current_state.md` verso di esso.

### Regola per la chiusura del Batch (Ruolo: Presenter):
Al completamento di un intero Batch di lavoro, prima del checkpoint, l'agente deve assumere il ruolo di Presenter: generare un riepilogo in linguaggio umano che spieghi i componenti o le tabelle create, come interagiscono tra loro e l'impatto sul sistema, per rendicontare l'avanzamento all'utente.

### Regola per il comando testuale "continuiamo da dare dove abbiamo lasciato":
Quando digiterò questa frase all'avvio di OpenCode, l'agente dovrà:
1. Leggere unicamente `.opencode/plan/current_state.md`.
2. Sulla base del "Next Task" e dei "Context Files Pointer", caricare nella memoria della chat esclusivamente i file necessari per quell'attività, lasciando il resto del repository invisibile per non intasare i token.
3. Caricare il System Prompt del sub-agente deputato estratto dal file `AGENTS.md` presente nella root del progetto.

Salva questa configurazione, esegui un primo `/checkpoint` fittizio per inizializzare il file con lo stato "Pronto per il Batch 1" e conferma.
```

Una volta ricevuta la conferma, pulisci definitivamente la sessione digitando:
```bash
/clear
```

---

## GUIDA OPERATIVA: Come lavorare alle Feature da questo momento in poi

Il tuo progetto è ora configurato. La memoria volatile dell'IA è a 0 token. Ecco l'esempio pratico di come procedere per sviluppare il primo task di codice in modo isolato o in parallelo.

### 1. Riapertura della Sessione (Inizio di un Task)
Quando apri OpenCode per lavorare, digiti la frase chiave:
> **Tu:** continuiamo da dove abbiamo lasciato

**Cosa fa OpenCode dietro le quinte:**
L'agente legge `current_state.md` -> Vede che il task corrente è il `T1.1 (Creazione Tabelle DB)` -> Vede che l'agente richiesto è `Supabase-DB-Dev` -> Legge in `AGENTS.md` il profilo di quell'agente -> Carica nel contesto *solo* `architecture.md` e la cartella `supabase/migrations/`. La chat è pulita, focalizzata e non ha allucinazioni.

### 2. Sviluppo in Contemporanea (Esecuzione Parallela)
Se la roadmap dice che due task hanno il flag `Parallel: True` (es. Il DB-Dev crea le tabelle e il Frontend-Dev crea la UI con i dati mock):
* Apri **due schede del terminale separate** con OpenCode attivo.
* Nella Scheda A digiti: `Attiva Vue-Frontend-Dev per il Task T2.1 della roadmap_2026-05-20.md`.
* Nella Scheda B digiti: `Attiva Supabase-DB-Dev per il Task T1.1 della roadmap_2026-05-20.md`.
I due agenti lavoreranno in parallelo senza toccare l'uno i file dell'altro, eliminando i conflitti.

### 3. Fase di Controllo e Chiusura Sessione
Prima di fare il checkpoint, evochi l'agente di controllo per validare il codice:
> **Tu:** Attiva QA-Controller aka QA-Critico per verificare il codice appena scritto rispetto ai criteri di accettazione del Task corrente.

Se il `QA-Controller` approva, invii il comando finale per salvare lo stato e resettare la memoria:
> **Tu:** /checkpoint

L'agente aggiorna il segnalibro in `current_state.md`, indicando i file modificati e passando il testimone logico al sub-agente successivo. Digiti `/clear` e la finestra dei token torna immediatamente a zero, pronta per l'attività successiva.