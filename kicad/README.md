# kicad/ — hardware sub-projects

Each subdirectory is a **standalone git repository** with its own history and
remotes. They are gitignored from the umbrella `mag-lora` repo (see
`../.gitignore`), so they show up as untracked noise there and must not be
committed to it.

| Directory | Status | Remotes | Notes |
|---|---|---|---|
| `xiao-mag-lora/` | **Active** | Gitea `dave/xiao-mag-lora` · GitHub `wittend/xiao-mag-lora` | XIAO ESP32-S3 + Wio-SX1262 point-to-point LoRa node: C++/PlatformIO firmware, host decoder, design docs. See its `README.md` and `project-docs/`. |
| `wio-mag-lora/` | Dormant | — (empty repo, no commits) | Earlier Wio-S3 integrated-module exploration; docs + datasheets only. |

Work inside each sub-project using its own git; keep cross-project material
(requirements, host client, shared docs) in the umbrella repo.
