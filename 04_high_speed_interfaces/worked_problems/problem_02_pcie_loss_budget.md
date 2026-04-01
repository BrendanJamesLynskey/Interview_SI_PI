# Problem 02: Building a PCIe Gen 4 and Gen 5 Channel Loss Budget

## Problem Statement

You are designing a PCIe x16 add-in card (AIC) slot for a server motherboard. The host CPU's PCIe root complex is at one end of the board; the PCIe x16 slot connector is 220 mm away. The design must support both PCIe Gen 4 (16 GT/s) and Gen 5 (32 GT/s) add-in cards, so the channel must pass COM analysis at Gen 5 specifications.

**Given channel components:**

- PCB trace: 220 mm of stripline routing on the motherboard
- CPU package: contributes approximately -1.5 dB at 16 GHz (Gen 5 Nyquist)
- PCIe x16 slot connector: PCIe Gen 5 rated (SAS-PCI-Express-6, or equivalent)
- AIC trace: 50 mm of PCB trace inside the add-in card from slot connector to the PCIe device
- Add-in card connector (gold-finger edge): approximately -0.5 dB at 16 GHz
- Vias: 2 signal transitions on the motherboard (backdrilled) + 2 on the AIC (backdrilled)
- Crosstalk (FEXT) from adjacent PCIe lanes: must be evaluated

**PCB stackup:**

- Motherboard: 16-layer, 3.2 mm thick, Isola Megtron 6 (Dk = 3.70 at 10 GHz, Df = 0.004 at 10 GHz)
- AIC board: 8-layer, 1.6 mm thick, Isola Megtron 6 (same material)

**Routing geometry (motherboard trace, stripline on Layer 5):**

- Trace width: 0.10 mm
- Trace thickness: 18 μm (0.5 oz copper)
- Dielectric height (Layer 4 to Layer 5): 0.10 mm
- Differential pair gap: 0.10 mm

**Tasks:**

1. Compute the motherboard trace insertion loss at 16 GHz (Gen 5 Nyquist) using the Hammerstad-Jensen model for conductor loss and the dielectric loss model.
2. Compute the total channel insertion loss at 16 GHz.
3. Apply the PCIe Gen 5 insertion loss limit and determine if the channel passes.
4. If the channel fails, propose the minimum change required to make it pass, and quantify the improvement.
5. Calculate the same budget for Gen 4 (Nyquist = 8 GHz) and confirm that Gen 4 always passes if Gen 5 passes.

---

## Solution

### Task 1 — Motherboard Trace Insertion Loss at 16 GHz

**Conductor loss (skin-effect dominated):**

The surface resistance per unit length for a copper conductor at frequency $f$ is:

$$R_s(f) = \frac{1}{\sigma \delta_s} = \sqrt{\frac{\pi f \mu_0}{\sigma}}$$

where $\sigma = 5.8 \times 10^7$ S/m for copper and $\mu_0 = 4\pi \times 10^{-7}$ H/m.

At $f = 16$ GHz:

$$\delta_s = \sqrt{\frac{1}{\pi f \mu_0 \sigma}} = \sqrt{\frac{1}{\pi \times 16 \times 10^9 \times 4\pi \times 10^{-7} \times 5.8 \times 10^7}}$$

$$\delta_s = \sqrt{\frac{1}{\pi \times 16 \times 10^9 \times 4\pi \times 10^{-7} \times 5.8 \times 10^7}}$$

Let us compute the denominator:

$$\pi \times 4\pi \times 16 \times 10^9 \times 5.8 \times 10^7 = 4\pi^2 \times 9.28 \times 10^{17} \approx 39.48 \times 9.28 \times 10^{17} \approx 3.664 \times 10^{19}$$

At 1 GHz, copper skin depth is approximately 2.1 μm. Skin depth scales as $1/\sqrt{f}$:

$$\delta_s(16 \text{ GHz}) = \frac{2.1 \text{ μm}}{\sqrt{16}} = \frac{2.1}{4} = 0.525 \text{ μm}$$

For a 0.10 mm wide, 18 μm thick trace, the conductor loss per unit length is approximated by the Hammerstad-Jensen model. A practical formula for conductor loss in dB/mm for a strip conductor at high frequencies is:

$$\alpha_c \approx \frac{R_{sq}}{Z_0 \times w} \times \frac{1}{2} \times 8.686 \text{ dB/Np} \times \frac{1}{\text{m}}$$

Using a practical approximation with the surface resistance:

$$R_{sq}(16 \text{ GHz}) = \frac{1}{\sigma \cdot \delta_s} = \frac{1}{5.8 \times 10^7 \times 0.525 \times 10^{-6}} = \frac{1}{30.45} \approx 0.0329 \text{ Ω/sq}$$

For a single conductor of width $w = 0.10$ mm:

$$R_{conductor} = \frac{R_{sq}}{w} = \frac{0.0329 \text{ Ω/sq}}{0.10 \times 10^{-3} \text{ m}} = 329 \text{ Ω/m}$$

For a differential pair (two conductors, loss shared), and converting to dB/mm:

$$\alpha_c \approx \frac{R_{conductor}}{2 Z_0} \times 8.686 \times 10^{-3} \text{ dB/mm}$$

where $Z_0 = 50$ Ω (single-ended) or 100 Ω (differential — use $Z_{diff}/2 = 50$ Ω effectively for loss):

$$\alpha_c = \frac{329}{2 \times 50} \times 8.686 \times 10^{-3} = 3.29 \times 8.686 \times 10^{-3} \approx 0.0286 \text{ dB/mm}$$

**Roughness correction:** Real PCB copper has surface roughness of 1–4 μm RMS. At 16 GHz with $\delta_s = 0.525$ μm, the surface roughness-to-skin-depth ratio is approximately 2–8, so the Huray or Hammerstad roughness factor $K_{SR}$ is approximately 1.5–2.5. Applying $K_{SR} = 1.8$:

$$\alpha_{c,rough} = 1.8 \times 0.0286 = 0.0515 \text{ dB/mm}$$

**Dielectric loss:**

$$\alpha_d = 27.3 \times \frac{Df \times \sqrt{Dk}}{\lambda_0} \text{ dB/m}$$

where $\lambda_0 = c/f$ is the free-space wavelength. At 16 GHz:

$$\lambda_0 = \frac{3 \times 10^8}{16 \times 10^9} = 0.01875 \text{ m} = 18.75 \text{ mm}$$

For Megtron 6 at 16 GHz: $Df \approx 0.005$, $Dk \approx 3.65$:

$$\alpha_d = 27.3 \times \frac{0.005 \times \sqrt{3.65}}{18.75 \times 10^{-3}} = 27.3 \times \frac{0.005 \times 1.910}{0.01875}$$

$$\alpha_d = 27.3 \times \frac{0.00955}{0.01875} = 27.3 \times 0.5093 \approx 13.90 \text{ dB/m} = 0.01390 \text{ dB/mm}$$

**Total trace loss per mm at 16 GHz:**

$$\alpha_{total} = \alpha_{c,rough} + \alpha_d = 0.0515 + 0.0139 = 0.0654 \text{ dB/mm}$$

**Motherboard trace loss for 220 mm:**

$$IL_{trace,MB} = 220 \times 0.0654 = -14.4 \text{ dB}$$

---

### Task 2 — Total Channel Insertion Loss at 16 GHz

**Compute each component:**

**AIC trace (50 mm, same Megtron 6 material):**

$$IL_{trace,AIC} = 50 \times 0.0654 = -3.3 \text{ dB}$$

**CPU package:**

$$IL_{pkg} = -1.5 \text{ dB} \text{ (given)}$$

**PCIe Gen 5 rated slot connector (motherboard side):**

A Gen 5 rated PCIe connector at 16 GHz: approximately -1.5 to -2.5 dB. Use $IL_{conn,MB} = -2.0$ dB.

**AIC gold-finger edge connector:**

$$IL_{conn,AIC} = -0.5 \text{ dB} \text{ (given)}$$

**Motherboard vias (2 × backdrilled):**

On a 3.2 mm thick board, the via stub after backdrill is designed to be ≤ 0.25 mm. The via transition loss at 16 GHz for a well-designed backdrilled via is approximately -0.4 to -0.7 dB:

$$IL_{vias,MB} = 2 \times (-0.5) = -1.0 \text{ dB}$$

**AIC vias (2 × backdrilled, 1.6 mm board):**

On a 1.6 mm board the stub is smaller; via loss is approximately -0.3 to -0.5 dB each:

$$IL_{vias,AIC} = 2 \times (-0.4) = -0.8 \text{ dB}$$

**Total channel insertion loss:**

$$IL_{total} = IL_{trace,MB} + IL_{trace,AIC} + IL_{pkg} + IL_{conn,MB} + IL_{conn,AIC} + IL_{vias,MB} + IL_{vias,AIC}$$

$$IL_{total} = -14.4 - 3.3 - 1.5 - 2.0 - 0.5 - 1.0 - 0.8 = -23.5 \text{ dB}$$

---

### Task 3 — Compare to PCIe Gen 5 Limit

**PCIe Gen 5 channel insertion loss limit:**

The PCIe Base Specification 5.0 defines the reference channel model (RCM). The maximum insertion loss at Nyquist (16 GHz) for the Gen 5 specification is approximately:

- **Add-in card channel (card + motherboard combined):** $IL_{max} \approx -36 \text{ dB}$ at 16 GHz (from PCIe Gen 5 Annex 4A channel model for the "Long Channel" case)

The calculated $IL_{total} = -23.5$ dB.

**Margin:**

$$Margin = IL_{max} - IL_{total} = -23.5 - (-36) = +12.5 \text{ dB}$$

**Result: The channel passes Gen 5 with a margin of +12.5 dB at Nyquist.**

However, the full COM analysis must also check:
- Return loss ($S_{11}$) — reflections from via transitions, connectors, and impedance steps
- FEXT from adjacent lanes
- Jitter contributions

The comfortable insertion loss margin (+12.5 dB) strongly suggests COM will also pass.

**What if this were on FR4?**

On standard FR4 (Df = 0.022 at 16 GHz, Dk = 4.2, roughness factor ~2.5):

$$\alpha_{d,FR4}(16 \text{ GHz}) \approx 27.3 \times \frac{0.022 \times \sqrt{4.2}}{18.75 \times 10^{-3}} \approx 27.3 \times \frac{0.0451}{0.01875} = 65.6 \text{ dB/m} = 0.0656 \text{ dB/mm}$$

$$\alpha_{c,FR4} \approx 0.0515 \times 2.0 = 0.0657 \text{ dB/mm} \text{ (rougher copper on FR4)}$$

$$\alpha_{total,FR4} \approx 0.131 \text{ dB/mm at 16 GHz}$$

$$IL_{trace,MB,FR4} = 220 \times 0.131 = -28.8 \text{ dB}$$

$$IL_{total,FR4} = -28.8 - 3.3 \times (0.131/0.0654) - 1.5 - 2.0 - 0.5 - 1.0 - 0.8$$
$$= -28.8 - 10.0 - 1.5 - 2.0 - 0.5 - 1.0 - 0.8 = -44.6 \text{ dB}$$

This would be -44.6 dB vs the -36 dB limit → the channel **fails** by 8.6 dB on FR4.

---

### Task 4 — If the Channel Failed: Minimum Fix Required

The current design passes by +12.5 dB, so no fix is needed for this Megtron 6 design. However, let us work through the hypothetical scenario where a cheaper mid-loss laminate is substituted (Df = 0.012 at 16 GHz):

$$\alpha_{d,midloss}(16 \text{ GHz}) = 27.3 \times \frac{0.012 \times 1.910}{0.01875} = 27.3 \times 1.221 = 33.3 \text{ dB/m} = 0.0333 \text{ dB/mm}$$

$$\alpha_{total,midloss} \approx 0.0515 + 0.0333 = 0.0848 \text{ dB/mm}$$

$$IL_{trace,MB,midloss} = 220 \times 0.0848 = -18.7 \text{ dB}$$

$$IL_{total,midloss} = -18.7 - 4.2 - 1.5 - 2.0 - 0.5 - 1.0 - 0.8 = -28.7 \text{ dB}$$

This still passes (-28.7 dB vs -36 dB limit), but with only +7.3 dB margin — less comfortable.

Now suppose the design also had a worse connector (non-Gen5-rated at -4 dB):

$$IL_{total,failing} = -28.7 - (-2.0 + -4.0) = -30.7 \text{ dB at 16 GHz}$$

Add a non-backdrilled via pair: +3 dB worse:

$$IL_{total,failing} = -30.7 - 3.0 = -33.7 \text{ dB}$$

Still passes, but barely (+2.3 dB). Given COM simulation typically adds 1–3 dB crosstalk penalty, this channel would likely fail COM.

**Minimum fix in this failing scenario:**

1. **Upgrade to Gen 5 rated connector:** Saves ~2 dB. Cost: connector upgrade (~$0.50/connector at board quantities).
2. **Backdrill the motherboard vias:** Saves ~2 dB per via pair. Cost: one additional drill pass (~$0.05/board at volume).

These two changes together recover approximately +4 dB — sufficient to pass COM comfortably. The laminate upgrade (to Megtron 6) would be reserved for a more severe deficit.

---

### Task 5 — Gen 4 Loss Budget Confirmation

At Gen 4 (16 GT/s), the Nyquist frequency is $f_N = 8$ GHz.

**Trace loss at 8 GHz (Megtron 6):**

Conductor loss scales as $\sqrt{f}$:

$$\alpha_{c,rough}(8 \text{ GHz}) = \alpha_{c,rough}(16 \text{ GHz}) \times \sqrt{\frac{8}{16}} = 0.0515 \times 0.707 = 0.0364 \text{ dB/mm}$$

Dielectric loss scales linearly with frequency:

$$\alpha_d(8 \text{ GHz}) = \alpha_d(16 \text{ GHz}) \times \frac{8}{16} = 0.0139 \times 0.5 = 0.0070 \text{ dB/mm}$$

$$\alpha_{total}(8 \text{ GHz}) = 0.0364 + 0.0070 = 0.0434 \text{ dB/mm}$$

**Component losses at 8 GHz:**

| Component | Loss at 16 GHz | Loss at 8 GHz |
|---|---|---|
| MB trace 220 mm | -14.4 dB | $220 \times 0.0434 = -9.5$ dB |
| AIC trace 50 mm | -3.3 dB | $50 \times 0.0434 = -2.2$ dB |
| CPU package | -1.5 dB | ~-0.9 dB |
| PCIe slot connector | -2.0 dB | ~-1.2 dB |
| AIC gold-finger | -0.5 dB | ~-0.3 dB |
| MB vias (2×) | -1.0 dB | ~-0.5 dB |
| AIC vias (2×) | -0.8 dB | ~-0.4 dB |
| **Total** | **-23.5 dB** | **-15.0 dB** |

**PCIe Gen 4 channel insertion loss limit at 8 GHz:** approximately -28 dB for a 40 cm reference channel.

$$Margin_{Gen4} = -15.0 - (-28) = +13 \text{ dB}$$

**Confirmation:** Gen 4 passes with even more margin than Gen 5. This is expected: channel insertion loss decreases approximately as $\sqrt{f} + f$ (combined conductor + dielectric), so halving the frequency reduces loss by approximately 3–6 dB on typical laminates. A channel that passes Gen 5 always passes Gen 4 on the same physical channel, confirming backward compatibility.

---

## Summary Table

| Component | Gen 5 loss at 16 GHz | Gen 4 loss at 8 GHz |
|---|---|---|
| Motherboard trace (220 mm, Megtron 6) | -14.4 dB | -9.5 dB |
| AIC trace (50 mm, Megtron 6) | -3.3 dB | -2.2 dB |
| CPU package | -1.5 dB | -0.9 dB |
| PCIe Gen 5 slot connector | -2.0 dB | -1.2 dB |
| AIC gold-finger edge connector | -0.5 dB | -0.3 dB |
| MB backdrilled vias (×2) | -1.0 dB | -0.5 dB |
| AIC backdrilled vias (×2) | -0.8 dB | -0.4 dB |
| **Total** | **-23.5 dB** | **-15.0 dB** |
| **Specification limit** | **-36 dB** | **-28 dB** |
| **Margin** | **+12.5 dB** | **+13.0 dB** |

## Key Learning Points

1. **Low-loss laminate is the most impactful single choice for PCIe Gen 5 on long channels.** Dielectric loss at 16 GHz on FR4 is approximately 5× higher than on Megtron 6, making FR4 impractical for channel lengths > 150 mm at Gen 5.

2. **Conductor roughness correction is significant at Gen 5.** The roughness factor multiplier (1.5–2.5×) means the effective conductor loss is significantly higher than the ideal smooth-conductor model. Use measured $S_{21}$ data or a roughness-corrected model when building loss budgets.

3. **Connector selection matters at Gen 5.** A 2 dB difference between a Gen 4 and Gen 5 rated connector at 16 GHz can be the difference between passing and failing COM when channel margin is tight.

4. **Backdrilling vias is essential for long boards at Gen 5.** A non-backdrilled via on a 3.2 mm board adds ~3 dB of loss at 16 GHz due to stub resonance near the Nyquist frequency.

5. **The Gen 5 channel always passes Gen 4.** Insertion loss decreases with frequency; a channel designed for Gen 5 has ample margin at the Gen 4 Nyquist frequency. Backward compatibility in PCIe does not require re-validation of the physical channel.
