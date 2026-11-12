# ASD: preparazione allo scritto (2027-01-27)

Fonti: pagina Moodle "Modalità d'esame 2026-2027" (ASD-B), simulazione d'esame ASD-A 2025/26 (quiz "Simulazione esame", il tuo tentativo del 2026-06-03, 9 min, `Non ancora valutato`), appunti `../Algoritmi-Strutture-Dati/appunti/`. Convenzione: `[STIMA]` mio, `[DA VERIFICARE]` da controllare.

## Formato (da Moodle)

- Esame in due fasi, un solo voto: **scritto sulla teoria** (2/3 del voto, min. 18/30, senza lode) e **orale di laboratorio** (1/3, min. 18/30). Per l'orale si discutono gli esercizi del laboratorio, che vanno **consegnati su Git prima di fare lo scritto**.
- Scritto sulla piattaforma Esami, solo dai laboratori del Dipartimento, con prenotazione su ESSE3. Contiene domande chiuse, aperte ed esercizi di codifica su CodeRunner.
- Durata dello scritto: non indicata `[DA VERIFICARE]`.

## La simulazione 2025/26 (33 punti)

| # | Tipo | Punti | Cosa chiede | Tuo risultato |
|---|---|---|---|---|
| 1 | Abbinamento sì/no | 4 | 4 affermazioni asintotiche (Ω, Θ, O) | 3/4 |
| 2 | Scelta multipla | 3 | Ricorrenza di un algoritmo ricorsivo, svolgimento, ordine di grandezza | 0/3 |
| 3 | CodeRunner (C) | 5 | `degree(kTree t)`: grado di un albero in rappresentazione figlio-fratello | 0/5 |
| 4 | CodeRunner (C) | 4 | `maxHeapify` per heap massimo | 0/4 |
| 5 | CodeRunner (C) | 3 | `merge` distruttivo di liste ordinate in O(p+q) | 0/3 |
| 6 | Aperta | 8 | `Moo`: cosa calcola, Θ del tempo, algoritmo asintoticamente migliore | risposta data, non valutata |
| 7 | Aperta | 6 | Grafo bipartito: partizione o motivo per cui non esiste | non data |

Quello che dice: **la codifica in C vale 12 punti su 33 (36%)** e a mano non l'hai fatta. Chi passa lo scritto deve scrivere a mano liste, alberi e heap. Il resto è asintotica, ricorrenze e analisi (21 punti).

## Soluzioni della simulazione

**1.** `3^(n-2)+2^(n+5) ∈ Ω(3^n)`: sì (`3^(n-2) = 3^n/9`, c = 1/9). `(n²+6n+9)/(n+1) ∈ Θ(n)`: sì, perché `(n+3)²/(n+1)` cresce come n. `∈ Θ(n²)`: no. `800n+n²+n log n ∈ O(n log n)`: no (c'è n², che domina).

**2.** `T(n) = T(n-1) + n²` (il testo dell'algoritmo non era leggibile dall'export `[DA VERIFICARE]`); dopo k svolgimenti `T(n-k) + Σ (n-i)²` e il risultato `Θ(n³)`. Trucco: leggi la forma `T(n) = T(n-1) + f(n)`: la ricorrenza lineare somma `f` n volte; se `f = n²`, la somma è `Θ(n³)`.

**3.** `degree`:
```c
int degree(kTree t) {
    int c = 0, m = 0;
    for (kTree f = t->child; f != NULL; f = f->sibling) {
        c++;
        int d = degree(f);
        if (d > m) m = d;
    }
    return c > m ? c : m;
}
```
Ricorda: nel formato figlio/fratello `child` è il primo figlio, `sibling` il successivo. Il grado di un nodo è la lunghezza della catena `child → sibling → …`.

**4.** `maxHeapify` (nota: `left`/`right` ritornano `i` se il figlio non esiste):
```c
void maxHeapify(int H[], int i, int hd) {
    int m = i, l = left(i, hd), r = right(i, hd);
    if (l != i && H[l] > H[m]) m = l;
    if (r != i && H[r] > H[m]) m = r;
    if (m != i) {
        int t = H[i]; H[i] = H[m]; H[m] = t;
        maxHeapify(H, m, hd);
    }
}
```

**5.** `merge` distruttivo O(p+q) (nessuna `malloc`, cambi solo i `next`):
```c
list merge(list l, list m) {
    if (l == NULL) return m;
    if (m == NULL) return l;
    list head, t;
    if (l->info <= m->info) { head = l; l = l->next; } else { head = m; m = m->next; }
    t = head;
    while (l != NULL && m != NULL) {
        if (l->info <= m->info) { t->next = l; l = l->next; }
        else { t->next = m; m = m->next; }
        t = t->next;
    }
    t->next = (l != NULL) ? l : m;
    return head;
}
```

**6.** `Moo` calcola la massima differenza in valore assoluto tra due elementi, cioè `max(A) - min(A)`. Il tempo è **Θ(n²)**: il doppio ciclo fa sempre `n(n-1)/2` confronti, in ogni caso. Algoritmo migliore: una sola scansione che tiene minimo e massimo, ritorna `max - min`: **Θ(n)**. Errori della tua risposta: "calcola il valore assoluto della differenza tra due celle" non dice cosa calcola (ma cosa fa una riga); l'ordinamento + ricerca binaria dà `n log n`, che migliora il quadratico ma non è il migliore possibile, e la ricerca binaria non serve. Regola: descrivi il *risultato*, non le istruzioni; poi chiedi "si può fare in una passata?".

**7.** (L'immagine del grafo non è nell'export.) Metodo: visita in ampiezza dal nodo di partenza colorando a livelli alterni; se trovi un arco tra due nodi dello stesso colore, il grafo non è bipartito e quell'arco chiude un ciclo di lunghezza dispari (che è il motivo da scrivere); altrimenti `V1` = colore 0, `V2` = colore 1. Un grafo è bipartito se e solo se non ha cicli di lunghezza dispari. Su grafi sconnessi ripeti da ogni componente.

## Piano di esercizio (con le tue lacune per primo)

Regola generale: **si scrive a mano**, senza appunti, con il timer. Poi si confronta con gli appunti e si segnano gli errori in una lista, che si rilegge il giorno prima.

1. **Codifica in C (prima priorità, 12 punti)**: fai questi, 25 minuti l'uno, due per sessione.
   - liste: `reverse` (ricorsivo e iterativo), `deleteElem`, `insertOrdered`, `union` di liste ordinate, `merge` e `split` per il Mergesort (appunti 04, `12-esercizi-svolti.md` sezione 6);
   - alberi: altezza, numero di foglie, `degree`, `search`/`insert`/`successor`/`delete` su BST (06, sezione 8);
   - heap: `maxHeapify`, `extractMax`, `buildMaxHeap`, `heapSort` (08);
   - hash: inserimento e ricerca a concatenamento e a indirizzamento aperto (05, sezione 7).
2. **Asintotica e ricorrenze (7 punti)**: 10 affermazioni di Θ/O/Ω al giorno con la tecnica del limite del rapporto e della sostituzione di variabile. Ricorrenze: `T(n)=aT(n/b)+f(n)` con il Master theorem, `T(n)=T(n-1)+f(n)` con la somma, `T(n)=2T(n-1)+f(n)` (esponenziale). Disegna l'albero di ricorsione.
3. **Algoritmi in pseudocodice (14 punti)**: i tipi della simulazione sono il "cosa calcola questo ciclo/algoritmo", il Θ del tempo (tutti i casi) e la versione più veloce. Allenati con gli esercizi 5.x dell'eserciziario (analisi di algoritmi) e con quelli sui grafi (BFS, DFS, bipartito, ciclo, componenti).
4. **Dimostrazioni ricorrenti**: halting, `Ω(n log n)` per l'ordinamento, correttezza di Dijkstra, invarianti di Selection e Insertion: una pagina a testa da riscrivere senza guardare.
5. **Simulazione a tempo**: una a settimana da fine novembre a metà gennaio (piano-settimanale). Struttura da replicare: 15 min domande chiuse, 20 min ricorrenze/asintotica, 50 min codifica, 35 min aperte. Durata totale 2 h `[STIMA]`; adatta quando saprai la durata vera.

## Materiali

- Eserciziario Moodle (ASD-B 26/27, cartella "Eserciziario") e per ogni lezione "Esercizi" + "Svolgimenti" (ID risorse 294847-294971): il PDF è già nei raw per ASD-A, non ancora ristudiato per B; usa le versioni con soluzione solo dopo averle provate.
- Quiz Moodle dei temi: bandiera/bubblesort, divide-et-impera, mediana (Find), programmazione dinamica, liste: sono allenamento a risposta chiusa, valgono prima del 27/1.
- "Esame di prova" sulla piattaforma Esami (`esami.i-learn.unito.it`): richiede un login separato, non entrato. Se lo apri nel browser e mi dici di continuare, scarico le tracce e le soluzioni.

## Da fare

- Controllare ora lo stato della consegna dei laboratori su Git.
- Prenotare lo scritto su ESSE3 (data 2027-01-27 dal piano).
- Aggiungere agli appunti: grafi bipartiti (BFS a due colori) `[INT]` e CodeRunner (come si testa in locale).
