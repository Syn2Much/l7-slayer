# Slayer 7

Multi-vector Layer 7 DDoS toolkit built in Go. Routes all traffic through rotating proxies, pools connections for maximum throughput, and ships 6 attack methods — from raw volumetric floods to protocol-level exploits like HTTP/2 Rapid Reset. One binary, no config files, just flags and fire.

- 6 attack methods — GET/POST floods, RUDY slow POST, API/JSON flood, HTTP/2 Rapid Reset, WebSocket flood
- Rotating proxy support — HTTP CONNECT with auth, auto-dedup, raw CONNECT tunneling for h2/ws
- Connection pooling — 32 transports per unique proxy, keep-alive, zero-copy drain
- Lock-free atomic counters — live RPS ticker every 2s
- Randomized fingerprints — varied payloads, content types, headers, endpoints per request
- Auto-reconnect — dead connections rotate back instantly
- Single binary — `go build -o slayer .` and go

## Quick Start

```bash
  ______   __                                                 ______
 /      \ |  \                                               /      \
|  $$$$$$\| $$  ______   __    __   ______    ______        |  $$$$$$\ __     __  _______
| $$___\$$| $$ |      \ |  \  |  \ /      \  /      \       | $$___\$$|  \   /  \|       \
 \$$    \ | $$  \$$$$$$\| $$  | $$|  $$$$$$\|  $$$$$$\       \$$    \  \$$\ /  $$| $$$$$$$\
 _\$$$$$$\| $$ /      $$| $$  | $$| $$    $$| $$   \$$       _\$$$$$$\  \$$\  $$ | $$  | $$
|  \__| $$| $$|  $$$$$$$| $$__/ $$| $$$$$$$$| $$            |  \__| $$   \$$ $$  | $$  | $$
 \$$    $$| $$ \$$    $$ \$$    $$ \$$     \| $$             \$$    $$    \$$$   | $$  | $$
  \$$$$$$  \$$  \$$$$$$$ _\$$$$$$$  \$$$$$$$ \$$              \$$$$$$      \$     \$$   \$$
                        |  \__| $$
                         \$$    $$
                          \$$$$$$
go build -o slayer .

./slayer -t https://target.com -m httpget -w 4000 -d 30 -p proxies.txt
```

## Flags

```
-t  target URL         (required)
-m  attack method      (default: httpget)
-w  worker goroutines  (default: 2048)
-d  duration seconds   (default: 30)
-p  proxy file         (default: proxies.txt)
```

## Methods

| Method | Type | What It Does |
|--------|------|-------------|
| `httpget` | Volumetric | GET flood — tight loop, keep-alive, drain & discard |
| `httppost` | Volumetric | POST flood — 5 randomized form templates + JSON variants |
| `rudy` | Slowloris | R.U.D.Y. — declares 1–51 MB body, drips 1 byte/0.5–2.5s |
| `apiflood` | Application | REST/GraphQL — 6 payload generators, random endpoints & tokens |
| `rapidreset` | Protocol | CVE-2023-44487 — raw h2 HEADERS→RST_STREAM loop, 100-frame batches |
| `wsflood` | Persistent | WebSocket — mixed text/binary/ping frames, auto-reconnect |


## Proxy Format

```
http://user:pass@gateway:port
```

One per line. HTTP CONNECT with basic auth. Raw methods (`rapidreset`, `wsflood`) handle CONNECT tunneling and `Proxy-Authorization` internally.

## Dependencies

```
golang.org/x/net/http2    — raw h2 framing + hpack
github.com/gorilla/websocket — ws dial + framing
```
