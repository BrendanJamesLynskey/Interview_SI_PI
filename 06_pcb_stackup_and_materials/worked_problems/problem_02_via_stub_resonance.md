# Problem 02: Via Stub Resonance

## Problem Statement

You are reviewing the signal integrity of a PCIe Gen4 (16 GT/s) design on a 12-layer board. A SerDes differential pair enters the board from a cage-mount SFP+ connector footprint on Layer 1 (top surface) and routes on Layer 4 as a buried stripline.

**Board and via specifications:**

- Total board thickness: 2.0 mm
- Dielectric material: Megtron 6, $\epsilon_r = 3.7$
- Layer count: 12 layers, approximately equal spacing
- Via drill diameter: 0.25 mm (10 mil)
- Via pad diameter: 0.45 mm (18 mil)
- Via antipad diameter: 0.85 mm (33.5 mil)
- Signal via connects L1 pad to L4 pad
- Via plating thickness: 25 µm (1 mil)
- Copper foil weight: ½ oz (18 µm) on all layers

**Questions:**

1. Calculate the via stub length (the unused via barrel below L4).
2. Calculate the first resonant frequency of the un-drilled stub.
3. Determine whether the stub will cause a problem for PCIe Gen4 (state the criterion used).
4. Calculate the minimum insertion loss contribution from the stub at the PCIe Gen4 Nyquist frequency (8 GHz), using the shunt-stub approximation.
5. Determine the required backdrill target stub length to reduce the stub's insertion loss penalty to less than 0.3 dB at 8 GHz.
6. Calculate the new resonance frequency after backdrilling to the target length.

---

## Worked Solution

### Step 1 — Via Stub Length Calculation

**Layer spacing:**

With 12 layers and 2.0 mm total board thickness, the average layer-to-layer dielectric spacing (between copper layers) is:

$$\Delta z_{avg} = \frac{2.0 \text{ mm}}{12 - 1} = \frac{2.0}{11} \approx 0.182 \text{ mm per inter-layer gap}$$

**Via useful length (L1 to L4):**

The signal exits at Layer 4, so the via barrel is electrically used from L1 to L4, spanning 3 layer gaps:

$$l_{useful} = 3 \times 0.182 = 0.545 \text{ mm}$$

**Via stub length (L4 to L12, spanning 8 layer gaps):**

$$l_{stub} = 8 \times 0.182 = 1.455 \text{ mm}$$

**Total via length check:**

$$l_{useful} + l_{stub} = 0.545 + 1.455 = 2.0 \text{ mm} \checkmark$$

---

### Step 2 — First Resonant Frequency of the Un-Drilled Stub

The via stub is an open-circuited transmission-line stub. It presents a short circuit (zero impedance, maximum reflection) at the quarter-wave resonance:

**Phase velocity in the via dielectric:**

$$v_p = \frac{c}{\sqrt{\epsilon_{r,eff}}}$$

The effective permittivity of the via environment is approximately the bulk laminate value. For Megtron 6, $\epsilon_r = 3.7$:

$$v_p = \frac{3 \times 10^{11} \text{ mm/s}}{\sqrt{3.7}} = \frac{3 \times 10^{11}}{1.924} = 1.559 \times 10^{11} \text{ mm/s} = 155.9 \text{ mm/ns}$$

**First resonance ($\lambda/4$):**

$$f_{res,1} = \frac{v_p}{4 \times l_{stub}} = \frac{155.9 \text{ mm/ns}}{4 \times 1.455 \text{ mm}} = \frac{155.9}{5.82} = 26.8 \text{ GHz}$$

**The first stub resonance occurs at approximately 26.8 GHz.**

Higher-order resonances occur at odd harmonics: $3 \times 26.8 = 80.4$ GHz, $5 \times 26.8 = 134$ GHz, etc.

---

### Step 3 — PCIe Gen4 Stub Impact Assessment

**PCIe Gen4 Nyquist frequency:**

$$f_{Nyq} = \frac{16 \text{ GT/s}}{2} = 8 \text{ GHz}$$

**Assessment criterion:**

The stub resonance should be at least $3 \times f_{Nyq}$ from the Nyquist frequency to avoid significant signal band degradation. Some sources use a tighter criterion of $f_{res} > 4 \times f_{Nyq}$ for margin.

$$\frac{f_{res,1}}{f_{Nyq}} = \frac{26.8}{8} = 3.35$$

The ratio is 3.35 — marginally above the $3 \times$ threshold but below the more conservative $4 \times$ threshold.

**Assessment:** The stub resonance is outside the primary PCIe Gen4 signal band. However, with $f_{res}/f_{Nyq} = 3.35$, the below-resonance stub loading at 8 GHz is not negligible. The stub creates a reactive discontinuity below its resonance frequency that contributes additional insertion loss through capacitive loading. This must be quantified (Step 4) rather than dismissed.

**Additionally:** PCIe Gen4 uses 128b/130b encoding with a PRBS-based test pattern. The IHV-defined compliance channel must meet $|S_{dd21}| \leq -10$ dB at 8 GHz. Stub loss is additive to conductor loss and dielectric loss in the total channel budget. A 1–2 dB stub contribution on a channel with a 10 dB budget could be the difference between passing and failing compliance.

---

### Step 4 — Insertion Loss from Stub at 8 GHz

**Via stub characteristic impedance:**

The stub is a section of via barrel. Using the coaxial approximation:

$$Z_{via} = \frac{60}{\sqrt{\epsilon_r}} \ln\left(\frac{D_{antipad}}{D_{barrel}}\right)$$

The finished hole diameter is the drill diameter minus plating allowance:

$$D_{barrel} = D_{drill} - 2 \times t_{plating}? \quad \text{No — } D_{barrel} \text{ is the drilled hole after plating}$$

Actually: the drill bit diameter is 0.25 mm; the plated finished hole is approximately $0.25 - 2 \times 0.025 = 0.20$ mm (the plating fills inward). The antipad is 0.85 mm.

$$Z_{via} = \frac{60}{\sqrt{3.7}} \ln\left(\frac{0.85}{0.20}\right) = 31.18 \times \ln(4.25) = 31.18 \times 1.447 = 45.1 \text{ Ω}$$

**Shunt-stub admittance at 8 GHz:**

The electrical length of the stub at 8 GHz:

$$\theta = \frac{2\pi f l_{stub}}{v_p} = \frac{2\pi \times 8 \times 10^9 \times 1.455 \times 10^{-3}}{1.559 \times 10^{11}} = \frac{2\pi \times 11.64 \times 10^6}{1.559 \times 10^{11}}$$

Wait — keeping consistent units (GHz and mm, with $v_p = 155.9$ mm/ns):

$$\theta = \frac{2\pi f [\text{GHz}] \times l_{stub} [\text{mm}]}{v_p [\text{mm/ns}]} \times 10^{-9+9} = \frac{2\pi \times 8 \times 1.455}{155.9} = \frac{73.19}{155.9} = 0.4695 \text{ rad}$$

The stub admittance (open-circuited stub, shunt-connected):

$$Y_{stub} = \frac{j}{Z_{via}} \tan(\theta) = \frac{j}{45.1} \tan(0.4695) = \frac{j}{45.1} \times 0.508 = j \times 0.01127 \text{ S}$$

**Voltage transmission coefficient ($Z_0 = 50$ Ω):**

$$S_{21} = \frac{1}{1 + Y_{stub} \cdot Z_0 / 2} = \frac{1}{1 + j \times 0.01127 \times 25} = \frac{1}{1 + j \times 0.2817}$$

$$|S_{21}| = \frac{1}{\sqrt{1 + 0.2817^2}} = \frac{1}{\sqrt{1 + 0.07935}} = \frac{1}{\sqrt{1.07935}} = \frac{1}{1.03891} = 0.9625$$

$$|S_{21}|_{dB} = 20 \log_{10}(0.9625) = -0.332 \text{ dB}$$

**The un-drilled stub contributes approximately 0.33 dB of insertion loss at the PCIe Gen4 Nyquist frequency of 8 GHz.**

This is for a single via transition. A PCIe Gen4 channel typically has two via transitions (transmitter launch + receiver launch). Total stub loss: approximately $2 \times 0.33 = 0.66$ dB from via stubs alone.

In the context of a 10 dB IHV channel loss budget at 8 GHz, a 0.66 dB contribution is significant (6.6% of budget). Combined with connector loss (~1.0–1.5 dB per connector pair at 8 GHz) and trace loss (~3–5 dB for 200 mm of Megtron 6 at 8 GHz), the stubs push the budget further.

**Recommendation:** The stub is marginal. Backdrilling should be evaluated.

---

### Step 5 — Required Backdrill Target Stub Length

**Target:** stub insertion loss $\leq 0.3$ dB at 8 GHz.

From the shunt-stub model, the stub loss in dB is:

$$\alpha_{stub} = -20 \log_{10}\left|\frac{1}{1 + j\tan(\theta) \cdot Z_0 / (2 Z_{via})}\right|$$

For small $\theta$ (stub well below resonance): $\tan(\theta) \approx \theta = 2\pi f l_{stub}/v_p$, and:

$$\alpha_{stub} \approx 10 \log_{10}\left(1 + \left(\frac{\pi f l_{stub} Z_0}{v_p Z_{via}}\right)^2\right)$$

Setting $\alpha_{stub} = 0.3$ dB:

$$1 + \left(\frac{\pi \times 8 \times l_{stub} \times 50}{155.9 \times 45.1}\right)^2 = 10^{0.03} = 1.0715$$

$$\left(\frac{1256.6 \times l_{stub}}{7031}\right)^2 = 0.0715$$

$$\frac{1256.6 \times l_{stub}}{7031} = \sqrt{0.0715} = 0.2674$$

$$l_{stub} = \frac{0.2674 \times 7031}{1256.6} = \frac{1880}{1256.6} = 1.496 \text{ mm}$$

Hmm — this is slightly larger than our current 1.455 mm stub. That means the un-drilled stub loss (0.33 dB) is above our 0.3 dB target, but only just. Backdrilling to 1.4 mm would give the same result. Let us instead solve more carefully with the exact formula at 8 GHz.

**Target: $l_{stub}$ such that $|S_{21}|^2$ gives loss $\leq 0.3$ dB.**

$10^{0.3/10} - 1 = 10^{0.03} - 1 = 0.0715$.

$$\left(\frac{\pi f l_{stub} Z_0}{v_p Z_{via}}\right)^2 \leq 0.0715$$

$$l_{stub} \leq \frac{v_p Z_{via}}{\pi f Z_0} \sqrt{0.0715} = \frac{155.9 \times 45.1}{\pi \times 8 \times 50} \times 0.2674$$

$$= \frac{7031}{1256.6} \times 0.2674 = 5.595 \times 0.2674 = 1.496 \text{ mm}$$

The current stub (1.455 mm) is already slightly below this limit. Let us recheck: we computed $|S_{21}|_{dB} = -0.332$ dB, which is 0.032 dB above the 0.3 dB threshold — barely. Using the exact formula:

$$l_{stub,max} = 1.496 \text{ mm for 0.3 dB limit}$$

Since our stub is 1.455 mm, the stub already just barely passes the 0.3 dB criterion. However, the calculation does not include reflection-mode loss (the $S_{11}$ returned energy also reduces $S_{21}$ in the exact two-port sense), so the effective stub loss is slightly higher than the shunt-model approximation.

**Prudent engineering margin:** Apply an additional 50% margin on stub length to account for model uncertainty and to reduce stub loss at the 3rd harmonic (24 GHz, which may be relevant for PCIe Gen4 FEC). The backdrilled target should be:

$$l_{stub,target} = 1.496 \times 0.50 = 0.75 \text{ mm}$$

Hmm — that seems large for a backdrill. Let us use the standard industry practice: backdrill to leave a **0.25 mm (10 mil) residual stub**, which:

- Provides >50% margin on the 0.3 dB loss criterion
- Is achievable within standard fabricator depth tolerance of ±50 µm
- Eliminates any concern about sub-resonance loading across the full frequency band of interest

**Specified backdrill stub length: 0.25 mm.**

---

### Step 6 — Resonance Frequency After Backdrilling to 0.25 mm

$$f_{res,backdrilled} = \frac{v_p}{4 \times l_{stub,backdrilled}} = \frac{155.9 \text{ mm/ns}}{4 \times 0.25 \text{ mm}} = \frac{155.9}{1.0} = 155.9 \text{ GHz}$$

**After backdrilling, the first stub resonance is at approximately 156 GHz.**

This is well beyond the reach of any PCIe signal generation and detection circuit, and well beyond the useful frequency range of the PCB dielectric model. The via transition is effectively stub-free for all practical signal bands.

**Insertion loss from 0.25 mm backdrilled stub at 8 GHz:**

$$\theta_{new} = \frac{2\pi \times 8 \times 0.25}{155.9} = \frac{12.57}{155.9} = 0.0807 \text{ rad}$$

$$\tan(\theta_{new}) = 0.0808$$

$$|S_{21}| = \frac{1}{\sqrt{1 + (0.0808 \times 50 / (2 \times 45.1))^2}} = \frac{1}{\sqrt{1 + (0.0895)^2}} = \frac{1}{\sqrt{1.008}} = 0.9960$$

$$|S_{21}|_{dB} = 20\log_{10}(0.9960) = -0.035 \text{ dB}$$

**Stub loss after backdrilling: 0.035 dB — negligible.**

---

### Summary of Results

| Parameter | Value |
|---|---|
| Stub length (un-drilled) | 1.455 mm |
| First resonance (un-drilled) | 26.8 GHz |
| $f_{res} / f_{Nyq}$ (un-drilled) | 3.35 |
| Stub loss at 8 GHz (un-drilled) | 0.33 dB |
| Target backdrill stub length | 0.25 mm (10 mil) |
| First resonance after backdrilling | 156 GHz |
| Stub loss at 8 GHz after backdrilling | 0.035 dB |
| Loss improvement from backdrilling | 0.30 dB per via |
| Two-via PCIe channel improvement | ~0.60 dB |

---

### Common Interview Pitfalls

**Forgetting to account for effective $\epsilon_r$ in the via:** Using $c$ directly (without $\sqrt{\epsilon_r}$) in the resonance formula overpredicts the resonance frequency by a factor of $\sqrt{3.7} \approx 1.92$ — a factor of nearly 2 error.

**Assuming the stub is always harmful:** A 26.8 GHz resonance on a PCIe Gen4 channel (8 GHz Nyquist) is not creating a resonance in-band. The harm is sub-resonance reactive loading (0.33 dB). Whether this requires correction depends on the total channel budget. Always calculate the actual loss contribution before mandating backdrilling.

**Specifying backdrill depth by total depth rather than stub length:** Fabricators execute backdrilling to a specified depth from the back side of the board. The designer must specify the intended stub length (or equivalently, the drill depth from the back face), not just "backdrill the PCIe vias." A clear specification is: "Backdrill signal vias on L4 from the bottom face to within 10 mil (0.25 mm) of the L4 pad centre plane."

**Not checking backdrill depth tolerance:** A ±50 µm tolerance on 0.25 mm stub means the actual stub could be 0.20–0.30 mm, with resonance ranging from 130–195 GHz. This is acceptable. But if the engineer had specified a 0.10 mm target stub, the ±50 µm tolerance would mean the backdrill might accidentally cut into the L4 pad, destroying the via connection. Always leave adequate margin between the backdrill depth and the last active pad.
