# Stackup Design

## Overview

PCB stackup design is the discipline of specifying the number, order, type, and thickness of copper and dielectric layers that make up a printed circuit board. For high-speed digital designs, the stackup defines the reference plane geometry that sets characteristic impedance, controls return-current paths, governs plane-to-plane coupling, and determines electromagnetic emission behaviour. Every controlled-impedance trace, differential pair, and power delivery network depends directly on stackup decisions made before routing begins.

This document covers stackup symmetry, reference plane strategies, standard layer counts from 4 to 10+, and the signal-plane-signal pairing rules that underpin all high-speed PCB layout.

---

## Tier 1: Fundamentals

### Q1. Why must a PCB stackup be symmetric about its mid-plane?

**Answer:**

A PCB stackup is symmetric about its mid-plane to prevent **board warpage during the lamination process**.

During lamination, the board assembly is heated to cure the prepreg resin, then cooled. Copper and FR4 have different coefficients of thermal expansion (CTE):

- Copper: $\alpha_{Cu} \approx 17$ ppm/°C
- FR4 epoxy-glass (in-plane): $\alpha_{FR4} \approx 14$–17 ppm/°C (in-plane); $\approx 50$–70 ppm/°C (z-axis)

If the copper distribution differs between the top and bottom halves of the board, the two halves contract by different amounts on cooling. This creates a bending moment that bows the board.

**Symmetric stackup rule:** For every copper layer at distance $d$ above the mid-plane, there must be a layer of equal copper weight and coverage at distance $d$ below the mid-plane. Prepreg and core thicknesses must also mirror across the centre.

**Electrical consequence of asymmetry:** An asymmetric stackup also creates unequal trace heights above their respective reference planes on opposite board faces. This produces different impedances for nominally identical traces routed on the top versus bottom signal layers, causing variability that cannot be compensated by a single trace width specification.

**Manufacturing tolerance:** In practice, perfect copper balance is not achievable because signal layers have irregular copper patterns. Fabricators apply a **copper balancing layer** — a solid or cross-hatched copper pour in unused areas of signal layers — to bring the copper percentage of each layer close to 50% before applying the symmetry check.

**Common mistake:** Specifying a 6-layer board as Signal / Ground / Signal / Signal / Power / Signal without ensuring the prepreg thicknesses in the upper half mirror those in the lower half. Even with electrically plausible layer assignments, an unbalanced dielectric buildup will warp.

---

### Q2. What is a reference plane, and what purpose does it serve for a signal trace?

**Answer:**

A reference plane is a continuous or near-continuous copper layer that provides a low-impedance return path for signal currents flowing on an adjacent trace.

**Physical function:**

When a signal current flows along a trace, the return current flows in the reference plane directly beneath (or above) that trace. This return current mirrors the signal current almost exactly, confined to a width of approximately $3 \times h$ centred below the trace, where $h$ is the trace-to-plane separation. This tight current loop minimises loop inductance and sets the trace's characteristic impedance.

**The image charge model:** The reference plane acts as an electromagnetic mirror. The trace and its return current form a transmission line with characteristic impedance:

$$Z_0 = \frac{1}{v_p \cdot C_{per\_unit\_length}}$$

The capacitance per unit length $C$ is determined by the trace geometry (width $W$, thickness $T$) and the dielectric height $h$ above the plane. Removing or interrupting the reference plane destroys the controlled-impedance environment.

**Reference plane requirements:**
1. The plane must be continuous under the trace — splits, slots, and voids in the plane increase loop inductance and impedance discontinuities.
2. The plane must extend at least $3h$ beyond the trace edge on both sides.
3. Via anti-pads (the clearance holes cut through the plane copper for each via) must not merge to create effective slots.

**Why planes must not be split under signal traces:** If a trace crosses a plane split, the return current cannot flow under the crossing point. It must detour around the split, increasing the loop area, radiating like a loop antenna, and creating an impedance discontinuity. Return-path splits are among the most common root causes of EMI failures and signal integrity violations in production boards.

---

### Q3. Draw and describe a standard 4-layer stackup. Identify which layers carry what functions.

**Answer:**

The canonical 4-layer stackup from top to bottom is:

```
Layer 1  — Signal (Top)           — Component side, fine-pitch routing
           [Prepreg ~100–200 µm]
Layer 2  — Ground plane (GND)     — Primary return reference for L1
           [Core ~1.0–1.6 mm]
Layer 3  — Power plane (PWR)      — Power distribution, return ref for L4
           [Prepreg ~100–200 µm]
Layer 4  — Signal (Bottom)        — Secondary routing layer
```

**Total board thickness:** typically 1.6 mm finished for standard designs; the core carries most of the thickness.

**Key characteristics of this stackup:**

1. **L1 references GND (L2):** Layer 1 traces have L2 as their primary reference plane. The close spacing (100–200 µm prepreg) gives low impedance and tight coupling.

2. **L4 references PWR (L3):** Layer 4 traces reference the power plane. While electrically valid (power planes are AC ground for signal return purposes due to decoupling capacitors), this creates noise coupling between the power distribution network and signal return currents.

3. **GND-PWR proximity:** L2 (GND) and L3 (PWR) are separated only by the core. This provides natural plane capacitance:

$$C_{plane} = \epsilon_r \epsilon_0 \frac{A}{h}$$

For a 1600 × 1000 mm² board with 1 mm core separation and $\epsilon_r = 4.2$: $C \approx 59$ nF — a meaningful contribution to high-frequency PDN decoupling.

**Limitations of the 4-layer stackup:**

- Thin prepreg creates asymmetric coupling between the outer signals and the inner planes.
- Only two routing layers — insufficient for dense designs.
- L4 referencing a power plane couples signal return noise into the PDN.
- Impedance control on L4 requires special care because power planes may have splits.

**Practical use:** 4-layer boards are adequate for designs with clock frequencies below ~200 MHz and moderate density. Above this, 6-layer or higher is strongly preferred.

---

### Q4. What does the term "signal-plane-signal" mean, and why is it the fundamental pairing rule?

**Answer:**

"Signal-plane-signal" (S-P-S) refers to the stackup construction rule where every signal layer is immediately adjacent to a reference plane, so the layer sequence always has a signal layer sandwiched between two planes, or at minimum referenced to one nearby plane.

**The pairing rule in practice:**

| Arrangement | Valid? | Notes |
|---|---|---|
| Signal / Plane / Signal | Yes | Both signal layers reference the same plane (stripline-like) |
| Plane / Signal / Plane | Yes | Optimal — signal is a stripline between two reference planes |
| Signal / Signal / Plane | Marginal | Upper signal has no nearby reference plane |
| Signal / Signal / Signal | No | No plane reference; uncontrolled impedance |
| Plane / Plane / Signal | Marginal | Wastes a plane layer; signal references only one plane |

**Physical reason for the rule:**

The return current of a signal trace flows in the adjacent plane layer. If two signal layers are adjacent with no plane between them, three problems arise:

1. **No defined return path:** The return current must find a remote plane, increasing loop inductance by 10–100x.
2. **Broadside coupling:** Adjacent signal layers with traces running in parallel create strong broadside crosstalk, typically 10–20 dB worse than same-layer (edge-coupled) crosstalk.
3. **Uncontrolled impedance:** Without a nearby plane, impedance depends on distant geometry that varies unpredictably across the board.

**Preferred arrangement — dual-referenced signal layers:**

The optimal arrangement is two reference planes sandwiching a signal layer:

$$\text{GND} \rightarrow \text{Signal} \rightarrow \text{GND}$$

This makes the signal a **stripline** with both an upper and lower reference. The symmetric geometry minimises radiation, provides the tightest impedance control, and offers two return current paths.

---

### Q5. Define the terms prepreg and core as used in PCB fabrication.

**Answer:**

**Core** is a rigid, fully-cured dielectric laminate with copper foil bonded to both sides. It is the starting material for any PCB layer stack. Cores are manufactured to precise, calibrated thicknesses (e.g., 0.1 mm, 0.2 mm, 0.5 mm, 1.0 mm). The copper on each side of a core is etched to form the desired layer patterns before lamination.

**Prepreg** (pre-impregnated material) is a woven glass fabric that has been impregnated with uncured or partially-cured epoxy resin. Prepreg is the bonding layer placed between cores (or between a core and an outer copper foil) during lamination. When the stack is pressed and heated, the prepreg resin flows, fills gaps in the copper pattern, and cures to bond the layers together permanently.

**Key differences:**

| Property | Core | Prepreg |
|---|---|---|
| Cure state | Fully cured | B-stage (partially cured) |
| Rigidity before lamination | Rigid, self-supporting | Flexible, cloth-like |
| Copper bonded | Yes, both sides | No copper bonded (added during lamination) |
| Thickness tolerance | Tight (±5–10 µm) | Looser (±15–25 µm, resin flow dependent) |
| Dielectric constant | Well characterised | Slightly variable (resin content varies) |

**Implication for impedance control:** Signal layers on cores have better impedance tolerance than signal layers on prepreg, because core thickness is more tightly controlled. For the most critical high-speed signal layers, specifying them on cores (or between cores) reduces impedance variability. Many fabricators offer tighter prepreg thickness control using "controlled-depth" prepreg.

---

## Tier 2: Intermediate

### Q6. Describe a complete 6-layer stackup suitable for a design with DDR4 memory and 1 GHz LVDS buses. Justify each layer assignment.

**Answer:**

```
Layer 1  — Signal (Top)           — Surface-mount components, short stubs
           [Prepreg ~100 µm, e.g., 2116 half-stack]
Layer 2  — Ground plane (GND)     — Reference for L1; primary EMI shield
           [Core ~0.36 mm]
Layer 3  — Signal (Inner 1)       — High-speed routing, stripline
           [Prepreg ~100 µm]
Layer 4  — Ground plane (GND)     — Reference for L3 and L5
           [Core ~0.36 mm]
Layer 5  — Signal (Inner 2)       — High-speed routing, stripline
           [Prepreg ~100 µm]
Layer 6  — Signal (Bottom)        — Component side reverse
```

Wait — L6 has no adjacent plane. A better 6-layer arrangement is:

```
Layer 1  — Signal (Top)           — Component routing, microstrip
           [Prepreg ~100 µm]
Layer 2  — Ground plane (GND)     — L1 reference
           [Core ~0.36 mm]
Layer 3  — Signal (Inner 1)       — Stripline between L2 and L4
           [Prepreg ~100 µm]
Layer 4  — Power plane (PWR)      — L3 and L5 reference; PDN
           [Core ~0.36 mm]
Layer 5  — Signal (Inner 2)       — Stripline between L4 and L6
           [Prepreg ~100 µm]
Layer 6  — Ground plane (GND)     — L5 reference; bottom EMI shield
```

**Justification by layer:**

**L1 (Signal, Top):** Component placement requires surface access. L1 is a microstrip layer (one reference plane below). Assign shorter, lower-speed nets here: power rail routing stubs, I2C, SPI, control lines. Keep high-speed differential pairs away from L1 to avoid microstrip radiation.

**L2 (GND):** Continuous ground plane. Provides clean, uninterrupted return for L1 microstrip traces. Also provides the top reference for the L3 stripline sandwich.

**L3 (Signal, Inner 1):** Best layer for critical high-speed nets — DDR4 data bytes, LVDS lanes. As a buried stripline between two planes (L2 GND and L4 PWR), it has the lowest radiation, best impedance control, and lowest crosstalk. Route DDR4 address/command/data here.

**L4 (PWR):** Power distribution layer. Acts as AC ground for signal return (with decoupling). Some fabricators split this layer for multiple voltage domains. Avoid routing signals across PWR plane splits.

**L5 (Signal, Inner 2):** Second stripline layer between L4 and L6. Route orthogonally to L3 to minimise broadside coupling. LVDS differential pairs work well here. Routing direction conventions: L3 horizontal, L5 vertical (or vice versa).

**L6 (GND):** Bottom ground plane. Provides L5 reference. Also shields the bottom side of the board, improving EMI performance.

**Impedance targets:**
- L1 microstrip, 50 Ω single-ended: width ≈ 100 µm (depends on $h$ = 100 µm, $\epsilon_r$ = 4.2)
- L3/L5 stripline, 50 Ω: narrower width due to two planes, $h_{total}$ = 200 µm total
- DDR4 differential (100 Ω): width/gap per field solver

**Common mistake:** Placing DDR4 address signals on L1 (microstrip). The DDR4 address bus is a stub-terminated network — the stubs radiate efficiently from a microstrip layer. Burying it in L3 stripline reduces radiated emissions significantly.

---

### Q7. Describe an 8-layer stackup for a PCIe Gen4 design. Explain how the power and ground layers are arranged and why.

**Answer:**

A recommended 8-layer stackup for PCIe Gen4 (16 GT/s, Nyquist frequency 8 GHz):

```
Layer 1  — Signal (Top)           — Microstrip, component attachment
           [Prepreg 3 × 1080 = ~150 µm]
Layer 2  — Ground plane (GND)     — L1 reference
           [Core ~100 µm]
Layer 3  — Signal (Inner 1)       — Stripline (preferred for PCIe lanes)
           [Prepreg 2116 = ~100 µm]
Layer 4  — Ground plane (GND)     — L3 lower reference, L5 upper reference
           [Core ~100 µm]
Layer 5  — Signal (Inner 2)       — Stripline
           [Prepreg 2116 = ~100 µm]
Layer 6  — Power plane (PWR)      — PCIe VCC, L5 lower reference
           [Core ~100 µm]
Layer 7  — Signal (Inner 3)       — Stripline
           [Prepreg 3 × 1080 = ~150 µm]
Layer 8  — Ground plane (GND)     — L7 reference; bottom EMI shield
```

**Design rationale:**

**Multiple ground planes (L2, L4, L8) with one power plane (L6):**

This is the preferred PI-aware arrangement for high-speed boards. Ground planes outnumber power planes 3:1. Each signal layer has at least one ground plane as its primary reference. The power plane (L6) serves L5 and L7 as a secondary reference, but L5 is primarily referenced by L4 (GND) and L6 (PWR), which forms a useful PDN capacitor pair.

**Why not power/ground alternation?**

An alternating GND-PWR-GND-PWR scheme maximises PDN capacitance but means every other signal layer references a power plane. Power planes with splits (multiple voltage rails) disrupt signal return paths. By using predominantly ground planes as references, return-path integrity is maintained even when the power plane is split for multiple rails.

**PCIe Gen4 requirements on this stackup:**

- PCIe differential pairs should be routed on L3 or L5 (dual-referenced striplines) for minimum radiation and best channel performance.
- Insertion loss budget at 8 GHz limits total trace loss to approximately 10 dB (IHV-defined per spec). Thin prepreg (100 µm) raises characteristic impedance slightly, requiring narrow traces and reducing conductor loss per unit length compared to a thick dielectric design.
- Via backdrilling required for vias connecting surface connectors to inner layers (stub resonance at 8 GHz with typical via lengths exceeds 3 dB loss in the pass band without drilling).

**Plane-pair PDN capacitance:**

L4 (GND) and L6 (PWR) separated by L5 prepreg (~100 µm) form a plane capacitor:

$$C_{plane} = \frac{\epsilon_r \epsilon_0 A}{h} = \frac{4.2 \times 8.85 \times 10^{-12} \times 0.15}{100 \times 10^{-6}} \approx 556 \text{ pF per cm}^2$$

For a 10 cm × 8 cm board area, this gives roughly 44 nF of distributed PDN capacitance from the plane pair alone, effective from hundreds of MHz up to several GHz.

---

### Q8. What is a "via stub" and how does it affect a stackup design decision for 10+ layer boards?

**Answer:**

A via stub is the unused portion of a through-hole via that extends beyond the last signal connection point toward the opposite board surface. It acts as a shunt transmission-line stub attached to the via transition.

**Physical model:**

Consider a 10-layer board where a PCIe differential pair enters on Layer 1 (surface connector) and routes on Layer 3 (inner stripline). The via is drilled through all 10 layers. The via barrel connects L1 to L3. Below the L3 pad, the remaining via barrel — from L3 to L10 — is the stub.

```
L1  ●──── signal enters here
L2  |
L3  ●──── signal exits here (routes on L3)
L4  |
... |  ← this portion is the stub
L10 ●──── blind end of drill
```

**Stub resonance:**

The stub is an open-circuited transmission-line stub of length $l_{stub}$. It presents a short-circuit (minimum impedance) at the quarter-wave resonant frequency:

$$f_{resonance} = \frac{v_p}{4 \cdot l_{stub}} = \frac{c}{4 \cdot l_{stub} \sqrt{\epsilon_{r,eff}}}$$

where $v_p$ is the phase velocity in the via dielectric and $\epsilon_{r,eff}$ is the effective relative permittivity of the via environment (approximately 4.0–4.5 for FR4).

A 2 mm stub in FR4 resonates at approximately:

$$f = \frac{3 \times 10^{11} \text{ mm/s}}{4 \times 2 \text{ mm} \times \sqrt{4.2}} \approx 18.3 \text{ GHz}$$

At 8 GHz (PCIe Gen4 Nyquist), a 2 mm stub creates significant insertion loss and group delay distortion — even away from resonance, the stub is reactive.

**Impact on stackup decisions:**

For 10+ layer boards with high-speed signals entering at a surface connector and exiting at an inner layer:

1. **Layer assignment:** Route critical high-speed signals on layers close to the surface where the connector lands. A PCIe connector on L1 should route on L3 rather than L8, minimising stub length.

2. **Backdrill requirement:** If layer assignment cannot keep stubs short, specify backdrilling. The fabricator drills from the opposite board face to remove the stub barrel to within a specified distance of the last connected pad. Achievable stub lengths after backdrilling: 8–10 mil (200–250 µm), giving resonance well above 40 GHz.

3. **Board thickness reduction:** A thinner 10-layer board (e.g., 1.0 mm total) has shorter absolute via stubs than a 2.4 mm board. Reducing board thickness is a stackup-level decision that benefits via performance across all layers.

---

### Q9. Explain the concept of "return current path" and describe what happens to it when a signal crosses a plane split.

**Answer:**

**Return current path:**

When a signal current $I_{signal}$ flows in a trace, an equal and opposite current $-I_{signal}$ must flow in the return path to complete the circuit. At high frequencies (above approximately $f > \frac{v_p}{\pi \times \text{trace length}}$), the return current flows in the plane of lowest impedance, which is directly beneath the trace in the nearest reference plane. The current density follows a distribution approximately:

$$J(x) \propto \frac{h}{h^2 + (x - x_0)^2}$$

where $x_0$ is the position directly below the trace centre and $h$ is the trace-to-plane height. Approximately 80% of the return current flows within a width of $3h$ below the trace.

**Effect of a plane split:**

A plane split is a gap (etched slot) in the reference plane, typically created to separate two voltage domains. If a signal trace routes across a plane split, the return current cannot cross the gap. Instead, it must travel along the edge of the split to the nearest common ground via, then across.

**Consequences:**

1. **Increased loop inductance:** The return current detours by the split width plus via distances. This increases loop area by orders of magnitude compared to the ideal case, raising loop inductance proportionally. A 5 mm detour on a 1 mm trace-to-plane stack increases inductance by roughly $L \approx \mu_0 \times \frac{5 \text{ mm}^2}{1 \text{ mm}} \approx 6$ nH — a severe discontinuity at multi-GHz frequencies.

2. **Impedance discontinuity:** The impedance of the trace jumps at the split crossing because the return path is interrupted. The trace segment over the split has higher impedance (less capacitance to a nearby return plane).

3. **EMI radiation:** The large loop formed by the diverted return current is an efficient loop antenna. Even a 10 mm × 10 mm loop carries sufficient common-mode current to exceed FCC Class B radiated emission limits at frequencies above 100 MHz.

4. **Crosstalk injection:** The diverted return current couples into adjacent circuits near the split edge.

**Rule:** Never route a high-speed signal across a plane split. If unavoidable, use a stitching capacitor (typically 100 nF, 0402) bridging the split, placed as close to the crossing as possible. The stitching capacitor provides an AC path for the return current across the split.

---

## Tier 3: Advanced

### Q10. Design a 10-layer stackup for a board with PCIe Gen5 (32 GT/s), DDR5, and a 100GbE SerDes interface. Specify layer assignments, dielectric materials, and reference plane strategy. Justify each decision.

**Answer:**

**Requirements summary:**

| Interface | Nyquist frequency | Key constraint |
|---|---|---|
| PCIe Gen5 | 16 GHz | Via stubs critical; low loss material needed |
| DDR5 | ~4.8 GHz fundamental | Plane reference, stub minimisation |
| 100GbE SerDes | 26.5625 GHz (PAM4) | Ultra-low loss material; tight impedance |

**Material selection:** Standard FR4 (Df ≈ 0.02 at 10 GHz) is inadequate for PCIe Gen5 and 100GbE. Use a low-loss laminate such as **Megtron 6** (Df ≈ 0.002 at 10 GHz) or **IS680** for the entire board.

**Proposed 10-layer stackup:**

```
Layer 1   — Signal (Top)              — Microstrip, DDR5 address, short-run
             [Prepreg: 2 × 1080, ~100 µm, Megtron 6]
Layer 2   — Ground (GND1)             — L1 reference; primary EMI
             [Core A: ~150 µm, Megtron 6]
Layer 3   — Signal (Inner 1)          — PCIe Gen5 TX/RX pairs (strip)
             [Prepreg: 2 × 1080, ~100 µm, Megtron 6]
Layer 4   — Ground (GND2)             — L3 upper ref; L5 lower ref
             [Core B: ~100 µm, Megtron 6]
Layer 5   — Signal (Inner 2)          — 100GbE SerDes pairs (strip)
             [Prepreg: 2 × 1080, ~100 µm, Megtron 6]
Layer 6   — Ground (GND3)             — L5 lower ref; L7 upper ref
             [Core C: ~100 µm, Megtron 6]
Layer 7   — Signal (Inner 3)          — DDR5 data bytes (strip)
             [Prepreg: 2 × 1080, ~100 µm, Megtron 6]
Layer 8   — Power (PWR)               — L7 lower ref; PDN
             [Core D: ~150 µm, Megtron 6]
Layer 9   — Signal (Inner 4)          — DDR5 address / control (strip)
             [Prepreg: 2 × 1080, ~100 µm, Megtron 6]
Layer 10  — Ground (GND4)             — L9 lower ref; bottom EMI shield
```

**Total board thickness:** approximately 1.1–1.2 mm

**Layer-by-layer justification:**

**L1 (Signal, Top):** Microstrip layer for short DDR5 address/command traces and component interconnections. Microstrip is acceptable for DDR5 (4.8 GHz fundamental) on low-loss material. Surface component placement mandates access.

**L2 (GND1) and L3 (Signal, Inner 1) — PCIe Gen5:**

PCIe Gen5 differential pairs route on L3 as a symmetric stripline (GND2 above, GND2 — wait, GND2 is L4). L3 has L2 above and L4 below: optimal dual-referenced stripline. The 100 µm prepreg above and below L3 gives maximum conductor loss reduction (thin dielectric = lower impedance = wider trace for same $Z_0$ = lower R per unit length).

SerDes connector footprints land on L1. Via transitions to L3 create stubs from L3 to L10. With ~800 µm of board below L3, backdrilling to within 200 µm of L4 leaves 200 µm stub. Stub resonance:

$$f_{stub} = \frac{c}{4 \times 0.2 \text{ mm} \times \sqrt{3.7}} \approx 195 \text{ GHz}$$

Well above 16 GHz — backdrilling essential but achievable.

**L5 (Signal, Inner 2) — 100GbE SerDes:**

100GbE PAM4 at 26.5625 GHz Nyquist requires the lowest insertion loss and cleanest return path. L5, sandwiched between GND2 (L4) and GND3 (L6), is the best dual-referenced stripline. Routing SerDes here maximises isolation and gives the lowest crosstalk. Via stubs from surface connectors to L5: ~600 µm below L5, backdrill to 200 µm stub → resonance > 130 GHz.

**L7 (Signal, Inner 3) — DDR5 data:**

DDR5 data bytes have tight fly-by timing requirements. L7 (between GND3 at L6 and PWR at L8) is a good stripline with GND reference above and power plane below. DDR5 data lines are short (< 50 mm typically), so loss is less critical than for SerDes. Routing data bytes here isolates them from the SerDes layers.

**L8 (PWR):**

Power plane provides PDN for CPU/FPGA core voltage. Pairs with GND3 (L6) across L7 prepreg for distributed PDN capacitance. Note: L8 PWR provides the return reference for L7 and L9. PWR splits for multiple rails must be carefully managed to avoid crossing by DDR5 address traces on L9.

**L9 (Signal, Inner 4) — DDR5 address/command:**

DDR5 address/command/clock are lower frequency than data bytes and acceptable on L9 (between PWR at L8 and GND4 at L10). Routing these here frees L7 for data bytes where impedance uniformity is more critical.

**L10 (GND4):** Bottom ground plane, L9 reference, bottom EMI shield.

**Impedance targets (to be confirmed by field solver):**

| Layer | Structure | Target $Z_0$ |
|---|---|---|
| L1 microstrip | 50 Ω SE, 100 Ω diff | W ≈ 100 µm, h = 100 µm |
| L3 stripline | 50 Ω SE, 100 Ω diff | W ≈ 75 µm, h_above = h_below = 100 µm |
| L5 stripline | 50 Ω SE, 100 Ω diff | Same geometry as L3 |
| L7 stripline | 40 Ω SE (DDR5) | Per DDR5 channel design guide |

**Common interview pitfall:** Assigning the 100GbE SerDes to L1 microstrip because "the connector is on the surface." The connector pads are on the surface, but the differential pairs should transition to inner stripline layers as quickly as possible via backdrilled vias. The loss and radiation characteristics of microstrip at 26 GHz are substantially worse than stripline.

---

## Quick Reference: Standard Stackup Templates

| Layer count | Typical use case | Signal layers | Plane layers |
|---|---|---|---|
| 4 | Simple MCU, <200 MHz | 2 (L1, L4) | 2 (L2 GND, L3 PWR) |
| 6 | DDR3/4, 1G Ethernet | 3–4 | 2–3 |
| 8 | PCIe Gen3/4, DDR5 | 4–5 | 3–4 |
| 10 | PCIe Gen4/5, SerDes, mixed | 5–6 | 4–5 |
| 12+ | Multi-chip modules, 400GbE | 6–8 | 4–6 |

**Layer assignment rules of thumb:**

- Assign the highest-frequency, most critical signals to the inner stripline layers with dual GND references.
- Keep power planes away from the outer faces (thermal and EMI reasons).
- Ensure each signal layer has at least one adjacent ground plane.
- Route orthogonally on adjacent signal layers to reduce broadside coupling.
- Never cross a power plane split with a high-speed signal trace.
- Backdrill any via with a stub length $> \lambda/20$ at the highest signal frequency.
