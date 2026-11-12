# 05 – Applicazioni client-side con React

Fonte: slide "4 - Applicazioni client-side con REACT" (`03-TWeb-ReactTeoria-04-2627-ppt.pdf`, Moodle TWeb 2026/27, versione aggiornata il 2026-11-12 con le slide nascoste; Picardi e Mauro). Convenzione: `[INT]` = integrazione mia, `[DA VERIFICARE]` = punto ambiguo. Non ci sono ancora slide su router, Spring, JPA e sessioni: vedi README.

## Mappa degli argomenti

Architettura a tre strati e flussi di comunicazione; dove sta la logica (client, server, mista); perché React, libreria e non framework; componenti, gerarchie, SPA; pattern MVVM; JSX/TSX; props e delega al genitore; state e `useState`; form con binding bidirezionale; ciclo di rendering; `useEffect` con dipendenze e cleanup.

## Architettura: tre strati e flussi

Un'applicazione ha tre strati: **User interface**, **Application business logic**, **Persistenza**, più eventuali **servizi di terze parti** (es. pagamenti con carta). Cinque flussi:

| Flusso | Che cosa succede | Esempio |
|---|---|---|
| 1 retrieval + presentation | dati persistenti → business logic → UI; la business logic può elaborare anche la presentazione (aggregato o espanso, quali campi) | esami di uno studente → stato di avanzamento → azioni possibili |
| 2a azione solo UI | l'evento resta nell'interfaccia | allargare un pannello, selezionare |
| 2b azione non persistente | la business logic interpreta il dato ma non si salva nulla | applicare un filtro (es. riordino per anno accademico, che richiede di capire il dominio) |
| 2c azione con modifica persistente | UI → business logic (verifica/elabora) → persistenza; poi la UI rilegge, ripassa dalla business logic e aggiorna la vista | iscriversi a un esame |
| 3 aggiornamento dall'esterno (push) | strutturalmente come il flusso 1, ma l'evento è esterno e l'app è già attiva | arriva il post di un altro utente |

**Dove stanno i tre strati.** La UI è sempre lato client (è la vista del browser). La persistenza è di norma sul server remoto, per due motivi: accesso da più dispositivi e natura collaborativa/multiutente. La business logic dipende dal tipo di app:
- **prevalentemente client-side**: app per un singolo utente e molto interattive (editor di diagrammi, Google Documents); salvano periodicamente sul server; sono le app dette *serverless*;
- **prevalentemente server-side**: app di business con molti servizi integrati e sicurezza alta; il server è la *source of truth* e la UI non ha logica propria (in una banca la gestione dei fondi non può stare nel front-end, perché il codice che gira sul dispositivo dell'utente è manipolabile);
- **headless**: solo back-end che espone API, a cui si collegano front-end diversi;
- **"metà e metà"**: in realtà due applicazioni distinte, ha senso solo se sono davvero disaccoppiabili concettualmente; altrimenti rende difficile sapere dove mettere un pezzo di logica.
Regola generale, non assoluta: app per un utente → primo modello; app di business → secondo.

## React: libreria, non framework

- React è propriamente una **libreria**; "il" framework di React è Next.js. Si usa con **Vite** e **npm** per creare e gestire il progetto. Supporta UI a **componenti** e **Single Page Application**; Angular e Vue sono framework a tutto tondo, simili in questo.
- Una libreria è un insieme di package da importare e usare. React include un **runtime** (oggetti e funzioni creati all'avvio che gestiscono aspetti del funzionamento dei componenti) ma non altera i meccanismi di JS/TS; la sua estensione di linguaggio (JSX/TSX) richiede la compilazione, come TypeScript. Un vero framework crea una specie di macchina virtuale "dentro" l'ambiente di esecuzione, con un contesto altamente modificato: Angular monitora i cambiamenti nelle proprietà degli oggetti mostrati; Vue crea copie addizionate di metodi e proprietà.
- I framework sono più completi (oltre la UI); in React gli altri aspetti sono affidati a librerie separate. Perché React per la didattica: è "leggero" e con JS/TS già noti se ne capiscono i meccanismi.
- **React non è object-oriented**: è basato sugli aspetti funzionali di JS/TS.
- Il vantaggio di un framework: evitare il **micromanagement**, cioè la gestione manuale e di basso livello di micro-eventi dell'interfaccia, e programmare in modo modulare guardando gli obiettivi dell'applicazione.

## Componenti UI

Un componente UI ha: un **aspetto grafico** (template: HTML e CSS), una **parte del Data Model** che visualizza o modifica (esistono anche componenti statici, di navigazione o di show/hide), un **comportamento** (cosa può fare l'utente e come).

- **Gerarchie**: un componente è costruito da *istanze* di altri; è **genitore** di quelle istanze, che sono i **figli**. Un componente in astratto non è figlio di nessuno: lo sono le sue istanze, e istanze diverse possono avere genitori diversi. La definizione del componente non deve dire chi lo conterrà, solo come potrà comunicare con chi lo conterrà. La *gerarchia statica* elenca tutti i figli che un componente dichiara, indipendentemente da quelli mostrati in un dato momento.
- **Componenti dinamici**: cambiano aspetto in base a dati e interazione, anche il set dei figli. Due modi di far sparire e riapparire un figlio: **show/hide** (cambiare la classe CSS, resta nell'HTML ma invisibile) oppure **mount/unmount** (aggiungerlo e toglierlo dal DOM). Si preferisce show/hide quando serve spegnere/accendere spesso e in fretta; mount/unmount per un sotto-componente sostanzioso mostrato di rado, o quando la struttura nasce dai dati e i dati cambiano.

## Single Page Application

Le "pagine" sono componenti figli di un macro-componente **App**, che ne monta uno alla volta: navigare significa smontare un figlio e montarne un altro. Il browser scarica una sola pagina e tutti gli script; non ci sono mai navigazioni vere né collegamenti ipertestuali che farebbero reload. Due modi: **gestione manuale** (menu come pulsanti che montano/smontano) o **router** (libreria che definisce route, analoghe a quelle lato server, e per ciascuna quali componenti montare; il router intercetta la route senza passare dal browser, e la barra mostra la route attiva dando l'illusione di navigare). Il router non è nel programma del corso.

## Model-View-ViewModel (MVVM)

Pattern architetturale per UI a componenti: **Modello** (i dati e i vincoli su fruizione ed elaborazione), **View** (ciò che l'utente vede e con cui interagisce: il DOM), **View-Model** (il comportamento dei componenti: legge i dati dal modello, richiede le modifiche, rilegge i dati aggiornati, presenta i dati alla view; riceve gli eventi UI e aggiorna la presentazione). Il collegamento View-Model/View è in parte automatizzato dal **binding**:
- **property binding**: il framework si accorge dei cambiamenti nel view-model e aggiorna la view; ogni framework ha la sua strategia;
- **event binding**: basato su callback ed event listener, associa una funzione a un evento UI.

In React il componente è il View-Model.

## Componenti in React, funzione di rendering, JSX/TSX

- Un componente è una **funzione di rendering** che costruisce il sottoalbero del DOM della view. Il nome della funzione è il nome del componente (e diventa un tag usabile nella view di altri componenti). Il tipo è `FunctionComponent`; l'output deve essere un **ReactElement**.
- React richiede funzioni **pure**: output che dipende solo dall'input, nessuna modifica a strutture esterne allo scope. Vale per la funzione di rendering, non per le callback al suo interno.
- Quando i dati da cui dipende la view cambiano, React richiama la funzione e sostituisce l'elemento nel DOM: è il property binding. Le callback specificate nella funzione di rendering sono l'event binding.
- **JSX/TSX** è HTML interpolato con espressioni JS/TS tra `{ }`: `const elem = <p>Pagina creata da {utente.nome} {utente.cognome}</p>`.
- Regole: l'espressione deve avere **una sola radice** (per più radici si usa il tag vuoto `<>…</>`); tutti i tag chiusi (`<img src="…"/>`); attributi kebab-case diventano **camelCase** (`data-value` → `dataValue`); `class` diventa **`className`** (parola riservata).
- Interpolazione in tre punti: come contenuto intero di un tag (`<ul>{lista}</ul>`), come testo dentro un tag (`<p>{nome} ha vinto {x+y} euro!</p>`), come valore di un attributo (`<div className={val}>`). Il valore diventa stringa, oppure (nei primi due casi) un ReactElement o un array di ReactElement. Nell'attributo si associano i comportamenti agli eventi: `onClick`, `onMouseDown`, `onKeyPress`: `<button onClick={(e) => myFunction(e, …)}>`.

## Props e delega al genitore

- Le **props** sono i parametri d'ingresso del componente, assegnati dal genitore quando lo istanzia, come attributi del tag. La forma dell'oggetto props si dichiara con una **interface**:
```tsx
interface BookProps { bIS: BookInBookshelf; }
function Book({ bIS }: BookProps): ReactElement { … }
// genitore
const bookElems = books.map(book => <Book bIS={book} />);
```
- Vantaggi: **riuso**, **configurabilità** (il genitore decide aspetti della view del figlio), **gestione del flusso di informazioni** (ogni componente conosce solo la parte del data model che gli serve; il genitore spartisce i dati). Il più importante: **se il valore di una prop cambia, React rigenera la view** richiamando la funzione.
- **Object destructuring** (`const {x, y} = pippo`, `function fun({x, y})`) e array destructuring: sintassi che dichiara variabili dai campi di un oggetto o dagli elementi di un array; molto comune nei componenti.

## Stato e `useState`

- Senza stato la view sarebbe statica. Le variabili locali non vanno bene: la funzione viene rieseguita a ogni rendering e le variabili si ridichiarano (le modifiche andrebbero perse). Lo **stato** è tenuto "da React" in un deposito esterno ai componenti.
- Il componente ottiene da React il valore corrente delle sue variabili di stato e una funzione **setter**. Il setter **non** si chiama nel corpo del componente ma solo in callback asincrone (listener, `then`, effetti), perché provoca un re-rendering: chiamarlo nel corpo darebbe un **loop infinito**. Se cambia uno state, React rigenera la view.
- **`useState<T>(iniziale)`** restituisce un array di due elementi `[valore, setter]` (array destructuring): `const [activeItem, setActiveItem] = useState<ContentNavItem>("libreria")`. Il valore iniziale vale solo al primo rendering. Il setter prende un valore di tipo `T`, oppure una **funzione di aggiornamento** `(prec: T) => T` che calcola il nuovo valore dal precedente.
- Esempio dalle slide: `type ContentNavItem = "negozio" | "libreria" | "lend" | "borrow" | "amici";` e un `<div className={…} onClick={() => setActiveItem("negozio")}>`.
- **Hook**: funzioni della libreria che "agganciano" il codice al ciclo di vita gestito da React. I nomi iniziano sempre con `use`. Altri: `useReducer`, `useContext`, `useRef`, `useMemo`, `useCallback`, `useEffect`. Si possono creare hook custom (non trattati). **Regola d'uso**: solo nello scope principale di una funzione-componente o di un custom hook.

## Delega dell'evento al genitore

Se lo stato di un componente deve cambiare per un evento nel figlio (es. lista con un solo elemento selezionato: ogni voce sa di essere cliccata, ma per deselezionare le altre serve la visione d'insieme del genitore, che tiene lo stato), il figlio riceve tra le **props una callback del genitore** da chiamare quando l'evento si verifica. Il figlio può delegare in toto o in parte (chiamare la callback del genitore e anche una propria). Convenzione: le props-callback si chiamano **`on` + nome dell'evento** (`onAction`, `onClick`, `onSelection`). Anche gli attributi `onClick` degli elementi HTML in JSX sono props: gli "elementi HTML" in JSX sono componenti interni di React.

**Livelli di astrazione degli eventi.** Il figlio può esporre direttamente l'evento UI (`onClick`) oppure, elaborandoli, un evento di livello più alto (**evento custom**, es. `onSelection` quando la selezione cambia davvero: se si clicca tre volte sulla stessa voce, solo il primo click seleziona).

## Form in React: binding bidirezionale

In una SPA non si usa il submit automatico del browser (provocherebbe il reload della pagina). Un componente **form-like** è qualunque componente con elementi di input (`INPUT`, `SELECT`, `TEXTAREA`); anche i pulsanti di conferma/reset possono essere qualunque elemento cliccabile. Tre passi:
1. per ogni input uno **state** con il suo valore (`text` → string; `select` → il `value` dell'`option` scelta, string; `checkbox` → boolean);
2. assegnare all'input l'attributo `value` uguale allo state;
3. gestire `onChange` modificando lo state: `<input type="text" value={stateVar} onChange={(e) => setStateVar(e.target.value)} />`.

È sia property binding sia event binding, quindi **binding bidirezionale**.

Tre tipologie di componente form-like, in base a come il form raggiunge il suo obiettivo:
- **pannello di controllo**: sempre presente, i controlli riflettono sulle view di altri componenti; la delega al genitore avviene a quasi ogni cambio di stato (es. filtri);
- **modulo**: sempre presente, l'azione si compie alla conferma; la delega avviene solo al pulsante di conferma, il resto è stato interno (es. iscrizione a una newsletter in una colonna laterale);
- **modal dialog**: non presente nella view, appare per chiedere dati finché non si conferma o annulla; la visibilità spetta al genitore (un componente invisibile non può cambiare il proprio stato); la delega serve sia per confermare sia per annullare (es. dati di pagamento di un ordine).

## Il ciclo di rendering

1. **Inizializzazione**: `App()` e i componenti vengono eseguiti; React determina gli state di ogni componente, le dipendenze fra state+props del componente e le props dei figli, poi esegue il rendering effettivo, compreso quello dei figli.
2. **Cambiamento di stato**: si invoca il setter di una variabile di stato; React determina a quale componente appartiene, le conseguenze su altre variabili/espressioni della funzione di rendering, quali props dei figli risultano modificate; re-rendering effettivo **solo di ciò che è cambiato** (elementi montati/smontati, elementi con attributi cambiati, figli con props cambiate).
3. **Cosa provoca i cambiamenti di stato**: callback UI, funzioni asincrone (es. `fetch`), effetti.

## `useEffect`

Un **effetto** è codice da eseguire in conseguenza di cambiamenti di props o state ma che non può stare nella funzione di rendering, perché influisce sul mondo esterno (violerebbe la purezza), può causare un rerendering, o interferisce col rendering. Esempi: **fetch** del dato necessario alla vista (il caso più comune), sottoscrizione a un servizio esterno con callback, logging su DB locale, animazioni.

`useEffect(funzioneEffetto, dipendenzeEffetto?)`. La funzione viene messa nella coda degli eventi asincroni, quindi gira a rendering finito. L'array delle dipendenze elenca le variabili dello scope locale il cui cambiamento richiede di rieseguire l'effetto:
- array **vuoto** `[]`: l'effetto gira solo al primo rendering;
- array **non specificato**: gira a ogni rendering.

**Effetto + setState.** Dentro un effetto si può chiamare il setter (è asincrono) ma attenzione ai loop infiniti tra cambi di stato e dipendenze. Caso tipico: al primo rendering (e talvolta a un cambio di prop) il componente fa una fetch in un effetto F, all'arrivo (`then`) chiama `setState(dato)`, che provoca un rerendering. **Le dipendenze di F non devono contenere `dato`**, né altre variabili calcolate da `dato`, altrimenti F si riesegue in loop.

**Funzione di cleanup.** L'effetto può restituire un'altra funzione (senza argomenti né ritorno) che annulla, invalida o termina l'effetto. React la chiama **subito prima di rieseguire lo stesso effetto** e **subito prima di smontare il componente**. Serve per: sottoscrizioni o connessioni da chiudere prima di rifarle (evita notifiche in eccesso e memory leak); evitare che una fetch vecchia, finita dopo una nuova, sovrascriva il dato corretto (schema: scelta = A, `valid = true`, parte fetch A; scelta cambia in B → cleanup `valid = false` → nuovo effetto `valid = true`, parte fetch B; quando arriva A si controlla `valid`); evitare di modificare uno state di un componente già smontato (React rimuove gli state dei componenti non più validi). In sviluppo React **esegue ogni effetto due volte di fila** per far emergere le pulizie mancanti.

## Cose da saper fare all'esame `[INT]`

- Disegnare i tre strati e dire a quale flusso corrisponde un caso d'uso (1, 2a, 2b, 2c, 3) e dove collocare la business logic.
- Spiegare perché React è una libreria e che cosa fa il runtime; confrontare con un framework (Angular, Vue, Next.js).
- Definire componente, gerarchia, genitore/figlio, show/hide contro mount/unmount, SPA con e senza router.
- Spiegare MVVM e i due binding; scrivere un componente funzionale con props tipate e destructuring.
- Spiegare perché lo stato non può essere una variabile locale e perché il setter non si chiama nel corpo del componente; scrivere un `useState` con funzione di aggiornamento.
- Scrivere un form controllato con `value` + `onChange`; distinguere pannello di controllo, modulo e modal dialog.
- Scrivere un `useEffect` con fetch e array di dipendenze corretto, e spiegare il cleanup con il flag `valid`.

## Dubbi

- Il router e la gestione di Spring/JPA, sessioni e credenziali non sono ancora nelle slide `[DA VERIFICARE]`.
- Slide "Il ciclo di rendering" e gli schemi di `useEffect` hanno figure persi nell'estrazione del testo: ricontrollare sul PDF originale per i dettagli.
