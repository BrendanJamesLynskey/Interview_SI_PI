# Problem 01: Insertion Loss Budget for a High-Speed Serial Link

## Problem Statement

You are a signal integrity engineer reviewing a PCIe Gen 4 (16 GT/s) host-to-add-in-card design. The channel consists of the following segments:

- **CPU package:** Ball grid array launch, on-package trace, solder bump. Treated as a lumped 2-port with measured $|S_{21}|_{pkg,TX} = -1.5$ dB at 8 GHz.
- **PCB trace (motherboard):** 18 inches of stripline routing, 50 $\Omega$ differential ($Z_{diff} = 100\ \Omega$), using Megtron 6 laminate ($D_f = 0.004$, $D_k = 3.7$).
- **Edge connector (PCIe x16 slot):** Molex 67491 connector, datasheet specifies $|S_{21}| = -1.0$ dB at 8 GHz per mated pair.
- **Add-in card PCB trace:** 4 inches of stripline on FR4 ($D_f = 0.020$, $D_k = 4.0$).
- **Device package:** BGA launch and on-package trace. Treated as a lumped 2-port with $|S_{21}|_{pkg,RX} = -1.0$ dB at 8 GHz.

**Additional parameters:**

- Two vias with backdrilling on the motherboard: each contributes $-0.5$ dB at 8 GHz.
- Two vias without backdrilling on the add-in card: each contributes $-1.2$ dB at 8 GHz (stub resonance at 9 GHz, but significant loss at 8 GHz).
- PCIe Gen 4 maximum channel insertion loss limit: **-36 dB** at 8 GHz (Nyquist frequency for 16 GT/s NRZ).

**Questions:**

1. Calculate the total channel insertion loss at 8 GHz. Does the channel pass the PCIe Gen 4 limit?
2. If the channel fails, identify the dominant loss contributors and propose specific modifications to bring the channel within specification.
3. The connector vendor claims their component can be replaced with a lower-loss version at $-0.3$ dB. How does this change the margin?
4. Calculate the skin-effect and dielectric loss separately for the 18-inch motherboard trace, and identify which loss mechanism dominates at 8 GHz.

---

## Worked Solution

### Step 1: Tabulate and Sum All Loss Contributors

Build the insertion loss budget by listing every component in series. Insertion loss values are positive numbers in dB (signal magnitude decreases).

| Component | IL at 8 GHz (dB) | Notes |
|---|---|---|
| TX package | 1.5 | Measured S21 |
| Motherboard trace (18 in, Megtron 6) | ? | Calculate below |
| Via 1 (backdrilled, motherboard) | 0.5 | Datasheet/simulation |
| Via 2 (backdrilled, motherboard) | 0.5 | Datasheet/simulation |
| PCIe edge connector | 1.0 | Datasheet at 8 GHz |
| Add-in card trace (4 in, FR4) | ? | Calculate below |
| Via 3 (no backdrill, add-in card) | 1.2 | Simulation result |
| Via 4 (no backdrill, add-in card) | 1.2 | Simulation result |
| RX package | 1.0 | Measured S21 |

---

### Step 2: Calculate PCB Trace Insertion Loss

The total insertion loss of a PCB stripline trace has two main components:

**Skin-effect loss:**

$$\alpha_{skin}(f) = \frac{R_s}{2} \sqrt{\frac{f}{f_0}} \cdot \frac{1}{Z_0}\ \text{Np/m}$$

A convenient approximation in practical units (dB per inch at frequency $f$ in GHz) for a 50 $\Omega$ stripline:

$$\alpha_{skin} \approx \frac{A_{skin}}{\sqrt{f}}\ \text{dB/inch at } f\ \text{GHz}$$

where $A_{skin}$ depends on trace geometry and copper finish. For a 5 mil wide, 50 $\Omega$ stripline on standard 1 oz copper (RMS roughness $\approx 0.6\ \mu$m):

$$A_{skin} \approx 0.035\ \text{dB/inch/}\sqrt{\text{GHz}}$$

**Dielectric loss:**

$$\alpha_{diel}(f) = \pi f \sqrt{\varepsilon_r} D_f / c = A_{diel} \cdot f\ \text{dB/inch at } f\ \text{GHz}$$

$$A_{diel} = \frac{\pi \sqrt{\varepsilon_r} D_f}{c \cdot \ln(10)/20}$$

In practical SI units:

$$A_{diel} \approx 4.34 \cdot \pi \cdot \sqrt{\varepsilon_r} \cdot D_f / c$$

where $c = 11.8$ inch/ns. More directly:

$$A_{diel} = 2.3 \sqrt{\varepsilon_r} D_f\ \text{dB/(inch·GHz)}$$

**Motherboard trace (Megtron 6, $D_f = 0.004$, $D_k = 3.7$):**

Skin-effect loss at 8 GHz:

$$\alpha_{skin}(8) = 0.035 \times \sqrt{8} = 0.035 \times 2.83 = 0.099\ \text{dB/inch}$$

Dielectric loss at 8 GHz:

$$\alpha_{diel}(8) = 2.3 \times \sqrt{3.7} \times 0.004 \times 8 = 2.3 \times 1.924 \times 0.004 \times 8 = 0.142\ \text{dB/inch}$$

Total loss per inch at 8 GHz (Megtron 6):

$$\alpha_{total} = 0.099 + 0.142 = 0.241\ \text{dB/inch}$$

Total for 18 inches:

$$IL_{MB} = 18 \times 0.241 = 4.34\ \text{dB}$$

**Add-in card trace (FR4, $D_f = 0.020$, $D_k = 4.0$):**

Skin-effect loss at 8 GHz (same copper geometry assumed):

$$\alpha_{skin}(8) = 0.035 \times \sqrt{8} = 0.099\ \text{dB/inch}$$

Dielectric loss at 8 GHz:

$$\alpha_{diel}(8) = 2.3 \times \sqrt{4.0} \times 0.020 \times 8 = 2.3 \times 2.0 \times 0.020 \times 8 = 0.736\ \text{dB/inch}$$

Total loss per inch at 8 GHz (FR4):

$$\alpha_{total} = 0.099 + 0.736 = 0.835\ \text{dB/inch}$$

Total for 4 inches:

$$IL_{AIC} = 4 \times 0.835 = 3.34\ \text{dB}$$

---

### Step 3: Complete the Budget and Check Compliance

| Component | IL at 8 GHz (dB) |
|---|---|
| TX package | 1.50 |
| Motherboard trace (18 in, Megtron 6) | 4.34 |
| Via 1 (backdrilled) | 0.50 |
| Via 2 (backdrilled) | 0.50 |
| PCIe edge connector | 1.00 |
| Add-in card trace (4 in, FR4) | 3.34 |
| Via 3 (no backdrill) | 1.20 |
| Via 4 (no backdrill) | 1.20 |
| RX package | 1.00 |
| **Total** | **14.58 dB** |

**Wait — this is far below the 36 dB limit. Is that correct?**

Yes. The total channel IL of 14.58 dB is well within the PCIe Gen 4 -36 dB limit. The channel passes with margin $= 36 - 14.58 = 21.4\ \text{dB}$.

**Important note for the interview:** The question asked to check whether the channel fails. It does not — it has substantial margin. The original problem framing assumed a channel that might be marginal, but this well-designed channel with Megtron 6 on the motherboard and a short add-in card trace passes comfortably. The PCIe Gen 4 -36 dB limit is relevant for long server backplane channels (24+ inches with FR4 or challenging via structures).

Let us instead consider a **failure scenario** by changing the conditions to reflect a longer, less-optimised design:

**Modified scenario (marginal channel):**
- Motherboard trace: 30 inches on FR4 ($D_f = 0.020$, $D_k = 4.0$)
- Add-in card trace: 8 inches on FR4
- Vias: no backdrilling on any via (4 vias, each -1.5 dB)
- All other components unchanged

**Recalculate:**

Motherboard trace (30 in, FR4):

$$IL_{MB} = 30 \times (0.099 + 0.736) = 30 \times 0.835 = 25.05\ \text{dB}$$

Add-in card trace (8 in, FR4):

$$IL_{AIC} = 8 \times 0.835 = 6.68\ \text{dB}$$

| Component | IL (dB) |
|---|---|
| TX package | 1.50 |
| Motherboard trace (30 in, FR4) | 25.05 |
| Via 1 (no backdrill) | 1.50 |
| Via 2 (no backdrill) | 1.50 |
| PCIe edge connector | 1.00 |
| Add-in card trace (8 in, FR4) | 6.68 |
| Via 3 (no backdrill) | 1.50 |
| Via 4 (no backdrill) | 1.50 |
| RX package | 1.00 |
| **Total** | **41.23 dB** |

**This channel FAILS.** Total IL = 41.23 dB exceeds the 36 dB limit by **5.23 dB**.

---

### Step 4: Identify Dominant Failures and Propose Fixes

**Dominant loss contributors:**

| Component | IL (dB) | % of Total | Reducible? |
|---|---|---|---|
| Motherboard trace | 25.05 | 60.8% | Yes |
| Add-in card trace | 6.68 | 16.2% | Limited |
| Vias (×4) | 6.00 | 14.6% | Yes |
| Packages (TX+RX) | 2.50 | 6.1% | No |
| Connector | 1.00 | 2.4% | Marginal |

**Fix 1 — Upgrade motherboard laminate (highest impact):**

Replace FR4 ($D_f = 0.020$) with Megtron 6 ($D_f = 0.004$):

$$IL_{MB,Megtron6} = 30 \times (0.099 + 0.004 \times 2.3 \times 2 \times 8) = 30 \times (0.099 + 0.147) = 30 \times 0.246 = 7.38\ \text{dB}$$

Saving vs FR4: $25.05 - 7.38 = 17.67$ dB. This alone brings the channel below 36 dB.

New total: $41.23 - 17.67 = 23.56$ dB. Passes with 12.4 dB margin.

**Fix 2 — Backdrill all vias:**

Replace 4× unbackdrilled vias (1.5 dB each) with backdrilled vias (0.5 dB each):

Saving: $4 \times (1.5 - 0.5) = 4.0$ dB.

Combined with Fix 1: total = $23.56 - 4.0 = 19.56$ dB. Excellent margin.

**Fix 3 — Reduce trace length (if routing allows):**

Every inch of FR4 trace eliminated saves 0.835 dB. Reducing motherboard trace from 30 to 25 inches saves $5 \times 0.835 = 4.2$ dB.

**Priority order for this design:**

1. Upgrade motherboard laminate to Megtron 6 or Panasonic Megtron 7 — recovers 17+ dB, passes the channel.
2. Backdrill all vias — prevents resonant notches and saves 4 dB.
3. Optimise routing — minimise unnecessary bends and stubs.

---

### Step 5: Effect of Improved Connector (Question 3)

With the lower-loss connector ($-0.3$ dB instead of $-1.0$ dB):

Saving: $1.0 - 0.3 = 0.7$ dB.

For the original passing channel (14.58 dB): margin improves from 21.4 dB to 22.1 dB.
For the failing channel after Fix 1 only (23.56 dB): margin improves from 12.4 dB to 13.1 dB.

The connector improvement provides a small benefit (0.7 dB). It does not rescue a failing channel on its own, but every dB of margin is valuable for manufacturing yield at the upper end of trace length tolerances.

---

### Step 6: Skin-Effect vs Dielectric Loss Comparison (Question 4)

For the 18-inch motherboard trace on Megtron 6 at 8 GHz:

**Skin-effect loss:**
$$IL_{skin} = 18 \times 0.099 = 1.78\ \text{dB}$$

**Dielectric loss:**
$$IL_{diel} = 18 \times 0.142 = 2.56\ \text{dB}$$

**Dielectric loss dominates** even on Megtron 6 at 8 GHz.

The crossover frequency where $\alpha_{skin} = \alpha_{diel}$:

$$0.035\sqrt{f} = 2.3 \times 1.924 \times 0.004 \times f$$

$$\frac{0.035}{\sqrt{f}} = 0.01769$$

$$\sqrt{f} = 0.035 / 0.01769 = 1.98 \implies f = 3.9\ \text{GHz}$$

Below 3.9 GHz, skin-effect loss dominates. Above 3.9 GHz, dielectric loss dominates.

At 16 GHz (PCIe Gen 5 Nyquist): $\alpha_{skin} = 0.035 \times 4.0 = 0.14$ dB/inch; $\alpha_{diel} = 0.142 \times (16/8) = 0.284$ dB/inch. Dielectric loss is 2× larger.

This analysis explains why low-$D_f$ laminate materials are the most effective loss reduction strategy for frequencies above 5 GHz.

---

## Summary Table

| Parameter | Value |
|---|---|
| Original channel total IL (original parameters) | 14.58 dB |
| PCIe Gen 4 IL limit | 36.0 dB |
| Margin (original) | +21.4 dB (passes) |
| Modified (failing) channel total IL | 41.23 dB |
| Deficit | -5.23 dB (fails) |
| IL after laminate upgrade (Megtron 6, 30 in) | 23.56 dB |
| IL after laminate + backdrill | 19.56 dB |
| Skin-effect crossover frequency (Megtron 6) | 3.9 GHz |

## Key Takeaways

1. **Loss budget is additive in dB.** Each component's insertion loss is summed linearly in dB.
2. **Dielectric loss dominates above 4 GHz** on typical PCB traces, making laminate $D_f$ the primary lever for high-speed channels.
3. **FR4 is not suitable for PCIe Gen 4/5 traces longer than ~12 inches** due to $D_f = 0.020$.
4. **Via stubs are a disproportionate loss contributor** relative to their physical size. Backdrilling is mandatory for >10 Gbps links above 6+ GHz.
5. **Package loss is fixed** and must be known before PCB routing length targets can be set.
6. **Margin is valuable.** A 12 dB positive margin provides robust yield against trace etching tolerances ($\pm 0.5$ mil width → $\pm 1$ dB) and laminate thickness variation ($\pm 10\%$ → $\pm 0.5$ dB impedance).
