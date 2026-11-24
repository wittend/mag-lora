# mag-lora

A remote sensor for gathering magnetometer data from the PNI RM-3100-CM via
LoRa RF signalling, interoperable with the HAMSCI Personal Space Weather
Station and similar projects. See [`requirements.md`](requirements.md) for
goals, deliverables, constraints, and non-goals.

## Repository organization

This is the **umbrella repo**: shared requirements, cross-cutting docs, and
the (future) Linux host-side client. The hardware node designs live in
**standalone sub-project repos** under `kicad/` — each has its own git
history and remotes and is gitignored here (see `kicad/README.md`):

| Sub-project | Path | What it is |
|---|---|---|
| xiao-mag-lora | `kicad/xiao-mag-lora/` | **Active.** Solar-powered point-to-point LoRa telemetry node on Seeed XIAO ESP32-S3 + Wio-SX1262: C++/PlatformIO firmware (real SX1262 driver), 26-byte frame codec, host decoder, design docs. Remotes: Gitea `dave/xiao-mag-lora`, GitHub `wittend/xiao-mag-lora`. |
| wio-mag-lora | `kicad/wio-mag-lora/` | Dormant. Earlier Wio-S3 integrated-module exploration — docs and datasheets only. |

## Layout

- `kicad/` — hardware sub-projects (standalone repos, see above)
- `src/` — host-side client code (Go module `maglora`; not yet implemented)
- `docs/` — cross-project documentation
- `assets/` — shared images, icons, static source material
- `contrib/` — community contributions, patches, ideas
- `tests/` — unit tests for host-side code
- `requirements.md` — goals, constraints, non-goals
- `CHANGELOG.md` — what changed, per [Keep a Changelog](https://keepachangelog.com/)
- `CURRENT_STATUS.md` — current organization and status snapshot

## Current status

Early phase. The node-side proof of concept (frame codec, SX1262 driver, host
decoder) is complete and host-tested in `kicad/xiao-mag-lora`; the first
2-node bench link test is pending. The Linux host client and web app are not
yet started. See `CURRENT_STATUS.md` for details.

## Testing

Host-side unit tests (Go) run from the repo root once code lands in `src/`:

```
go test ./...
```

Firmware and decoder tests live in `kicad/xiao-mag-lora` (PlatformIO + a
no-toolchain host test: `make selftest` there).

## License

MIT — see [LICENSE](LICENSE).
