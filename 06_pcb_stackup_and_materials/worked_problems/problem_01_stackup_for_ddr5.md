# Problem 01: Stackup for DDR5

## Problem Statement

You are the lead PCB designer for a server memory subsystem board. The board must support:

- **Interface:** DDR5-4800 (4800 MT/s per pin, 2400 MHz clock, 64-bit wide data bus + 8 ECC bits = 72 signals, plus address/command/control)
- **Components:** One CPU/SoC package (BGA, 0.65 mm ball pitch) and two DDR5 RDIMMs per channel (connector footprints)
- **Topology:** Fly-by daisy-chain for address/command/clock; point-to-point for data bytes
- **Maximum trace length:** 100 mm for data, 150 mm for address/command
- **Operating environment:** Industrial temperature (−40°C to +85°C), requiring $T_g > 170°$C laminate

**Design requirements:**

1. Specify a complete 8-layer stackup with dielectric materials, layer assignments, and approximate layer thicknesses.
2. State the impedance targets for DDR5 single-ended and differential signals.
3. Calculate the required trace widths for data lines on the chosen inner signal layer.
4. Identify the key EMI and SI risks in this design and state how the stackup mitigates them.
5. Specify whether backdrilling is needed for BGA escape vias.

---

## Worked Solution

### Step 1 — DDR5 Signal Requirements

**DDR5-4800 key parameters (JEDEC JESD79-5B):**

| Parameter | Value |
|---|---|
| Data rate | 4800 MT/s per pin (2400 MHz clock, DDR) |
| Data signal Nyquist | 2400 MHz = 2.4 GHz |
| Address/command Nyquist | ~1.2 GHz (command encoded at half data rate) |
| Single-ended impedance (JEDEC spec) | 40 Ω (data), 40 Ω (address) |
| Differential clock impedance | 100 Ω |
| Signal voltage (VDD) | 1.1 V |

At 2.4 GHz Nyquist, standard FR4 is adequate from a loss perspective. However, the industrial temperature requirement mandates a laminate with $T_g > 170°$C. High-$T_g$ FR4 (e.g., Isola IS410, Shengyi S1000-2) meets this while maintaining approximately the same $D_k/D_f$ as standard FR4.

**Material selection:** Isola IS410 (high-$T_g$ FR4 class):
- $D_k$ at 1 GHz: 4.20
- $D_f$ at 1 GHz: 0.016
- $T_g$: 175°C
- CTE (z-axis): 55 ppm/°C

---

### Step 2 — 8-Layer Stackup Definition

**Proposed stackup:**

```
Layer 1   — Signal (Top, S1)       — BGA breakout, short stubs, SMT pads
             [Prepreg: 2116 style, ~110 µm, IS410]
Layer 2   — Ground (GND1)          — L1 reference; primary EMI shield
             [Core A: 200 µm, IS410]
Layer 3   — Signal (Inner 1, S2)   — DDR5 data bytes (DQ, DQS, DM/DBI)
             [Prepreg: 2116 style, ~110 µm, IS410]
Layer 4   — Ground (GND2)          — L3 lower ref; L5 upper ref
             [Core B: 200 µm, IS410]
Layer 5   — Signal (Inner 2, S3)   — DDR5 address, command, control (CA bus)
             [Prepreg: 2116 style, ~110 µm, IS410]
Layer 6   — Power (PWR, VDD)       — DDR5 VDD = 1.1 V distribution; L5 lower ref
             [Core C: 200 µm, IS410]
Layer 7   — Signal (Inner 3, S4)   — Auxiliary: power sequencing, management bus
             [Prepreg: 2116 style, ~110 µm, IS410]
Layer 8   — Ground (GND3)          — L7 reference; bottom EMI shield
```

**Total board thickness:** approximately 1.4 mm (suitable for server board form factor, compatible with ATX/SSI-EEB keepout rules).

**Symmetry check:**

| Position from mid-plane | Layer | Copper type | Dielectric |
|---|---|---|---|
| +3 | L1 (Signal) | ~30% coverage | Prepreg 110 µm |
| +2 | L2 (GND) | ~70% coverage | Core 200 µm |
| +1 | L3 (Signal) | ~30% coverage | Prepreg 110 µm |
| 0 | L4 (GND) | ~70% coverage | Core 200 µm |
| -1 | L5 (Signal) | ~30% coverage | Prepreg 110 µm |
| -2 | L6 (PWR) | ~60% coverage | Core 200 µm |
| -3 | L7 (Signal) | ~25% coverage | Prepreg 110 µm |
| — | L8 (GND) | ~70% coverage | — |

Note: L6 (PWR) copper coverage differs from L2 (GND). To maintain symmetry, copper balancing fills should be added to L6 and L7, or the L2/L3/L6/L7 dielectric thicknesses should be adjusted. A fully balanced design would use two ground planes at L2 and L7 and split the power to L3 and L6 — but this reduces signal layer count. The pragmatic solution is copper fills on L6/L7.

---

### Step 3 — Impedance Targets and Trace Width Calculation

**DDR5 impedance requirements:**

| Signal type | Standard | Target $Z_0$ |
|---|---|---|
| DQ (data) single-ended | JEDEC JESD79-5B | 40 Ω ± 10% (36–44 Ω) |
| DQS (strobe) differential | JEDEC JESD79-5B | 80 Ω differential (40 Ω each) |
| CA (command/address) single-ended | JEDEC JESD79-5B | 40 Ω ± 10% |
| CK (differential clock) | JEDEC JESD79-5B | 85 Ω differential |

**Trace width calculation — L3 stripline (DQ data signals):**

Geometry: L3 is a stripline between GND1 (L2) and GND2 (L4). From the stackup, Core A (200 µm) lies between L2 and L3 and the prepreg (110 µm) between L3 and L4, so $b_{L2-L4} = 200 + 110 = 310$ µm. The trace is offset rather than centred, so the centred-stripline formula below is an approximation.

Using $b = 310$ µm, $T = 18$ µm (½ oz copper), $\epsilon_r = 4.20$:

$$40 = \frac{60}{\sqrt{4.20}} \ln\left(\frac{4 \times 310}{0.67\pi(0.8W + 18)}\right) = 29.28 \ln\left(\frac{1240}{2.104(0.8W + 18)}\right)$$

$$\ln\left(\frac{1240}{2.104(0.8W + 18)}\right) = \frac{40}{29.28} = 1.366$$

$$\frac{1240}{2.104(0.8W + 18)} = e^{1.366} = 3.919$$

$$2.104(0.8W + 18) = \frac{1240}{3.919} = 316.4 \implies 0.8W + 18 = 150.4 \implies W = \frac{150.4 - 18}{0.8} = 165.5 \text{ µm}$$

**DQ line trace width on L3: approximately 166 µm (6.5 mil).**

**Verify with sensitivity check:**

Manufacturing width variation: ±20 µm (typical for ½ oz copper).

Width sensitivity at $W = 166$ µm (differentiating the formula, $dZ_0/dW = -29.28 \times 0.8/(0.8W + T)$):

$$\frac{dZ_0}{dW} \approx -\frac{29.28 \times 0.8}{150.4} = -0.156 \text{ Ω/µm}$$

$$\Delta Z_0 = \pm 20 \times 0.156 = \pm 3.1 \text{ Ω} \quad (\pm 7.8\%)$$

The ±7.8% width-only variation is inside the ±10% tolerance specification, but leaves little room for dielectric-thickness and Dk variation. Options to tighten it:

- Increase copper fill on L3 to use ½ oz copper with controlled-depth etch (reduces width variation to ±12 µm).
- Specify tight etch tolerance on L3 in the fabrication notes.

Field-solver verification (Polar Si9000 or Mentor Hyperlynx) should be used for the final width before Gerber release.

---

### Step 4 — EMI and SI Risk Assessment

**Risk 1: DDR5 data bus radiation (L1 microstrip)**

If data signals are routed on L1 (microstrip), the high-speed edges (DDR5-4800 rise time ~100 ps) radiate efficiently from the unshielded microstrip geometry. At 2.4 GHz, microstrip radiation from a 100 mm trace is substantial.

**Mitigation:** Route all DQ and DQS signals on L3 (stripline). Only use L1 for short BGA breakout traces (< 5 mm) before transitioning to L3 via vias.

**Risk 2: Return path interruption under address bus (L5)**

The L5 address bus references L4 (GND) and L6 (PWR). If the L6 power plane is split for multiple voltage rails (VDD, VDDQ, VDDIO), address traces crossing the split will have their return current diverted. This creates impedance discontinuities and potential EMI.

**Mitigation:** Define L6 power plane splits such that no address trace crosses a split. Route the CA bus orthogonally to the split boundaries where possible. Place stitching capacitors (100 nF, 0402) at any unavoidable crossings.

**Risk 3: DQS differential skew from glass weave**

DDR5 DQS (differential strobe) is a critical timing reference. Glass weave skew on L3 prepreg could cause intra-pair skew that violates the 20 ps max skew budget.

**Mitigation:** Route DQS pairs at 45° to the glass weave direction (IS410 uses 2116 weave; route DQS at 45° as per IPC-2141A guidance). Alternatively, specify spread-weave prepreg for L2-L3 prepreg layer only.

**Risk 4: Simultaneous switching noise (SSN) on VDD**

DDR5 operates at 1.1 V with tight voltage tolerance (±3%). Simultaneous switching of 72 data bits generates burst current demands from the CPU VDD pin. The L6 power plane provides PDN impedance; its plane-pair capacitance with L4 (GND) must be evaluated.

Plane capacitance (L4-L6, separated by L5 prepreg + L5 copper + core contribution ~310 µm effective):

$$C_{plane} \approx \frac{4.20 \times 8.85 \times 10^{-12} \times A_{overlap}}{310 \times 10^{-6}}$$

For a 60 × 80 mm VDD plane: $C_{plane} \approx 0.58$ nF. This provides high-frequency PDN support from ~100 MHz up to the plane resonance frequency.

---

### Step 5 — BGA Via Backdrilling Assessment

**BGA geometry:**

- CPU BGA ball pitch: 0.65 mm
- Via drill: 0.20 mm (8 mil)
- Board thickness: 1.4 mm

**Signal transition:** DQ signals enter via L1 BGA pads, route on L3.

**Stub length:**

Via useful depth to L3, working through the stackup (L1 to L2 is 110 µm prepreg, L2 to L3 is the 200 µm Core A):

- L1 to L2: ~110 µm (prepreg thickness)
- L2 to L3: ~200 µm (Core A)
- Via signal exit at L3: total depth from surface = ~310 µm (plus copper)

Stub = remaining via below L3 = $1400 - 310 = 1090$ µm.

**Stub resonance frequency:**

$$f_{res} = \frac{c/\sqrt{\epsilon_r}}{4 \times l_{stub}} = \frac{300/\sqrt{4.2}}{4 \times 1.09 \text{ mm}} = \frac{146.3 \text{ mm/ns}}{4.36 \text{ mm}} = 33.6 \text{ GHz}$$

**Is backdrilling needed?**

DDR5-4800 Nyquist: 2.4 GHz. The stub resonance at 33.6 GHz is $33.6/2.4 = 14 \times f_{Nyquist}$. The stub is far above the signal band.

**Below-resonance stub loading check:**

$$\alpha_{stub} \approx 20 \log_{10}\left|1 + j\frac{\pi \times f_{Nyq} \times l_{stub} \times Z_0}{v_p \times Z_{via}}\right|$$

$$= 20 \log_{10}\left|1 + j\frac{\pi \times 2.4 \times 1.09 \times 50}{146.3 \times 45}\right| = 20 \log_{10}\left|1 + j\frac{410.9}{6584}\right|$$

$$= 20 \log_{10}(1 + 0.0624^2)^{1/2} \approx 20 \log_{10}(1.00195) \approx 0.017 \text{ dB}$$

**Stub contribution at 2.4 GHz: 0.017 dB — completely negligible.**

**Verdict: Backdrilling is NOT required** for DDR5-4800 BGA vias on this 1.4 mm, 8-layer board. The stub resonance is 14× the Nyquist frequency and the sub-resonance loading loss is less than 0.02 dB.

This would change for future DDR6 or higher data rate interfaces. At DDR5-6400 (Nyquist = 3.2 GHz), the stub still poses no problem. At DDR5-8800 (Nyquist = 4.4 GHz), recheck stub loading — still likely acceptable on a 1.4 mm board.

---

### Step 6 — Design Summary

| Parameter | Value |
|---|---|
| Layer count | 8 |
| Total board thickness | ~1.4 mm |
| Dielectric material | Isola IS410, $T_g = 175°$C |
| DQ trace layer | L3 (stripline) |
| CA trace layer | L5 (stripline) |
| DQ target impedance | 40 Ω ± 10% |
| DQ trace width on L3 | ~166 µm (6.5 mil) |
| DQS differential impedance | 80 Ω differential |
| CK differential impedance | 85 Ω differential |
| Backdrilling required | No (stub resonance >> Nyquist) |
| Key EMI mitigation | DQ/CA on buried stripline; 45° DQS routing |

---

### Common Interview Pitfalls

**Specifying FR4 without noting $T_g$:** Standard FR4 at $T_g = 130°$C would fail the industrial temperature requirement. Always verify $T_g$ against operating temperature plus derating margin (typically $T_{op,max} + 20°$C < $T_g$).

**Routing DQ on L1 microstrip:** A frequent mistake. DDR5 DQ signals at 4800 MT/s have fast edges that radiate aggressively from microstrip. Always bury DQ in stripline.

**Assuming 100 Ω differential for DDR5 DQS:** DDR5 uses 80 Ω differential for DQS (each trace at 40 Ω single-ended), not 100 Ω. Confusing this with PCIe/LVDS (100 Ω differential) is common.

**Forgetting the fly-by termination impact:** DDR5 address/command uses fly-by topology with on-die termination (ODT) on the DRAM and source series termination at the CPU. The 40 Ω trace impedance is designed to work with the ODT values in the termination tree — routing at 50 Ω instead would require ODT value changes and may cause reflections at the unterminated fly-by stubs.
