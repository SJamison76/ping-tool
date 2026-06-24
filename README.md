# ST-Pinger-Tool Ping Scanner

A high-performance, stable network ping scanner for Windows 11. Built with Electron + Node.js. Designed to scan massive subnets (/16 = 65,534 hosts) without slowing your machine or dropping your network connection.


---

## Features

- **Massive subnet scanning** — CIDR notation (e.g. `10.16.0.0/16`), IP ranges (`10.0.0.1-10.0.0.254`), or single IPs
- **CSV import** — load IP lists from a file (one IP/CIDR/range per row, first column)
- **300 concurrent pings** — tuned for speed without network saturation
- **Live results** — rows appear as responses come in, color-coded green (alive) / red (dead)
- **Filter & search** — filter by All / Alive / Dead, search by IP substring
- **Export CSV** — save results at any time
- **Stats bar** — live counters for total, scanned, alive, dead, and scan rate (hosts/sec)
- **Frameless native window** — clean dark UI, no browser chrome

---

## Setup (First Time)
npm install
npm start

To build the .exe installer:
npm install
npm run build

### Prerequisites
- **Node.js** v18 or newer → https://nodejs.org
- **Windows 11** (also works on Windows 10)

### Install & Run

```bat
cd ping-scanner
npm install
npm start
```

That's it. The app window opens immediately.

### Build a Standalone `.exe` (Optional)

```bat
npm run build
```

Produces an installer in `dist/`. Double-click to install. No Node.js required after that.

---

## Usage

| Action | How |
|--------|-----|
| Scan a subnet | Type `10.16.0.0/16` in the Target box, click **▶ Start Scan** |
| Scan a range | Type `192.168.1.1-192.168.1.254` |
| Scan one host | Type `10.0.0.1` |
| Import IP list | Click **📂 Import CSV**, pick your file |
| Stop mid-scan | Click **⏹ Stop** or press `Esc` |
| Start scan (keyboard) | `Ctrl+Enter` |
| Filter results | Click **All / Alive / Dead** pills |
| Search by IP | Type in the Search IP box |
| Export results | Click **⬇ Export CSV** (enabled after scan) |
| Clear everything | Click **✕ Clear** |

---

## CSV Import Format

First column = IP address, CIDR, or range. One per row. Header rows are ignored automatically.

```
IP Address,Description
10.0.0.1,Router
10.0.0.0/24,Office subnet
192.168.1.1-192.168.1.50,Lab range
```

---

## Performance Notes

- **Concurrency: 300 simultaneous pings** — this is the sweet spot for Windows. Going higher can cause the OS to start dropping pings internally.
- **Timeout: 1000ms per host** — hosts that don't reply within 1 second are marked dead.
- **A /16 (65,534 hosts) takes roughly 3–5 minutes** depending on your NIC and network.
- **A /24 (254 hosts) takes under 5 seconds.**
- The scanner uses Windows' built-in `ping.exe` via child processes — no raw sockets, no NPCAP, no driver installation needed.

---

## Tuning (Advanced)

Edit `src/main.js` top section:

```js
const CONCURRENCY  = 300;   // simultaneous pings (try 200–500)
const PING_TIMEOUT = 1000;  // ms wait per host (try 500 for fast LANs)
const PING_COUNT   = 1;     // packets per host (keep at 1 for speed)
```

Lower `CONCURRENCY` if you notice dropped packets on your NIC. Raise it carefully if you have a high-end NIC and a fast switch.

---

## Architecture

```
main.js          — Electron main process: ping engine, IPC, file dialogs
preload.js       — Context bridge: exposes safe API to renderer
src/index.html   — All UI: HTML + CSS + vanilla JS renderer
```

The ping engine spawns worker "coroutines" (async functions pulling from a shared queue). Each worker calls `ping.exe -n 1 -w 1000 -4 <ip>`, parses the exit code and output, and streams the result to the renderer via IPC events.

Results are buffered in 60ms DOM batches to keep the UI smooth even at high scan rates.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `npm install` fails | Make sure Node.js ≥18 is installed and in PATH |
| App window doesn't open | Run `npm start` from inside the `ping-scanner` folder |
| All hosts showing DEAD | Check Windows Firewall isn't blocking outbound ICMP |
| Slow scan rate | Try lowering CONCURRENCY to 150 if NIC is overwhelmed |
| Build fails (icon) | Replace `assets/icon.png` with a real 256×256 PNG |

---

## License

MIT — use freely, modify as needed.
