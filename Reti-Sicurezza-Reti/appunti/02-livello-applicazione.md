# 02 – Livello di applicazione

Fonte: Drive › Terzo anno › Reti e Sicurezza delle Reti › Slide Deck - Reti › "02 - Livello di Applicazione" (Kurose-Ross, *Computer Networking: A Top-Down Approach*, 8a ed., cap. 2). Il PDF caricato si ferma alla slide sulla distribuzione di file client-server contro P2P: restano da coprire P2P, video streaming e CDN, socket programming con UDP e TCP. `[INT]` = integrazione o calcolo mio, non presente nelle slide: da ricontrollare. `[DA VERIFICARE]` = punto ambiguo.

## Mappa degli argomenti

Il capitolo sale dal livello più alto dello stack: prima i principi comuni a tutte le applicazioni di rete (architetture, processi e socket, servizi di trasporto richiesti), poi i protocolli applicativi veri e propri, cioè HTTP per il Web, SMTP e IMAP per la posta e il DNS per i nomi.

- [Principi delle applicazioni di rete](#principi-delle-applicazioni-di-rete)
- [Servizi di trasporto e protocolli di Internet](#servizi-di-trasporto-e-protocolli-di-internet)
- [Web e HTTP](#web-e-http)
- [Cookie](#cookie)
- [Web cache e GET condizionale](#web-cache-e-get-condizionale)
- [Da HTTP/1.1 a HTTP/3](#da-http11-a-http3)
- [Posta elettronica: SMTP e IMAP](#posta-elettronica-smtp-e-imap)
- [DNS](#dns)
- [Distribuzione di file: client-server contro P2P](#distribuzione-di-file-client-server-contro-p2p)
- [Cose da saper fare all'esame](#cose-da-saper-fare-allesame)
- [Dubbi](#dubbi)

## Principi delle applicazioni di rete

**Definizione.** Scrivere un'applicazione di rete significa scrivere programmi che girano su end system diversi e comunicano attraverso la rete, per esempio il web server e il browser. **Intuizione.** Il nucleo della rete non esegue applicazioni utente: tutta la logica sta ai bordi, ed è questo che permette di sviluppare e diffondere nuove applicazioni rapidamente.

Le architetture sono due. Nel paradigma **client-server** il server è un host sempre acceso con indirizzo IP permanente, spesso in data center per poter scalare; i client lo contattano, possono essere connessi a intermittenza, avere IP dinamici e non comunicano direttamente fra loro (esempi: HTTP, IMAP, FTP). Nell'architettura **peer-to-peer** non c'è un server sempre attivo: sistemi terminali arbitrari comunicano direttamente, ciascuno chiedendo servizio agli altri e fornendone a sua volta. Il vantaggio è l'*auto-scalabilità* (ogni nuovo peer porta capacità di servizio oltre che domanda), il prezzo è una gestione complessa perché i peer sono connessi a intermittenza e cambiano IP (esempio: BitTorrent).

<div align="center">

<table>
<tr>
<td align="center"><img src="img/02-client-server.png" height="300" alt="Architettura client-server: un server centrale sempre acceso comunica con i client"><br><sub>Client-server (slide 5).</sub></td>
<td align="center"><img src="img/02-p2p.png" height="300" alt="Architettura peer-to-peer: gli host comunicano direttamente fra loro"><br><sub>Peer-to-peer (slide 6).</sub></td>
</tr>
</table>

</div>


A comunicare sono **processi**, cioè programmi in esecuzione su un host. Due processi sullo stesso host comunicano con la comunicazione interprocesso definita dal sistema operativo, processi su host diversi si scambiano **messaggi**. Il processo che inizia la comunicazione è il *client*, quello che aspetta di essere contattato è il *server*; anche le applicazioni P2P hanno processi di entrambi i tipi. Un processo manda e riceve messaggi tramite la propria **socket**, analoga a una porta: il mittente spinge il messaggio fuori dalla porta e conta sull'infrastruttura di trasporto dall'altro lato per farlo arrivare alla socket del ricevente. Sono quindi due le socket coinvolte, una per lato; la parte sopra la socket è controllata dallo sviluppatore, quella sotto (trasporto, rete, link, fisico) dal sistema operativo.

<p align="center">
<img src="img/02-socket.png" width="640" alt="Due host con lo stack di protocolli, ciascuno con una socket tra processo applicativo e trasporto, collegati da Internet"><br>
<sub><i>Figura: la socket separa ciò che controlla lo sviluppatore da ciò che controlla il sistema operativo (slide 8).</i></sub>
</p>


Per ricevere messaggi un processo ha bisogno di un identificatore. L'indirizzo IP a 32 bit dell'host non basta, perché sullo stesso host girano molti processi: l'identificatore è la coppia **indirizzo IP + numero di porta**. Porte note: 80 per un server HTTP, 25 per un mail server. Per mandare una richiesta HTTP al web server `gaia.cs.umass.edu` si usano IP 128.119.245.12 e porta 80.

## Servizi di trasporto e protocolli di Internet

Un'applicazione sceglie il protocollo di trasporto in base a quattro esigenze: **integrità dei dati** (trasferimento file e transazioni web vogliono affidabilità al 100%, l'audio tollera qualche perdita), **timing** (telefonia Internet e giochi interattivi hanno bisogno di basso ritardo), **throughput** (le applicazioni multimediali richiedono un minimo, quelle "elastiche" usano quello che trovano) e **sicurezza** (cifratura, integrità).

| Applicazione | Perdita dati | Throughput | Sensibile al tempo |
|---|---|---|---|
| Trasferimento file, e-mail, documenti Web | no | elastico | no |
| Audio/video in tempo reale | tollerante | audio 5 kbps-1 Mbps, video 10 kbps-5 Mbps | sì, decine di ms |
| Streaming audio/video | tollerante | come sopra | sì, pochi secondi |
| Giochi interattivi | tollerante | kbps o più | sì, decine di ms |
| Messaggistica | no | elastico | sì e no |

Internet offre due protocolli di trasporto. **TCP** garantisce trasporto affidabile fra processo mittente e ricevente, **controllo di flusso** (il mittente non travolge il ricevente) e **controllo di congestione** (il mittente viene rallentato se la rete è sovraccarica); è orientato alla connessione, quindi richiede un setup fra client e server, e non offre timing, throughput minimo o sicurezza. **UDP** fornisce trasferimento non affidabile e non offre nulla di tutto ciò, né il setup di connessione. Le slide chiedono "perché esiste UDP?" senza rispondere: la risposta è nei capitoli sul trasporto.

| Applicazione | Protocollo applicativo | Trasporto |
|---|---|---|
| Trasferimento file | FTP (RFC 959) | TCP |
| E-mail | SMTP (RFC 5321) | TCP |
| Web | HTTP (RFC 7230, 9110) | TCP |
| Telefonia Internet | SIP (RFC 3261), RTP (RFC 3550) o proprietari | TCP o UDP |
| Streaming | HTTP, DASH | TCP |
| Giochi interattivi | proprietari (WoW, FPS) | UDP o TCP |

TCP e UDP "puri" non cifrano: una password in chiaro passata alla socket attraversa Internet in chiaro. **TLS** (Transport Layer Security) offre connessioni TCP cifrate con integrità dei dati e autenticazione degli endpoint ed è implementato a livello di applicazione: le applicazioni usano librerie TLS, che a loro volta usano TCP, e ciò che esce dalla socket viaggia cifrato (approfondimento nel capitolo 8).

## Web e HTTP

Una pagina web è fatta di **oggetti** (file HTML, immagini JPEG, applet, audio...) possibilmente su server diversi, di solito un file HTML base che ne referenzia altri; ogni oggetto è indirizzato da una **URL** (nome host + path, per esempio `www.someschool.edu/someDept/pic.gif`).

**HTTP** (hypertext transfer protocol) è il protocollo applicativo del Web, con architettura client-server: il client è il browser che richiede, riceve e mostra gli oggetti, il server è il web server che li invia. HTTP usa **TCP**: il client apre la connessione (creando una socket) verso la porta 80 del server, il server accetta, si scambiano messaggi HTTP e infine la connessione viene chiusa. Il protocollo è **stateless**: il server non conserva informazione sulle richieste passate. È una scelta voluta, perché i protocolli che mantengono *stato* sono complessi: bisogna conservare la storia e, se client o server cadono, riconciliare le loro viste che possono diventare incoerenti.

Le connessioni sono di due tipi. In HTTP **non persistente** si apre una connessione TCP, si invia al più un oggetto e si chiude, quindi scaricare più oggetti richiede più connessioni. In HTTP **persistente** (HTTP/1.1) la connessione resta aperta dopo la risposta e gli oggetti successivi viaggiano sulla stessa connessione.


<table>
<tr>
<td valign="top">

*Esempio non persistente dalle slide.* L'utente digita `www.someSchool.edu/someDepartment/home.index`, una pagina con testo e 10 immagini JPEG: il client apre la connessione TCP verso la porta 80 (1a), il server la accetta (1b), il client manda la richiesta con la URL (2), il server prepara la risposta con l'oggetto e la manda (3), poi chiude la connessione (4); il client riceve la risposta, interpreta l'HTML, trova le 10 immagini (5) e ripete i passi 1-5 per ciascuna (6).

Il **RTT** (*round-trip time*) è il tempo per un piccolo pacchetto per andare dal client al server e tornare. Per ogni oggetto in HTTP non persistente servono 1 RTT per aprire la connessione TCP, 1 RTT per la richiesta HTTP e i primi byte di risposta e il tempo di trasmissione del file:

`tempo di risposta non persistente = 2·RTT + tempo di trasmissione del file`

</td>
<td valign="top" align="center">

<img src="img/02-rtt.png" width="380" alt="Diagramma temporale di HTTP non persistente: un RTT per aprire la connessione TCP, un RTT per la richiesta, poi il tempo di trasmissione del file"><br><sub>Figura: tempo di risposta non persistente (slide 22).</sub>

</td>
</tr>
</table>


Questo costa 2 RTT per oggetto e overhead di sistema operativo per ogni connessione, e i browser reagiscono aprendo più connessioni parallele. Con HTTP persistente il server lascia aperta la connessione e il client manda le richieste appena incontra un oggetto referenziato: può bastare un solo RTT per tutti gli oggetti referenziati, dimezzando il tempo di risposta. `[INT]` Per la pagina dell'esempio (1 file base + 10 immagini, ignorando i tempi di trasmissione): non persistente seriale 11 · 2 = 22 RTT; persistente con richieste in pipeline 2 RTT per il file base (connessione + richiesta) più 1 RTT per tutte le immagini, cioè circa 3 RTT.

Un **messaggio di richiesta** HTTP è in formato ASCII leggibile: una *request line* (metodo, URL, versione, per esempio `GET /index.html HTTP/1.1`), poi righe di header (`Host`, `User-Agent`, `Accept`, `Accept-Language`, `Accept-Encoding`, `Connection: keep-alive`), e una riga vuota (CR LF) che chiude gli header; nelle POST segue il corpo. Ogni riga termina con CR e LF. I metodi sono:

1. `GET`: richiede un oggetto; si può passare dati nell'URL dopo il `?` (per esempio `www.somesite.com/animalsearch?monkeys&banana`);
2. `POST`: i dati di un form (input utente) viaggiano nell'*entity body*;
3. `HEAD`: chiede solo gli header che il server restituirebbe a un GET;
4. `PUT`: carica un nuovo oggetto sul server, sostituendo completamente quello all'URL indicato.

<p align="center">
<img src="img/02-richiesta-http.png" width="680" alt="Messaggio di richiesta HTTP con request line, righe di header e riga vuota finale"><br>
<sub><i>Figura: formato del messaggio di richiesta HTTP (slide 24).</i></sub>
</p>


Il **messaggio di risposta** ha una *status line* (versione, codice di stato, frase), righe di header (`Date`, `Server`, `Last-Modified`, `ETag`, `Accept-Ranges`, `Content-Length`, `Content-Type`...), riga vuota e infine i dati, per esempio il file HTML richiesto. I codici principali compaiono nella prima riga della risposta:

| Codice | Significato |
|---|---|
| 200 OK | richiesta riuscita, oggetto nel messaggio |
| 301 Moved Permanently | oggetto spostato, nuova posizione nel campo `Location` |
| 400 Bad Request | richiesta non compresa dal server |
| 404 Not Found | documento non trovato su questo server |
| 505 HTTP Version Not Supported | versione HTTP non supportata |

<p align="center">
<img src="img/02-risposta-http.png" width="640" alt="Messaggio di risposta HTTP con status line, righe di header e dati"><br>
<sub><i>Figura: formato del messaggio di risposta HTTP (slide 26).</i></sub>
</p>


Per provare HTTP dal lato client si può aprire una connessione TCP a mano con `nc -c -v gaia.cs.umass.edu 80`, digitare una GET minima (`GET /kurose_ross/interactive/index.php HTTP/1.1`, riga `Host: gaia.cs.umass.edu`, poi due invii) e leggere la risposta, oppure catturare lo scambio con Wireshark.

## Cookie

Poiché HTTP è stateless, i siti usano i **cookie** per mantenere un po' di stato fra le transazioni. Ci sono quattro componenti: una riga di header `Set-cookie` nella risposta HTTP, una riga `Cookie` nelle richieste successive, un file di cookie sull'host dell'utente gestito dal browser e un database back-end sul sito. Alla prima richiesta di un utente il sito crea un ID univoco (il "cookie") e una voce per quell'ID nel database; le richieste successive di quell'utente al sito contengono il valore del cookie e permettono al sito di riconoscerlo, anche a distanza di una settimana. Si usano per autorizzazione, carrelli degli acquisti, raccomandazioni e stato di sessione (web mail). Il problema generale è come tenere lo stato: o negli endpoint del protocollo, sui due lati, per più transazioni, oppure nei messaggi stessi, come fanno i cookie.

<p align="center">
<img src="img/02-cookie.png" width="560" alt="Scambio di messaggi HTTP fra client e server Amazon con set-cookie 1678, cookie nelle richieste successive e voce nel database back-end"><br>
<sub><i>Figura: cookie, cioè stato fra le transazioni (slide 30).</i></sub>
</p>


Il rischio riguarda la **privacy**. I cookie di *prima parte* vengono dal sito scelto dall'utente. Quelli di *terze parti* vengono da un sito che l'utente non ha scelto di visitare: se una pagina del NY Times incorpora un annuncio da AdX.com, il browser fa una GET anche verso AdX, che risponde con un `Set-cookie` proprio (per esempio 7493) e ottiene dal campo `Referrer` quale pagina sta guardando l'utente. Ritrovando lo stesso cookie 7493 su altri siti che usano AdX (nell'esempio, socks.com e poi di nuovo il NY Times), AdX ricostruisce la navigazione e può restituire pubblicità mirata (nell'esempio, calzini). Il tracciamento può anche essere invisibile, tramite un link nascosto invece di un annuncio. I cookie di terze parti sono disattivati di default in Firefox e Safari. In Europa il **GDPR** (recital 30, maggio 2018) considera gli identificatori online, cookie compresi, dati personali quando permettono di identificare una persona, quindi soggetti alla normativa; l'utente deve avere controllo esplicito sul loro uso.

## Web cache e GET condizionale


<table>
<tr>
<td valign="top">

Una **web cache** (o *proxy server*) soddisfa le richieste senza coinvolgere il server di origine. Il browser è configurato per mandare tutte le richieste alla cache: se l'oggetto è presente, lo restituisce; altrimenti lo chiede all'origine, lo salva e lo inoltra al client. La cache è quindi server per il client originario e client per l'origine, e il server di origine indica nell'header quali oggetti possono essere messi in cache. I vantaggi sono la riduzione del tempo di risposta (la cache è vicina al client), la riduzione del traffico sul link d'accesso dell'istituzione e, dato che Internet è piena di cache, la possibilità anche per content provider "poveri" di distribuire contenuti efficacemente.

*Esempio svolto dalle slide.* Un'istituzione ha un link d'accesso da 1,54 Mbps verso Internet e una LAN da 1 Gbps; l'RTT dal router istituzionale ai server di origine è 2 s; gli oggetti sono da 100 kbit e le richieste dei browser sono 15 al secondo, quindi il traffico verso i browser è 15 · 100 kbit = 1,50 Mbps. Allora l'utilizzazione del link d'accesso è 1,50 / 1,54 = **0,97** (ritardo di coda dell'ordine dei minuti), l'utilizzazione della LAN è **0,0015**, e il ritardo end-to-end è ritardo Internet + ritardo d'accesso + ritardo LAN = 2 s + minuti + microsecondi.

</td>
<td valign="top" align="center">

<img src="img/02-cache-esempio.png" width="460" alt="Rete istituzionale con LAN a 1 Gbps, router, link d'accesso a 1,54 Mbps verso Internet pubblica e server di origine"><br><sub>Figura: scenario dell'esempio della cache (slide 40).</sub>

</td>
</tr>
</table>


- *Opzione 1:* comprare un link d'accesso da 154 Mbps. L'utilizzazione scende a **0,0097** e il ritardo di accesso a millisecondi, ma è costoso.
- *Opzione 2:* installare una cache locale (economica). Con **hit rate 0,4** il 40% delle richieste è servito dalla cache in pochi ms e il 60% va all'origine. Il traffico sul link d'accesso è 0,6 · 1,50 = **0,9 Mbps**, utilizzazione 0,9 / 1,54 = **0,58**, con ritardo di coda in ms. Il ritardo medio è `0,6 · 2,01 + 0,4 · (~ms) ≈ 1,2 s`: inferiore al link da 154 Mbps e più economico.


<table>
<tr>
<td valign="top">

Il **GET condizionale** evita di rimandare un oggetto che il browser ha già in cache e aggiornato (nessun ritardo di trasmissione né uso di risorse di rete). Il client mette nella richiesta la data della copia in cache con `If-modified-since: <data>`. Se la copia è ancora valida il server risponde `304 Not Modified` senza oggetto, altrimenti risponde `200 OK` con i dati.

</td>
<td valign="top" align="center">

<img src="img/02-get-condizionale.png" width="340" alt="Due scambi di GET condizionale: risposta 304 Not Modified se l'oggetto non è cambiato, 200 OK con dati se è stato modificato"><br><sub>Figura: GET condizionale (slide 46).</sub>

</td>
</tr>
</table>


## Da HTTP/1.1 a HTTP/3

L'obiettivo di HTTP/2 è ridurre il ritardo nelle richieste di più oggetti. HTTP/1.1 aveva introdotto più GET in pipeline su una sola connessione TCP, ma il server risponde nell'ordine di arrivo (FCFS). Quindi un oggetto piccolo può dover aspettare dietro uno grande, ed è il **head-of-line (HOL) blocking**; inoltre la ritrasmissione dei segmenti TCP persi ferma la trasmissione di tutti gli oggetti.

**HTTP/2** (RFC 7540, 2015) lascia invariati metodi, codici di stato e la maggior parte degli header, ma dà al server più flessibilità: l'ordine di invio è deciso dalle **priorità** specificate dal client, non necessariamente FCFS; il server può fare **push** di oggetti non richiesti; gli oggetti sono divisi in **frame** la cui trasmissione viene intercalata, così il HOL blocking si attenua. Nell'esempio con un oggetto grande O1 e tre piccoli O2, O3, O4 richiesti dopo, HTTP/1.1 consegna in ordine e O2-O4 aspettano O1, mentre con HTTP/2 i frame sono interleaved e i piccoli arrivano subito con O1 appena in ritardo.

<div align="center">

<table>
<tr>
<td align="center"><img src="img/02-hol-http11.png" height="190" alt="HTTP/1.1: un oggetto grande O1 bloccante, O2, O3 e O4 consegnati dopo di lui"><br><sub>HTTP/1.1: O2, O3, O4 aspettano O1 (slide 49).</sub></td>
<td align="center"><img src="img/02-hol-http2.png" height="190" alt="HTTP/2: i frame degli oggetti sono intercalati, O2, O3 e O4 arrivano subito e O1 solo un po' dopo"><br><sub>HTTP/2: frame intercalati (slide 50).</sub></td>
</tr>
</table>

</div>


Resta il problema che HTTP/2 usa una sola connessione TCP: una perdita di pacchetti blocca ancora tutti gli oggetti (quindi i browser aprono comunque più connessioni parallele) e su TCP "puro" non c'è sicurezza. **HTTP/3** aggiunge sicurezza e controllo di errori e congestione *per oggetto* (più pipelining) sopra **UDP**, tema ripreso nel livello di trasporto.

## Posta elettronica: SMTP e IMAP


<table>
<tr>
<td valign="top">

La posta ha tre componenti: gli **user agent** (il "mail reader" per scrivere, modificare e leggere, come Outlook o il client dell'iPhone), i **mail server** (con la *mailbox* dei messaggi in arrivo dell'utente e la coda dei messaggi in uscita) e il protocollo **SMTP** fra mail server. Il mail server mittente fa da client e quello ricevente da server.

**SMTP** (RFC 5321) usa TCP sulla porta 25, con trasferimento diretto dal server mittente al server destinatario, in tre fasi: handshaking (saluto), trasferimento dei messaggi e chiusura. L'interazione è comando-risposta come in HTTP: i comandi sono testo ASCII, le risposte hanno un codice di stato e una frase.

*Scenario Alice-Bob.* Alice scrive il messaggio per `bob@someschool.edu` con il suo user agent, che lo manda al suo mail server con SMTP e lo mette in coda. Il lato client SMTP del server apre una connessione TCP con il mail server di Bob e invia il messaggio; il server di Bob lo mette nella mailbox di Bob, che lo legge con il suo user agent. Una sessione tipica:

</td>
<td valign="top" align="center">

<img src="img/02-email-componenti.png" width="360" alt="Tre mail server collegati da frecce SMTP, ciascuno con i propri user agent, la coda dei messaggi in uscita e le mailbox"><br><sub>Figura: user agent, mail server e SMTP (slide 53).</sub>

<img src="img/02-smtp.png" width="290" alt="Diagramma temporale SMTP fra client e server: apertura TCP, handshaking (220, HELO, 250 Hello) e trasferimento dei messaggi"><br><sub>Figura: fasi di una sessione SMTP (slide 55).</sub>

</td>
</tr>
</table>


```
S: 220 hamburger.edu
C: HELO crepes.fr
S: 250 Hello crepes.fr, pleased to meet you
C: MAIL FROM: <alice@crepes.fr>
S: 250 alice@crepes.fr... Sender ok
C: RCPT TO: <bob@hamburger.edu>
S: 250 bob@hamburger.edu ... Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Do you like ketchup?
C: How about pickles?
C: .
S: 250 Message accepted for delivery
C: QUIT
S: 221 hamburger.edu closing connection
```

Il confronto con HTTP è utile. Entrambi hanno interazione comando-risposta in ASCII con codici di stato e usano connessioni persistenti. HTTP è **pull** (il client tira gli oggetti dal server), SMTP è **push** (il client spinge il messaggio sul server). SMTP richiede che header e corpo siano in ASCII a 7 bit e usa la sequenza `CRLF.CRLF` come fine del messaggio; HTTP incapsula ogni oggetto in una propria risposta, mentre SMTP manda più oggetti in un messaggio multipart.

SMTP (RFC 5321) è il protocollo di scambio, come HTTP è definito dall'RFC 7231 (citato dalle slide); la sintassi del messaggio è definita da **RFC 2822** e corrisponde al ruolo di HTML nel Web. Il messaggio ha righe di header (`To:`, `From:`, `Subject:`, che sono diverse dai comandi SMTP `MAIL FROM` e `RCPT TO`), una riga vuota e il corpo in soli caratteri ASCII.

SMTP serve a consegnare e depositare il messaggio sul server del destinatario; per **leggerlo** serve un protocollo di accesso alla posta. **IMAP** (RFC 3501) lascia i messaggi sul server e permette recupero, cancellazione e organizzazione in cartelle. Servizi come Gmail, Hotmail o Yahoo! Mail offrono un'interfaccia **HTTP** sopra SMTP (per l'invio) e IMAP o POP (per il recupero).

## DNS

**Definizione.** Il **Domain Name System** è un database distribuito, implementato come gerarchia di molti name server, e anche un protocollo a livello applicazione con cui host e server DNS si scambiano messaggi per risolvere i nomi (traduzione nome-indirizzo). **Intuizione.** Le persone usano nomi (`cs.umass.edu`), i router usano indirizzi IP a 32 bit: serve un servizio che li mappi, e dato che è una funzione essenziale di Internet la complessità è messa ai bordi, nel livello applicazione.

I servizi DNS sono la traduzione hostname → IP, l'**host aliasing** (nome canonico e alias), il **mail server aliasing** e la **distribuzione del carico** (a un nome corrispondono più indirizzi IP, per i web server replicati). La domanda è perché non centralizzare: un database centrale avrebbe un singolo punto di guasto, volumi di traffico enormi (solo i server DNS di Comcast gestiscono 600 miliardi di query al giorno, quelli di Akamai 2,2 mila miliardi), sarebbe lontano da molti client e difficile da mantenere. In breve, non scala. Le caratteristiche richieste sono: circa miliardi di record semplici, trilioni di query al giorno, molte più letture che scritture, prestazioni critiche (quasi ogni transazione di Internet passa dal DNS), decentralizzazione organizzativa e fisica con milioni di organizzazioni responsabili dei propri record, e affidabilità e sicurezza.

La **gerarchia** ha tre livelli: i **root server**, i server **TLD** (Top-Level Domain) e i server **autoritativi**.

- I **root name server** sono il contatto di ultima istanza dei name server che non riescono a risolvere un nome; senza di loro Internet non funzionerebbe. Sono 13 "server" logici nel mondo, ciascuno replicato molte volte (circa 200 server negli USA), e la radice è gestita da ICANN. DNSSEC fornisce autenticazione e integrità dei messaggi.
- I **TLD server** sono responsabili di `.com`, `.org`, `.net`, `.edu`, `.aero`, `.jobs`, `.museums` e di tutti i domini nazionali (`.cn`, `.uk`, `.fr`, `.ca`, `.jp`...). Network Solutions è il registro autoritativo per `.com` e `.net`, Educause per `.edu`.
- I **server autoritativi** sono i name server dell'organizzazione stessa, con le mappature autoritative fra nomi e IP dei suoi host; possono essere gestiti dall'organizzazione o da un fornitore.

<p align="center">
<img src="img/02-dns-gerarchia.png" width="680" alt="Gerarchia DNS a tre livelli: root, server TLD (.com, .org, .edu) e server autoritativi (yahoo.com, amazon.com, pbs.org, nyu.edu, umass.edu)"><br>
<sub><i>Figura: gerarchia distribuita del DNS (slide 65).</i></sub>
</p>



<table>
<tr>
<td valign="top">

Il **local DNS server** (ogni ISP ne ha uno) non appartiene strettamente alla gerarchia. Un host manda le sue query al local server, che risponde dalla propria cache di traduzioni recenti, che potrebbero essere obsolete, oppure inoltra la richiesta nella gerarchia. Per trovare il proprio: `scutil --dns` su macOS, `ipconfig /all` su Windows. In prima approssimazione, per `www.amazon.com` il client interroga la radice per trovare il server `.com`, poi il server `.com` per trovare il server di `amazon.com`, e infine questo per l'IP di `www.amazon.com`.

Ci sono due modi di risolvere. Nella **query iterativa** il server contattato risponde con il nome del prossimo server da interrogare ("non so questo nome, chiedi a quest'altro"): nell'esempio dell'host `engineering.nyu.edu` che cerca `gaia.cs.umass.edu`, il local server `dns.nyu.edu` interroga in sequenza root, TLD e il server autoritativo `dns.cs.umass.edu`. Nella **query ricorsiva** il compito della risoluzione ricade sul server contattato, che interroga a sua volta il successivo: così però c'è un carico pesante ai livelli alti della gerarchia.

</td>
<td valign="top" align="center">

<img src="img/02-dns-iterativa.png" width="420" alt="Query iterativa: l'host chiede al local DNS server, che interroga in sequenza root, TLD e server autoritativo (frecce numerate da 1 a 8)"><br><sub>Query iterativa (slide 70).</sub>

<img src="img/02-dns-ricorsiva.png" width="420" alt="Query ricorsiva: il local DNS server chiede al root, che a sua volta interroga il TLD, che interroga il server autoritativo (frecce numerate da 1 a 8)"><br><sub>Query ricorsiva (slide 71).</sub>

</td>
</tr>
</table>


Il **caching** funziona così: quando un name server apprende una mappatura la mette in cache e risponde subito con quella a query successive, migliorando i tempi. Le voci scadono dopo un **TTL**, e i server TLD sono tipicamente in cache nei local server. Le voci possono essere obsolete: se un host cambia IP la novità si propaga ovunque solo allo scadere dei TTL, quindi la traduzione è *best-effort*.

I dati sono memorizzati in **resource record** (RR) nel formato `(name, value, type, ttl)`:

| Tipo | `name` | `value` |
|---|---|---|
| A | hostname | indirizzo IP |
| NS | dominio (es. `foo.com`) | hostname del server autoritativo del dominio |
| CNAME | alias | nome canonico (es. `www.ibm.com` è in realtà `servereast.backup2.ibm.com`) |
| MX | (dominio) | nome del mail server SMTP associato |

I **messaggi DNS** di query e di risposta hanno lo stesso formato. L'header contiene l'*identification* a 16 bit (la risposta usa lo stesso numero della query) e i *flag* (query o risposta, recursion desired, recursion available, risposta autoritativa), poi i quattro contatori (numero di domande, di RR di risposta, di RR autoritativi, di RR aggiuntivi) e quattro sezioni: *questions* (nome e tipo richiesti), *answers* (RR in risposta), *authority* (record per i server autoritativi), *additional info* (informazioni utili aggiuntive).

*Registrare un dominio.* Per la nuova startup "Network Utopia" si registra `networkutopia.com` presso un registrar (per esempio Network Solutions), fornendo nomi e IP dei name server autoritativi primario e secondario. Il registrar inserisce nel TLD server `.com` due record, un NS e un A: `(networkutopia.com, dns1.networkutopia.com, NS)` e `(dns1.networkutopia.com, 212.212.212.1, A)`. Poi si crea localmente il server autoritativo con IP 212.212.212.1, che contiene un record A per `www.networkutopia.com` e un record MX per `networkutopia.com`.

Sulla **sicurezza** del DNS, gli attacchi **DDoS** ai root server (bombardamento di traffico) non hanno avuto successo finora, grazie al filtraggio del traffico e al fatto che i local server tengono in cache gli IP dei TLD e possono evitare i root; quelli ai TLD server sono potenzialmente più pericolosi. Gli attacchi di **spoofing** intercettano le query e restituiscono risposte false, per esempio con il **DNS cache poisoning**. La contromisura sono i servizi di autenticazione di **DNSSEC** (RFC 4033).

## Distribuzione di file: client-server contro P2P

Il problema posto è: quanto tempo serve per distribuire un file di dimensione `F` da un server a `N` peer? La risorsa limitata è la capacità di upload e download: `u_s` è la capacità di upload del server, `u_i` e `d_i` quella di upload e download del peer `i`, e la rete è supposta con banda abbondante.

Nel modello **client-server** il server deve inviare in sequenza `N` copie, e inviarne una richiede `F/u_s`, quindi `N·F/u_s` in totale; inoltre ogni client deve scaricare una copia, e il più lento impiega `F/d_min`, con `d_min` la minima velocità di download fra i client. Il tempo di distribuzione è quindi limitato inferiormente da:

`D_c-s ≥ max{ N·F/u_s , F/d_min }`

Cresce **linearmente** con `N`. Le slide disponibili finiscono qui: il caso P2P, il video streaming con le CDN e il socket programming mancano ancora.

## Esercizi svolti collegati

Le soluzioni degli esercizi del cap. 2 sono in [08-esercizi-cap2.md](08-esercizi-cap2.md): lettura di un GET HTTP (nome del server, versione, connessione persistente), tempo di risposta con DNS e connessioni HTTP persistenti/non persistenti, effetto di una cache Web sul ritardo medio e tempo di distribuzione di un file. Gli argomenti che mancavano a questo file (P2P, streaming, CDN e socket programming) sono ora in [04-p2p-streaming-cdn.md](04-p2p-streaming-cdn.md) e [05-socket-programming.md](05-socket-programming.md).

## Cose da saper fare all'esame

Per gli esercizi:

1. calcolare il tempo di risposta HTTP non persistente e persistente in funzione di RTT e tempo di trasmissione, con un dato numero di oggetti;
2. fare i conti dell'esempio della cache: utilizzazione del link di accesso con un certo hit rate, e ritardo medio come media pesata fra origine e cache;
3. scrivere o leggere una richiesta e una risposta HTTP (metodi, header, codici di stato) e una sessione SMTP (`HELO`, `MAIL FROM`, `RCPT TO`, `DATA`, `.`, `QUIT`);
4. seguire una risoluzione DNS indicando quali server vengono interrogati e quali record (A, NS, CNAME, MX) vengono restituiti, in versione iterativa e ricorsiva;
5. calcolare il limite inferiore `D_c-s` per la distribuzione di un file.

Per la teoria bisogna saper spiegare:

- client-server contro P2P e cosa si intende per socket e per indirizzamento di un processo (IP + porta);
- le quattro esigenze di trasporto delle applicazioni e i servizi di TCP e UDP, con il ruolo di TLS;
- perché HTTP è stateless e come i cookie aggiungono stato, con la differenza fra cookie di prima e di terza parte;
- cosa fanno web cache e GET condizionale (`If-modified-since`, `304`);
- il HOL blocking e come HTTP/2 e HTTP/3 lo affrontano;
- la differenza fra SMTP (push, consegna) e i protocolli di accesso (IMAP, HTTP), e il confronto SMTP-HTTP;
- perché il DNS non è centralizzato, la gerarchia root-TLD-autoritativo, il caching con TTL e le minacce al DNS.

## Dubbi

- `[DA VERIFICARE]` La slide sui cookie di terze parti dice che saranno disattivati in Chrome "nel 2023": il testo è datato, chiedere o controllare lo stato attuale.
- `[DA VERIFICARE]` Nella slide sul metodo PUT il testo dice che il contenuto sostituisce il file "nel corpo della richiesta POST": sembra un refuso per PUT.
- `[DA VERIFICARE]` Le slide citano per HTTP sia l'RFC 7231 (in confronto a RFC 5321/2822) sia le RFC 7230 e 9110: verificare quale usare come riferimento.
- Le slide chiedono "perché esiste UDP?" e "cosa sono le analogie umane di risorse riservate o a richiesta?" senza rispondere: la prima ha risposta nei capitoli sul trasporto.
- Il PDF caricato su Drive finisce alla slide di apertura di P2P (file distribution client-server): chiedere se è stato caricato in parte o se le slide su P2P, video e CDN, socket arriveranno separatamente.

## Esercizi svolti collegati

Gli esercizi del capitolo 2 di Sereno (SMTP, HTTP persistente e non persistente, intestazioni HTTP, proxy e cache, DNS, P2P centralizzato) sono svolti in [12-esercizi-sereno-cap1-cap2.md](12-esercizi-sereno-cap1-cap2.md); i problemi di Botta sono in [08-esercizi-cap2.md](08-esercizi-cap2.md).
