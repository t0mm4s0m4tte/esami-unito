# Tecnologie Web: preparazione (scritto + progetto)

Fonti: Moodle corso 3637 (26/27), scheda del corso, forum del 2026-11-12, appunti `../Tecnologie-Web/appunti/`, progetto in `C:\Users\tommy\OneDrive\Documenti\Claude\03-projects\progetto-tweb-2026-2027`. Convenzione: `[STIMA]` mio, `[DA VERIFICARE]` da controllare.

## Formato

- Voto finale = media di due prove, ciascuna in trentesimi: **scritto di teoria** (domande a risposta chiusa e aperta) e **progetto con discussione**.
- Il 12/11 la docente Noemi Mauro ha scritto sul forum: le slide di JavaScript, TypeScript, Promise e React erano state caricate senza le slide nascoste, che fanno parte delle dispense; l'esame di teoria verterà su slide e dispense e avrà domande a risposta multipla e aperte. Ho scaricato le versioni aggiornate in `_moodle-raw/26-27-nuovi/Tecnologie-Web/`: il PDF di JavaScript/TypeScript è passato da 64 mila a 186 mila caratteri di testo (tre volte), quindi gli appunti `03` sono incompleti.
- Validità: se non superi lo scritto entro l'appello di settembre 2027 devi ripresentare il progetto, e viceversa.
- Progetto: front-end TypeScript + React, back-end Java + Spring (+ JPA), individuale o in coppia, versione "base" o "estesa" (18-30 punti la estesa). Per l'estesa servono visione (150-300 parole), ideazione (2-3 pagine con scenari, attori, obiettivi, casi d'uso), proof of concept e considerazioni finali (1-2 pagine). Discussione di 20-30 minuti con le slide del template e domande individuali; anche domande di teoria legate al progetto.
- Il vostro progetto è sulle pale eoliche (Aeolus): decisioni, tema e requisiti sono in `progetto-tweb-2026-2027` (OB, ES, OP, stato 10/11 con 7 punti aperti, tra cui la scelta di database e il nome dell'app).

## Scadenze del progetto

| Appello | Consegna | Discussione |
|---|---|---|
| Gennaio | 2027-01-24 | 2027-01-28 |
| Febbraio | 2027-02-08 | `[DA VERIFICARE]` |

Iscrizioni: ESSE3 chiude 1 settimana prima; slot su Moodle e consegna zip chiudono 3 giorni prima. Tutti i componenti della coppia si iscrivono.

## Consiglio sulla scadenza `[STIMA]`

Il progetto estesa richiede circa 90 ore (documenti, front-end, back-end, slide). Dal piano ne hai circa 54 prima del 24/1: non bastano. **Mira alla consegna dell'8 febbraio** e al giro di discussione di febbraio; usa il 24/1 solo se a metà gennaio sei già ben oltre il proof of concept. Lo scritto di teoria si può fare in gennaio, perché non dipende dal progetto.

## Preparare lo scritto

1. **Aggiornare gli appunti con le slide nuove**: `03` (JS/TS/Promise con le slide nascoste, raddoppiano) e un nuovo `05-react.md` (slide `03-TWeb-ReactTeoria-04-2627-ppt.pdf`, aggiornato il 12/11). Manca ancora Spring, JPA e sessioni/credenziali (non ancora a lezione). Lo faccio io: vedi "Da fare".
2. **Schede di teoria** per argomento (HTTP, DOM e eventi, closure e prototipi, TypeScript, Promise e async/await, React: componenti, props, state, hook, effetti; più Spring/JPA quando escono). Ogni scheda: definizione, un esempio di codice, un errore tipico.
3. **Quiz a risposta chiusa**: 10 domande per scheda, risposta in 30 secondi. Le domande aperte vanno allenate scrivendo 5 righe per argomento.
4. **Esercizi**: il GitLab `gitlab2.educ.di.unito.it/tweb2627` (LendABook, step 0-6) e gli esercizi Moodle di Vanilla JS e TypeScript. Il GitLab richiede il login e non l'ho aperto: se lo apri tu nel browser posso leggerlo.

## Piano del progetto (bozza)

1. Entro il 6/12: scelta database, nome app, scenario e casi d'uso; documenti di visione e ideazione (500 parole + 2-3 pagine).
2. 7-20/12: back-end minimo con Spring e H2, un caso d'uso completo end-to-end.
3. 21/12-10/1: front-end React con TypeScript sul caso d'uso, secondo caso d'uso.
4. 11-24/1: sessioni, credenziali, rifinitura, documento di considerazioni finali.
5. 25/1-7/2: slide col template, prova di discussione, consegna dell'8/2.
Dettaglio e decisioni sono nel `brain/` del progetto, non qui.

## Da fare

- ~~Riassumere le slide nuove di JS/TS e React~~: fatto il 2026-11-12 (`03` integrato, `05-react.md` nuovo).
- Controllare quando escono le slide di Spring/JPA.
- Chiedere al professore le questioni aperte di OB13 (icone, animazioni) prima di sviluppare il front-end.
