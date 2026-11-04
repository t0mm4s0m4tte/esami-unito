# 01 – Uno sguardo di insieme alle applicazioni web

Fonti: Drive › Terzo anno › Tecnologie Web › Slide Deck - TWeb › "01 – Uno Sguardo di Insieme alle Applicazioni Web" (Picardi); Moodle 2026/27 › `01-TWeb-IntroTeoria-01-2627-ppt.pdf` (45 slide, aggiornate all'8 ottobre 2026), da cui viene la sezione "Server, web server, web service e porte" (slide 8-11), assente nel deck di Drive. Il resto del deck di Moodle ha gli stessi argomenti, che non ho riconfrontato slide per slide [DA VERIFICARE].

## Mappa degli argomenti

- [Cos'è il Web e come si indirizzano le risorse](#cosè-il-web-e-come-si-indirizzano-le-risorse)
- [Server, web server, web service e porte](#server-web-server-web-service-e-porte)
- [HTML e ipertesto](#html-e-ipertesto)
- [Risorse statiche, dinamiche e programmi lato server](#risorse-statiche-dinamiche-e-programmi-lato-server)
- [Anatomia di una transazione HTTP](#anatomia-di-una-transazione-http)
- [Pagine passive, interattive e programmi lato client](#pagine-passive-interattive-e-programmi-lato-client)
- [Struttura di richieste e risposte HTTP](#struttura-di-richieste-e-risposte-http)
- [L'applicazione web completa: client-side + server-side ("full stack")](#lapplicazione-web-completa-client-side--server-side-full-stack)
- [Front-end e back-end, architettura a strati](#front-end-e-back-end-architettura-a-strati)
- [Integrazioni da fonti esterne](#integrazioni-da-fonti-esterne)
- [Cose da saper fare all'esame](#cose-da-saper-fare-allesame)
- [Dubbi](#dubbi)

## Cos'è il Web e come si indirizzano le risorse

Il World Wide Web è un servizio di Internet di tipo client/server, costruito sopra il protocollo applicativo HTTP, pensato per pubblicare e condividere contenuti digitali. Il client (tipicamente un browser) manda una richiesta HTTP, il Web Server risponde. All'origine il Web serviva a condividere documenti e media, quindi la richiesta serviva soprattutto a dire *quale* risorsa si vuole e la risposta a *contenerla*, con le meta-informazioni che permettono al client di mostrarla bene.

<p align="center">
<img src="img/01-client-server-web.png" width="680" alt="Web client (per esempio un browser) e web server che si scambiano una richiesta HTTP e una risposta HTTP"><br>
<sub><i>Figura: il Web come servizio client/server basato su HTTP (slide 2).</i></sub>
</p>


Le richieste sono di due famiglie. Con una **GET** si chiede una risorsa, quindi serve poterla individuare in modo univoco. Con una **POST** si inviano dati al server perché li elabori o li conservi: la destinazione è ancora una "risorsa", ma di tipo particolare, cioè uno script o un'applicazione capace di eseguire un calcolo sui dati ricevuti.

Per individuare le risorse si usa la **URL**, una stringa con formato preciso: `protocol://hostname[:port]/path[?querystring][#fragment]`, per esempio `http://www.organizzazione.it:80/utenti/profilo?name=picardi#phone`. I componenti sono:
- *protocol*: dice a quale protocollo applicativo ci si riferisce (HTTP, ma anche FTP o SMTP: le URL non sono esclusive del Web);
- *host*: individua il server ed è fatto di sottodominio, dominio di secondo livello e dominio di primo livello (`www`, `organizzazione`, `it`); con "dominio" si intende di solito la coppia secondo+primo livello, acquistata presso un provider e registrata presso il Registro del TLD per garantirne l'unicità;
- *porta*: serve perché su uno stesso host possono rispondere più servizi, ciascuno su un proprio canale numerico; di norma non si scrive perché è implicita (80 per HTTP, 443 per HTTPS, la versione cifrata);
- *path*: può essere un vero percorso di cartelle fino a un file, oppure un percorso logico che identifica una funzione di un'applicazione; i segmenti sono separati da `/`;
- *query string*: testo opzionale di coppie chiave=valore separate da `&`, che comincia con `?`;
- *fragment* (dopo `#`): indica un punto preciso dentro una pagina HTML a cui il browser deve posizionarsi.

HTTP lavora sopra TCP e IP, che identificano gli host con indirizzi numerici. Per questo prima di spedire la richiesta il browser interroga il **DNS** (Domain Name System), il servizio applicativo che traduce lo hostname nell'indirizzo IP; solo dopo la richiesta HTTP può partire.

## Server, web server, web service e porte

*Sezione dalle slide 8-11 del deck Moodle 2026/27.*

Un **server** è un sistema, in genere un programma eseguito su un computer, che fornisce un servizio ad altri programmi detti client: il termine è molto generale. Ci sono web server (richieste HTTP/HTTPS), mail server, database server, file server, DNS server, e uno stesso computer può eseguirne molti insieme e quindi offrire più servizi. Il **browser**, dal canto suo, è un'applicazione che contiene dei motori: quello di rendering per HTML/CSS e quello JavaScript. Un **web server** è un server che parla soprattutto HTTP e HTTPS: riceve richieste e restituisce risposte, per esempio una pagina HTML, immagini, CSS e JavaScript, oppure l'accesso a delle API. Esempi: Apache, Nginx, Microsoft IIS, Tomcat.

Un **web service** (servizio web) *non* è la stessa cosa di un web server: è una funzionalità accessibile in rete che permette a due applicazioni di comunicare e scambiarsi dati, l'esempio tipico è un'API REST (una richiesta `GET /api/prodotti` che riceve i prodotti in JSON). Il web service viene reso disponibile attraverso un web server.

L'indirizzo IP identifica una macchina in una rete, ma la stessa macchina esegue più servizi: per distinguerli si usano le **porte**, e quando una richiesta arriva a un certo IP e a una certa porta il sistema operativo sa a quale programma consegnarla. Porte comuni: 80 HTTP, 443 HTTPS, 22 SSH, 25 e 587 SMTP, 5432 PostgreSQL. Una porta però non identifica per forza un singolo web service: servizi diversi possono stare dietro lo stesso web server e la stessa porta, e il server li distingue dal **percorso dell'URL** (`https://example.com:443/api/utenti` e `.../api/prodotti` usano entrambi la 443).

Le slide usano come esempio le tecnologie che userai nel progetto. **Spring Boot** non è di per sé un web server: è un framework Java per applicazioni, spesso di back-end, che normalmente include un web server integrato, molto spesso **Tomcat**; Tomcat riceve le richieste HTTP e le passa all'applicazione. Durante lo sviluppo risponde di solito su `http://localhost:8080`. Il percorso di una richiesta è: browser → HTTP → Tomcat → Spring Boot (Controller → Service → Database), dove il Controller è il componente che gestisce certe richieste HTTP. **React** invece non è un web server ma una libreria JavaScript per costruire l'interfaccia grafica; in sviluppo, però, strumenti come **Vite** avviano un development server che permette al browser di raggiungere l'applicazione React, di solito su `http://localhost:5173`. Si hanno quindi due server HTTP distinti su due porte diverse (React/Vite sulla 5173, Spring Boot/Tomcat sulla 8080). Quando React deve mostrare i prodotti fa una richiesta `GET http://localhost:8080/api/prodotti`; Spring Boot interroga il database se serve e risponde in genere con un JSON, che React mostra all'utente. In produzione React e Spring Boot possono usare la stessa porta, perché i file di React possono essere serviti dallo stesso server del back-end.

Questo spiega anche perché in sviluppo comparirà il problema della **Same-Origin Policy** descritto più sotto: le due porte sono due origini diverse.

## HTML e ipertesto

Prima del Web, in ambito scientifico si scambiavano risorse via Internet in modo macchinoso: bisognava conoscere l'host, collegarsi e scorrere le cartelle, ognuna organizzata a modo suo, e ripetere tutto per ogni riferimento a un altro documento. Il Web nasce alla fine degli anni '80 con due idee: le URL, che danno a ogni risorsa un indirizzo uniforme indipendente da dove sta, e i documenti **ipertestuali e multimediali**. Un ipertesto è un testo "multidimensionale", senza un'unica sequenza di lettura obbligata, in cui ci sono rimandi ad altre parti o ad altri testi.

<p align="center">
<img src="img/01-ipertesto.png" width="540" alt="Rete di documenti HTML collegati fra loro e a immagini, video, audio e PDF tramite link"><br>
<sub><i>Figura: le pagine HTML formano una rete di documenti collegati (slide 9).</i></sub>
</p>


**HTML** (HyperText Markup Language) è un linguaggio di *markup*, cioè di annotazione, non di programmazione: non dice a un agente di calcolo come elaborare dati. Un documento HTML è plain text con delle etichette (tag) che dicono al browser come strutturare la pagina ("questo è un titolo", "questo è un paragrafo"), più i link ad altre pagine e ai media usati per mostrarla. Si affianca a **CSS**, che dice invece quale aspetto grafico avere. I link tra pagine sono ciò che rende le risorse "navigabili" e dà al Web il suo nome di ragnatela.

## Risorse statiche, dinamiche e programmi lato server

Una risorsa individuata da una URL è **statica** se corrisponde a un file già esistente che il server deve solo restituire (il path logico non deve per forza coincidere col path nel file system). È **dinamica** se viene calcolata al momento: la URL rimanda a un programma che esegue un calcolo e ne restituisce il risultato, usando i dati della richiesta HTTP, il file system, database o servizi di terze parti.

Un Web Server è un software installato sulla macchina host. Con le giuste estensioni può eseguire programmi in linguaggi tipicamente interpretati (JavaScript) o ibridi (Java, con una JVM al proprio interno): sono le applicazioni **server-side**. Quale funzione si attiva dipende da path, query string e, per le POST, dai dati nel body; questi stessi elementi sono l'input del calcolo. Alla fine l'applicazione impacchetta l'output in una risposta HTTP. Nel laboratorio si vedranno applicazioni server-side in JavaScript e Java, ma altri linguaggi comuni sono C#, PHP, Python, Ruby.

## Anatomia di una transazione HTTP

La sequenza è questa:
1. il browser riceve una URL: scritta dall'utente, cliccata da un link, oppure generata da una pagina HTML che richiede altre risorse (immagini, CSS, script: ognuna è un'altra richiesta), o ancora prodotta da codice client-side;
2. chiede l'IP al DNS e manda la richiesta a quell'indirizzo, con dentro la URL originale;
3. quando arriva all'host, un primo smistamento avviene sulla **porta**: serve verificare che sia davvero un web server a rispondere lì;
4. il server guarda il path e decide se la risorsa esiste e se è statica o dinamica. Se statica, la incapsula così com'è; se dinamica, esegue il programma e incapsula l'output (anche quando lo scopo non è produrre un output, per esempio salvare un dato in un DB, l'esito dell'esecuzione va comunque comunicato). Se non esiste, risponde comunque con un codice di errore: il famoso *404 Not Found*;
5. il browser "spacchetta" la risposta, riconosce il tipo di dato e lo mostra (HTML, immagini, audio, video, PDF), oppure propone il download se non sa gestirlo.

<p align="center">
<img src="img/01-transazione-http.png" width="560" alt="Sequenza di una transazione HTTP: il client chiede l'IP al DNS server, manda la richiesta al web server, che restituisce la risorsa statica o il risultato del calcolo"><br>
<sub><i>Figura: anatomia di una transazione HTTP (slide 14).</i></sub>
</p>


## Pagine passive, interattive e programmi lato client

Una pagina HTML è **passiva** se si può solo leggere e al massimo navigare tramite link. È **interattiva** quando il browser "fa qualcosa" (oltre a navigare) in risposta alle azioni dell'utente, e questo "fare qualcosa" è esecuzione di codice. Una buona interattività deve anche modificare l'aspetto della pagina per far capire che qualcosa è successo. Se l'effetto è solo sulla visualizzazione l'interattività è *semplice*; se è una vera elaborazione che può generare richieste a un server si parla di **applicazione client-side**.

Ogni browser contiene un motore di esecuzione JavaScript. Quando arriva una pagina con uno script, il browser lo esegue: l'effetto principale è popolare l'ambiente di esecuzione con funzioni da eseguire in risposta a eventi (click, movimenti del mouse, compilazione di campi, scroll). L'ambiente offre a queste funzioni una rappresentazione a oggetti della pagina (il DOM) che esse possono modificare, cambiando così ciò che l'utente vede. Può anche mantenere variabili di stato: in questo modo la pagina diventa la user interface di un'applicazione. Il codice client-side può fare richieste HTTP, con una particolarità: la risposta non è gestita dal browser ma ricevuta dal codice che ha fatto la richiesta, che la elabora e aggiorna la pagina. Il browser esegue solo JavaScript, quindi ciò che è scritto in altri linguaggi va tradotto in JS.

<p align="center">
<img src="img/01-ruolo-browser.png" width="560" alt="Il browser riceve la pagina HTML, richiede le risorse necessarie, esegue gli script JavaScript e reagisce agli eventi utente e di rete"><br>
<sub><i>Figura: il ruolo del browser (slide 19).</i></sub>
</p>


### Il ciclo del browser

1. Il browser ottiene una pagina HTML con una transazione precedente.
2. Scopre di quali altre risorse ha bisogno per il rendering (fogli di stile, immagini, script) e le richiede.
3. Appena ha il necessario fa il **rendering**. Se la pagina è passiva il ciclo finisce qui.
4. Se ci sono script, li esegue (di solito li si fa partire solo a pagina caricata).
5. Da lì arrivano gli eventi, di due tipi: **eventi utente** (click, tasti, scroll, tap) ed **eventi di rete** (arrivo della risposta a una richiesta HTTP, timeout).
6. Quando un evento arriva, il browser attiva la funzione registrata per esso, che può generare nuove richieste HTTP e/o modificare la pagina.

Le richieste HTTP sono **non bloccanti**: partono e restituiscono subito il controllo, mentre la gestione della risposta è affidata a una funzione registrata in anticipo. Quando la pagina cambia, il browser la ri-renderizza.

## Struttura di richieste e risposte HTTP

Ogni messaggio ha una **start line**, gli **header** (coppie `nome: valore`), una riga vuota e un **body** (detto anche payload). Start line e header sono le meta-informazioni.

<p align="center">
<img src="img/01-richiesta-risposta.png" width="560" alt="Messaggio di richiesta e messaggio di risposta HTTP affiancati, con start line, header, riga vuota e body"><br>
<sub><i>Figura: struttura di richiesta e risposta HTTP (slide 22; da Ackermann, Full Stack Web Development, 2023).</i></sub>
</p>


Nella **richiesta** la start line contiene il metodo HTTP (GET: il server deve fornire informazioni; POST: il server deve elaborare i dati ricevuti), la destinazione (URL completa o solo path assoluto, perché l'host è quello che sta ricevendo) e la versione del protocollo. Molti header sono impostati dal browser; quelli più spesso impostati dal programmatore sono `Accept` (tipi accettati in risposta), `Content-Type` (tipo del body) e `User-Agent` (che client è). I tipi si esprimono in formato **MIME**, nato per gli allegati email, nella forma `tipo/sottotipo`: `text/html`, `text/plain`, `text/css`, `text/javascript`, `image/png`, `image/jpeg`, `application/json`, `application/pdf`, `application/xml`, `video/mp4`. Le GET non hanno body, perché servono a ottenere dati. Le POST hanno un body, tipicamente `application/x-www-form-urlencoded` (i dati di un form inviato dal browser) oppure `application/json` (dati elaborati da un'app client-side).

<p align="center">
<img src="img/01-request-line.png" width="560" alt="Richiesta HTTP POST con la start line scomposta in metodo, destinazione e versione"><br>
<sub><i>Figura: la start line di una richiesta HTTP (slide 24).</i></sub>
</p>


Nella **risposta** la start line contiene versione, **status code** e status text. Lo status code è diviso in categorie dalla prima cifra: 2XX successo, 4XX errore del client, 5XX errore del server (esempio: 404, "Not found"). Header comuni: `Content-Type`, `Content-Length` (lunghezza del body in byte), `Server`, `Date`. Il body c'è sempre ed è tipicamente `text/html` (la pagina da mostrare), `application/json` (dati per un'app client-side) oppure fogli di stile, script, immagini e altri file.

<p align="center">
<img src="img/01-response-line.png" width="560" alt="Risposta HTTP 201 Created con la status line scomposta in versione, status code e status text"><br>
<sub><i>Figura: la status line di una risposta HTTP (slide 28).</i></sub>
</p>


## L'applicazione web completa: client-side + server-side ("full stack")

Il Web Server è sempre online (salvo manutenzione). Pubblica in modo statico la pagina HTML a cui sono collegati gli script client-side e, in parallelo, tiene in esecuzione l'app server-side, "avviata" dal web server all'accensione così come le app desktop sono avviate dal sistema operativo. L'app server-side risponde a un insieme di URL, ciascuna corrispondente a una funzionalità: quell'insieme è la sua **API**.

<p align="center">
<img src="img/01-full-stack.png" width="560" alt="Applicazione web completa: il browser esegue l'app client-side, il web server esegue l'app server-side che accede a un database, collegate da richieste e risposte HTTP"><br>
<sub><i>Figura: applicazione client-side + server-side (slide 33).</i></sub>
</p>


Il browser richiede la pagina HTML, scarica media e script, renderizza e avvia l'ambiente JavaScript: questo è l'avvio dell'app client-side. La memoria dell'ambiente non si resetta finché la pagina resta aperta, e ogni pagina ha un ambiente indipendente.

Quando l'utente interagisce, l'evento attiva una funzione dell'app client-side. Se questa deve ottenere o salvare dati, fa una richiesta HTTP a una URL dell'API server-side (che l'app client deve conoscere); la richiesta non blocca, e conviene che le elaborazioni siano brevi perché l'app client è **single-thread** (esistono i Web Workers per i thread in background ma nel corso non si vedono) e un'elaborazione lunga dà la sensazione di interfaccia bloccata.

Il web server inoltra la richiesta all'app server-side, che guarda metodo, path, query string, eventuale body e il suo `Content-Type` per capire cosa fare. Quasi sempre interagisce con un **database server** per leggere o salvare informazioni. Il risultato diventa il body della risposta, con il `Content-Type` corretto. Il server deve poter gestire molte richieste in parallelo: la maggior parte dei web server crea un thread per ogni richiesta, mentre **Node.js** usa un unico loop a thread singolo con elaborazioni non bloccanti (asincrone).

La risposta torna al browser, che la traduce in un evento di rete per l'app client; la funzione registrata al momento della richiesta viene richiamata e può fare altre richieste e/o modificare la pagina. A quel punto il browser ri-renderizza solo le parti modificate, ed è questo che rende fluida l'interazione; la ri-renderizzazione avviene quando l'ambiente JS rilascia il thread.

## Front-end e back-end, architettura a strati

L'**architettura a strati** è un pattern in cui la logica applicativa è divisa in livelli, ciascuno dei quali offre servizi al livello superiore usando quelli del livello inferiore. Il concetto centrale è l'**interfaccia** tra due strati, che definisce formalmente le richieste accettabili (input) e le risposte attese (output). Ne derivano due vantaggi: ogni strato può essere modificato, corretto e rilasciato separatamente se l'interfaccia verso l'alto non cambia; e sullo stesso strato possono convivere moduli alternativi che implementano la stessa interfaccia, intercambiabili senza toccare lo strato superiore.

La suddivisione più comune: il **back-end** è il livello più basso e gestisce la **persistenza dei dati** (nelle app desktop è uno strato sottile che carica/salva file; nelle web app dialoga con un database server, traduce tra la rappresentazione dei dati nell'app e quella del DB, per esempio i record di una tabella relazionale, e può occuparsi di verifiche, transazioni e rollback). Il **front-end** è il livello più alto e interagisce con l'utente: contiene la User Interface (che è a tutti gli effetti l'interfaccia verso lo "strato superiore", che qui è una persona) e i moduli che elaborano le informazioni per renderle fruibili. Possono esserci **strati intermedi** per elaborazioni che non appartengono né all'uno né all'altro e che si vogliono rendere indipendenti da entrambi.

<p align="center">
<img src="img/01-front-back-end.png" width="520" alt="Tre strati dell'applicazione: il front-end che interagisce con l'utente in alto, il back-end che gestisce la persistenza dei dati in basso, strati intermedi in mezzo"><br>
<sub><i>Figura: front-end e back-end (slide 38).</i></sub>
</p>


Nelle web app il front-end è necessariamente lato client (solo lì c'è la UI); il back-end è tipicamente, ma non per forza, lato server; gli strati intermedi possono stare da una parte o dall'altra. Elaborare lato client non sovraccarica il server, ma è possibile solo se i dati che servono sono già sul client (e può non esserlo per riservatezza o perché sono troppi da trasferire); elaborare lato server dà accesso a tutto il database ma, se l'elaborazione è costosa, crea problemi di scalabilità. Spesso si usa "back-end" come sinonimo di server-side e "front-end" di client-side, ma attenzione: i primi due termini descrivono l'organizzazione *logica*, gli altri due quella *implementativa*. (La slide originale scrive i sinonimi al contrario; vedi Dubbi.)

## Integrazioni da fonti esterne

*Questa parte non è nelle slide: viene da MDN, dalla documentazione ufficiale di Node.js e dall'RFC 9110, e serve a colmare cose che il corso lascia aperte. L'elenco esatto dei metodi safe e idempotenti viene dall'indice dell'RFC e da MDN ma non ho riletto il testo delle sezioni 9.2.1 e 9.2.2: da ricontrollare prima di impararlo a memoria.*

### Altri metodi HTTP, status code, REST

Oltre a GET e POST, HTTP definisce:
- **PUT**: sostituire la risorsa all'URL indicato con il corpo inviato;
- **DELETE**: eliminarla;
- **PATCH**: modificarla parzialmente.

Un metodo è *safe* se non è pensato per cambiare lo stato del server (GET, HEAD, OPTIONS) ed è *idempotente* se ripetere la stessa richiesta più volte ha lo stesso effetto di una sola: lo sono tutti i safe più PUT e DELETE. POST non è né safe né idempotente (due POST uguali possono creare due risorse) e PATCH in generale non è idempotente. L'idempotenza conta perché dopo un errore di rete il client può riprovare in sicurezza.

Gli status code più comuni, oltre alla regola delle classi (1xx informativi, 2xx successo, 3xx redirezione, 4xx errore del client, 5xx errore del server):

| Codice | Significato |
|---|---|
| 200 OK | successo generico |
| 201 Created | risorsa creata (tipico dopo una POST) |
| 301 / 302 | redirect permanente / temporaneo (nuovo URL nell'header `Location`) |
| 400 Bad Request | richiesta malformata |
| 401 Unauthorized | in realtà "non autenticato" |
| 403 Forbidden | autenticato ma non autorizzato |
| 404 Not Found | risorsa inesistente |
| 500 Internal Server Error | errore generico del server |

Un'API **REST** modella il dominio come risorse identificate da URL (`/articoli/42`) e le manipola con i metodi HTTP standard, senza stato di sessione sul server tra una richiesta e l'altra e in genere con rappresentazioni JSON. In pratica molte API dette "REST" non rispettano tutti i vincoli originali di Fielding (lo ammette anche MDN).

### Concorrenza lato server: thread-per-richiesta contro event loop

Nel modello tradizionale (per esempio Apache con thread) ogni connessione è servita da un thread dedicato: il codice è lineare e semplice, e se un thread aspetta disco o database il sistema operativo lo sospende mentre gli altri proseguono. Il costo è memoria e cambi di contesto per ogni connessione, che limitano i client contemporanei. **Node.js** sceglie l'opposto: un solo thread esegue tutto il codice JavaScript con un event loop, e l'I/O di rete e di file è non bloccante, delegato al kernel o a un piccolo pool di worker interni (libuv); quando termina si accoda una callback. Con pochissimi thread si gestiscono moltissime connessioni inattive, ideale per carichi *I/O-bound*. Il prezzo è che il codice sincrono lungo o il calcolo pesante non viene interrotto da nessuno e **blocca tutti i client** finché non finisce (la documentazione ufficiale lo dice chiaramente); si rimedia spezzando il lavoro o spostandolo in worker thread. "Single-thread" va letto come "un solo thread per il codice JS": Node ha comunque thread interni.

### Same-Origin Policy e CORS

Un'**origine** è la combinazione di protocollo, host e porta (il percorso non conta; anche solo `http` contro `https` o una porta diversa danno origini diverse). La **Same-Origin Policy** è una regola del browser: uno script caricato da un'origine non può *leggere* le risposte di un'altra. Non vieta di incorporare risorse altrui (immagini, script da una CDN) né di inviare form: vieta di leggerne il contenuto da JavaScript. Un `fetch` verso un altro dominio quindi parte, ma lo script riceve la risposta solo se il server la autorizza. **CORS** è il meccanismo con cui il server rilassa la regola, rispondendo con `Access-Control-Allow-Origin`. Per richieste "non semplici" (PUT/DELETE, header personalizzati, `Content-Type: application/json`) il browser manda prima una richiesta di *preflight* `OPTIONS`. Un errore CORS si risolve quindi lato server, non nel client, ed è una protezione del browser (curl non la applica). Lo incontreremo quando il front-end farà `fetch` verso un back-end con origine diversa, per esempio porte diverse in sviluppo.

## Cose da saper fare all'esame

- Scomporre una URL nei suoi componenti e spiegare cosa fa ciascuno, incluso il ruolo di DNS e porta.
- Descrivere passo per passo cosa succede dal momento in cui l'utente digita una URL a quando vede la pagina, distinguendo risorsa statica, dinamica e inesistente.
- Distinguere GET e POST, indicare quali header e quale body usano, e riconoscere le categorie di status code.
- Spiegare la differenza tra pagina passiva e interattiva e tra interattività semplice e applicazione client-side.
- Descrivere il flusso completo di un'app full stack (evento utente → richiesta asincrona → app server-side + DB → risposta → evento di rete → aggiornamento della pagina) e perché le richieste sono non bloccanti.
- Spiegare vantaggi dell'architettura a strati e differenza tra front/back-end (logico) e client/server-side (implementativo).

## Dubbi

- La slide dice che "spesso back-end è sinonimo di client-side e front-end di server-side": è un refuso. Nell'uso standard è il contrario: front-end ≈ client-side (ciò che gira nel browser) e back-end ≈ server-side.
- Il confronto thread-per-richiesta contro event loop di Node.js è solo accennato nelle slide ("lo vedremo meglio in seguito"): ho aggiunto la sezione "Integrazione" qui sopra da fonti esterne, da riconfrontare con ciò che dirà il docente.
