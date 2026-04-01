# Problem 03: Stackup Optimisation

## Problem Statement

You are the SI engineer for a new 8-layer PCB that must support two high-speed interfaces:

- **DDR5-4800:** 64 data bits + 8 ECC bits routed as single-ended microstrip on the top layer (L1) at 40 $\Omega$ single-ended (80 $\Omega$ differential)
- **PCIe Gen 5 (16 GT/s):** 16 differential pairs routed as differential stripline on an inner layer at 85 $\Omega$ differential impedance; requires total channel insertion loss $\leq -28$ dB at the Nyquist frequency of 8 GHz

**Current stackup proposal (before optimisation):**

| Layer | Function | Copper weight | Dielectric | Thickness |
|---|---|---|---|---|
| L1 | DDR5 signal (microstrip) | 1 oz (35 $\mu$m) | — | — |
| Dielectric 1-2 | Core | — | FR-4 ($D_k = 4.5$, $D_f = 0.02$) | 100 $\mu$m |
| L2 | Ground plane | 0.5 oz | — | — |
| Dielectric 2-3 | Prepreg | — | FR-4 ($D_k = 4.3$, $D_f = 0.022$) | 150 $\mu$m |
| L3 | PCIe signal (stripline) | 0.5 oz (17 $\mu$m) | — | — |
| Dielectric 3-4 | Core | — | FR-4 ($D_k = 4.5$, $D_f = 0.02$) | 150 $\mu$m |
| L4 | Ground plane | 0.5 oz | — | — |
| Dielectric 4-5 | Core | — | FR-4 ($D_k = 4.5$, $D_f = 0.02$) | 200 $\mu$m |
| L5 | Power plane (3.3 V) | 1 oz | — | — |
| Dielectric 5-6 | Prepreg | — | FR-4 ($D_k = 4.3$, $D_f = 0.022$) | 150 $\mu$m |
| L6 | PCIe signal (stripline) | 0.5 oz | — | — |
| Dielectric 6-7 | Core | — | FR-4 ($D_k = 4.5$, $D_f = 0.02$) | 150 $\mu$m |
| L7 | Ground plane | 0.5 oz | — | — |
| Dielectric 7-8 | Prepreg | — | FR-4 ($D_k = 4.3$, $D_f = 0.022$) | 100 $\mu$m |
| L8 | DDR5 signal (microstrip) | 1 oz | — | — |

**Questions:**

1. Estimate the DDR5 microstrip trace width required to achieve 40 $\Omega$ single-ended impedance on L1 using the IPC-2141 microstrip formula.
2. Estimate the PCIe differential stripline trace width and spacing for 85 $\Omega$ differential impedance on L3 (between L2 ground and L4 ground) using the edge-coupled stripline formula.
3. A field solver shows the L3 PCIe stripline has a single-ended loss of −2.1 dB/inch at 8 GHz on the current FR-4 stackup. The PCIe channel total length is 15 inches. Assess whether the channel meets the −28 dB insertion loss budget, including 1.5 dB for connector and via transitions.
4. If the loss budget is exceeded, propose stackup modifications to meet the budget. Justify each modification with quantitative reasoning.
5. Identify two additional signal integrity risks in the current stackup proposal and recommend mitigations.

---

## Worked Solution

### Step 1 — DDR5 microstrip trace width for 40 $\Omega$ on L1

The IPC-2141 microstrip impedance formula for a trace on the top surface above a reference plane:

$$Z_0 = \frac{87}{\sqrt{\varepsilon_r + 1.41}} \ln\left(\frac{5.98H}{0.8W + T}\right)$$

where:
- $H$ = dielectric height above the reference plane (L2) = 100 $\mu$m
- $W$ = trace width (to find)
- $T$ = trace thickness = 35 $\mu$m (1 oz copper)
- $\varepsilon_r$ = dielectric constant of the substrate = 4.5

Note: The microstrip effective $\varepsilon_r$ used in the formula is accounted for within the 87 coefficient and the $\sqrt{\varepsilon_r + 1.41}$ factor.

**Solve for $W$ with $Z_0 = 40\ \Omega$:**

$$40 = \frac{87}{\sqrt{4.5 + 1.41}} \ln\left(\frac{5.98 \times 100}{0.8W + 35}\right)$$

$$\sqrt{5.91} = 2.431$$

$$40 = \frac{87}{2.431} \ln\left(\frac{598}{0.8W + 35}\right) = 35.79 \ln\left(\frac{598}{0.8W + 35}\right)$$

$$\ln\left(\frac{598}{0.8W + 35}\right) = \frac{40}{35.79} = 1.1177$$

$$\frac{598}{0.8W + 35} = e^{1.1177} = 3.057$$

$$0.8W + 35 = \frac{598}{3.057} = 195.6\ \mu\text{m}$$

$$0.8W = 160.6\ \mu\text{m} \implies W = 200.7\ \mu\text{m} \approx 201\ \mu\text{m}$$

**Result: A trace width of approximately 200 $\mu$m (8 mils) is required for 40 $\Omega$ microstrip on L1.**

**Sanity check:** $W/H = 200/100 = 2$. For microstrip, when $W > H$, impedance tends toward lower values. With a $H = 100\ \mu$m core, a 200 $\mu$m trace giving 40 $\Omega$ is physically reasonable — standard 50 $\Omega$ microstrip would be narrower ($W \approx 100\ \mu$m for the same stack height).

---

### Step 2 — PCIe differential stripline trace width and spacing for 85 $\Omega$ differential on L3

L3 stripline is buried between L2 ground (100 + 150 = 250 $\mu$m below L1) and L4 ground. The total dielectric separation between L2 and L4 spans prepreg 2-3 (150 $\mu$m) + dielectric 3-4 (150 $\mu$m) = 300 $\mu$m. The L3 trace is centred, so $b = 300\ \mu$m total thickness and $H$ = distance from trace centre to each reference plane = 150 $\mu$m.

**Single-ended stripline impedance (Wadell formula, centred stripline):**

$$Z_{0,SE} = \frac{60}{\sqrt{\varepsilon_r}} \ln\left(\frac{4b}{0.67\pi(0.8W + T)}\right)$$

where $b = 300\ \mu$m (distance between reference planes), $T = 17\ \mu$m (0.5 oz copper), $\varepsilon_r = 4.5$ (average of core and prepreg values, approximated as 4.4 for a prepreg-core combination).

**Targeting $Z_{0,diff} = 85\ \Omega$:**

For edge-coupled differential stripline, the odd-mode impedance (which equals half the differential impedance) is:

$$Z_{0,diff} = 2 \times Z_{0,odd}$$

The odd-mode impedance is related to the single-ended impedance by:

$$Z_{0,odd} \approx Z_{0,SE} \left(1 - 0.347 e^{-2.9 \cdot S/b}\right)$$

where $S$ = edge-to-edge spacing between the two traces of the differential pair.

Target $Z_{0,odd} = 85/2 = 42.5\ \Omega$.

**First, find the required single-ended impedance (for zero coupling, $S \to \infty$):**

For $Z_{0,odd} = 42.5\ \Omega$ with the coupling correction applied at $S = 2W$ (a common starting spacing):

$$42.5 \approx Z_{0,SE} \left(1 - 0.347 e^{-2.9 \cdot 2W/300}\right)$$

This requires an iterative solution. Start with $S = 2W$ and $Z_{0,SE} \approx 50\ \Omega$ as an initial guess.

Estimate $W$ for $Z_{0,SE} = 50\ \Omega$ using the Wadell formula:

$$50 = \frac{60}{\sqrt{4.4}} \ln\left(\frac{4 \times 300}{0.67\pi(0.8W + 17)}\right)$$

$$\sqrt{4.4} = 2.098$$

$$50 = 28.60 \ln\left(\frac{1200}{2.104(0.8W + 17)}\right)$$

$$\ln\left(\frac{1200}{2.104(0.8W + 17)}\right) = 1.748$$

$$\frac{1200}{2.104(0.8W + 17)} = e^{1.748} = 5.743$$

$$2.104(0.8W + 17) = \frac{1200}{5.743} = 208.9$$

$$0.8W + 17 = \frac{208.9}{2.104} = 99.3\ \mu\text{m}$$

$$0.8W = 82.3 \implies W \approx 103\ \mu\text{m}$$

**Apply differential coupling correction with $S = 2W \approx 206\ \mu$m:**

$$Z_{0,odd} \approx 50 \times \left(1 - 0.347 \times e^{-2.9 \times 206/300}\right) = 50 \times \left(1 - 0.347 \times e^{-1.992}\right)$$

$$= 50 \times (1 - 0.347 \times 0.1363) = 50 \times (1 - 0.04729) = 50 \times 0.9527 = 47.6\ \Omega$$

$$Z_{0,diff} = 2 \times 47.6 = 95.2\ \Omega$$

This is too high (target is 85 $\Omega$). We need a wider trace (lower $Z_{0,SE}$) or tighter spacing.

**Iterate: Target $Z_{0,SE} \approx 44\ \Omega$ with tighter spacing $S = 1.5W$:**

Solving for $W$ at $Z_{0,SE} = 44\ \Omega$: following the same algebra gives $W \approx 135\ \mu$m.

At $S = 1.5W = 202\ \mu$m:

$$Z_{0,odd} \approx 44 \times \left(1 - 0.347 \times e^{-2.9 \times 202/300}\right) = 44 \times (1 - 0.347 \times e^{-1.953}) = 44 \times (1 - 0.347 \times 0.1418)$$

$$= 44 \times 0.9508 = 41.8\ \Omega$$

$$Z_{0,diff} = 2 \times 41.8 = 83.6\ \Omega$$

Close to 85 $\Omega$. Fine-tuning with a 2D field solver would converge at approximately:

**Result: $W \approx 130$–135 $\mu$m (5.1–5.3 mils), $S \approx 130$–200 $\mu$m (5–8 mils edge-to-edge) for 85 $\Omega$ differential stripline on L3.**

In practice, the precise values are verified with a 2D field solver (Polar SI9000, Ansys 2D Extractor) rather than closed-form formulas, which carry ±5% accuracy.

---

### Step 3 — PCIe insertion loss budget assessment

**Channel configuration:**

- Total channel length: 15 inches
- Loss budget total: −28 dB (from PCIe Gen 5 specification)
- Connector and via transition loss: 1.5 dB (given)
- Available for trace loss: $28 - 1.5 = 26.5$ dB

**Trace loss at 8 GHz (Nyquist for 16 GT/s):**

The field solver reports −2.1 dB/inch single-ended loss at 8 GHz. This is the single-ended loss; for a differential pair, the differential loss is closely approximated by the single-ended loss for tightly coupled pairs (within ±0.2 dB).

$$\text{IL}_{trace} = 2.1 \times 15 = 31.5\ \text{dB}$$

**Total channel loss:**

$$\text{IL}_{total} = 31.5 + 1.5 = 33.0\ \text{dB}$$

**Assessment:** The channel exceeds the −28 dB budget by **5 dB**. The current stackup does not meet PCIe Gen 5 requirements on FR-4. Corrective action is required.

---

### Step 4 — Stackup modifications to meet the loss budget

The loss at 8 GHz has two components:

$$\alpha_{total}(f) = \alpha_c(f) + \alpha_d(f) \approx A\sqrt{f} + B \cdot f \cdot \sqrt{D_k} \cdot D_f$$

At 8 GHz on FR-4 ($D_f = 0.020$):

- Typical split on FR-4 1 oz microstrip: $\alpha_c \approx 40\%$ of total loss, $\alpha_d \approx 60\%$
- On the 0.5 oz stripline: conductor loss is higher (thinner copper), so $\alpha_c \approx 50\%$

We need to reduce total loss from 2.1 dB/inch to at most $26.5/15 = 1.77$ dB/inch — a reduction of **0.33 dB/inch (16%)**.

**Modification A — Change to low-loss dielectric material:**

Replace FR-4 ($D_f = 0.020$) with a low-loss laminate on the PCIe signal layers. Candidate materials:

| Material | $D_k$ (10 GHz) | $D_f$ (10 GHz) | Relative cost |
|---|---|---|---|
| FR-4 standard | 4.5 | 0.020 | 1× |
| Isola IS410 | 4.0 | 0.013 | 1.5× |
| Isola I-Tera MT40 | 3.45 | 0.0031 | 3× |
| Panasonic Megtron 6 | 3.4 | 0.002 | 4× |
| Panasonic Megtron 7 | 3.3 | 0.0015 | 5.5× |

For a 16% reduction in dielectric loss (assuming $\alpha_d$ accounts for 50% of total loss):

Required $D_f$ reduction: $0.020 \times (1 - 0.16/0.50) = 0.020 \times 0.68 = 0.0136$.

Isola IS410 at $D_f = 0.013$ meets this requirement with a modest 1.5× cost premium. Estimate revised loss:

$$\frac{\alpha_{d,new}}{\alpha_{d,old}} = \frac{D_f^{new} \cdot \sqrt{D_k^{new}}}{D_f^{old} \cdot \sqrt{D_k^{old}}} = \frac{0.013 \times \sqrt{4.0}}{0.020 \times \sqrt{4.5}} = \frac{0.013 \times 2.0}{0.020 \times 2.121} = \frac{0.026}{0.04242} = 0.613$$

With 50% dielectric loss fraction: revised total loss $= 2.1 \times (0.5 + 0.5 \times 0.613) = 2.1 \times 0.807 = 1.69$ dB/inch.

Total channel loss: $1.69 \times 15 + 1.5 = 25.35 + 1.5 = 26.85$ dB. **Margin: 1.15 dB.** This just passes.

**Modification B — Use heavier copper on L3 (1 oz instead of 0.5 oz):**

Heavier copper reduces DC resistance and mitigates skin-effect loss at lower frequencies. At 8 GHz, skin depth on copper is approximately $\delta = 740$ nm — the current crowds into a thin surface layer regardless of copper weight. The benefit is primarily at lower frequencies (< 1 GHz). Changing from 0.5 oz to 1 oz copper at 8 GHz reduces conductor loss by approximately 10–15%.

If $\alpha_c$ is 50% of total loss, a 12% reduction in conductor loss reduces total loss by 6%:

$$\alpha_{total,new} \approx 2.1 \times (0.5 + 0.5 \times 0.88) = 2.1 \times 0.94 = 1.97\ \text{dB/inch}$$

This alone is insufficient. Combined with Modification A:

$$\alpha_{total} \approx 2.1 \times (0.5 \times 0.88 + 0.5 \times 0.613) = 2.1 \times 0.747 = 1.57\ \text{dB/inch}$$

Total: $1.57 \times 15 + 1.5 = 23.55 + 1.5 = 25.05$ dB. **Margin: 2.95 dB.** This provides comfortable margin.

**Modification C — Optimise trace width (wider trace, lower conductor loss):**

Wider traces reduce current density and ohmic loss. However, on L3 stripline, widening the trace reduces $Z_{0,SE}$, requiring the differential spacing to be increased to maintain 85 $\Omega$ differential. This increases the pair pitch, potentially affecting the routing channel density.

A 20% increase in trace width (from 130 $\mu$m to 156 $\mu$m) reduces conductor loss by approximately $\sqrt{1.2} - 1 \approx 10\%$ in the skin-effect regime. This provides a modest improvement. Not recommended as a primary fix due to routing density impact.

**Recommended stackup change:**

Replace the core and prepreg materials for the L2-L4 stack (surrounding the PCIe signals) with Isola IS410 ($D_k = 4.0$, $D_f = 0.013$). Increase L3 copper weight to 1 oz. Keep FR-4 on non-critical layers (L5–L8 and DDR5 layers) to control cost.

**Revised estimate after modifications A + B:**

$$\text{IL}_{total} \approx 25.05\ \text{dB} \leq 28\ \text{dB}\quad \checkmark$$

**Margin: 2.95 dB** — adequate for PCIe Gen 5 compliance including manufacturing tolerance.

---

### Step 5 — Additional SI risks in the current stackup

**Risk 1: L3 signal layer referenced to power plane (L5) is too distant.**

The L3–L4 ground reference is proper, but L5 is a power plane. If any PCIe trace must route over the L4–L5 gap (e.g., at a via transition from L3 to L6), the trace transitions from a ground reference to a power plane reference. The increased separation to a power return causes:

- Impedance increase at the transition point
- Return current flows through decoupling capacitors (inductance path) rather than the ground plane directly, adding impedance to the return path
- Increased radiation from the current loop

**Mitigation:** Add a copper ground layer between L4 (ground) and L5 (power), e.g., add a split ground/power layer, or ensure PCIe traces never route in the L4–L5 dielectric. Alternatively, assign L5 as a solid ground plane and move the power distribution to an embedded coin or power island within a signal layer.

**Risk 2: DDR5 signals on L1 microstrip with only 100 $\mu$m core to L2 ground.**

At $H = 100\ \mu$m with $W \approx 200\ \mu$m, the trace is wide relative to the ground separation ($W/H = 2$). The closely spaced ground plane below L1 creates strong ground coupling — which is good for impedance control — but:

- **Crosstalk:** With wide traces at $W = 200\ \mu$m and nominal DDR5 spacing of 1–2× line width (~200–400 $\mu$m pitch), the edge-to-edge trace separation is 0–200 $\mu$m. This produces significant capacitive coupling (NEXT and FEXT). DDR5 fly-by topology is particularly sensitive to FEXT on the address/command bus.
- **Microstrip field exposure:** L1 microstrip has the electric field partially in air above the trace. Any conformal coating, component placement above the trace, or other dielectric material changes the effective $D_k$ and shifts the impedance from the target 40 $\Omega$.

**Mitigation:** Increase the L1-to-L2 dielectric to 130–150 $\mu$m to allow the trace to be narrowed to $W \approx 130\ \mu$m. Narrower traces reduce capacitive coupling between adjacent signals. Enforce a minimum spacing rule of 3× trace width (edge-to-edge) for DDR5 address/command signals. If FEXT remains a concern, consider routing half the DDR5 signals on L8 with L7 as the reference.

---

### Summary of Recommendations

| Issue | Root cause | Recommended change |
|---|---|---|
| PCIe insertion loss exceeds budget | FR-4 high $D_f$ + thin copper | Upgrade L2-L4 to IS410; increase L3 Cu to 1 oz |
| Return current path discontinuity at L4-L5 | Power plane L5 adjacent to signal region | Add solid ground layer between L4 and L5 |
| DDR5 crosstalk risk | Wide traces at narrow pitch on L1 | Increase H to 130–150 $\mu$m; narrow traces; enforce 3W spacing rule |

---

### Common Interview Pitfalls

**Forgetting the factor of 2 in the dielectric loss formula:** Dielectric loss scales as $f \cdot \sqrt{D_k} \cdot D_f$. Changing material from $D_f = 0.02$ to $D_f = 0.013$ does not halve the total loss — it reduces only the dielectric loss component. If conductor and dielectric loss are roughly equal, the total loss reduction is approximately 17.5%, not 35%.

**Using single-ended loss for differential channels without verification:** Differential insertion loss $\approx$ single-ended insertion loss when the pair is tightly coupled and the odd-mode is well-defined. When pairs are loosely coupled (wide spacing) or routed with ground stitching between pairs, the modes decouple and the differential insertion loss must be measured directly on the differential pair, not inferred from single-ended measurement.

**Selecting the most expensive low-loss material without cost justification:** Megtron 7 at $D_f = 0.0015$ would pass easily, but it costs 5.5× FR-4. Isola IS410 at $D_f = 0.013$ passes with 2.95 dB margin at 1.5× cost. In a real design, specify the cheapest material that meets the requirement with a reasonable margin — not the best-performing material available.

**Ignoring the manufacturing tolerance stack-up:** The $\pm 10\%$ tolerance on dielectric thickness and $\pm 10\%$ on trace width means the achieved impedance can vary by ±5 $\Omega$ from nominal. A design that passes with zero margin in simulation may fail when fabricated. Always design for a minimum 1–2 dB insertion loss margin to accommodate manufacturing variability.
