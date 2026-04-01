# Problem 03: Verifying SerDes Channel Compliance Against a Specification

## Problem Statement

You are an SI engineer at a company designing a 25GbE server NIC (Network Interface Card). The NIC uses a SerDes-based Ethernet PHY IC (25GBASE-KR, IEEE 802.3by) and connects to the server motherboard backplane via an OCuLink or OCP NIC connector. The SerDes channels run from the NIC ASIC to the connector, across the backplane connector, and on the motherboard to the CPU's Ethernet PHY or retimer.

**Channel topology:**

```
NIC ASIC SerDes TX ──[NIC PCB trace]──[NIC connector]──[MB connector]──[MB PCB trace]── CPU RX PHY
```

**Given measurements and parameters:**

- NIC PCB trace: 80 mm, Dk = 3.8, Df = 0.008 (low-loss laminate), trace width = 0.10 mm, stripline
- NIC card connector: -0.8 dB at 12.89 GHz (characterised by connector manufacturer)
- Motherboard connector: -0.8 dB at 12.89 GHz
- Motherboard PCB trace: 120 mm, Dk = 3.9, Df = 0.010 (mid-loss laminate)
- CPU RX PHY package: -0.6 dB at 12.89 GHz
- Vias: 3 total (NIC BGA escape, card connector, MB connector) — all backdrilled, each -0.3 dB at 12.89 GHz
- FEXT from adjacent lanes: measured $|S_{FEXT}|$ = -35 dB at 12.89 GHz
- Transmitter jitter (RJ): 0.20 ps rms from SerDes datasheet
- Transmitter differential swing: 800 mV peak-to-peak
- Receiver CTLE range: 0 to -20 dB at 12.89 GHz (fully adjustable)
- Receiver DFE: 5 taps, each with ±100 mV range

**IEEE 802.3by 25GBASE-KR channel specification (relevant clauses):**

- Maximum channel insertion loss at 12.89 GHz: -28.1 dB (Annex 93A, Reach 1 channel model)
- Maximum return loss ($|S_{11}|$) any frequency 50 MHz to 12.89 GHz: ≤ -12 dB (differential, at any port)
- Maximum FEXT from any single aggressor lane: -25 dB at 12.89 GHz
- Minimum received eye height (before equalization): referenced by COM metric
- COM requirement: ≥ 0 dB

**Additional measured S-parameter data (from a VNA measurement of the complete channel):**

| Frequency (GHz) | $|S_{21}|$ (dB) | $|S_{11}|$ (dB) |
|---|---|---|
| 0.1 | -0.5 | -28 |
| 1.0 | -2.8 | -22 |
| 4.0 | -8.1 | -18 |
| 8.0 | -14.3 | -14 |
| 12.89 | -22.6 | -11 |

**Tasks:**

1. Compare the measured $|S_{21}|$ against the 25GBASE-KR channel insertion loss specification.
2. Evaluate whether the return loss violates the specification.
3. Estimate whether the FEXT is within specification.
4. Perform a simplified COM analysis using the given parameters to estimate the COM margin.
5. Identify any violations and propose corrective actions.

---

## Solution

### Task 1 — Insertion Loss Compliance

The 25GBASE-KR specification limits channel insertion loss to -28.1 dB at the Nyquist frequency of 12.89 GHz.

**Measured:** $|S_{21}|$ = -22.6 dB at 12.89 GHz.

**Comparison:**

$$IL_{measured} = -22.6 \text{ dB} > IL_{limit} = -28.1 \text{ dB (less negative = lower loss)}$$

**Result: PASS.** The channel has $28.1 - 22.6 = 5.5$ dB of margin against the insertion loss limit at Nyquist.

**Cross-check with calculated estimate:**

Using the loss model from Problem 02 methodology:

At 12.89 GHz:
- NIC trace 80 mm (Dk=3.8, Df=0.008): $\alpha_d = 27.3 \times 0.008 \times \sqrt{3.8} / (c/12.89 \times 10^9 \times 10^3) = 27.3 \times 0.008 \times 1.949 / 23.27 = 0.01822 \text{ dB/mm}$

  Conductor roughness factor at 12.89 GHz: $\delta_s = 2.1/\sqrt{12.89} = 0.585$ μm, roughness ~3× $\delta_s$ → $K_{SR} \approx 1.7$

  $\alpha_c = 0.0515 \times (12.89/16)^{0.5} \times (1.7/1.8) = 0.0515 \times 0.897 \times 0.944 = 0.0437 \text{ dB/mm}$

  Wait — more carefully: $\alpha_{c,rough}(16 \text{ GHz}) = 0.0515$ dB/mm (from Problem 02). At 12.89 GHz: $\alpha_{c,rough}(12.89) = 0.0515 \times \sqrt{12.89/16} = 0.0515 \times 0.897 = 0.0462 \text{ dB/mm}$

  $\alpha_{NIC} = 0.0462 + 0.01822 = 0.0644 \text{ dB/mm}$; $IL_{NIC} = 80 \times 0.0644 = -5.15 \text{ dB}$

- MB trace 120 mm (Dk=3.9, Df=0.010):
  $\alpha_d = 27.3 \times 0.010 \times 1.975 / 23.27 = 0.02317 \text{ dB/mm}$
  $\alpha_c = 0.0462 \text{ dB/mm}$ (approximately same copper)
  $\alpha_{MB} = 0.0462 + 0.02317 = 0.0694 \text{ dB/mm}$; $IL_{MB} = 120 \times 0.0694 = -8.33 \text{ dB}$

- NIC connector: -0.8 dB
- MB connector: -0.8 dB
- CPU package: -0.6 dB
- Vias (3 × -0.3): -0.9 dB

Calculated total: $-5.15 - 8.33 - 0.8 - 0.8 - 0.6 - 0.9 = -16.6 \text{ dB}$

The measured value of -22.6 dB is worse by approximately 6 dB, suggesting either the connectors are performing worse than quoted, additional coupling losses not modeled, or the laminate $Df$ is higher at 12.89 GHz than specified (Df increases with frequency for most laminates). The measurement should be trusted over the calculation.

---

### Task 2 — Return Loss Compliance

The 25GBASE-KR specification requires $|S_{11}| \leq -12$ dB at any frequency from 50 MHz to 12.89 GHz.

**Measured return loss:**

| Frequency (GHz) | $|S_{11}|$ (dB) | Limit (dB) | Pass/Fail |
|---|---|---|---|
| 0.1 | -28 | -12 | Pass |
| 1.0 | -22 | -12 | Pass |
| 4.0 | -18 | -12 | Pass |
| 8.0 | -14 | -12 | Pass |
| 12.89 | -11 | -12 | **FAIL** |

**Result: FAIL at 12.89 GHz.** The measured return loss of -11 dB violates the -12 dB specification by 1 dB.

**Physical interpretation:**

$|S_{11}| = -11$ dB corresponds to a reflection coefficient magnitude of:

$$|\Gamma| = 10^{-11/20} = 0.282$$

This means 28.2% of the incident voltage is being reflected. The return loss violation at 12.89 GHz (near Nyquist) degrades the channel's frequency response through destructive interference between the forward and reflected waves, creating ripple in the $|S_{21}|$ response. Even though the average $|S_{21}|$ passes, the worst-case $|S_{21}|$ at frequencies near the ripple trough could be 1–2 dB worse than the average.

**Likely cause:** A 1 dB violation at Nyquist, but passing at all lower frequencies, is the signature of a via transition with a resonance slightly below 12.89 GHz. Check the via stub lengths. A backdrilled via with a 0.25 mm residual stub has a resonance at:

$$f_{res} = \frac{1}{4 \times 0.25 \times 6.59 \times 10^{-3}} = \frac{1}{6.59 \times 10^{-4}} \approx 1517 \text{ GHz}$$

That is far too high. Consider a via with a BGA capture pad that creates a capacitive discontinuity. The via's capacitance and inductance form a resonant structure. A via with 0.5 pF capacitance and 0.2 nH inductance resonates at:

$$f_{res} = \frac{1}{2\pi\sqrt{LC}} = \frac{1}{2\pi\sqrt{0.5 \times 10^{-12} \times 0.2 \times 10^{-9}}} = \frac{1}{2\pi \times 3.16 \times 10^{-11}} \approx 5 \text{ GHz}$$

The resonance is at 5 GHz, but its effect on return loss extends up to 12.89 GHz. Alternatively, an impedance mismatch at the connector interface could cause this pattern.

---

### Task 3 — FEXT Compliance

The 25GBASE-KR specification limits FEXT from any single aggressor to -25 dB at 12.89 GHz.

**Measured:** $|S_{FEXT}|$ = -35 dB at 12.89 GHz.

**Result: PASS** with 10 dB margin.

$-35 \text{ dB} < -25 \text{ dB (limit)} \rightarrow$ the FEXT is 10 dB better than required.

**FEXT amplitude in absolute terms:**

The FEXT voltage at the victim receiver relative to the transmitter voltage:

$$V_{FEXT}/V_{TX} = 10^{-35/20} = 0.01778 = 1.778\%$$

For an 800 mV swing: $V_{FEXT} = 0.01778 \times 400 \text{ mV} = 7.1 \text{ mV}$ (amplitude relative to single-ended swing).

This 7.1 mV FEXT noise must be added to the total noise budget. In isolation, this is not significant for a receiver with ~250 mV signal amplitude at the slicer, but it contributes to the integrated noise floor.

---

### Task 4 — Simplified COM Analysis

**Step 1 — Signal amplitude after channel and CTLE at the slicer.**

Received signal amplitude before equalization (at the receiver input):

$$A_{rx} = A_{tx} \times 10^{IL_{S21}/20} = 400 \text{ mV} \times 10^{-22.6/20} = 400 \times 0.0741 = 29.6 \text{ mV (peak)}$$

(Using single-sided amplitude: $A_{tx} = 800/2 = 400$ mV peak single-sided differential.)

After CTLE with optimum setting (-22.6 dB boost at Nyquist to fully compensate the channel):

The CTLE cannot perfectly compensate a real channel (it is a smooth rational function, not the exact inverse of the channel). In practice, the CTLE removes most of the low-frequency roll-off and the DFE handles the residual ISI. Assume the CTLE is set to provide 20 dB of boost at Nyquist (near its maximum of 20 dB), partially correcting the channel.

Residual channel loss after CTLE: $-22.6 + 20 = -2.6$ dB at Nyquist.

Signal amplitude after CTLE:

$$A_{after-CTLE} = 400 \times 10^{-2.6/20} = 400 \times 0.740 = 296 \text{ mV}$$

The DFE with 5 taps cancels post-cursor ISI. Assume residual ISI after DFE: 10 mV rms.

**Effective signal amplitude at slicer:** $A_{slicer} \approx 296$ mV (accounting for DFE reduction of eye closure).

**Step 2 — Noise contributions.**

**Transmitter RJ:**

$$\sigma_{J} = 0.20 \text{ ps rms}$$

Signal slope at transition (rise time ≈ 30 ps for 25 GHz bandwidth):

$$\frac{dV}{dt} = \frac{0.8 \times A_{tx}}{t_r} = \frac{0.8 \times 400}{30 \times 10^{-3}} \approx 10.7 \text{ V/ns}$$

$$\sigma_{amp,J} = \sigma_J \times \frac{dV}{dt} = 0.20 \times 10^{-12} \times 10.7 \times 10^9 = 2.14 \text{ mV rms}$$

**FEXT noise:**

From the measurement: $V_{FEXT} = 7.1$ mV peak. Assuming Gaussian distribution (multiple aggressors), $\sigma_{FEXT} \approx 7.1/3 = 2.4$ mV rms.

**Receiver thermal noise:**

Assuming -57 dBm integrated noise in 12.89 GHz bandwidth:

$$P_{noise} = 10^{-57/10} \text{ mW} = 2.0 \times 10^{-6} \text{ mW} = 2.0 \text{ nW}$$

$$V_{noise} = \sqrt{P \times Z} = \sqrt{2.0 \times 10^{-9} \times 50} = \sqrt{10^{-7}} = 3.16 \times 10^{-4} \text{ V} \approx 0.316 \text{ mV rms}$$

**Residual ISI after DFE:**

$$\sigma_{ISI} \approx 10 \text{ mV rms (estimated)}$$

**Return loss induced noise:**

The -11 dB return loss at Nyquist creates a reflected wave that re-enters the signal path. The reflected power is:

$$P_{reflected} = |\Gamma|^2 \times P_{incident} = (0.282)^2 = 0.0795 = 7.95\%$$

This adds a deterministic jitter component and amplitude variation at the receiver. Approximate additional noise: ~5 mV rms (estimated based on reflected wave amplitude × equalized response).

$$\sigma_{RL} \approx 5 \text{ mV rms}$$

**Total noise (RSS):**

$$\sigma_{total} = \sqrt{\sigma_J^2 + \sigma_{FEXT}^2 + \sigma_{rx}^2 + \sigma_{ISI}^2 + \sigma_{RL}^2}$$

$$\sigma_{total} = \sqrt{2.14^2 + 2.4^2 + 0.316^2 + 10^2 + 5^2} = \sqrt{4.58 + 5.76 + 0.1 + 100 + 25} = \sqrt{135.4} \approx 11.6 \text{ mV rms}$$

**Step 3 — Compute COM.**

$$SNR = 20\log_{10}\left(\frac{A_{slicer}}{\sigma_{total}}\right) = 20\log_{10}\left(\frac{296}{11.6}\right) = 20\log_{10}(25.5) \approx 28.1 \text{ dB}$$

Required SNR for BER = $10^{-15}$ (NRZ): $Q = 8.0$, $SNR_{req} = 20\log_{10}(8.0) = 18.1$ dB.

$$COM_{approx} = 28.1 - 18.1 = +10.0 \text{ dB}$$

**Result: COM ≈ +10 dB.** The channel passes COM by approximately 10 dB.

**Sensitivity of COM to the return loss violation:**

If the return loss induced noise were zero (ideal return loss): $\sigma_{RL} = 0$.

$$\sigma_{total,ideal-RL} = \sqrt{4.58 + 5.76 + 0.1 + 100} = \sqrt{110.4} = 10.5 \text{ mV rms}$$

$$COM_{ideal-RL} = 20\log_{10}(296/10.5) = 20\log_{10}(28.2) = 29.0 \text{ dB} → COM = +10.9 \text{ dB}$$

The return loss violation reduces COM by approximately 0.9 dB — marginal, but contributes to overall degradation.

---

### Task 5 — Violations and Corrective Actions

**Confirmed violation:**

| Check | Result | Measured | Limit | Margin |
|---|---|---|---|---|
| Insertion loss at 12.89 GHz | PASS | -22.6 dB | -28.1 dB | +5.5 dB |
| Return loss at 12.89 GHz | **FAIL** | -11 dB | -12 dB | -1 dB |
| FEXT at 12.89 GHz | PASS | -35 dB | -25 dB | +10 dB |
| COM | PASS | +10 dB (est.) | ≥ 0 dB | +10 dB |

**Return loss violation (-11 dB vs -12 dB requirement):**

The failure is at the Nyquist frequency specifically, suggesting a resonant reflection source at or near 12.89 GHz.

**Investigation steps:**

1. **Port-by-port S-parameter de-embedding:** Measure $S_{11}$ at each interface separately (NIC ASIC output, NIC card connector, MB connector, CPU input). The frequency at which $|S_{11}|$ peaks identifies the physical location.

2. **Via capacitance audit:** Calculate the via capacitance for each of the 3 via transitions:

$$C_{via} \approx \frac{1.41 \times \epsilon_r \times T}{\ln(D_{antipad}/D_{via})}$$

where $T$ is the board thickness traversed, $D_{antipad}$ is the antipad diameter, and $D_{via}$ is the via diameter. Reduce antipad size or increase it (depending on whether capacitive or inductive mismatch dominates) to move the resonance above 12.89 GHz.

3. **Connector return loss re-measurement:** Connectors quoted as -0.8 dB insertion loss may have poor return loss at 12.89 GHz not specified in the data sheet. Request full S-parameter data from the connector manufacturer and simulate the complete channel.

**Proposed corrective action:**

Given the 1 dB violation margin and the COM passing with +10 dB margin, the most pragmatic approach is:

**Option A (preferred if board respin is not planned):**

- Verify that the actual COM from the full MATLAB simulation passes (the simplified estimate above has ±3 dB uncertainty).
- Request a specification waiver from the system architect documenting the 1 dB return loss violation with supporting COM analysis showing +10 dB margin.
- This is a common approach in practice: a 1 dB RL violation that is compensated by 10 dB COM margin is often accepted with documented justification.

**Option B (for next board revision):**

- Increase the antipad diameter on the via transitions to reduce via capacitance. A 10% increase in antipad diameter reduces $C_{via}$ by approximately 15–20%, shifting the resonant frequency upward.
- Re-route the connector footprint with RF-optimised pads (narrower SMD pad width to reduce capacitance at the connector-to-PCB transition).
- Simulate the revised via model before PCB fabrication to verify return loss improvement of ≥ 1.5 dB at 12.89 GHz.

**Option C (if return loss root cause is the MB connector):**

- Replace the MB connector with a higher-performance variant that has -15 dB return loss at 12.89 GHz.
- Cost impact: connector upgrade may add $0.20–0.80 per port at volume.

---

## Summary and Key Learning Points

1. **Insertion loss and return loss are independent failures.** A channel can pass insertion loss and fail return loss simultaneously. Both must be checked for full 802.3by compliance.

2. **Return loss failures at Nyquist typically arise from resonant structures (vias, connector footprints).** The resonance frequency can be estimated from the geometry and moved above Nyquist by adjusting antipad dimensions or via aspect ratio.

3. **COM provides a consolidated pass/fail that accounts for equalization.** A channel with 5.5 dB of insertion loss margin and a minor return loss violation can still have a positive COM of +10 dB once equalization is included. Always run both the per-metric compliance check AND COM analysis.

4. **The CTLE cannot fully compensate a real channel.** The channel's $S_{21}$ response at Nyquist is -22.6 dB; the CTLE provides -20 dB maximum. The residual -2.6 dB attenuation after CTLE reduces the signal at the slicer, which is why the total noise budget must be tight.

5. **FEXT margin here (10 dB) provides significant noise budget relief.** If the system had been densely routed with poor trace-to-trace spacing, the FEXT contribution could have pushed the COM below 0 dB even with the channel insertion loss passing.

6. **Documentation and simulation evidence support specification waivers.** In production hardware, not every channel parameter will be perfectly within spec. The correct engineering response to a marginal violation is to run the full COM analysis with the actual S-parameters and present the result — positive COM justifies proceeding.
