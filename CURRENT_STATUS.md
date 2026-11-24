# mag-lora — Current Status

**Date:** 2026-11-23 · **Snapshot of:** repo organization + component status, with recommended next steps.

## 1. Repository organization

Three git repos, nested:

```
mag-lora/                          ← umbrella repo (this one)
├── src/                           Go module `maglora` (host client — not started)
├── docs/ assets/ contrib/ tests/  umbrella-level material (mostly empty)
├── requirements.md                goals / deliverables / constraints / non-goals
├── README.md                      umbrella layout + pointers
└── kicad/                         ← NOT tracked by the umbrella (gitignored)
    ├── README.md                  layout doc for the sub-projects
    ├── xiao-mag-lora/             ← standalone repo (active project)
    └── wio-mag-lora/              ← standalone repo (dormant, empty)
```

| Repo | Remotes | Role |
|---|---|---|
| `mag-lora` (umbrella) | Gitea `dave/mag-lora` · GitHub `wittend/mag-lora` | Shared requirements, cross-cutting docs, future Linux host client |
| `kicad/xiao-mag-lora` | Gitea `dave/xiao-mag-lora` · GitHub `wittend/xiao-mag-lora` | **The active project** — node firmware + host decoder + design docs |
| `kicad/wio-mag-lora` | — (no remotes, zero commits) | Dormant early Wio-S3 exploration; docs/datasheets only |

The umbrella intentionally does **not** track the sub-project repos (see
`kicad/README.md`); each keeps its own history and remotes.

## 2. Component status

### 2.1 Umbrella (`mag-lora`)
- Scaffold + requirements + layout docs. **Host client not started**
  (`src/go.mod` exists: module `maglora`, Go 1.24).
- `tests/`, `docs/`, `assets/`, `contrib/` are placeholders.

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

**Open / blocking:**
- 🔶 **The one remaining unknown for the bench:** the 4 SX1262 **control
  pins** (CS/DIO1/BUSY/RST) in `include/xiao_pins.hpp` are `[BENCH-VERIFY]`
  best-guesses (SPI D8/D9/D10 confirmed). Firmware prints the pin map +
  `begin: OK/FAIL` at boot — fix path is a one-line edit per pin.
- **Not started:** real RM3100 + MCP9808 sensor reads (dummy sample today),
  GNSS on-device verification, WiFi/BLE provisioning (off in normal op),
  Linux host client, SPA web app, history storage, field node (solar +
  LiFePO4 + PVC enclosure), range evaluation, CI/artifacting (REQ-015).
- ⚠️ **No KiCad PCB design exists yet** — the folder name says "kicad", but
  there are no `.kicad_pro/.sch/.pcb` files; only docs + datasheets.
- ⚠️ Repo hygiene (local, uncommitted): `requirements/xaio-mag-ora_…` →
  `xaio-mag-lora_…` rename staged; the xiao README still references the old
  filename in two places.
- 160 MB on disk is the gitignored `.pio` toolchain cache, not source.

### 2.3 `wio-mag-lora` (dormant)
Empty git repo (no commits, no remotes). Contains a proposed-approach
requirements file, `software_requirements.md`, and Wio-S3/SX1262/L76K
datasheets. Superseded by xiao as the active path (Wio-S3 remains the later
production form-factor target).

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

**C. Housekeeping:**
10. xiao repo: commit the requirements-file rename; fix the two stale
    `xaio-mag-ora` references in its README.
11. Decide the fate of `wio-mag-lora`: delete it, or keep it as the home for
    the later Wio-S3 production variant (in which case it needs its own
    remotes and real content).
12. Add CI to the xiao repo (REQ-015) — at minimum `run_host_test.sh` +
    `decode.py --selftest` + `pio run -e native`.
13. Fill in the completion-report evidence (sections 3–4) in
    `kicad/xiao-mag-lora/project-docs/02-completion-report.md` as each
    requirement verifies.
14. Start the KiCad design when the Wio-S3 modules arrive (production form
    factor).

**D. Push policy:** nothing has been pushed in this session; all umbrella
commits are local on `main`. Push to Gitea/GitHub when ready.
