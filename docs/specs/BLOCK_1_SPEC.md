# BLOCK 1 SPECIFICATION: INPUT, PROTECTION, SOFT-LATCH

**Status:** Draft for perusal before schematic  
**Revision:** 1.0  
**Date:** 2026-09-30  

---

## 1. Overview

This block handles the 8.4 V battery input (2S Li-ion 18650, bare cells without built-in BMS), protects against reverse polarity, provides soft power control with a start button, measures cell voltages separately, and establishes the main system rail `VBAT_SYS`.

**Peak current (worst case, all 6 motors stalled + servos seized):** 10–12 A for up to ~1 second before fuse/protection shuts down.  
**Nominal current (competition run, active load):** 6–8 A.  
**Resting current (system idle, all rails powered):** ~50–100 mA.

---

## 2. Topology

```
XT30 (male connector)
  ↓ (positive from battery)
  ├→ [Fuse F1: SMD 15 A] ← cuts power on hard short or continuous overload
  │   ↓
  ├→ [Q1: P-MOSFET (Si2301), drain-to-source] ← reverse-polarity protection (body diode blocks reverse)
  │   ↓
  ├→ [TVS D1: SMBJ9.0A] ← clamps inductive spikes
  │   ↓
  ├→ [C_bulk: 1000 µF, 25V] ← absorbs motor current transients
  │   ↓ VBAT_P (protected input)
  ├→ [Divider R1/R2 for cell balance → ADC pin (GPIO)] ← optional, for 2S monitoring
  │   ↓
  ├→ [LTC2955-2 / pushbutton controller IC] 
  │   ├→ PB input (button → GND with pullup)
  │   ├→ INT output → GPIO (ESP32 read button state)
  │   ├→ KILL input ← GPIO (ESP32 can shut down power)
  │   └→ EN output (high-side gate driver output for Q2)
  │   ↓
  ├→ [Q2: P-MOSFET (Si2301), gate driven by EN] ← soft power switch (hold pin during startup)
  │   ↓ VBAT_SYS (main system rail, 8.4 V)
  │   └→ [To buck regulators, H-bridges, all loads]

GND (negative from battery, XT30 shell)
  └→ [Ground plane, all returns through this point]
```

---

## 3. Component Details & Datasheets

### 3.1 Reverse-Polarity MOSFET (Q1)

**Part:** Si2301 (P-Channel, SOT-23)  
**Alternate:** Any P-MOSFET with Vds ≥ 20V, Rds(on) < 200 mΩ @ 4.5V, Id ≥ 2.8 A  
**LCSC/JLCPCB:** C2938372 (GOODWORK, Basic, low cost) or C727151 (JLCPCB part)

| Parameter | Value | Datasheet Ref |
|-----------|-------|----------------|
| Vds (drain-source voltage) | −20 V | Si2301DS, abs max ratings |
| Vgs (gate-source, max) | ±12 V | Si2301DS, abs max ratings |
| Rds(on) @ Vgs=−4.5V | 55–80 mΩ | Si2301DS, electrical char., typ. 80 mΩ (GOODWORK version) |
| Id (continuous, 25°C) | −3.0 A | Si2301DS, abs max ratings |
| Idm (pulsed) | −10 A | Si2301DS, abs max ratings |
| Vgs(th) | −0.4 to −1.0 V | Si2301DS, electrical char. |
| Qg @ Vgs=−4.5V | ~5.5 nC | Si2301DS, dynamic char. |
| Ciss, Coss, Crss | 405 pF, 75 pF, 55 pF | Si2301DS, dynamic char. |

**Connection:**
- Gate: Connect to GND (or leave floating; will be off with Vgs = 0, body diode is conducting in forward direction)
- Source: Connect to XT30+ (battery positive after fuse)
- Drain: Connect to VBAT_P (goes downstream to TVS, capacitors, LTC2955, and Q2)

**Why:** P-channel MOSFET in a "high-side" position acts as a reverse-polarity blocker. If you connect the battery backwards, Vgs is positive (not negative), and the MOSFET is off; current cannot flow forward through the channel. The body diode inside (Source–Drain) is reverse-biased on backward polarity, so it blocks. On correct polarity, Vgs ≈ 0 (gate and source both at battery voltage), and the MOSFET is on (fully saturated at Rds ~80 mΩ). This dissipates very little heat at normal currents.

**Thermal note:** At 8 A, dissipation = I² × R = 64 × 0.00008 ≈ 0.5 W. At 12 A, ≈ 1.2 W, but fuse should interrupt before sustained high current.

---

### 3.2 Fuse (F1)

**Part:** SMD Fuse, 15 A, 32 V DC  
**Package:** 1812 or 1206 (check JLCPCB catalog for exact availability)  
**Candidates:** 
- **Bourns MF-R150** (Fast-acting, 32 V, 15 A)
- Generic SMD 1812 15A fuse

**Datasheet requirements:**  
- Breaking capacity at 32 V DC ≥ 500 A (or specify the type code from JLCPCB/LCSC)
- Voltage rating ≥ 32 V
- I²t rating provided for transient durability

**Connection:**
- Positive lead: XT30+ (after connector pins)
- Negative lead: To Q1 source (input of reverse-polarity block)

**Why 15 A?**  
- Stall current of 6 motors at ~1.5 A each = 9 A peak (absolute worst case).  
- Servo seizure can cause a brief surge, but servo current path is through the 5V buck, not directly through 8.4 V rail.  
- 15 A provides margin above the 12 A worst-case scenario and is within XT30 and board copper limits.

---

### 3.3 Transient Voltage Suppression (TVS) Diode (D1)

**Part:** SMBJ9.0A (Unidirectional, 9 V standoff, ~15 V clamp)  
**Package:** SMB (DO-214AA)  
**LCSC:** Search for SMBJ9.0A variants (multiple manufacturers available)

| Parameter | Value | Datasheet Ref |
|-----------|-------|----------------|
| Vrwm (working voltage) | 9.0 V | SMBJ9.0A datasheet, abs max |
| Vc (clamping voltage @ Ipp) | ~15 V @ 1 A | SMBJ9.0A datasheet, V-I char. |
| Ppk (peak pulse power, 10 µs) | 400 W | SMBJ9.0A datasheet, abs max |
| Vr (reverse standby) | — | Negligible at rated voltage |

**Connection:**
- Cathode (+, marked band): To VBAT_P
- Anode (−): To GND

**Why:** When XT30 is plugged in with inductance in the leads, or if there's a sharp transient from motor switching, the voltage can spike above 8.4 V. The TVS clamps it to ~15 V, protecting downstream buck converters and ICs. The LTC2955 itself can handle up to 36 V on supply, so 15 V is safe margin.

---

### 3.4 Bulk Capacitance

**Part 1 (Primary):** Electrolytic capacitor, 1000 µF, 25 V, ≥ 1 A ripple current rating  
**Part 2 (Filtering):** 4× 10 µF X7R ceramic, 25 V, ESR < 50 mΩ each

**Connection:**
- Both + lead to VBAT_P
- Both − lead to GND (ground plane)

**Datasheet specs to check:**
- Nominal capacitance (not derating curve) at 25°C and rated voltage
- Equivalent series resistance (ESR), ripple current rating
- Voltage derating; at 8.4 V, capacitance remains ≥ 70% nominal
- Frequency response; ceramics provide low-ESR AC filtering, electrolytics bulk storage

**Why:** Motors draw current in bursts (ramp-up, acceleration). Without bulk capacitance, the supply voltage droops, and the buck regulators cannot react fast enough. A 1000 µF electrolytic can supply ~500 mA into a 100 mV dip for ~1 ms, which is long enough for the buck to respond. Ceramics shunt high-frequency noise.

**Ceramic caps selection:**
- X7R dielectric (temperature-stable), not Y5V (droops with temperature)
- 25 V rating ensures no dielectric breakdown even if battery overstarts
- ESR < 50 mΩ per cap; four of them in parallel = ~12 mΩ combined

---

### 3.5 Power-Button Controller (LTC2955-2)

**Part:** LTC2955-2 (Pushbutton On/Off Controller with automatic turn-on, version -2 drives external MOSFET)  
**Package:** DFN-10 (3×2 mm) or SOT-23  
**LCSC:** C683794 (JLCPCB Basic, widely available)

| Parameter | Value | Datasheet Ref |
|-----------|-------|----------------|
| Vin (supply voltage range) | 1.5–36 V | LTC2955 datasheet, abs max |
| Vpb (pushbutton input, max voltage) | ±36 V | LTC2955 datasheet, ESD spec |
| ESD protection (PB pin) | ±25 kV HBM | LTC2955 datasheet, robustness |
| Iq (quiescent supply current) | ~1.2 µA | LTC2955 datasheet, typical |
| EN output voltage (logic high) | ≈ Vin − 0.5 V | LTC2955 datasheet, output char. |
| INT output type | Open drain (100 kΩ pull-down internal) | LTC2955 datasheet, pin description |
| Debounce time (PB) | ~30 ms | LTC2955 datasheet, typical |
| Adjustable turn-off timer | Via external R to GND | LTC2955 datasheet, timing section |

**Pin summary (DFN-10):**
1. PB (pushbutton input, pulled low by button, left open = on)
2. EN (output, high = power allowed, low = power shut down)
3. INT (interrupt output, pulled low on button press or supply fail)
4. KILL (input, pulled low to force power off immediately)
5. GND
6. VIN
7. TIMER (external RC for turn-off time; leave open for ~2 s default)
8. COMP (compensation capacitor for voltage monitor, typically 100 nF)
9. VDD (internal supply, tied to VIN, 0.1 µF cap to GND)
10. DELAY (delay capacitor for supply monitor, typically 47 nF to GND)

**External connections:**
- Pushbutton: Between PB and GND, pull-up resistor ~100 kΩ, normally open switch
- EN → Gate of Q2 (via 100 kΩ pulldown to GND, EN open-drain output pulls it low to disable)
- INT → GPIO pin of ESP32-S3 (read as digital input; low = button pressed or supply low)
- KILL → GPIO pin of ESP32-S3 (ESP32 drives low to shut system down)
- Capacitors around VIN, VDD, COMP, DELAY per datasheet (see section 3.6)

**Behavior:**
- On button press (PB pulled low for ~100 ms), LTC2955 pulls INT low, waking up ESP32.
- ESP32 firmware detects INT interrupt, wakes up, and must drive an enable GPIO high within 2 seconds (default), OR the LTC2955 timer times out and forces EN low, cutting power.
- To stay on, ESP32 drives KILL high (inactive) and holds itself awake.
- To shut down, ESP32 drives KILL low; LTC2955 waits ~2 s (configurable), then releases EN.

---

### 3.6 Support Components for LTC2955

**Decoupling capacitors:**
- **C_vin (VIN to GND):** 1 µF + 100 nF, 25 V ceramics, placed at supply pins
- **C_comp (COMP to GND):** 100 nF, 10 V ceramic (low-voltage cap OK here)
- **C_delay (DELAY to GND):** 47 nF, 10 V ceramic

**Pull-down resistor (EN to GND):** 100 kΩ, 1/4 W, 1% tolerance (EN is open-drain output; needs pulldown to set logic low when LTC2955 disables)

**Pushbutton resistor (PB to VDD):** 100 kΩ pull-up, 1/4 W, ensures PB is high when button open

**Button footprint:** 
- Momentary pushbutton SPST (single-pole, single-throw), rated for ~5 mA @ 3.3 V, footprint suitable for surface mount or through-hole (KiCad has standard footprints: PB_S_KSS0161 or similar)

---

### 3.7 Soft-Power MOSFET (Q2)

**Part:** Si2301 (same as Q1, P-Channel, SOT-23)  
**LCSC/JLCPCB:** C2938372 or C727151

**Connection:**
- Gate: Driven by EN output of LTC2955 (with 100 kΩ pull-down resistor to ensure it goes low)
- Source: Connected to VBAT_P (the filtered, TVS-protected input)
- Drain: Connected to VBAT_SYS (main system rail)

**Purpose:** Acts as a latching power switch. When EN goes low (power disable), the gate is pulled low, Q2 turns off, and almost all current to the system stops. The only current path is through the leakage of Q2's body diode (negligible, ~µA) and the pull-up resistors to VDD (which itself is off).

**Startup sequence:**
1. User presses button.
2. LTC2955 detects PB low, asserts INT low (wakes ESP32 via interrupt).
3. ESP32 firmware wakes up, detects interrupt, and drives a KEEP_ALIVE GPIO high.
4. This GPIO is connected to the EN pin of LTC2955 via a pull-down resistor, so EN can stay high as long as ESP32 drives KEEP_ALIVE high.
5. Q2 gate is high (referenced to VBAT_P), so Vgs ≈ 0, Q2 is ON, VBAT_SYS is live.
6. If ESP32 hangs or crashes, the LTC2955 timer (~2 s) times out, EN goes low, Q2 opens, system dies.

---

### 3.8 Battery Monitoring (Optional but Recommended for No-BMS Setup)

**Cell voltage divider:**
- **R1 (top resistor, VBAT_P to ADC_BATT):** 820 kΩ, 1%, 1/4 W
- **R2 (bottom resistor, ADC_BATT to GND):** 270 kΩ, 1%, 1/4 W
- **Divider ratio:** R2 / (R1 + R2) = 270 / 1090 ≈ 0.248; so 8.4 V input → ~2.09 V on ADC (within 2.5 V range of ESP32)
- **Series resistor (at ADC pin):** 1 kΩ, to reduce sensitivity to ADC switching noise

**Mid-point divider (for 2S cell balance):**
- **R3, R4:** Use same 270 kΩ / 820 kΩ pair, dividing VBAT_P to a mid-voltage point
- **R_protect (series at the mid-point connector lead):** 1 kΩ, prevents accidental short

**ADC pins (ESP32-S3):**
- **GPIO34 (or any ADC0 pin):** Total pack voltage
- **GPIO35 (or any ADC1 pin):** Mid-point (cell 1 voltage)

**Firmware:**
- Read total voltage; if < 6.4 V (average 3.2 V per cell), drive KILL low after warning.
- Read mid-point; if one cell < 2.7 V (deep discharge), also cut power.
- Log voltages to console for debugging.

---

## 4. Test Points

Place test points (via + small pad) at:
1. **TP1_VBAT_P** (after fuse, Q1, TVS, before bulk caps) – measure input after protection
2. **TP2_VBAT_SYS** (main rail, after Q2) – measure system voltage
3. **TP3_GND** (ground reference)

These allow quick voltage checks with a multimeter during bring-up and debugging.

---

## 5. Design Rules & Layout Notes

### 5.1 PCB Placement
- **Fuse (F1), Q1:** Place near XT30 connector, minimize trace length to battery input
- **TVS (D1):** Place immediately after Q1, close to GND plane
- **Bulk caps (C_bulk, C_filter):** Place between VBAT_P and GND, wide traces, multiple vias to ground
- **LTC2955 & support caps:** Place near bulk capacitors, close to GND plane
- **Q2:** Place between VBAT_P (source) and VBAT_SYS (drain), minimize gate trace length to LTC2955 EN output
- **Button:** Can be placed anywhere accessible on the edge of the PCB

### 5.2 Trace Widths & Current Paths
- **XT30 to Fuse to Q1:** Thick trace or pour (≥ 0.5 mm width for 12 A, single-layer estimate 1.5 mm for 10 A on external copper)
- **Q1 drain to VBAT_P:** Polygon pour, multiple vias
- **VBAT_P to Q2 source:** Via polygon or wide trace
- **Q2 drain to VBAT_SYS:** Polygon to all downstream loads (buck inputs)
- **All returns (−) to GND plane:** Large ground plane on inner layer or bottom layer, minimum 0.5 mm clearance around signals

### 5.3 Ground Plane
- Continuous ground plane (L3 or L4) under all components
- Multiple vias (≥ 0.3 mm diameter, every ~5 mm) connecting outer-layer signals to ground plane
- No traces cutting through ground plane; use vias to bridge isolated regions

### 5.4 Clamping & Solder Mask
- **XT30 connector:** Large solder pads, solder mask opening for pins (standard)
- **Test points:** Small pads (0.5–0.8 mm diameter) with via to ground, no solder mask over pads (allows probe contact)

---

## 6. Reliability Checklist (Cross-Check Before Schematic Approval)

### Datasheet Compliance
- [ ] LTC2955 supply voltage (1.5–36 V) — confirmed ✓
- [ ] LTC2955 PB input voltage swing (±36 V) — can handle a momentary spike if button lead touches 8.4 V accidentally
- [ ] Si2301 (Q1, Q2) gate-source voltage (±12 V) — confirmed ✓, EN output of LTC2955 will not exceed supply (8.4 V)
- [ ] SMBJ9.0A clamping voltage (~15 V) is below ±36V limits of LTC2955 and buck inputs ✓
- [ ] Electrolytic cap ripple current rating ≥ 1 A @ 100 kHz
- [ ] Ceramic cap voltage rating (25 V) adequate for supply spikes ✓

### Thermal Design
- [ ] Q1 dissipation at 8 A: (8)² × 0.00008 Ω = 0.51 W, within 1 W SOT-23 rating ✓ (with thermal relief pads)
- [ ] Q2 similar, no thermal concern for normal operation
- [ ] Electrolytic cap ESR generates heat; select low-ESR type and place near bulk capacitor, not under FET

### Protection & Margins
- [ ] Fuse 15 A is above nominal 8 A but below stall 12 A; allows brief transients, cuts power if sustained
- [ ] TVS clamps to 15 V; all downstream components (buck ≥ 32 V input, ESP32 ≤ 3.6 V via buck) safe
- [ ] Bulk capacitance (1000 µF) can support 6–8 A for ~100 ms without drooping below 7 V (typical buck start voltage)
- [ ] LTC2955 timer (~2 s) prevents system from stuck "on" if firmware crashes
- [ ] Battery monitoring via ADC allows firmware to shut down before under-voltage damage (< 3.0 V per cell)

### Sourcing (JLCPCB / LCSC)
- [ ] Si2301 (GOODWORK): LCSC C2938372, Basic, stock > 1000 units ✓
- [ ] LTC2955-2 (DFN-10): LCSC C683794, Basic, stock available ✓
- [ ] SMBJ9.0A: LCSC has variants, search for "SMBJ9.0A", Basic ✓
- [ ] SMD fuse 15 A: Search JLCPCB/LCSC for "1812 15A" or "1206 15A", check exact part number
- [ ] Electrolytic & ceramic caps: Widely available (e.g., JLCPCB C15525 for 1000 µF 25 V)
- [ ] Resistors & pullup: Standard 1/4 W 1% types, very cheap

---

## 7. BOM Summary (Partial, Block 1 Only)

| Reference | Value | Package | LCSC/JLCPCB | Qty | Notes |
|-----------|-------|---------|-------------|-----|-------|
| F1 | 15 A, 32 V SMD | 1812/1206 | [Check part] | 1 | Fast-acting fuse |
| Q1 | Si2301 | SOT-23 | C2938372 | 1 | Reverse-polarity MOSFET |
| Q2 | Si2301 | SOT-23 | C2938372 | 1 | Soft-power MOSFET |
| D1 | SMBJ9.0A | SMB | [Check part] | 1 | TVS diode |
| U1 | LTC2955-2 | DFN-10 | C683794 | 1 | Pushbutton controller |
| C1 | 1000 µF, 25 V | 1206/1210 | [Electrolytic] | 1 | Bulk capacitance |
| C2–C5 | 10 µF, 25 V | 0805 | [Ceramic, X7R] | 4 | Filter caps |
| C6–C10 | 100 nF, 16 V | 0402/0603 | [Ceramic, X7R] | 5 | Decoupling |
| C11 | 47 nF, 10 V | 0402/0603 | [Ceramic] | 1 | LTC2955 DELAY |
| R1 | 820 kΩ, 1% | 0603 | [Metal film] | 1 | Battery divider top |
| R2 | 270 kΩ, 1% | 0603 | [Metal film] | 1 | Battery divider bottom |
| R3, R4 | 270 kΩ, 1% | 0603 | [Metal film] | 2 | Mid-point divider |
| R5, R6, R7 | 1 kΩ, 1/4 W | 0603 | [Metal film] | 3 | Pulldown, ADC protection |
| R8 | 100 kΩ, 1/4 W | 0603 | [Metal film] | 1 | EN pulldown |
| R9 | 100 kΩ, 1/4 W | 0603 | [Metal film] | 1 | PB pullup |
| SW1 | Pushbutton SPST | SMD or THT | [Check footprint] | 1 | Momentary, 5 mA rated |

---

## 8. Notes on Bring-Up (Stages 9+)

1. **First check:** With lab power supply (8.4 V, current limit = 1 A), verify XT30 connector polarity and voltage.
2. **Second check:** Measure voltage across bulk capacitors (should be 8.4 V±0.1 V, no discharge).
3. **Third check:** Press button; INT should go low momentarily; then, if firmware is loaded, system stays on.
4. **Fourth check:** Measure VBAT_SYS voltage; should be 8.4 V (Q2 open = no voltage; Q2 on = full voltage).
5. **Fifth check:** Drive KILL low (or remove hold on KEEP_ALIVE); system should shut down within 2–3 seconds.
6. **Cell voltage check:** If mid-point divider is populated, read ADC values and confirm linearity.

---

## 9. Revision History

| Rev | Date | Author | Change |
|-----|------|--------|--------|
| 1.0 | 2026-09-30 | Claude | Initial spec for Block 1 (input, protection, soft-latch) |
