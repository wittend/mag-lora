# mag-lora — agent context (personal, not shared)

Auto-loaded pi context file. Standing preferences and tooling notes for
working in this project tree.

## KiCad tooling — use the `konnect` MCP for PCB design work

Dave (2026-11-27): the **konnect** MCP server is available and should be
used when working on KiCad **PCB designs** in this project — preferred over
hand-editing `.kicad_pcb` s-expressions or kicad-cli alone.

- It's toolset-based: only `project` + `config` are loaded by default.
  Load what you need per task, e.g. for host-mag-usb PCB layout:
  `pcb_components` (place/repair footprints), `pcb_board` (outline/zones),
  `pcb_routing` (traces/vias/pours), `placement` (auto-placement),
  `verification` (DRC/design rules), `pcb_export` (Gerber/BOM/P&P).
- Other useful toolsets: `sch_components`/`sch_wiring`/`sch_batch`
  (schematic capture), `sch_export` (ERC, netlist, live-PCB sync),
  `sch_analysis`, `library` (author symbols/footprints), `integration`
  (Freerouting auto-routing, JLCPCB parts DB).
- Unload toolsets when done to keep context small.
- Plain `kicad-cli` via bash remains fine for quick one-off checks
  (`sch export netlist`, `pcb drc`) and for the xiao repo, which predates
  this note.
- Project layout context lives in `kicad/README.md` and
  `CURRENT_STATUS.md` (keep the latter current with each commit).

## Standing rules

- `CURRENT_STATUS.md` (umbrella) must be updated with each commit that
  changes project state.
- No pushes to Gitea/GitHub without explicit approval per push batch.
- Boring, standard tooling; C++ firmware; ESP32-S3; PlatformIO.
- Sub-project repos under `kicad/` are standalone git repos (gitignored
  from the umbrella) — each with Gitea `dave/*` remotes.
