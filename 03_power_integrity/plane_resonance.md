# Plane Resonance

## Overview

Power and ground planes are not passive, uniform conductors. At high frequencies they behave as parallel-plate transmission structures that support resonant electromagnetic modes. When the PDN excites these modes — through the switching currents of ICs placed on the board — the plane pair can exhibit sharp impedance peaks at resonant frequencies, causing large localised voltage deviations that are invisible to a designer who thinks only in terms of lumped capacitance. This document covers the physics of plane capacitance, the cavity resonance model, resonant frequency calculation, and the techniques used to suppress or avoid resonance effects.

---

## Tier 1: Fundamentals

### Q1. What is plane capacitance, and how does a PCB power/ground plane pair contribute to PDN decoupling?

**Answer:**

A PCB power/ground plane pair separated by a thin dielectric forms a parallel-plate capacitor. For a pair of rectangular planes of area $A$ (in m²), dielectric constant $\varepsilon_r$, and separation $h$ (in m):

$$C_{plane} = \varepsilon_0 \varepsilon_r \frac{A}{h}$$

where $\varepsilon_0 = 8.854\times10^{-12}\ F/m$.

**Practical calculation:**

A useful rule of thumb: for a dielectric with $\varepsilon_r \approx 4$ (FR4) and separation $h = 4\ mils\ (100\ \mu m)$:

$$C_{plane} \approx \frac{4 \times 8.854\times10^{-12}}{100\times10^{-6}} \approx 354\ pF/cm^2$$

For a 100 cm² (10 cm × 10 cm) board area dedicated to a single power/ground pair:

$$C_{plane} \approx 354\ pF/cm^2 \times 100\ cm^2 \approx 35.4\ nF$$

**Why plane capacitance matters:**

Plane capacitance provides "free" decoupling with very low inductance. Because the capacitance is distributed uniformly across the plane pair, the inductance of accessing it from any via is only the spreading inductance — much lower than the ESL of a discrete capacitor. Plane capacitance is most effective above the SRF of the last MLCC stage (where discrete capacitors become inductive) and below the frequency where plane resonance peaks emerge.

**Enhancing plane capacitance:**

- Use thinner dielectrics between the power/ground pair: halving $h$ doubles $C_{plane}$ and halves spreading inductance
- Use high-$\varepsilon_r$ laminate materials (e.g., $\varepsilon_r = 10\text{–}20$) for the power/ground pair — some embedded capacitance laminates achieve hundreds of pF/cm²
- Maximise the area of the coupled power/ground pair

**Limitation:** Beyond a few hundred MHz, the plane pair no longer behaves as a lumped capacitor — it becomes a transmission cavity, and resonant modes appear. The lumped capacitance model breaks down at these frequencies.

---

### Q2. What is power plane cavity resonance? Provide a qualitative explanation and identify the governing physical parameters.

**Answer:**

A power/ground plane pair enclosed within the PCB board area forms a 2D electromagnetic cavity. The power plane acts as one plate of a parallel-plate waveguide; the ground plane acts as the other. Electromagnetic waves can travel laterally (in the $x$ and $y$ directions) through the thin dielectric between the planes, reflecting off the plane edges.

**Why resonance occurs:**

When the wavelength of a propagating wave in the plane pair equals a simple fraction of the plane dimensions, standing waves form. At these resonant frequencies, the electric field in the cavity has a spatial pattern (mode shape) that can be constructively reinforced. A current injected by an IC at a node that coincides with a voltage antinode of the resonant mode sees a very high impedance — a resonance peak in the PDN.

**Governing parameters:**

1. **Plane dimensions:** $a$ (length) and $b$ (width) in metres
2. **Dielectric constant $\varepsilon_r$:** determines the propagation velocity in the plane cavity
3. **Dielectric thickness $h$:** affects the characteristic impedance and resonance Q-factor
4. **Boundary conditions:** planes are (approximately) open circuits at their edges, so electric field has antinodes at the edges

**Velocity in the cavity:**

$$v_{ph} = \frac{c}{\sqrt{\varepsilon_r}}$$

For FR4 with $\varepsilon_r = 4$: $v_{ph} = 3\times10^8 / 2 = 1.5\times10^8\ m/s$.

**First resonance (half-wave along length $a$):**

$$f_{10} = \frac{v_{ph}}{2a} = \frac{c}{2a\sqrt{\varepsilon_r}}$$

For a 10 cm × 10 cm board with FR4:

$$f_{10} = \frac{1.5\times10^8}{2\times0.10} = 750\ MHz$$

This is squarely in the frequency range of interest for modern high-speed designs.

---

### Q3. What is the relationship between plane resonance and the PDN impedance profile? Why does resonance cause problems?

**Answer:**

Plane resonance creates sharp peaks in the PDN self-impedance $|Z_{11}(f)|$ at the resonant frequencies. At a resonant frequency, the plane cavity stores energy and presents a high impedance at points of maximum electric field (voltage antinodes).

**Impact on PDN impedance:**

Between discrete capacitor SRFs (where they are inductive) and the onset of plane resonance, the plane capacitance provides low, reasonably flat impedance. At resonant frequencies, the plane impedance rises sharply — potentially to hundreds of milliohms — well above the target impedance $Z_{target}$.

**Spatial dependence:**

The resonance peak impedance depends on where on the board the current is injected (aggressor IC) and where the voltage is observed (victim IC). An IC placed at the centre of the board excites different modes than one placed at a corner.

- **Corner location:** Couples strongly to the (1,1), (1,3), (3,1) modes (modes with antinodes at corners)
- **Edge centre location:** Couples strongly to the (1,0) and (0,1) modes
- **Board centre location:** Does not couple to even-numbered modes (which have nodes at the centre); couples most strongly to (1,1) mode

This spatial non-uniformity means that moving an IC on the board can dramatically change which resonant frequencies affect it — a powerful but practically challenging design variable.

**Q-factor:** The quality factor of plane resonances is primarily limited by:
- Dielectric loss (loss tangent of the PCB laminate)
- Resistive loss in the copper planes
- Energy absorbed by components (capacitors act as loads on the cavity)

For typical FR4, $Q \approx 30\text{–}100$ at resonance, giving peak impedances that are $Q$ times the characteristic impedance of the cavity. Adding lossy components or lossy materials reduces $Q$ and lowers peak impedances.

---

## Tier 2: Intermediate

### Q4. Derive the resonant frequencies of a rectangular power/ground plane cavity and identify the dominant modes.

**Answer:**

The power/ground plane pair forms a 2D parallel-plate cavity with open-circuit boundary conditions at all four edges (the fringing fields at the edges are treated as magnetic walls in the standard TM-mode model).

**Governing equation:**

For TM (transverse magnetic) modes, where the electric field is perpendicular to the planes (in the $z$-direction), the resonant frequencies satisfy:

$$f_{mn} = \frac{v_{ph}}{2}\sqrt{\left(\frac{m}{a}\right)^2 + \left(\frac{n}{b}\right)^2}$$

where:
- $v_{ph} = c/\sqrt{\varepsilon_r}$ — phase velocity in the dielectric
- $m, n = 0, 1, 2, \ldots$ — mode indices (not both zero)
- $a$ — plane length (larger dimension)
- $b$ — plane width (smaller dimension)

**Physical interpretation:** Mode $(m, n)$ has $m$ half-wavelengths along the $a$ dimension and $n$ half-wavelengths along the $b$ dimension.

**Dominant modes for a square board ($a = b = L$):**

| Mode $(m,n)$ | Resonant frequency | Node/antinode pattern |
|---|---|---|
| (1, 0) | $v_{ph}/(2L)$ | Antinode at edges perpendicular to $a$, node at centre along $a$ |
| (0, 1) | $v_{ph}/(2L)$ | Same but along $b$ (degenerate with (1,0) for square) |
| (1, 1) | $v_{ph}\sqrt{2}/(2L)$ | Antinodes at all four corners, node at centre |
| (2, 0) | $v_{ph}/L$ | Two half-waves along $a$ |
| (2, 1) | $v_{ph}\sqrt{5}/(2L)$ | Mixed mode |

**Numerical example:** For $a = 15\ cm$, $b = 10\ cm$, $\varepsilon_r = 4$:

$$v_{ph} = 1.5\times10^8\ m/s$$

$$f_{10} = \frac{1.5\times10^8}{2\times0.15} = 500\ MHz$$

$$f_{01} = \frac{1.5\times10^8}{2\times0.10} = 750\ MHz$$

$$f_{11} = \frac{1.5\times10^8}{2}\sqrt{\left(\frac{1}{0.15}\right)^2 + \left(\frac{1}{0.10}\right)^2} = 75\times10^6\sqrt{44.4 + 100} = 75\times10^6 \times 12.0 = 901\ MHz$$

These frequencies are in the 500 MHz to 1 GHz range — directly coinciding with the frequencies generated by modern high-speed IC switching edges.

---

### Q5. Explain electromagnetic interference (EMI) from plane resonance. How do plane resonances radiate?

**Answer:**

Plane resonances that are excited by device switching currents do not remain confined to the PDN — they can radiate as electromagnetic interference.

**Radiation mechanism:**

The resonating plane cavity has a voltage distribution with antinodes at the plane edges. A time-varying voltage at the edge of the plane drives the electric field between the planes out into the free space beyond the board edge. This fringe field at the edge is equivalent to a slot antenna radiating in the plane of the board.

**Radiation efficiency:**

The plane cavity acts as a cavity-backed slot antenna. The slot length is the plane edge dimension; for the (1,0) mode, the effective slot length is $a$ (half-wave resonance). The radiation is polarised parallel to the board and is maximum broadside to the plane edge.

**EMI severity:** At resonance, the voltage at the plane edges can be significantly amplified compared to the driving voltage (by a factor approximately equal to the cavity Q). Even a few millivolts of PDN noise at a plane resonance can produce tens of millivolts of edge voltage, which can radiate efficiently if $a$ is a significant fraction of the free-space wavelength at the resonant frequency.

**Regulatory implications:** Many products fail radiated emissions (FCC Class B, CISPR 32) at frequencies that correlate exactly with calculated plane resonances. The diagnostic tool is to measure radiated emissions while sweeping the clock frequency — if a prominent emission moves with the clock (and its harmonics), it is likely a PDN-excited plane resonance.

**Mitigation for EMI:**

- Suppress the plane resonance (see Q6)
- Ensure the ground plane extends to the edge of the board (reducing the edge slot impedance by creating a "cap" at the edge)
- Add via stitching around the board perimeter to create a shielded wall that reduces radiation from the edge

---

### Q6. What are the primary techniques for suppressing power plane resonances? Compare their effectiveness and trade-offs.

**Answer:**

**Technique 1: Electromagnetic bandgap (EBG) structures**

EBG structures are periodic patterns etched into one of the planes (typically the power plane) to create a stop-band for wave propagation. Common implementations:

- Mushroom-type EBG: a periodic array of patches on one layer connected to the reference plane by vias, forming an LC ladder that has a stop-band (forbidden band) around the resonant frequency of the LC unit cell
- High-impedance surface: similar to mushroom EBG

**Effectiveness:** Creates a frequency band (the stop-band) where propagation is suppressed; resonances within the stop-band are dramatically attenuated (40–60 dB suppression possible).

**Trade-offs:** Complex PCB layout, vias consume routing space, stop-band bandwidth is limited, effective primarily in one frequency decade. Not compatible with dense via fields around BGAs.

**Technique 2: Distributed decoupling capacitors**

Placing many MLCCs uniformly across the plane (not just near ICs) increases the distributed loading on the cavity. Each capacitor acts as a lumped load that absorbs energy and damps resonances.

**Effectiveness:** Moderate — reduces Q-factor from ~50 to ~5–10 in a well-populated board. Does not eliminate resonances but reduces peak heights. Good supplementary technique.

**Trade-offs:** Uses board area, adds cost, needs vias. Effectiveness diminishes above the SRF of the capacitors (when they become inductive, they can actually reinforce resonances rather than damp them).

**Technique 3: Lossy materials (embedded resistive laminates)**

Replacing the standard FR4 between power and ground planes with a resistive or lossy laminate (e.g., a laminate with high dielectric loss tangent, or an embedded resistive film) dissipates cavity resonance energy.

**Effectiveness:** Very effective — can reduce resonance peaks by 20–40 dB. The loss is broadband and frequency-independent, addressing all resonant modes simultaneously.

**Trade-offs:** More expensive laminate, must be sourced as a special stackup, may not be available from all PCB manufacturers. Also increases low-frequency PDN impedance slightly (the resistive layer adds series resistance to the plane capacitance).

**Technique 4: Plane shape and slot design**

Changing the shape of the power plane (e.g., making it non-rectangular, adding cutouts) disrupts the resonant mode structure and shifts resonant frequencies away from harmonics of the clock.

**Effectiveness:** Can shift specific resonances but may create new ones. Not a general solution.

**Technique 5: Split planes and multiple small planes**

Replacing one large power plane with several smaller zones reduces the maximum plane dimension, pushing all resonant frequencies upward. If plane segments are smaller than $\lambda_{eff}/4$ at the highest frequency of interest, resonances do not develop.

**Effectiveness:** Eliminates resonances in the covered frequency range but requires careful design to avoid plane discontinuities that create impedance disruptions for signal return currents.

---

## Tier 3: Advanced

### Q7. Derive the characteristic impedance of the power/ground plane cavity and explain how it determines the peak impedance at resonance.

**Answer:**

The power/ground plane pair is a parallel-plate transmission line (or waveguide) with plate separation $h$, plate width $w$, and dielectric constant $\varepsilon_r$. The characteristic impedance for TEM propagation along the plane:

$$Z_0^{plane} = \frac{\eta h}{w} = \frac{377\ \Omega \cdot h}{w\sqrt{\varepsilon_r}}$$

where $\eta = \mu_0/\varepsilon_0/\sqrt{\varepsilon_r} = 377/\sqrt{\varepsilon_r}\ \Omega$ is the wave impedance in the dielectric.

For a square cross-section ($w = h$): $Z_0^{plane} = 377/\sqrt{\varepsilon_r}\ \Omega$.

For a practical plane pair, $w \gg h$ (the planes are wide compared to their separation), so $Z_0^{plane} \ll 1\ \Omega$. For example, with $h = 100\ \mu m$, $w = 10\ cm$, $\varepsilon_r = 4$:

$$Z_0^{plane} = \frac{377 \times 100\times10^{-6}}{0.10 \times \sqrt{4}} \approx \frac{0.0377}{0.2} = 0.189\ \Omega$$

**Peak impedance at resonance:**

The plane cavity at resonance has a voltage distribution that peaks at the antinodes. For a 1D resonance (mode (1,0)), the input impedance looking into a shorted (or open-circuited) transmission line of length $a$ at resonance is:

For an open-circuited plane edge (standard model), the input impedance of a lossless line of length $a$ at frequency $f_{10} = v_{ph}/(2a)$:

$$Z_{in} = Z_0^{plane} \cot\left(\frac{\pi}{2}\right) \rightarrow \infty \quad \text{(lossless)}$$

With finite loss (loss tangent $\tan\delta$), the peak impedance is:

$$|Z_{peak}| \approx \frac{Z_0^{plane}}{2 \alpha a} = \frac{Z_0^{plane}}{2 \cdot (\pi f \tan\delta / v_{ph}) \cdot a}$$

For $\tan\delta = 0.02$ (FR4), $a = 0.10$ m, $f = 750\ MHz$, $v_{ph} = 1.5\times10^8\ m/s$:

$$\alpha = \frac{\pi \times 750\times10^6 \times 0.02}{1.5\times10^8} = \frac{\pi \times 0.02 \times 5}{1} \approx 0.314\ \text{Np/m}$$

$$|Z_{peak}| \approx \frac{0.189}{2 \times 0.314 \times 0.10} = \frac{0.189}{0.0628} \approx 3\ \Omega$$

This is far above any PDN target impedance — confirming that plane resonances must be actively suppressed or avoided. The actual peak height also depends on Q (copper loss, component loading), but 1–10 $\Omega$ peaks are typical.

---

### Q8. How do you model the input impedance of a power/ground plane pair including resonances? Describe the cavity model and how it is incorporated into PDN analysis.

**Answer:**

The standard analytical model for a rectangular plane pair is the transmission cavity (or cavity resonator) model. It expresses the input admittance at port $(x_1, y_1)$ due to a current source at the same or a different port $(x_2, y_2)$:

**2D cavity model (Green's function approach):**

The admittance $Y_{11}$ at a single port $(x_1, y_1)$ (self-admittance) for a rectangular cavity of dimensions $a \times b$ is:

$$Y_{11}(f) = \frac{-j\omega \varepsilon_r \varepsilon_0}{ab} \sum_{m=0}^{\infty}\sum_{n=0}^{\infty} \frac{\Lambda_{mn} \cos^2(m\pi x_1/a)\cos^2(n\pi y_1/b)}{k_{mn}^2 - k^2}$$

where:
- $k = \omega\sqrt{\mu_0 \varepsilon_0 \varepsilon_r}(1 - j\tan\delta/2)$ — complex wave number
- $k_{mn}^2 = (m\pi/a)^2 + (n\pi/b)^2$ — resonance wave number for mode $(m,n)$
- $\Lambda_{mn} = 1$ for $m = 0$ or $n = 0$; $\Lambda_{mn} = 2$ for both $m, n \neq 0$; $\Lambda_{00} = 0$ (not physical)

**Practical implementation:**

The double sum converges slowly and is typically truncated after enough modes to cover the frequency range of interest. For PDN analysis up to 3 GHz on a 10 cm board, modes up to (5,5) are usually sufficient.

**Transfer admittance $Y_{21}$** (between two ports) uses the product of cosines evaluated at the two port locations instead of the square of cosines.

**Incorporating the cavity model into PDN simulation:**

1. Extract the plane pair admittance matrix $[Y_{plane}]$ at each frequency using the cavity model
2. Convert to impedance: $[Z_{plane}] = [Y_{plane}]^{-1}$
3. Add the impedances of discrete capacitors connected to the planes as shunt elements
4. Add the VRM as a controlled source with its output impedance
5. Compute the total PDN self-impedance at the IC power pins

**Modern software approach:** Commercial PI tools (Ansys SIwave, Cadence Clarity) use full-wave 2.5D or 3D EM analysis that automatically captures all resonant modes without the truncation approximation. The cavity model is useful for hand calculations and first-cut estimates.

---

### Q9. Describe the concept of an electromagnetic bandgap (EBG) structure for PDN noise suppression. Derive the stop-band frequency from the equivalent circuit.

**Answer:**

An EBG (electromagnetic bandgap) structure applied to a power plane creates a frequency band in which electromagnetic wave propagation between two points on the power distribution network is forbidden. Within this stop-band, PDN noise generated at one location cannot propagate to another location on the board.

**Mushroom-type EBG construction:**

The power plane is etched into a periodic array of square metal patches of size $d$ connected by vias of inductance $L_{via}$ to the ground plane. The gaps between patches have fringe capacitance $C_{gap}$. Each unit cell of the EBG can be modelled as a parallel LC resonator connected to the transmission-line ground plane:

```
    |-----L_{via}-----|
    |                 |
  --+--[patch patch]--+--   (repeated periodically)
    |                 |
   GND               GND
       |---C_{gap}---|
           (fringe cap between adjacent patches)
```

The unit cell has a parallel resonant frequency:

$$f_0 = \frac{1}{2\pi\sqrt{L_{via} C_{gap}}}$$

**Stop-band derivation:**

For a periodic Bloch-mode structure, the dispersion relation gives a stop-band (forbidden band) centred on $f_0$. The normalised bandwidth of the stop-band is:

$$\frac{\Delta f}{f_0} = \frac{1}{\pi}\sqrt{\frac{L_{via}/C_{gap}}{Z_0^{plane,eq}^2}}$$

In practice, the stop-band extends from approximately $f_0/2$ to $f_0$ for a well-designed EBG.

**Design procedure:**

1. Identify the target frequency $f_{target}$ (e.g., a clock harmonic where plane resonance is problematic)
2. Design $L_{via}$ and $C_{gap}$ so $f_0 \approx f_{target}$:
   - $L_{via}$: controlled by via diameter and height (typically 0.3–1.5 nH)
   - $C_{gap}$: controlled by the gap width between patches (typically 0.05–0.5 pF)
3. Verify stop-band coverage with EM simulation
4. Place the EBG between the aggressor IC (switching regulator noise source) and the victim IC (sensitive analogue or RF circuit)

**Example:** For $L_{via} = 0.8\ nH$, $C_{gap} = 0.2\ pF$:

$$f_0 = \frac{1}{2\pi\sqrt{0.8\times10^{-9} \times 0.2\times10^{-12}}} = \frac{1}{2\pi\sqrt{1.6\times10^{-22}}} = \frac{1}{2\pi \times 1.265\times10^{-11}} \approx 1.26\ GHz$$

The stop-band would suppress propagation from approximately 630 MHz to 1.26 GHz.

**Limitations of EBG:**
- Only suppresses propagation within the stop-band — outside the band, noise propagates normally
- EBG structures require routing area and via resources
- They are one-directional structures — they block propagation between two zones but do not suppress radiation or resonance within a zone
- Complex to design for broadband suppression (multiple EBG zones with different unit cell sizes needed)

---

## Summary: Plane Resonance Quick Reference

| Parameter | Formula | Notes |
|---|---|---|
| Plane capacitance | $C = \varepsilon_0 \varepsilon_r A/h$ | Free decoupling from plane structure |
| Mode $(m,n)$ resonant frequency | $f_{mn} = \frac{c}{2\sqrt{\varepsilon_r}}\sqrt{(m/a)^2+(n/b)^2}$ | $m,n \geq 0$, not both zero |
| Phase velocity in dielectric | $v_{ph} = c/\sqrt{\varepsilon_r}$ | FR4: $\approx 1.5\times10^8$ m/s |
| First resonance (longest dimension $a$) | $f_{10} = v_{ph}/(2a)$ | Dominant mode, lowest frequency |
| Plane characteristic impedance | $Z_0 = 377h/(w\sqrt{\varepsilon_r})$ | Very low for wide planes |
| EBG unit-cell resonance | $f_0 = 1/(2\pi\sqrt{L_{via}C_{gap}})$ | Centre of stop-band |

---

## Key Mitigation Strategies Summary

| Problem | Root Cause | Mitigation |
|---|---|---|
| Plane resonance impedance peak | Standing wave in plane cavity | EBG, lossy laminate, plane splitting |
| High Q resonance | Low dielectric loss, few capacitors | Use high-loss laminate; populate capacitors across board |
| Resonance at clock harmonic | Board dimension matched to $\lambda/2$ | Change board dimensions, relocate IC |
| EMI from plane resonance | Fringe field radiation from edges | Via stitching perimeter, extended ground plane to board edge |
| Resonance couples between ICs | Shared power plane | Split planes, moat around sensitive circuits |
