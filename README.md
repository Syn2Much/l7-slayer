# Slayer 7 - Application Layer Attack Suite 



A high-performance, multi-method HTTP stress testing tool written in Go. Supports multiple attack vectors, proxy rotation, and concurrent worker pools.

> **⚠️ Disclaimer:** This tool is intended for authorized security testing and research purposes only. Only use against systems you own or have explicit written permission to test. Unauthorized use is illegal.

---

## Features

- **6 Attack Methods** — HTTP GET, HTTP POST, HTTP/2 Rapid Reset, WebSocket Flood, RUDY (slow POST), API JSON Flood, 
- **Proxy Support** — Load proxies from file with automatic deduplication and connection pooling (32 clients per proxy)
- **Direct Mode** — Run without proxies for local/authorized testing
- **Massive Concurrency** — Default 2048 workers with configurable count
- **Randomized Fingerprints** — 300+ user agents spanning Chrome, Firefox, Edge, Safari, mobile browsers, and more
- **Dynamic Payloads** — Randomized form data, JSON bodies, GraphQL queries, and binary blobs per request
- **Live Statistics** — Real-time RPS, sent count, and error tracking with colored terminal output
- **HTTP/2 Rapid Reset** — Raw framer implementation of CVE-2023-44487 with HPACK encoding and batched frame writes
- **RUDY (R U Dead Yet)** — Slow POST that drips bytes at 0.5–2.5s intervals to hold connections open indefinitely

---

## Installation

### Prerequisites

- Go 1.21+

### Build

```bash
go mod init slayer
go mod tidy
go build -o slayer .
```

### Dependencies

```
github.com/gorilla/websocket
golang.org/x/net/http2
golang.org/x/net/http2/hpack
```

---

## Usage

```
slayer -t <url> [-m method] [-w workers] [-d duration] [-p proxyfile]
```

### Flags

| Flag | Default | Description |
|------|---------|-------------|
| `-t` | *(required)* | Target URL (e.g. `https://example.com`) |
| `-m` | `httpget` | Attack method |
| `-w` | `2048` | Number of concurrent workers |
| `-d` | `30` | Duration in seconds |
| `-p` | *(none)* | Path to proxy file (one per line, omit for direct mode) |

### Methods

| Method | Description |
|--------|-------------|
| `httpget` | Standard HTTP GET flood |
| `httppost` | HTTP POST with randomized form/JSON payloads |
| `rudy` | Slow POST — declares huge Content-Length, drips 1 byte at a time |
| `apiflood` | JSON API flood with randomized endpoints and complex nested payloads |
| `rapidreset` | HTTP/2 Rapid Reset (CVE-2023-44487) — sends HEADERS+RST_STREAM in batches |
| `wsflood` | WebSocket connection flood with mixed text/binary/ping messages |

### Examples

```bash
# Basic GET flood for 60 seconds
./slayer -t https://target.com -m httpget -d 60

# POST flood through proxies with 4096 workers
./slayer -t https://target.com -m httppost -w 4096 -d 120 -p proxies.txt

# RUDY slow POST attack
./slayer -t https://target.com -m rudy -w 500 -d 300

# HTTP/2 Rapid Reset through proxies
./slayer -t https://target.com -m rapidreset -w 1024 -p proxies.txt

# WebSocket flood
./slayer -t https://target.com -m wsflood -w 2048 -d 60

# API JSON flood
./slayer -t https://target.com -m apiflood -w 2048 -d 90
```

---

## Proxy File Format

One proxy URL per line. Supports HTTP/HTTPS proxies with optional authentication:

```
http://proxy1.example.com:8080
http://user:pass@proxy2.example.com:3128
socks5://proxy3.example.com:1080
```

---

## Architecture

### Connection Pooling

- **Proxied mode:** 32 `http.Client` instances per unique proxy, each with independent transport connection pools (up to 250 connections per host)
- **Direct mode:** Pool of 32 direct clients
- Workers randomly select a client from the pool on each iteration

### Payload Generation

POST and API flood methods generate randomized payloads on every request including login forms, search queries, feedback forms, multi-param spam, base64 blobs, bulk JSON inserts, deeply nested objects, and GraphQL mutations. Content type alternates between `application/x-www-form-urlencoded` and `application/json`.

### HTTP/2 Rapid Reset

Opens a raw TLS connection with ALPN `h2` negotiation, sends the HTTP/2 client preface, then continuously writes batches of 100 HEADERS + RST_STREAM frame pairs using a buffered framer. Supports CONNECT tunneling through proxies. A background goroutine consumes server frames to prevent connection stalls.

### RUDY

Declares a Content-Length of 1–51 MB, then sends the body 1 byte at a time with 0.5–2.5 second delays between each byte. The payload loops infinitely to keep the connection open until the stop signal.

---


## License

For authorized testing only. Use responsibly.
