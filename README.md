# Protocolul CHORD în MPI

**Tema 2 – Algoritmi Paraleli și Distribuiți (APD)**
Ciurea Ana Sorina – 331CC

Simulare a protocolului **CHORD** (Distributed Hash Table) folosind **MPI**. Fiecare proces MPI reprezintă un nod din inelul CHORD și comunică cu celelalte noduri pentru a rezolva cereri de lookup. Inelul este static, dar rutarea respectă complexitatea **O(log N)** prin folosirea *finger table*-urilor.

## Structura proiectului

| Fișier | Descriere |
|--------|-----------|
| `tema2.c` | Implementarea protocolului CHORD |
| `Makefile` | Compilare cu `mpicc` |
| `README.pdf` | Documentația originală a temei |

## Compilare și rulare

```bash
make            # generează executabilul tema2
mpirun -np <N> ./tema2
make clean
```

Fiecare rank `r` citește fișierul `in<r>.txt` cu formatul:

```
<ID CHORD al nodului>
<numărul de lookup-uri>
<cheie_1> <cheie_2> ... <cheie_k>
```

Spațiul de ID-uri are `2^M = 16` poziții (`M = 4`).

Pentru fiecare lookup, inițiatorul afișează traseul parcurs:

```
Lookup <key>: <id_1> -> <id_2> -> ... -> <responsabil>
```

## Arhitectură și implementare

### 1. Construirea finger table (`build_finger_table`)
Tabela este construită o singură dată, la inițializare:
- pentru fiecare intrare `i` (de la `0` la `M-1`): `start = (id + 2^i) % RING_SIZE`;
- succesorul lui `start` se caută în lista sortată a nodurilor existente (`sorted_ids`) cu `find_successor_simple`, care întoarce primul nod cu `ID >= start` (sau primul nod din inel, la wrap-around).

### 2. Rutare eficientă (`closest_preceding_finger`)
- se parcurge finger table-ul de la `M-1` la `0`;
- se returnează primul `finger[i].node` aflat în intervalul circular `(self.id, key)`;
- dacă nu există un astfel de finger, se întoarce succesorul direct (`self.successor`).

### 3. Gestionarea cererilor (`handle_lookup_request`)
Procesează mesajele `TAG_LOOKUP_REQ`:
1. adaugă ID-ul nodului curent în traseul mesajului;
2. dacă `key ∈ (self.id, self.successor]`, succesorul este responsabil → este adăugat în traseu și se trimite `TAG_LOOKUP_REP` către inițiator;
3. altfel, cererea este trimisă mai departe (`TAG_LOOKUP_REQ`) către nodul ales de `closest_preceding_finger`.

### 4. Protocolul de terminare
- fiecare nod își inițiază lookup-urile trimițând `TAG_LOOKUP_REQ` către propriul rank;
- în bucla de servire, nodul contorizează răspunsurile `TAG_LOOKUP_REP` primite;
- când și-a terminat toate cererile locale, trimite `TAG_DONE` tuturor celorlalte noduri;
- nodul se oprește doar după ce și-a terminat cererile **și** a primit `TAG_DONE` de la toți ceilalți participanți.

Comunicarea se face exclusiv cu `MPI_Send` / `MPI_Recv` și tag-urile `TAG_LOOKUP_REQ`, `TAG_LOOKUP_REP`, `TAG_DONE`.

## Punctaj

| Criteriu | Punctaj |
|----------|---------|
| Corectitudine | 40/40 |
| Eficiență | 40/40 |
| **Total** | **80/80** |
