# kicad/ — hardware sub-projects

Each subdirectory is a **standalone git repository** with its own history and
remotes. They are gitignored from the umbrella `mag-lora` repo (see
`../.gitignore`), so they show up as untracked noise there and must not be
committed to it.

| Directory | Status | Remotes | Notes |
|---|---|---|---|
| `xiao-mag-lora/` | **Active** | Gitea `dave/xiao-mag-lora` · GitHub `wittend/xiao-mag-lora` | XIAO ESP32-S3 + Wio-SX1262 point-to-point LoRa node: C++/PlatformIO firmware, host decoder, design docs. See its `README.md` and `project-docs/`. |
| `wio-mag-lora/` | Production variant | Gitea `dave/wio-mag-lora` (2026-11-27) | Wio-S3 production variant — requirements + reference assets (incl. official Seeed KiCad package). Initial commit 2026-11-25. |
| `host-mag-usb/` | Phase 3 | Gitea `dave/host-mag-usb` (2026-11-27) | Host-side LoRa receiver USB dongle — schematic v1 (generated, netlist-verified `71d0d83`). Initial commit 2026-11-26 (seed: Dave's `USBStick-MagExtender`). |

Work inside each sub-project using its own git; keep cross-project material
(requirements, host client, shared docs) in the umbrella repo.
