# COMPONENT LIST — BLOCK 1 (INPUT, PROTECTION, SOFT-LATCH)

**Status:** Preliminary, requires stock verification on JLCPCB/LCSC  
**Date:** 2026-09-30  
**Total Estimated Cost (Block 1 only):** ~$5–8 USD  

---

## Active Components

### Microcontrollers / ICs

| Reference | Part Number | Manufacturer | LCSC/JLCPCB | Package | Stock Status | Price (qty 1) | Notes |
|-----------|-------------|--------------|-------------|---------|--------------|---------------|-------|
| U1 | LTC2955-2 | Analog Devices | C683794 | DFN-10 | Check | ~$2.50 | Pushbutton controller, open-drain EN output, INT, KILL |

### MOSFETs

| Reference | Part Number | Manufacturer | LCSC/JLCPCB | Package | Stock Status | Price (qty 1) | Notes |
|-----------|-------------|--------------|-------------|---------|--------------|---------------|-------|
| Q1 | Si2301 | GOODWORK | C2938372 (LCSC) | SOT-23 | ✓ Basic | ~$0.02 | Reverse-polarity protection, Rds 80 mΩ @ 4.5V |
| Q2 | Si2301 | GOODWORK | C2938372 (LCSC) | SOT-23 | ✓ Basic | ~$0.02 | Soft-power switch, same part as Q1 |
| **Alt (Q1/Q2)** | Si2301 | TWGMC | C727151 (JLCPCB) | SOT-23 | ✓ Extended | ~$0.03 | JLCPCB variant, slightly higher Rds |

### Diodes

| Reference | Part Number | Manufacturer | LCSC/JLCPCB | Package | Stock Status | Price (qty 1) | Notes |
|-----------|-------------|--------------|-------------|---------|--------------|---------------|-------|
| D1 | SMBJ9.0A | — | [Search LCSC] | SMB | ✓ Basic | ~$0.10 | TVS, 9V standoff, ~15V clamp |
| **Alt** | SMBJ7.0A | — | [Search LCSC] | SMB | ✓ Basic | ~$0.10 | 7V standoff (lower threshold) |

### Fuses

| Reference | Part Number | Manufacturer | LCSC/JLCPCB | Package | Stock Status | Price (qty 1) | Notes |
|-----------|-------------|--------------|-------------|---------|--------------|---------------|-------|
| F1 | [Search: SMD fuse 15A 32V] | Various | [Check LCSC] | 1812 or 1206 | ? | ~$0.15–0.30 | **Action:** Search JLCPCB/LCSC for "15A 32V" SMD fuse, verify breaking capacity |
| **Alt (safer)** | ATO-15 + Holder | Automotive | [THT breakaway] | Through-hole | ✓ | ~$0.50–1.00 | Traditional automotive fuse, easy to replace, hand-soldered |

---

## Passive Components

### Electrolytic Capacitors

| Reference | Capacitance | Voltage | ESR | Package | LCSC/JLCPCB | Stock | Price | Notes |
|-----------|-------------|---------|-----|---------|-------------|-------|-------|-------|
| C1 | 1000 µF | 25 V | <100 mΩ | 1210 | [Search] | ✓ | ~$0.30–0.50 | Bulk storage, motor transients |

### Ceramic Capacitors (X7R, 25 V)

| Reference | Capacitance | Voltage | Package | LCSC/JLCPCB | Stock | Price | Qty | Notes |
|-----------|-------------|---------|---------|-------------|-------|-------|-----|-------|
| C2–C5 | 10 µF | 25 V | 0805 | [Search] | ✓ | ~$0.05 each | 4 | Filter, low-ESR |
| C6–C10 | 100 nF | 16 V | 0402 | [Search] | ✓ | ~$0.01 each | 5 | Decoupling (VDD, COMP, DELAY) |
| C11 | 47 nF | 10 V | 0402 | [Search] | ✓ | ~$0.01 | 1 | LTC2955 DELAY cap |

### Resistors (1%, 1/4 W, Metal Film)

| Reference | Resistance | Tolerance | Package | LCSC/JLCPCB | Stock | Price | Qty | Notes |
|-----------|------------|-----------|---------|-------------|-------|-------|-----|-------|
| R1 | 820 kΩ | 1% | 0603 | [Standard] | ✓ | ~$0.01 | 1 | Battery voltage divider (top) |
| R2, R3, R4 | 270 kΩ | 1% | 0603 | [Standard] | ✓ | ~$0.01 | 3 | Dividers (bottom and mid-point) |
| R5, R6, R7 | 1 kΩ | 1% | 0603 | [Standard] | ✓ | ~$0.01 | 3 | Series protection (ADC, mid-point), pulldown |
| R8 | 100 kΩ | 1% | 0603 | [Standard] | ✓ | ~$0.01 | 1 | EN pulldown |
| R9 | 100 kΩ | 1% | 0603 | [Standard] | ✓ | ~$0.01 | 1 | PB pullup |

---

## Connectors

| Reference | Type | Connector | Package | LCSC/JLCPCB | Stock | Price | Notes |
|-----------|------|-----------|---------|-------------|-------|-------|-------|
| J1 | Power In | XT30 (male, 2-pin) | Through-hole | [Search] | ✓ | ~$0.50–1.00 | Battery connector, rated 30 A |
| J2 | Test | JST-XH 3-pin (optional) | Through-hole or SMD | [Search] | ✓ | ~$0.20 | Mid-point voltage tap for 2S monitoring |
| SW1 | Button | Pushbutton SPST | SMD or THT | [Search] | ✓ | ~$0.10–0.30 | Momentary, 5 mA at 3.3 V |

---

## Recommended Sourcing Strategy

### Immediate Actions
1. **Search LCSC / JLCPCB for:**
   - "SMD fuse 15A 32V" in 1812/1206 packages → pick one with available stock and confirm breaking capacity
   - "SMBJ9.0A" → multiple manufacturers, any Basic variant is fine
   - "LTC2955-2 DFN-10" → should be C683794 at LCSC

2. **Fill in missing LCSC codes** for capacitors, resistors, and button (standard parts, easily found)

3. **Verify stock levels** on JLCPCB for the 5-piece order (especially LTC2955, Si2301)

### Part Selection Priority
- **Critical (must find):** LTC2955-2, Si2301, SMD 15 A fuse, SMBJ9.0A
- **High importance:** 1000 µF 25 V electrolytic cap (low ESR), 10 µF X7R ceramics
- **Standard (any brand):** 100 nF, 1%, resistors, pushbutton

### Cost Optimization
- Prefer JLCPCB Basic parts (faster assembly, cheaper)
- Prefer Extended only if no Basic alternative (e.g., some cap values)
- For a 5-piece order, component cost should remain < $30 total (PCB ~$20–30 depending on layer count and size)

---

## Alternative Parts (If Unavailable)

### Button Controller
- **LTC2954-2** (older, similar, check pinout): DFN-8 or SOT-23, less common but works same way
- **Dumb protection (no auto turn-on):** Use discrete P-MOSFET + RC timing circuit, but more complex and less reliable

### P-MOSFET (Q1, Q2)
- **FDV301P** (another small P-channel): SOT-23, Rds ~150 mΩ @ 4.5 V, rated 20 V, 2.5 A
- **SIR322DP** (dual MOSFET): SOT-23-6, two P-channels in one package, useful if space is tight
- **Larger options (if thermal concern):** FDS6612, BSS83, but overkill for this current and probably more expensive

### TVS Diode
- **SMBJ7.0A** (7 V standoff): Slightly higher clamping voltage (~13 V), still safe
- **SMBJ5.0A** (5 V standoff): Lower threshold, useful if buck inputs are lower tolerance
- **Unidirectional vs. bidirectional:** Use unidirectional (cathode to VBAT, anode to GND) for 8.4 V input rail

### Fuse
- **Littelfuse 15A automotive ATO fuse + THT holder:** More reliable, replaceable, if space allows
- **Polyfuse (PTC thermistor):** Self-resetting, but slower response and less reliable for hard shorts; **not recommended** for motor power

---

## Stock Check Checklist

Before ordering, fill in the LCSC/JLCPCB stock status:

- [ ] LTC2955-2 DFN-10: C683794, stock = ___ units, price per unit = $___
- [ ] Si2301 SOT-23: C2938372 (LCSC) or C727151 (JLCPCB), stock = ___, price = $___
- [ ] SMBJ9.0A: LCSC code = ___, stock = ___, price = $___
- [ ] SMD 15 A fuse: LCSC/JLCPCB code = ___, stock = ___, price = $___
- [ ] Electrolytic 1000 µF 25 V: LCSC code = ___, stock = ___, price = $___
- [ ] Ceramic 10 µF 25 V: LCSC code = ___, stock = ___, price = $___
- [ ] 100 nF ceramic: LCSC code = ___, stock = ___, price = $___
- [ ] Resistors (1%, 1/4 W): Any code works, total cost for all = $___
- [ ] Capacitors (misc): Total cost = $___
- [ ] Connectors (XT30, button, JST-XH): Total cost = $___

**Total Block 1 component cost (qty 1):** $___  
**Total Block 1 for 5-piece order (assembly):** $___  

---

## Notes
- Prices are estimated from typical LCSC/JLCPCB ranges; actual prices fluctuate.
- "Basic" parts can be assembled by JLCPCB at no extra charge; "Extended" parts incur a $3–5 surcharge per type.
- For a cost-sensitive build, minimize Extended parts and consolidate to fewer unique values.
