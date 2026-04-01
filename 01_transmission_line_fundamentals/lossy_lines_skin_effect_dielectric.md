# Lossy Lines: Skin Effect and Dielectric Loss

## Overview

Every real PCB transmission line is lossy. The two dominant mechanisms are conductor loss (due to finite copper conductivity and the skin effect) and dielectric loss (due to the dissipative properties of the laminate material). At data rates above 5 Gb/s, these losses become the primary design constraint and can close the eye completely on long channels without equalisation.

This document covers the physics of each loss mechanism, the quantitative formulas used to estimate and budget them, the figure of merit Df (dissipation factor / loss tangent), the frequency dependence of both mechanisms, and how they combine to produce the total channel insertion loss.

---

## Tier 1: Fundamentals

### Q1. What is skin effect, and how does it cause conductor loss to increase with frequency?

**Answer:**

**Definition:**

At DC, current flows uniformly across the entire cross-section of a conductor. At AC, the self-induced magnetic field inside the conductor creates eddy currents that oppose the penetration of the current into the conductor's interior. The current distribution is not uniform — it is concentrated in a thin shell near the surface. This phenomenon is the skin effect.

**Skin depth:**

The skin depth $\delta_s$ is the characteristic depth at which the current density falls to $1/e$ ($\approx 37$%) of its surface value:

$$\delta_s = \sqrt{\frac{2}{\omega \mu \sigma}} = \sqrt{\frac{1}{\pi f \mu_0 \sigma}}$$

where:
- $f$ = frequency (Hz)
- $\mu_0 = 4\pi \times 10^{-7}$ H/m (permeability of free space; $\mu_r = 1$ for copper)
- $\sigma = 5.8 \times 10^7$ S/m for pure copper (annealed)
- Electrodeposited (ED) copper used in PCBs has $\sigma \approx 5.0$–$5.5 \times 10^7$ S/m

**Typical skin depths for copper:**

| Frequency | Skin depth $\delta_s$ |
|---|---|
| 1 MHz | 66.3 $\mu$m |
| 10 MHz | 21.0 $\mu$m |
| 100 MHz | 6.6 $\mu$m |
| 1 GHz | 2.1 $\mu$m |
| 10 GHz | 0.66 $\mu$m |

A 1 oz copper trace is approximately 35 $\mu$m (1.4 mil) thick. At 10 GHz, the skin depth is only 0.66 $\mu$m — less than 2% of the conductor thickness. Virtually all current flows in a thin shell, and the effective resistance is far higher than the DC value.

**Effect on resistance:**

For a trace of width $w$ and skin depth $\delta_s$, the surface resistance per square is:

$$R_{sq} = \frac{1}{\sigma \delta_s} = \sqrt{\frac{\pi f \mu_0}{\sigma}}\ \Omega/\square$$

The AC resistance per unit length for a trace of width $w$ is approximately:

$$R_{ac}(f) \approx \frac{R_{sq}}{w} = \frac{1}{w} \sqrt{\frac{\pi f \mu_0}{\sigma}} \propto \sqrt{f}$$

**The $\sqrt{f}$ frequency dependence** is the defining signature of skin-effect conductor loss. Since loss $\propto R \propto \sqrt{f}$, conductor loss in dB increases at 10 dB/decade — half the rate of a simple RC roll-off (20 dB/decade).

---

### Q2. What is the dissipation factor (Df, or loss tangent $\tan\delta$), and how does dielectric loss contribute to transmission line attenuation?

**Answer:**

**Definition:**

The dissipation factor $Df$ (also called the loss tangent $\tan\delta$ or $\tan\delta_e$) quantifies the energy dissipated per cycle in a dielectric material relative to the energy stored. It is defined as the ratio of the imaginary to the real part of the complex permittivity:

$$Df = \tan\delta = \frac{\varepsilon''}{\varepsilon'}$$

where the complex permittivity is $\varepsilon = \varepsilon' - j\varepsilon''$ and $\varepsilon' = \varepsilon_r \varepsilon_0$ is the real (energy-storing) part.

**Attenuation from dielectric loss:**

The dielectric loss contribution to the attenuation constant $\alpha_d$ is:

$$\alpha_d = \frac{\pi f \sqrt{\varepsilon_{eff}}}{c} \cdot Df \quad \text{(Np/m)}$$

Converting to dB/m (multiply by $20/\ln 10 = 8.686$):

$$\alpha_d\ [\text{dB/m}] = 8.686 \cdot \frac{\pi f \sqrt{\varepsilon_{eff}}}{c} \cdot Df$$

**Key property:** Dielectric loss increases linearly with frequency (not $\sqrt{f}$ as for conductor loss). At high frequency, dielectric loss dominates over conductor loss.

**Typical Df values for PCB laminates:**

| Material | Df at 10 GHz | Classification |
|---|---|---|
| FR4 standard | 0.020–0.025 | High loss |
| FR4 low-loss (e.g., Panasonic Megtron 6) | 0.004–0.005 | Medium loss |
| Rogers 4350B | 0.0037 | Low loss |
| Isola I-Tera MT40 | 0.0030–0.0035 | Low loss |
| PTFE (Teflon, e.g., Rogers 5880) | 0.0009 | Ultra-low loss |

For a 10-inch stripline at 10 GHz:
- FR4 ($Df = 0.022$, $\varepsilon_r = 4.0$): $\alpha_d = 8.686 \times \pi \times 10^{10} \times 2 / (3 \times 10^8) \times 0.022 \times 0.0254 \approx 12$ dB — catastrophic
- Rogers 4350B ($Df = 0.0037$): $\alpha_d \approx 2$ dB — manageable

This shows why FR4 is unsuitable for 10+ Gb/s channels beyond 6–8 inches without equalization, while premium laminates extend reach to 20+ inches.

---

### Q3. What is the total attenuation of a transmission line, and how do conductor and dielectric losses add?

**Answer:**

The total attenuation constant $\alpha_{total}$ is the sum of the conductor and dielectric contributions:

$$\alpha_{total} = \alpha_c + \alpha_d$$

**Conductor loss** (skin effect dominated, for a microstrip):

$$\alpha_c = \frac{R_{sq}}{Z_0 w}\ \text{(Np/m)} \quad \propto \sqrt{f}$$

A commonly used approximation for a microstrip:

$$\alpha_c\ [\text{dB/inch}] \approx \frac{0.1}{Z_0 w[\text{mil}]} \sqrt{f[\text{GHz}]}$$

**Dielectric loss:**

$$\alpha_d\ [\text{dB/inch}] \approx 2.3 \sqrt{\varepsilon_r} \cdot Df \cdot f[\text{GHz}]$$

(for stripline; the coefficient differs slightly for microstrip due to $\varepsilon_{eff}$)

**Total insertion loss per unit length:**

$$\text{IL}(f)\ [\text{dB/inch}] = \alpha_c \sqrt{f} + \alpha_d f$$

At low frequency, $\alpha_c \sqrt{f}$ dominates. At high frequency, $\alpha_d f$ dominates. The crossover frequency depends on the material: for FR4, dielectric loss dominates above approximately 2–3 GHz; for low-loss laminates, the crossover is at 5–10 GHz.

**Example — 50 $\Omega$ FR4 stripline, 6-mil trace, $\varepsilon_r = 4.3$, $Df = 0.022$:**

At 5 GHz:
$$\alpha_c \approx \frac{0.1}{50 \times 6} \sqrt{5} = \frac{0.1 \times 2.236}{300} \approx 0.00075\ \text{dB/inch}\ \times \text{length scaling factor}$$

Practical values: at 5 GHz, a typical 50 $\Omega$ FR4 stripline has approximately 0.8–1.0 dB/inch total loss. At 10 GHz, this rises to 1.8–2.2 dB/inch. A 10-inch channel at 10 GHz would have 18–22 dB insertion loss — far beyond what most receivers can tolerate without equalization.

---

## Tier 2: Intermediate

### Q4. Derive the surface impedance of copper at high frequency and show that the conductor loss increases as $\sqrt{f}$.

**Answer:**

**Surface impedance definition:**

The surface impedance $Z_s$ relates the tangential electric field $E_{tan}$ at the conductor surface to the surface current density $J_s$ (current per unit width flowing in the surface layer):

$$Z_s = \frac{E_{tan}}{J_s} = R_s + jX_s\ (\Omega/\square)$$

**Derivation from Maxwell's equations:**

Inside a good conductor, the Helmholtz equation gives:

$$\frac{d^2 E_x}{dy^2} = j\omega\mu\sigma E_x + (j\omega)^2\mu\varepsilon E_x \approx j\omega\mu\sigma E_x$$

(The displacement current $j\omega\varepsilon$ term is negligible for good conductors where $\sigma \gg \omega\varepsilon$.)

The solution decaying into the conductor (in the $+y$ direction) is:

$$E_x(y) = E_0 e^{-y/\delta_s} e^{-jy/\delta_s} = E_0 e^{-(1+j)y/\delta_s}$$

The surface current density is found by integrating from the surface to infinity:

$$J_s = \int_0^\infty J_x(y)\, dy = \int_0^\infty \sigma E_x(y)\, dy = \sigma E_0 \int_0^\infty e^{-(1+j)y/\delta_s} dy = \frac{\sigma E_0 \delta_s}{1+j}$$

Therefore:

$$Z_s = \frac{E_0}{J_s} = \frac{1+j}{\sigma \delta_s}$$

Substituting $\delta_s = \sqrt{2/(\omega\mu_0\sigma)}$:

$$Z_s = \frac{1+j}{\sigma} \cdot \sqrt{\frac{\omega\mu_0\sigma}{2}} = \frac{(1+j)}{\sqrt{2}} \sqrt{\frac{\omega\mu_0}{\sigma}} = \sqrt{\frac{j\omega\mu_0}{\sigma}}$$

The real part (surface resistance) is:

$$R_s = \text{Re}(Z_s) = \frac{1}{\sigma \delta_s} = \sqrt{\frac{\pi f \mu_0}{\sigma}} \propto \sqrt{f}$$

**The resistance per square of the conductor surface is proportional to $\sqrt{f}$, confirming that conductor loss increases as $\sqrt{f}$.** Since attenuation $\alpha_c = R'/(2Z_0)$ where $R'$ is the resistance per unit length:

$$\alpha_c \propto R_s \propto \sqrt{f}$$

**Equal real and imaginary parts:** Note that $Z_s = R_s(1+j)$, meaning $X_s = R_s$. The reactive part of the surface impedance is equal to the resistive part. This extra inductance is the internal inductance of the conductor — it contributes a frequency-dependent phase shift that is important for accurate time-domain modelling of lossy interconnects.

---

### Q5. Explain the concept of surface roughness and its effect on conductor loss at high frequency. What is the Hammerstad-Jensen roughness correction factor?

**Answer:**

**Why surface roughness matters:**

At low frequency, the skin depth is many micrometres and the conductor surface appears smooth to the electromagnetic field — the current flows uniformly regardless of small surface irregularities.

At high frequency (e.g., 10 GHz, $\delta_s = 0.66\ \mu$m), the skin depth is smaller than the typical peak-to-valley surface roughness of PCB copper foils. For standard electrodeposited (ED) copper:
- Standard grade: $R_z$ (10-point roughness) = 6–10 $\mu$m
- High-density-interconnect (HDI) copper: $R_z$ = 1–3 $\mu$m
- Very smooth (HVLP — Hyper Very Low Profile): $R_z < 1\ \mu$m

When $\delta_s \ll R_z$, the current must follow the valley-and-peak contour of the rough surface rather than taking a straight path. The effective path length is longer, increasing the resistance beyond the smooth-conductor value.

**Hammerstad-Jensen correction factor:**

The roughness correction multiplier applied to the smooth-surface resistance $R_s$ is:

$$K_H = 1 + \frac{2}{\pi} \arctan\!\left[1.4 \left(\frac{\Delta}{\delta_s}\right)^2\right]$$

where $\Delta$ is the RMS surface roughness.

**Behaviour:**
- At low frequency ($\delta_s \gg \Delta$): $K_H \approx 1$ (no correction)
- At high frequency ($\delta_s \ll \Delta$): $K_H \to 2$ (resistance doubles — the rough surface roughly doubles the effective path length)

**Practical implication:**

For standard ED copper with $\Delta \approx 1.5\ \mu$m:

At 10 GHz ($\delta_s = 0.66\ \mu$m): $(1.5/0.66)^2 = 5.17$

$$K_H = 1 + \frac{2}{\pi} \arctan(1.4 \times 5.17) = 1 + \frac{2}{\pi} \arctan(7.24) = 1 + \frac{2}{\pi}(1.43) = 1 + 0.91 = 1.91$$

Surface roughness nearly doubles the conductor loss at 10 GHz on standard copper. Using ultra-smooth copper (HVLP, $\Delta \approx 0.3\ \mu$m) at 10 GHz:

$(0.3/0.66)^2 = 0.206$

$$K_H = 1 + \frac{2}{\pi} \arctan(0.289) = 1 + \frac{2}{\pi}(0.282) = 1 + 0.18 = 1.18$$

Only 18% additional loss — a significant improvement. This is why premium laminate materials specify smooth or very smooth copper as part of the low-loss solution; the conductor roughness correction is often comparable to the intrinsic conductor loss itself.

---

### Q6. How do skin effect and dielectric loss each scale with frequency, and what does this imply for the shape of a channel's insertion loss profile?

**Answer:**

**Frequency scaling:**

| Mechanism | Attenuation scaling | dB/decade slope |
|---|---|---|
| Conductor loss (skin effect) | $\alpha_c \propto \sqrt{f}$ | 10 dB/decade |
| Dielectric loss | $\alpha_d \propto f$ | 20 dB/decade |
| Combined (high-f dominated by dielectric) | $\propto f$ | 20 dB/decade |

**Shape of insertion loss $|S_{21}|$ vs. frequency:**

At low frequency: conductor loss dominates, and the slope of $|S_{21}|$ in dB is approximately 10 dB/decade (sloping at $\sqrt{f}$, i.e., 10 dB for a 10x increase in frequency).

At high frequency: dielectric loss dominates, and the slope steepens to approximately 20 dB/decade.

The transition frequency $f_{cross}$ where the two contributions are equal is material-dependent:

$$\alpha_c\sqrt{f_{cross}} = \alpha_d f_{cross}$$

$$f_{cross} = \left(\frac{\alpha_c}{\alpha_d}\right)^2$$

For FR4 ($Df = 0.022$): $f_{cross} \approx 1$–$3$ GHz.
For Rogers 4350B ($Df = 0.0037$): $f_{cross} \approx 5$–$10$ GHz.

**Implications for signal integrity:**

1. **Low-loss materials shift the crossover to higher frequency,** keeping dielectric loss manageable across a wider bandwidth and making equalisation easier.

2. **Pre-emphasis/CTLE equalisation boosts high frequencies.** The equalizer must compensate the insertion loss slope. If the slope is 10 dB/decade (conductor-dominated), a single-pole boost is adequate. If the slope is 20 dB/decade (dielectric-dominated), a steeper equalizer is required — potentially consuming more power and generating more noise amplification.

3. **PAM-4 is more sensitive than NRZ.** PAM-4 has four amplitude levels with only 1/3 the spacing between adjacent levels compared to NRZ. An uncompensated channel loss of 3 dB closes the PAM-4 eye by approximately 9 dB (3 dB per amplitude and 3 levels), making equalisation accuracy critical.

4. **Roughness makes the high-frequency conductor loss slope steeper.** The Hammerstad roughness correction factor saturates at 2× for $\delta_s \ll \Delta$, meaning the roughness contribution adds to conductor loss with a slope steeper than $\sqrt{f}$ in the transition region. This creates an intermediate region between $\sqrt{f}$ and $f$ slopes that requires accurate modelling.

---

## Tier 3: Advanced

### Q7. A signal integrity engineer is comparing two PCB laminates for a 28 Gb/s NRZ channel that is 15 inches long. Laminate A is standard FR4: $\varepsilon_r = 4.3$, $Df = 0.022$. Laminate B is a low-loss material: $\varepsilon_r = 3.6$, $Df = 0.004$. Estimate the insertion loss at the Nyquist frequency (14 GHz) for each material and determine whether either channel is viable without equalization (assuming a receiver tolerance of 20 dB).

**Answer:**

**Approach:** Use the approximation formula for stripline insertion loss:

$$\text{IL}(f)\ [\text{dB/inch}] \approx A_c \sqrt{f[\text{GHz}]} + A_d \cdot \varepsilon_r^{1/2} \cdot Df \cdot f[\text{GHz}]$$

For a 50 $\Omega$ stripline with 6-mil trace width and 1 oz copper (approximate values):
- Conductor coefficient: $A_c \approx 0.06$–$0.08$ dB/inch/$\sqrt{\text{GHz}}$
- Dielectric coefficient: $A_d \approx 9.1$ (derived from $2.3 \sqrt{\varepsilon_r} Df f$ formula)

**Laminate A — FR4:**

Conductor loss at 14 GHz:
$$\alpha_c = 0.07 \times \sqrt{14} = 0.07 \times 3.742 = 0.262\ \text{dB/inch}$$

Dielectric loss at 14 GHz:
$$\alpha_d = 2.3 \times \sqrt{4.3} \times 0.022 \times 14 = 2.3 \times 2.074 \times 0.022 \times 14 = 1.46\ \text{dB/inch}$$

Total per inch: $0.262 + 1.46 = 1.72\ \text{dB/inch}$

Total for 15 inches: $15 \times 1.72 = \mathbf{25.8\ \text{dB}}$

**Laminate B — Low-loss:**

Conductor loss at 14 GHz: same trace geometry, $\alpha_c = 0.262\ \text{dB/inch}$

Dielectric loss at 14 GHz:
$$\alpha_d = 2.3 \times \sqrt{3.6} \times 0.004 \times 14 = 2.3 \times 1.897 \times 0.004 \times 14 = 0.245\ \text{dB/inch}$$

Total per inch: $0.262 + 0.245 = 0.507\ \text{dB/inch}$

Total for 15 inches: $15 \times 0.507 = \mathbf{7.6\ \text{dB}}$

**Verdict:**

| | Insertion loss at 14 GHz (15-inch channel) | Viable without EQ? |
|---|---|---|
| FR4 | 25.8 dB | No ($> 20$ dB limit) |
| Low-loss laminate | 7.6 dB | Yes (12.4 dB margin) |

FR4 exceeds the 20 dB receiver limit by nearly 6 dB — this channel cannot be equalised to a functional state without amplification (which introduces noise). The low-loss material achieves the same reach with approximately 12 dB of equalization headroom.

**Additional note:** These estimates are approximate. A real channel budget must include:
- Connector and via discontinuity losses (1–3 dB per connector pair)
- Package loss (1–2 dB per end)
- Copper roughness correction (increases conductor loss by up to 1.8× on standard ED copper)
- Mating connector return loss (contributes additional insertion loss through multi-path reflections)

Including these, the practical insertion loss budgets would be roughly 5–8 dB worse than the trace-only estimates above.

---

### Q8. Describe the Djordjevic-Sarkar (DS) dielectric model and explain why it is preferred over a constant-Dk/Df model for wideband channel simulation.

**Answer:**

**Motivation:**

The dielectric constant and dissipation factor of PCB laminates are not constant — they vary with frequency due to molecular polarisation mechanisms. A simulation model that uses a single fixed $\varepsilon_r$ and $Df$ value (e.g., from the datasheet at 1 GHz) will be accurate only near that frequency. At the Nyquist frequency of a 28 Gb/s channel (14 GHz), the actual $\varepsilon_r$ may be 5–10% lower, and $Df$ may be different. The errors in the simulated $S_{21}$ can be 2–4 dB.

**More critically,** a frequency-independent $\varepsilon_r$ model violates causality. A system with a non-causal impulse response cannot be physically realised and will produce incorrect time-domain results — the simulated pulse will appear to arrive before it is launched.

**Djordjevic-Sarkar model:**

The DS model approximates the frequency-dependent permittivity using a sum of relaxation poles distributed over a wide frequency range:

$$\varepsilon_r(\omega) = \varepsilon_\infty + \frac{\Delta\varepsilon}{\ln(\omega_2/\omega_1)} \ln\!\left(\frac{\omega_2 + j\omega}{\omega_1 + j\omega}\right)$$

where:
- $\varepsilon_\infty$ is the permittivity at infinite frequency
- $\Delta\varepsilon = \varepsilon_{r,DC} - \varepsilon_\infty$ is the total dispersion
- $\omega_1$ and $\omega_2$ are the lower and upper corner frequencies of the relaxation distribution (typically $2\pi \times 10^3$ and $2\pi \times 10^{12}$ rad/s for PCB materials)

**Properties of the DS model:**

1. **Causal:** The Kramers-Kronig relations are satisfied by construction. The model has correct time-domain behaviour.

2. **Broadband accuracy:** The model is fit to measured $\varepsilon_r(f)$ and $Df(f)$ data across the full bandwidth of interest (typically 100 MHz to 50 GHz). It captures the known trend of decreasing $\varepsilon_r$ with frequency and the peaked $Df$ response.

3. **Single reference point:** The model can be parameterised from a single $(\varepsilon_r, Df)$ measurement pair at one frequency, extrapolating the trend across frequency based on the assumed relaxation distribution. This is convenient since datasheets often list values at only one or two frequencies.

4. **Industry adoption:** The Djordjevic-Sarkar model is implemented in all major SI simulation tools (ADS, HyperLynx, CST, Cadence AWR) and is specified as the required dielectric model in IEEE 802.3 (Ethernet) and PCIe compliance channel simulations above 10 Gb/s.

**Comparison with Wideband Debye:**

The Wideband Debye model is an alternative with explicit pole-residue form that can be implemented directly in SPICE. It is more flexible (can fit more complex frequency responses) but harder to parameterise from limited datasheet information. For most PCB laminate modelling, DS and Wideband Debye give equivalent accuracy; DS is more commonly used in PCB-specific tools.

**Practical impact:** Using a constant-Dk model vs. DS model for a 28 Gb/s, 15-inch channel simulation can produce:
- Up to 2–3 dB error in $|S_{21}|$ at 14 GHz
- 20–50 ps timing error in the pulse response peak location
- Incorrect eye diagram opening height, potentially leading to false pass/fail decisions in compliance simulation

---

### Q9. A PCB engineer measures insertion loss of a 10-inch stripline at several frequencies and obtains the following data:

| Frequency (GHz) | $|S_{21}|$ (dB) |
|---|---|
| 1 | −1.8 |
| 4 | −4.2 |
| 9 | −7.5 |
| 16 | −14.0 |

**Separate the conductor and dielectric loss contributions and determine $Df$ of the laminate (assume $\varepsilon_r = 4.0$, copper roughness correction $K_H = 1.5$ at all frequencies for simplicity).**

**Answer:**

**Model:**

Total loss (dB): $\text{IL}(f) = A_c \sqrt{f} + A_d f$

where $A_c$ and $A_d$ are the per-inch conductor and dielectric coefficients times the 10-inch length:

$$\text{IL}_{total}(f) = (A_c \times 10) \sqrt{f} + (A_d \times 10) f$$

Let $C_1 = A_c \times 10$ and $C_2 = A_d \times 10$.

**Step 1 — Fit the model:**

Use two data points to extract $C_1$ and $C_2$. Use $f = 1$ GHz and $f = 9$ GHz (widely spaced for better condition number):

At $f = 1$ GHz: $1.8 = C_1 \times 1 + C_2 \times 1 \implies C_1 + C_2 = 1.8$ ... (i)

At $f = 9$ GHz: $7.5 = C_1 \times 3 + C_2 \times 9$ ... (ii)

From (i): $C_1 = 1.8 - C_2$. Substitute into (ii):

$$7.5 = 3(1.8 - C_2) + 9 C_2 = 5.4 - 3C_2 + 9C_2 = 5.4 + 6C_2$$

$$6C_2 = 2.1 \implies C_2 = 0.35\ \text{dB}/\text{GHz (for 10 inches)}$$

$$C_1 = 1.8 - 0.35 = 1.45\ \text{dB}/\sqrt{\text{GHz}} \text{ (for 10 inches)}$$

**Step 2 — Verify with remaining data points:**

At $f = 4$ GHz: $\text{IL} = 1.45 \times 2 + 0.35 \times 4 = 2.90 + 1.40 = 4.30$ dB. Measured: 4.2 dB. Error: 0.1 dB. Good.

At $f = 16$ GHz: $\text{IL} = 1.45 \times 4 + 0.35 \times 16 = 5.80 + 5.60 = 11.40$ dB. Measured: 14.0 dB. Error: 2.6 dB.

The discrepancy at 16 GHz suggests additional loss (possibly higher roughness correction or dielectric dispersion at high frequency). A better fit across all four points would use a least-squares regression.

**Step 3 — Extract $Df$ from $C_2$:**

The dielectric loss formula per inch for stripline:

$$\alpha_d\ [\text{dB/inch}] = 2.3 \sqrt{\varepsilon_r} \cdot Df \cdot f[\text{GHz}]$$

Per 10 inches: $C_2 = 10 \times 2.3 \times \sqrt{4.0} \times Df = 10 \times 2.3 \times 2 \times Df = 46\ Df$

$$Df = \frac{C_2}{46} = \frac{0.35}{46} \approx 0.0076$$

**Conclusion:** The extracted $Df \approx 0.0076$ is consistent with a mid-grade laminate (better than standard FR4 at 0.020–0.022, worse than Rogers 4350B at 0.0037). The material is likely a moderate-loss laminate such as Isola FR408HR or similar.

**Cross-check conductor loss:** $C_1 = 1.45$ dB/$\sqrt{\text{GHz}}$ for 10 inches = 0.145 dB/inch/$\sqrt{\text{GHz}}$. For a 50 $\Omega$ line with roughness $K_H = 1.5$:

$$\alpha_c = K_H \times \frac{0.07}{\sqrt{Z_0 \times w[\text{mil}]}} \approx 1.5 \times 0.07/\sqrt{50 \times 6} \approx 0.06\ \text{dB/inch}/\sqrt{\text{GHz}}$$

The extracted value of 0.145 is somewhat higher, suggesting the trace may be narrower or rougher than assumed. In a real measurement, copper roughness and trace geometry would be extracted from a cross-section to improve accuracy.

---

## Quick Reference: Lossy Lines

| Quantity | Formula |
|---|---|
| Skin depth | $\delta_s = \sqrt{1/(\pi f \mu_0 \sigma)}$ |
| Surface resistance | $R_s = 1/(\sigma \delta_s) = \sqrt{\pi f \mu_0 / \sigma}$ |
| Copper $\sigma$ | $5.8 \times 10^7$ S/m (annealed); $\approx 5.0$–$5.5 \times 10^7$ S/m (ED) |
| Cu skin depth at 1 GHz | 2.1 $\mu$m |
| Cu skin depth at 10 GHz | 0.66 $\mu$m |
| Conductor loss scaling | $\alpha_c \propto \sqrt{f}$ (10 dB/decade) |
| Dielectric loss | $\alpha_d = (2.3\sqrt{\varepsilon_r} \cdot Df \cdot f)$ dB/inch |
| Dielectric loss scaling | $\alpha_d \propto f$ (20 dB/decade) |
| Total attenuation | $\alpha_{total} = \alpha_c + \alpha_d$ |
| Hammerstad roughness | $K_H = 1 + (2/\pi)\arctan[1.4(\Delta/\delta_s)^2]$; max value 2 |
| Dissipation factor | $Df = \tan\delta = \varepsilon''/\varepsilon'$ |

| FR4 loss summary | Value |
|---|---|
| Dk at 1 GHz | 4.2–4.3 |
| Df at 1 GHz | 0.018–0.022 |
| Insertion loss at 5 GHz | 0.8–1.0 dB/inch |
| Insertion loss at 10 GHz | 1.8–2.2 dB/inch |
| Maximum reach at 10 Gb/s | 6–8 inches (with EQ) |

| Low-loss laminate (e.g., Rogers 4350B) | Value |
|---|---|
| Dk at 10 GHz | 3.48 |
| Df at 10 GHz | 0.0037 |
| Insertion loss at 10 GHz | ~0.4–0.5 dB/inch |
| Maximum reach at 28 Gb/s NRZ | 15–20 inches (with EQ) |
