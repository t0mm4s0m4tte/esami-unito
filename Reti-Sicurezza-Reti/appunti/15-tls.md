# Reti 15 - TLS e HTTPS

Fonte: Moodle › Reti e Sicurezza delle Reti › Slides (Sicurezza) › "06 - Transport Layer Security" (`06_tls.pdf`, Idilio Drago e Paolo Castagno, a.a. 2026/2027). Le slide sono in inglese, usano esempi reali con `www.unito.it` (catture `tshark`, `openssl s_client`, `dig`) e ogni blocco parte da un "Problem". Il testo estratto ripete molte volte le slide per via delle animazioni e perde le figure: dove un punto dipende da una figura è segnato `[DA VERIFICARE sul PDF]`. `[INT]` = integrazione mia, non presente nelle slide. Prerequisiti: [06-fondamenti-crittografici.md](06-fondamenti-crittografici.md) (Diffie-Hellman, firma, MAC, AEAD) e [14-pki-certificati.md](14-pki-certificati.md) (certificati, CA, catena di fiducia).

## Mappa degli argomenti

Perché serve TLS e dove sta nello stack; versioni di SSL/TLS; handshake e record protocol; record e numeri di sequenza; handshake TLS 1.2 e suite di cifratura; scambio di chiavi autenticato, RSA contro ECDHE e forward secrecy; protezione dell'handshake con `Finished`; handshake TLS 1.3; confronto 1.2 contro 1.3; ripresa di sessione, 0-RTT e replay; SNI ed Encrypted Client Hello; attacchi di downgrade (POODLE) e Heartbleed.

## Perché TLS e dove sta

**Problema**: i dati HTTP attraversano in chiaro tutte le reti del percorso; un attaccante in un router legge password e cookie (**riservatezza**), può cambiare le pagine o rispondere al posto del server (**integrità** e **autenticazione**). **TLS protegge TCP** con cifratura, un **MAC per messaggio** e un **certificato del server**.

- TLS gira **sopra TCP**, con un'interfaccia socket. **HTTPS = HTTP su TLS**, porta **443** (HTTP in chiaro: porta 80). Anche altri protocolli usano TLS: IMAP 993, submission 465, DNS 853 (DoT).
- **QUIC** gira su UDP (porta 443) con le chiavi di un handshake TLS 1.3 e trasporta HTTP/3.

## Versioni

SSL 2.0 e 3.0 (Netscape, 1995 e 1996) vietati rispettivamente dal 2011 e dal 2015. L'IETF ha standardizzato TLS 1.0 (1999, basato su SSL 3.0) e TLS 1.1 (2006): **vietati dal 2021**. **TLS 1.2 (2008) e TLS 1.3 (2018, revisionato 2026) restano ammessi.** `www.unito.it` rifiuta TLS 1.0 e 1.1 con un alert `protocol_version`.

## Handshake e record protocol

Problema: all'apertura della connessione TCP non esiste una chiave condivisa. Sequenza: apertura TCP (SYN, SYN+ACK, ACK) sulla porta 443; **handshake protocol** (autentica il server e crea le chiavi: da Client Hello a Finished); **record protocol** (trasporta i messaggi HTTP cifrati e autenticati); chiusura con un alert `close_notify` (così un taglio da parte dell'attaccante viene riconosciuto) e poi FIN.

## Record protocol

- TLS divide i dati in **frammenti di al più 16 KB**. In TLS 1.3 si aggiunge in coda il **tipo di contenuto reale** (23 = application data).
- **AEAD** (authenticated encryption with associated data, es. AES-256-GCM) cifra e autentica con la chiave di sessione e aggiunge un **tag di 16 byte**.
- **Header di 5 byte in chiaro**: tipo, versione, lunghezza. Esempio: `17 03 03 00 4b`: tipo `0x17` (application data), versione `0x0303` (TLS 1.2, per compatibilità), lunghezza `0x004b` = 75 byte (58 byte di dati + 1 di tipo + 16 di tag). Altri tipi: handshake (22), alert (21), change cipher spec (20).
- **Numero di sequenza**: ogni lato conta i record; il numero **non viene inviato** ma entra nel **nonce**. Un record rimosso o riordinato fa fallire il controllo del tag e la connessione si chiude.
- Con il **key log** del client (`tls.keylog_file`) un analizzatore come tshark decifra il traffico (esempio nelle slide: la richiesta `HEAD / HTTP/1.1` di 58 byte).

## Handshake TLS 1.2

Problema: client e server supportano algoritmi diversi e non hanno una chiave condivisa. **Due round trip**:
1. **Client Hello** → il client offre versioni e suite, con un *random* di 32 byte. Il server risponde con **Server Hello, Certificate, Server Key Exchange, Server Hello Done** (sceglie, manda il certificato e il suo valore Diffie-Hellman firmato). Facoltativo: Certificate Request, per autenticare anche il client.
2. **Client Key Exchange, Change Cipher Spec, Finished** (il client manda il suo valore; entrambi calcolano le chiavi; Change Cipher Spec le attiva) e il server risponde con **Change Cipher Spec, Finished**. Poi parte la richiesta HTTP cifrata. **Finished**, il primo messaggio cifrato, chiude l'handshake.

**Cipher suite**: nome che elenca gli algoritmi, es. `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256` = scambio chiavi **ECDHE** (Diffie-Hellman su curve ellittiche, effimero, valori nuovi a ogni connessione) + firma del server **ECDSA** + protezione dei record **ChaCha20-Poly1305** + **SHA-256** per derivare le chiavi. Il client elenca le sue suite (27 per OpenSSL nell'esempio), il server ne sceglie una (codice `0xcca9` per `www.unito.it`).

**Scambio di chiavi autenticato.** Un valore DH da solo non dice chi l'ha mandato. Il server **firma il proprio valore B** con la sua chiave privata `P_RS`; il client controlla la firma con la chiave pubblica `P_US` del certificato e manda A. Entrambi calcolano lo stesso **pre-master secret** `s`, che non viaggia mai in rete; `master secret = PRF(s, client random, server random)` (PRF costruita su HMAC). Se un malware sul router sostituisce B con il proprio `B_A`, non può firmarlo senza `P_RS`: il client chiude l'handshake.

## RSA contro ECDHE: forward secrecy

- Con lo **scambio di chiavi RSA** (`TLS_RSA_…`) il client cifra `s` con `P_US`. Un attaccante registra le connessioni cifrate (per esempio a un exchange point) e aspetta: se un anno dopo ruba `P_RS`, decifra `s` e quindi **tutti** i record registrati.
- Con **ECDHE** `P_RS` serve solo a firmare e i segreti `a` e `b` vengono cancellati dopo l'handshake: rubare `P_RS` dopo non rivela `s` e le connessioni passate restano segrete. È la **forward secrecy**.
- Lo scambio RSA è **rimosso in TLS 1.3 e vietato in TLS 1.2 da luglio 2026** (RFC 10015, secondo le slide).

**Client Hello modificato.** Client Hello viaggia in chiaro: un gateway compromesso può togliere le suite forti e far scegliere al server una debole. Difesa: **Finished** contiene un **MAC calcolato con le nuove chiavi su tutti i messaggi dell'handshake** (in TLS 1.2: `PRF(master secret, "client finished", hash dei messaggi)`). Se l'attaccante ha cambiato l'elenco delle suite i MAC non coincidono e l'handshake fallisce; senza le chiavi non può correggerli.

**Esempio TLS 1.2** (`tshark`): Client Hello (con `SNI = www.unito.it`); Server Hello, Certificate, Server Key Exchange, Server Hello Done; Client Key Exchange, Change Cipher Spec, Encrypted Handshake Message; New Session Ticket, Change Cipher Spec, Encrypted Handshake Message; Application Data; Encrypted Alert; FIN. **Il certificato viaggia in chiaro**, leggibile da chiunque osservi. Senza chiavi, Finished appare come "Encrypted Handshake Message" e HTTP come "Application Data".

## Handshake TLS 1.3

Problema: TLS 1.2 richiede due round trip e manda il certificato in chiaro. In TLS 1.3 il client manda il proprio valore DH **già nel Client Hello** (*key share*); **Server Hello** porta il valore del server e entrambi calcolano le chiavi. Da lì tutto è cifrato: **Encrypted Extensions, Certificate, Certificate Verify, Finished** (il certificato è cifrato; `Certificate Verify` firma con `P_RS` tutti i messaggi precedenti). Il client manda Finished e la richiesta dopo **un solo round trip**. Se il server non accetta nessuno dei gruppi offerti manda una **Hello Retry Request** (un round trip in più).

Dettagli dalle catture: dopo il Server Hello ogni record appare come "Application Data" con il tipo reale cifrato; **Change Cipher Spec** non porta nulla in 1.3 e serve solo a far sembrare la connessione TLS 1.2 ai vecchi dispositivi di rete.

**Client Hello (esempio).** I campi di versione dicono vecchie versioni (record `TLS 1.0 0x0301`, hello `TLS 1.2 0x0303`) e l'estensione **`supported_versions`** offre TLS 1.3 e 1.2; il *random* nuovo a ogni connessione entra nella derivazione delle chiavi; 30 suite (es. `TLS_AES_256_GCM_SHA384` `0x1302`); `server_name` in chiaro; **due key share** per due gruppi (`X25519MLKEM768`: X25519 con **ML-KEM**, progettato per resistere ai computer quantistici, e `x25519`), il server ne sceglie uno. **Server Hello**: `supported_versions` seleziona TLS 1.3 e il server sceglie `TLS_AES_256_GCM_SHA384` e la prima key share. Una suite TLS 1.3 nomina solo protezione dei record e hash: le estensioni portano il resto.

## TLS 1.2 e TLS 1.3 a confronto

| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Round trip prima dei dati | 2 | 1 |
| Scambio di chiavi | ECDHE (RSA e DHE vietati) | ECDHE, DHE o ibrido |
| Protezione dei record | AEAD, o CBC con HMAC | solo AEAD |
| Certificato | in chiaro | cifrato |
| Nome delle suite | scambio chiavi, firma, protezione, hash | protezione, hash |
| Versione scelta in | campi di versione | `supported_versions` |
| Implementazioni (RFC 9325) | must support | should support, must prefer |

## Ripresa di sessione, 0-RTT, replay

- **Session ticket**: problema, ogni nuova connessione allo stesso server ripete l'handshake completo. Alla fine di un handshake completo il server manda un **ticket**: lo stato della sessione (suite, master secret) **cifrato con una chiave nota solo al server**, che il client conserva. In una nuova connessione il client mette il ticket nel Client Hello; il server manda subito Finished, **senza certificato e senza scambio di chiavi** (*abbreviated handshake*). Esempio: ticket di 192 byte valido 18 ore (`Lifetime Hint 64800 s`). In TLS 1.3 il ticket arriva dopo l'handshake e si ripresenta come **PSK** (pre-shared key).
- **Early data / 0-RTT**: una connessione ripresa aspetta comunque un round trip. Con la PSK il client cifra la richiesta e la manda **già nel Client Hello**, senza attesa. Le chiavi 0-RTT derivano solo dalla PSK: **il client deve assumere che non ci sia forward secrecy**.
- **Replay**: l'early data non dipende dal Server Hello; un attaccante nel router copia il primo volo e lo rimanda, e il server **esegue la richiesta due volte**. Difese: i client mandano early data **solo per richieste sicure da ripetere** (GET, non un pagamento); i server possono rifiutarlo con **`425 Too Early`**.

## Nomi e attacchi

- **SNI (Server Name Indication)**: un indirizzo serve più siti, ciascuno con il proprio certificato (esempio: `www.unito.it`, `www.unimi.it` e `www.unitn.it` sugli stessi due indirizzi `104.18.10.29` e `104.18.11.29`). Il client scrive il nome nel Client Hello (estensione `server_name`) così il server sceglie il certificato prima di qualsiasi cifratura. Senza nome (`-noservername`) il server non ha un certificato e chiude con un alert (`handshake failure`, numero 40).
- **Encrypted Client Hello (ECH, 2026)**: il nome in Client Hello è in chiaro e rivela ogni sito visitato. ECH cifra il Client Hello con una chiave che il server pubblica nel DNS (record `HTTPS`). Nell'esempio, il record HTTPS di `www.unito.it` non contiene una chiave ECH, quindi il nome viaggia in chiaro.
- **Downgrade e POODLE** (CVE-2014-3566, Möller, Duong, Kotowicz, Google, 2014): un attaccante nel mezzo forza una versione vecchia e difettosa. Molti client ripetevano un handshake fallito con una versione più vecchia; l'attaccante scartava i primi tentativi e forzava **SSL 3.0**, che perde i cookie. Ogni ritentativo era un handshake nuovo, quindi Finished non vedeva nulla. **Difesa**: i client non devono fare fallback (RFC 9325), ma l'attaccante può togliere TLS 1.3 dall'offerta; per questo un server TLS 1.3 che negozia TLS 1.2 termina il proprio *random* con i byte **`DOWNGRD`** (poi `01`); un client che aveva offerto TLS 1.3 li riconosce e **interrompe**.
- **Heartbeat e Heartbleed** (CVE-2014-0160, OpenSSL Security Advisory, 7 aprile 2014): l'estensione heartbeat (2012) mantiene viva la connessione: il server restituisce un payload della lunghezza dichiarata. Un attaccante manda 1 byte e dichiara 65535. **OpenSSL 1.0.1 fino a 1.0.1f non controllava la lunghezza** e restituiva fino a 64 KB di memoria del server, compresa la **chiave privata**: con quella l'attaccante impersona il server finché il certificato non viene revocato o scade. Lezione: un protocollo corretto può fallire per un bug nell'implementazione.

## Cose da saper fare all'esame `[INT]`

- Dire a cosa serve TLS (riservatezza, integrità, autenticazione), dove sta nello stack (sopra TCP, porta 443; QUIC su UDP) e quali versioni sono ammesse (1.2 e 1.3).
- Disegnare l'handshake TLS 1.2 (due round trip, messaggi dei due voli) e quello TLS 1.3 (un round trip, certificato cifrato) e dire cosa porta ogni messaggio.
- Spiegare il record protocol: frammenti da 16 KB, AEAD con tag da 16 byte, header da 5 byte, numero di sequenza nel nonce, `close_notify`.
- Spiegare il ruolo di ECDHE, della firma del valore DH e del certificato; dimostrare che cosa succede se un attaccante sostituisce B.
- Spiegare la **forward secrecy** confrontando scambio RSA e ECDHE.
- Spiegare perché `Finished` difende da un Client Hello modificato (MAC sull'intero handshake).
- Spiegare ticket, ripresa abbreviata, 0-RTT e il rischio di replay con la difesa (richieste sicure, `425 Too Early`).
- Spiegare SNI ed ECH; il downgrade (POODLE) e la difesa `DOWNGRD`; Heartbleed.
- Tabella di confronto TLS 1.2 contro 1.3.

## Dubbi

- Le animazioni e le figure sono persi nel testo: gli schemi dei messaggi sono ricostruiti dalle didascalie `[DA VERIFICARE sul PDF]`.
- "TLS 1.3 (2018, revisionato 2026)" e "RFC 10015" sono citati come nelle slide; non ho verificato i numeri di RFC fuori dal corso.
- Mancano le altre lezioni di Sicurezza (malware e successive) e il cap. 6 di Kurose (collegamento), non ancora su Moodle.
