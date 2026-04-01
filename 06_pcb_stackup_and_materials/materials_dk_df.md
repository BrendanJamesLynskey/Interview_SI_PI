# Materials: Dk and Df

## Overview

The dielectric material used in a PCB determines how fast signals propagate, how much signal energy is lost over length, and how consistently impedance can be held across a production run. For designs operating above 1 GHz, material selection is a first-order design decision — not a procurement afterthought. This document covers the physical meaning of relative permittivity ($D_k$) and loss tangent ($D_f$), their frequency dependence, the properties of commonly used high-speed laminates including Megtron 6 and IS680, and the often-overlooked effect of glass weave on signal integrity.

---

## Tier 1: Fundamentals

### Q1. What is relative permittivity ($D_k$) and how does it affect signal propagation on a PCB trace?

**Answer:**

Relative permittivity $\epsilon_r$ (commonly written $D_k$ in PCB material datasheets) is the ratio of the permittivity of the dielectric material to the permittivity of free space:

$$\epsilon_r = \frac{\epsilon}{\epsilon_0}$$

where $\epsilon_0 = 8.854 \times 10^{-12}$ F/m.

**Effect on propagation velocity:**

The phase velocity of a signal on a transmission line embedded in a homogeneous dielectric is:

$$v_p = \frac{c}{\sqrt{\epsilon_r \mu_r}} \approx \frac{c}{\sqrt{\epsilon_r}}$$

for non-magnetic PCB dielectrics ($\mu_r \approx 1$). Therefore a higher $D_k$ slows the signal:

| Material | Typical $D_k$ at 1 GHz | Propagation velocity |
|---|---|---|
| Air | 1.0 | $c = 300$ mm/ns |
| PTFE (Rogers 4003) | 3.55 | $159$ mm/ns |
| FR4 (standard) | 4.2–4.5 | $146$ mm/ns |
| Megtron 6 | 3.7 | $156$ mm/ns |

**Effect on characteristic impedance:**

For a microstrip trace, the characteristic impedance depends on $\epsilon_r$ through the effective permittivity $\epsilon_{eff}$:

$$\epsilon_{eff} = \frac{\epsilon_r + 1}{2} + \frac{\epsilon_r - 1}{2} \cdot \frac{1}{\sqrt{1 + 12h/W}}$$

$$Z_0 = \frac{87}{\sqrt{\epsilon_{eff} + 1.41}} \ln\left(\frac{5.98h}{0.8W + T}\right)$$

A 10% increase in $D_k$ causes approximately a 5% decrease in $Z_0$. For a 50 Ω target, a shift from $D_k = 4.0$ to $D_k = 4.4$ moves the impedance to roughly 47.6 Ω — outside the typical ±10% manufacturing tolerance if not compensated.

**Effect on propagation delay:**

$$t_{pd} = \frac{l}{v_p} = \frac{l\sqrt{\epsilon_{eff}}}{c}$$

A 100 mm trace on FR4 ($\epsilon_{eff} \approx 3.5$) has $t_{pd} \approx 623$ ps. The same trace on Rogers 4003 ($\epsilon_{eff} \approx 3.0$) has $t_{pd} \approx 577$ ps — a 46 ps difference that matters for matched-length differential pairs and timing margins.

---

### Q2. What is loss tangent ($D_f$) and how does it cause signal attenuation?

**Answer:**

The loss tangent $\tan \delta$ (written $D_f$ in PCB datasheets) is the ratio of the imaginary to the real part of the complex permittivity:

$$\tan \delta = \frac{\epsilon''}{\epsilon'}$$

where $\epsilon = \epsilon' - j\epsilon''$. The imaginary part $\epsilon''$ represents energy dissipated as heat in the dielectric. A higher $D_f$ means more of the signal's electromagnetic energy is absorbed per unit length.

**Dielectric attenuation per unit length:**

$$\alpha_d = \frac{\pi f \sqrt{\epsilon_{eff}}}{c} \cdot \tan \delta$$

in nepers/metre. Converting to dB/m: $\alpha_{d,dB} = 8.686 \times \alpha_d$.

For a rough approximation in dB per unit length:

$$\alpha_d \approx 27.3 \frac{\sqrt{\epsilon_r}}{c} f \cdot D_f \quad [\text{dB/m}]$$

**Practical attenuation numbers (100 mm trace):**

| Material | $D_f$ at 10 GHz | Dielectric loss at 10 GHz (100 mm) |
|---|---|---|
| FR4 (standard) | ~0.020 | ~2.4 dB |
| Megtron 6 | ~0.002 | ~0.24 dB |
| IS680 | ~0.004 | ~0.48 dB |
| Rogers 4003C | ~0.0027 | ~0.32 dB |

FR4's dielectric loss at 10 GHz is roughly 10x higher than Megtron 6. On a 200 mm trace, FR4 dielectric loss alone exceeds 4.8 dB — consuming a large portion of the 10 dB total loss budget for PCIe Gen4.

**Frequency scaling:** Dielectric loss scales linearly with frequency:

$$\alpha_d(f) \propto f \cdot D_f(f)$$

Because $D_f$ itself also increases with frequency for most materials (see Q4), the total dielectric loss grows faster than linearly with frequency. This is the primary reason that material selection becomes critical above 5 GHz.

---

### Q3. What are the two main loss mechanisms in a PCB trace, and how do they scale with frequency?

**Answer:**

The total insertion loss of a PCB transmission line is dominated by two loss mechanisms:

**1. Conductor loss (skin-effect loss):**

At high frequencies, current crowds into the outer skin of the conductor with skin depth $\delta_s$:

$$\delta_s = \sqrt{\frac{\rho}{\pi f \mu}}$$

where $\rho$ is the resistivity of copper ($\rho_{Cu} = 1.72 \times 10^{-8}$ Ω·m at 25°C). As frequency increases, $\delta_s$ decreases and the effective cross-section carrying current shrinks, increasing the conductor's AC resistance per unit length.

Conductor attenuation scales as:

$$\alpha_c \propto \sqrt{f}$$

**2. Dielectric loss:**

As described in Q2, dielectric loss scales approximately as:

$$\alpha_d \propto f \cdot D_f$$

**Comparing the two at different frequencies:**

At low frequencies (< 1 GHz), conductor loss dominates. At high frequencies (> 5–10 GHz on fine traces), dielectric loss often dominates. The crossover frequency depends on trace width, dielectric constant, and $D_f$.

For a typical 100 µm trace on FR4:

| Frequency | $\alpha_c$ (dB/100mm) | $\alpha_d$ (dB/100mm) | Total |
|---|---|---|---|
| 1 GHz | ~0.5 | ~0.24 | ~0.74 |
| 5 GHz | ~1.1 | ~1.2 | ~2.3 |
| 10 GHz | ~1.6 | ~2.4 | ~4.0 |
| 28 GHz | ~2.6 | ~6.7 | ~9.3 |

At 28 GHz, dielectric loss is 2.5x conductor loss on FR4. Switching to Megtron 6 reduces the 28 GHz total to approximately:

| Component | Megtron 6 (28 GHz) |
|---|---|
| $\alpha_c$ | ~2.6 dB (unchanged — conductor properties identical) |
| $\alpha_d$ | ~0.67 dB |
| **Total** | **~3.3 dB** |

This illustrates why low-$D_f$ material is essential for signals above 10 GHz: it cannot reduce conductor loss (which is a function of the copper, not the dielectric), but it dramatically reduces the dominant dielectric component.

**Surface roughness:** A third mechanism — conductor surface roughness — also increases effective conductor resistance at high frequencies. The Huray model or Cannonball model is used to account for copper foil roughness (VLP, HVLP, or RTF copper grades) in simulation. Surface roughness increases effective $\alpha_c$ by a factor of 1.2–2.5 at 10 GHz depending on copper type.

---

### Q4. What is FR4? What are its approximate $D_k$ and $D_f$ values, and at what frequencies does it become unsuitable for high-speed designs?

**Answer:**

FR4 (Flame Retardant 4) is a composite material consisting of woven fiberglass fabric reinforcement embedded in a brominated epoxy resin matrix. It is the dominant PCB substrate material due to its low cost, wide fabricator availability, mechanical strength, and adequate electrical properties at low to moderate frequencies.

**Nominal properties of standard FR4:**

| Parameter | Value | Notes |
|---|---|---|
| $D_k$ at 1 MHz | 4.5–4.8 | Varies with resin content and manufacturer |
| $D_k$ at 1 GHz | 4.2–4.5 | Decreases with frequency |
| $D_k$ at 10 GHz | 4.0–4.3 | Further decrease |
| $D_f$ at 1 MHz | 0.010–0.015 | Low loss at low frequency |
| $D_f$ at 1 GHz | 0.015–0.020 | Rising |
| $D_f$ at 10 GHz | 0.020–0.025 | High — limits usefulness |
| Glass transition temperature $T_g$ | 130–170°C | Higher $T_g$ grades available |
| CTE (z-axis) | 50–70 ppm/°C | Relevant for via reliability |

**Frequency limits of FR4:**

| Interface | Max frequency of interest | FR4 suitability |
|---|---|---|
| PCI Express Gen1 (2.5 GT/s) | 1.25 GHz | Adequate |
| USB 3.0 (5 Gb/s) | 2.5 GHz | Marginal |
| DDR4-3200 | 1.6 GHz | Adequate |
| PCIe Gen3 (8 GT/s) | 4 GHz | Marginal (short traces only) |
| PCIe Gen4 (16 GT/s) | 8 GHz | Inadequate for > 150 mm |
| PCIe Gen5 (32 GT/s) | 16 GHz | Unsuitable |
| 25G SerDes (NRZ) | 12.5 GHz | Unsuitable |

**Rule of thumb:** Standard FR4 is generally suitable for NRZ interfaces up to 5 Gb/s on traces under 100 mm. Above 10 Gb/s or trace lengths exceeding 150 mm, low-loss materials should be evaluated.

---

## Tier 2: Intermediate

### Q5. Describe the properties of Megtron 6 and IS680. How do they compare to FR4 and to each other?

**Answer:**

**Megtron 6 (Panasonic):**

Megtron 6 is a glass-reinforced resin laminate with a proprietary low-loss resin system. It is one of the industry standard choices for PCIe Gen4/Gen5, 100GbE, and high-speed SerDes boards.

| Parameter | Megtron 6 | Megtron 6 (GHz range) |
|---|---|---|
| $D_k$ | 3.7 | Stable 3.62–3.74 from 1 GHz to 10 GHz |
| $D_f$ | 0.002 | 0.002 at 1 GHz; rises to ~0.004 at 10 GHz |
| $T_g$ | >200°C | Low CTE |
| Copper foil options | VLP, HVLP, RTF | Low roughness foil available |
| Typical cost vs FR4 | 3–5× more expensive | Premium laminate |

The key advantage of Megtron 6 is its very low $D_f$, approximately 5–10x lower than FR4 at 10 GHz. Insertion loss for a 200 mm stripline trace at 10 GHz is approximately 2 dB (Megtron 6) versus 8–10 dB (FR4).

**IS680 (Isola Group):**

IS680 is a competitive low-loss laminate from Isola, targeting similar applications to Megtron 6.

| Parameter | IS680 |
|---|---|
| $D_k$ at 2 GHz | 3.77 |
| $D_f$ at 2 GHz | 0.004 |
| $D_f$ at 10 GHz | ~0.006–0.008 |
| $T_g$ | >180°C |
| Cost vs FR4 | 2–4× more expensive |

IS680 has somewhat higher $D_f$ than Megtron 6 but is notably less expensive. It is appropriate for PCIe Gen3/Gen4 applications where the loss budget is not as tight as Gen5 or 100GbE.

**Comparison table:**

| Property | FR4 (std) | IS680 | Megtron 6 | Rogers 4003C |
|---|---|---|---|---|
| $D_k$ at 10 GHz | 4.2 | 3.77 | 3.64 | 3.55 |
| $D_f$ at 10 GHz | 0.022 | 0.007 | 0.004 | 0.0027 |
| $T_g$ | 130–170°C | 180°C | 200°C | >280°C |
| Relative cost | 1× | 2–4× | 3–5× | 5–10× |
| Processability | Excellent | Good | Good | Requires care |
| Common use | General purpose | PCIe Gen3/4 | PCIe Gen5, 100GbE | RF/microwave, mmWave |

**When Megtron 6 is mandatory (not just preferred):**

- PCIe Gen5 traces longer than 100 mm
- 100GbE CAUI-4 or CAUI-10 on-board links
- 25GBASE-T or 56G PAM4 routing
- Any application where insertion loss from L1 to L2 (S21) must stay within 10 dB at the Nyquist frequency over the full trace length

---

### Q6. Explain why $D_k$ and $D_f$ both vary with frequency. What mechanisms drive this variation?

**Answer:**

The frequency dependence of $D_k$ and $D_f$ arises from the physical process of **dielectric relaxation** — the delay in the response of polarisable molecules to a time-varying electric field.

**Polarisation mechanisms in PCB dielectrics:**

PCB epoxy-glass laminates contain several species of polarisable groups:

1. **Electronic polarisation:** Electron cloud displacement around nuclei. This mechanism is extremely fast (response in the UV range), contributing essentially constant, low $\epsilon'$ across the GHz range.

2. **Ionic polarisation:** Relative displacement of ions in the lattice. Fast, but slower than electronic. Mostly stable across microwave frequencies.

3. **Orientational (dipolar) polarisation:** Reorientation of permanent dipole moments (e.g., hydroxyl –OH groups and carbonyl C=O groups in epoxy resin). This mechanism has a characteristic relaxation frequency $f_0$ in the GHz range for typical epoxies.

**Debye relaxation model:**

The complex permittivity follows a Debye-like response:

$$\epsilon(f) = \epsilon_{\infty} + \frac{\epsilon_s - \epsilon_{\infty}}{1 + j(f/f_0)}$$

where $\epsilon_s$ is the static (low-frequency) permittivity, $\epsilon_\infty$ is the high-frequency limit, and $f_0$ is the relaxation frequency.

Separating real and imaginary parts:

$$\epsilon'(f) = \epsilon_{\infty} + \frac{\epsilon_s - \epsilon_{\infty}}{1 + (f/f_0)^2}$$

$$\epsilon''(f) = \frac{(\epsilon_s - \epsilon_{\infty})(f/f_0)}{1 + (f/f_0)^2}$$

**Observable consequences:**

- $D_k = \epsilon'(f)$ **decreases** with frequency as the slower dipolar polarisations can no longer follow the field. For FR4, $D_k$ drops from ~4.8 at 1 MHz to ~4.1 at 10 GHz.

- $D_f = \epsilon''/\epsilon'$ **increases** with frequency in the sub-relaxation regime and peaks near $f_0$. For FR4, $D_f$ approximately doubles from 1 GHz to 10 GHz.

**Practical implication for simulation:**

Using a single constant $D_k/D_f$ value (typically the 1 GHz datasheet value) to simulate a 28 GHz channel will produce an optimistic result. The simulator should use frequency-dependent Wideband Debye or Djordjevic-Sarkar material models, which fit measurement data across the full frequency band. Most 2.5D electromagnetic solvers (Sigrity, HFSS) support these models natively.

**Measurement method:** $D_k$ and $D_f$ vs. frequency are measured using a split-post dielectric resonator (SPDR) or strip-line resonator method, calibrated across multiple frequencies. The IPC-TM-650 test methods 2.5.5.5 (dielectric constant) and 2.5.5.3 (dissipation factor) define the standard procedures.

---

### Q7. What is the glass weave effect and how does it cause differential skew on a PCB?

**Answer:**

**What is glass weave?**

PCB laminates use woven fiberglass fabric as mechanical reinforcement. The fabric consists of glass fibre bundles (yarns) woven in an orthogonal pattern. Standard fabric styles include 106, 1080, 2116, and 7628, each with different yarn counts, bundle sizes, and weave densities.

**The glass weave effect:**

The woven fabric creates a spatially periodic variation in dielectric constant. Where a glass yarn bundle passes under a trace, the effective $D_k$ is higher (glass $D_k \approx 6$). Where the trace is over an epoxy resin pocket (the void between yarn bundles), the effective $D_k$ is lower (resin $D_k \approx 3.0$–$3.5$).

For a differential pair routed with trace spacing of 150 µm and yarn bundle pitch of ~800 µm (typical of 2116 style), the two traces may lie at different positions relative to the weave pattern:

```
Positive trace (+): centred over a glass yarn bundle  →  D_k_eff ≈ 4.5
Negative trace (−): centred over a resin void         →  D_k_eff ≈ 3.8
```

**Skew calculation:**

The differential propagation delay difference per unit length is:

$$\Delta t_{skew} = \frac{\sqrt{\epsilon_{eff,+}} - \sqrt{\epsilon_{eff,-}}}{c}$$

For the example above:

$$\Delta t_{skew} = \frac{\sqrt{4.5} - \sqrt{3.8}}{3 \times 10^{11}} \approx \frac{2.121 - 1.949}{3 \times 10^{11}} \approx 5.7 \text{ ps/100mm}$$

Over a 200 mm trace: ~11 ps of intra-pair skew. At 10 Gb/s NRZ (UI = 100 ps), this is 11% of the UI — significant.

**Mitigation strategies:**

1. **Route differentials at 45° to the weave:** The IPC-2141A standard recommends routing differential pairs at 45° (or any non-0°/90° angle) to the glass weave direction. Each trace then samples the same alternating glass/resin pattern, equalising their average $D_k$.

2. **Use spread-weave or flat-weave prepregs:** Panasonic Megtron 6 is available with spread-glass fabric where the fibre bundles are mechanically spread before impregnation. This creates a more uniform $D_k$ distribution and reduces the spatial period of the variation. Spread glass reduces glass weave skew by approximately 3–5×.

3. **Use very fine weave styles:** Fabric style 106 has the tightest weave (most glass bundles per inch) and smallest bundle pitch. Traces averaged over many bundles see a more uniform composite $D_k$.

4. **Use random-weave reinforcement:** Some PTFE-based laminates use random non-woven glass reinforcement, eliminating the periodic structure entirely.

**Why this matters at high speed:**

For 25G/56G/112G SerDes, the allowable intra-pair skew is typically 5–10% of the unit interval. At 112 Gb/s PAM4 (UI = 8.9 ps per PAM4 symbol), even 1–2 ps of glass-weave-induced skew is a concern. At these speeds, spread-weave low-loss laminates are essentially mandatory.

---

### Q8. How does copper surface roughness affect insertion loss at high frequencies, and what copper foil grades are available?

**Answer:**

**Skin effect and roughness interaction:**

At high frequencies, current flows in a skin layer of depth $\delta_s$. The skin depth at 10 GHz on copper is:

$$\delta_s = \sqrt{\frac{\rho}{\pi f \mu_0}} = \sqrt{\frac{1.72 \times 10^{-8}}{\pi \times 10^{10} \times 4\pi \times 10^{-7}}} \approx 0.66 \text{ µm}$$

Standard electrodeposited (ED) copper foil has a treatment side with surface roughness (RMS) $R_q$ of 1–3 µm — comparable to or larger than the skin depth. Current must travel around the rough peaks and valleys, increasing the effective path length and therefore the effective resistance per unit length.

**Roughness models:**

The **Huray model** represents the roughness as a distribution of hemispherical copper "cannonballs" on the flat base surface:

$$K_{Huray} = 1 + \frac{A_{spheres}}{A_{flat}} \left[1 - e^{-2\delta_s/r}\right]$$

where $r$ is the sphere radius. The correction factor $K_{Huray} > 1$ multiplies the smooth-surface conductor loss.

For rough ED foil at 10 GHz: $K_{Huray} \approx 1.8$–$2.5$ (80–150% increase in conductor loss).
For very low profile (VLP) foil at 10 GHz: $K_{Huray} \approx 1.1$–$1.3$.

**Copper foil grades:**

| Grade | Abbreviation | RMS roughness | Loss increase at 10 GHz |
|---|---|---|---|
| Standard electrodeposited | STD / HTE | 3–6 µm | 2–3× |
| Low profile | LP | 1–3 µm | 1.5–2× |
| Very low profile | VLP | 0.5–1.5 µm | 1.2–1.5× |
| Hyper very low profile | HVLP | 0.3–0.8 µm | 1.1–1.2× |
| Reverse-treated foil | RTF | 0.5–1.2 µm (bond side) | 1.2–1.4× |

**RTF (Reverse-Treated Foil):** In standard ED copper, the rough treatment is on the bond side (facing the resin). In RTF, the treatment is on the opposite face, so the signal-carrying smooth side faces the dielectric. This can slightly reduce the roughness seen by the current at the copper-dielectric interface.

**Practical impact example — 200 mm trace at 10 GHz:**

With standard HTE copper: conductor loss ≈ 4 dB (200 mm). With HVLP copper: ≈ 1.6 dB — a 2.4 dB saving. Combined with Megtron 6 dielectric (saving ~4 dB vs FR4), the total improvement vs a "worst case" FR4 / HTE board is approximately 6.4 dB — the difference between a link working or failing.

---

## Tier 3: Advanced

### Q9. You are designing a 112 Gb/s PAM4 backplane channel. Characterise the material requirements, quantify the dielectric and conductor loss budgets, and specify the laminate and copper foil.

**Answer:**

**Channel requirements:**

112 Gb/s PAM4 NRZ carries 2 bits per symbol at 56 GBaud. Nyquist frequency:

$$f_{Nyquist} = \frac{56 \times 10^9}{2} = 28 \text{ GHz}$$

IEEE 802.3ck defines the channel insertion loss mask. For a chip-to-chip on-board channel (not backplane), the typical allowable IL is −28 dB at 28 GHz (budget varies by topology — see CEI-112G-VSR, MR specifications).

For a backplane (connector-to-connector), the budget is tighter: the total channel from ASIC package pin to ASIC package pin must meet the host specification, with the PCB channel typically allocated 15–20 dB of insertion loss at 28 GHz.

Assume: **Maximum PCB trace loss = 18 dB at 28 GHz** for a 300 mm trace.

**Loss budget per unit length:**

$$\text{Max loss/length} = \frac{18 \text{ dB}}{300 \text{ mm}} = 0.060 \text{ dB/mm} = 60 \text{ dB/m at 28 GHz}$$

**Conductor loss contribution:**

Conductor loss scales as $\sqrt{f}$. Normalised to 1 GHz baseline and accounting for a target trace width of ~75 µm (50 Ω stripline on low-$D_k$ material):

$$\alpha_c(28 \text{ GHz}) \approx \alpha_c(1 \text{ GHz}) \times \sqrt{28} \approx \alpha_c(1 \text{ GHz}) \times 5.29$$

For a 75 µm wide, 18 µm thick copper trace:
$$\alpha_c(1 \text{ GHz}) \approx 0.35 \text{ dB/100mm}$$
$$\alpha_c(28 \text{ GHz}) \approx 0.35 \times 5.29 \approx 1.85 \text{ dB/100mm}$$

With surface roughness correction (HVLP foil, $K \approx 1.15$ at 28 GHz):
$$\alpha_c(28 \text{ GHz, HVLP}) \approx 1.85 \times 1.15 \approx 2.13 \text{ dB/100mm}$$

For 300 mm: $\alpha_c = 6.4$ dB conductor loss.

**Remaining budget for dielectric loss:**

$$\alpha_d \le 18 - 6.4 = 11.6 \text{ dB over 300 mm}$$

$$\alpha_d \le \frac{11.6}{300} = 0.0387 \text{ dB/mm} \text{ at 28 GHz}$$

**Required $D_f$:**

Using the dielectric loss formula:

$$\alpha_d = 27.3 \frac{\sqrt{\epsilon_r} \cdot f}{c} \cdot D_f$$

$$0.0387 \frac{\text{dB}}{\text{mm}} = 27.3 \times \frac{\sqrt{3.7} \times 28 \times 10^9}{3 \times 10^{11} \text{ mm/s}} \times D_f$$

$$D_f \le \frac{0.0387}{27.3 \times \frac{\sqrt{3.7} \times 28}{300}} = \frac{0.0387}{27.3 \times \frac{1.924 \times 28}{300}} = \frac{0.0387}{27.3 \times 0.1797} \approx \frac{0.0387}{4.90} \approx 0.0079$$

**Required $D_f \leq 0.008$ at 28 GHz.**

**Material selection:**

| Material | $D_f$ at 28 GHz (approx) | Meets budget? |
|---|---|---|
| FR4 (standard) | ~0.025 | No — 3× over limit |
| IS680 | ~0.010 | Marginal |
| Megtron 6 | ~0.005 | Yes — with margin |
| Rogers 4003C | ~0.003 | Yes — comfortable margin |

**Specification:**

- **Laminate:** Megtron 6 (Panasonic R-5775G) or Rogers 4003C for highest-speed lanes
- **Copper foil:** HVLP (RMS roughness < 0.5 µm) — essential; standard ED foil would add ~3–4 dB at 28 GHz
- **Trace geometry:** Symmetric stripline, 75 µm width, 100 µm dielectric above and below (dual GND planes)
- **Via treatment:** Backdrilled to < 200 µm stub — via stub resonance at 28 GHz with a 1 mm stub would be ~100 GHz (acceptable), but connector launch vias on thick boards need careful management
- **Glass weave:** Spread-weave or 106-style fabric to minimise differential skew; route differentials at 45° to weave direction

**Design margin:**

With Megtron 6 and HVLP copper:
- Conductor loss (300 mm): 6.4 dB
- Dielectric loss (300 mm, $D_f = 0.005$): 7.3 dB
- Total: 13.7 dB vs budget of 18 dB → **4.3 dB margin**

This margin covers connector insertion loss (~1.5 dB per mated pair at 28 GHz for high-speed backplane connectors such as Amphenol Crossbow or Molex iPass+), via transitions (~0.5 dB each), and manufacturing tolerance.

---

### Q10. Explain the Djordjevic-Sarkar (D-S) model and why it is used in simulation of PCB dielectrics. What error does using a constant-$D_k$/$D_f$ model introduce at 28 GHz?

**Answer:**

**The Djordjevic-Sarkar model:**

The D-S model is a broadband, causal, Kramers-Kronig-consistent model for dielectric materials. It represents the complex permittivity as a superposition of Debye relaxation terms distributed across a frequency range $[f_1, f_2]$:

$$\epsilon(f) = \epsilon_\infty + \frac{\Delta\epsilon}{\ln(f_2/f_1)} \cdot \ln\left(\frac{f_2 + jf}{f_1 + jf}\right)$$

where:
- $\epsilon_\infty$ is the high-frequency limit of $\epsilon'$
- $\Delta\epsilon = \epsilon_s - \epsilon_\infty$ is the relaxation amplitude
- $f_1$ and $f_2$ define the distribution frequency window (typically $10^3$ to $10^{18}$ Hz)
- $jf = j2\pi f$ in the exponent (symbolic notation)

**Why D-S is preferred over constant-property models:**

1. **Causality:** The D-S model satisfies the Kramers-Kronig relations, which require that the real and imaginary parts of any physical response function are Hilbert transform pairs. A constant-$D_k$ / constant-$D_f$ model violates causality in the time domain, producing non-physical pre-cursor artifacts in SPICE time-domain simulations.

2. **Frequency accuracy:** The D-S model matches measured $D_k(f)$ and $D_f(f)$ data across the full simulation bandwidth from DC to millimetre wave. A single-point datasheet $D_k = 4.2$, $D_f = 0.02$ describes the material only at one frequency (typically 1 GHz).

3. **Pass-band group delay accuracy:** Because $D_k$ decreases with frequency, the phase velocity increases with frequency. This creates a dispersive transmission line. The D-S model captures this dispersion correctly; a constant-$D_k$ model does not.

**Quantified error — constant vs. D-S at 28 GHz (FR4 example):**

| Quantity | Constant model (1 GHz values) | D-S model | Error |
|---|---|---|---|
| $D_k$ at 28 GHz | 4.2 (forced constant) | 3.9 (actual) | +7.7% |
| Propagation velocity error | — | 3.8% too slow | Timing error |
| $D_f$ at 28 GHz | 0.020 (forced constant) | 0.025 (actual) | -20% |
| Dielectric loss (300 mm) | 7.3 dB | 9.1 dB | -1.8 dB (optimistic) |
| Total insertion loss error | — | — | ~2 dB optimistic |

A 2 dB error in a channel with an 18 dB budget is an 11% error — sufficient to cause an unexpected compliance failure in silicon bring-up. This is a common reason why simulation-to-measurement correlation fails when single-point material models are used for high-speed channels.

**Practical requirement:** Any simulation of a channel operating above 10 GHz should use a D-S or equivalent Wideband Debye material model, populated with $D_k(f)$ / $D_f(f)$ measurement data from the laminate manufacturer across the frequency range of interest. The IPC-2141A material measurement test structures enable extraction of broadband material parameters from test coupons on the production panel.

---

## Quick Reference: Material Properties Summary

| Material | $D_k$ at 10 GHz | $D_f$ at 10 GHz | Typical use |
|---|---|---|---|
| FR4 (standard) | 4.2 | 0.022 | < 5 Gb/s, general purpose |
| FR4 (low-halogen) | 4.1–4.3 | 0.018–0.022 | Slightly improved |
| IS680 (Isola) | 3.77 | 0.007 | PCIe Gen3/4, USB 3.x |
| Megtron 6 (Panasonic) | 3.64 | 0.004 | PCIe Gen5, 100GbE, 56G |
| Megtron 7 (Panasonic) | 3.37 | 0.002 | 112G, 400GbE |
| Rogers 4003C | 3.55 | 0.0027 | RF/microwave, mmWave |
| Rogers 4350B | 3.66 | 0.0037 | RF, phased arrays |
| PTFE (Taconic TLX) | 2.45 | 0.0019 | mmWave, antenna boards |

**Key design rules:**
- Use $D_f \leq 0.005$ for interfaces above 10 GHz Nyquist
- Specify spread-weave glass fabric for differential pairs above 25 Gb/s
- Use HVLP copper for all channels above 14 GHz Nyquist
- Always use frequency-dependent (D-S) material models in simulation above 5 GHz
