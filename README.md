# mag-lora

A project for creating a remote sensor for gathering data from the RM-3100CM sensor module via a LoRa RF signalling.

## Layout

- `src/` — code
- `docs/` — longer documentation
- `assets/` — images, icons, static source material
- `contrib/` — community contributions, patches, ideas
- `tests/` — unit tests (see `tests/README.md` for how to run them)
- `requirements.md` — goals, constraints, non-goals
- `CHANGELOG.md` — what changed, per [Keep a Changelog](https://keepachangelog.com/)

## Runtime environment

Go (toolchain pinned in `go.mod`). Build and run:
```
go build ./...
go run ./cmd/<main>
```
Web frontend (templ + htmx): regenerate templ output with `go generate ./...`.
WASM components (C via wasi-sdk): build into `dist/` and run with wasmtime.

## Testing

Unit tests use the standard library `testing` package:
```
go test ./...
```
Other testing (wasmtime component smoke tests, manual checklists):
describe it here as it exists.

## License

MIT — see [LICENSE](LICENSE).
