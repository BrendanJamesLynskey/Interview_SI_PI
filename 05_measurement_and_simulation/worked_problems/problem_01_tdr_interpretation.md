# Problem 01: TDR Interpretation

## Problem Statement

You are performing a TDR measurement on a PCB channel to verify impedance continuity before bring-up of a 10 Gbps interface. The TDR instrument has a 50 $\Omega$ reference impedance and launches a step with 35 ps rise time. The incident voltage is 250 mV.

The displayed waveform (voltage at the TDR launch port vs. time) is described by the following sequence of features:

| Time (ps) | Displayed voltage | Feature description |
|---|---|---|
| 0 | 250 mV | Incident step arrives at launch port |
| 0–100 | 250 mV | Flat — well-matched launch section |
| 100 | Ramps down to 235 mV over ~50 ps | Gradual step down beginning at 100 ps |
| 150–800 | 235 mV | Flat plateau |
| 800 | Drops sharply to 195 mV in < 35 ps | Abrupt step down at 800 ps |
| 800–1050 | 195 mV | Flat plateau |
| 1050 | Ramps up to 225 mV over ~100 ps | Slow ramp up beginning at 1050 ps |
| 1250 | 225 mV stable | Final plateau |

**Assume:**
- The PCB dielectric is FR-4 with $\varepsilon_r = 4.0$ (use the nominal value for distance calculations)
- The TDR is calibrated such that the reference impedance at $t = 0$ is 50 $\Omega$ exactly
- All features are well above the TDR's noise floor
- The propagation velocity in the PCB: $v_p = c/\sqrt{\varepsilon_r}$

**Questions:**

1. Calculate the propagation velocity $v_p$ in the FR-4 PCB.
2. For each labelled feature, identify the physical cause (impedance higher/lower than 50 $\Omega$, capacitive discontinuity, inductive discontinuity, short, open) and whether it is a real signal or likely a measurement artefact.
3. Calculate the characteristic impedance of each flat plateau region.
4. Calculate the physical location (distance from the TDR launch) of each impedance event.
5. Identify which features represent acceptable impedance and which require corrective action for a 10 Gbps channel targeting 50 $\Omega$ ± 10%.
6. The feature at 1050 ps ramps up over 100 ps. Estimate the series inductance that would produce this ramp, assuming the surrounding line has $C' = 100$ pF/m and $Z_0 = 50\ \Omega$.

---

## Worked Solution

### Step 1 — Propagation velocity

$$v_p = \frac{c}{\sqrt{\varepsilon_r}} = \frac{3 \times 10^8\ \text{m/s}}{\sqrt{4.0}} = \frac{3 \times 10^8}{2} = 1.5 \times 10^8\ \text{m/s}$$

Equivalently, $v_p = 15\ \text{cm/ns} = 6.67\ \text{ps/cm}$.

---

### Step 2 — Feature identification

Recall that the TDR displays the voltage at the **launch port**. The relationship between the displayed voltage $V_{display}$, the incident voltage $V_i = 250$ mV, and the reflection coefficient $\Gamma$ is:

$$V_{display} = V_i(1 + \Gamma) \implies \Gamma = \frac{V_{display} - V_i}{V_i}$$

**Feature at 100 ps (gradual ramp down to 235 mV over 50 ps):**

The gradual transition (slower than the 35 ps instrument rise time) indicates a distributed or reactive discontinuity, not a step in characteristic impedance.

A ramp **down** indicates a **negative** $\Gamma$ — the impedance seen looking into the line is locally lower than the reference. A gradual reduction is the signature of a **shunt capacitance** (capacitive discontinuity). A capacitor in shunt to ground initially appears as a short (pulls voltage down) then charges to the line voltage and appears as an open, causing the voltage to recover partially — but the plateau at 235 mV suggests the structure is a distributed capacitance (e.g., a wide pad or connector footprint) rather than a discrete lumped capacitor, because the plateau is sustained rather than recovering.

**Verdict: Capacitive discontinuity — likely a connector launch pad or SMD pad.**

**Feature at 800 ps (sharp step down to 195 mV, < 35 ps transition):**

The transition time ($< 35$ ps) is at the instrument's resolution limit — this is a fast impedance change, consistent with a step change in characteristic impedance (not a reactive element). The voltage drops abruptly, indicating a negative $\Gamma$ — the trace impedance from 800 ps onwards is **lower than 50 $\Omega$**.

**Verdict: Step to a lower-impedance trace section.**

**Feature at 1050 ps (gradual ramp up to 225 mV over 100 ps):**

A gradual transition in the **upward** direction after a lower-impedance section indicates a **series inductance**. A series inductor initially appears as a large impedance (causes positive $\Gamma$ briefly), then the current builds up and the effective impedance drops, creating the characteristic slow ramp. The upward ramp is consistent with a via transition, bond wire, or connector pin — structures with significant series loop inductance.

**Verdict: Series inductive discontinuity — likely a via or connector pin.**

**Feature summary:**

| Time | Voltage | $\Gamma$ | Physical interpretation | Verdict |
|---|---|---|---|---|
| 0–100 ps | 250 mV | 0 | 50 $\Omega$ matched launch | Good |
| 100–150 ps | 250 → 235 mV | $-0.06$ | Shunt capacitance (pad/connector) | Investigate |
| 150–800 ps | 235 mV | $-0.06$ | Sub-50 $\Omega$ or loaded trace section | Check |
| 800–1050 ps | 195 mV | $-0.22$ | Lower-impedance trace | Likely fail |
| 1050–1250 ps | 195 → 225 mV | ramp | Series inductance (via or pin) | Investigate |
| 1250 ps+ | 225 mV | $-0.10$ | Sub-50 $\Omega$ termination or load | Check |

---

### Step 3 — Characteristic impedance of each plateau

Using $Z = Z_{ref} \cdot (1+\Gamma)/(1-\Gamma)$ with $Z_{ref} = 50\ \Omega$:

**Plateau 1 (0–100 ps): $V = 250$ mV**

$$\Gamma = \frac{250 - 250}{250} = 0$$

$$Z_1 = 50 \times \frac{1 + 0}{1 - 0} = 50\ \Omega$$

The launch section is exactly 50 $\Omega$.

**Plateau 2 (150–800 ps): $V = 235$ mV**

$$\Gamma = \frac{235 - 250}{250} = \frac{-15}{250} = -0.06$$

$$Z_2 = 50 \times \frac{1 + (-0.06)}{1 - (-0.06)} = 50 \times \frac{0.94}{1.06} = 50 \times 0.887 = 44.3\ \Omega$$

**Plateau 3 (800–1050 ps): $V = 195$ mV**

$$\Gamma = \frac{195 - 250}{250} = \frac{-55}{250} = -0.22$$

$$Z_3 = 50 \times \frac{1 + (-0.22)}{1 - (-0.22)} = 50 \times \frac{0.78}{1.22} = 50 \times 0.639 = 32.0\ \Omega$$

**Plateau 4 (1250 ps+): $V = 225$ mV**

$$\Gamma = \frac{225 - 250}{250} = \frac{-25}{250} = -0.10$$

$$Z_4 = 50 \times \frac{1 + (-0.10)}{1 - (-0.10)} = 50 \times \frac{0.90}{1.10} = 50 \times 0.818 = 40.9\ \Omega$$

**Note on multi-section accuracy:** The impedances of sections 3 and 4 are approximate because the TDR source sees section 3 through the mismatch of section 2. For sections where $|\Gamma|$ is small (< 0.15), the single-stage formula is accurate to a few percent. For section 3 ($\Gamma = -0.22$), a full T-matrix correction would give a more precise answer, but the single-stage estimate is sufficient for triage.

---

### Step 4 — Physical location of each event

TDR time-to-distance conversion uses the round-trip relationship. The event at time $t$ is located at distance:

$$d = \frac{v_p \cdot t}{2}$$

The factor of 2 accounts for the signal travelling to the discontinuity and back.

| Event time | $d = v_p \cdot t / 2$ | Location |
|---|---|---|
| 100 ps | $1.5 \times 10^8 \times 100 \times 10^{-12} / 2 = 0.0075$ m | 7.5 mm from launch |
| 800 ps | $1.5 \times 10^8 \times 800 \times 10^{-12} / 2 = 0.060$ m | 60 mm from launch |
| 1050 ps | $1.5 \times 10^8 \times 1050 \times 10^{-12} / 2 = 0.0788$ m | 78.8 mm from launch |

**Physical interpretation of distances:**

- 7.5 mm: connector or SMA launch pad location. Consistent with the footprint of an SMA edge connector.
- 60 mm: a second impedance transition. This is mid-board — the trace changes impedance here. Could be a layer change via, a routing through a different reference plane region, or a gap in the reference plane.
- 78.8 mm: a series inductive element. Likely a via transitioning from one routing layer to another, or a connector pin.

---

### Step 5 — Compliance assessment for 10 Gbps (50 $\Omega$ ± 10%)

The 50 $\Omega$ ± 10% tolerance corresponds to $Z_0 \in [45\ \Omega, 55\ \Omega]$.

| Section | Calculated $Z_0$ | In 50 $\Omega$ ± 10%? | Action required |
|---|---|---|---|
| Launch (0–100 ps) | 50 $\Omega$ | Yes | None |
| Section 2 (150–800 ps) | 44.3 $\Omega$ | No (below 45 $\Omega$) | Trace too wide or too close to reference plane; adjust trace width |
| Section 3 (800–1050 ps) | 32.0 $\Omega$ | No (well below 45 $\Omega$) | Severe impedance violation; likely a routing through a much higher-$D_k$ region, an over-wide trace, or a poorly designed via transition |
| Section 4 (1250 ps+) | 40.9 $\Omega$ | No (below 45 $\Omega$) | Below tolerance; may be a terminated load with wrong value, or the trace continues into a lower-impedance region |

**The capacitive discontinuity at 7.5 mm** (connector pad) causes a 50 ps transition. For a 10 Gbps signal ($t_{r} \approx 35$–50 ps, UI = 100 ps), a capacitive discontinuity with a 50 ps time constant represents a significant mid-symbol disturbance — likely causing eye closure unless the transmitter pre-emphasis compensates for it.

**Summary:** Only the launch section passes. Sections 2, 3, and 4 all require corrective action. The 32 $\Omega$ section 3 is the most critical fault.

---

### Step 6 — Series inductance estimation from ramp duration

The ramp at 1050 ps rises from 195 mV to 225 mV over 100 ps. This corresponds to a transition from a 32 $\Omega$ section (section 3) to a 40.9 $\Omega$ region (section 4), with the series inductance acting as the transition mechanism.

**Lumped inductor TDR response:**

A series inductor $L$ in a transmission line of impedance $Z_0$ on both sides creates a time-domain voltage step whose rise time is characterised by the L/R time constant of the inductive element:

$$\tau_{inductive} = \frac{L}{Z_0 + Z_0} = \frac{L}{2Z_0}$$

The factor of 2 arises because the inductor is driven from a $Z_0$ source and loaded into $Z_0$ — the Thevenin resistance seen by the inductor is $2Z_0$.

The ramp duration (10–90% rise time of the inductive feature) is approximately $2.2\tau_{inductive}$:

$$t_{ramp} \approx 2.2 \cdot \frac{L}{2Z_0}$$

Using $Z_0 \approx 50\ \Omega$ and $t_{ramp} = 100\ \text{ps}$ (the observed ramp from the inductive feature):

$$L = \frac{t_{ramp} \cdot 2Z_0}{2.2} = \frac{100 \times 10^{-12} \times 100}{2.2} = \frac{10^{-8}}{2.2} \approx 4.5\ \text{nH}$$

**The series inductance is approximately 4.5 nH.**

**Cross-check against physical expectation:**

A PCB via has a series inductance of approximately 0.5–1.5 nH. A connector pin is 1–3 nH. A bond wire is 1–5 nH. A value of 4.5 nH suggests a high-inductance element — possibly a connector pin or a long bond wire with poor grounding return path. This is 3–9x higher than a well-designed via, indicating a packaging or connector problem.

**Impact at 10 Gbps:**

The series inductance forms a high-pass filter with the transmission line capacitance. The 3 dB frequency is:

$$f_{-3dB} = \frac{Z_0}{2\pi L} = \frac{50}{2\pi \times 4.5 \times 10^{-9}} \approx 1.77\ \text{GHz}$$

The Nyquist frequency for 10 Gbps is 5 GHz — well above 1.77 GHz. This inductance will significantly attenuate the high-frequency content of the signal, adding eye closure and increasing jitter. Corrective action is required: backdrill the via stub, use a via-in-pad design, or select a lower-inductance connector footprint.

---

### Common Interview Pitfalls

**Forgetting the factor of 2 in distance calculations:** TDR time represents a round trip. Always divide by 2 when converting time to distance. A common mistake is $d = v_p \cdot t$ instead of $d = v_p \cdot t / 2$.

**Misidentifying gradual vs. abrupt transitions:** An abrupt transition (faster than the TDR rise time) is a change in characteristic impedance. A gradual transition (slower than the rise time) is a reactive element. This distinction is diagnostic — capacitors ramp down, inductors ramp up.

**Applying the single-section impedance formula to multi-section structures:** The formula $Z = Z_{ref}(1+\Gamma)/(1-\Gamma)$ is exact only when looking into a single impedance from a matched source. When multiple sections are present, the display voltage at any point is affected by all preceding sections. For small mismatches ($|\Gamma| < 0.15$), the error is acceptably small; for large mismatches ($|\Gamma| > 0.2$), use T-matrix or chain-matrix de-embedding for accuracy.

**Not checking the resolution limit:** The TDR has a minimum resolvable feature size of $v_p \cdot t_r / 2 = 1.5 \times 10^8 \times 35 \times 10^{-12} / 2 \approx 2.6$ mm. Any feature shorter than 2.6 mm will appear smeared or partially invisible in the TDR display.
