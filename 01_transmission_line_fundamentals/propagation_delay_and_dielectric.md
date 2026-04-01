# Propagation Delay and Dielectric Properties

## Overview

The speed at which a signal travels down a PCB trace is not the speed of light in vacuum. It is determined by the dielectric constant (Dk, or $\varepsilon_r$) of the surrounding material. For a PCB engineer, propagation velocity and delay directly govern whether a trace must be treated as a transmission line, how long delay-matched routes must be, and where inter-symbol interference begins to matter. The dielectric constant also has a frequency dependence (dispersion) that causes pulse broadening and eye closure on long channels.

This document covers propagation velocity, the electrical-length rule of thumb for transmission-line treatment, and the frequency dependence of Dk.

---

## Tier 1: Fundamentals

### Q1. What is the propagation velocity of a signal on a PCB trace, and how does it depend on the dielectric constant?

**Answer:**

The propagation velocity of a TEM (transverse electromagnetic) wave is:

$$v_p = \frac{c}{\sqrt{\varepsilon_{eff}}}$$

where $c = 2.998 \times 10^8$ m/s is the speed of light in vacuum and $\varepsilon_{eff}$ is the effective relative permittivity of the medium surrounding the conductor.

**For stripline** (trace fully embedded in dielectric):

$$\varepsilon_{eff} = \varepsilon_r \quad \Rightarrow \quad v_p = \frac{c}{\sqrt{\varepsilon_r}}$$

**For microstrip** (trace on outer layer, part of the field in air):

$$\varepsilon_{eff} = \frac{\varepsilon_r + 1}{2} + \frac{\varepsilon_r - 1}{2}\left(1 + \frac{12h}{w}\right)^{-1/2} < \varepsilon_r$$

Because $\varepsilon_{eff} < \varepsilon_r$ for microstrip, microstrip signals travel faster than stripline signals in the same material.

**Typical values:**

| Geometry | Material | $\varepsilon_r$ | $\varepsilon_{eff}$ | $v_p$ (m/s) | Delay (ps/inch) |
|---|---|---|---|---|---|
| Microstrip | FR4 | 4.3 | ~3.3 | $1.65 \times 10^8$ | ~154 |
| Stripline | FR4 | 4.3 | 4.3 | $1.44 \times 10^8$ | ~174 |
| Microstrip | Rogers 4350B | 3.48 | ~2.7 | $1.82 \times 10^8$ | ~139 |
| Stripline | Rogers 4350B | 3.48 | 3.48 | $1.60 \times 10^8$ | ~157 |
| Coax (air) | Air | 1.0 | 1.0 | $3.00 \times 10^8$ | ~85 |

**Rule of thumb:** FR4 microstrip — approximately 6 inches per nanosecond (150 ps/inch). FR4 stripline — approximately 5.7 inches per nanosecond (175 ps/inch). These numbers are used constantly in signal integrity calculations.

---

### Q2. What is the "one-sixth rise time rule" for determining when a trace must be treated as a transmission line?

**Answer:**

A trace must be treated as a transmission line — meaning reflections, termination, and wave-propagation effects are significant — when the propagation delay of the trace $T_D$ is a significant fraction of the signal's rise time $t_r$.

The widely used criterion is:

$$\boxed{T_D \geq \frac{t_r}{6}}$$

If this condition is met, the trace is "electrically long" and transmission-line effects are significant. If $T_D < t_r/6$, the trace is "electrically short" and can be treated as a lumped capacitor with negligible reflections.

**Derivation/justification:**

Consider a rising edge travelling onto an unterminated (open) transmission line. The reflected wave returns and adds to the incident wave at the source. The observable ringing at the source will be significant only if the round-trip delay ($2 T_D$) is large enough that the reflected wave arrives after the incident edge has settled. If the round-trip delay is much less than $t_r$, the reflection overlaps with the rising edge and the two superpose smoothly — appearing as a slight slowing of the edge rather than a distinct reflection.

The critical threshold for a "distinct" reflection disturbance is approximately $2 T_D = t_r/3$, giving $T_D = t_r/6$.

**Practical application:**

For a signal with $t_r = 500$ ps (rise time from 20% to 80%):

$$T_D^{max} = \frac{500\ \text{ps}}{6} = 83\ \text{ps}$$

At 154 ps/inch for FR4 microstrip:

$$l_{max} = \frac{83\ \text{ps}}{154\ \text{ps/inch}} \approx 0.54\ \text{inch} \approx 1.4\ \text{cm}$$

Any trace longer than about 0.5 inch at 500 ps rise time requires transmission-line treatment. For a 100 ps rise time (10 Gb/s NRZ), this threshold drops to approximately 0.1 inch (2.5 mm) — meaning virtually every trace on the board must be treated as a transmission line.

**Common mistake:** Applying the $t_r/6$ rule with the clock period instead of the rise time. The relevant parameter is always the rise time, not the data rate. A 1 GHz clock with a 200 ps rise time gives a much shorter threshold length than a naive "$t_r = 1/f_{clk}$" assumption.

---

### Q3. What is the difference between the phase velocity and group velocity of a signal on a transmission line? When do they differ?

**Answer:**

**Phase velocity** $v_\phi$ is the velocity at which a single-frequency (monochromatic) sinusoidal wave travels:

$$v_\phi = \frac{\omega}{\beta}$$

where $\omega$ is the angular frequency and $\beta$ is the phase constant (rad/m).

**Group velocity** $v_g$ is the velocity at which the envelope of a narrowband waveform (a signal with a small spread of frequencies around a carrier) travels:

$$v_g = \frac{d\omega}{d\beta}$$

**When they are equal (ideal, dispersion-free line):**

For a lossless, non-dispersive line: $\beta = \omega/v_p$ where $v_p$ is constant. Then $v_g = d\omega/d\beta = v_p = v_\phi$. Phase and group velocity are identical and frequency-independent.

**When they differ (dispersive line):**

Dispersion occurs when the phase velocity depends on frequency: $\beta(\omega) \neq \omega \times \text{const}$. Sources of dispersion in PCB transmission lines:

1. **Dielectric dispersion:** $\varepsilon_r$ decreases with frequency (see Q5). Lower $\varepsilon_r$ at high frequency means higher $v_\phi$ at high frequency. Different frequency components of a pulse travel at different speeds, causing pulse spreading (inter-symbol interference).

2. **Microstrip dispersion:** For microstrip, $\varepsilon_{eff}$ increases with frequency as the electric field becomes more concentrated in the dielectric (less fringing into air). This creates dispersion that is absent in stripline.

3. **Loss-induced dispersion (low-frequency):** At low frequencies where $R \gg \omega L$, $\beta$ departs from the lossless $\omega \sqrt{LC}$ relationship and the line becomes highly dispersive. This is the regime of telephony cables, not high-speed PCB signals.

**Practical implication:** On a 28 Gb/s PCIe Gen 5 channel, the signal bandwidth spans from DC to approximately 14 GHz. If $v_\phi$ varies by 1–2% across this range, the high-frequency components of a bit transition arrive slightly before the low-frequency components, smearing the edge transition and closing the eye. This is one reason lossy-line simulations in IBIS-AMI or ADS include frequency-dependent Dk models rather than a fixed single value.

---

### Q4. Calculate the propagation delay and determine whether transmission-line treatment is required for the following trace: length $l = 3$ inches, FR4 stripline, $\varepsilon_r = 4.3$, signal rise time $t_r = 800$ ps.

**Answer:**

**Step 1 — Propagation velocity for FR4 stripline:**

$$v_p = \frac{c}{\sqrt{\varepsilon_r}} = \frac{3 \times 10^8}{\sqrt{4.3}} = \frac{3 \times 10^8}{2.074} = 1.446 \times 10^8\ \text{m/s}$$

Converting to inches per second: $1.446 \times 10^8\ \text{m/s} \times 39.37\ \text{in/m} = 5.69 \times 10^9\ \text{in/s}$

Propagation delay per inch: $1/(5.69 \times 10^9\ \text{in/s}) = 175.7\ \text{ps/inch}$

**Step 2 — Total propagation delay:**

$$T_D = 3\ \text{in} \times 175.7\ \text{ps/in} = 527\ \text{ps}$$

**Step 3 — Apply the one-sixth rule:**

Threshold: $t_r/6 = 800/6 = 133\ \text{ps}$

Since $T_D = 527\ \text{ps} \gg 133\ \text{ps}$, the trace is electrically long.

**Conclusion:** This trace must be treated as a transmission line. The ratio $T_D/t_r = 527/800 = 0.66$, meaning the one-way delay is two-thirds of the rise time — well past the threshold. Reflections, termination, and wave propagation effects will significantly affect the waveform. Failure to terminate this trace could produce overshoots of 50–100% of the logic swing.

---

## Tier 2: Intermediate

### Q5. Explain the frequency dependence of dielectric constant (Dk dispersion) in PCB laminates. What causes it, and what is its practical significance for signal integrity?

**Answer:**

**Mechanism — molecular polarisation:**

The dielectric constant $\varepsilon_r$ arises from polarisation of molecules in the electric field: electrons, ions, and dipolar molecules all contribute. Each polarisation mechanism has a characteristic relaxation frequency above which it can no longer follow the oscillating field and its contribution to $\varepsilon_r$ drops.

For PCB laminates (typically epoxy-glass composites):
- The dominant mechanism at microwave frequencies is the orientation polarisation of polar functional groups (hydroxyl -OH and amine groups) in the epoxy resin matrix.
- These groups relax at frequencies in the GHz range, causing $\varepsilon_r$ to decrease as frequency increases from 1 MHz to 10+ GHz.

**Magnitude of dispersion for FR4:**

| Frequency | Typical FR4 Dk |
|---|---|
| 1 MHz | 4.7–4.8 |
| 100 MHz | 4.4–4.5 |
| 1 GHz | 4.2–4.3 |
| 5 GHz | 4.0–4.1 |
| 10 GHz | 3.8–4.0 |

The total variation from 1 MHz to 10 GHz is approximately 15–20% in Dk.

**Kramers-Kronig relation:**

Dk dispersion and dielectric loss (Df) are not independent — they are linked by the Kramers-Kronig relations, which follow from causality. At frequencies where Df is large (high loss), the Dk is falling rapidly. This is the reason high-loss materials (high Df) also show more pronounced Dk dispersion.

**Practical significance:**

1. **Phase velocity dispersion:** Lower $\varepsilon_r$ at high frequency means higher $v_p$ at high frequency. Different spectral components of a signal travel at slightly different velocities. Over a long channel (12–20 inches), this smears edge transitions and degrades the eye.

2. **Impedance calculation errors:** A PCB manufacturer who measures Dk at 1 MHz and uses that value to calculate 50 $\Omega$ trace widths will produce traces that are 1–2 $\Omega$ too high at 5 GHz (because the actual $\varepsilon_r$ at 5 GHz is lower, making $v_p$ higher and $Z_0$ slightly higher). This is typically within tolerance, but for tight-tolerance designs, the manufacturer must use Dk values measured at the relevant frequency.

3. **Differential pair phase mismatch:** If two traces of a differential pair are routed in regions with different Dk (e.g., one passing over a copper pour, one over a void), the phase velocities differ, causing skew between the positive and negative signals — degrading common-mode rejection.

4. **Simulation accuracy:** SPICE models of PCB channels that use a single fixed Dk produce incorrect results at high frequency. Accurate simulation requires a Djordjevic-Sarkar or Wideband Debye model that captures the frequency dependence of both Dk and Df across the signal bandwidth.

---

### Q6. Two signals are routed as a differential pair on a 20-inch stripline using FR4 ($\varepsilon_r = 4.3$ at 100 MHz). The effective Dk at 10 GHz is 4.0. Calculate the timing skew between the high-frequency and low-frequency components of a 10 Gb/s data stream.

**Answer:**

**Step 1 — Propagation velocities:**

At 100 MHz ($\varepsilon_r = 4.3$):

$$v_{p,low} = \frac{c}{\sqrt{4.3}} = \frac{3 \times 10^8}{2.074} = 1.446 \times 10^8\ \text{m/s}$$

At 10 GHz ($\varepsilon_r = 4.0$):

$$v_{p,high} = \frac{c}{\sqrt{4.0}} = \frac{3 \times 10^8}{2.000} = 1.500 \times 10^8\ \text{m/s}$$

**Step 2 — Propagation delays:**

Convert 20 inches to metres: $20 \times 0.0254 = 0.508$ m.

$$T_{D,low} = \frac{l}{v_{p,low}} = \frac{0.508}{1.446 \times 10^8} = 3.513\ \text{ns}$$

$$T_{D,high} = \frac{l}{v_{p,high}} = \frac{0.508}{1.500 \times 10^8} = 3.387\ \text{ns}$$

**Step 3 — Dispersion-induced skew:**

$$\Delta T = T_{D,low} - T_{D,high} = 3.513 - 3.387 = 126\ \text{ps}$$

**Significance:** At 10 Gb/s (NRZ), the bit period is 100 ps. A 126 ps dispersion-induced skew exceeds one full bit period — this would close the eye entirely on a 20-inch FR4 channel without equalisation.

**Practical note:** This calculation explains why 10 Gb/s and higher data rates are not routed on standard FR4 over long distances. Channels beyond 12–15 inches at 10 Gb/s typically require:
- Low-Dk/low-Df materials (Dk more stable with frequency)
- CTLE or DFE equalisation at the receiver
- TX pre-emphasis to pre-distort the signal

---

### Q7. What is the "electrical length" of a transmission line, and how is it expressed in degrees? Why is this unit used in RF design?

**Answer:**

**Electrical length in degrees:**

The electrical length $\theta$ is the phase shift accumulated by a wave travelling the physical length $l$ of the line:

$$\theta = \beta l = \frac{2\pi}{\lambda} l = \frac{2\pi f l}{v_p}\ \text{radians} = \frac{360^\circ f l}{v_p}\ \text{degrees}$$

where $\lambda = v_p/f$ is the wavelength at frequency $f$.

At the resonant frequency where $\theta = 90^\circ = \lambda/4$, the electrical length is also expressed as a $\lambda/4$ transformer.

**Why degrees (or wavelengths) are used in RF:**

In RF and microwave engineering, a transmission line is often intentionally designed to be a specific fraction of a wavelength. The function of the line changes dramatically with electrical length:

- $\theta = 0^\circ$ (electrically short): the line acts as a lumped element (inductor or capacitor depending on termination)
- $\theta = 90^\circ$ ($\lambda/4$): a quarter-wave transformer; an open-circuit load appears as a short circuit at the input and vice versa
- $\theta = 180^\circ$ ($\lambda/2$): a half-wave section; the input impedance equals the load impedance (the line is transparent)

Expressing length in degrees rather than physical units removes frequency from the specification — a $\lambda/4$ matching section at 5 GHz is a completely different physical length than a $\lambda/4$ section at 10 GHz, but both perform the same function at their respective frequencies.

**Digital SI context:** The $t_r/6$ rule is equivalent to saying a trace must be shorter than approximately $\theta = 30^\circ$ at the frequency $f = 0.35/t_r$ (the bandwidth associated with the rise time) to be treated as lumped. For traces that are longer, the wave-propagation picture is necessary.

---

## Tier 3: Advanced

### Q8. A 28 Gb/s PAM-4 channel uses a 15-inch stripline on a low-loss laminate with $\varepsilon_r = 3.5$ at 7 GHz (the Nyquist frequency) and $\varepsilon_r = 3.7$ at 100 MHz. Quantify the inter-symbol interference contribution from dielectric dispersion alone and explain how a causal dielectric model differs from a constant-Dk model in SPICE.

**Answer:**

**Step 1 — Dispersion-induced delay spread:**

Physical length: $15 \times 0.0254 = 0.381$ m.

At 100 MHz ($\varepsilon_r = 3.7$):
$$T_{D,LF} = \frac{0.381}{c/\sqrt{3.7}} = \frac{0.381 \times \sqrt{3.7}}{3 \times 10^8} = \frac{0.381 \times 1.924}{3 \times 10^8} = 2.442\ \text{ns}$$

At 7 GHz ($\varepsilon_r = 3.5$):
$$T_{D,HF} = \frac{0.381 \times \sqrt{3.5}}{3 \times 10^8} = \frac{0.381 \times 1.871}{3 \times 10^8} = 2.375\ \text{ns}$$

$$\Delta T_{disp} = 2.442 - 2.375 = 67\ \text{ps}$$

**Step 2 — ISI significance for PAM-4 at 28 Gb/s:**

For 28 Gb/s PAM-4, the symbol rate is 14 Gbaud (two bits per symbol). The symbol period is:

$$T_{sym} = \frac{1}{14 \times 10^9} \approx 71\ \text{ps}$$

A dispersion-induced pulse spreading of 67 ps is nearly one full symbol period. This means energy from one symbol bleeds into the adjacent symbol at approximately the full amplitude — a severe ISI penalty. The effective eye height at the PAM-4 receiver would be reduced significantly before accounting for conductor loss.

**Step 3 — Causal vs. constant-Dk model:**

A constant-Dk SPICE model assigns a single value of $\varepsilon_r$ (typically at some reference frequency, often 1 GHz) and propagates all frequency components at the same velocity. The result is:
- Correct attenuation profile (if a frequency-dependent $\alpha$ is used)
- Incorrect phase: all spectral components arrive at the same time, so no dispersion ISI

A causal dielectric model (Djordjevic-Sarkar or Wideband Debye) satisfies the Kramers-Kronig relations:

$$\varepsilon_r(\omega) = \varepsilon_\infty + \frac{\sigma_0}{j\omega\varepsilon_0} + \sum_k \frac{\Delta\varepsilon_k}{1 + j\omega/\omega_k}$$

where each pole $\omega_k$ represents a molecular relaxation mechanism. This gives both the correct real part (Dk decreasing with frequency) and the correct imaginary part (Df peaking near each relaxation) simultaneously.

**Practical consequence of using constant Dk in simulation:**

The simulated eye diagram will show:
1. Correct attenuation-induced ISI (if $\alpha(\omega)$ is modelled)
2. Missing dispersion ISI (underestimates total ISI)
3. Systematic phase error in time-domain waveforms

For a 28 Gb/s PAM-4 link at 15 inches on a modern laminate, the dispersion ISI and conductor-loss ISI are comparable in magnitude. Using a constant-Dk model underestimates total ISI by 30–50%, potentially leading to a channel being classified as compliant when it is not. IEEE 802.3 channel compliance standards explicitly require causal, frequency-dependent material models for channel simulation above 10 Gb/s.

---

### Q9. Derive the relationship between the propagation delay per unit length and the per-unit-length capacitance, and use it to explain why a higher-Dk material always produces a slower propagation speed but also provides better coupling between an aggressor and victim trace.

**Answer:**

**Derivation — delay vs. capacitance:**

From the lossless transmission line:

$$v_p = \frac{1}{\sqrt{LC}}$$

Propagation delay per unit length:

$$T_D' = \frac{1}{v_p} = \sqrt{LC}\ \text{ (time per unit length)}$$

From the medium's properties, the phase velocity is also $v_p = c/\sqrt{\varepsilon_{eff}\mu_r}$. For non-magnetic materials ($\mu_r = 1$):

$$v_p = \frac{c}{\sqrt{\varepsilon_{eff}}}$$

The per-unit-length capacitance for a given trace geometry scales with the medium permittivity:

$$C = \varepsilon_{eff} C_{air}$$

where $C_{air}$ is the capacitance the same geometry would have in free space. The per-unit-length inductance $L$ is independent of the dielectric constant (it depends only on the magnetic field geometry, which is unchanged for $\mu_r = 1$).

Therefore:

$$v_p = \frac{1}{\sqrt{LC}} = \frac{1}{\sqrt{L \cdot \varepsilon_{eff} C_{air}}} = \frac{1}{\sqrt{\varepsilon_{eff}}} \cdot \frac{1}{\sqrt{L C_{air}}} = \frac{c}{\sqrt{\varepsilon_{eff}}}$$

This is consistent — the capacitance increase by $\varepsilon_{eff}$ directly explains why higher-Dk materials slow the signal. Since $L$ is unchanged, $Z_0 = \sqrt{L/C} \propto 1/\sqrt{\varepsilon_{eff}}$ also decreases with higher Dk.

**Coupling and crosstalk:**

Crosstalk between an aggressor and victim trace depends on the mutual capacitance $C_m$ and mutual inductance $M$ between them. The mutual capacitance $C_m$ scales with $\varepsilon_{eff}$ in exactly the same way as the self-capacitance. Therefore:

$$NEXT_{voltage} \propto \frac{C_m}{C} + \frac{M}{L} = \text{constant w.r.t. } \varepsilon_{eff}$$

This might suggest that Dk does not affect the crosstalk ratio. However, the situation is more nuanced:

1. **Absolute coupling:** For a fixed aggressor voltage, the induced noise voltage on the victim is proportional to $C_m$. Higher Dk increases $C_m$, increasing the absolute crosstalk voltage.

2. **Trace geometry:** On a high-Dk material, the trace width needed for 50 $\Omega$ is narrower (because the increased $C$ must be offset by a narrower trace). Narrower traces are spaced more closely for the same board density, increasing $C_m/C_{self}$ and therefore the normalised coupling.

3. **Electric field concentration:** Higher Dk concentrates the electric field more tightly in the dielectric layer. The fringing field that would cross to adjacent traces is reduced. This effect partially offsets the coupling increase — it is why well-designed differential pairs on low-loss laminates (lower Dk) can still exhibit good isolation despite using wider traces.

**Summary:** Higher Dk slows propagation (more capacitance per unit length) and can increase coupling between adjacent traces both through higher absolute mutual capacitance and through the geometric adjustments (narrower traces) needed to maintain target impedance.

---

## Quick Reference: Propagation Delay and Dielectric

| Quantity | Formula |
|---|---|
| Propagation velocity | $v_p = c / \sqrt{\varepsilon_{eff}}$ |
| Delay per unit length | $T_D' = \sqrt{L'C'} = \sqrt{\varepsilon_{eff}} / c$ |
| FR4 microstrip delay | ~154 ps/inch (~6 in/ns) |
| FR4 stripline delay | ~175 ps/inch (~5.7 in/ns) |
| Electrical-length rule | Treat as T-line if $T_D \geq t_r/6$ |
| Bandwidth from rise time | $f_{-3dB} \approx 0.35 / t_r$ |
| Electrical length (degrees) | $\theta = 360^\circ f T_D$ |
| $\lambda/4$ resonance length | $l = v_p / (4f)$ |

| FR4 Dk vs. frequency | Approximate value |
|---|---|
| 1 MHz | 4.7–4.8 |
| 1 GHz | 4.2–4.3 |
| 10 GHz | 3.8–4.0 |

| Rule of thumb | Value |
|---|---|
| Electrically short (lumped) | $T_D < t_r/6$ |
| Electrically long (T-line) | $T_D \geq t_r/6$ |
| 28 Gb/s NRZ symbol period | ~36 ps |
| 10 Gb/s NRZ symbol period | ~100 ps |
| 28 Gb/s PAM-4 symbol period | ~71 ps |
