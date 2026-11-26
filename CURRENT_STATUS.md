# mag-lora — Current Status

**Date:** 2026-11-23 · **Snapshot of:** repo organization + component status, with recommended next steps.
**Last updated:** 2026-11-26 — host-mag-usb decisions recorded (`813d98c`): **Q2 = ESP32-S3** (in-field config via WiFi + node parity), **Q4 = RX-only v1** (TX kept for future re-provisioning). Earlier today: mag-usb interop research done (`b702b48`) — **Q1 = Option A** (dongle emulates Pololu USB-I²C + RM3100 + MCP9808; stock PSWS stack unmodified); `host-mag-usb` sub-project created; Phase 1 prep (teardown checklist + xiao-carrier scaffold). All repos with remotes pushed.
**Convention:** keep this file current with each commit that changes project state.

## 1. Repository organization

Three git repos, nested:

```
mag-lora/                          ← umbrella repo (this one)
├── src/                           Go module `maglora` (host client — not started)
├── docs/ assets/ contrib/ tests/  umbrella-level material
├── requirements.md                goals / deliverables / constraints / non-goals
├── README.md                      umbrella layout + pointers
└── kicad/                         ← NOT tracked by the umbrella (gitignored)
    ├── README.md                  layout doc for the sub-projects
    ├── xiao-mag-lora/             ← standalone repo (active project)
    ├── wio-mag-lora/              ← standalone repo (production variant; initial commit, no remotes)
    └── host-mag-usb/              ← standalone repo (host receiver; initial commit, no remotes)
```

| Repo | Remotes | Role |
|---|---|---|
| `mag-lora` (umbrella) | Gitea `dave/mag-lora` · GitHub `wittend/mag-lora` | Shared requirements, cross-cutting docs, future Linux host client |
| `kicad/xiao-mag-lora` | Gitea `dave/xiao-mag-lora` · GitHub `wittend/xiao-mag-lora` | **The active project** — node firmware + host decoder + design docs |
| `kicad/wio-mag-lora` | — (no remotes yet) | Wio-S3 production variant — requirements + reference assets committed 2026-11-25 |
| `kicad/host-mag-usb` | — (no remotes yet) | Host-side LoRa receiver USB dongle (Phase 3) — power-rail skeleton + requirements, 2026-11-26 |

The umbrella intentionally does **not** track the sub-project repos (see
`kicad/README.md`); each keeps its own history and remotes.

## 2. Component status

### 2.1 Umbrella (`mag-lora`)
- Scaffold + requirements + layout docs + **`docs/pcb_design_plan.md`**
  (PCB design sketch, 2026-11-25). **Host client not started**
  (`src/go.mod` exists: module `maglora`, Go 1.24).
- `tests/`, `assets/`, `contrib/` are placeholders.

### 2.2 `xiao-mag-lora` (active) — PAUSED since 2026-10-25
Solar-powered P2P LoRa node on XIAO ESP32-S3 + Wio-SX1262 (Kit SKU
113010003, in hand). 26-byte binary frames, 1 rec/s, GNSS clock-only
discipline. US915 §15.247 — **no TX duty cycle** (evidence in `assets/fcc/`).

**Verified on 2026-11-23 (ran locally):**
- ✅ Host unit tests **31/31 pass** (`src/firmware/test/run_host_test.sh`) —
  codec byte-exact vs the documented worked example, ring buffer, seq
  wrap/loss, `$GPRMC`→epoch, CRC/magic rejection.
- ✅ Decoder self-test passes byte-for-byte (`src/host/decode.py --selftest`).
- ✅ Device firmware builds green (PlatformIO, env `wio_s3_lora`), real
  **SX1262 driver active** (RadioLib 7.8.1 jgromes fork), US915 / SF9 /
  125 kHz / 17 dBm.
- ✅ Design docs complete: hardware strategy, BOM, payload spec, calculations,
  PoC bench walkthrough; project-docs lifecycle series in place.

**Done (software for the first 2-node link):** frame codec, sample store,
link layer (seq + gap retransmit), NMEA parser, `Radio` interface + real
SX1262 driver, host JSON decoder, bench `Makefile`
(`build/flashA/flashB/linktest/cap/decode/selftest`).

**2026-11-25 update:** asset reorg for the final design (Wio-S3 docs moved
to the wio sub-project; L76K doc set + RM3100-CB KiCad symbol/footprints
added; all duplicates removed, incl. a reference-design zip from
Jeremy Erhardt). Requirements deltas: prefer off-the-shelf XIAO-series
components; SPA shared with the wio sibling. L76K PCN reviewed:
fix-LED removed only — pin/electrical interface unchanged.

**Open / blocking:**
- 🔶 **The one remaining unknown for the bench:** the 4 SX1262 **control
  pins** (CS/DIO1/BUSY/RST) in `include/xiao_pins.hpp` are `[BENCH-VERIFY]`
  best-guesses (SPI D8/D9/D10 confirmed). Firmware prints the pin map +
  `begin: OK/FAIL` at boot — fix path is a one-line edit per pin.
- **Not started:** real RM3100 + MCP9808 sensor reads (dummy sample today),
  GNSS on-device verification, WiFi/BLE provisioning (off in normal op),
  Linux host client, SPA web app, history storage, field node (solar +
  LiFePO4 + PVC enclosure), range evaluation, CI/artifacting (REQ-015).
- 🏗️ **KiCad project scaffolded** (`kicad/xiao-carrier`, KiCad 10, 5 subsystem
  sheets, lib tables) — schematic capture not started. Footprint gap: **L76K
  (109100021) not in the OPL library**; XIAO ESP32-S3 + Wio-SX1262 available
  there. Teardown decisions D1/D5 may reshape scope before capture.
- 160 MB on disk is the gitignored `.pio` toolchain cache, not source.

### 2.3 `wio-mag-lora` (production variant)
Initial commit 2026-11-25 (`6b2fefd`), no remotes yet. Requirements +
software requirements + reference assets (incl. the official Seeed Wio-S3
KiCad package) are committed. This is the home of the Wio-S3 production
form factor; xiao remains the active bench/PoC path.

### 2.4 `host-mag-usb` (host receiver, Phase 3) — new 2026-11-26
Standalone sub-repo (`fb08706`, no remotes yet). Seeded from Dave's
`USBStick-MagExtender` project (July 2026, USB/I2C magnetometer extender
power-rail skeleton: USB-C VBUS → AMS1117-3.3, ferrite bead, LEDs) —
renamed to `host-mag-usb` and brought into the project. Vendor USB-C/ESD
component libs retained (2sayle template origin, no upstream LICENSE —
noted in its README). **Q1 (interop surface) answered 2026-11-26** —
`docs/magusb_interop.md`: Option A — the dongle emulates the Pololu USB-I²C
adapter (115200 CDC-ACM) + RM3100 (0x20) + MCP9808, so stock mag-usb,
mag-recorder, and gmag_webui run **unmodified**; fallback B (patch mag-usb)
with a dongle input backend; bench path C (host-side JSONL relay) first.
**Q2 decided 2026-11-26: ESP32-S3** (in-field config via WiFi + node parity);
**Q4 decided: RX-only v1** (TX kept for future re-provisioning); Q3 (antenna)
still open.
ERC/DRC: only skeleton-typical noise
(lib resync mismatches, text style, unconnected pads).

## 3. Recommended next steps

**A. Prove the link (highest value, unblocks everything else):**
1. Flash + bench the 2-node link: `make build && make flashA PORTA=… &&
   make flashB PORTB=… && make linktest PORTB=…` (in `kicad/xiao-mag-lora`).
2. Resolve the 4 control pins if `begin: FAIL` — read the real pins off the
   Wio-SX1262-for-XIAO board (datasheet PDF p.6 in `assets/`), edit
   `include/xiao_pins.hpp`, reflash. Gate details in `docs/poc_walkthrough.md`.
3. Confirm: receiver heartbeat `rx=↑ lost=0`, JSON on stdout, gap detection
   by separating the nodes.

**B. Then, in order:**
4. Wire the real sensor (RM3100 magnetometer + MCP9808 temperature, I2C) —
   replace the dummy `read_sample()`.
5. Verify L76K GNSS lock + clock discipline on-device.
6. WiFi/BLE provisioning mode (REQ-010).
7. Start the Linux host client + SPA in the umbrella `src/` (REQ-012…014) —
   per requirements: Go/C/C++/Rust, portability across major distros.
8. Field node: solar + LiFePO4 + PVC enclosure (REQ-008/009), incl. the
   P1-Pro power-wiring teardown (Q5).
9. Range evaluation to the ≤1 km LoS target (REQ-017).

**C. PCB design (plan: `docs/pcb_design_plan.md`):**
10. Phase 1: xiao carrier v1 — KiCad project scaffolded (`kicad/xiao-carrier`);
    run the P1 teardown (checklist in xiao `docs/`), then schematic capture.
11. Phase 2: wio carrier (production form factor) — port the
    power/RF/sensor/GNSS blocks onto the Wio-S3 connector.
12. Phase 3: host receiver — `kicad/host-mag-usb` started (power-rail
    skeleton + requirements). Q1 (Option A), Q2 (ESP32-S3), Q4 (RX-only v1)
    decided; open Q3 (antenna) →
    schematic capture. First interop test: bench a host-side JSONL relay
    against mag-recorder.

**D. Housekeeping:**
13. Create Gitea/GitHub remotes for `wio-mag-lora` (kept as the home of
    the Wio-S3 production variant).
14. Add CI to the xiao repo (REQ-015) — at minimum `run_host_test.sh` +
    `decode.py --selftest` + `pio run -e native`.
15. Fill in the completion-report evidence (sections 3–4) in
    `kicad/xiao-mag-lora/project-docs/02-completion-report.md` as each
    requirement verifies.

**E. Push policy:** everything pushed to both remotes (Gitea + GitHub);
wio-mag-lora and host-mag-usb are local-only until remotes exist. Keep
pushing as work lands.
