# Reti 14 - PKI e certificati

Fonte: Moodle › Reti e Sicurezza delle Reti › Slides (Sicurezza) › "05 - Public Key Infrastructure" (`05_pki.pdf`, circa 30 slide ripetute con animazioni nel testo estratto, Idilio Drago e Paolo Castagno, a.a. 2026/2027). Le slide sono in inglese, ogni blocco parte da un "Problem" e usa esempi reali (`www.unito.it`, `openssl`, `curl`, i siti di prova `badssl.com`). Il testo estratto perde le figure (frecce, scambi di messaggi): dove un punto dipende da una figura è segnato `[DA VERIFICARE sul PDF]`. `[INT]` = integrazione mia, non presente nelle slide. Prerequisito: [06-fondamenti-crittografici.md](06-fondamenti-crittografici.md) (chiave pubblica, firma digitale, hash). Le slide richiamano il TLS, che nel corso si vede altrove.

## Mappa degli argomenti
Si parte dal problema di fondo (una chiave pubblica non ha un nome) e dalla soluzione, il certificato firmato da una CA. Poi la catena di fiducia (CA radice e intermedie, verifica della catena, trust store), il formato X.509 con un esempio reale, le verifiche che un client fa su un certificato e la revoca (CRL, OCSP, stapling, come la gestiscono i browser). Seguono la riduzione della durata dei certificati, l'emissione automatica con ACME e gli attacchi alla validazione (BGP hijack, validazione multi-prospettiva), i certificati emessi in modo scorretto (caso DigiNotar) e infine la Certificate Transparency.

## Perché servono i certificati
Una chiave pubblica non porta con sé un nome. Se il client riceve la chiave pubblica P_US del server `www.unito.it` su un Wi-Fi pubblico, un attaccante sullo stesso Wi-Fi può sostituirla con la propria P_UA senza che nessuno se ne accorga; poi legge il messaggio M cifrato dal client per P_UA e lo cifra di nuovo per P_US verso il vero server (**attaccante nel mezzo**, *man in the middle*). Niente lega P_US al nome del server.

La soluzione è il **certificato**: lega un nome a una chiave pubblica per un periodo di validità. Una **certification authority (CA)** verifica che chi lo richiede controlli il nome (per esempio `www.unito.it`) e firma con la propria chiave privata P_R,CA un certificato che contiene il nome, la chiave pubblica P_US e la scadenza (nell'esempio "until 1 Dec 2026"). La terminologia segue l'RFC 4949 (PKI: CA, certificati e procedure). Il client verifica la firma con la chiave pubblica della CA, P_U,CA, installata con il browser o con il sistema. Se un access point falso, che ha lo stesso nome della rete pubblica, sostituisce P_US con P_UA, la firma non è più valida: qualsiasi modifica rompe la firma, e solo la CA possiede P_R,CA.

## Catena di fiducia
**CA radice e CA intermedie.** Problema: una chiave di firma tenuta online può essere rubata. Una **CA radice** è *autofirmata* e sta nei trust store; la sua chiave privata resta su una rete isolata. La radice firma le **CA intermedie**, le cui chiavi online firmano i certificati dei server. Se un intermedio viene compromesso la radice lo **revoca**, e i trust store non cambiano. Nell'esempio delle slide la radice è GTS Root R4 (Google Trust Services), l'intermedio WE1 e il server `www.unito.it`; altri server hanno certificati firmati da altri intermedi.

**Verifica della catena.** Il client si fida di GTS Root R4, non di WE1. Il server invia il proprio certificato, emesso da WE1 (campo *issuer*: WE1), e anche il certificato di WE1, il cui *subject* coincide con l'issuer del primo; la chiave di WE1 verifica la firma del primo. Il certificato di WE1 è a sua volta firmato da GTS Root R4, che è nel trust store del client: la catena finisce in questo **trust anchor**, che è di fiducia perché sta nello store.

**Trust store.** Contiene i trust anchor: Debian ne ha 120, dallo store di Mozilla (`grep -c 'BEGIN CERTIFICATE' /etc/ssl/certs/ca-certificates.crt`). Una radice è autofirmata (issuer uguale a subject), perciò la sua firma non dimostra nulla sul suo proprietario: è fidata solo perché è nello store. I *root program* (Mozilla, Google, Apple, Microsoft) ammettono e rimuovono radici e richiedono audit annuali. Esempio citato: Chrome ha smesso di fidarsi dei nuovi certificati Entrust a novembre 2024, dopo sei anni di mancato rispetto delle regole (Google Security Blog, 27 giugno 2024).

**Esempio: la catena di www.unito.it.** Con `openssl s_client -connect www.unito.it:443 -servername www.unito.it -showcerts` il server invia tre certificati: il numero 0 (il proprio), il numero 1 (WE1) e il numero 2 (GTS Root R4). Quest'ultimo è un **cross-certificate**, firmato da GlobalSign Root CA (issuer `GlobalSign nv-sa, Root CA`), e serve ai client che si fidano solo di GlobalSign Root CA. Con `openssl verify -show_chain -untrusted we1.pem unito.pem` si costruisce la catena dal certificato 0 e dal certificato 1 fino a GTS Root R4 (profondità 0: `CN=www.unito.it`, 1: WE1, 2: GTS Root R4) e si ottiene `unito.pem: OK`. Il certificato 2 non serve, perché GTS Root R4 è già nello store.

## Il formato X.509
Lo standard è X.509 (ITU-T, 1988; profilo per Internet in RFC 5280). Nel certificato di `www.unito.it` (letto con `openssl x509 -in unito.pem -noout -text`): **Serial Number** (unico per ciascun issuer); **Signature Algorithm** `ecdsa-with-SHA256` (ECDSA: Elliptic Curve Digital Signature Algorithm); **Issuer** (la CA: `C=US, O=Google Trust Services, CN=WE1`); **Validity** (Not Before 2 settembre 2026, Not After 1 dicembre 2026: 90 giorni); **Subject** (il proprietario: `CN=www.unito.it`, CN = common name); **Subject Public Key Info** (algoritmo `id-ecPublicKey`, chiave da 256 bit); infine il **Signature Value**, la firma ECDSA di WE1 su tutti i campi.

**Estensioni.** Un'estensione *critical* deve essere conosciuta dal client, altrimenti il certificato va rifiutato. Nell'esempio: *Key Usage* (critical) `Digital Signature` (le firme, solo per autenticare un server TLS); *Extended Key Usage* `TLS Web Server Authentication`; *Basic Constraints* (critical) `CA:FALSE` (questa chiave non può firmare altri certificati); *Authority Information Access* con `CA Issuers - URI:http://i.pki.goog/we1.crt` (dove ottenere il certificato dell'issuer, se manca); *Subject Alternative Name* `DNS:www.unito.it` (i nomi di host, confrontati con il nome richiesto).

## Validazione di un certificato
Il client controlla ogni certificato della catena. Quattro controlli (la tabella delle slide è un esempio di `openssl verify` su `unito.pem` senza WE1, al 2 dicembre 2026, per unito.it, come certificato client):

| Controllo | Campi | Errore |
|---|---|---|
| Catena fino a un trust anchor | issuer, firma | `unable to get local issuer certificate` |
| Periodo di validità | Not Before, Not After | `certificate has expired` |
| Nome dell'host | Subject Alternative Name | `hostname mismatch` |
| Uso della chiave | Extended Key Usage | `unsuitable certificate purpose` |

Basta un controllo fallito per rifiutare il certificato e chiudere la connessione. I siti di prova `badssl.com` lo mostrano con `curl` (che esce con codice 60): `expired.badssl.com` (oltre Not After: `certificate has expired (10)`), `wrong.host.badssl.com` (certificato per un altro nome: `no alternative certificate subject name matches target hostname`), `self-signed.badssl.com` (`self-signed certificate (18)`), `untrusted-root.badssl.com` (una catena verso una radice fuori dal trust store: `self-signed certificate in certificate chain (19)`).

## Revoca
Problema: una chiave rubata funziona fino a Not After. Un certificato è valido da Not Before a Not After (qui 90 giorni). Se la chiave viene rubata e il certificato revocato, solo i client che controllano la revoca lo rifiutano prima di Not After; gli altri lo accettano. Su richiesta del proprietario la CA deve revocarlo entro 24 ore.

**CRL (certificate revocation list).** Problema: le revoche devono arrivare al client. La CA (WE1 nell'esempio) pubblica una lista firmata con i numeri di serie dei certificati revocati (con data e motivo), con *this update* e *next update*. La CA ne pubblica una nuova almeno una volta a settimana, e entro 24 ore da una revoca. Il client la scarica dall'URL indicato in *CRL Distribution Points* e cerca il numero di serie. Esempio: `curl https://revoked.badssl.com/` termina con 0, cioè accetta il certificato revocato, perché `curl` controlla la revoca solo se richiesto. Dal certificato si leggono il seriale (`050A72B7...9549`) e l'URL della CRL (`http://ye2.c.lencr.org/74.crl`); la CRL di YE2, un intermedio di Let's Encrypt, elenca 1561 certificati; il seriale compare con data di revoca 15 settembre 2026 e motivo *Key Compromise*; con `curl --crlfile ca.crl.pem` il certificato viene respinto (`certificate revoked (23)`).

**OCSP (Online Certificate Status Protocol).** Problema: il client ha bisogno di un solo stato. Il client manda a un *OCSP responder* della CA il numero di serie e riceve uno stato firmato: *good*, *revoked* o *unknown*. Svantaggi: la richiesta rivela il sito alla CA e a chi osserva l'HTTP in chiaro; ritarda l'handshake (100 ms alla mediana in Firefox). **OCSP stapling**: il server ottiene lo stato del proprio certificato e lo invia nell'handshake TLS insieme al certificato, quindi il client non interroga nessun responder. L'OCSP è diventato facoltativo per le CA dal 2024, e Let's Encrypt ha spento i suoi responder il 6 agosto 2025.

**Come i browser controllano la revoca.** Firefox riceve ogni 12 ore una codifica compatta di *tutte* le revoche (CRLite). Chrome in generale non controlla né OCSP né CRL online, e riceve un sottoinsieme delle revoche (CRLSets).

## Durata massima dei certificati
Problema: la revoca protegge solo i client che la controllano; accorciare la durata riduce il danno di una chiave rubata. Prima di marzo 2026 un certificato (e una chiave rubata) restava valido fino a 398 giorni. Il CA/Browser Forum (CA e produttori di browser) ha abbassato il massimo a 200 giorni da marzo 2026, a 100 giorni da marzo 2027 e a 47 giorni da marzo 2029. Il certificato di `www.unito.it` dura 90 giorni, oltre il massimo che varrà da marzo 2029. Un certificato valido al massimo 7 giorni non richiede informazioni di revoca: Let's Encrypt emette certificati *short-lived* di 160 ore.

## Emissione automatica: ACME
Problema: rinnovi frequenti richiedono emissione automatica. **ACME** (Automatic Certificate Management Environment, RFC 8555, 2019): il client ACME (sul server) ordina un certificato con richieste firmate con la chiave del proprio account; la CA risponde con un *token* t che il client deve dimostrare di poter pubblicare, sul web server (**HTTP-01**, all'URL `/.well-known/acme-challenge/t`) oppure nel DNS (**DNS-01**); il client invia la chiave pubblica P_US e riceve il certificato. Let's Encrypt emette certificati solo tramite ACME.

**Validazione dirottata.** Problema: il controllo della CA attraversa la rete. Un attaccante ordina un certificato per `www.unito.it` e riceve un token che non può pubblicare sul server; vicino alla CA, devia verso di sé il traffico per quel nome (**BGP hijack**; BGP è il protocollo di routing tra le reti di Internet), la CA ottiene il token dall'attaccante ed emette il certificato. Difesa: la **validazione multi-prospettiva**: la CA ripete il controllo da luoghi distanti almeno 500 km; una verifica remota raggiunge il vero server, non trova nessun token e la CA non emette il certificato. Da marzo 2026 una CA usa almeno tre prospettive remote in due regioni e la maggioranza deve concordare.

## Certificati emessi in modo scorretto
Problema: qualsiasi CA può firmare per qualsiasi nome, quindi la CA meno attenta determina la sicurezza di ogni sito. Se una CA compromessa dà all'attaccante un certificato per `www.unito.it` con P_UA, il certificato supera tutti i controlli e l'attaccante nel mezzo legge e modifica il traffico senza alcun avviso; il vero server non lo vede mai.

**Caso DigiNotar (2011).** Un certificato fraudolento `*.google.com` emesso da DigiNotar fu usato contro utenti soprattutto in Iran. DigiNotar confermò più di 200 certificati di questo tipo, per più di 20 domini. Mozilla rimosse DigiNotar dal proprio root program e Chrome rifiutò tutte le sue CA (Google Online Security Blog, 29 agosto 2011; Mozilla Security Blog, 2 settembre 2011).

## Certificate Transparency
Problema: il proprietario non vede mai un certificato emesso in modo scorretto. **CT (Certificate Transparency)**, RFC 6962: prima di emettere un certificato la CA lo invia a *log* pubblici e *append-only*; ogni log promette la pubblicazione con un **SCT** (signed certificate timestamp) firmato; il certificato finale contiene due SCT. Chrome e Safari rifiutano un certificato pubblico senza SCT da almeno due log. Esempio: il certificato di `www.unito.it` contiene due SCT del 2 settembre 2026; i *Log ID* (hash delle chiavi dei log) identificano nella lista dei log di Chrome i log Google Argon2026h2 e Cloudflare Nimbus2026; al 23 ottobre 2026 i due log contenevano 3,5 e 6,7 miliardi di voci. I log sono alberi hash (alberi di Merkle), dove modifiche o rimozioni si rilevano. I **monitor**, come il proprietario di un nome, cercano nei log certificati mai richiesti (per esempio `www.unito.it` emesso da una CA compromessa): un certificato accettato da Chrome o Safari è nei log, quindi il monitor lo trova.

## Cose da saper fare all'esame
- Spiegare perché una chiave pubblica da sola non basta (attacco man in the middle) e cosa lega un certificato.
- Descrivere cosa contiene un certificato X.509 (serial, issuer, validity, subject, chiave pubblica, algoritmo e firma, estensioni come SAN, Key Usage, Basic Constraints) e cosa significa "critical".
- Spiegare CA radice e intermedie, perché la radice è tenuta isolata, e come un client verifica una catena fino a un trust anchor; spiegare perché una radice autofirmata è fidata solo perché è nel trust store.
- Elencare i controlli di validazione (catena, validità, nome dell'host, uso della chiave) e riconoscere gli errori corrispondenti.
- Confrontare CRL, OCSP e OCSP stapling (privacy, latenza, aggiornamento) e dire come Firefox e Chrome gestiscono la revoca.
- Spiegare perché si accorcia la durata dei certificati e cosa sono i certificati short-lived.
- Descrivere ACME (HTTP-01, DNS-01), l'attacco con BGP hijack alla validazione e la difesa multi-prospettiva.
- Spiegare il problema dei certificati emessi per errore o da CA compromesse (DigiNotar) e come Certificate Transparency (log, SCT, monitor) lo affronta.

## Dubbi
- Le date e i numeri delle slide (durata massima dei certificati, spegnimento degli OCSP di Let's Encrypt, 3,5 e 6,7 miliardi di voci nei log) sono quelli del PDF 2026/27 e non li ho confrontati con altre fonti.
- Le animazioni (le slide sono ripetute con una riga in più alla volta) rendono il testo estratto molto ripetitivo; i punti che ho riassunto sono quelli dell'ultima versione di ogni slide.
- Non è chiaro dal testo se la CRL sia descritta come obbligatoria per tutte le CA: dico solo quello che dicono le slide (aggiornamento almeno settimanale e entro 24 ore da una revoca).
