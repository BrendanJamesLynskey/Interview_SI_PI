# IEEE P370 De-Embedding and S-Parameter Quality Metrics

## Overview

Measured S-parameters are almost never the S-parameters of the thing one wants to know about. They are the S-parameters of the **device under test (DUT)** plus everything on the path to it — cables, launches, fixtures, connectors, test board traces. The process of removing fixture contributions from measured data is called **de-embedding**, and for high-speed differential links it is governed by the IEEE P370-2020 standard. An equally important question is whether the resulting (de-embedded or raw) S-parameter file is *valid for time-domain simulation* — whether it is causal, passive, reciprocal, and free of singularities. Yuriy Shlepnev's (Simberian) framework for S-parameter quality metrics is widely adopted as the industry standard. This document covers both topics: how to de-embed correctly under IEEE P370, and how to validate an S-parameter file before using it in simulation.

---

## Tier 1: Fundamentals

### Q1. What is de-embedding, and why is it needed?

**Answer:**

**The measurement reality:**

A VNA measures S-parameters at its cal plane, which is typically at the end of the cable where a calibration kit (SOLT, TRL, or electronic cal) was applied. But the DUT of interest is some distance beyond that cal plane — on a PCB, accessed through SMA connectors, fixture launches, and a short trace. The measurement thus includes:

$$S_{measured} = [S_{fixture,\ input}] \otimes [S_{DUT}] \otimes [S_{fixture,\ output}]$$

where "$\otimes$" denotes cascading in the T-parameter domain.

**De-embedding goal:** recover $[S_{DUT}]$ from $[S_{measured}]$ by removing the known fixture contributions.

**Why it is needed:**

1. **Fixture contributions are significant:** SMA launches typically add 0.5–2 dB of IL and noticeable return-loss ripple; a short fixture trace adds another 0.5–1 dB. At 25+ GHz these can dominate a short DUT's own loss.
2. **Vendor compliance reports demand de-embedded data:** PCIe CEM, 802.3 COM, and USB-IF all specify the measurement reference plane at the DUT's launch point, not at the VNA cal plane.
3. **Channel modelling:** to concatenate DUT S-parameters with a PCB channel model in simulation, the DUT file must not double-count the fixture losses.

**Common approaches, from simplest to most rigorous:**

- **Port extension / phase rotation:** a quick hack — subtract a uniform phase delay from each port to "move" the reference plane. Ignores fixture losses; use only for rough checks.
- **Coefficient de-embedding with a known fixture model:** if a high-fidelity EM model of the fixture exists, subtract it mathematically. Limited by fixture model accuracy.
- **2X-Thru de-embedding:** measure a "through" structure that is *exactly twice* the fixture (two fixtures back-to-back). Mathematically split into two identical fixture halves and subtract one half from each side of the DUT measurement. Standard PCB-measurement approach.
- **IEEE P370 de-embedding:** a rigorous extension of 2X-Thru with quality metrics. Industry standard post-2020.

---

### Q2. What is 2X-Thru de-embedding, and when does it work well?

**Answer:**

**Procedure:**

1. **Build a "2X-thru" test structure** on the same PCB as the DUT — a trace of length exactly equal to twice the fixture length, with identical launches at each end but no DUT in between.
2. **Measure the 2X-thru** on the VNA. The measured S-parameters are $S_{2X}$.
3. **Decompose $S_{2X}$ into two identical halves:** mathematically split the T-matrix of $S_{2X}$ into $T_{half} \cdot T_{half}$, where $T_{half}$ is the single-fixture T-matrix.
4. **De-embed the DUT** by pre-multiplying the measured DUT T-matrix by $T_{half}^{-1}$ on each side.

**Assumptions:**

- The 2X-thru is *truly* twice the fixture — same PCB trace width, same layer, same connectors, same manufacturing batch.
- The fixture is **symmetric and reciprocal**.
- No significant mode conversion in the fixture.

**When it works well:**

- Symmetric single-ended or differential test fixtures.
- Linear passive fixtures with predictable losses.
- PCB measurements up to 20 GHz.

**When it fails:**

- The 2X-thru has different trace width or routing layer than the DUT access — resulting in impedance mismatch at the DUT reference plane.
- Significant connector variation: the input and output connectors have different contact pressure/angle, so they are not actually "identical halves".
- Measurement noise or small calibration errors cause the T-matrix square-root operation to be ill-conditioned.

These failure modes motivated IEEE P370, which adds quality metrics to quantify how good the 2X-thru-based de-embedding actually is and provides corrections when it is not good.

---

### Q3. What is IEEE P370, and what does it add to 2X-thru?

**Answer:**

**IEEE P370-2020** is an IEEE standard titled *"Electrical Characterization of Printed Circuit Board and Related Interconnects at Frequencies up to 50 GHz"*. It codifies best-practice de-embedding and adds formal quality metrics.

**Key additions over plain 2X-thru:**

**1. Impedance correction:**

P370 does not assume the fixture impedance is 50 Ω (or 100 Ω differential). It measures the fixture impedance from the 2X-thru TDR response and applies a correction. If the 2X-thru is at 48 Ω while the DUT is measured in a 50 Ω environment, this correction prevents a false impedance step at the de-embedding reference plane.

**2. Fixture quality metrics:**

P370 defines numerical scores that quantify the validity of the de-embedding:

- **Metric 1 — Self-consistency:** after de-embedding, the fixture's own S-parameters computed from the 2X-thru should satisfy consistency checks.
- **Metric 2 — Passivity preservation:** de-embedded DUT data must remain passive (no frequency where eigenvalues exceed 1).
- **Metric 3 — Causality preservation:** de-embedded data must still be causal.

Each metric has a numeric score; P370 recommends rejecting de-embedding if any score exceeds a published threshold.

**3. Common-mode handling:**

For differential fixtures with mode conversion, P370 extends the method to handle the mixed-mode 4×4 matrix rigorously rather than assuming pure differential.

**4. Reporting standard:**

A P370-compliant de-embedding report includes the raw measurement, the 2X-thru measurement, the computed fixture halves, the de-embedded DUT, *and* all quality metrics. This enables peer review.

**Adoption:**

P370 is the required method for any high-stakes channel characterisation as of 2022. Most modern VNA post-processing software (Keysight PLTS, Rohde & Schwarz, Anritsu, Amphenol CP Lab) supports P370 natively.

---

## Tier 2: Intermediate

### Q4. What are the quality metrics for S-parameter files (causal, passive, reciprocal), and how are they computed?

**Answer:**

A valid S-parameter model must satisfy three physical requirements:

**1. Causality:**

$h(t) = 0$ for $t < 0$. In frequency domain: the real and imaginary parts of $S_{21}(f)$ are **Hilbert transform pairs** (Kramers–Kronig relations).

**Causality metric:** compute the Hilbert transform of $|S_{21}(f)|$ and compare to the measured phase. The deviation is the causality error. Well-behaved models have < 1% deviation; severely non-causal models have 10–30%.

**Consequences of violating causality:** pre-cursor ringing in time-domain simulation, misleading eye-diagram results.

**2. Passivity:**

The network cannot generate energy — the eigenvalues of $\mathbf{I} - \mathbf{S}(f)^H \mathbf{S}(f)$ must be non-negative at every frequency. Equivalent: the singular values of $\mathbf{S}(f)$ must be ≤ 1.

**Passivity metric:** compute the maximum eigenvalue violation across frequency. A violation of 0.001 is usually acceptable; 0.01 or more causes simulation divergence.

**Consequences of violating passivity:** SPICE simulations grow unboundedly; circuit simulators return "no convergence".

**3. Reciprocity:**

For passive networks without magnetic materials, $S_{ij} = S_{ji}$ for all port pairs. Reciprocity violation indicates a non-reciprocal element (amplifier, circulator, isolator) or a measurement error.

**Reciprocity metric:** $|S_{ij} - S_{ji}|$ at each frequency. PCB interconnects with no active elements should have this < 0.001.

**Where quality metrics live:**

- **Simberian SIMBEOR** and similar tools have built-in quality dashboards showing all three metrics.
- **IEEE P370** bundles these into its overall de-embedding quality score.
- **Touchstone 2.0** files can include explicit reference-impedance and frequency-point spacing information to help validate the data.

**Practical rule:** before running a long simulation, validate the S-parameter file's quality metrics. An hour of simulation on a bad file produces invalid results — a 10-second validation saves the hour.

---

### Q5. Describe how to correct a non-passive S-parameter file without destroying its frequency content.

**Answer:**

**Passivity enforcement algorithms** modify a non-passive S-parameter matrix to satisfy passivity while minimising the change to the data.

**Approach 1 — SVD-based correction:**

For each frequency point, compute the SVD of $\mathbf{S}(f)$: $\mathbf{S} = \mathbf{U}\mathbf{\Sigma}\mathbf{V}^H$. If any singular value $\sigma_i > 1$, clamp it to 1: $\tilde{\sigma}_i = \min(\sigma_i, 1)$. Reconstruct $\tilde{\mathbf{S}} = \mathbf{U}\tilde{\mathbf{\Sigma}}\mathbf{V}^H$.

This enforces passivity at each frequency independently. It is simple and fast but can distort the model significantly if many frequencies needed correction.

**Approach 2 — Optimisation-based correction:**

Find the passive $\tilde{\mathbf{S}}(f)$ that minimises $\sum_f \|\tilde{\mathbf{S}}(f) - \mathbf{S}(f)\|^2$ subject to passivity constraints. This preserves the model more faithfully. Algorithms like "Matrix Fitting" (Gustavsen) or "Adaptive Sampling Convex Optimisation" are used.

**Approach 3 — Rational-function fitting with enforced passivity:**

Fit the frequency response to a rational transfer function (poles and zeros) with all poles in the left half-plane and passive conditions enforced. The fit produces a passive model by construction.

**Caveats:**

1. **Correction hides measurement errors.** A large passivity violation usually indicates a measurement or calibration problem, not a minor model imperfection. Fixing it numerically without investigating the cause is dangerous.
2. **Over-aggressive correction changes the DUT behaviour.** A 5% reduction in $|S_{21}|$ near a resonance changes the resonance frequency in simulation.
3. **Best practice:** first identify the source of non-passivity (bad calibration, coupling between ports, mode-conversion modelling error) and fix that; only use numerical enforcement as a last resort.

**Common mistake:** applying passivity enforcement to a model that is slightly non-passive (say, max violation 0.002) and declaring victory. At 0.002 violation, most simulators run without issue and the fix is cosmetic.

---

### Q6. What are common causes of non-causality in measured S-parameter data, and how do you fix them?

**Answer:**

**Cause 1 — Phase reference errors:**

The VNA calibration establishes a phase reference plane. If that plane is not at the physical port of the DUT (e.g., cal plane is 1 mm inside the SMA connector), the measured phase has a constant offset that appears as apparent negative delay in the time domain.

**Fix:** apply a time-shift de-embedding (port extension) to move the reference plane to the correct physical location.

**Cause 2 — Truncated frequency sweep:**

If the sweep starts at a nonzero frequency (say, 100 MHz), the low-frequency part of the spectrum is missing. The IFFT that converts to time domain behaves as if $S_{21}(f) = 0$ for $f < 100$ MHz, causing ringing (Gibbs phenomenon) that looks like pre-cursor energy.

**Fix:** extrapolate $S_{21}(f)$ down to DC using physically sensible models (e.g., DC = measured low-frequency trend; avoid sudden step). Or, start the sweep at a lower frequency (10 MHz or below).

**Cause 3 — Frequency discretisation / aliasing:**

The IFFT requires uniform frequency spacing. If the measurement was exported with non-uniform or insufficient frequency samples, interpolation errors create non-causal artefacts.

**Fix:** re-measure with uniform sampling at or below the Nyquist frequency for the time window of interest.

**Cause 4 — Reciprocity violation from bad calibration:**

If SOLT or TRL was performed imperfectly, $S_{12} \neq S_{21}$ in ways that translate to non-causal time-domain behaviour.

**Fix:** recalibrate; verify cal by measuring a known-good standard before the DUT.

**Quick check for causality:**

Compute the IFFT of $S_{21}(f)$ and plot $h(t)$. Integrate the absolute value of $h(t)$ for $t < 0$ (pre-cursor energy). Well-behaved models have < 1% pre-cursor energy. More than 5% is a problem to investigate.

---

## Tier 3: Advanced

### Q7. How does causality enforcement via minimum-phase reconstruction work?

**Answer:**

**Concept:**

A minimum-phase system has all its poles and zeros in the left half-plane and is the "most compact" causal reconstruction of a given magnitude response. The Hilbert transform of the log-magnitude gives the phase:

$$\phi(f) = -\mathcal{H}\{\log|S_{21}(f)|\}$$

where $\mathcal{H}\{\cdot\}$ is the Hilbert transform. This procedure:

1. Discards any non-causal phase component.
2. Preserves the magnitude response exactly.
3. Produces a causal impulse response.

**Algorithm:**

1. Compute $\log|S_{21}(f)|$.
2. Compute its Hilbert transform → the minimum phase $\phi_{min}(f)$.
3. Replace the original phase with $\phi_{min}$: $S_{21,new}(f) = |S_{21}(f)|\,e^{j\phi_{min}(f)}$.
4. The new $S_{21}$ is causal and has the same magnitude response.

**When it is appropriate:**

- The original phase was corrupted by measurement error but the magnitude is trustworthy.
- The underlying network is known to be minimum-phase (passive PCB interconnects usually are).

**When it is inappropriate:**

- The network has inherent non-minimum-phase behaviour (all-pass filters, feedback structures) — forcing minimum phase changes the system's delay characteristics.
- The magnitude response has deep notches — minimum phase reconstruction ambiguity grows large near zeros.

**Common use:** 2X-thru de-embedding occasionally produces slightly non-causal DUT data at very high frequencies where measurement noise dominates. Minimum-phase reconstruction salvages the high-frequency portion while preserving the trustworthy lower-frequency data.

---

### Q8. Debug scenario: a channel simulation shows unexpected pre-cursor ringing in the eye diagram that did not appear in the lab. What is the likely cause and fix?

**Answer:**

**Symptom:** the simulated eye has significant pre-cursor energy — apparent response before the main edge arrival — but a BERT measurement of the same channel shows a clean edge with only post-cursor ISI (as expected from a causal channel).

**Diagnosis:**

The simulation is using a **non-causal S-parameter model**. The simulator dutifully computes what the model predicts, which includes causality-violating pre-cursor content that does not correspond to physical reality.

**Debug steps:**

1. **Check causality metric** on the S-parameter file. Compute the pre-cursor fraction of $h(t) = \text{IFFT}\{S_{21}(f)\}$. If > 1%, causality is the issue.

2. **Identify the non-causality source:**
   - If the fixture was de-embedded with 2X-thru, verify the de-embedding quality metrics (P370 method).
   - If the model came from a 3-D EM simulator, check the frequency range and sampling density.
   - If the model came from a vendor, ask for their causality quality report.

3. **Fix options (in order of preference):**
   - **Re-extract the model** with proper frequency range (DC to 2× Nyquist, uniform spacing).
   - **Apply minimum-phase reconstruction** (see Q7) if the magnitude is trustworthy.
   - **Apply a known-causal rational-function fit** (Vector Fitting tool) that preserves magnitude but enforces causality in the pole-zero structure.
   - **Last resort — use a first-order causality enforcement** in the simulator (shift phase to remove pre-cursor energy). This is a hack and should be documented.

**Preventive measure:**

Before any high-stakes simulation, run automated causality/passivity checks and reject non-conforming files. Many workflows integrate this as a pre-simulation gate:

```python
# Pseudocode
if causality_violation(s2p_file) > 0.01:
    raise Exception("S-parameter file causality violation exceeds threshold")
```

---

## Summary: De-Embedding and Quality Quick Reference

| Requirement | Metric | Typical accept threshold |
|---|---|---|
| Causality | Pre-cursor energy fraction | < 1% |
| Passivity | Max eigenvalue violation | < 0.001 |
| Reciprocity | $|S_{ij} - S_{ji}|$ | < 0.001 |
| P370 overall quality | Published P370 score | Green / Yellow / Red — accept Green, investigate Yellow |

| De-embedding method | Best for | Limitation |
|---|---|---|
| Port extension | Quick TDR checks | Ignores fixture loss |
| 2X-thru | Symmetric PCB fixtures | Requires identical 2X-thru coupon |
| IEEE P370 | All PCB measurements up to 50 GHz | Requires 2X-thru plus impedance correction |
| TRL calibration | In-fixture calibration | Requires additional cal standards |
| Coefficient-based | Known fixture model | Fixture model accuracy limits |

---

## Key References

- **IEEE P370-2020:** *Electrical Characterization of Printed Circuit Board and Related Interconnects at Frequencies up to 50 GHz*. Authoritative standard.
- **Yuriy Shlepnev (Simberian)**, "S-parameter Quality Metrics for Signal Integrity Applications", DesignCon papers 2010–2023.
- **B. Gustavsen, A. Semlyen**, "Rational Approximation of Frequency Domain Responses by Vector Fitting", IEEE Trans. Power Delivery 1999.
- **E. Bogatin et al.**, *Signal and Power Integrity — Simplified*, 3rd ed., Pearson 2017 — de-embedding chapter.
