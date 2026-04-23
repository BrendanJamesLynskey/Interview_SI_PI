# Copper Surface Roughness and Conductor Loss Models

## Overview

Above a few GHz, the copper-to-laminate bond side of a PCB trace is not smooth at the scale of the skin depth. The peak-to-valley profile of electrodeposited copper is routinely 1–10 µm, while at 10 GHz the skin depth is only ~0.66 µm. The electromagnetic field is forced to follow the rough surface contour, increasing the effective path length and therefore the conductor loss. At 25 GHz and above, surface roughness contributes **30–60% of total conductor loss** — often more than the dielectric loss. This document covers the widely used surface-roughness models (Hammerstad–Jensen, Huray "snowball", modified Huray/Cannonball-Huray), how each is parameterised, and how to pick a model for a real design. Key references: Eric Bogatin's material, Bert Simonovich's Cannonball-Huray work, and Hammerstad & Jensen (IEEE MTT 1980).

---

## Tier 1: Fundamentals

### Q1. Why does surface roughness increase conductor loss at high frequency?

**Answer:**

At any frequency, current in a good conductor concentrates within roughly one skin depth of the surface:

$$\delta = \sqrt{\frac{1}{\pi f \mu \sigma}}$$

For copper at 1 GHz: $\delta \approx 2.1$ µm. At 10 GHz: 0.66 µm. At 25 GHz: 0.42 µm.

**When the skin depth is comparable to or smaller than the surface-roughness features, the current must follow the bumpy surface contour.** The effective surface area (and therefore the resistance) of a rough conductor is larger than of a smooth conductor of the same "macroscopic" dimensions. The effect can be viewed as:

- **Increased path length:** the current's surface path is longer than the straight-line distance, so resistance per unit length is higher.
- **Higher surface impedance:** $Z_s(f)$ is multiplied by a roughness factor $K_{rough}(f) > 1$ that grows with $f$ and saturates once $\delta \ll R_z$.
- **Effective conductivity reduction:** the roughness can also be modelled as a frequency-dependent effective $\sigma$ reduction.

**Roughness grades for PCB copper:**

| Copper type | $R_a$ (rms roughness) | $R_z$ (peak-to-valley) |
|---|---|---|
| Standard ED copper (STD) | 1.5–3 µm | 6–10 µm |
| Low-profile ("RTF", "HTE LP") | 0.5–1 µm | 2–4 µm |
| Very low profile (VLP) | 0.3–0.5 µm | 1–2 µm |
| Hyper Very Low Profile (HVLP, "Ultra") | < 0.3 µm | < 1 µm |
| Ultra HVLP / "HVLP3" / "EV" | < 0.1 µm | < 0.5 µm |

Modern high-speed designs above 25 Gb/s use VLP or HVLP copper. Below 10 Gb/s, standard ED copper is fine.

---

### Q2. What is the Hammerstad–Jensen roughness model, and when is it adequate?

**Answer:**

The Hammerstad–Jensen model (1980) is the oldest and simplest widely-used roughness correction. It gives the multiplier $K_{rough}$ as a function of the skin depth and the RMS roughness $\Delta$:

$$K_{HJ}(f) = 1 + \frac{2}{\pi}\arctan\left[1.4\left(\frac{\Delta}{\delta(f)}\right)^2\right]$$

**Asymptotic behaviour:**

- At low frequency ($\delta \gg \Delta$): $K_{HJ} \rightarrow 1$ (no correction).
- At high frequency ($\delta \ll \Delta$): $K_{HJ} \rightarrow 2$ (saturates at 2× the smooth-conductor loss).

**Why it saturates at 2:**

Hammerstad–Jensen is based on the idea that the rough surface doubles the effective surface area in the extreme limit. This is a reasonable approximation for moderately rough ED copper and for frequencies where $\delta$ is comparable to $\Delta$, but it does not capture the loss continuing to increase above ~10 GHz for very rough surfaces.

**When Hammerstad–Jensen is adequate:**

- Frequencies below ~10 GHz.
- Moderately rough copper (standard ED or RTF).
- First-order sanity checking — it gives a conservative (low) estimate of roughness loss for very rough surfaces.

**When it fails:**

- Above 20 GHz: actual measurements show roughness multipliers up to 3–4× for very rough copper, exceeding Hammerstad–Jensen's saturation limit.
- For HVLP copper at any frequency: it over-predicts loss for very smooth surfaces (practically this is a conservative error).

---

### Q3. What is the Huray "snowball" model and how does it differ from Hammerstad–Jensen?

**Answer:**

Huray (2010) proposed a physics-based model that accounts for the actual geometry of electrodeposited copper surfaces. ED copper consists of stacks of approximately spherical grain "nodules" — like a pile of snowballs — with a specific density and size distribution determined by the deposition process.

**Model structure:**

$$K_{Huray}(f) = 1 + \frac{A_{hex}}{A_{matte}}\sum_{i} N_i \cdot \frac{3}{2}\left[1 - \frac{1}{1 + \delta(f)/a_i}\right]^{-1}$$

Where:
- $a_i$ is the radius of the spheres in tile $i$.
- $N_i$ is the number of spheres per unit area in tile $i$.
- $A_{hex}/A_{matte}$ is a geometric packing factor.

In the two-parameter simplified form (one sphere size):

$$K_{Huray}(f) = 1 + \frac{N \cdot 4\pi a^2}{A}\cdot\frac{1}{1 + \delta(f)/a + \delta(f)^2/(2a^2)}$$

**Parameters typically used:**

- $a$: "snowball" radius, ~0.5–2 µm for ED copper, ~0.2–0.5 µm for HVLP.
- $N/A$: surface number density, typically $N/A \approx 1/(2\pi a^2)$ for close-packed.

**Key differences from Hammerstad–Jensen:**

1. **Does not saturate at 2×:** Huray can exceed 2× for very rough or dense nodule distributions.
2. **Physically parameterised:** $a$ and $N$ can be measured by SEM of the copper surface, making the model calibrated to real copper rather than a bulk roughness number.
3. **Captures frequency dependence more accurately:** Huray tracks actual measured loss more closely up to 50+ GHz.

**Where Huray is used:**

- Vendor-supplied high-speed stackup models (Panasonic Megtron 7N, Isola Astra, Rogers Tachyon).
- Field-solver tools (Ansys HFSS, Cadence Sigrity) default to Huray or derivatives above 10 GHz.
- IEEE 802.3bj/ck/df compliance channel modelling.

---

## Tier 2: Intermediate

### Q4. What is "Cannonball-Huray" and why was it introduced?

**Answer:**

Cannonball-Huray is a simplification of the Huray model proposed by Bert Simonovich (LAMSIM Enterprises, DesignCon 2016) that uses a geometric assumption (close-packed spheres like stacked cannonballs) to reduce the number of free parameters.

**Motivation:**

The general Huray model requires measuring or fitting $a$ and $N/A$ for each specific copper foil — time-consuming and often proprietary to the laminate vendor. The Cannonball-Huray model assumes the copper nodules are in a **close-packed hexagonal arrangement of spheres** and relates the sphere radius $a$ to the RMS roughness $R_z$ via a geometric formula:

$$a = \frac{R_z}{\text{constant}}, \quad N/A = \frac{1}{\pi a^2}\cdot\text{packing factor}$$

Typically a single "effective" $a$ is chosen and $N/A$ is fixed by the close-packing assumption. The model then needs only one parameter — $R_z$ — which is a datasheet-reported number.

**Formula (simplified):**

$$K_{CBH}(f) = 1 + 3\cdot\frac{\delta(f)/a + \delta(f)^2/(2a^2)}{1 + \delta(f)/a + \delta(f)^2/(2a^2)}$$

with $a \approx R_z/3$ or so (exact constant depends on the original Simonovich paper's fit).

**When it is useful:**

- Quick first-pass simulations using only datasheet roughness values.
- Correlation against Huray-full models for sanity check.
- Field-solver "auto-calibration" against IEEE 802.3 reference channels.

**Simonovich's validation:** shows Cannonball-Huray matches measured insertion loss of a 0.5 oz / HVLP Panasonic Megtron 6 stackup to within ±0.5 dB at 25 GHz — accurate enough for design work.

---

### Q5. Quantify how much conductor loss is contributed by roughness for a typical 25 Gb/s channel.

**Answer:**

**Setup:**

- 50 Ω stripline, standard 1 oz RTF copper, 5 mil wide.
- Nyquist frequency for 25 Gb/s NRZ: 12.5 GHz.
- $R_z$ of RTF copper: ~3 µm.
- Skin depth at 12.5 GHz: $\delta = 0.59$ µm.
- $\delta/R_z = 0.20$ — firmly in the roughness-dominated regime.

**Smooth-conductor loss (baseline):**

For 50 Ω, 5 mil stripline at 12.5 GHz: $\alpha_{c,smooth} \approx 0.07 \cdot \sqrt{12.5} = 0.25$ dB/inch.

**Apply roughness multiplier:**

Hammerstad–Jensen: $K_{HJ} \approx 1 + (2/\pi)\arctan[1.4 \cdot (3/0.59)^2] \approx 1 + (2/\pi) \cdot \pi/2 \approx 2.0$ (saturated).
Huray: $K_{Huray} \approx 2.5$ (typical for RTF).
Cannonball-Huray: $K_{CBH} \approx 2.3$.

Pick $K = 2.3$ as typical:

$$\alpha_{c,rough} = 2.3 \times 0.25 = 0.58\ \text{dB/inch}$$

**Roughness contribution alone:**

$$\alpha_{roughness} = \alpha_{c,rough} - \alpha_{c,smooth} = 0.58 - 0.25 = 0.33\ \text{dB/inch}$$

That is, roughness contributes ~0.33 dB/inch, compared to 0.25 dB/inch from skin effect alone. **Roughness is slightly more than the bare skin-effect loss** at this frequency.

**Compare to dielectric loss:**

For Megtron 6 at 12.5 GHz ($D_f \approx 0.005$): $\alpha_d \approx 2.3 \cdot \sqrt{3.7} \cdot 0.005 \cdot 12.5 \approx 0.28$ dB/inch.

**Total:**

$$\alpha_{total} = 0.58 + 0.28 = 0.86\ \text{dB/inch}$$

**Breakdown:**
- Skin effect (bare conductor): 0.25 dB/inch (29%).
- Roughness (additional conductor): 0.33 dB/inch (38%).
- Dielectric: 0.28 dB/inch (33%).

**Roughness is the largest single contributor** to the loss — bigger than dielectric, bigger than bare skin effect. Using HVLP copper ($R_z < 1$ µm, $K \approx 1.3$) would drop the conductor-roughness contribution to ~0.08 dB/inch, saving roughly 0.25 dB/inch — enormous over a 10-inch channel.

---

### Q6. How do you select the copper grade for a given SerDes channel?

**Answer:**

**Decision variables:**

1. **Channel length** and the loss budget at Nyquist.
2. **Frequency at Nyquist.**
3. **Cost sensitivity.**
4. **Availability** — HVLP copper is not universally available at all fabricators and not for all laminate families.

**Rule-of-thumb guide:**

| Nyquist frequency | Channel length | Recommended copper |
|---|---|---|
| ≤ 5 GHz | any | Standard ED or RTF |
| 5–12 GHz | ≤ 15 inches | RTF (reverse-treated) |
| 5–12 GHz | > 15 inches | VLP |
| 12–25 GHz | any | VLP or HVLP |
| 25–40 GHz | any | HVLP |
| > 40 GHz | any | HVLP-3 / Ultra HVLP |

**Cost premium:**

| Copper | Typical cost premium |
|---|---|
| Standard ED | 0% (baseline) |
| RTF | 0–5% |
| VLP | 10–20% |
| HVLP | 20–40% |
| HVLP-3 / Ultra | 40–80% |

**Yield considerations:**

HVLP copper has reduced bond strength to the laminate compared to rougher copper (rougher copper keys mechanically into the resin). Fabricators sometimes report yield issues on HVLP — specifically, trace lifting during soldering or thermal cycling. Ask the fab about their HVLP process window before committing the design.

---

## Tier 3: Advanced

### Q7. How do you extract and validate a roughness model from measured insertion-loss data?

**Answer:**

**Procedure:**

1. **Fabricate a test coupon** with differential pairs of varying lengths (say 4 inch, 8 inch, 12 inch). Route on the target stackup with the target copper.
2. **Measure $|S_{21}|(f)$ for each length on a VNA** up to at least 2× the design Nyquist (50+ GHz for 25 Gb/s designs).
3. **Compute α(f) from delta loss:**

$$\alpha(f)\ [\text{dB/inch}] = \frac{|S_{21,long}|(f) - |S_{21,short}|(f)}{L_{long} - L_{short}}$$

This removes fixture, via, and connector losses.

4. **Separate conductor and dielectric loss:**
   - At low frequency (0.5–2 GHz), loss is conductor-dominated (skin effect): fit to $\alpha_c = A_c \sqrt{f}$ with no roughness.
   - At high frequency (above the crossover), extract residual roughness as the difference between the measured $\alpha_c(f)$ and the smooth-conductor prediction.

5. **Fit the Huray $a$, $N/A$** (or Cannonball-Huray $R_z$ alone) to match the measured α(f) curve.

6. **Validate**: use the extracted model to predict loss on a separate test coupon of different length or geometry. Agreement within 0.5 dB at Nyquist is considered good correlation.

**Common pitfalls:**

- **Fixture de-embedding errors:** if the fixture structures are not properly removed (2X-thru or IEEE P370 methods), the extracted loss is polluted by connector and via losses.
- **Resin-over-glass variability:** fibre-weave effects cause trace-to-trace variability in α(f) within a panel. Measure multiple pairs and average.
- **Temperature dependence:** α(f) varies by up to 10% between 0°C and 85°C for typical laminates. Match test conditions to the target operating temperature.

---

### Q8. The gap between roughness-induced loss and bare skin-effect loss is one of the main frequency-scaling limits of copper PCBs. What are the alternatives being developed?

**Answer:**

Several alternatives are being explored as copper-on-laminate reaches its fundamental limits at 200+ Gb/s:

**1. Improved copper surface treatments:**

- "Bondtreat" chemistries that achieve sub-200 nm peak-to-valley while maintaining laminate adhesion.
- Non-electrodeposited copper (sputtered or ALD) with crystalline orientations that have fewer rough grain boundaries.
- Polymer-modified copper for even lower roughness.

Effectiveness: incremental (~10–20% loss reduction at 30+ GHz).

**2. Polymer-clad copper (Panasonic R-5775, Isola Astra MT77):**

Ultra-low-profile copper laminated with low-$D_k$ polymer cladding that eliminates the need for glass reinforcement in signal layers. Roughness factor approaches 1.0 (negligible contribution).

Effectiveness: significant (~30–50% loss reduction at 30 GHz).

**3. Glass-free / organic build-up substrates (OBS):**

Used in package substrates; increasingly migrating to PCB. No glass → no fibre-weave + controlled copper surface. Loss at 50 GHz comparable to coaxial cable.

**4. Optical interconnect at board level (co-packaged optics, CPO):**

Replace electrical SerDes entirely with optical at distances > a few cm. Loss of fibre is ~0.2 dB/km at 1550 nm — utterly negligible for any PCB application. Roadblocks: laser and modulator integration cost, power efficiency, and thermal.

**5. 2.5D / 3D packaging (interposer, hybrid bonding):**

Move high-speed signals out of the PCB and onto a silicon interposer or directly between stacked dice. PCB-level SerDes then runs only at the (slower) package-to-package interface.

**The endgame:** by 2030 it is expected that high-speed PCB SerDes at 224 Gb/s will push copper-on-laminate to its limits, with optical interconnect and chiplet-level 3D packaging displacing long-reach PCB electrical channels.

---

## Summary: Roughness and Loss Quick Reference

| Copper grade | $R_z$ (µm) | $K_{rough}$ at 25 GHz | Added α at 25 GHz |
|---|---|---|---|
| Standard ED | 6–10 | 3–4 | +0.8 dB/inch |
| RTF / LP | 3 | 2.3 | +0.35 dB/inch |
| VLP | 1–2 | 1.5 | +0.15 dB/inch |
| HVLP | < 1 | 1.2 | +0.07 dB/inch |
| HVLP-3 / Ultra | < 0.5 | 1.05 | +0.02 dB/inch |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Skin depth | $\delta = 1/\sqrt{\pi f \mu \sigma}$ |
| Rough conductor loss | $\alpha_{c,rough} = K_{rough}(f) \cdot \alpha_{c,smooth}$ |
| Hammerstad–Jensen | $K_{HJ} = 1 + (2/\pi)\arctan[1.4(\Delta/\delta)^2]$ |
| Huray (simplified) | $K_{Huray} = 1 + \frac{N\cdot 4\pi a^2}{A}\cdot\frac{1}{1+\delta/a + (\delta/a)^2/2}$ |
| Cannonball-Huray | Use $a \approx R_z/3$ with Huray form, single-parameter |
| Conductor+roughness+dielectric (total α) | $\alpha_{total}(f) = K_{rough}(f) A_c \sqrt{f} + 2.3 \sqrt{\varepsilon_r} Df \cdot f$ |
