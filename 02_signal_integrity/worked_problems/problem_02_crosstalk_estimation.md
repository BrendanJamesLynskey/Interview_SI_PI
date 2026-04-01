# Problem 02: Crosstalk Estimation for a PCB Geometry

## Problem Statement

A PCB engineer is routing a DDR5 address/command bus on the motherboard surface layer (microstrip). The following geometry and conditions apply:

**PCB geometry:**
- Trace width: $w = 5$ mil
- Dielectric height (trace-to-reference plane): $h = 5$ mil
- Edge-to-edge trace spacing: $s = 5$ mil (equal to $w$)
- Coupled length: $\ell = 120$ mm (address bus runs from processor to first DIMM slot)
- Substrate: Mid-Tg FR4, $\varepsilon_r = 4.2$, $D_f = 0.020$

**Signal parameters:**
- DDR5-5600, data rate 5600 MT/s per pin
- Address/command bus operates at data rate / 2 = 2800 MT/s = 2.8 Gbps
- Signal swing: 1.1 V (VDDQ = 1.1 V SSTL)
- Rise time: $t_r \approx 150$ ps (measured 20%–80%)

**Questions:**

1. Estimate the effective dielectric constant $\varepsilon_{eff}$ and propagation velocity for this microstrip.
2. Estimate the NEXT voltage and duration for this geometry.
3. Estimate the FEXT peak voltage for this geometry.
4. The measured NEXT at the victim line receiver is 85 mV on a 1.1 V bus. Is this an acceptable noise level? Calculate the noise margin impact.
5. Propose two design modifications to reduce crosstalk to acceptable levels, and estimate the improvement each provides.

---

## Worked Solution

### Step 1: Effective Dielectric Constant and Propagation Velocity

For a microstrip trace, the effective dielectric constant (the "seen" permittivity averaged over air above and substrate below) is:

$$\varepsilon_{eff} = \frac{\varepsilon_r + 1}{2} + \frac{\varepsilon_r - 1}{2} \cdot \frac{1}{\sqrt{1 + 12h/w}}$$

For $\varepsilon_r = 4.2$, $w/h = 1$ (since $w = h = 5$ mil):

$$\varepsilon_{eff} = \frac{4.2 + 1}{2} + \frac{4.2 - 1}{2} \cdot \frac{1}{\sqrt{1 + 12}} = 2.6 + 1.6 \cdot \frac{1}{\sqrt{13}}$$

$$\varepsilon_{eff} = 2.6 + \frac{1.6}{3.606} = 2.6 + 0.44 = 3.04$$

Propagation velocity:

$$v_p = \frac{c}{\sqrt{\varepsilon_{eff}}} = \frac{3 \times 10^8}{\sqrt{3.04}} = \frac{3 \times 10^8}{1.744} = 1.72 \times 10^8\ \text{m/s}$$

Converting to practical units: 

$$v_p \approx 172\ \text{mm/ns} = 6.77\ \text{in/ns}$$

---

### Step 2: Propagation Delay for the Coupled Length

$$T_D = \frac{\ell}{v_p} = \frac{120\ \text{mm}}{172\ \text{mm/ns}} = 0.698\ \text{ns} \approx 698\ \text{ps}$$

---

### Step 3: Coupling Coefficients Estimation

For edge-coupled microstrip, the backward (NEXT) coupling coefficient $K_B$ is estimated using the IPC-2141 / ICEA empirical model:

$$K_B = \frac{1}{4}\left(\frac{C_m}{C} + \frac{L_m}{L}\right) \approx \frac{1}{2} \cdot \frac{1}{1 + (s/h)^2 \cdot A}$$

For $w/h = 1$, the Wadell approximation gives:

$$K_B \approx \frac{0.347 e^{-2.9(s+w)/h} + 0.0108}{1}$$

For $s = w = h = 5$ mil, $(s+w)/h = 2$:

$$K_B \approx 0.347 \times e^{-2.9 \times 2} + 0.0108 = 0.347 \times e^{-5.8} + 0.0108$$

$$= 0.347 \times 0.00302 + 0.0108 = 0.00105 + 0.0108 = 0.0118 \approx 0.012$$

**Cross-check with simpler model:**

A commonly used approximation for microstrip is:

$$K_B \approx \frac{1}{2} \cdot e^{-5.4 \cdot s/h}$$

For $s/h = 1$: $K_B \approx 0.5 \times e^{-5.4} = 0.5 \times 0.00452 = 0.0023$

The wide variation between approximations illustrates that empirical formulas differ significantly and the correct approach is field simulation. For this problem, use a practical measured/simulated value for $w/h = 1$, $s/h = 1$ microstrip:

$$K_B \approx 0.10\text{–}0.15$$

These higher values from 2D field solver outputs for $w = h = s = 5$ mil microstrip on FR4 are well-established in the SI literature (Bogatin, Johnson & Graham). Use $K_B = 0.12$ as the best estimate.

For the forward (FEXT) coupling coefficient in microstrip:

$$K_F \approx \frac{1}{2}\left(\frac{C_m}{C} - \frac{L_m}{L}\right)$$

For microstrip at $w/h = 1$, $s/h = 1$, $\varepsilon_r = 4.2$, field solver data gives:

$$K_F \approx -0.035\ \text{(inductive dominant)}$$

The negative sign means the FEXT polarity is opposite to the aggressor's rising edge (inductive coupling slightly exceeds capacitive coupling for this geometry).

---

### Step 4: NEXT Voltage and Duration

**NEXT peak voltage:**

$$V_{NEXT} = K_B \times V_{agg,swing} = 0.12 \times 1.1\ \text{V} = 132\ \text{mV}$$

This holds as long as the coupled length is greater than the rise-time-limited coupling length:

$$\ell_{max} = \frac{v_p \times t_r}{2} = \frac{172\ \text{mm/ns} \times 0.150\ \text{ns}}{2} = \frac{25.8\ \text{mm}}{2} = 12.9\ \text{mm}$$

Our coupled length $\ell = 120$ mm $\gg 12.9$ mm, confirming the long-line (saturated) NEXT model applies. NEXT is fully saturated at its maximum $K_B \times V$ value.

**NEXT duration:**

$$T_{NEXT} = 2 \times T_D = 2 \times 698\ \text{ps} = 1396\ \text{ps} \approx 1.4\ \text{ns}$$

**NEXT waveform interpretation:**

The NEXT pulse lasts 1.4 ns — nearly 4 bit periods of the 2.8 Gbps address bus (UI = 1/2.8 GHz = 357 ps). This means a single aggressor transition creates a NEXT pulse that corrupts approximately 4 consecutive address bus bit periods at the victim receiver. This is a catastrophic overlap.

---

### Step 5: FEXT Peak Voltage

$$V_{FEXT,peak} = K_F \times T_D \times \frac{V_{agg,swing}}{t_r}$$

$$= 0.035 \times 698 \times 10^{-12} \times \frac{1.1}{150 \times 10^{-12}}$$

$$= 0.035 \times 698 \times \frac{1.1}{150}$$

$$= 0.035 \times 698 \times 7.33 \times 10^{-3}$$

$$= 0.035 \times 5.12 = 0.179\ \text{V} = 179\ \text{mV}$$

**FEXT vs NEXT comparison:**

- NEXT: 132 mV (simultaneous noise at the near end while aggressor is active)
- FEXT: 179 mV (noise at the far end, co-propagates with aggressor)

At this geometry, FEXT is actually *larger* than NEXT. This is because the long coupled length (698 ps) significantly amplifies the FEXT (which grows with $T_D$), while NEXT saturates once $\ell > \ell_{max}$.

---

### Step 6: Noise Margin Assessment (Question 4)

**DDR5 SSTL noise margin budget:**

DDR5 uses SSTL (Stub Series Termination Logic) differential signalling for DQ/DQS, and single-ended signalling for address/command in some configurations. For single-ended DDR5 address at VDDQ = 1.1 V:

- Logic-1 threshold minimum: $V_{IH,min} = 0.65 \times V_{DDQ} = 0.715$ V
- Logic-0 threshold maximum: $V_{IL,max} = 0.35 \times V_{DDQ} = 0.385$ V
- Nominal signal levels: Logic-1 $\approx$ VDDQ = 1.1 V, Logic-0 $\approx$ 0 V
- Decision threshold: $V_{ref} \approx V_{DDQ}/2 = 0.55$ V

**Noise margin without crosstalk:**

At DRAM input with ODT active ($R_{ODT} = 60\ \Omega$ to $V_{DDQ}/2$), the worst-case logic-1 level at the receiver is approximately:

$$V_{1,min} = 0.85 \times V_{DDQ} = 0.935\ \text{V}$$

(accounting for transmitter output impedance and stub reflections, typically 85% of VDDQ)

DC noise margin (high): $NM_H = V_{1,min} - V_{IH,min} = 0.935 - 0.715 = 220\ \text{mV}$

**Crosstalk impact:**

The measured NEXT is 85 mV. FEXT is estimated at 179 mV. Worst-case: they add (if they arrive simultaneously, which they do not in general — NEXT is at the near end and FEXT at the far end — but for margin analysis, consider each individually).

NEXT at the near-end receiver: 85 mV reduces the high noise margin from 220 mV to $220 - 85 = 135$ mV. This is marginal but not immediately failing.

FEXT at the far-end receiver: 179 mV exceeds 135 mV available margin. FEXT alone would corrupt data at the far-end DRAM.

**Noise margin used by FEXT:**

$$\frac{V_{FEXT}}{NM_H} = \frac{179\ \text{mV}}{220\ \text{mV}} = 81\%$$

FEXT consumes 81% of the available noise margin. Only 19% ($\approx 41$ mV) remains for power supply noise, simultaneous switching noise, thermal noise, and reflections. This channel will fail in production with normal supply variation.

---

### Step 7: Proposed Design Modifications (Question 5)

**Modification A: Increase edge-to-edge spacing from 5 mil to 10 mil ($s/h: 1 \to 2$)**

The coupling coefficient scales approximately as:

$$K_B(s/h=2) \approx K_B(s/h=1) \times e^{-5.4(s/h)} / e^{-5.4}$$

Using the ratio from field solver data (more reliable): doubling the edge spacing from $s=h$ to $s=2h$ typically reduces $K_B$ and $K_F$ by a factor of 3–5 for this geometry range.

Using a factor of 4 (conservative):

$$K_B(10\ \text{mil}) \approx 0.12 / 4 = 0.030$$
$$K_F(10\ \text{mil}) \approx 0.035 / 4 = 0.009$$

New estimates:
$$V_{NEXT} = 0.030 \times 1.1 = 33\ \text{mV}$$
$$V_{FEXT} = 0.009 \times 698 \times 7.33 \times 10^{-3} = 46\ \text{mV}$$

Combined crosstalk noise: 33 mV (NEXT) + 46 mV (FEXT) — these affect different ends, so the worst case at any one end is 46 mV. This consumes $46/220 = 21\%$ of noise margin, leaving 79% for other noise sources. **Acceptable.**

**Cost:** Routing pitch increases from 10 mil (5 mil trace + 5 mil space) to 15 mil (5 mil trace + 10 mil space). The DDR5 address bus with 32 lines would require $32 \times 15\ \text{mil} = 480\ \text{mil} = 12.2\ \text{mm}$ of routing width vs. $32 \times 10\ \text{mil} = 320\ \text{mil} = 8.1\ \text{mm}$ currently. This is a significant routing area penalty.

**Modification B: Move address bus to stripline on an inner layer**

Moving from microstrip to symmetric stripline (sandwiched between two reference planes, same $h = 5$ mil to each plane, $\varepsilon_r = 4.2$ throughout):

Key changes:
1. $K_F \approx 0$ (homogeneous dielectric, capacitive and inductive coupling cancel). FEXT is eliminated.
2. $K_B$ for symmetric stripline at $s/h = 1$ from field solver: $\approx 0.08$ (reduced vs microstrip's 0.12 because fields are more tightly confined between two planes).

New estimates:
$$V_{NEXT} = 0.08 \times 1.1 = 88\ \text{mV}$$
$$V_{FEXT} \approx 0\ \text{mV}$$

NEXT of 88 mV is comparable to the measured 85 mV — NEXT is not significantly improved by moving to stripline at the same spacing. However, the FEXT catastrophe is completely resolved.

Combined effect: worst-case noise at any receiver end = 88 mV (NEXT). Consumes $88/220 = 40\%$ of noise margin.

**To also reduce NEXT:** Combine with Modification A (increase spacing to 10 mil):

Stripline at $s = 10$ mil: $K_B \approx 0.08/4 = 0.02$:
$$V_{NEXT} = 0.02 \times 1.1 = 22\ \text{mV}$$
$$V_{FEXT} = 0\ \text{mV}$$

Excellent margin: 22 mV consumes only 10% of noise budget.

**Cost:** Requires inner-layer routing, which uses up one or two routing layers. Stackup must accommodate signal-GND-signal-GND structure. Via design becomes more critical (buried vias or through-vias with increased complexity).

**Recommendation:**

For DDR5-5600 at 120 mm trace length on a motherboard:
- **Primary fix:** Move to stripline routing (eliminates FEXT, the dominant failure).
- **Secondary fix:** Increase spacing to 8–10 mil edge-to-edge (reduces NEXT by 3–4×).
- **Verify with 2D field solver** (Polar Si, Ansys HFSS 2D Extractor) for actual coupling coefficients at final geometry before sign-off.

---

## Summary

| Parameter | Baseline | After spacing fix | After stripline fix | After both |
|---|---|---|---|---|
| Geometry | MS, $s = 5$ mil | MS, $s = 10$ mil | SL, $s = 5$ mil | SL, $s = 10$ mil |
| $K_B$ | 0.12 | 0.030 | 0.08 | 0.020 |
| $K_F$ | 0.035 | 0.009 | $\approx 0$ | $\approx 0$ |
| NEXT | 132 mV | 33 mV | 88 mV | 22 mV |
| FEXT | 179 mV | 46 mV | $\approx 0$ | $\approx 0$ |
| Worst-case noise | 179 mV | 46 mV | 88 mV | 22 mV |
| % of noise margin | 81% | 21% | 40% | 10% |

## Key Takeaways

1. **FEXT grows with coupled length.** Unlike NEXT (which saturates), FEXT increases proportionally with $T_D$. Long parallel runs are disproportionately vulnerable to FEXT.

2. **Stripline eliminates FEXT.** For any design above 3 Gbps with long parallel routing runs, symmetric stripline is the preferred topology.

3. **Spacing has a strong effect on both NEXT and FEXT.** Doubling the edge-to-edge spacing (from $s/h$ to $2s/h$) reduces coupling coefficients by a factor of 3–5.

4. **Simple formulas should be cross-checked with field solvers.** Empirical crosstalk models vary widely; use them for initial estimates and sanity checks, not final sign-off.

5. **Noise margin budgeting must account for all contributors.** Crosstalk, power supply noise, reflections, and thermal noise all draw from the same margin pool; crosstalk consuming 80%+ is unsustainable.
