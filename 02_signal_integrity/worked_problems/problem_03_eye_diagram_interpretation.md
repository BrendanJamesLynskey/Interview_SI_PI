# Problem 03: Eye Diagram Interpretation and Issue Identification

## Problem Statement

You are debugging a 25 Gbps NRZ SerDes link that is failing production BER testing. The link is a chip-to-chip connection across a PCB with 12 inches of differential stripline routing, two vias, and a high-density connector. The receiver has a CTLE (programmable peaking 0–12 dB) and a 2-tap DFE.

You capture eye diagrams at two points in the signal chain using a 100 GHz equivalent-time sampling oscilloscope:

**Eye Diagram A — at the transmitter output (probed differentially at the chip package ball):**

- Eye height: 400 mV (differential, peak-to-peak)
- Eye width: 32 ps at BER = $10^{-12}$ (extrapolated from bathtub)
- UI = 40 ps (25 Gbps)
- Crossing point asymmetry: 0→1 transitions cross at −20 mV, 1→0 transitions cross at +20 mV (offset from zero differential)
- Edge distribution: smooth, single-peak Gaussian shape on each crossing

**Eye Diagram B — at the receiver input (probed at the PCB via pad before the RX chip):**

- Eye height: 45 mV (differential)
- Eye width: 18 ps at BER = $10^{-12}$
- Crossing point asymmetry: same pattern as Eye A — 0→1 at −20 mV, 1→0 at +20 mV
- Edge distribution: bimodal — two distinct crossing clusters separated by 8 ps

**Additional observations:**

- BER bathtub has a floor at approximately $10^{-8}$ (the curve does not drop below $10^{-8}$ regardless of sampling phase). This is called a **BER floor**.
- Enabling CTLE at maximum peaking (12 dB) improves eye height to 120 mV and eye width to 22 ps.
- Enabling the DFE (2 taps) further improves eye height to 160 mV but does NOT improve the BER floor.
- Running the link with PRBS7 instead of PRBS31 raises the BER floor from $10^{-8}$ to $10^{-5}$.

**Questions:**

1. What does the crossing-point asymmetry in both eyes tell you? Quantify the duty cycle distortion (DCD).
2. Identify all jitter components present and classify them (RJ, DDJ, PJ, DCD).
3. Explain the bimodal edge distribution in Eye B. What is its physical cause?
4. Explain the BER floor. What does it indicate, and why does the DFE not fix it?
5. What does the pattern-length sensitivity (PRBS7 vs PRBS31) tell you?
6. Propose a systematic debug plan to identify and fix the root cause, in order of priority.

---

## Worked Solution

### Analysis 1: Crossing-Point Asymmetry and DCD

**Observation:** The 0→1 transitions cross the differential zero at $-20$ mV and 1→0 transitions cross at $+20$ mV. This asymmetry persists through the channel (same offset in both Eye A and Eye B), confirming it is a transmitter-side characteristic rather than a channel artefact.

**Physical interpretation — Duty Cycle Distortion (DCD):**

DCD occurs when the rise time and fall time of the transmitter are different, or when the transmit driver has a DC offset that shifts one transition type earlier or later than the other.

The crossing voltage offset translates directly into a timing offset (DCD) via the signal slope at the crossing:

$$DCD_{pp} = \frac{\Delta V_{cross}}{dV/dt|_{cross}}$$

At the transmitter output, with a 400 mV differential swing and a rise time of approximately $t_r \approx 0.25 \times UI = 10$ ps (assuming a smooth transition):

$$\frac{dV}{dt}\bigg|_{cross} \approx \frac{0.8 \times 400\ \text{mV}}{10\ \text{ps}} = 32\ \text{V/ns}$$

The crossing offset is $\Delta V_{cross} = 20 - (-20) = 40$ mV:

$$DCD_{pp} = \frac{40\ \text{mV}}{32\ \text{V/ns}} = 1.25\ \text{ps}$$

This 1.25 ps DCD is a deterministic jitter component. At 25 Gbps (UI = 40 ps), DCD is $1.25/40 = 3.1\%$ of UI.

**Why the crossing is symmetric at $\pm 20$ mV (not at a non-zero single value):** Both polarities are offset in opposite directions from zero. This means the differential signal has equal positive and negative DCD — the driver is not generating a DC offset, but rather the 0→1 and 1→0 transitions have different characteristics (slightly different $dV/dt$ or drive strength). This is typical of a CMOS or CML driver where the pull-up and pull-down transistors are not perfectly matched.

---

### Analysis 2: Jitter Classification

**From Eye A (transmitter output):**

- **Smooth single-peak Gaussian crossings** → RJ present (no significant DDJ or PJ at the transmitter output; the channel hasn't added pattern-dependent distortion yet).
- **Eye width of 32 ps** out of 40 ps UI → TJ = $40 - 32 = 8$ ps.
- **DCD = 1.25 ps** (confirmed above).
- **Estimate $\sigma_{RJ}$:** With TJ = 8 ps and DCD = 1.25 ps: $DJ_{pp} \approx DCD_{pp} = 1.25$ ps; $14.07 \times \sigma_{RJ} = TJ - DJ_{pp} = 8 - 1.25 = 6.75$ ps; $\sigma_{RJ} = 6.75/14.07 = 0.48$ ps.

**Transmitter jitter summary:**
- $DCD_{pp} = 1.25$ ps
- $\sigma_{RJ} = 0.48$ ps
- $TJ(10^{-12}) = 8$ ps (0.2 UI) — acceptable for a 25 Gbps TX (typical spec: TJ $< 0.25$ UI = 10 ps)

**From Eye B (receiver input):**

- **Bimodal crossing distributions separated by 8 ps** → DDJ (data-dependent jitter) from ISI. The two crossing populations correspond to edges preceded by a short run vs. a long run of identical bits.
- **Eye width degraded from 32 ps to 18 ps** → The channel added additional jitter. Channel jitter contribution: $32 - 18 = 14$ ps additional TJ.
- **DCD crossing asymmetry unchanged** → confirms DCD originated at TX and passed through the channel unchanged (as expected — DCD is a low-frequency characteristic unaffected by channel high-frequency roll-off).

**Receiver input jitter summary:**
- $DCD_{pp} = 1.25$ ps (unchanged from TX)
- $DDJ_{pp} \approx 8$ ps (from bimodal separation — this is the peak-to-peak ISI-induced timing shift)
- $DJ_{pp} = DCD_{pp} + DDJ_{pp} \approx 9.25$ ps
- RJ: $\sigma_{RJ}$ from wall slope — comparing Eye B width to Eye A width suggests $\sigma_{RJ}$ is not significantly changed by the passive channel (passive channels don't amplify RJ). So $\sigma_{RJ} \approx 0.48$ ps still applies.
- Verification: $TJ = DJ_{pp} + 14.07\,\sigma_{RJ} = 9.25 + 14.07 \times 0.48 = 9.25 + 6.75 = 16$ ps. But measured TJ = $40 - 18 = 22$ ps. The discrepancy of 6 ps suggests either additional RJ at the receiver probe point, additional DDJ from longer patterns, or the bimodal separation underestimates the full DDJ range.

---

### Analysis 3: Bimodal Edge Distribution — Physical Cause

**Observation:** Eye B shows two distinct crossing clusters separated by 8 ps.

**Physical cause — ISI from channel dispersion:**

The 12-inch differential stripline has frequency-dependent insertion loss. At 25 Gbps, the Nyquist frequency is 12.5 GHz. At 12.5 GHz, 12 inches of typical low-loss stripline might have 15–20 dB insertion loss. This attenuates the high-frequency components of the signal.

The effect in the time domain: each bit transition's timing depends on whether it was preceded by a run of same-polarity bits (the DC level has settled and the edge starts from the "correct" level) or by alternating bits (high-frequency content is present, contributing to a faster transition). The "settled-DC" pattern crosses the threshold at a different time than the "alternating-bit" pattern.

**Two-population model:**

For a 3-tap channel with significant post-cursor $h[1]$:
- After a long run of 0s followed by a 1→0 transition: the residual post-cursor ISI from the run of 0s delays the edge crossing.
- After alternating 1010 pattern: no accumulated ISI, edge crosses at the "clean" position.

The 8 ps separation between the two crossing clusters is the peak-to-peak DDJ for this channel. The bimodal shape is consistent with the two most extreme ISI scenarios having different crossing times.

**PRBS test pattern effect:** With PRBS31, very long runs (up to 31 bits of the same value) are exercised, maximising the ISI accumulation and DDJ. With a shorter PRBS7, the maximum run length is 7 bits, so the most extreme ISI scenarios are not exercised. This explains why PRBS7 shows a better eye than PRBS31 — it does not stress the DDJ component fully. (This also explains the pattern-length sensitivity in the BER floor, addressed below.)

---

### Analysis 4: BER Floor — Cause and Why DFE Fails to Fix It

**Observation:** The BER bathtub does not fall below $10^{-8}$ regardless of sampling phase. This is a BER floor.

**What a BER floor indicates:**

A BER floor means there is a deterministic mechanism causing errors at a rate that does not depend on sampling phase. This is inconsistent with a jitter-limited failure (jitter creates sampling-phase-dependent BER). BER floors in NRZ links are caused by:

1. **Residual ISI from insufficient equalisation.** If the DFE cannot fully cancel post-cursor ISI (e.g., tap saturation), some ISI patterns always close the eye below the decision threshold, regardless of sampling phase.

2. **Crosstalk at a fixed amplitude** from a synchronous aggressor at a fixed phase relationship to the victim.

3. **Mode conversion** ($S_{dc21}$ jitter-like effect at fixed phase).

4. **Reflections from impedance discontinuities** that arrive at a fixed delay relative to the main cursor, creating a constant ISI pattern at specific bit patterns.

**Why the DFE does not fix the BER floor:**

Enabling the DFE improved eye height from 120 mV to 160 mV (40 mV improvement — post-cursor ISI is being cancelled correctly), but the BER floor persisted.

Two most likely explanations:

**Explanation A — Crosstalk-induced BER floor:**

An asynchronous crosstalk aggressor on the PCB is causing bit errors when the aggressor and victim signals are in a specific phase relationship. The phase relationship repeats because both operate from the same system reference clock (synchronous crosstalk). The DFE cancels ISI from the victim's own channel but cannot cancel crosstalk — the crosstalk is not correlated with the victim's own past decisions.

**Explanation B — Impedance discontinuity creating a deterministic ISI notch:**

A via stub resonance or connector discontinuity creates a sharp null in $S_{dd21}$ at a specific frequency. Bit patterns with spectral energy at that frequency always produce a corrupted decision. The DFE's tap span may not extend far enough in time to cancel the echo from the discontinuity.

**Clue from PRBS pattern dependence:** The BER floor worsens from $10^{-8}$ to $10^{-5}$ with PRBS7 vs PRBS31. If the floor were from additive crosstalk (which is independent of the victim's pattern), the floor would be the same for both PRBS lengths. The pattern dependence strongly suggests the floor is **data-pattern dependent** — pointing to ISI from a residual echo (a reflection from a via or connector) that the DFE is not cancelling.

**Why the DFE fails:** The reflection echo arrives at a delay longer than the DFE's 2-tap span. At 25 Gbps (UI = 40 ps), 2 DFE taps cover 2 × 40 = 80 ps of post-cursor. If the reflecting via stub creates an echo at $t = 150$ ps (3.75 UI), the DFE taps cannot reach it. The 3.75-UI post-cursor echo is outside the cancellation window.

---

### Analysis 5: Pattern-Length Sensitivity (PRBS7 vs PRBS31)

**Observation:** BER floor rises from $10^{-8}$ (PRBS31) to $10^{-5}$ (PRBS7).

**Interpretation:**

PRBS7 has shorter maximum run lengths (7 bits of the same value) vs PRBS31 (31 bits). The fact that PRBS7 gives *worse* BER floor than PRBS31 seems counter-intuitive at first — usually more challenging patterns are longer.

The explanation is: **the specific bit pattern that generates the worst-case echo falls outside PRBS7 but is present in PRBS31.** With PRBS7, fewer bit patterns are exercised. If the fatal pattern occurs rarely but at a high error rate when it does occur, a test with only 127 unique patterns (PRBS7 = $2^7 - 1 = 127$ patterns) might hit the fatal pattern more frequently as a fraction of total bits. A PRBS31 test ($2^{31} - 1 \approx 2 \times 10^9$ patterns) dilutes the fatal pattern among $10^{9}$ more patterns, reducing the effective BER even if the individual failure rate per occurrence is the same.

More likely: the longer runs in PRBS31 (up to 31 bits) accumulate more DC level, which *shifts* the threshold level and brings the marginal bit patterns into a safer margin region for certain reflections. PRBS7 lacks these DC-shift patterns and keeps the eye in a configuration where the reflection echo has larger impact.

**This confirms the ISI/reflection origin of the BER floor** rather than additive noise or crosstalk. A true crosstalk-induced floor would be largely pattern-independent (the aggressor is asynchronous and hits the victim in all patterns at the same statistical rate).

---

### Analysis 6: Systematic Debug Plan

**Priority 1 — Locate the reflection/echo source (BER floor root cause)**

1. Measure $S_{dd21}$ with a VNA from TX package ball to RX package ball. Look for a sharp notch in the insertion loss at a specific frequency.

2. Compute the IFFT (time-domain impulse response) of $S_{dd21}$. A discrete echo at time $t_{echo}$ after the main cursor indicates a reflection at distance $d = v_p \times t_{echo}/2$ from the launch point.

3. Correlate the echo timing to the physical design: two vias and one connector. Calculate expected echo delays:
   - Via stub echo delay: $2 \times L_{stub}/v_p$ (round-trip within the stub)
   - Connector reflection: round-trip from connector back to TX

4. If a via stub resonance is found: backdrill the via to remove the stub.

5. If a connector reflection is found: add appropriate impedance matching (stub tuning, or replace the connector).

**Priority 2 — Address DDJ / bimodal crossing (equalisation check)**

1. Sweep CTLE peaking from 0 to 12 dB while monitoring eye diagram. Record eye height and BER bathtub at each setting.

2. The optimal CTLE peaking is the one minimising BER. If optimal peaking is near maximum (12 dB), the channel may require more equalisation than the CTLE can provide — consider increasing DFE tap count.

3. Extend DFE to 5 taps (if the SerDes supports it) and check whether the BER floor improves. If DFE tap 3, 4, or 5 has a large weight after convergence, the echo/ISI source is confirmed to be more than 2 UI away.

**Priority 3 — Characterise and correct DCD (minor impact)**

1. Measure the TX crossing asymmetry with VNA-calibrated equipment. Confirm DCD = 1.25 ps.

2. If the SerDes has TX output impedance calibration or duty-cycle correction, enable it and re-measure.

3. DCD of 1.25 ps on a 40 ps UI (3.1%) is within typical spec (5%). This is not a compliance failure but should be noted for margin calculations.

**Priority 4 — Crosstalk assessment**

1. Identify all aggressor signals routed adjacent to the failing victim link.

2. Measure $S_{dc21}$ of the victim channel — high mode conversion ($|S_{dc21}| > -20$ dB) indicates differential pair imbalance converting aggressor common-mode noise to victim differential noise.

3. If a specific crosstalk aggressor is suspected: disable all other signals on the board, run the link with only the aggressor active (no victim data), and measure the induced noise on the victim using an oscilloscope.

**Root cause hypothesis (most likely):**

Given the pattern-dependent BER floor and DFE ineffectiveness, the most likely root cause is an **unbackdrilled via stub on one or both vias** creating a resonant echo. This is an extremely common failure mode in 25+ Gbps designs.

**Expected fix:** Backdrilling both vias will eliminate the stub resonance, remove the echo, and drop the BER floor below the BER noise floor of the BERT ($< 10^{-14}$). CTLE + 2-tap DFE will then be sufficient to close the residual ISI.

---

## Summary of Identified Issues

| Issue | Evidence | Root Cause | Fix |
|---|---|---|---|
| DCD 1.25 ps | Crossing asymmetry $\pm 20$ mV in Eye A | TX driver pull-up/pull-down mismatch | TX duty cycle correction or calibration |
| DDJ 8 ps bimodal | Bimodal crossings in Eye B | Channel ISI from 12 GHz roll-off | CTLE peaking + DFE (already partially addressed) |
| BER floor at $10^{-8}$ | Flat bathtub bottom; DFE ineffective | Via stub echo beyond DFE tap span | Backdrill vias; verify $S_{dd21}$ notch |
| Pattern sensitivity | PRBS7 worse than PRBS31 | Data-pattern dependent ISI/echo | Same fix as BER floor |
| Eye closure (45 mV) | 89% eye closure from 400 mV TX to 45 mV RX | Channel insertion loss + via stubs | Low-loss laminate + backdrill |

## Key Takeaways

1. **Eye height and width tell different stories.** Eye height reveals noise and ISI margins; eye width (or TJ at BER) reveals jitter decomposition.

2. **A BER floor is never a jitter problem.** Jitter creates a sampling-phase-dependent BER. A floor means a noise or ISI source that overwhelms the decision regardless of phase.

3. **DFE limitations are a key debug clue.** If DFE improves eye height but not BER, the ISI source has a delay beyond the DFE tap span. This points to reflections, not simple dispersive loss.

4. **Pattern-length sensitivity identifies DDJ vs. additive noise.** Worse BER with shorter PRBS = ISI origin. Unchanged BER = additive noise origin.

5. **Crossing asymmetry is always DCD.** If both 0→1 and 1→0 crossings are offset from zero in opposite directions, it is DCD from the transmitter, not a threshold calibration issue.
