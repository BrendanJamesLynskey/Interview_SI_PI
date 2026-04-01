# Problem 01: DDR Routing Rules Applied to a Board Layout Scenario

## Problem Statement

You are the SI engineer reviewing a DDR5-5600 memory subsystem layout for a server CPU platform. The design uses one RDIMM slot on a 12-layer PCB. The memory controller is on the CPU package (BGA, 1 mm pitch). The DIMM slot connector is 120 mm from the CPU BGA edge. A colleague has produced the initial routing pass and asks you to review it for signal integrity compliance.

The following routing measurements have been extracted from the PCB design tool:

**Clock (CK/CK# fly-by chain):**

- CK traces from CPU BGA escape to DIMM connector: 138 mm
- CK trace intra-pair skew: 18 mm length difference between CK+ and CK-

**Command/Address fly-by chain (CA[13:0], CS#):**

- CA[0] to CA[6]: stub length off the fly-by main route to each RDIMM Register (RCD) input = 8 mm
- CA[7] to CA[13]: stub length = 3 mm
- Series damping resistors (22 Ω) on CA lines: present

**Data byte lane 2 (DQ[23:16], DQS2/DQS2#, DM2#):**

- DQS2/DQS2# intra-pair skew: 7 mm length difference
- DQ16 length: 52 mm, DQ17: 55 mm, DQ18: 47 mm, DQ19: 66 mm, DQ20: 51 mm, DQ21: 54 mm, DQ22: 53 mm, DQ23: 49 mm
- DQS2: 52 mm, DQS2#: 59 mm (this is the intra-pair skew of 7 mm stated above)
- DM2#: 44 mm

**Layer assignment:**

- CK/CA signals: Layer 3 (stripline, between L2 ground plane and L4 ground plane)
- DQ/DQS signals: Layer 9 (stripline, between L8 and L10 ground planes)
- Vias: Through-hole, no backdrill, board thickness = 2.4 mm

**DDR5-5600 routing specification (from JEDEC and CPU platform design guide):**

- CA/CK stub length from fly-by main route: ≤ 2.5 mm
- CK intra-pair skew: ≤ 5 mil (≈ 0.127 mm)
- DQS intra-pair skew: ≤ 5 mil
- DQ-to-DQS matching within byte lane: ±15 mil (±0.38 mm)
- DQ-to-DM# matching within byte lane: ±15 mil
- Via stub resonance: stub length × 4 × $t_{pd}$ should not resonate within the signal bandwidth

**Tasks:**

1. Identify all routing rule violations in the above measurements.
2. Quantify the stub resonance concern for the through-hole vias.
3. Propose specific corrective actions for each violation.
4. Explain what would happen electrically if the CA stub violations were left uncorrected.

---

## Solution

### Task 1 — Identify All Routing Rule Violations

**Violation 1 — CK intra-pair skew (CRITICAL):**

Measured: 18 mm length difference between CK+ and CK-.
Specification: ≤ 0.127 mm (5 mil).

This is a factor of >140× over the limit. An 18 mm skew corresponds to approximately:

$$\Delta t_{CK} = 18 \text{ mm} \times 6.59 \text{ ps/mm} = 118.6 \text{ ps}$$

At DDR5-5600 the bit period is $1/5600 \times 10^{6} \approx 178.6$ ps; the CK period is $2 \times 178.6 = 357$ ps. A 118.6 ps intra-pair skew is 33% of the CK period — catastrophic. This converts a differential clock into a common-mode-dominated signal with very poor differential amplitude. All DRAMs and the RCD will receive a degraded clock and will likely fail to lock the on-die DLL.

**Violation 2 — CA stub length, CA[0]–CA[6] (CRITICAL):**

Measured: 8 mm stub length.
Specification: ≤ 2.5 mm.

All seven CA bits CA[0]–CA[6] have stubs 3.2× longer than the DDR5 limit. See Task 4 for the electrical consequence.

**Violation 3 — DQS2 intra-pair skew (CRITICAL):**

Measured: DQS2 = 52 mm, DQS2# = 59 mm → 7 mm difference.
Specification: ≤ 0.127 mm.

A 7 mm intra-pair skew on the DQS strobe:

$$\Delta t_{DQS2} = 7 \times 6.59 = 46.1 \text{ ps}$$

At DDR5-5600, the DQS period is 357 ps. A 46 ps DQS intra-pair skew is 12.9% of the DQS period. Write DQS centring training will attempt to compensate, but the skew also converts differential DQS to common-mode, reducing the differential strobe amplitude to:

$$V_{diff,degraded} \approx V_{diff,nominal} \times \cos\left(\pi \times \frac{\Delta t}{T_{DQS}}\right) = V_{diff} \times \cos(0.129\pi) \approx 0.916 \times V_{diff}$$

A ~9% amplitude reduction, combined with increased EMI from the common-mode component, makes this a high-severity violation.

**Violation 4 — DQ byte lane 2 length matching:**

Specification: All DQ and DM# signals within ±15 mil (±0.38 mm) of DQS2. Since DQS2 = 52 mm (taking the shorter of the pair as the reference), each DQ must be between 51.62 mm and 52.38 mm.

| Signal | Length (mm) | Deviation from DQS2=52mm | Within ±0.38mm? |
|---|---|---|---|
| DQ16 | 52 | 0 mm | Yes |
| DQ17 | 55 | +3 mm | **NO — +3 mm violation** |
| DQ18 | 47 | -5 mm | **NO — -5 mm violation** |
| DQ19 | 66 | +14 mm | **NO — +14 mm violation** |
| DQ20 | 51 | -1 mm | **NO — -1 mm violation** |
| DQ21 | 54 | +2 mm | **NO — +2 mm violation** |
| DQ22 | 53 | +1 mm | **NO — +1 mm violation** |
| DQ23 | 49 | -3 mm | **NO — -3 mm violation** |
| DM2# | 44 | -8 mm | **NO — -8 mm violation** |

Eight of nine DQ/DM signals violate the ±0.38 mm matching requirement. The worst case is DQ19 at +14 mm over the DQS reference, corresponding to:

$$\Delta t_{DQ19} = 14 \text{ mm} \times 6.59 \text{ ps/mm} = 92.3 \text{ ps}$$

At DDR5-5600, the DQ setup window ($t_{DS}$) is approximately 20 ps. A 92 ps skew between DQ and DQS at the DRAM is over 4× the setup time — the byte lane will not function without correction.

**Summary of violations:**

| Violation | Severity | Rule | Measured | Limit |
|---|---|---|---|---|
| CK intra-pair skew | Critical | ≤ 0.127 mm | 18 mm | 142× over |
| CA[0]–CA[6] stub length | Critical | ≤ 2.5 mm | 8 mm | 3.2× over |
| DQS2 intra-pair skew | Critical | ≤ 0.127 mm | 7 mm | 55× over |
| DQ17 byte lane matching | High | ±0.38 mm | +3 mm | 8× over |
| DQ18 byte lane matching | High | ±0.38 mm | -5 mm | 13× over |
| DQ19 byte lane matching | Critical | ±0.38 mm | +14 mm | 37× over |
| DQ20–DQ23 byte lane | High | ±0.38 mm | -1 to +3 mm | 3–8× over |
| DM2# byte lane matching | High | ±0.38 mm | -8 mm | 21× over |

---

### Task 2 — Via Stub Resonance

Through-hole vias with no backdrill on a 2.4 mm thick board have a stub that extends from the signal's exit layer to the opposite board surface.

For a signal on Layer 9 (assume Layer 9 is approximately 80% of the way through the board from Layer 1): the stub extends from Layer 9 to the bottom of the board.

Stub length estimate: $l_{stub} \approx 2.4 \times (1 - 0.80) = 0.48$ mm (stub below Layer 9 to board bottom).

Quarter-wave resonance frequency:

$$f_{resonance} = \frac{v_{pd}}{4 \times l_{stub}} = \frac{1}{4 \times l_{stub} \times t_{pd}} = \frac{1}{4 \times 0.48 \times 6.59 \times 10^{-3}} = \frac{1}{12.65 \times 10^{-3}} \approx 79 \text{ GHz}$$

This is above the signal bandwidth of DDR5-5600 (Nyquist $\approx 2.8$ GHz for the clock signal), so the Layer 9 vias are **not a resonance concern** at DDR5 speeds.

However, for Layer 3 (CK/CA signals), if Layer 3 is approximately 20% from the top:

Stub below signal layer: $l_{stub} \approx 2.4 \times (1 - 0.20) = 1.92$ mm.

$$f_{resonance} = \frac{1}{4 \times 1.92 \times 6.59 \times 10^{-3}} \approx 19.7 \text{ GHz}$$

Still above the DDR5-5600 Nyquist frequency. For DDR5, via stub resonance on a 2.4 mm board is not a primary concern at up to 6400 MT/s (Nyquist ≈ 3.2 GHz). However, if this design were pushed to DDR5-8400 (Nyquist ≈ 4.2 GHz) or if the vias had longer stubs due to deeper layer placement, backdrill would become necessary.

**Note:** While the resonance frequency is safely above the Nyquist, the via capacitance on every CA and DQ signal adds a lumped capacitive load of approximately 0.2–0.4 pF per via, which must be included in simulations. At DDR5-5600 this is acceptable but should be verified in post-route simulation.

---

### Task 3 — Corrective Actions

**Fix Violation 1 — CK intra-pair skew:**

This is almost certainly a routing error — the CK+ and CK- traces were not co-routed as a matched differential pair. The correct fix is to re-route both CK traces together as a differential pair in the PCB EDA tool:

1. Select both CK+ and CK- traces.
2. Apply the "differential pair routing" tool to co-route them with matched length.
3. Verify that no splits occur where one trace goes around an obstacle without the other.
4. Final intra-pair skew must be ≤ 5 mil = 0.127 mm. Check with the design rule check (DRC) after re-routing.
5. The 18 mm difference must be eliminated by adding meanders to the shorter trace (CK+) to match the length of CK-, or both traces can be shortened and matched together.

**Fix Violation 2 — CA stub lengths (8 mm → ≤ 2.5 mm):**

The CA stubs from the fly-by main route to the RDIMM RCD (Register Clock Driver) are too long. Options:

1. **Re-route the fly-by chain closer to the RCD input pins:** If the fly-by main route can be moved to pass within 2.5 mm of each RCD input ball, the stub length drops to specification. This may require moving component placement.
2. **Use the RCD's longer internal flyby path:** RDIMM RCDs have internal signal paths. Some DDR5 RCDs allow the CA signals to be routed with stubs up to 5 mm when a specific series termination value is used. Check the specific RCD's platform guide.
3. **If re-routing is not possible:** Run post-route simulation with the 8 mm stubs and the 22 Ω series resistors. The resistors may be effective enough to damp the stub reflections to within spec. If simulation shows eye opening > spec minimum, document the exception with simulation evidence.

**Fix Violation 3 — DQS2 intra-pair skew:**

Same root cause as the CK intra-pair skew — differential pair routing was not applied. Fix by re-routing DQS2+/DQS2- as a matched differential pair. Target: DQS2 and DQS2# lengths within ±0.127 mm of each other. The absolute length should be set to the median of the byte lane DQ lengths (approximately 52–55 mm based on corrected DQ lengths).

**Fix Violations 4 — DQ byte lane length matching:**

All DQ and DM# signals in byte lane 2 must be length-matched to DQS2 within ±15 mil (±0.38 mm). After fixing DQS2 intra-pair skew (new DQS2 reference length: 55 mm, matching DQS2#), each DQ must be between 54.62 mm and 55.38 mm.

Approach:
1. Identify the longest DQ in the byte lane: DQ19 at 66 mm.
2. DQ19 cannot be shortened without re-routing. Instead, set DQ19 as the byte lane reference length (66 mm) and add meanders to all other signals to match:
   - DQ16: add 14 mm of meanders
   - DQ17: add 11 mm of meanders
   - DQ18: add 19 mm of meanders
   - DQ20: add 15 mm of meanders
   - DQ21: add 12 mm of meanders
   - DQ22: add 13 mm of meanders
   - DQ23: add 17 mm of meanders
   - DM2#: add 22 mm of meanders
   - DQS2/DQS2#: adjust both traces to 66 mm length
3. Verify all resulting lengths are within 66 ± 0.38 mm.

Note: Adding meanders increases trace inductance and slightly increases propagation delay — this is acceptable and expected. The write levelling training will handle the resulting DQS-to-byte-lane offset.

---

### Task 4 — Electrical Consequence of Uncorrected CA Stub Violations

The CA[0]–CA[6] stubs of 8 mm, running on a PCB with $t_{pd} = 6.59$ ps/mm, have a round-trip delay of:

$$t_{rt} = 2 \times 8 \text{ mm} \times 6.59 \text{ ps/mm} = 105.4 \text{ ps}$$

At DDR5-5600, the CA signals are command-rate at the base clock (CK period ≈ 357 ps for DDR5-5600). The UI for the CA signals is 357 ps.

The reflection from the stub end arrives back at the fly-by junction after 105.4 ps. The reflection coefficient at the open stub end (assuming the RCD input is high-impedance) is approximately $\Gamma \approx +1$ (capacitively terminated stub end), with the reflection amplitude limited by the 22 Ω series resistor on the fly-by main route.

**Stub reflection voltage:**

The reflected waveform adds to the main signal at the fly-by junction with a time delay of 105.4 ps. For a 0 → 1 transition, the initial step voltage at the junction is:

$$V_{initial} = V_{swing} \times \frac{Z_{load}}{Z_0 + Z_{load}}$$

After the stub round-trip, the reflected wave modifies the waveform at 105.4 ps post-transition. At DDR5-5600, the next transition (minimum) could arrive at 178.6 ps. The stub reflection arrives at 105.4 ps — **59%** of the way to the next data edge. This creates a ringing artifact that overlaps the pre-cursor of the next bit edge.

**Pre-cursor ISI effect:**

The 8 mm stub reflection is equivalent to a pre-cursor ISI that the training algorithm cannot compensate. Write levelling only corrects phase alignment; it cannot correct waveform distortion caused by stub reflections. The CA signal arrives at DRAM 0 (closest to the controller) with less distortion, but each successive DRAM in the fly-by chain sees the cumulative reflections from all preceding stubs — DRAMs near the end of the chain see the worst distortion.

**Quantitative result:**

A SPICE simulation of a DDR5-5600 fly-by CA chain with 8 mm stubs and 22 Ω series resistors would typically show:
- Eye opening at the last DRAM reduced by 30–50% in height compared to a 2.5 mm stub design
- Pre-cursor ringing amplitude: 15–25% of signal swing
- Worst-case eye opening at 400 mV swing: approximately 200–250 mV (vs 320+ mV specification minimum)

The design would fail write training with a high probability, and even if training locked, the margins would be insufficient for reliable operation across voltage and temperature corners.

---

## Summary and Key Learning Points

1. **Intra-pair skew dominates differential signal quality.** Even a few mm of length mismatch between + and - signals of a differential pair catastrophically degrades the differential amplitude and introduces common-mode noise. PCB EDA differential pair routing tools exist specifically to prevent this — they must be used.

2. **DDR5 fly-by stub lengths are tighter than DDR4** (2.5 mm vs 5 mm) due to the higher data rate and increased sensitivity to stub reflections. Component placement must be planned to achieve short stubs before routing begins.

3. **Byte lane length matching must be to ±15 mil.** This is a tight tolerance requiring careful use of the PCB EDA tool's length-matching features. The byte lane as a whole can be at any absolute length; the inter-signal relative length within the lane is what matters.

4. **Via stub resonance is not typically a concern for DDR5 at 2.4 mm board thickness**, but must be recalculated if the board is thicker or the data rate increases to DDR5-8400 and beyond.

5. **The write levelling algorithm compensates for absolute delay differences between byte lanes, but cannot correct intra-lane DQ-to-DQS skew, intra-pair skew, or stub reflections.** PCB-level routing rules exist to bound these errors to values that training can handle.
