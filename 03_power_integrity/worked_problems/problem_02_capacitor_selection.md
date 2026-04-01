# Problem 02: Capacitor Selection for a PDN Design

## Problem Statement

You are designing the decoupling network for a DDR5 memory interface on a server motherboard. The DDR5 VDD rail has the following specifications:

- Nominal voltage: $V_{nom} = 1.1\ V$
- Total current: $I_{total} = 6\ A$ (steady state)
- Maximum current transient step: $\Delta I = 4\ A$ (occurs during burst read/write operations)
- Current transient rise time: $t_{rise} = 500\ ps$
- Voltage tolerance: $\pm 2\%$ of nominal
- VRM loop bandwidth: $f_{BW} = 80\ kHz$ (constraint: the DDR5 PMIC has a limited loop bandwidth)

The available MLCC capacitors from your approved vendor list are:

| Part | Capacitance | Rated voltage | Dielectric | Case | $R_{ESR}$ | $L_{ESL}$ | Cost/unit |
|---|---|---|---|---|---|---|---|
| M1 | 10 µF | 6.3 V | X5R | 0402 | 5 m$\Omega$ | 0.7 nH | $0.04 |
| M2 | 10 µF | 6.3 V | X5R | 0201 | 8 m$\Omega$ | 0.35 nH | $0.09 |
| M3 | 100 nF | 25 V | X7R | 0402 | 50 m$\Omega$ | 0.5 nH | $0.02 |
| M4 | 100 nF | 25 V | C0G | 0402 | 100 m$\Omega$ | 0.45 nH | $0.05 |
| M5 | 10 nF | 50 V | C0G | 0201 | 150 m$\Omega$ | 0.3 nH | $0.03 |

And a bulk polymer capacitor:

| Part | Capacitance | Rated voltage | $R_{ESR}$ | $L_{ESL}$ |
|---|---|---|---|---|
| P1 | 220 µF | 4 V | 12 m$\Omega$ | 3 nH |

**Tasks:**

**(a)** Derive the target impedance, justifying your budget allocation. The DDR5 VDD rail is one of the most demanding rails for voltage accuracy — comment on why.

**(b)** Determine the frequency range that must be covered by the PDN design.

**(c)** Calculate the required bulk capacitance and determine the minimum number of P1 capacitors.

**(d)** For mid-frequency coverage (around the SRF of the M1 capacitors), determine the required count after applying voltage coefficient derating. Compare M1 (0402) with M2 (0201) for this application.

**(e)** Determine whether M3 (X7R, 100 nF) or M4 (C0G, 100 nF) is more appropriate for high-frequency bypass near the DDR5 memory devices. Calculate the SRF for both and explain your choice.

**(f)** Calculate the anti-resonance between the P1 bulk stage and the M1 mid-frequency stage. Does it require mitigation?

**(g)** Propose a final BOM (bill of materials) with capacitor placement strategy and justify each stage choice.

---

## Solution

### Part (a): Target Impedance for DDR5 VDD

**Why DDR5 VDD is demanding:**

DDR5 differs from DDR4 in that the on-DIMM PMIC is absent in some configurations, or the motherboard VRM directly supplies VDD. The JEDEC DDR5 specification defines:
- Nominal VDD: 1.1 V
- Tolerance: VDD$_{min}$ = 1.07 V, VDD$_{max}$ = 1.13 V → $\pm 27.3\ mV$ total window

This is tighter than DDR4 ($\pm 3\%$) and reflects the sensitivity of DDR5 receivers and write/read timing to supply noise. Excessive supply ripple degrades eye height and increases bit error rate.

**Budget allocation ($\pm 27\ mV$ total):**

| Contributor | Allocation |
|---|---|
| PMIC DC accuracy (on-die regulation) | 8 mV |
| PCB IR drop to DIMMs | 7 mV |
| Thermal drift over temperature range | 3 mV |
| **AC PDN ripple** | **9 mV** |
| Total | 27 mV |

**Target impedance:**

$$\Delta V_{max,AC} = 9\ mV$$

$$\boxed{Z_{target} = \frac{\Delta V_{max}}{I_{step}} = \frac{9\ mV}{4\ A} = 2.25\ m\Omega}$$

Round down slightly for margin: **use $Z_{target} = 2\ m\Omega$** for design calculations.

---

### Part (b): Frequency Range

**Lower bound:**

$$f_{low} = f_{BW,PMIC} = 80\ kHz$$

**Upper bound:**

$$t_{rise} = 500\ ps$$

$$f_{high} = \frac{0.35}{t_{rise}} = \frac{0.35}{500\times10^{-12}} = 700\ MHz$$

The 700 MHz upper bound reflects the very fast current transients in DDR5 at 4800 MT/s and above. PCB-mounted capacitors are generally ineffective above approximately 300 MHz; above that frequency, on-package and on-die capacitance in the DDR5 devices handles the demand. Design the PCB PDN to 300 MHz and note that the device's internal capacitance covers the remainder.

**Design frequency range:**

$$\boxed{80\ kHz\ \text{to}\ 300\ MHz \text{ (PCB-mounted capacitors)}}$$

---

### Part (c): Bulk Capacitance

**Transient energy method (dominant constraint for low $f_{BW}$):**

VRM response time: $\tau = 1/f_{BW} = 1/80\ kHz = 12.5\ \mu s$

Charge demanded during the VRM latency:

$$Q = \Delta I \times \tau = 4 \times 12.5\times10^{-6} = 50\ \mu C$$

Voltage droop:

$$\Delta V = \frac{Q}{C_{bulk}} \leq \Delta V_{max} = 9\ mV$$

$$C_{bulk,min} = \frac{Q}{\Delta V_{max}} = \frac{50\times10^{-6}}{9\times10^{-3}} \approx 5.6\ mF$$

**Impedance method check:**

$$C_{bulk,min,impedance} = \frac{1}{2\pi f_{low} \cdot Z_{target}} = \frac{1}{2\pi \times 80\times10^3 \times 2\times10^{-3}} = \frac{1}{1005} \approx 994\ \mu F$$

The energy method gives the larger, more demanding result: $C_{bulk,min} = 5.6\ mF$.

**Number of P1 capacitors:**

Capacitance requirement:

$$N_{P1} \geq \frac{5.6\ mF}{220\ \mu F} = 25.5 \rightarrow N_{P1} = 26$$

ESR check:

$$R_{ESR,P1,total} = \frac{12\ m\Omega}{26} = 0.46\ m\Omega < 2\ m\Omega \quad \checkmark$$

$$\boxed{N_{P1} = 26\ \text{polymer capacitors (220\ \mu F each)}}$$

**Comment:** 26 capacitors represents significant board area and cost. This is a direct consequence of the low PMIC loop bandwidth (80 kHz). In a real design, qualifying a PMIC with 200+ kHz bandwidth would reduce the requirement to $5.6\ mF \times (80/200) = 2.2\ mF$, or approximately 10 capacitors.

---

### Part (d): Mid-Frequency MLCC — M1 vs. M2 Comparison

**Voltage coefficient derating for X5R at 1.1 V on a 6.3 V rated capacitor:**

Bias ratio: $1.1/6.3 = 17.5\%$ of rated voltage.

For X5R at approximately 18% bias: capacitance typically retains approximately 85% of nominal.

$$C_{eff,M1} = C_{eff,M2} = 10\ \mu F \times 0.85 = 8.5\ \mu F$$

(Same derating applies to both since they share the same capacitance value and dielectric type.)

**SRF calculation (derated):**

For M1 (0402, $L_{ESL} = 0.7\ nH$, $C_{eff} = 8.5\ \mu F$):

$$f_{SRF,M1} = \frac{1}{2\pi\sqrt{0.7\times10^{-9} \times 8.5\times10^{-6}}} = \frac{1}{2\pi\sqrt{5.95\times10^{-15}}} = \frac{1}{2\pi \times 77.1\times10^{-9}} \approx 2.06\ MHz$$

For M2 (0201, $L_{ESL} = 0.35\ nH$, $C_{eff} = 8.5\ \mu F$):

$$f_{SRF,M2} = \frac{1}{2\pi\sqrt{0.35\times10^{-9} \times 8.5\times10^{-6}}} = \frac{1}{2\pi\sqrt{2.975\times10^{-15}}} = \frac{1}{2\pi \times 54.5\times10^{-9}} \approx 2.92\ MHz$$

M2 (0201) achieves a higher SRF than M1 (0402) because of its lower ESL.

**Required count — ESR criterion:**

$$N_{M1,min} = \frac{R_{ESR,M1}}{Z_{target}} = \frac{5\ m\Omega}{2\ m\Omega} = 2.5 \rightarrow 3$$

$$N_{M2,min} = \frac{R_{ESR,M2}}{Z_{target}} = \frac{8\ m\Omega}{2\ m\Omega} = 4$$

**Comparative summary:**

| Property | M1 (0402, 10 µF X5R) | M2 (0201, 10 µF X5R) |
|---|---|---|
| SRF (derated) | 2.06 MHz | 2.92 MHz |
| $R_{ESR}$ | 5 m$\Omega$ | 8 m$\Omega$ |
| Minimum count (ESR) | 3 | 4 |
| Cost at minimum count | $0.12 | $0.36 |
| Frequency coverage | 1–10 MHz | 1.5–15 MHz |
| SMT process requirement | Standard 0402 | Requires 0201 process |

**Recommendation:** Use M1 (0402) — lower ESR, lower cost, standard process capability. M2 would be appropriate only if 0201 placement is already required elsewhere on the board and space near the DIMM connector is extremely limited.

$$\boxed{N_{M1} = 3\ \text{to}\ 4 \text{ units. Use M1 (0402) X5R.}}$$

---

### Part (e): M3 (X7R) vs. M4 (C0G) for High-Frequency Bypass

**SRF for M3 (X7R, 100 nF 0402, $L_{ESL} = 0.5\ nH$):**

$$f_{SRF,M3} = \frac{1}{2\pi\sqrt{0.5\times10^{-9} \times 100\times10^{-9}}} = \frac{1}{2\pi\sqrt{5\times10^{-17}}}$$

$$= \frac{1}{2\pi \times 7.07\times10^{-9}} \approx 22.5\ MHz$$

**SRF for M4 (C0G, 100 nF 0402, $L_{ESL} = 0.45\ nH$):**

$$f_{SRF,M4} = \frac{1}{2\pi\sqrt{0.45\times10^{-9} \times 100\times10^{-9}}} = \frac{1}{2\pi \times 6.71\times10^{-9}} \approx 23.7\ MHz$$

The SRFs are very similar (22.5 MHz vs. 23.7 MHz) because the ESL values and capacitance values are nearly identical.

**Voltage coefficient comparison:**

- M3 (X7R): moderate voltage coefficient. At 1.1 V on a 25 V cap (4.4% bias), the capacitance derating is less than 5%. Effectively stable at this rail voltage.
- M4 (C0G): zero voltage coefficient by definition (class I dielectric). Capacitance is perfectly stable across voltage, temperature ($\pm 30\ ppm/°C$), and time.

**ESR comparison at required count:**

$$N_{M3,min} = \frac{50\ m\Omega}{2\ m\Omega} = 25 \quad N_{M4,min} = \frac{100\ m\Omega}{2\ m\Omega} = 50$$

**Cost comparison:**

$$\text{Cost M3} = 25 \times \$0.02 = \$0.50 \quad \text{Cost M4} = 50 \times \$0.05 = \$2.50$$

**Recommendation:** Use M3 (X7R, 100 nF 0402). The voltage coefficient of X7R at 4.4% bias is negligible for this rail, making the main C0G advantage irrelevant. M3 has lower ESR per unit and is 5x cheaper at the required count. C0G would be mandatory if the rail voltage were close to the capacitor's rated voltage — for example, a 3.3 V rail using 6.3 V-rated 100 nF caps (52% bias) where X7R capacitance can drop by 30–40%.

$$\boxed{\text{Select M3 (X7R, 100 nF 0402), use 25 units}}$$

---

### Part (f): Anti-Resonance Check — P1 Bulk vs. M1 MLCC

**Effective ESL of P1 parallel bank ($N_{P1} = 26$):**

$$L_{P1,eff} = \frac{L_{ESL,P1}}{N_{P1}} = \frac{3\ nH}{26} = 0.115\ nH$$

**Total effective capacitance of M1 bank (derated, using $N_{M1} = 3$ initially):**

$$C_{M1,total} = 3 \times 8.5\ \mu F = 25.5\ \mu F$$

**Anti-resonance frequency:**

$$f_{AR} = \frac{1}{2\pi\sqrt{L_{P1,eff} \times C_{M1,total}}} = \frac{1}{2\pi\sqrt{0.115\times10^{-9} \times 25.5\times10^{-6}}}$$

$$= \frac{1}{2\pi\sqrt{2.93\times10^{-15}}} = \frac{1}{2\pi \times 54.2\times10^{-9}} \approx 2.94\ MHz$$

**Anti-resonance peak impedance (lossless approximation):**

$$|Z_{AR}| = \sqrt{\frac{L_{P1,eff}}{C_{M1,total}}} = \sqrt{\frac{0.115\times10^{-9}}{25.5\times10^{-6}}} = \sqrt{4.51\times10^{-6}} \approx 2.12\ m\Omega$$

**Assessment:** $|Z_{AR}| = 2.12\ m\Omega$ is just above $Z_{target} = 2\ m\Omega$. This is a marginal violation that must be addressed.

**Mitigation — increase $N_{M1}$ to 4:**

$$C_{M1,4} = 4 \times 8.5\ \mu F = 34\ \mu F$$

$$|Z_{AR,4}| = \sqrt{\frac{0.115\times10^{-9}}{34\times10^{-6}}} = \sqrt{3.38\times10^{-6}} \approx 1.84\ m\Omega < 2\ m\Omega \quad \checkmark$$

$$\boxed{N_{M1} = 4\ \text{resolves the anti-resonance violation}}$$

---

### Part (g): Final BOM and Placement Strategy

**Bill of Materials:**

| Stage | Part | Count | $f_{SRF}$ | $R_{ESR,total}$ | Coverage |
|---|---|---|---|---|---|
| Bulk | P1 (220 µF polymer) | 26 | ~230 kHz | 0.46 m$\Omega$ | DC to ~300 kHz |
| Mid-freq | M1 (10 µF 0402 X5R) | 4 | 2.06 MHz | 1.25 m$\Omega$ | 500 kHz to 10 MHz |
| High-freq | M3 (100 nF 0402 X7R) | 25 | 22.5 MHz | 2.0 m$\Omega$ | 10 MHz to 100 MHz |

**Placement strategy:**

**P1 Bulk capacitors:** Place within 10–15 mm of the PMIC output, distributed evenly along the DDR5 memory channel. Use polygon copper pour on the VDD plane to minimise trace inductance between the capacitors and the DIMM slot connectors.

**M1 Mid-frequency MLCCs:** Place 1 per DIMM slot side (2 per slot, 4 total for a dual-slot channel), as close as possible to the DIMM connector VDD pins (within 3–5 mm). Use two vias per capacitor.

**M3 High-frequency MLCCs:** Distribute 6–7 around each DIMM slot connector, placed directly adjacent to the connector power and ground pins. Use via-in-pad construction if available. For a back-side placement option, mirror placement under the DIMM slot provides the lowest-inductance path to the connector pins.

**Design verification checklist:**

- [ ] Simulate full PDN with all components from 80 kHz to 300 MHz
- [ ] Verify $|Z_{PDN}(f)| \leq 2\ m\Omega$ across the full band
- [ ] Re-verify anti-resonance between all adjacent stages after applying voltage derating
- [ ] Confirm P1 capacitors at 1.1 V on a 4 V-rated cap: 27.5% bias — acceptable
- [ ] Confirm no high-frequency MLCC is placed more than 8 mm from a DIMM VDD pin

---

## Summary of Key Results

| Parameter | Value |
|---|---|
| $Z_{target}$ | 2 m$\Omega$ |
| Frequency range | 80 kHz to 300 MHz |
| Bulk cap count | 26 × 220 µF polymer |
| Mid-freq MLCC count | 4 × 10 µF 0402 X5R (M1) |
| High-freq MLCC count | 25 × 100 nF 0402 X7R (M3) |
| Anti-resonance frequency | 2.94 MHz |
| Anti-resonance peak (after mitigation) | 1.84 m$\Omega$ ✓ |

---

## Exam Tips

1. **The DDR5 VDD rail has no on-DIMM secondary regulation** in all configurations. Every millivolt of PDN noise is directly visible to the memory receiver. This is why the tolerance is tighter than most digital rails — always state this reasoning in a PDN discussion.

2. **When comparing capacitor case sizes**, the decisive metric for high-frequency performance is ESL, not capacitance. A smaller case always wins for high-frequency bypass, but the ESR disadvantage (smaller cases often have higher ESR) may require more units — always check both constraints.

3. **Voltage coefficient derating is mandatory** for X5R and X7R in any rigorous PDN calculation. Forgetting it leads to over-optimistic impedance estimates. C0G (NP0) has zero voltage coefficient and is immune to this issue — select C0G when the DC bias exceeds roughly 30% of the capacitor's rated voltage.

4. **Anti-resonance peaks between adjacent stages** are the most common cause of PDN failures that pass individual-stage checks. Always compute $f_{AR}$ and $|Z_{AR}|$ for every adjacent stage pair and compare against $Z_{target}$.
