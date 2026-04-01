# Via Design and Backdrilling

## Overview

Vias are the transition structures that carry signals between PCB layers. While they appear in every board design, their electrical behaviour becomes a dominant performance limiter at frequencies above a few gigahertz. A via is not simply a wire between layers: it is a complex discontinuity with inductance, capacitance, and resonant behaviour that can create reflections, insertion loss, and cross-coupling between channels. Understanding via modelling, stub resonance, backdrilling, and pad/antipad geometry is essential for any high-speed PCB design role.

---

## Tier 1: Fundamentals

### Q1. Describe the electrical model of a through-hole via. What are the dominant parasitic elements?

**Answer:**

A through-hole via in a multilayer PCB is modelled as a lumped or distributed discontinuity with the following primary parasitic elements:

**1. Via barrel inductance ($L_{via}$):**

The via barrel (the plated cylinder through the board) carries the signal current from one layer to another. It behaves as a short transmission-line section, but at the dimensions involved (barrel length typically 1–2 mm, barrel diameter 0.1–0.3 mm) it can be modelled as a series inductance for frequencies below $\sim$10 GHz:

$$L_{via} \approx \frac{\mu_0 h}{2\pi} \ln\left(\frac{D_2}{D_1}\right)$$

where $h$ is the via height (board thickness), $D_2$ is the antipad diameter, and $D_1$ is the via barrel diameter. For a typical through-hole via in a 1.6 mm board: $L_{via} \approx 0.5$–$1.5$ nH.

**2. Via pad capacitance ($C_{pad}$):**

Each pad ring on a copper layer adds capacitance to ground through the dielectric. For each reference plane layer the via traverses, the antipad opening (the clearance hole in the plane) reduces but does not eliminate this capacitance:

$$C_{pad} \approx \frac{\epsilon_r \epsilon_0 \pi (D_{pad}^2 - D_{drill}^2)}{4 t_{dielectric}}$$

Typical values: $C_{pad} \approx 0.05$–$0.2$ pF per pad.

**3. Via stub (shunt stub):**

For a via that does not use the full board depth — for example, a signal entering on L1 and exiting on L3 in a 10-layer board — the remaining via barrel below L3 acts as an open-circuited shunt stub. This stub is the single most damaging element to high-speed performance and is discussed in detail in later questions.

**Lumped model:**

```
Signal in (L1)
     │
     L_via (series inductance of barrel)
     │
     ├── C_pad (shunt capacitance to GND plane)
     │
Signal out (L3)
     │
     └── L_stub / C_stub (the shunt stub below L3)
```

**Return path via:**

Every signal via must have an adjacent return via (ground via) to close the return current loop. The distance between the signal via and its ground return via determines the effective inductance of the via transition. A signal via with no nearby return via has much higher inductance, creating a larger discontinuity. The ground return via should be placed within $\leq 2 \times$ the board thickness from the signal via.

---

### Q2. What is via stub resonance, and at what frequency does a 1.5 mm stub resonate?

**Answer:**

**Via stub resonance:**

The stub is an open-circuited transmission-line segment attached in shunt at the via transition point. An open-circuited stub of length $l$ presents a short circuit (zero impedance) to the signal line at frequencies where the stub is $\lambda/4$ long:

$$l = \frac{\lambda}{4} = \frac{v_p}{4f} \implies f_{resonance} = \frac{v_p}{4l}$$

At resonance, the stub shunts the signal path to ground, creating a notch in the insertion loss (S21) that can exceed −20 dB. At odd harmonics ($3f$, $5f$, ...) the resonance repeats. Between resonances, the stub adds reactive loading that degrades group delay and creates reflections.

**Phase velocity in the via:**

The via barrel passes through the PCB dielectric. The effective $\epsilon_r$ of the via environment is approximately the board material's bulk $\epsilon_r$, though the actual value depends on drill size and plating. For FR4-class materials, $\epsilon_{r,eff} \approx 4.0$–$4.4$:

$$v_p = \frac{c}{\sqrt{\epsilon_{r,eff}}}$$

**Resonance frequency for a 1.5 mm stub:**

Taking $\epsilon_{r,eff} = 4.2$:

$$v_p = \frac{3 \times 10^{11} \text{ mm/s}}{\sqrt{4.2}} = \frac{3 \times 10^{11}}{2.049} = 1.464 \times 10^{11} \text{ mm/s}$$

$$f_{resonance} = \frac{v_p}{4 \times l_{stub}} = \frac{1.464 \times 10^{11}}{4 \times 1.5} = \frac{1.464 \times 10^{11}}{6} = 2.44 \times 10^{10} \text{ Hz} = 24.4 \text{ GHz}$$

A 1.5 mm stub resonates at **approximately 24 GHz**.

**Is backdrilling needed?**

At 24 GHz Nyquist frequency (112 Gb/s PAM4), this is right at the signal band — the notch falls in the signal band, causing catastrophic loss. However, for PCIe Gen4 (Nyquist = 8 GHz), a 24 GHz resonance is outside the signal band, and the stub is acceptable (though it may still cause performance degradation from reactive loading below resonance).

**Rule of thumb:** Backdrilling is needed when $f_{resonance} \leq 1.5 \times f_{Nyquist}$ or when stub loss contribution at Nyquist exceeds 1–2 dB.

---

### Q3. What is backdrilling (back-drilling)? Describe the process and what it achieves.

**Answer:**

**Backdrilling process:**

Backdrilling is a secondary drilling operation performed after standard board fabrication. A drill bit is lowered from the opposite board face into the existing via hole and removes the via barrel plating (and surrounding dielectric) to a controlled depth. The drill bit diameter is slightly larger than the original via drill (typically 0.1–0.2 mm larger) to cleanly remove the plating from the stub section.

```
Before backdrilling:         After backdrilling:

L1  ●── signal              L1  ●── signal
L2  |                       L2  |
L3  ●── signal exit         L3  ●── signal exit
L4  |                       L4      (plating removed)
... |  ← stub               ...     (stub removed)
L10 ●── bottom of drill     L10 ○── backdrill reaches here
                                     (within target depth)
```

The depth of the backdrill is controlled to leave a residual stub of target length $l_{stub,target}$, typically 8–10 mil (0.2–0.25 mm).

**What backdrilling achieves:**

1. **Eliminates stub resonance:** With $l_{stub} = 0.2$ mm, the resonance frequency becomes:

$$f_{resonance} = \frac{1.464 \times 10^{11}}{4 \times 0.2} = 183 \text{ GHz}$$

This is far beyond any PCB signal frequency, effectively eliminating the resonance from all practical signal bands.

2. **Reduces stub capacitive loading:** Even below resonance, the stub adds shunt capacitance that degrades the S21 and group delay. Removing the stub removes this loading.

3. **Improves return loss (S11):** The stub creates a shunt path that reflects signal energy back toward the source. Removing it improves impedance matching at the via transition.

**Backdrill limitations and cost:**

- Each backdrilled via requires additional drill cycle time. Cost increases approximately 20–40% for backdrilled boards.
- Depth tolerance: ±50–75 µm (2–3 mil) is typical. The resulting stub length is $l_{nominal} \pm \Delta l$, requiring that the stub target includes margin.
- Not all vias can be backdrilled. Vias carrying DC power or those with pads on inner layers below the backdrill depth cannot be drilled.
- Blind and buried vias are alternatives for new designs, but backdrilling is more cost-effective for boards where all vias start as through-holes.

---

## Tier 2: Intermediate

### Q4. Derive the condition for minimum stub length after backdrilling, given a target maximum loss from the stub at the Nyquist frequency.

**Answer:**

**Via stub as a shunt element:**

The open-circuited stub of length $l_{stub}$ presents an admittance at the signal node:

$$Y_{stub}(f) = \frac{j}{Z_{via}} \tan\left(\frac{2\pi f l_{stub}}{v_p}\right)$$

where $Z_{via}$ is the characteristic impedance of the via stub (approximately 40–60 Ω for a typical via geometry in FR4).

**Insertion loss from the stub:**

For a shunt admittance $Y$ on a $Z_0 = 50$ Ω line, the voltage transmission coefficient is:

$$S_{21} = \frac{1}{1 + Y \cdot Z_0 / 2}$$

Taking the magnitude and converting to dB:

$$|S_{21}|_{dB} = -20 \log_{10}\left|1 + \frac{Z_0}{2} \cdot \frac{j}{Z_{via}} \tan(\theta)\right|$$

where $\theta = 2\pi f l_{stub}/v_p$.

For small arguments ($\theta \ll \pi/2$, i.e., well below resonance):

$$\tan(\theta) \approx \theta = \frac{2\pi f l_{stub}}{v_p}$$

$$|S_{21}|_{dB} \approx -20 \log_{10}\left|1 + j\frac{\pi f l_{stub} Z_0}{v_p Z_{via}}\right|$$

For small attenuation:

$$|S_{21}|_{dB} \approx -20 \log_{10}(1 + x^2)^{1/2} \approx -20 \log_{10}(1) - 10 x^2 / \ln(10)$$

where $x = \pi f l_{stub} Z_0 / (v_p Z_{via})$.

For a maximum allowable stub loss of $\alpha_{max}$ dB at Nyquist frequency $f_{Nyq}$:

$$\alpha_{max} = -20 \log_{10}\sqrt{1 + \left(\frac{\pi f_{Nyq} l_{stub} Z_0}{v_p Z_{via}}\right)^2}$$

Solving for $l_{stub}$:

$$l_{stub} \leq \frac{v_p Z_{via}}{\pi f_{Nyq} Z_0} \sqrt{10^{\alpha_{max}/10} - 1}$$

**Numerical example — PCIe Gen5 ($f_{Nyq} = 16$ GHz), $\alpha_{max} = 0.5$ dB, $Z_0 = 50$ Ω, $Z_{via} = 45$ Ω:**

$$l_{stub} \leq \frac{(1.464 \times 10^{11} \text{ mm/s}) \times 45}{pi \times 16 \times 10^9 \times 50} \sqrt{10^{0.05} - 1}$$

$$= \frac{6.588 \times 10^{12}}{2.513 \times 10^{12}} \times \sqrt{0.1220}$$

$$= 2.62 \times 0.349 = 0.914 \text{ mm}$$

This means a stub less than ~0.9 mm contributes less than 0.5 dB loss at 16 GHz Nyquist (before resonance effects). For a 10-layer board with signal on L3 and a total via height of 1.5 mm, the stub length is about 1.05 mm — marginally exceeding this limit. The design choice is then:

- Backdrill to reduce the stub to < 0.9 mm, or
- Move the signal to a deeper layer to reduce the stub length, or
- Accept the slightly higher stub loss and verify the total channel budget still passes.

---

### Q5. How is via impedance calculated, and what are the key geometric parameters?

**Answer:**

**Via as a coaxial structure:**

The via barrel, padstack, and antipads form an approximately coaxial structure. The characteristic impedance can be approximated using a coaxial impedance formula:

$$Z_{via} = \frac{60}{\sqrt{\epsilon_{r,eff}}} \ln\left(\frac{D_{antipad}}{D_{barrel}}\right)$$

where:
- $D_{antipad}$ is the diameter of the clearance hole in the reference plane (antipad diameter)
- $D_{barrel}$ is the finished hole (drill) diameter
- $\epsilon_{r,eff}$ is the effective permittivity of the via environment

**Typical dimensions and impedances:**

| Parameter | Typical value |
|---|---|
| Drill diameter $D_{drill}$ | 0.2–0.5 mm (8–20 mil) |
| Pad diameter $D_{pad}$ | $D_{drill}$ + 0.2 mm (typical annular ring) |
| Antipad diameter $D_{antipad}$ | $D_{drill}$ + 0.5–1.0 mm |
| $\epsilon_{r,eff}$ (FR4) | ~3.8–4.2 |

**Example calculation:**

- $D_{barrel} = 0.25$ mm, $D_{antipad} = 0.75$ mm, $\epsilon_{r,eff} = 4.0$:

$$Z_{via} = \frac{60}{\sqrt{4.0}} \ln\left(\frac{0.75}{0.25}\right) = \frac{60}{2} \times \ln(3) = 30 \times 1.099 = 33 \text{ Ω}$$

This 33 Ω via impedance is significantly below the 50 Ω transmission line impedance. The via transition is therefore **capacitive** (lower impedance → more shunt capacitance). This is the typical case for standard vias.

**Increasing via impedance:**

To raise via impedance toward 50 Ω:
1. **Larger antipad:** Increase $D_{antipad}$ from 0.75 mm to 1.2 mm → $Z_{via}$ increases to approximately 44 Ω. Risk: antipad merging with adjacent vias on dense boards, and reduced plane integrity.
2. **Smaller drill:** Reduce drill diameter from 0.25 mm to 0.15 mm → $Z_{via}$ increases. Limited by aspect ratio ($\leq$ 10:1 drill depth to diameter for reliable plating).
3. **Non-functional pads (NFP):** Remove pad rings on reference plane layers where there is no connection — this allows the antipad opening to be larger without compromising copper-to-copper clearance rules.

**For differential vias:**

Two adjacent signal vias carrying a differential pair form a coupled structure. The differential via impedance $Z_{diff,via}$ depends on the spacing $s$ between the two via centres:

$$Z_{diff,via} \approx 2 Z_{via,single} \times \left(1 - \frac{k_{mutual}}{1 + k_{mutual}}\right)$$

For closely spaced vias ($s = 0.6$ mm, $D_{barrel} = 0.25$ mm), the mutual coupling reduces the differential impedance below $2Z_{via,single}$. Target differential via impedance is 90–100 Ω. A field-solver simulation (HFSS, Sigrity) is needed for accurate differential via characterisation above 5 GHz.

---

### Q6. What is a "via-in-pad" design and when is it used? What are the fabrication requirements?

**Answer:**

**Via-in-pad:**

Via-in-pad (also called via-in-pad plated over, VIPPO) is a design where the via lands directly under a component pad, rather than offset from it with a trace stub connecting the two. The via hole is drilled, plated, filled with epoxy or copper fill, and then plated over with copper — creating a flat, solderable surface flush with the PCB surface.

**When via-in-pad is used:**

1. **Fine-pitch BGAs:** Modern BGAs (ball pitch 0.5 mm, 0.4 mm, 0.35 mm) do not have room for conventional via escape routing between pad rows. Via-in-pad allows each BGA ball to have its own via directly beneath it.

2. **Thermal performance:** In power devices (LDOs, MOSFETs), the central thermal pad is connected to an inner ground plane via an array of filled vias directly under the pad. This provides a very low thermal resistance path $R_{theta}$ to the inner plane acting as a heat spreader.

3. **Stub elimination:** Filled vias are shorter than standard through-holes if designed as blind/buried, or they use backfill to control stub. For high-speed signals, a via-in-pad on L1 connecting to L2 can be implemented as a blind via (drilled L1-to-L2 only), eliminating any stub entirely.

4. **Space efficiency:** Via-in-pad eliminates the via-to-pad trace stub (typically 0.3–0.5 mm long), saving routing space and reducing stub length.

**Fabrication requirements:**

- **Hole fill:** The drilled via must be completely filled (typically with either copper electrofill or non-conductive epoxy fill). A void in the fill material causes solder wicking during reflow, creating a solder-void defect under the pad.
- **Plating over:** After fill and cure, the via is plated over and planarized to create a flat, coplanar solderable surface. The final pad finish (ENIG, HASL, OSP) is applied uniformly.
- **Cost premium:** Via-in-pad adds 20–30% to board fabrication cost due to the fill and plate-over process steps.
- **Thermal via array:** For thermal applications, a recommended fill material is copper electrofill (thermal conductivity $\sim$400 W/m·K) rather than epoxy ($\sim$0.3 W/m·K). Thermal vias should be 0.15–0.3 mm finished diameter, minimum 1 mm² total via copper area for power devices.

---

## Tier 3: Advanced

### Q7. Model a differential via transition from surface to L3 on a 10-layer board. Describe the S-parameter signatures of the un-drilled and backdrilled via, and explain how to verify the model against TDR measurements.

**Answer:**

**Board specification:**

- 10-layer board, 1.4 mm total thickness
- Megtron 6, $\epsilon_r = 3.7$
- Signal enters at L1 (surface connector), routes on L3 (stripline)
- Via geometry: 0.25 mm drill, 0.45 mm pad, 0.85 mm antipad
- L1–L3 thickness: ~0.2 mm (signal via useful length)
- L3–L10 thickness: ~1.2 mm (stub length, un-drilled case)

**Un-drilled via S-parameter signatures:**

1. **S21 (insertion loss):**
   - Notch at stub resonance frequency:
     $$f_{res} = \frac{c/\sqrt{3.7}}{4 \times 1.2 \text{ mm}} = \frac{155.8 \text{ mm/ns}}{4.8 \text{ mm}} = 32.5 \text{ GHz}$$
   - Below resonance: gradual increase in loss due to reactive loading. At 16 GHz (half-resonance frequency), the insertion loss penalty is approximately 1–2 dB from the stub alone.
   - Phase deviation: the stub adds frequency-dependent group delay variation (dispersion), particularly in the band 10–32 GHz.

2. **S11 (return loss):**
   - The stub presents a shunt admittance that degrades return loss. Expect S11 > −15 dB above ~10 GHz.
   - Below resonance, the reflection is capacitive in character (S11 phase rolls clockwise in a Smith chart plot).

3. **Differential mode vs. common mode:**
   - Two closely spaced vias (0.6 mm centre-to-centre) have mutual coupling. The differential S-parameter $S_{dd21}$ shows the stub notch. The mode conversion $S_{cd21}$ (common-to-differential) should be −30 to −40 dB if the via pair is geometrically symmetric.

**Backdrilled via (stub = 0.2 mm) S-parameter signatures:**

1. **S21:** Notch moves to $f_{res} = 155.8 / (4 \times 0.2) = 195$ GHz — outside all practical signal bands. The insertion loss below 40 GHz is dominated by the via inductance and pad capacitance, not the stub. A well-designed backdrilled via shows < 0.5 dB additional loss at 16 GHz Nyquist.

2. **S11:** Improved return loss. The remaining discontinuity is the via inductance-capacitance mismatch. This can be partially compensated by antipad sizing to target $Z_{via} \approx 50$ Ω.

3. **Resonance-free band:** From DC to ~130 GHz, no stub resonance. A clean, well-matched via transition.

**Verification against TDR:**

TDR (Time-Domain Reflectometry) launches a fast-rise-time step and measures reflected voltage versus time.

**Un-drilled via TDR signature:**

```
Voltage
  │         ┌──(line impedance, 50 Ω)
  │──────────│
  │          └──(capacitive dip at via transition — pad capacitance)
  │               └──(inductive rise — via barrel inductance)
  │                    └──(reflection from stub load)
  └──────────────────────────────── Time
```

- A capacitive dip of ~−5 to −10 mV corresponds to the via pad capacitance.
- A subsequent inductive bump of ~+5 mV corresponds to the barrel inductance.
- The stub adds a trailing oscillation at the stub resonance period.

**Backdrilled via TDR signature:**

- The capacitive dip and inductive bump remain (they are intrinsic to the via).
- The stub-related trailing oscillation is gone or much reduced.
- A clean, quickly-settling TDR waveform is the sign of a well-backdrilled, well-designed via transition.

**Model-to-measurement correlation steps:**

1. Extract S-parameters using a 2-port VNA with a GRL (Guided Replication Load) or SOLT calibration to the probe tips, then de-embed the PCB fixture using an ISS (impedance standard substrate) reference.
2. Time-gate the TDR measurement to isolate the via transition from the launch fixture.
3. Export the field-solver model (HFSS or Sigrity 3D) as an S-parameter (Touchstone) file.
4. Overlay simulated vs. measured $|S_{dd21}|$ and $|S_{dd11}|$ on the same frequency axis.
5. Adjust model parameters: $\epsilon_{r,eff}$, $D_f$, drill diameter (within ±5% of nominal), antipad diameter. A good correlation is $|S_{21}|$ within ±0.5 dB and resonance frequency within ±5%.

---

### Q8. A 16-layer board with 2.4 mm total thickness has 25G SerDes signals entering at L1 connectors and routing on L3 and L4. Calculate the stub length for each case, the resonance frequency, and the maximum acceptable stub length to keep the insertion loss penalty below 1 dB at 12.5 GHz Nyquist.

**Answer:**

**Board geometry:**

- Total thickness: 2.4 mm
- Layer spacing (simplified, equal-thickness assumption): $2.4/15 \approx 0.16$ mm per layer
- L1 to L3 distance: $2 \times 0.16 = 0.32$ mm (via useful length for L3 routing)
- L1 to L4 distance: $3 \times 0.16 = 0.48$ mm (via useful length for L4 routing)
- Stub for L3 signal: $2.4 - 0.32 = 2.08$ mm
- Stub for L4 signal: $2.4 - 0.48 = 1.92$ mm

**Via phase velocity (Megtron 6, $\epsilon_r = 3.7$):**

$$v_p = \frac{c}{\sqrt{3.7}} = \frac{300 \text{ mm/ns}}{1.924} = 155.9 \text{ mm/ns}$$

**Stub resonance frequencies:**

For L3 routing (2.08 mm stub):

$$f_{res,L3} = \frac{v_p}{4 \times l_{stub}} = \frac{155.9}{4 \times 2.08} = \frac{155.9}{8.32} = 18.7 \text{ GHz}$$

For L4 routing (1.92 mm stub):

$$f_{res,L4} = \frac{155.9}{4 \times 1.92} = \frac{155.9}{7.68} = 20.3 \text{ GHz}$$

**Nyquist frequency for 25G NRZ:**

$$f_{Nyq} = \frac{25 \text{ Gb/s}}{2} = 12.5 \text{ GHz}$$

Both stubs (18.7 GHz and 20.3 GHz) have resonances above the 12.5 GHz Nyquist, but the ratio $f_{res}/f_{Nyq}$ is 1.50 and 1.62 respectively — these stubs are in the dangerous zone where below-resonance loading causes significant degradation. The IPC recommendation is that stub resonance should be at least $3\times f_{Nyq}$ to avoid signal degradation.

**Maximum acceptable stub length (1 dB loss limit at 12.5 GHz):**

Using the formula from Q4 with $\alpha_{max} = 1.0$ dB, $Z_0 = 50$ Ω, $Z_{via} = 40$ Ω:

$$l_{stub,max} = \frac{v_p Z_{via}}{\pi f_{Nyq} Z_0} \sqrt{10^{\alpha_{max}/10} - 1}$$

$$= \frac{155.9 \times 40}{\pi \times 12.5 \times 10^3 \times 50} \times \sqrt{10^{0.1} - 1}$$

Wait — careful with units. Using mm/ns and GHz:

$$l_{stub,max} = \frac{155.9 \text{ mm/ns} \times 40}{3.1416 \times 12.5 \text{ GHz} \times 50}$$

Converting consistently (1 GHz = 1 ns$^{-1}$):

$$= \frac{6236 \text{ mm/ns}}{1963.5 \text{ ns}^{-1}} \times \sqrt{10^{0.1} - 1}$$

$$= 3.176 \text{ mm} \times \sqrt{0.2589} = 3.176 \times 0.509 = 1.62 \text{ mm}$$

**Conclusion:**

Both stubs (2.08 mm and 1.92 mm) exceed the maximum allowable length of **1.62 mm** for 1 dB stub loss at 12.5 GHz. **Backdrilling is required.**

**Required backdrill depth:**

The backdrill must remove enough of the stub to reduce it to $\leq 1.62$ mm (conservative) or $\leq 0.3$ mm (best practice). Target residual stub after backdrilling: **0.25 mm** (10 mil — typical fabricator achievable depth tolerance with ±50 µm uncertainty).

For L3 signal: backdrill from L16 to depth $2.4 - 0.32 - 0.25 = 1.83$ mm from L1 surface.
For L4 signal: backdrill from L16 to depth $2.4 - 0.48 - 0.25 = 1.67$ mm from L1 surface.

With 0.25 mm residual stub:
$$f_{res,backdrilled} = \frac{155.9}{4 \times 0.25} = 155.9 \text{ GHz}$$

Well above the signal band.

---

## Quick Reference: Via Design Rules

| Parameter | Rule or typical value |
|---|---|
| Backdrill trigger | $f_{res} \leq 3 \times f_{Nyquist}$ |
| Target backdrilled stub | 8–10 mil (0.2–0.25 mm) |
| Return via placement | $\leq 2 \times$ board thickness from signal via |
| Antipad-to-trace clearance | $\geq$ 5 mil (0.127 mm) minimum |
| Via impedance target | 40–60 Ω (aim for $Z_0 \pm 20\%$) |
| Aspect ratio for plating | $\leq$ 10:1 (depth/diameter) |
| Pad annular ring | $\geq$ 75 µm (3 mil) per IPC-6012 Class 3 |
| Via-in-pad fill | Copper electrofill preferred for thermal; epoxy for signal |
| Non-functional pad removal | Recommended on all reference plane layers for backdrilled vias |
| Differential via spacing | 0.5–0.8 mm centre-to-centre for 100 Ω differential target |

**Stub resonance formula:**

$$\boxed{f_{resonance} = \frac{c}{4 \cdot l_{stub} \cdot \sqrt{\epsilon_{r,eff}}}}$$

**Via inductance (rule of thumb):**

$$L_{via} \approx 0.5\text{–}1.5 \text{ nH for a } 1.6 \text{ mm board thickness}$$

**Cannonball/Huray roughness correction factor:** $K \approx 1.1$–$2.5$ at 10 GHz depending on copper grade; use electromagnetic field solver or $K$-factor models for simulation above 5 GHz.
