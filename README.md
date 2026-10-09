# Arta Board

Main control PCB for an ESP32-S3 robot: 2S Li-ion (8.4 V) input with protection and
soft-latch power control, 5 V / 3.3 V buck rails, 6x LEGO LPF2 motor H-bridges
(TB67H450), 4x servos, BMI270 IMU, and an I2C expansion bus (TCA9548A).
4-layer board, fabricated at JLCPCB.

## Repository layout

```
docs/
  specs/            Approved architecture, BOM and checklist documents (source of truth for decisions)
  datasheets/       PDF datasheets, added per component as they are used
hardware/
  kicad/            KiCad 10 project (arta_board), hierarchical sheets per block
```

ESP32 firmware lives in a separate repository (not in this one).

## Open decisions (from FULL_BOARD_ARCHITECTURE_AND_CHECKLIST.md, Section 0)

> This is the only list of unresolved questions. Everything else in the
> architecture document is a fixed decision. Table copied verbatim.

| № | Вопрос | Моя рекомендация по умолчанию | Почему |
|---|--------|-------------------------------|--------|
| 1 | Чип buck 5V | **TPS56528** (синхронный, до 6А) | MP2307 не тянет 4.5А пик, TPS5450 асинхронный и греется |
| 2 | Чип buck 3.3V | **AP63203** (вход до 32В, 2А) | Текущий LDO-вариант из ТЗ грелся бы на 0.85-1.36 Вт и был завязан на просадки серв на линии 5В — убрали эту связь |
| 3 | ID1/ID2 на LPF2 | **Трактовать как простой UART/GPIO**, НЕ полный протокол умных LEGO-моторов | У вас "дамми-логика" ~10 мА на порт — это не поведение умного мотора (который требует хэндшейк 2400→115200 бод). Если ошибаюсь — нужен SC16IS752 (I2C/SPI→UART расширитель на 2-8 каналов) |
| 4 | H-мост для LPF2 моторов | **TB67H450** (как в исходном плане) | До 1.5А на канал, логика 3.3В, подходит под ваши цифры по стопору |
| 5 | PCA9685 | **Не ставить**, ШИМ серв напрямую с ESP32-S3 (LEDC) | У вас всего 4 серво — хватает каналов LEDC, не нужен I2C-посредник с задержкой |

**Blocking effect:** `power_buck.kicad_sch` must not be started until items 1–2 are
confirmed; `motors.kicad_sch` must not be started until item 3 is confirmed.

## Schematic sheets and reviewer responsibility

KiCad project: `hardware/kicad/arta_board.kicad_pro` (KiCad 10).

| Sheet | Block | Reviewer |
|---|---|---|
| `power_input.kicad_sch` | Block 1: input, protection, soft-latch | **TBD — to be assigned** |
| `power_buck.kicad_sch` | Block 2: buck 5V / 3.3V | **TBD — to be assigned** |
| `mcu.kicad_sch` | ESP32-S3 | **TBD — to be assigned** |
| `imu.kicad_sch` | BMI270 + power isolation | **TBD — to be assigned** |
| `motors.kicad_sch` | 6x TB67H450 + 6x LPF2 | **TBD — to be assigned** |
| `servos.kicad_sch` | 4x servos (direct LEDC) | **TBD — to be assigned** |
| `i2c_bus.kicad_sch` | TCA9548A + 6x I2C ports + SPI | **TBD — to be assigned** |
| `interface.kicad_sch` | Buzzer + RGB LED | **TBD — to be assigned** |

## Getting started (new team member)

1. Install KiCad 10 (files were generated with 10.0.5).
2. Open `hardware/kicad/arta_board.kicad_pro`.
3. Read, in this order:
   - `docs/specs/FULL_BOARD_ARCHITECTURE_AND_CHECKLIST.md` — architecture + final pre-power-on checklist.
   - `docs/specs/BLOCK_1_SPEC.md` — current block under work.
   - `docs/specs/COMPONENT_LIST_BLOCK_1.md` — BOM with LCSC sourcing status.
4. Working rules (non-negotiable):
   - Every resistor/capacitor/pin in a schematic carries a `Datasheet` property
     citing the spec section and the datasheet page that justifies it.
   - No value, part number or pinout is invented: `docs/specs/` + `docs/datasheets/`
     are the only sources of truth. Missing data = stop and ask.
   - Run ERC after every sheet change:
     `kicad-cli sch erc <sheet>.kicad_sch --severity-all`
     (run it on each sub-sheet file; the CLI does not traverse hier sheets from the root).
   - Run DRC after board changes: `kicad-cli pcb drc arta_board.kicad_pcb`.
     Any ERC/DRC error = stop and report, never continue silently.
   - Commits: [CONTRIBUTING.md](CONTRIBUTING.md) — conventional commits
     (`type(scope): subject`) + design rules.
   - Components with `[Check]` / `[Search]` in the LCSC column of
     `COMPONENT_LIST_BLOCK_1.md` are not finalized: their footprints stay
     unassigned until a part is confirmed.
5. Board stackup: JLCPCB standard 4-layer 1.6 mm (1 oz / 0.5 oz / 0.5 oz / 1 oz,
   7628*1 prepreg, FR-4), set in `arta_board.kicad_pcb` (see `.kicad_pcb` `stackup`
   section). No controlled impedance requirements.

## Status

- [x] Stage 0: repo structure, KiCad project skeleton, 4-layer board, README
- [~] Stage 0.5: `power_input.kicad_sch` — symbols placed, **not wired** (waiting for
      review; U1 LTC2955-2 custom symbol blocked pending datasheet)
- [ ] Netting of Block 1
- [ ] Blocks 2+ (each gated by the open decisions above)
