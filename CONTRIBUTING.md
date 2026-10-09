# Contributing

## Commit messages — Conventional Commits

All commits must follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

- **subject**: imperative mood, no trailing period, ≤ 72 chars.
  Explain *what changed and why*, not how.
- **scope** (recommended): the affected area, from the list below.
- **footer**: use `Refs: <issue>` / `Closes: <issue>` to link issues.

### Types

| Type       | Use for                                                        |
|------------|----------------------------------------------------------------|
| `feat`     | A new schematic block, new component/net, new board feature    |
| `fix`      | A correction to the design (wrong value, wrong pin, DRC fix)   |
| `docs`     | Specs, README, CONTRIBUTING, comments/annotations              |
| `refactor` | Reorganization with no design change (sheet split, net rename) |
| `test`     | ERC/DRC run scripts, bring-up checklists, measurements         |
| `chore`    | Tooling, repo housekeeping, license, dependency updates        |
| `build`    | Build/CI changes (rare in this repo)                           |

### Scopes

| Scope              | Covers                                                        |
|--------------------|---------------------------------------------------------------|
| `power_input`      | Block 1 sheet + its components                                |
| `power_buck`       | Block 2 sheet (5 V / 3.3 V rails)                             |
| `mcu`              | ESP32-S3 sheet                                                |
| `imu`              | BMI270 sheet + power isolation                                |
| `motors`           | TB67H450 / LPF2 sheet                                         |
| `servos`           | Servo sheet                                                   |
| `i2c_bus`          | TCA9548A + external I2C/SPI ports                             |
| `interface`        | Buzzer + RGB LED sheet                                        |
| `board`            | `arta_board.kicad_pcb`: stackup, outlines, layers, DRC rules  |
| `bom`              | `COMPONENT_LIST_BLOCK_1.md`, LCSC codes, sourcing             |
| `specs`            | `docs/specs/*`                                                |
| `datasheets`       | `docs/datasheets/*`                                           |

A commit touching several areas: use the primary scope in the header and
list the rest in the body.

### Examples

```
feat(power_input): place U1 LTC2955-2 and support network per datasheet p.9-11
fix(power_input): swap D1 orientation, cathode to VBAT_P (ERC pin_to_pin)
docs(bom): confirm F1 = Bourns MF-R150 1812, LCSC C123456, stock ok
feat(board): set 4-layer stackup to JLCPCB 1.6mm standard (1oz/0.5oz/0.5oz/1oz)
chore: initialize repository with CERN-OHL-P license
```

### Breaking changes

A design change that invalidates already-reviewed work (topology change,
part replacement, net-class change) is a **breaking change**: add `!` after
the type/scope and a `BREAKING CHANGE:` footer describing the impact.

```
feat!(power_buck): replace TPS56528 with MP2307

BREAKING CHANGE: 5V rail current limit drops to 4A; servo supply
decoupling (C10) must be re-evaluated.
```

## Design rules (apply to every commit)

1. Every resistor/capacitor/pin in a schematic keeps its `Datasheet`
   property citing the spec section and datasheet page that justifies it.
2. No invented values, part numbers or pinouts. `docs/specs/` +
   `docs/datasheets/` are the only sources of truth. Missing data = stop and
   ask; never "guess a similar part".
3. Components marked `[Check]` / `[Search]` in the LCSC column are not
   finalized: do not commit them with assigned footprints.
4. ERC must be clean for every touched sheet before committing
   (`kicad-cli sch erc <sheet>.kicad_sch --severity-all`, run per sub-sheet).
   Known, accepted exceptions must be listed in the commit body.
5. DRC must be clean for board changes (`kicad-cli pcb drc arta_board.kicad_pcb`).
6. Architecture changes (block topology, power routing, §0 decisions) require
   an explicit decision recorded in `FULL_BOARD_ARCHITECTURE_AND_CHECKLIST.md`
   in the same commit, or linked from its footer.
7. Commit only the files you changed; do not commit `*-backups/`, `*.bak`,
   `*_cache`, `*.kicad_prl` (already gitignored) or ERC/DRC `*.rpt` reports.

## Branching & PRs

- Work on a branch named `<type>/<scope>-<short-desc>`
  (e.g. `feat/power_input-ltc2955`, `fix/board-stackup`).
- One logical change per commit; separate design changes from docs changes.
- A PR must state: what changed, which spec section / datasheet pages it is
  based on, and the ERC/DRC status.
