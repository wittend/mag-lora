# PCB Design Plan (sketch)

*Status: sketch — intentionally not final. Neither KiCad design is in final
form; this document frames the work and the decisions that shape it.*

## 1. Scope — what gets designed

Three boards, in two tiers:

| Board | Tier | Variant | Purpose |
|---|---|---|---|
| **Remote carrier** | custom | **xiao** (active) | Carry + interconnect XIAO ESP32-S3 (Sense) + Wio-SX1262 + L76K + sensors; power tree from P1-Pro; RF + antenna; mechanical fit to 2" PVC |
| **Remote carrier** | custom | **wio** (production form factor) | Same concept on the integrated Wio-S3 Wireless LoRa module; reuses power/RF/sensor/GNSS blocks, swaps the MCU/radio connector block |
| **Host receiver** | custom *or* off-the-shelf | — | LoRa RX + USB to the Linux host; feeds the host client / mag-usb interop path |

**Off-the-shelf (per 2026-11-25 requirement: "off-the-shelf Seed Studio
XIAO series where possible"):** XIAO ESP32-S3 (Sense), Wio-SX1262 (for
XIAO), L76K (for XIAO), Wio-S3, FPC antenna A-02, SenseCAP P1-Pro (power
subsystem: solar + 4× 18650 LiFePO4 charge mgmt, its own radio unused).
**Custom:** the carrier(s) and possibly the host receiver (a USB LoRa dongle
can serve v1 — see §4). The RM3100-CB is an existing board (no custom
design) — we mount it, using the `RM3100-CB.pretty` footprints now in
`kicad/*/assets/`.

## 2. Remote carrier — subsystems (both variants)

1. **Compute/radio interface**
   - xiao: XIAO ESP32S3-Sense footprint/header stack; Wio-SX1262 plugs into
     the XIAO expansion side; L76K likewise. Pin map per
     `xiao_pins.hpp` (SPI D8/D9/D10 confirmed; CS/DIO1/BUSY/RST D11/D12/D1/D0
     `[BENCH-VERIFY]`).
   - wio: Wio-S3 module footprint + connector (official Seeed KiCad files
     are in `kicad/wio-mag-lora/assets/` — footprint, symbol, full schematics).
2. **Power** — P1-Pro 12.8 V (or its 5 V output, pending Q5 teardown) →
   buck to 5 V for the compute stack; battery-voltage sensing; reverse
   polarity + current limiting; power sequencing so radio TX peaks don't
   brown out the S3. (BOM #6.)
3. **Sensor interface** — RM3100-CB (I2C, D4/D5) + MCP9808 temperature
   (same I2C bus); 4.7 k pull-ups; ESD on the bus. Mounting choice open:
   module footprint vs. pads-only (`RM3100-CB.pretty` has both) vs. header
   socket (cf. `PinSocket_PNI_RM3100.kicad_mod` in the Jeremy Erhardt
   reference zip).
4. **RF** — LoRa: wio variant uses the Wio-S3's onboard SMA; xiao variant
   has IPEX on the Wio-SX1262 → IPEX→SMA or IPEX feed-through. 50 Ω
   microstrip, ESD/TVS at the connector. 2.4 GHz FPC A-02 connector for
   WiFi/BT provisioning (off in normal op).
5. **GNSS** — L76K UART (D6/D7 @ 9600, clock-only) + active GNSS patch
   antenna supply. PCN checked 2026-11-25: no pin/electrical changes
   (fix-LED removed) — interface unaffected; lock must be confirmed in
   firmware (parse NMEA), not by LED.
6. **Status/I-O** — user LED, optional test points for bench bring-up,
   battery/solar status sensing.

**Mechanical (both variants):** must fit 2" Sch 40 PVC (≈52.5 mm ID) —
circular board ~50 mm max, or stacked 2-layer boards; end-cap SMA and
GNSS-antenna penetrations gasketed. 1"–1.5" pipe fitting provisions for
pole mount.

**Pin-budget note (xiao):** the XIAO exposes ~11 GPIOs and the current pin
map uses most of them. Anything new (battery sense, fuel gauge) must reuse
I2C (MCP9808 bus) or a voltage divider on a still-free pin — do a full
pin audit in the xiao design before finalizing.

## 3. Host-side receiver

Two paths, decide at phase 3:
- **A. Off-the-shelf USB LoRa dongle (SX1262-class) for v1** — fastest to
  the interop milestone; host client (Go/C/Rust) reads frames over USB-serial.
- **B. Custom USB board** — USB-C + USB bridge (CP2102/CH340) or native-USB
  MCU + SX1262 (SPI) + SMA/IPEX + LNA. Cost-conscious production target per
  the umbrella requirements.

Either way the host-side firmware decodes the 26 B frames (codec already
exists, host-tested) and presents the data to the **mag-usb / HAMSCI
Personal Space Weather Station** path. **Open:** the exact interop surface —
virtual-COM emulating the RM3100-CB serial protocol vs. direct data feed to
mag-usb's pipeline. Confirm against the HAMSCI mag-usb docs before
committing board B.

## 4. Design inputs on hand (verified 2026-11-25)

- `RM3100-CB.kicad_sym` (8-pin symbol, KiCad 8.0) + `RM3100-CB.pretty/`
  (module + pads-only footprints) — in both sub-project `assets/`
- Official **Wio-S3 KiCad package** (footprint, symbol, power-topology /
  LoRa / interface schematics) — `kicad/wio-mag-lora/assets/`
- Datasheets: Semtec SX1261/SX1262, ESP32-S3R8, L76K (+ PCN — no changes),
  RM3100-CB user manual, P1-Pro wiki/docs, Wio-SX1262-for-XIAO
- `Jeremy Erhardt-….zip` — reference RM3100 remote designs (Eagle/KiCad,
  incl. `rm3100_Pi_Hat_remote_minimal` + RM3100 pin-socket footprint)
- XIAO pin-multiplexing + sleep-mode docs; `reference-design-of-s3.png`
- Existing strategy: `kicad/xiao-mag-lora/docs/hardware_strategy.md`
  (P2P model, power subsystem decision, mechanical) + `docs/bom.md`

## 5. Suggested stackup / DRC defaults

- 2-layer, 1 oz Cu, ENIG or immersion silver; 0.15 mm min track/space
  (consumer fab) — no high-speed constraints expected.
- RF: 50 Ω controlled impedance on outer layer; keep LoRa + 2.4 GHz traces
  short, no splits under them; antenna ESD/TVS ≤ 0.5 pF.
- Board outline circular ≤ 50 mm (PVC constraint); 3–4 mounting posts or
  tab slots to pipe standoffs.

## 6. Phasing

1. **Phase 0 (pending, still the gate):** 2-node bench link on the PoC kit —
   resolve the 4 `[BENCH-VERIFY]` control pins, prove RX/loss/JSON.
2. **Phase 1:** **xiao carrier v1** — design around parts already in hand
   (XIAO + Wio-SX1262 + L76K); do the P1-Pro teardown (Q5) first; bench the
   carrier with the PoC firmware before ordering anything new.
3. **Phase 2:** **wio carrier** — port power/RF/sensor/GNSS blocks onto the
   Wio-S3 connector; this is the production form factor.
4. **Phase 3:** **host receiver** — off-the-shelf dongle first for the
   interop test with mag-usb; custom board only if the production
   requirement demands it.
5. **Phase 4:** production hardening — enclosures/sealing verification,
   BOM cost pass, panelization, range test (REQ-017).

## 7. Open questions (design-blocking marked *)

1. *\* P1-Pro teardown (Q5):* 5 V tap point, battery-voltage access, radio
   disabled, connector choice.
2. *\* Wio-SX1262 control pins:* bench-verify before the xiao carrier locks
   the header/side routing (firmwise prints the map; one-line fix path).
3. Host interop surface: mag-usb virtual-COM emulation vs. direct feed (§3).
4. Battery monitoring: divider ADC (needs a free pin) vs. I2C fuel gauge on
   the MCP9808 bus.
5. GNSS antenna: active patch model + supply current; top-hat vs. cap
   feed-through in the pipe.
6. RM3100-CB mounting: module footprint vs. pads vs. socket (§2.3).
7. 2-layer vs. 4-layer: 2-layer assumed; revisit only if RF bring-up
   demands better isolation.
