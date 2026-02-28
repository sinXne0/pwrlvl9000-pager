# PWRLVL9000 — WiFi Pineapple Pager Pentest Console

A full-featured pentesting web UI payload for the **[WiFi Pineapple Pager](https://shop.hak5.org/products/wifi-pineapple-pager)** (M5 series).

Runs a Python3 HTTP server on port **9000**, serving a hacker-themed single-page app with 9 tool tabs accessible from any browser on the same network.

> **For authorized penetration testing and security research only.**

---

## Features

| Tab | Tool | Description |
|-----|------|-------------|
| **WiFi** | airodump-ng / aireplay-ng | Scan APs, capture handshakes, deauth clients |
| **Probe** | airodump-ng (client mode) | Sniff probe requests, track visible devices |
| **Rogue AP** | hostapd + dnsmasq | Evil twin AP + beacon flood (mdk4) |
| **WPS** | wash + reaver/bully | Scan WPS networks, launch pixie dust attack |
| **Web** | Python http.client | HTTP/HTTPS web scanner (headers, forms, paths) |
| **Network** | raw sockets + ARP | Port scanner + ARP table |
| **Capture** | tcpdump + hcxdumptool | Packet capture with BPF filter + PMKID capture |
| **Loot** | aircrack-ng + hcxpcapngtool | View handshakes, crack with wordlist, convert to `.22000` |
| **Shell** | /bin/sh | Root terminal with command history |

**UI extras:** Matrix rain background, glitch header, XP leveling system, animated necromancer sprite, live SSE event stream, toast notifications.

---

## Requirements

### On the Pager (OpenWrt)
- Python 3 (`python3-base`) — included on Pager
- `aircrack-ng` suite — `airodump-ng`, `aireplay-ng`, `aircrack-ng`
- `hostapd`, `dnsmasq` — for Rogue AP
- `wash`, `reaver` or `bully` — for WPS attacks
- `mdk4` or `mdk3` — for beacon flood
- `tcpdump` — for packet capture
- `hcxdumptool`, `hcxpcapngtool` — optional, for PMKID capture and `.22000` conversion

Tools are discovered automatically via `which` + path search (including `/mmc/usr/sbin/`). Missing tools produce a clean error rather than crashing.

---

## Install

### Copy to Pager

```bash
# From your local machine:
scp -r pwrlvl9000/ root@172.16.42.1:/sd/payloads/user/

# Or use the install script if present:
bash install.sh
```

### Activate

Load the payload from the Pager's payload menu. The web console starts on port **9000**.

Open from a connected device:
```
http://172.16.42.1:9000
```

Press **B** (back button) on the Pager to stop the server and return to the main menu.

---

## File Structure

```
pwrlvl9000/
├── payload.sh          # Pager launcher script
├── server.py           # Python3 backend (HTTP + SSE, ~600 lines)
├── pager_display.py    # Direct pager screen control (libpagerctl)
├── lib/
│   ├── pagerctl.py     # Pager hardware API wrapper
│   └── libpagerctl.so  # Native pager control library
└── web/
    ├── index.html      # SPA shell (9 tabs)
    ├── css/
    │   └── pwrlvl.css  # Neon green hacker theme
    └── scripts/
        ├── app.js      # Tab router, SSE client, matrix rain, XP system
        ├── wifi.js     # WiFi scan + attack
        ├── probe.js    # Probe sniffer + client tracker
        ├── rogueap.js  # Evil twin + beacon flood
        ├── wps.js      # WPS scan + pixie dust attack
        ├── webscan.js  # Web scanner
        ├── network.js  # Port scanner + ARP
        ├── capture.js  # Packet capture + PMKID
        ├── loot.js     # Handshake vault + crack + convert
        └── terminal.js # Root shell
```

---

## Architecture

- **Backend**: `http.server.SimpleHTTPRequestHandler` (Bjorn pattern, OpenWrt-compatible)
- **Threading**: `socketserver.ThreadingTCPServer` — each request in its own thread
- **Live updates**: Server-Sent Events (`/events`) push tool output to all connected browsers in real time
- **No external Python deps**: only stdlib (`http.server`, `socketserver`, `subprocess`, `json`, `socket`, `threading`, `queue`, `glob`, `re`)

---

## Legal

This tool is intended for use on networks and devices you own or have explicit written authorization to test. Unauthorized use is illegal. The author assumes no liability for misuse.
