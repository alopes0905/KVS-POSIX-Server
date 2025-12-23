# KVS Subscription Server (Linux · POSIX · Concurrency)

A **Key-Value Store** implementation designed to practice **Operating Systems** fundamentals: POSIX APIs, multithreading, synchronization, inter-process communication, and persistence-oriented workflows.

**Highlights**
- In-memory **key → value** store with common operations (read/write/delete/show)
- **Job processing** from `.job` files with deterministic `.out` outputs
- **Concurrent execution** via a worker thread pool
- **Non-blocking backups** via `fork()` with a configurable maximum of simultaneous backups
- **Client/server mode** over **named pipes (FIFOs)** with **subscriptions** and **notifications**

> Platform: **Linux only** (POSIX). Windows is not supported.

---

## Build

Requirements: `make`, a C/C++ toolchain (e.g., `gcc/g++`), and POSIX support.

```bash
make
```

Clean:
```bash
make clean
```

---

## Run (example)

### 1) Start the server

Typical arguments:
- `jobs_dir` — directory containing `.job` files
- `max_threads` — number of worker threads
- `max_backups` — maximum concurrent backups
- `register_fifo` — FIFO path used for client registration

Example:
```bash
./src/server/kvs ./src/server/jobs 4 2 /tmp/register_fifo
```

### 2) Start a client

```bash
./src/client/client c1 /tmp/register_fifo
```

Where:
- `c1` is the client identifier
- `/tmp/register_fifo` is the same registration FIFO used by the server

---

## Jobs workflow

Place `*.job` files inside the `jobs_dir`. For each `X.job`, the server produces `X.out`.

- Lines starting with `#` are treated as comments.
- Commands are executed in order.
- `WAIT` pauses execution for the specified duration.

### Typical commands

- `WRITE [(k1,v1)(k2,v2)...]`
- `READ [k1,k2,...]`
- `DELETE [k1,k2,...]`
- `SHOW`
- `WAIT <ms>`
- `BACKUP`

> Backup file naming and exact output formatting depend on the implementation, but backups are launched in child processes and are limited by `max_backups`.

---

## Client/server (FIFOs) — subscriptions & notifications

Clients connect to the server using FIFOs. Once connected, a client can subscribe to keys and receive notifications whenever the value changes.

Common actions:
- connect / disconnect
- subscribe / unsubscribe
- receive notifications in a dedicated FIFO

Notification format (typical):
- `(key,value)` when updated
- `(key,DELETED)` when removed

---

## Project layout (typical)

This may vary slightly depending on your folder structure.

- `src/server/` — server implementation and entry point
- `src/client/` — client implementation and entry point
- `src/common/` or shared headers — protocol/utilities (if used)
- `src/server/jobs/` — example `.job` files (optional)

---

## Troubleshooting

### FIFO already exists
If you restart frequently, you may need to remove stale FIFOs:
```bash
rm -f /tmp/register_fifo
```

### Permissions
Prefer using `/tmp` for FIFOs, or ensure the chosen directory is writable.

---


## Author

André Miguel Carvalho Lopes  
- GitHub: https://github.com/alopes0905  
- LinkedIn: https://www.linkedin.com/in/andre-carvalho-lopes
