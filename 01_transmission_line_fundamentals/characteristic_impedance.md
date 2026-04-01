# Characteristic Impedance

## Overview

Characteristic impedance is the single most important parameter in high-speed PCB design. Every trace on a PCB that carries a signal faster than roughly 1 ns rise time must be treated as a transmission line, and the behaviour of that line is governed almost entirely by its characteristic impedance and the impedances of the source and load connected to it. Misunderstanding characteristic impedance is the root cause of reflections, overshoot, ringing, and EMI failures in the majority of real-world signal integrity problems.

This document covers the derivation of characteristic impedance from distributed circuit theory, the practical formulas used for microstrip and stripline geometries, and the engineering decisions involved in impedance control on production PCBs.

---

## Tier 1: Fundamentals

### Q1. What is characteristic impedance, and what determines its value?

**Answer:**

Characteristic impedance $Z_0$ is the ratio of voltage to current in a travelling wave on a transmission line under matched (reflection-free) conditions. It is a property of the line's geometry and materials — not of the signal frequency, not of the source or load impedance, and not of the line length.

The fundamental definition comes from the distributed-element model. A uniform transmission line is modelled as an infinite cascade of infinitesimal sections, each containing:

- Series resistance $R$ (conductor loss, $\Omega$/m)
- Series inductance $L$ (magnetic energy storage, H/m)
- Shunt conductance $G$ (dielectric loss, S/m)
- Shunt capacitance $C$ (electric energy storage, F/m)

Solving the telegrapher's equations for a lossless line ($R = 0$, $G = 0$) gives the wave equation with characteristic impedance:

$$Z_0 = \sqrt{\frac{L}{C}}$$

where $L$ and $C$ are the per-unit-length inductance and capacitance respectively (in H/m and F/m).

**Physical intuition:** A wider trace increases $C$ (more plate area, closer to the reference plane) and decreases $L$ (more cross-sectional area for current). Both effects reduce $Z_0$. A thicker dielectric increases $L$ (the magnetic field extends further) and decreases $C$ (larger gap between plates). Both effects increase $Z_0$. The balance between $L$ and $C$ determines whether $Z_0$ is 25 $\Omega$, 50 $\Omega$, or 100 $\Omega$.

**Common mistake:** Assuming $Z_0$ depends on frequency. For a lossless line it does not. For a real lossy line, $Z_0 = \sqrt{(R + j\omega L)/(G + j\omega C)}$ has a weak frequency dependence at low frequency (when $R \gg \omega L$), but at microwave PCB frequencies ($\omega L \gg R$) the imaginary terms dominate and $Z_0 \approx \sqrt{L/C}$ — essentially frequency-independent.

---

### Q2. Derive the general characteristic impedance from the telegrapher's equations. State clearly all assumptions.

**Answer:**

**Starting point — the telegrapher's equations:**

Consider a transmission line element of length $dz$. Applying Kirchhoff's voltage law around the loop:

$$V(z, t) = (R\, dz)\, I(z, t) + (L\, dz)\, \frac{\partial I}{\partial t} + V(z + dz, t)$$

Rearranging:

$$-\frac{\partial V}{\partial z} = R\, I + L\, \frac{\partial I}{\partial t}$$

Applying Kirchhoff's current law at the shunt branch:

$$-\frac{\partial I}{\partial z} = G\, V + C\, \frac{\partial V}{\partial t}$$

**Phasor form** (assuming time-harmonic excitation $e^{j\omega t}$, suppressed):

$$-\frac{dV}{dz} = (R + j\omega L)\, I = Z'\, I$$

$$-\frac{dI}{dz} = (G + j\omega C)\, V = Y'\, V$$

where $Z' = R + j\omega L$ is the series impedance per unit length and $Y' = G + j\omega C$ is the shunt admittance per unit length.

**Wave equation:** Differentiating the first equation with respect to $z$ and substituting the second:

$$\frac{d^2 V}{dz^2} = Z' Y'\, V = \gamma^2\, V$$

where the complex propagation constant is:

$$\gamma = \sqrt{Z' Y'} = \sqrt{(R + j\omega L)(G + j\omega C)} = \alpha + j\beta$$

with $\alpha$ the attenuation constant (Np/m) and $\beta$ the phase constant (rad/m).

**General solution:**

$$V(z) = V^+ e^{-\gamma z} + V^- e^{+\gamma z}$$

$$I(z) = \frac{V^+}{Z_0} e^{-\gamma z} - \frac{V^-}{Z_0} e^{+\gamma z}$$

The characteristic impedance is the ratio $V^+/I^+$ for a forward-travelling wave only:

$$\boxed{Z_0 = \sqrt{\frac{Z'}{Y'}} = \sqrt{\frac{R + j\omega L}{G + j\omega C}}}$$

**Lossless approximation** ($R = 0$, $G = 0$):

$$Z_0 = \sqrt{\frac{j\omega L}{j\omega C}} = \sqrt{\frac{L}{C}}$$

The $j\omega$ terms cancel exactly, confirming $Z_0$ is real and frequency-independent for a lossless line.

**Assumptions stated:**
1. The line is uniform (L, C, R, G constant along length)
2. TEM (transverse electromagnetic) propagation mode — valid for PCB traces well below the cutoff frequency of higher-order modes (typically > 50 GHz for standard traces)
3. Time-harmonic excitation used to obtain the phasor form

---

### Q3. What are the approximate characteristic impedance formulas for a microstrip and for a stripline? What are the key geometric parameters?

**Answer:**

**Microstrip** (trace on the outer layer of a PCB, above a single reference plane):

The most widely used closed-form approximation (IPC-2141A, valid to within 2% for practical geometries) is:

$$Z_0^{\text{MS}} = \frac{87}{\sqrt{\varepsilon_r + 1.41}} \ln\!\left(\frac{5.98\, h}{0.8\, w + t}\right) \quad \Omega$$

where:
- $w$ = trace width (same units as $h$ and $t$)
- $h$ = dielectric height (distance from trace to reference plane)
- $t$ = trace thickness (copper foil thickness)
- $\varepsilon_r$ = relative permittivity (Dk) of the dielectric

**Alternative Hammerstad-Jensen form** (more accurate, used in field solvers):

For $w/h \leq 1$:
$$Z_0 = \frac{60}{\sqrt{\varepsilon_{eff}}} \ln\!\left(\frac{8h}{w} + \frac{w}{4h}\right)$$

For $w/h > 1$:
$$Z_0 = \frac{120\pi}{\sqrt{\varepsilon_{eff}}\, \left[\frac{w}{h} + 1.393 + 0.667\ln\!\left(\frac{w}{h} + 1.444\right)\right]}$$

with effective permittivity:
$$\varepsilon_{eff} = \frac{\varepsilon_r + 1}{2} + \frac{\varepsilon_r - 1}{2}\left(1 + \frac{12h}{w}\right)^{-1/2}$$

Note the microstrip uses $\varepsilon_{eff}$ rather than $\varepsilon_r$ because part of the electric field exists in air above the trace, making the effective medium a mixture of air and dielectric.

**Stripline** (trace buried inside the PCB, centred between two reference planes):

$$Z_0^{\text{SL}} = \frac{60}{\sqrt{\varepsilon_r}} \ln\!\left(\frac{4b}{0.67\pi\, (0.8w + t)}\right) \quad \Omega$$

where:
- $b$ = total dielectric thickness between the two reference planes
- $w$ = trace width
- $t$ = trace thickness
- $\varepsilon_r$ = relative permittivity (no $\varepsilon_{eff}$ correction — the field is entirely in dielectric)

**Key difference between microstrip and stripline:**

| Property | Microstrip | Stripline |
|---|---|---|
| Reference planes | One (below) | Two (above and below) |
| Dielectric exposure | Partial (air above) | Full (all dielectric) |
| Effective Dk | $< \varepsilon_r$ (mixed air/dielectric) | $= \varepsilon_r$ (all dielectric) |
| Propagation velocity | Higher (lower Dk effect) | Lower |
| Dispersion | Moderate (Dk changes with frequency) | Lower |
| Radiation/EMI | More susceptible (open top) | Better shielded |
| Layer use | Outer layers | Inner layers |

---

### Q4. For a target $Z_0 = 50\ \Omega$ microstrip, what trace width is required given $h = 4\ \text{mil}$, $\varepsilon_r = 4.3$, $t = 0.7\ \text{mil}$?

**Answer:**

Using the IPC-2141A formula:

$$50 = \frac{87}{\sqrt{4.3 + 1.41}} \ln\!\left(\frac{5.98 \times 4}{0.8\, w + 0.7}\right)$$

Step 1 — compute the coefficient:

$$\frac{87}{\sqrt{5.71}} = \frac{87}{2.389} = 36.42$$

Step 2 — isolate the logarithm:

$$\ln\!\left(\frac{23.92}{0.8w + 0.7}\right) = \frac{50}{36.42} = 1.373$$

Step 3 — exponentiate:

$$\frac{23.92}{0.8w + 0.7} = e^{1.373} = 3.948$$

Step 4 — solve for $w$:

$$0.8w + 0.7 = \frac{23.92}{3.948} = 6.059$$

$$0.8w = 5.359 \implies \boxed{w \approx 6.7\ \text{mil}}$$

**Sanity check:** For FR4 ($\varepsilon_r \approx 4.3$) at 4 mil dielectric height, a 50 $\Omega$ microstrip trace of roughly 6–8 mil width is a well-known rule of thumb. The result is consistent.

**Practical note:** A PCB fabricator will run a field solver (Polar Si9000, Simberian, or similar) to verify the impedance before fabrication. The closed-form formula gives the starting point; the field solver gives the production value. Typical controlled-impedance tolerance is $\pm 10$%, though $\pm 5$% is achievable with tight stackup control.

---

### Q5. What is the relationship between $Z_0$ and the per-unit-length capacitance in a PCB trace, and why is this useful for measurement?

**Answer:**

From the lossless transmission line definition:

$$Z_0 = \sqrt{\frac{L}{C}} \quad \text{and} \quad v_p = \frac{1}{\sqrt{LC}}$$

Multiplying these two equations:

$$Z_0 \cdot v_p = \frac{1}{C} \implies \boxed{C = \frac{1}{Z_0 \cdot v_p}}$$

Dividing:

$$\frac{Z_0}{v_p} = L \implies \boxed{L = \frac{Z_0}{v_p}}$$

where $v_p = c / \sqrt{\varepsilon_{eff}}$ is the propagation velocity ($c = 3 \times 10^8$ m/s).

**Typical values for a 50 $\Omega$ microstrip on FR4** ($\varepsilon_{eff} \approx 3.3$, $v_p \approx 1.65 \times 10^8$ m/s):

$$C = \frac{1}{50 \times 1.65 \times 10^8} \approx 121\ \text{pF/m} = 3.1\ \text{pF/inch}$$

$$L = \frac{50}{1.65 \times 10^8} \approx 303\ \text{nH/m} = 7.7\ \text{nH/inch}$$

**Why useful:** A TDR (Time Domain Reflectometer) measures the step response of the transmission line. A capacitive discontinuity (e.g., a via pad, test point, or device pin capacitance) produces a negative-going reflection. Knowing that a 50 $\Omega$ line has approximately 3.1 pF/inch, a localised dip in TDR impedance can be converted to an excess capacitance estimate — a standard diagnostic technique.

---

## Tier 2: Intermediate

### Q6. A designer must route a 50 $\Omega$ differential pair. How does differential impedance relate to single-ended impedance, and what coupling effects must be considered?

**Answer:**

**Differential impedance definition:**

Differential impedance $Z_{diff}$ is the impedance measured between the two conductors of a differential pair when they are driven with equal and opposite signals ($+V$ and $-V$). It is not simply $2 \times Z_0$.

For a loosely coupled pair (wide spacing, weak coupling):

$$Z_{diff} \approx 2 Z_0$$

But as the traces approach each other, mutual inductance $M$ increases and mutual capacitance $C_m$ increases. The coupled-line equations give:

$$Z_{diff} = 2\sqrt{\frac{L - M}{C + 2C_m}}$$

where $L$ is the self-inductance per unit length and $M$ is the mutual inductance. Alternatively, using odd-mode impedance $Z_{odd}$:

$$Z_{diff} = 2 Z_{odd}$$

**Even and odd mode:**

A coupled pair supports two propagation modes:

| Mode | Excitation | Impedance |
|---|---|---|
| Even (common) | Both traces same polarity | $Z_{even} = Z_0(1 + k)$ |
| Odd (differential) | Traces opposite polarity | $Z_{odd} = Z_0(1 - k)$ |

where $k$ is the coupling coefficient. For weak coupling $k$ is small, $Z_{odd} \approx Z_0$, and $Z_{diff} \approx 2 Z_0$.

**Effect of trace spacing on $Z_{diff}$:**

Tighter spacing increases $k$, reduces $Z_{odd}$, and therefore reduces $Z_{diff}$ below $2 Z_0$. A designer targeting $Z_{diff} = 100\ \Omega$ with $Z_0 = 50\ \Omega$ single-ended must therefore slightly widen each trace to compensate for the coupling reduction — the exact correction depends on spacing and is calculated by a 2D field solver.

**Practical rule of thumb:** For typical differential pairs (3–4 mil traces, 4–6 mil spacing on inner layers), the coupling correction is approximately 5–10 $\Omega$. A pair that measures 50 $\Omega$ single-ended may only be 90 $\Omega$ differential when tightly coupled.

**Common mistake:** Routing a differential pair at $Z_0 = 50\ \Omega$ per trace and assuming $Z_{diff} = 100\ \Omega$ without checking spacing. If traces are closer than 3W (three times the trace width) edge-to-edge, significant coupling exists and field-solver verification is required.

---

### Q7. Describe the effect of the dielectric constant $\varepsilon_r$ on characteristic impedance. If a designer switches from FR4 ($\varepsilon_r = 4.3$) to a high-speed laminate with $\varepsilon_r = 3.0$, what happens to trace width to maintain $Z_0 = 50\ \Omega$?

**Answer:**

From the microstrip formula, $Z_0 \propto 1/\sqrt{\varepsilon_r}$ (approximately, since the logarithmic term also has a weak dependence through $\varepsilon_{eff}$). Lower $\varepsilon_r$ means lower shunt capacitance per unit length (the electric field couples less efficiently into the dielectric), which raises $Z_0$.

**Quantitative estimate:**

Using the IPC-2141A formula with fixed $h = 4$ mil, $t = 0.7$ mil, for a 50 $\Omega$ target:

For $\varepsilon_r = 4.3$: $w \approx 6.7$ mil (solved in Q4)

For $\varepsilon_r = 3.0$: The coefficient becomes $87/\sqrt{3.0 + 1.41} = 87/2.100 = 41.4$:

$$50 = 41.4 \ln\!\left(\frac{23.92}{0.8w + 0.7}\right)$$

$$\ln\!\left(\frac{23.92}{0.8w + 0.7}\right) = 1.208$$

$$0.8w + 0.7 = \frac{23.92}{e^{1.208}} = \frac{23.92}{3.347} = 7.147$$

$$w = \frac{6.447}{0.8} \approx 8.1\ \text{mil}$$

**Result:** Switching to the lower-Dk material requires a trace width increase from approximately 6.7 mil to 8.1 mil — about 21% wider — to maintain 50 $\Omega$.

**Physical explanation:** Lower $\varepsilon_r$ means less capacitance per unit length. $Z_0 = \sqrt{L/C}$ increases. To bring $Z_0$ back down to 50 $\Omega$, the designer must increase $C$ by widening the trace (more electrode area) or decreasing $h$ (smaller gap). Widening the trace is the more practical option.

**Design implication:** When switching laminate materials mid-project, all controlled-impedance trace widths must be recalculated. A stackup change that reduces $\varepsilon_r$ is not a drop-in replacement — it requires PCB layout updates.

---

### Q8. What is "impedance control" in PCB fabrication, and what process variables does the manufacturer control to achieve it?

**Answer:**

Impedance control is the manufacturing process of achieving a specified characteristic impedance (e.g., 50 $\Omega \pm 10$%) on designated signal layers of a PCB. It is specified in the fabrication drawing and verified by the PCB house using test coupons on the production panel.

**Variables the PCB manufacturer controls:**

1. **Trace width ($w$):** The etch process determines final trace width. Photo-resist exposure, development time, and etch chemistry all affect final width. Typical process variation is $\pm 0.5$ mil; tighter control ($\pm 0.2$ mil) requires fine-line processes.

2. **Dielectric thickness ($h$):** The prepreg and core materials have published Dk and thickness values, but these vary with resin content and press cycle. The manufacturer selects the dielectric build to meet the impedance target. Typical variation is $\pm 5–10$% of nominal.

3. **Copper foil thickness ($t$):** Standard foil is 0.5 oz, 1 oz, or 2 oz (0.7, 1.4, or 2.8 mil). Plating adds copper to via barrels and sometimes to surface layers. The effective trace thickness varies by $\pm 10$%.

4. **Dielectric constant ($\varepsilon_r$):** The laminate supplier provides Dk at a specified frequency (typically 1 MHz per IPC-TM-650). At high frequencies, Dk is lower (frequency dispersion). The manufacturer uses the high-frequency Dk value for impedance calculations.

**Impedance verification:**

PCB houses include impedance test coupons on the production panel — straight traces of known length with the same cross-section as the controlled-impedance nets. These are measured with a TDR before the board is shipped. A typical acceptance criterion is $\pm 10$% of the specified value.

**Tight tolerance ($\pm 5$%):** Requires specifying a specific laminate lot, fine-line etch process, and measurement at multiple coupon locations across the panel. Costs increase significantly.

**Designer's responsibility:** Specify the target impedance, the layers to which it applies, and the reference planes. Do not over-constrain by specifying both trace width and impedance simultaneously — allow the manufacturer to adjust trace width to hit the impedance target given their actual dielectric thickness.

---

## Tier 3: Advanced

### Q9. Derive the per-unit-length capacitance and inductance for a parallel-plate model of a microstrip, and explain why this simple model underestimates $C$ and overestimates $Z_0$.

**Answer:**

**Parallel-plate approximation:**

Model the microstrip as a parallel-plate capacitor with plate width $w$, plate separation $h$, and relative permittivity $\varepsilon_r$. The capacitance per unit length is:

$$C_{pp} = \varepsilon_0 \varepsilon_r \frac{w}{h}$$

Using the same geometry, the inductance per unit length from the parallel-plate inductor (the magnetic energy stored between the plates when current flows along the trace and returns through the plane) is:

$$L_{pp} = \mu_0 \frac{h}{w}$$

The parallel-plate characteristic impedance is therefore:

$$Z_0^{pp} = \sqrt{\frac{L_{pp}}{C_{pp}}} = \sqrt{\frac{\mu_0 h / w}{\varepsilon_0 \varepsilon_r w / h}} = \frac{1}{\sqrt{\varepsilon_r}} \sqrt{\frac{\mu_0}{\varepsilon_0}} \cdot \frac{h}{w} = \frac{377}{\sqrt{\varepsilon_r}} \cdot \frac{h}{w}$$

For $\varepsilon_r = 4.3$, $h/w = 0.6$: $Z_0^{pp} = 377/2.07 \times 0.6 \approx 109\ \Omega$. A field-solver value for the same geometry would give approximately 80 $\Omega$ — significantly lower.

**Why the parallel-plate model is inaccurate:**

1. **Fringing fields:** The electric field does not terminate exclusively in the dielectric below the trace. A substantial portion frings outward from the trace edges into the surrounding medium. For narrow traces ($w/h < 3$), fringing capacitance can be 30–50% of the total. The parallel-plate model ignores this, underestimating $C$ and overestimating $Z_0$.

2. **Mixed dielectric (for microstrip):** The fringing field above the trace passes through air ($\varepsilon_r = 1$) while the field below passes through dielectric ($\varepsilon_r > 1$). The correct effective permittivity $\varepsilon_{eff}$ is a weighted average between 1 and $\varepsilon_r$ depending on the fraction of field in each medium. The parallel-plate model incorrectly applies the full $\varepsilon_r$ everywhere.

3. **Magnetic fringing:** The return current in the reference plane spreads out beyond the trace width. The effective inductance per unit length is therefore lower than the ideal parallel-plate value (the current spreading reduces the enclosed loop area). The parallel-plate model overestimates $L$.

**Net effect:** The parallel-plate model underestimates $C$ and overestimates $L$, giving an overestimate of $Z_0$. For wide traces ($w/h > 10$), fringing is relatively small and the parallel-plate model becomes a reasonable approximation. For typical PCB geometries ($w/h \approx 0.5$–$3$), closed-form corrections (Hammerstad-Jensen) or numerical field solvers are required.

---

### Q10. During a board bring-up, TDR measurement shows the characteristic impedance of a controlled-impedance trace is 42 $\Omega$ instead of the specified 50 $\Omega$. Describe the systematic process for diagnosing the root cause, referencing the impedance formula and fabrication parameters.

**Answer:**

A TDR impedance reading of 42 $\Omega$ on a 50 $\Omega$ specified trace is an 16% low deviation — outside a $\pm 10$% tolerance window. The root cause is a combination of geometric parameters that have shifted the balance of $L/C$.

**Step 1 — Obtain fabrication data:**

Request the as-built stackup from the PCB manufacturer: measured trace width (from coupon cross-section or optical inspection), measured dielectric thickness (from cross-section), copper weight after plating, and the measured Dk of the dielectric lot.

**Step 2 — Apply the sensitivity equations:**

From the IPC-2141A formula $Z_0 = (87/\sqrt{\varepsilon_r + 1.41}) \ln(5.98h / (0.8w + t))$, the first-order sensitivities are:

*Sensitivity to trace width $w$:*
$$\frac{\partial Z_0}{\partial w} = -\frac{87}{\sqrt{\varepsilon_r + 1.41}} \cdot \frac{0.8}{0.8w + t} \cdot \frac{1}{\ln(\ldots)}$$

Increasing $w$ decreases $Z_0$ (more capacitance). A 1 mil increase in $w$ typically changes $Z_0$ by approximately $-2$ to $-3\ \Omega$ for standard 50 $\Omega$ microstrip geometries.

*Sensitivity to dielectric height $h$:*
$$\frac{\partial Z_0}{\partial h} = \frac{87}{\sqrt{\varepsilon_r + 1.41}} \cdot \frac{1}{h}$$

Decreasing $h$ decreases $Z_0$ (more capacitance). A 10% reduction in $h$ changes $Z_0$ by approximately $-3$ to $-5\ \Omega$.

*Sensitivity to $\varepsilon_r$:*
$$\frac{\partial Z_0}{\partial \varepsilon_r} = -\frac{87}{2(\varepsilon_r + 1.41)^{3/2}} \ln(\ldots)$$

Increasing $\varepsilon_r$ decreases $Z_0$.

**Step 3 — Identify likely culprits:**

An 8 $\Omega$ reduction (from 50 to 42) is consistent with:
- Trace width 2–4 mil wider than designed (over-etch compensation gone wrong, or incorrect trace width submitted to fab)
- Dielectric thickness 15–20% thinner than specified (prepreg over-compressed during lamination)
- $\varepsilon_r$ higher than assumed (wrong laminate lot, or Dk measured at 1 MHz used when high-frequency Dk is lower — this would increase $Z_0$, not decrease it, so less likely)

**Step 4 — Cross-section analysis:**

If coupon data is unavailable, request a cross-section from the fabricator. A polished cross-section shows actual trace width, trace thickness, and dielectric height under an optical microscope. This directly identifies whether width or height is the root cause.

**Step 5 — Corrective actions:**

| Root cause | Correction for next build |
|---|---|
| Trace too wide | Reduce drawn trace width; request fab confirmation before run |
| Dielectric too thin | Specify alternate prepreg combination; verify with test coupon |
| Wrong $\varepsilon_r$ | Obtain frequency-dependent Dk data from laminate datasheet; use high-frequency value (typically 1–4 GHz) |
| All within tolerance (stacking of worst cases) | Tighten individual tolerances or accept design at $\pm 15$% |

**Lessons for initial design:** Always design to the manufacturer's nominal stackup parameters, not the laminate datasheet nominal Dk. The laminate and prepreg combination as pressed by a specific fabricator has a different effective Dk than either component alone. Obtain the fabricator's controlled-impedance stackup specification before routing.

---

## Quick Reference: Characteristic Impedance

| Parameter | Typical Value (50 $\Omega$ microstrip, FR4) |
|---|---|
| Trace width $w$ | 6–8 mil (4 mil dielectric) |
| Dielectric height $h$ | 4 mil |
| Copper thickness $t$ | 0.7 mil (0.5 oz) |
| $\varepsilon_r$ (FR4 at 1 GHz) | 4.0–4.5 |
| $\varepsilon_{eff}$ (microstrip) | 3.0–3.5 |
| $C$ per unit length | ~3.1 pF/inch |
| $L$ per unit length | ~7.7 nH/inch |
| Propagation velocity | ~6 in/ns |
| Fabrication tolerance | $\pm 10$% standard, $\pm 5$% tight |

| Formula | Application |
|---|---|
| $Z_0 = \sqrt{L/C}$ | Lossless fundamental definition |
| $Z_0^{MS} = \frac{87}{\sqrt{\varepsilon_r + 1.41}} \ln\!\left(\frac{5.98h}{0.8w + t}\right)$ | Microstrip (IPC-2141A) |
| $Z_0^{SL} = \frac{60}{\sqrt{\varepsilon_r}} \ln\!\left(\frac{4b}{0.67\pi(0.8w + t)}\right)$ | Stripline |
| $Z_{diff} = 2 Z_{odd}$ | Differential pair |
| $C = 1/(Z_0 v_p)$ | PUL capacitance from $Z_0$ |
| $L = Z_0 / v_p$ | PUL inductance from $Z_0$ |
