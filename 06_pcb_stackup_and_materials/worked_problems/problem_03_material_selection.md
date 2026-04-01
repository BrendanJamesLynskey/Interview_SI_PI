# Problem 03: Material Selection

## Problem Statement

You are the SI lead for a new line card design. Three applications must be supported on the same physical PCB (a common constraint in telecom/datacenter switch designs):

**Application A — PCIe Gen5 host interface:**
- 32 GT/s NRZ
- Channel length: 180 mm (host ASIC to edge connector)
- Total channel insertion loss budget at Nyquist: −12 dB

**Application B — 100GBASE-KR4 backplane SerDes:**
- 25 Gb/s NRZ per lane, 4 lanes
- Channel length: 250 mm (chip to backplane connector)
- Total channel insertion loss budget at Nyquist: −15 dB (IEEE 802.3 clause 93)

**Application C — DDR5-5600 local memory:**
- 2800 MT/s (1400 MHz clock, 2800 MT/s effective)
- Channel length: 70 mm
- Total channel insertion loss budget: −10 dB at 2.8 GHz

**Board constraints:**
- 10-layer board, 1.4 mm total thickness
- Connector and via losses allocated: 2.0 dB per connector pair (Applications A and B); 0.3 dB for DDR5 vias (Application C)
- Copper foil specification will be chosen as part of this problem
- Maximum trace width on inner signal layers: 130 µm (routing density constraint)

**Questions:**

1. For each application, calculate the maximum allowable trace loss per 100 mm.
2. For each application, determine whether FR4, IS680, or Megtron 6 is adequate. Show the calculation.
3. Determine the minimum copper foil grade required (STD/HTE, VLP, HVLP) for Application A (PCIe Gen5).
4. Specify a single laminate for the entire board that satisfies all three applications and justify the choice.
5. Identify any remaining risk in the material selection.

---

## Worked Solution

### Step 1 — Maximum Allowable Trace Loss per 100 mm

**PCIe Gen5 (Application A):**

Total budget: −12 dB. Connector/via losses allocated:
- Transmitter ASIC via: ~0.5 dB
- Receiver ASIC via: ~0.5 dB
- Connector pair: 2.0 dB

Total non-trace loss: 3.0 dB. Remaining trace budget:

$$\alpha_{trace,max}^A = 12 - 3.0 = 9.0 \text{ dB over 180 mm}$$

$$\alpha_{per 100mm}^A = \frac{9.0}{180} \times 100 = 5.0 \text{ dB/100mm at 16 GHz}$$

**100GBASE-KR4 SerDes (Application B):**

Total budget: −15 dB. Connector/via losses:
- Transmitter via: ~0.5 dB
- Receiver via: ~0.5 dB
- Backplane connector pair: 2.0 dB

Total non-trace loss: 3.0 dB. Remaining trace budget:

$$\alpha_{trace,max}^B = 15 - 3.0 = 12.0 \text{ dB over 250 mm}$$

$$\alpha_{per 100mm}^B = \frac{12.0}{250} \times 100 = 4.8 \text{ dB/100mm at 12.5 GHz}$$

**DDR5-5600 (Application C):**

Total budget: −10 dB. Via losses:
- CPU BGA via: ~0.15 dB
- DDR5 DRAM via: ~0.15 dB
- Total via: 0.3 dB

Remaining trace budget:

$$\alpha_{trace,max}^C = 10 - 0.3 = 9.7 \text{ dB over 70 mm}$$

$$\alpha_{per 100mm}^C = \frac{9.7}{70} \times 100 = 13.9 \text{ dB/100mm at 2.8 GHz}$$

The DDR5-5600 budget is the most generous per unit length (lowest frequency, shortest traces). Applications A and B are the challenging ones.

---

### Step 2 — Material Adequacy Check

**Trace geometry assumption:**

For a 130 µm trace width constraint (inner stripline, 180 µm total dielectric between reference planes, ½ oz copper), the approximate 50 Ω stripline impedance is achievable at approximately 90–100 µm width, leaving the 130 µm constraint as a non-binding maximum here. We use a 90 µm stripline trace for the impedance-controlled calculations.

**Conductor loss model ($\sqrt{f}$ scaling):**

For a 90 µm wide, 18 µm thick stripline, conductor loss at 1 GHz baseline:

$$\alpha_c(1 \text{ GHz}) \approx 0.35 \text{ dB/100mm}$$

(This is a reference value for typical thin copper stripline; exact value requires field solver.)

Conductor loss at frequency $f$:

$$\alpha_c(f) = 0.35 \times \sqrt{f[\text{GHz}]} \text{ dB/100mm}$$

Surface roughness correction for different copper grades (at 10 GHz):
- STD/HTE copper (RMS = 3 µm): $K_{rough} \approx 2.0$ → effective $\alpha_c$ doubles
- VLP copper (RMS = 1 µm): $K_{rough} \approx 1.4$
- HVLP copper (RMS = 0.5 µm): $K_{rough} \approx 1.15$

We first evaluate with VLP copper.

**Dielectric loss model ($f \cdot D_f$ scaling, $\sqrt{\epsilon_r}$ factor):**

$$\alpha_d(f) = 27.3 \frac{\sqrt{\epsilon_r} \cdot f[\text{GHz}]}{300} \cdot D_f \text{ dB/100mm}$$

**--- FR4 evaluation ($D_k = 4.2$, $D_f = 0.020$ at relevant frequencies) ---**

For Applications A and B, $D_f$ at 16 GHz and 12.5 GHz is approximately 0.022–0.025. Use $D_f = 0.022$.

**FR4 at 16 GHz (PCIe Gen5, Application A):**

$$\alpha_d^{FR4}(16 \text{ GHz}) = 27.3 \times \frac{\sqrt{4.2} \times 16}{300} \times 0.022 = 27.3 \times \frac{2.049 \times 16}{300} \times 0.022$$

$$= 27.3 \times 0.1093 \times 0.022 = 0.0656 \text{ dB/mm} = 6.56 \text{ dB/100mm}$$

$$\alpha_c^{FR4}(16 \text{ GHz, VLP}) = 0.35 \times \sqrt{16} \times 1.4 = 0.35 \times 4 \times 1.4 = 1.96 \text{ dB/100mm}$$

$$\alpha_{total}^{FR4}(16 \text{ GHz}) = 6.56 + 1.96 = 8.52 \text{ dB/100mm}$$

**FR4 limit vs. budget of 5.0 dB/100mm: FAIL** (8.52 >> 5.0). FR4 is inadequate for PCIe Gen5.

**FR4 at 12.5 GHz (100GbE, Application B):**

$$\alpha_d^{FR4}(12.5) = 27.3 \times \frac{2.049 \times 12.5}{300} \times 0.022 = 27.3 \times 0.08538 \times 0.022 = 5.12 \text{ dB/100mm}$$

$$\alpha_c^{FR4}(12.5) = 0.35 \times \sqrt{12.5} \times 1.4 = 0.35 \times 3.536 \times 1.4 = 1.73 \text{ dB/100mm}$$

$$\alpha_{total}^{FR4}(12.5) = 6.85 \text{ dB/100mm}$$

**FR4 vs. budget of 4.8 dB/100mm: FAIL** (6.85 >> 4.8).

**FR4 at 2.8 GHz (DDR5, Application C):**

$$\alpha_d^{FR4}(2.8) = 27.3 \times \frac{2.049 \times 2.8}{300} \times 0.018 = 27.3 \times 0.01912 \times 0.018 = 0.940 \text{ dB/100mm}$$

$$\alpha_c^{FR4}(2.8) = 0.35 \times \sqrt{2.8} \times 1.4 = 0.35 \times 1.673 \times 1.4 = 0.820 \text{ dB/100mm}$$

$$\alpha_{total}^{FR4}(2.8) = 1.76 \text{ dB/100mm}$$

**FR4 vs. budget of 13.9 dB/100mm: PASS** (1.76 << 13.9). FR4 easily meets DDR5-5600 requirements.

**--- IS680 evaluation ($D_k = 3.77$, $D_f = 0.007$ at 12.5 GHz, $0.008$ at 16 GHz) ---**

**IS680 at 16 GHz (Application A):**

$$\alpha_d^{IS680}(16) = 27.3 \times \frac{\sqrt{3.77} \times 16}{300} \times 0.008 = 27.3 \times \frac{1.942 \times 16}{300} \times 0.008$$

$$= 27.3 \times 0.10357 \times 0.008 = 0.02262 \text{ dB/mm} = 2.26 \text{ dB/100mm}$$

$$\alpha_c^{IS680}(16, VLP) = 0.35 \times 4 \times 1.4 = 1.96 \text{ dB/100mm}$$

$$\alpha_{total}^{IS680}(16) = 2.26 + 1.96 = 4.22 \text{ dB/100mm}$$

**IS680 vs. budget of 5.0 dB/100mm: PASS** (4.22 < 5.0) — with 0.78 dB/100mm margin.

**IS680 at 12.5 GHz (Application B):**

$$\alpha_d^{IS680}(12.5) = 27.3 \times \frac{1.942 \times 12.5}{300} \times 0.007 = 27.3 \times 0.08092 \times 0.007 = 1.547 \text{ dB/100mm}$$

$$\alpha_c^{IS680}(12.5, VLP) = 1.73 \text{ dB/100mm}$$

$$\alpha_{total}^{IS680}(12.5) = 3.28 \text{ dB/100mm}$$

**IS680 vs. budget of 4.8 dB/100mm: PASS** (3.28 < 4.8) — with 1.52 dB/100mm margin.

**--- Megtron 6 evaluation ($D_k = 3.64$, $D_f = 0.004$ at 16 GHz) ---**

**Megtron 6 at 16 GHz (Application A):**

$$\alpha_d^{M6}(16) = 27.3 \times \frac{\sqrt{3.64} \times 16}{300} \times 0.004 = 27.3 \times \frac{1.908 \times 16}{300} \times 0.004$$

$$= 27.3 \times 0.10176 \times 0.004 = 0.01112 \text{ dB/mm} = 1.11 \text{ dB/100mm}$$

$$\alpha_c^{M6}(16, VLP) = 1.96 \text{ dB/100mm}$$

$$\alpha_{total}^{M6}(16) = 3.07 \text{ dB/100mm}$$

**Megtron 6 vs. budget of 5.0 dB/100mm: PASS** — with 1.93 dB/100mm margin.

---

### Step 3 — Copper Foil Grade for PCIe Gen5 (Application A)

**Conductor loss at 16 GHz for different copper grades:**

| Copper grade | $K_{rough}$ at 16 GHz | $\alpha_c$ at 16 GHz |
|---|---|---|
| STD/HTE | 2.2 | $0.35 \times 4 \times 2.2 = 3.08$ dB/100mm |
| VLP | 1.4 | $0.35 \times 4 \times 1.4 = 1.96$ dB/100mm |
| HVLP | 1.15 | $0.35 \times 4 \times 1.15 = 1.61$ dB/100mm |

**Total loss with different copper grades on IS680 (Application A at 16 GHz, budget = 5.0 dB/100mm):**

| Copper grade | $\alpha_d$ | $\alpha_c$ | Total | vs. Budget |
|---|---|---|---|---|
| STD/HTE | 2.26 | 3.08 | **5.34** | FAIL |
| VLP | 2.26 | 1.96 | **4.22** | PASS (+0.78 margin) |
| HVLP | 2.26 | 1.61 | **3.87** | PASS (+1.13 margin) |

**Minimum copper grade for Application A (PCIe Gen5) on IS680: VLP.**

STD/HTE copper fails PCIe Gen5 even on IS680 because conductor loss at 16 GHz dominates. VLP passes with 0.78 dB/100mm margin. HVLP provides additional margin and is recommended if the cost delta is acceptable.

**With Megtron 6 (for comparison):**

| Copper grade | $\alpha_d$ | $\alpha_c$ | Total | vs. Budget |
|---|---|---|---|---|
| STD/HTE | 1.11 | 3.08 | **4.19** | PASS (barely) |
| VLP | 1.11 | 1.96 | **3.07** | PASS (+1.93 margin) |
| HVLP | 1.11 | 1.61 | **2.72** | PASS (+2.28 margin) |

Megtron 6 + HVLP is the most comfortable combination but also the most expensive. Megtron 6 + VLP is the typical industry choice for PCIe Gen5.

---

### Step 4 — Single Laminate Recommendation for the Entire Board

**Summary of material adequacy:**

| Application | FR4 | IS680 | Megtron 6 |
|---|---|---|---|
| A: PCIe Gen5 @ 16 GHz | Fail | Pass (VLP req.) | Pass (VLP sufficient) |
| B: 100GbE @ 12.5 GHz | Fail | Pass | Pass |
| C: DDR5 @ 2.8 GHz | Pass | Pass | Pass |

**Recommendation: IS680 with VLP copper foil.**

**Justification:**

1. **All three applications pass** with IS680 + VLP. PCIe Gen5 at 5.0 dB/100mm budget is met at 4.22 dB/100mm (0.78 dB margin). 100GbE at 4.8 dB/100mm is met at 3.28 dB/100mm (1.52 dB margin). DDR5-5600 is trivially met.

2. **IS680 is less expensive than Megtron 6** — approximately 2–3× FR4 cost vs. 3–5× for Megtron 6. Over a large-volume production run, the cost difference is significant.

3. **IS680 processability** is good. It behaves similarly to standard FR4 in the fabrication shop (lamination parameters, drill speed, plating chemistry). Megtron 6 requires slightly modified drill parameters and lamination press profiles — not a showstopper, but IS680 is the simpler choice.

4. **IS680 has sufficient $T_g$** (>180°C) for typical datacenter operating environments. If the board requires soldering with Pb-free solder (SAC305, peak temperature ~260°C), IS680's $T_g$ is adequate for the brief thermal excursion.

5. **Megtron 6 is the conservative choice** if:
   - The channel models include worst-case dielectric and copper tolerance (±15% on $D_f$ is common in production batches)
   - The design is used across multiple board fabricator sources (IS680 has tighter single-source availability than Megtron 6)
   - The PCIe Gen5 channel length may increase in a future board revision

If the design team wants additional margin (e.g., to allow a longer PCIe Gen5 channel in the next revision) or if the fabricator's IS680 batch $D_f$ has been measured higher than nominal, **Megtron 6 + VLP copper** is the appropriate upgrade path.

---

### Step 5 — Remaining Risks

**Risk 1: IS680 $D_f$ batch variation**

IS680 datasheet specifies $D_f = 0.004$ at 2 GHz. At 16 GHz, the value can be 0.007–0.010 depending on frequency extrapolation and lot variation. If the worst-case $D_f$ at 16 GHz is 0.010:

$$\alpha_d^{IS680,worst}(16) = 27.3 \times \frac{1.942 \times 16}{300} \times 0.010 = 2.82 \text{ dB/100mm}$$

$$\alpha_{total}^{worst}(16) = 2.82 + 1.96 = 4.78 \text{ dB/100mm}$$

This is within the 5.0 dB budget (0.22 dB margin), but the margin is thin. **Action: Request lot-specific $D_f$ measurements from the laminate supplier at 16 GHz and verify that the worst-case value remains below 0.009.**

**Risk 2: Glass weave effect on differential pairs**

The 100GbE KR4 lanes and PCIe Gen5 lanes are all differential. IS680 uses standard woven glass (not spread-weave). If the differential pairs are routed parallel to the glass weave direction, intra-pair skew from glass weave inhomogeneity can cause differential-to-common mode conversion, degrading $S_{cc21}$ and worsening radiated EMI.

**Action:** Specify differential trace routing at 45° to the PCB x-axis (assuming x-axis is aligned to the warp or fill direction of the glass weave). Include this routing constraint in the design rules. Alternatively, specify IS680 with spread-weave option if available.

**Risk 3: Conductor loss model accuracy**

The $\sqrt{f}$ conductor loss model with a fixed roughness correction factor $K$ is an approximation. The Huray cannonball model provides better accuracy but requires copper surface roughness measurements from the foil supplier. VLP copper from different suppliers can have $K$ factors ranging from 1.2 to 1.6 at 16 GHz. Using $K = 1.4$ as the model value:

- Best case ($K = 1.2$): $\alpha_c = 1.68$ dB/100mm → total = 3.94 dB/100mm
- Nominal ($K = 1.4$): $\alpha_c = 1.96$ dB/100mm → total = 4.22 dB/100mm
- Worst case ($K = 1.6$): $\alpha_c = 2.24$ dB/100mm → total = 4.50 dB/100mm

All within the 5.0 dB budget. **Action:** Confirm the VLP copper roughness specification with the foil supplier and update the Huray model parameters before finalising the trace width.

**Risk 4: Frequency-dependent material model in simulation**

If the channel simulation uses constant-value $D_k = 3.77$ and $D_f = 0.007$ (the IS680 datasheet value at 2 GHz) rather than a Djordjevic-Sarkar frequency-dependent model, the simulation will underestimate insertion loss at 16 GHz. The actual $D_f$ at 16 GHz may be 0.010 vs. the 0.007 used in simulation.

**Action:** Extract a broadband Djordjevic-Sarkar material model from IS680 measured data (or request the laminate supplier's frequency-dependent data file) and use it in all channel simulations. Confirm the simulation predicts channel compliance before committing to a board layout.

---

### Final Material Specification

**Selected laminate:** Isola IS680
**Copper foil grade:** VLP (RMS roughness $\leq 1.0$ µm, confirmed by supplier datasheet)
**Copper weight:** ½ oz (18 µm) on all signal layers
**Glass weave:** Standard 2116 style (specify routing at 45° to weave for critical differential pairs)
**$T_g$:** > 180°C confirmed for all laminate lots
**Simulation model:** Djordjevic-Sarkar broadband model, populated with measured $D_k(f)$ / $D_f(f)$ data from 100 MHz to 20 GHz

**If Application A channel length increases to > 200 mm in future revision:** Upgrade to Megtron 6 + HVLP copper.

---

### Common Interview Pitfalls

**Using a single $D_f$ value across all frequencies:** A candidate who uses $D_f = 0.004$ (the IS680 1 GHz value) to compute loss at 16 GHz will underestimate dielectric loss by approximately 2.5×. Always use the material data at or near the operating frequency.

**Ignoring copper roughness:** Selecting a material entirely on $D_f$ without accounting for conductor roughness leads to optimistic loss estimates. At 16 GHz, the roughness penalty on STD copper can add more loss than the entire dielectric loss on a low-$D_f$ laminate. Both loss mechanisms must be evaluated together.

**Recommending Megtron 6 without justification:** Defaulting to the most expensive material without demonstrating that IS680 fails the budget is an incomplete engineering answer. In a production design, material cost is a real constraint. The engineer must show the calculation, demonstrate whether each material passes or fails, and recommend the lowest-cost material that satisfies all requirements.

**Forgetting the connector and via loss allocation:** A candidate who applies the full channel budget to the trace, ignoring connector and via losses, will compute a falsely relaxed loss-per-unit-length limit and may select a weaker material than required.
