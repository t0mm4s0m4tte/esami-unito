# Reti e Sicurezza delle Reti: preparazione allo scritto

Fonti: Moodle corso 3685 (26/27), appunti `../Reti-Sicurezza-Reti/appunti/` (01-14), README della materia. Convenzione: `[STIMA]` mio, `[DA VERIFICARE]` da controllare.

## Cosa so e cosa no

- 9 CFU, testo Kurose-Ross, docenti Sereno (reti) e Drago (sicurezza). Date appelli, durata e formato dello scritto: **non pubblicati** `[DA VERIFICARE]`. Su Moodle non ci sono tracce d'esame né simulazioni: `tracce.zip` contiene catture Wireshark, non esami.
- Su Moodle c'è una pagina "Kurose's online Exercises" che rimanda a `gaia.cs.umass.edu/kurose_ross/interactive/`: esercizi interattivi ufficiali del libro, validi per allenarsi.
- Il corso 2026/27 ha caricato di nuovo (e scaricato ora in `_moodle-raw/26-27-nuovi/Reti-Sicurezza-Reti/`): `06_tls.pdf` (Transport Layer Security), `SoluzioniProblemCapitolo2.pdf` e `00_introduzione.pdf`. Mancano ancora nei miei appunti: TLS e la sicurezza dopo la PKI (malware, ecc.), il cap. 6 di Kurose (livello di collegamento), non ancora su Moodle.

## Strategia per superare lo scritto `[STIMA]`

Reti si passa con due tipi di competenze: **esercizi numerici** (formule, tabelle, indirizzi) e **teoria a risposta aperta** (definizioni, protocolli, confronti). Il piano è:

1. **Esercizi numerici, 60% del tempo.** Ricorrono nei corsi Kurose-Ross e negli esercizi di Sereno già presenti negli appunti:
   - ritardo nodale, store-and-forward, throughput, trasmissione in pipeline (appunto 12; l'esercizio 10 del cap. 1 ha soluzione dubbia: 157 ms nel PDF, 93 ms in pipeline: chiedi al docente);
   - tempi HTTP con DNS, connessioni persistenti e non persistenti, proxy/cache, client-server vs P2P (08, 12);
   - TCP: numeri di sequenza e ACK, RTT e timeout (EstimatedRTT, DevRTT), finestra di congestione (slow start, AIMD, CUBIC), Go-Back-N e Selective Repeat con diagrammi temporali (09);
   - IPv4: sottoreti, CIDR, longest prefix match, frammentazione, NAT (10);
   - routing: Dijkstra e distance vector con tabelle, OSPF/BGP concettuali (11).
2. **Teoria, 40% del tempo.** Schede di una pagina per protocollo/argomento: HTTP, DNS, SMTP/IMAP, TCP, UDP, IP, ICMP, OSPF, BGP, SDN/OpenFlow; sicurezza: CIA, crittografia simmetrica/asimmetrica, hash/MAC, firma, autenticazione (FIDO2), controllo accessi, PKI, TLS. Per ognuna: a cosa serve, header principali, come funziona in 5 righe, un confronto (es. UDP vs TCP, DV vs LS).
3. **Simulazione a tempo**: 2 al mese da dicembre, ognuna di 2 h con 4 esercizi numerici + 6 domande teoriche scelte a caso dalle schede. In assenza di tracce vere, uso gli esercizi di Sereno e di Kurose come banco di prova. Se trovi vecchi scritti (compagni, docente), passameli: li metto in `_moodle-raw`.

## Priorità per argomento `[STIMA]`

| Argomento | Appunto | Peso | Stato |
|---|---|---|---|
| Livello trasporto (TCP) | 09 | alto | coperto, da esercitare |
| Livello di rete (data plane, IP) | 10 | alto | coperto |
| Routing (control plane) | 11 | medio | coperto |
| Livello applicazione | 02, 04, 08 | medio | coperto |
| Crittografia, autenticazione, PKI | 06, 07, 14 | alto (parte sicurezza) | coperto |
| Controllo accessi | 13 | medio | coperto |
| TLS | 15 | alto (sicurezza) | coperto (2026-11-12) |
| Collegamento (cap. 6) | - | medio | non su Moodle |
| Malware e altre lezioni di Sicurezza | - | da capire | non ancora a lezione |

## Da fare, in ordine

1. ~~Riassumere `06_tls.pdf`~~: fatto il 2026-11-12 (`appunti/15-tls.md`).
2. Preparare le 14 schede teoria (una per argomento) e 20 esercizi numerici con soluzioni dai materiali di Sereno/Kurose.
3. Chiedere o cercare la data dello scritto, il formato (carta o piattaforma, durata, se serve la calcolatrice) e il peso delle parti reti/sicurezza.
4. Il 10/1 controllo di fattibilità (vedi piano-settimanale).
