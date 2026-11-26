# kicad/ — hardware sub-projects

Each subdirectory is a **standalone git repository** with its own history and
remotes. They are gitignored from the umbrella `mag-lora` repo (see
`../.gitignore`), so they show up as untracked noise there and must not be
committed to it.

| Directory | Status | Remotes | Notes |
|---|---|---|---|
| `xiao-mag-lora/` | **Active** | Gitea `dave/xiao-mag-lora` · GitHub `wittend/xiao-mag-lora` | XIAO ESP32-S3 + Wio-SX1262 point-to-point LoRa node: C++/PlatformIO firmware, host decoder, design docs. See its `README.md` and `project-docs/`. |
| `wio-mag-lora/` | Production variant | — (no remotes yet) | Wio-S3 production variant — requirements + reference assets (incl. official Seeed KiCad package). Initial commit 2026-11-25. |
| `host-mag-usb/` | New (Phase 3) | — (no remotes yet) | Host-side LoRa receiver USB dongle — power-rail skeleton + requirements. Initial commit 2026-11-26 (seed: Dave's `USBStick-MagExtender`). |

Work inside each sub-project using its own git; keep cross-project material
(requirements, host client, shared docs) in the umbrella repo.
