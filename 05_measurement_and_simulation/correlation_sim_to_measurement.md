# Correlation: Simulation to Measurement

## Overview

Achieving good correlation between simulation and measurement is a core engineering discipline — and a frequent source of project delay when approached without a systematic methodology. Poor correlation is almost always traceable to a specific modelling error or measurement artefact rather than to the fundamental tools being wrong. This section covers the methodology for establishing and debugging sim-to-measurement correlation, the most common discrepancy sources, and how to triage a failing channel when simulation predicted passing. These topics come up regularly in senior SI/PI interviews, board bring-up discussions, and compliance failure reviews.

---

## Tier 1: Fundamentals

### Q1. What does "sim-to-measurement correlation" mean, and why is it important?

**Answer:**

Sim-to-measurement correlation is the process of comparing a simulation prediction to a physical measurement of the same structure and reconciling any differences. A design flow "correlates" when the simulation accurately predicts the measurement outcome — both in magnitude and in trend — to within an agreed tolerance.

**Why it matters:**

1. **Design confidence:** If simulation does not correlate, there is no basis for trusting the next design iteration. A simulation that passes but correlates poorly to hardware cannot be used to predict compliance.

2. **Root-cause debugging:** When hardware fails, correlation tells you whether to fix the model or fix the hardware. If the model is wrong, re-simulating will not help. If the hardware is wrong and the model was accurate, the simulation itself identified what to fix.

3. **Methodology validation:** Correlation is how companies validate their simulation methodology for a new material, process node, or tool version. Good correlation on a test vehicle means the same method can be trusted for the production design.

**Acceptable correlation tolerances (typical industry practice):**

| Parameter | Acceptable correlation error |
|---|---|
| Insertion loss $|S_{21}|$ | $\pm 1$ dB up to Nyquist |
| Return loss $|S_{11}|$ | $\pm 3$ dB (harder to correlate; sensitive to reference plane) |
| TDR impedance | $\pm 2\ \Omega$ on controlled-impedance sections |
| Eye height (simulation vs. scope) | $\pm 20$ mV |
| Eye width | $\pm 5\%$ of UI |
| PDN impedance | $\pm 2$ dB up to 500 MHz |

Tighter tolerances are achievable with careful methodology; looser tolerances indicate a model or measurement problem.

---

### Q2. List the most common reasons simulation insertion loss does not match measured insertion loss.

**Answer:**

Insertion loss discrepancy is the most common correlation failure. The reasons, in rough order of frequency:

**1. Dielectric constant ($D_k$) error:**

Simulation uses the nominal $D_k$ from the material datasheet (e.g., $D_k = 4.3$ at 1 GHz for standard FR-4). Actual $D_k$ varies with frequency ($D_k$ decreases at higher frequencies for FR-4), PCB fabrication lot, resin content, and glass weave style. A 5% $D_k$ error affects propagation delay and impedance but has a modest direct effect on insertion loss.

**2. Dissipation factor ($D_f$ / $\tan\delta$) error:**

Dielectric loss $\alpha_d \propto f \cdot \sqrt{D_k} \cdot D_f$. A 20% error in $D_f$ produces a 20% error in dielectric loss — which becomes the dominant error at high frequencies. Many ECAD tools use DC or 1 GHz $D_f$ values for dielectric characterisation; for 10+ GHz simulation, frequency-dependent $D_f$ from a causal Wideband Debye model is required.

**3. Copper roughness:**

Real PCB copper has a surface roughness (root-mean-square roughness $R_q$) of 0.3–3 $\mu$m depending on the foil type. Rough copper increases conductor loss by 1–5 dB/m at 10 GHz compared to a smooth conductor. Standard field solvers assume perfectly smooth copper. The Hammerstad-Jensen or Huray models correct for roughness:

$$\alpha_{c,rough}(f) = \alpha_{c,smooth}(f) \cdot K_{rough}(f)$$

where $K_{rough}$ is the correction factor. Not applying copper roughness correction is the most common cause of simulation underestimating measured insertion loss.

**4. Fixture and probe de-embedding errors:**

The raw measurement includes connectors, via transitions, and launch pads. If de-embedding is incomplete or the fixture model is wrong, the residual fixture loss is attributed to the DUT. This typically manifests as excess measured loss at high frequencies (connector transitions are more lossy than traces).

**5. Reference plane and trace width tolerance:**

PCB fabrication tolerances of $\pm 10\%$ on trace width and $\pm 10\%$ on dielectric height change $Z_0$ and thus affect loss slightly. More importantly, for microstrip, the distance from trace to reference plane changes the effective $D_k$ and thus the velocity and loss.

**6. Via stub resonance not in model:**

An unbackdrilled via stub creates a transmission null at the stub resonant frequency. If the simulation does not include the via stub (e.g., via modelled as a lumped inductor), the model will not show the notch.

---

## Tier 2: Intermediate

### Q3. Describe a systematic methodology for correlating a simulated S-parameter model to a VNA measurement of a PCB differential channel.

**Answer:**

**Phase 1 — Measurement quality assurance (before comparing to simulation):**

1. Verify VNA calibration: measure a known Through standard after SOLT calibration. $|S_{21}|$ should be $> -0.2$ dB and $|S_{11}|$ should be $< -40$ dB across the band. If not, recalibrate.

2. Verify fixture de-embedding: the de-embedded S-parameters should show $|S_{11}| > 10$ dB improvement over the raw data near the launch frequency. If the de-embedded $|S_{11}|$ is similar to the raw $|S_{11}|$, the fixture model is wrong or the calibration plane is misplaced.

3. Convert to mixed-mode: compute $|S_{DD21}|$ from the 4-port measurement. Verify $|S_{CD21}| < -25$ dB — excessive mode conversion indicates connector asymmetry, not a channel loss issue.

**Phase 2 — Simulation model preparation:**

1. Import the PCB stackup from the fabrication drawing (not the design intent). Use the fabricated layer thicknesses and finished copper weights, not the nominal values.

2. Use causal (frequency-dependent) dielectric properties. Obtain the Wideband Debye or Djordjevic-Sarkar parameters from the laminate vendor for the specific material and frequency range.

3. Include copper roughness correction using the Huray model coefficients from the copper foil specification sheet. For standard 1 oz RTF copper, typical Huray parameters are $R_s \approx 0.7\ \mu$m and $N_s \approx 20\ \text{spheres}/\mu\text{m}^2$.

4. Include the via models (3D HFSS or equivalent). Verify via stub length from the drill file and backdrill specification.

5. Use the connector's S-parameter model (supplied by the connector vendor) rather than a simplified lumped model.

**Phase 3 — Comparison:**

1. Overlay simulation and measurement $|S_{DD21}|$ on the same plot from 100 MHz to at least the 5th harmonic of the bit rate.

2. Note the frequency of correlation breakdown (where the curves diverge by more than 1 dB). This localises the source:

   - Divergence beginning at low frequency (< 1 GHz): likely DC resistance or connector model error
   - Divergence growing linearly with frequency: dielectric loss model error
   - Divergence growing as $\sqrt{f}$: conductor loss model error (skin effect, roughness)
   - Sharp notch in measurement not in simulation: via stub resonance missing from model
   - Periodic ripple in measurement not in simulation: reflection from a fixture element not de-embedded

3. Adjust one parameter at a time. Start with $D_f$ (biggest lever for high-frequency loss). Then copper roughness. Then via models. Never adjust multiple parameters simultaneously — this produces false correlation through parameter cancellation.

**Phase 4 — Convergence criterion:**

Correlation is accepted when the overlay meets the tolerance specifications listed in Q1. Document the final model parameters in a correlation report. These parameters become the "golden" values for the technology, usable on future designs.

---

### Q4. Simulation predicts the channel eye opening is 120 mV at $10^{-12}$ BER with 5 dB margin. Hardware measurement shows only 60 mV with the eye nearly failing. List your triage steps.

**Answer:**

A 50% reduction in eye opening (from 120 mV to 60 mV) is a severe discrepancy. The systematic triage approach:

**Step 1 — Verify the measurement is trustworthy.**

- Is the scope and probe bandwidth adequate for the bit rate? For 25 Gbps, the probe should be $\geq 20$ GHz.
- Is the test pattern the same PRBS length as the simulation (PRBS-31 vs PRBS-7 gives a different eye — PRBS-31 has more ISI)?
- Are all equalization settings (CTLE boost, DFE taps) in the hardware matched to the simulation model?
- Is the eye measured at the receiver pin or at a monitoring tap? A monitoring tap may attenuate the signal further.

**Step 2 — Measure the channel insertion loss with a VNA.**

Compare measured $|S_{DD21}|$ to the simulated $|S_{DD21}|$. If the measurement shows 3–5 dB more loss than simulation at Nyquist:

- This is the direct cause of the eye loss. Proceed to Q3 methodology to identify why the model underestimates loss.
- Common suspects: copper roughness not modelled; $D_f$ value too optimistic; via stubs not included.

**Step 3 — Check the transmitter output.**

Disconnect the channel and measure the transmitter output directly (into a 50 $\Omega$ load). Compare the measured eye amplitude to the specified $V_{TX}$. If the transmitter output is lower than specified:

- Transmitter supply voltage is below nominal (power integrity issue)
- Transmitter configuration register is not set to the expected pre-emphasis and swing setting
- IBIS model in simulation used a different configuration corner

**Step 4 — Check equalization settings.**

Measure the CTLE frequency response (if accessible) by injecting a sine sweep. Compare to the simulated CTLE gain curve. If the CTLE boost is lower than simulated:

- The CTLE is configured to a different setting (different register value or default reset state)
- The CTLE's actual $f_z$ and $f_p$ poles differ from the IBIS-AMI model

**Step 5 — Measure TDR on the channel.**

A TDR sweep reveals impedance discontinuities not captured in the model:

- Missing termination resistors (net open at end)
- Wrong termination values
- Manufacturing defect (trace break, via short, laminate void causing impedance step)

**Step 6 — Isolate simulation model components.**

Re-simulate the channel using the measured $|S_{DD21}|$ in place of the field solver model. If the eye prediction now matches hardware, the discrepancy was entirely in the channel model — proceed to root-cause the model as in Q3. If the eye prediction still does not match hardware with the measured channel, the discrepancy is in the IC model (IBIS-AMI inaccuracy) or in the measurement itself.

---

## Tier 3: Advanced

### Q5. Discuss the limitations of the Wideband Debye (Djordjevic-Sarkar) dielectric model. When does it fail to predict loss accurately?

**Answer:**

The Wideband Debye (or Djordjevic-Sarkar) model represents the complex permittivity of a PCB dielectric as a sum of Debye relaxation terms that remain causal and passive across all frequencies:

$$\varepsilon_r(f) = \varepsilon_\infty + \Delta\varepsilon \cdot \frac{\ln(f_{max}/f_{min})}{2\pi j} \cdot \ln\left(\frac{f_{max} + jf}{f_{min} + jf}\right)$$

The model enforces the Kramers-Kronig relations, ensuring that $D_k(f)$ and $D_f(f)$ are physically consistent (a material with frequency-dependent loss must also have frequency-dependent permittivity). It is fit to measured data at two or more reference frequencies.

**Limitations:**

**1. Single-relaxation approximation for complex materials:**

Standard FR-4 epoxy-glass contains multiple relaxation mechanisms (epoxy resin polarisation, glass fibre polarisation, moisture absorption) each with a different relaxation frequency. A single Debye term cannot simultaneously fit the measured $D_k$ and $D_f$ across a decade-wide frequency span. The Wideband Debye model uses a distributed approximation, but it still cannot capture sharp features (e.g., a narrow loss peak from moisture at 10 GHz).

**2. Glass weave effect (periodic dielectric perturbation):**

PCB dielectric contains a woven glass fibre reinforcement. The glass (low $D_f$) and resin (higher $D_f$) have different electrical properties. A differential pair traces over alternating glass and resin bundles. This periodic perturbation creates:

- Differential loss depending on the relative angle of the trace to the weave
- A form of mode conversion that is not captured in any homogeneous dielectric model (including Wideband Debye)

This is the glass weave skew effect — the primary cause of differential-to-common mode conversion in differential pairs.

**3. Moisture and temperature dependence:**

The Wideband Debye model is typically fit to dry room-temperature measurements. PCB dielectrics absorb moisture, which can increase $D_f$ by 10–30% at humid environments. Temperature also shifts the relaxation frequency. Neither effect is included in a fixed Wideband Debye fit.

**4. Frequency range fit quality:**

The model is fit at the frequencies where data is available. Above the highest measured frequency, the model extrapolates — and extrapolation may diverge from actual behaviour. For mm-wave designs ($> 50$ GHz), data must be measured at those frequencies; extrapolation from 10 GHz data is not reliable.

**When the model fails most significantly:**

- Differential-to-common mode conversion estimation (glass weave not modelled)
- Very high data rates ($> 50$ Gbps) where frequency-dependent dielectric behaviour at 25+ GHz is critical
- High-humidity environments
- Very precise delay matching where the temperature-dependent velocity shift matters

**Alternative models:** The Cole-Cole model and Havriliak-Negami model have additional fitting parameters that better capture non-Debye relaxation in materials with broad distributions of relaxation times. Some laminate vendors now supply Cole-Cole parameters for their high-speed materials.

---

### Q6. You are given a passing simulation (eye open, margin $> 3$ dB) and a failing board. After ruling out measurement error and transmitter issues, your TDR and VNA data show excellent correlation to the simulation model. What remaining sources could explain the discrepancy, and how do you investigate each?

**Answer:**

If the channel model (S-parameter) correlates to measurement but the full link simulation still predicts passing while hardware fails, the discrepancy must be in the IC models, the operating conditions, or the link configuration. Systematic investigation:

**1. IBIS-AMI model accuracy:**

The IBIS-AMI model's GetWave and Init phases may not accurately represent the silicon. Common inaccuracies:

- **DFE tap weights:** The simulated DFE may assume ideal tap weight adaptation, but the real device uses a specific adaptation algorithm with a finite step size. If the step size is too coarse for the channel, DFE residual ISI is higher than simulated.
- **CDR bandwidth mismatch:** The simulated CDR may have a different loop bandwidth than the silicon, affecting how jitter transfers through the CDR. A narrower CDR bandwidth reduces high-frequency jitter but increases low-frequency wander, affecting DFE performance.
- **Corner mismatch:** The IBIS-AMI model is typically characterised at the typical corner. The failing board may have devices at the fast-fast or slow-slow corner. Request worst-corner (slow-slow, high temperature, low supply) IBIS-AMI files from the IC vendor.

**Investigation:** Request the IC vendor's correlation data showing how well the AMI model matches silicon measurements for a known reference channel. If correlation data shows $> 20$ mV AMI model error, the model is the problem.

**2. Supply voltage out of specification:**

The IBIS model assumes nominal supply voltage. If the I/O supply is 5–10% below nominal due to PDN droop:

- Transmitter swing is reduced proportionally
- Driver rise time may increase (slower transistor switching at lower $V_{DD}$)
- Receiver threshold headroom is reduced

**Investigation:** Measure the SERDES I/O supply voltage under active data transmission with a power rail probe. Compare to the nominal operating voltage in the IBIS model header. Even 50 mV undershoot during data transmission can account for 20–30 mV of eye height loss.

**3. Crosstalk not included in simulation:**

Single-channel simulation does not include crosstalk from adjacent channels. In a PCIe x16 or DDR bus, multiple simultaneous switching aggressors inject FEXT and NEXT into the victim channel. If the simulation was run for an isolated pair but hardware has all channels switching simultaneously:

- Jitter injection from FEXT creates deterministic jitter
- NEXT power sum reduces the eye voltage margin

**Investigation:** Measure the channel eye with all adjacent lanes transmitting the same PRBS pattern (worst-case simultaneous switching). Compare to the measurement with only the victim lane active. A $> 20$ mV difference confirms crosstalk contribution.

**4. Spread spectrum clocking (SSC) not modelled:**

Many SerDes interfaces use spread spectrum clocking to reduce EMI. SSC imposes a slow ($\pm 0.5\%$, 30–33 kHz modulation) frequency variation on the transmit clock. The receiver CDR must track this variation. If the CDR bandwidth is insufficient, SSC induces excess jitter on the recovered clock. If the simulation used a clean clock and the hardware has SSC enabled, the measured eye will be wider in time (more jitter) than simulated.

**Investigation:** Measure with SSC disabled (if the device allows this via a register bit). If the eye improves significantly with SSC off, the CDR tracking bandwidth is the issue. Verify the AMI model includes SSC in the CDR model.

**5. PCB manufacturing variation at the system level:**

Even with correlated trace models, inter-layer registration, impedance etch variation, and via drill positional tolerance create variation across the board that is difficult to capture in a single simulation. A batch of boards will have a distribution of channel losses; a poorly correlated simulation uses the nominal model while the failing board represents the low end of the distribution.

**Investigation:** Measure the channel S-parameters on multiple boards from the same batch. If one board is consistently worse than the others and worse than the simulation by a systematic amount, fabrication outlier is the cause. Tighten the impedance tolerance specification for production screening.

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| Sim-to-measurement correlation | Comparison of simulated to measured results; quantifies model accuracy |
| $D_k$ (dielectric constant) | Real part of relative permittivity; governs propagation velocity and impedance |
| $D_f$ (dissipation factor) | $\tan\delta$; governs dielectric loss; increases with frequency |
| Copper roughness | Surface irregularity of PCB copper; increases conductor loss vs. smooth copper |
| Huray model | Snowball model for copper roughness loss correction |
| Wideband Debye model | Causal frequency-dependent dielectric model fit to measured $D_k$/$D_f$ data |
| Glass weave effect | Periodic $D_k$ variation from alternating glass bundles and resin; causes differential skew |
| De-embedding | Mathematical removal of fixture parasitics to isolate DUT S-parameters |
| IBIS-AMI corner | Characterisation of the AMI model at process/voltage/temperature extremes |
| SSC | Spread Spectrum Clocking — intentional clock frequency dithering to reduce EMI peak |
| FEXT | Far-End Crosstalk — coupling from aggressor to victim at the far end of the line |
| NEXT | Near-End Crosstalk — coupling from aggressor to victim at the near (source) end |
| TDR | Time Domain Reflectometer — identifies impedance discontinuities; first step in hardware triage |
| CDR bandwidth | Bandwidth of the clock and data recovery PLL; governs jitter tracking and filtering |
