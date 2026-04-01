# Decoupling Capacitor Placement

## Overview

Decoupling (bypass) capacitors are the primary tool for controlling PDN impedance between the VRM and the IC. Their effectiveness depends not only on their nominal capacitance but on their equivalent series resistance (ESR), equivalent series inductance (ESL), package case size, and critically, their placement on the PCB. A capacitor placed even a centimetre away from the device it is meant to bypass may be nearly useless at the frequencies that matter most. This document covers the physical behaviour of real capacitors, the anti-resonance problem between capacitor stages, capacitor selection methodology, and placement strategy from fundamentals through advanced optimisation.

---

## Tier 1: Fundamentals

### Q1. What is the purpose of a bypass capacitor, and how does it work physically?

**Answer:**

A bypass capacitor serves as a local charge reservoir. When an IC demands a sudden increase in current, the ideal PDN would supply that current instantly from the VRM. In reality, the VRM cannot respond in microseconds (limited by its loop bandwidth), and the inductance of PCB traces and planes means that current cannot flow instantly from any distant supply. The bypass capacitor, placed close to the IC's power pins, stores charge locally and discharges it into the device on demand — acting as a local energy reservoir that bridges the time gap until the VRM can respond.

**Charge/energy perspective:**

For a current step $\Delta I$ lasting a time $\tau$ (the VRM response time $\tau \approx 1/f_{BW}$), the charge that the capacitor must supply is:

$$Q = \Delta I \cdot \tau$$

The resulting voltage droop on the capacitor and hence on the power rail:

$$\Delta V = \frac{Q}{C_{total}} = \frac{\Delta I \cdot \tau}{C_{total}} = \frac{\Delta I}{C_{total} \cdot f_{BW}}$$

Setting $\Delta V \leq \Delta V_{max}$:

$$C_{total} \geq \frac{\Delta I}{f_{BW} \cdot \Delta V_{max}}$$

**Frequency domain perspective:**

The bypass capacitor lowers the PDN impedance in the frequency range around its self-resonant frequency (SRF). Below SRF the capacitor behaves capacitively (impedance falls with frequency); above SRF it behaves inductively (impedance rises with frequency). The capacitor most effectively bypasses noise at and near its SRF, where its impedance is at its minimum, $R_{ESR}$.

---

### Q2. What is ESR, and what determines a capacitor's ESR? Why does it matter for PDN design?

**Answer:**

ESR (equivalent series resistance) is the total resistive loss in a real capacitor, lumped into a single series element in the model. It arises from:

1. **Lead and termination resistance:** resistance of the metal end-caps, solder joints, and PCB lands (usually a few to tens of milliohms)
2. **Internal electrode resistance:** for MLCCs, the inner electrode metal (nickel or silver-palladium) contributes a few milliohms; for electrolytics, the aluminium foil and electrolyte contribute much more
3. **Dielectric loss:** energy dissipated in the dielectric material due to polarisation lag (represented as a loss tangent; small for ceramics, significant for electrolytics)

**Typical ESR values:**

| Capacitor type | Typical ESR |
|---|---|
| Aluminium electrolytic (100 µF–2200 µF) | 50–500 m$\Omega$ |
| Polymer electrolytic (100 µF–1000 µF) | 5–30 m$\Omega$ |
| X5R MLCC, 0402, 100 nF | 20–60 m$\Omega$ |
| X5R MLCC, 0805, 10 µF | 3–15 m$\Omega$ |
| X7R MLCC, 0201, 10 nF | 50–100 m$\Omega$ |

**Why ESR matters:**

At the SRF, $|Z_{cap}| = R_{ESR}$. This is the minimum impedance that capacitor can present to the PDN. To achieve $Z_{PDN} \leq Z_{target}$, the parallel combination of $N$ capacitors must satisfy:

$$R_{ESR,total} = \frac{R_{ESR,single}}{N} \leq Z_{target}$$

$$N \geq \frac{R_{ESR,single}}{Z_{target}}$$

For $Z_{target} = 5\ m\Omega$ and $R_{ESR,single} = 25\ m\Omega$, at least $N = 5$ capacitors in parallel are needed to achieve target impedance at the SRF. ESR also determines the Q-factor of any resonant peaks and how well anti-resonance peaks are damped.

---

### Q3. What is ESL, and how does it limit a capacitor's useful frequency range?

**Answer:**

ESL (equivalent series inductance) is the self-inductance of the capacitor's internal structure and external connections. For a surface-mount MLCC, ESL arises from:

1. **Internal electrode geometry:** the interleaved electrode structure has a finite loop area; current enters one end-cap and exits the other, forming a current loop of area $A_{loop}$: $L \propto \mu_0 A_{loop} / \ell$
2. **External terminations:** solder pads and the via connections add inductance in series
3. **PCB trace from pad to via:** even a 0.5 mm trace adds 0.3–0.5 nH

**Typical ESL values by case size:**

| Case size | Typical $L_{ESL}$ | $f_{SRF}$ with 100 nF |
|---|---|---|
| 1210 | 2–3 nH | 9–12 MHz |
| 0805 | 1–2 nH | 11–16 MHz |
| 0402 | 0.5–1 nH | 16–22 MHz |
| 0201 | 0.3–0.5 nH | 23–30 MHz |
| 01005 | 0.1–0.3 nH | 29–46 MHz |

**Impact:** Above the SRF, the capacitor presents inductive impedance $|Z| \approx 2\pi f L_{ESL}$. At 100 MHz, a 0402 MLCC with $L_{ESL} = 0.8\ nH$ presents $|Z| = 2\pi \times 10^8 \times 0.8\times10^{-9} \approx 0.5\ \Omega$ — far above any reasonable PDN target. The capacitor is useless as a bypass element at that frequency regardless of its capacitance value.

**ESL reduction strategies:**
- Use smaller case sizes (reduces internal electrode loop area)
- Use reverse-geometry capacitors (current enters along the long axis, reducing loop area)
- Mount with via-in-pad (eliminates trace from pad to via)
- Use two vias per capacitor (parallel via inductance)
- Place capacitor on the same side of the board as the IC, immediately adjacent to the power pins

---

### Q4. Explain the concept of a capacitor's self-resonant frequency (SRF) and its practical significance for capacitor selection.

**Answer:**

The SRF is the frequency at which the capacitive reactance $X_C = 1/(2\pi f C)$ and inductive reactance $X_L = 2\pi f L_{ESL}$ are equal and opposite, leaving only the resistive term $R_{ESR}$:

$$f_{SRF} = \frac{1}{2\pi\sqrt{L_{ESL} \cdot C}}$$

At this frequency, the capacitor presents its minimum impedance. It is also the transition point: below SRF the device is capacitive; above SRF it is inductive.

**Practical selection rule:** Select a capacitor whose SRF falls within the frequency range you need to bypass. A capacitor with SRF at 10 MHz is most effective at 10 MHz; it provides little benefit at 100 MHz (where it is inductive) or at 100 kHz (where its capacitive reactance may be sufficient but you might use bulk caps instead).

**SRF shifts with derating:**

MLCC capacitors using class II dielectrics (X5R, X7R) have capacitance that decreases with applied DC bias due to the voltage coefficient:

- At 0 V bias: nominal $C$
- At 50% of rated voltage: capacitance may drop to 60–80% of nominal
- At rated voltage: capacitance may drop to 30–60% of nominal

This shifts $f_{SRF}$ upward (because $C_{actual} < C_{nominal}$):

$$f_{SRF,biased} = f_{SRF,unbiased} \times \sqrt{\frac{C_{nominal}}{C_{actual}}}$$

**Example:** A 10 µF 0402 X5R capacitor rated 10 V, operated at 1 V bias (10% of rated voltage), may have only 70% of its nominal capacitance. $f_{SRF}$ shifts by a factor of $\sqrt{1/0.7} \approx 1.20$ — a 20% upward shift. At higher rail voltages (e.g., 3.3 V on a 10 V rated cap = 33% bias), the capacitance drop is more severe and must be accounted for in PDN calculations.

**Rule of thumb:** Always derate MLCC capacitance by at least 20–30% for voltage coefficient when computing PDN impedance.

---

## Tier 2: Intermediate

### Q5. Explain anti-resonance between capacitor stages. Derive the anti-resonance frequency and peak impedance for two capacitor stages in parallel.

**Answer:**

Anti-resonance is the phenomenon where the parallel combination of two capacitor stages exhibits a high-impedance peak at an intermediate frequency, rather than the low impedance one might naively expect.

**Physical mechanism:**

Consider two capacitor stages in parallel:
- Stage 1 (bulk): large $C_B$, large $L_B$ (e.g., $L_B = 5\ nH$, $C_B = 100\ \mu F$, $f_{SRF,B} \approx 225\ kHz$)
- Stage 2 (MLCC): small $C_M$, small $L_M$ (e.g., $L_M = 0.5\ nH$, $C_M = 10\ \mu F$, $f_{SRF,M} \approx 22.5\ MHz$)

At a frequency between their SRFs (say, 5 MHz), Stage 1 is above its SRF and behaves as an inductor of value $L_B$. Stage 2 is below its SRF and behaves as a capacitor of value $C_M$.

Their parallel combination is a parallel LC circuit. A parallel LC circuit has maximum impedance at resonance — this is the dual of a series LC circuit (which has minimum impedance at resonance). The peak impedance occurs at:

$$f_{AR} = \frac{1}{2\pi\sqrt{L_B \cdot C_M}}$$

**With ESR and ESL in full form (simplified, neglecting ESR):**

For Stage 1 above its SRF: $Z_1 \approx j\omega L_B$

For Stage 2 below its SRF: $Z_2 \approx 1/(j\omega C_M)$

Parallel combination:

$$Z_{par} = \frac{Z_1 Z_2}{Z_1 + Z_2} = \frac{(j\omega L_B)(1/j\omega C_M)}{j\omega L_B + 1/j\omega C_M} = \frac{L_B/C_M}{j\omega L_B + 1/(j\omega C_M)}$$

Denominator zero when $j\omega L_B = -1/(j\omega C_M)$, i.e., $\omega^2 = 1/(L_B C_M)$:

$$f_{AR} = \frac{1}{2\pi\sqrt{L_B \cdot C_M}}$$

At $f_{AR}$ the denominator approaches zero and the parallel impedance approaches a peak. The finite ESR of each stage limits the peak to:

$$|Z_{AR}|_{max} \approx \sqrt{\frac{L_B}{C_M}} \cdot \frac{1}{R_B/L_B\omega_{AR} + R_M C_M \omega_{AR}}$$

In the limit of low ESR:

$$|Z_{AR}|_{max} \approx \sqrt{\frac{L_B}{C_M}}$$

**Numerical example:** $L_B = 5\ nH$, $C_M = 10\ \mu F$:

$$f_{AR} = \frac{1}{2\pi\sqrt{5\times10^{-9} \times 10\times10^{-6}}} = \frac{1}{2\pi \times 7.07\times10^{-6}} \approx 22.5\ kHz$$

Wait — that seems low. Let me redo: $\sqrt{5\times10^{-9} \times 10\times10^{-6}} = \sqrt{5\times10^{-14}} = 2.24\times10^{-7}$

$$f_{AR} = \frac{1}{2\pi \times 2.24\times10^{-7}} \approx 710\ kHz$$

$$|Z_{AR}| = \sqrt{\frac{5\times10^{-9}}{10\times10^{-6}}} = \sqrt{5\times10^{-4}} \approx 22\ m\Omega$$

This peak of 22 m$\Omega$ at 710 kHz could easily exceed a $Z_{target}$ of 5–10 m$\Omega$.

---

### Q6. How do you mitigate anti-resonance between bulk capacitor and MLCC stages? What is the damping resistor technique?

**Answer:**

**Option 1: Add series resistance (damping resistor)**

The anti-resonance peak is limited by the ESR of the capacitors. Adding a deliberate series resistor $R_d$ in series with the bulk capacitor (or a subset of bulk capacitors) adds damping to the parallel tank:

$$|Z_{AR}|_{damped} \approx \frac{R_B + R_d}{2}$$

The optimal damping resistor value is approximately:

$$R_{d,opt} \approx \sqrt{\frac{L_B}{C_M}} - R_B \approx |Z_{AR}|_{ideal} - R_B$$

**Trade-off:** Adding $R_d$ increases the impedance of the bulk stage at low frequencies (around and below the bulk SRF). This raises $Z_{PDN}$ at low frequencies but tames the anti-resonance peak. The design must verify that the raised low-frequency impedance does not violate $Z_{target}$ in that range.

**Practical implementation:** Rather than one large bulk capacitor, use two in parallel. Place $R_d$ in series with one of them. The undamped capacitor provides low impedance at low frequencies; the damped one damps the resonance at intermediate frequencies.

**Option 2: Frequency overlap between stages**

Design the two stages so their impedance curves overlap well: Stage 1 has its SRF low enough that its impedance is still falling (not yet rising inductively) at the frequency where Stage 2 becomes effective. In practice this means the bulk stage SRF should be close to the MLCC stage SRF — but with large $L_B$ and large $C_B$, this is difficult without choosing very large MLCC values.

**Option 3: Increase Stage 2 capacitance**

From $|Z_{AR}| = \sqrt{L_B/C_M}$: doubling $C_M$ reduces the anti-resonance peak by a factor of $\sqrt{2}$ (3 dB). This is less efficient than adding damping but avoids the trade-off with low-frequency impedance.

**Option 4: Reduce Stage 1 ESL**

Reducing $L_B$ (using polymer capacitors with lower ESL, or stacked capacitors) directly lowers the anti-resonance impedance. This is often the best long-term solution: a polymer electrolytic has 2–5 nH ESL versus 10–20 nH for aluminium electrolytic.

---

### Q7. Describe a systematic capacitor selection methodology for a PDN design from scratch.

**Answer:**

**Step 1: Define the target impedance**

$$Z_{target} = \frac{\Delta V_{max,AC}}{I_{step}}$$

Also determine the frequency range: $f_{low} = f_{BW,VRM}$ to $f_{high} = 0.35/t_{rise}$.

**Step 2: Select bulk capacitors**

Criteria:
- Minimum capacitance: $C_{bulk} \geq 1/(2\pi f_{low} Z_{target})$
- ESR per capacitor: select type with ESR $< Z_{target} \times N_{bulk}$ (where $N_{bulk}$ is the number planned)
- Voltage rating: $\geq 1.5\times V_{nom}$ (for safety margin and to limit capacitance derating)
- Physical: fit within board area near VRM, correct case size

Recommended: Polymer aluminium (OS-CON type) for $C_{bulk}$ from 100 µF to 1000 µF, ESR 5–30 m$\Omega$, ESL 2–5 nH. These balance good low-frequency performance with lower ESL than standard electrolytics.

**Step 3: Select mid-frequency MLCCs**

For each frequency decade from 100 kHz to 10 MHz:
- Target SRF in the decade: choose $C$ and case size so $f_{SRF}$ is within the range
- Calculate number needed: $N = R_{ESR,single} / Z_{target}$
- Verify: total $C_{MLCC}$ must satisfy $C \geq 1/(2\pi f_{decade,low} Z_{target})$
- Check anti-resonance: $|Z_{AR}| = \sqrt{L_{prev,ESL}/C_{this}}$ must be $< Z_{target}$

Recommended: X5R or X7R dielectric MLCCs (0805 or 0402 case) in 1–10 µF values for mid-frequency coverage.

**Step 4: Select high-frequency MLCCs**

For coverage from 10 MHz to $f_{high}$:
- Use smallest practical case size (0201 or 01005) to minimise $L_{ESL}$
- Use low capacitance values (1–100 nF) to push SRF higher
- Place immediately adjacent to power pins (minimise via inductance)
- Via-in-pad construction strongly preferred

Recommended: X7R or C0G 0201 MLCCs in 10–100 nF values for high-frequency bypass.

**Step 5: Check voltage derating**

For every MLCC selected, verify:

$$C_{effective} = C_{nominal} \times k_{voltage}(V_{bias}/V_{rated})$$

where $k_{voltage}$ is read from the capacitor's voltage coefficient curve. Recompute $f_{SRF}$ and $|Z|$ using $C_{effective}$.

**Step 6: Verify with simulation**

Build a SPICE or power integrity simulation model with all selected capacitors (using manufacturer-provided SPICE models when available, or the R-L-C series model). Plot $|Z_{PDN}(f)|$ versus frequency and verify it stays below $Z_{target}$ across the entire band. Iterate on capacitor count and placement until the specification is met.

---

### Q8. What placement rules govern where bypass capacitors are positioned relative to the IC, and how does trace inductance affect effectiveness?

**Answer:**

**Rule 1: Proximity to power pins**

The closer the capacitor to the IC power pin, the lower the inductance of the connection:

$$L_{trace} \approx \mu_0 \cdot \frac{\ell}{w} \cdot h_{layer}$$

For a 1 mm trace of width 0.2 mm on a layer 0.1 mm above the reference plane: $L_{trace} \approx 4\pi\times10^{-7} \times (1/0.2) \times 0.1\times10^{-3} \approx 0.63\ nH$. This inductance adds directly to the capacitor's $L_{ESL}$, raising the effective SRF-limiting inductance.

**Rule 2: Via placement**

Each via adds approximately 0.3–1 nH of inductance depending on via diameter, length, and proximity to the ground plane. Best practice:
- Use two vias per capacitor: one to the power plane, one to the ground plane. Two vias in parallel halve the via inductance.
- Place vias immediately at the capacitor pads (via-in-pad or adjacent-via), not at the end of a pad trace.
- Shorten the via stub (back-drill or use a buried via to the nearest plane layer).

**Rule 3: Ground plane connection**

The return current path is as important as the power current path. An MLCC connected to a power plane through a short via but returning through a long trace to a distant ground via has high loop inductance dominated by the ground return.

**Rule 4: Do not chain capacitors**

Connecting two capacitors in a daisy chain (power → cap 1 → cap 2 → GND) adds the trace inductance between them. Capacitor 2 sees the inductance of the trace between cap 1 and cap 2 in series with its own ESL, making it far less effective than if connected directly.

**Rule 5: Component side placement**

Place decoupling capacitors on the same PCB layer as the IC (top side for top-side BGAs). Via transitions between layers add inductance. A capacitor on the bottom side of the board, accessed through a via from the IC on the top side, adds full board-thickness via inductance.

**Exception:** For BGA packages, placing capacitors in the BGA escape pattern (in the "void" area between BGA rows) or using capacitors on the opposite side of the board directly under the BGA footprint (back-side decoupling) can position them closer to the die than edge-of-package placement.

---

## Tier 3: Advanced

### Q9. What is via-in-pad (VIP) construction for decoupling capacitors, and how much does it reduce effective ESL? Provide a quantitative comparison.

**Answer:**

Via-in-pad (VIP) places the PCB via directly within the solderable pad of the capacitor, eliminating the trace segment between the component pad and the via. This is the single most effective PCB layout technique for reducing high-frequency capacitor ESL.

**Conventional layout (via adjacent to pad):**

The current path is: capacitor end-cap → solder → PCB pad → trace to via (length $\ell_{trace}$) → via barrel → power/ground plane.

The total series inductance:

$$L_{conv} = L_{ESL,cap} + L_{trace}(\ell_{trace}) + L_{via}$$

For a 0402 capacitor with a 0.5 mm trace to the via: $L_{ESL,cap} \approx 0.7\ nH$, $L_{trace} \approx 0.3\ nH$, $L_{via} \approx 0.5\ nH$:

$$L_{conv} \approx 0.7 + 0.3 + 0.5 = 1.5\ nH$$

**Via-in-pad layout:**

The trace length $\ell_{trace} = 0$ (via is in the pad):

$$L_{VIP} = L_{ESL,cap} + L_{via} \approx 0.7 + 0.3 = 1.0\ nH$$

With via-in-pad and micro-via (low-profile via): $L_{via} \approx 0.1\ nH$:

$$L_{VIP,micro} \approx 0.7 + 0.1 = 0.8\ nH$$

**Result:** VIP reduces effective series inductance by 30–50% compared to conventional adjacent-via placement. For small-value capacitors where $L_{ESL,cap}$ is very small (0201, 01005), the via and trace inductance can dominate — VIP provides an even larger relative improvement for these cases.

**SRF shift from VIP (with 100 nF capacitor):**

$$\frac{f_{SRF,VIP}}{f_{SRF,conv}} = \sqrt{\frac{L_{conv}}{L_{VIP}}} = \sqrt{\frac{1.5}{0.8}} \approx 1.37$$

VIP construction shifts the SRF of a 100 nF 0402 MLCC from approximately 18 MHz to approximately 25 MHz — meaningful coverage improvement at high frequencies.

**Manufacturing consideration:** VIP requires the via to be filled and planarised (copper-filled or resin-filled) before soldering. Unfilled vias in pads cause solder wicking and inconsistent solder joint formation. The added fabrication cost is justified for high-performance PDN designs above 100 MHz.

---

### Q10. How do you use electromagnetic simulation to optimise capacitor placement? What metrics do you extract and how are they used?

**Answer:**

EM simulation of the PCB PDN uses 2.5D or 3D EM solvers (e.g., Ansys SIwave, Cadence Clarity, Mentor HyperLynx PI, or open-source tools based on method-of-moments) to extract a frequency-domain impedance matrix for the power distribution system.

**What to model:**

The simulation model includes:
- PCB copper layers (power and ground planes, shapes, cutouts)
- Via geometry (drill size, pad size, anti-pad clearance, layer connections)
- Decoupling capacitor placements (as ports or as lumped component models)
- Package footprint (IC outline, ball map, power and ground pin locations)

**Key outputs:**

1. **Self-impedance $Z_{11}(f)$** at the IC power pins: the impedance seen by the device. Compare against $Z_{target}$.

2. **Transfer impedance $Z_{21}(f)$** between aggressor and victim locations: characterises power supply noise coupling.

3. **Current density plots** at specific frequencies: show where current is concentrated, revealing bottlenecks (e.g., narrow plane necks, anti-pad-dominated regions).

4. **Effective capacitance maps**: at a given frequency, show which physical capacitors are contributing to the PDN impedance and which are not.

**Optimisation workflow:**

1. Simulate with all capacitors placed at nominal locations.
2. Identify frequency regions where $|Z_{11}| > Z_{target}$.
3. At each problem frequency, examine the current density — does current flow through all capacitors or only a few?
4. Move capacitors closer to the high-current regions, or add capacitors at under-served locations.
5. Re-simulate to verify improvement.
6. Iterate until the impedance profile satisfies $Z_{target}$ across the full band.

**Automated optimisation:** Modern PI tools include capacitor placement optimisation engines. Given a set of available placement sites and capacitor types, they compute the minimum capacitor count and optimal placement to meet a specified $Z_{target}$ profile. These tools are valuable for complex BGAs with hundreds of power pins.

---

### Q11. What is back-side decoupling, and when is it preferable to front-side capacitor placement?

**Answer:**

Back-side decoupling places bypass capacitors on the opposite (bottom) side of the PCB, directly beneath the BGA footprint of the IC.

**Geometry of back-side decoupling:**

For a BGA device soldered to the top side of the board, back-side capacitors are positioned on the bottom side. The connection is through vias from the capacitor pads on the bottom side up through the board to the BGA balls on the top side. The current path length from the back-side capacitor to the IC power ball is approximately the PCB thickness plus the via length, typically 1.0–2.5 mm total.

**Comparison with front-side capacitors:**

Front-side capacitors cannot be placed directly under the BGA — they must go around the periphery. The closest a front-side capacitor can be to a central BGA power ball may be 5–15 mm, incurring significant spreading inductance through the power plane. A back-side capacitor directly beneath a central power ball may be only 1–2 mm away through the board.

**When back-side decoupling is better:**

- High-power BGAs with many power balls spread across the package interior
- When front-side routing density precludes adding capacitors near the BGA edge
- For frequencies above 50 MHz where the spreading inductance from edge-of-BGA capacitors exceeds the via inductance of back-side capacitors

**When back-side decoupling is worse:**

- Small, low-pin-count ICs where front-side proximity is achievable
- Very thin boards (below 0.8 mm) where the via inductance is inherently low from either side
- When the BGA balls provide no access to the bottom-side via pads (solid copper area under the BGA blocks all plane connections)

**Practical inductance comparison:**

Front-side cap, 8 mm from BGA power ball (plane spreading):

$$L_{spreading} \approx \frac{\mu_0 h}{2\pi} \ln\left(\frac{8}{0.15}\right) \approx \frac{4\pi\times10^{-7}\times100\times10^{-6}}{2\pi}\ln(53) \approx 20\ nH \times 3.97 \approx 3.2\ nH$$

(using $h = 100\ \mu m$, $d = 8\ mm$, $r_0 = 0.15\ mm$)

Back-side cap, 1.6 mm board thickness via:

$$L_{via} \approx 0.2 \times h_{via} \approx 0.2 \times 1.6 = 0.32\ nH$$

(using 0.2 nH/mm rule of thumb for a 0.3 mm diameter via)

The back-side capacitor's total inductance is approximately 10x lower in this scenario — a major advantage for high-frequency coverage.

---

## Summary: Decoupling Capacitor Selection Quick Reference

| Stage | Capacitor type | Value range | Case size | $f_{SRF}$ target | Placement |
|---|---|---|---|---|---|
| Bulk | Polymer electrolytic | 100 µF–1 mF | Radial/SMD | 200–500 kHz | Near VRM, within 20 mm of device |
| Mid-freq | X5R/X7R MLCC | 1–47 µF | 0805/0402 | 1–10 MHz | Within 5 mm of device pins |
| High-freq | X7R/C0G MLCC | 10–100 nF | 0402/0201 | 10–100 MHz | Adjacent to power pins, VIP preferred |
| Ultra HF | C0G MLCC | 1–10 nF | 0201/01005 | 100–500 MHz | In BGA keepout or back-side |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Self-resonant frequency | $f_{SRF} = 1/(2\pi\sqrt{L_{ESL} C})$ |
| Minimum impedance | $|Z_{min}| = R_{ESR}$ |
| $N$ caps in parallel, ESR | $R_{total} = R_{ESR}/N$ |
| $N$ caps in parallel, ESL | $L_{total} = L_{ESL}/N$ |
| Anti-resonance frequency | $f_{AR} = 1/(2\pi\sqrt{L_{bulk,ESL} C_{MLCC}})$ |
| Anti-resonance peak | $|Z_{AR}| \approx \sqrt{L_{bulk,ESL}/C_{MLCC}}$ |
| Trace inductance (approx) | $L_{trace} \approx \mu_0 (\ell/w) h$ |
| Via inductance (rule of thumb) | $L_{via} \approx 0.2\ \text{nH/mm}$ (for 0.3 mm drill) |
| Required cap count (ESR) | $N \geq R_{ESR,single}/Z_{target}$ |
