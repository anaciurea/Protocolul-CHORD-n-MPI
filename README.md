# CHORD Protocol in MPI

**Assignment 2 – Parallel and Distributed Algorithms (APD)**
Ciurea Ana Sorina – 331CC

Simulation of the **CHORD** protocol (Distributed Hash Table) using **MPI**. Each MPI process represents a node in the CHORD ring and communicates with the other nodes to resolve lookup requests. The ring is static, but routing achieves **O(log N)** complexity by using *finger tables*.

## Project structure

| File | Description |
|--------|-----------|
| `tema2.c` | CHORD protocol implementation |
| `Makefile` | Build with `mpicc` |
| `README.pdf` | Original assignment documentation |

## Build and run

```bash
make            # builds the tema2 executable
mpirun -np <N> ./tema2
make clean
```

Each rank `r` reads the file `in<r>.txt` with the following format:

```
<CHORD ID of the node>
<number of lookups>
<key_1> <key_2> ... <key_k>
```

The ID space has `2^M = 16` positions (`M = 4`).

For each lookup, the initiator prints the path taken:

```
Lookup <key>: <id_1> -> <id_2> -> ... -> <responsible_node>
```

## Architecture and implementation

### 1. Building the finger table (`build_finger_table`)
The table is built only once, at initialization:
- for each entry `i` (from `0` to `M-1`): `start = (id + 2^i) % RING_SIZE`;
- the successor of `start` is searched in the sorted list of existing nodes (`sorted_ids`) using `find_successor_simple`, which returns the first node with `ID >= start` (or the first node in the ring, on wrap-around).

### 2. Efficient routing (`closest_preceding_finger`)
- the finger table is traversed from `M-1` down to `0`;
- the first `finger[i].node` located in the circular interval `(self.id, key)` is returned;
- if no such finger exists, the direct successor (`self.successor`) is returned.

### 3. Request handling (`handle_lookup_request`)
Processes `TAG_LOOKUP_REQ` messages:
1. adds the current node's ID to the message path;
2. if `key ∈ (self.id, self.successor]`, the successor is responsible → it is added to the path and a `TAG_LOOKUP_REP` is sent to the initiator;
3. otherwise, the request is forwarded (`TAG_LOOKUP_REQ`) to the node chosen by `closest_preceding_finger`.

### 4. Termination protocol
- each node initiates its lookups by sending `TAG_LOOKUP_REQ` to its own rank;
- in the serving loop, the node counts the received `TAG_LOOKUP_REP` replies;
- once all its local requests are completed, it sends `TAG_DONE` to all other nodes;
- a node stops only after it has completed its own requests **and** has received `TAG_DONE` from all other participants.

Communication is done exclusively with `MPI_Send` / `MPI_Recv` and the tags `TAG_LOOKUP_REQ`, `TAG_LOOKUP_REP`, `TAG_DONE`.

## Grading

| Criterion | Score |
|----------|---------|
| Correctness | 40/40 |
| Efficiency | 40/40 |
| **Total** | **80/80** |
