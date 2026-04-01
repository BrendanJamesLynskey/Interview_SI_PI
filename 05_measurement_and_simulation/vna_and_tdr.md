# VNA and TDR

## Overview

The Vector Network Analyser (VNA) and Time Domain Reflectometer (TDR) are the two primary instruments for characterising transmission-line structures in SI/PI work. The VNA measures scattering parameters in the frequency domain with phase information; the TDR interrogates impedance profiles by launching a step and observing reflections in the time domain. Both require careful calibration and fixture management for results to be meaningful. Understanding their operating principles, calibration schemes, and limitations is essential for any engineer who develops, validates, or debugs high-speed channels.

---

## Tier 1: Fundamentals

### Q1. What physical quantity does a VNA measure, and how does it differ from a scalar network analyser?

**Answer:**

A VNA measures complex (magnitude and phase) S-parameters by comparing a reference signal to the signal transmitted or reflected at each port. For a two-port network the four S-parameters are:

$$S_{11} = \frac{b_1}{a_1}\bigg|_{a_2=0}, \quad S_{21} = \frac{b_2}{a_1}\bigg|_{a_2=0}, \quad S_{12} = \frac{b_1}{a_2}\bigg|_{a_1=0}, \quad S_{22} = \frac{b_2}{a_2}\bigg|_{a_1=0}$$

where $a_n$ is the normalised incident power wave at port $n$ and $b_n$ is the normalised reflected/transmitted wave. The measurement is performed over a swept frequency range; the result for each port pair at each frequency is a complex number (real and imaginary components, or equivalently magnitude and phase).

A scalar network analyser measures only the magnitude of $S_{21}$ or $S_{11}$ — it discards phase. This is sufficient for checking insertion loss against a mask, but it is inadequate for:

- Computing time-domain responses via inverse FFT (phase is required)
- Characterising delay-matched differential pairs
- Full de-embedding, which requires the complete complex S-matrix

**Why phase matters in SI:** The group delay of a channel — the frequency derivative of the transmission phase — determines inter-symbol interference (ISI). A channel with flat magnitude but non-linear phase distorts the eye diagram even without amplitude loss. A scalar instrument cannot reveal this.

**Common mistake:** Assuming a VNA measurement is inherently more accurate than a TDR. Each instrument is optimised for a different regime. A VNA excels at narrow-band to multi-GHz frequency characterisation; a TDR excels at spatial fault location and quick impedance profiling during bring-up.

---

### Q2. What is TDR, and what does the reflected waveform tell you about a transmission line?

**Answer:**

A Time Domain Reflectometer launches a fast-edge step voltage (or sometimes a pulse) into a transmission line and records the voltage waveform at the launch point over time. Any impedance discontinuity along the line partially reflects the incident wave; the time of arrival of each reflection identifies the discontinuity's distance from the launch port.

**Reflection coefficient at a discontinuity:**

$$\Gamma = \frac{Z_L - Z_0}{Z_L + Z_0}$$

where $Z_0$ is the source impedance (typically 50 $\Omega$) and $Z_L$ is the impedance of the discontinuity or load.

**Interpretation of the displayed waveform:**

| TDR waveform feature | Physical cause |
|---|---|
| Step up (positive $\Gamma$) | Impedance higher than reference (e.g., unterminated end, series gap) |
| Step down (negative $\Gamma$) | Impedance lower than reference (e.g., stub, capacitive via, short) |
| Ramp (gradual) up | Inductive discontinuity (series inductance) |
| Ramp (gradual) down | Capacitive discontinuity (shunt capacitance) |
| Flat plateau then return | Section of different-impedance transmission line |
| Oscillating ringing | Resonant structure (stub + capacitance) |

**Spatial resolution:**

The minimum resolvable feature size is determined by the rise time $t_r$ of the incident step and the propagation velocity $v_p$:

$$\Delta x_{\min} = \frac{v_p \cdot t_r}{2}$$

The factor of 2 appears because the reflected wave travels back to the source, so the round-trip time sets the resolution. For FR-4 ($v_p \approx 1.7 \times 10^8$ m/s) and a 35 ps rise-time step, the minimum resolvable length is approximately 3 mm.

---

### Q3. What is SOLT calibration? Describe each standard and what error it removes.

**Answer:**

SOLT (Short-Open-Load-Through) is the most common coaxial VNA calibration scheme. It characterises twelve systematic error terms (in the full two-port error model) by measuring four known impedance standards.

**The twelve systematic errors (two-port):**

Three error terms per port direction × two directions, plus six cross-terms. The dominant ones are:

- **Directivity:** leakage in the directional coupler inside the VNA
- **Source match:** impedance mismatch at the test port
- **Reflection tracking:** frequency response of the reflection path
- **Transmission tracking:** frequency response of the through path
- **Load match:** imperfect termination at the load port
- **Isolation:** leakage between ports not through the DUT

**The four standards:**

| Standard | Impedance | Removes |
|---|---|---|
| Short ($S_{11} = -1$, $\Gamma = -1$) | $Z = 0\ \Omega$ | Establishes reference plane for reflection, characterises directivity + reflection tracking |
| Open ($S_{11} = +1$, $\Gamma = +1$) | $Z = \infty$ | Completes the reflection error model (combined with Short and Load) |
| Load ($S_{11} = 0$, $\Gamma = 0$) | $Z = 50\ \Omega$ | Isolates source match and directivity by presenting a matched termination |
| Through (direct connection) | $Z_{line} = 50\ \Omega$ | Characterises transmission tracking and load match for forward and reverse directions |

After applying the SOLT correction, the VNA software mathematically removes all twelve systematic error terms from subsequent measurements. The reference plane is defined at the tips of the calibration standards — not at the connectors of the device under test (DUT). If test fixtures extend beyond that reference plane, additional de-embedding is required.

**Practical note on the Open standard:** A true open at microwave frequencies is not ideal — fringe capacitance at an open connector end introduces a small frequency-dependent phase shift. Calibration kits include a polynomial model of this fringe capacitance ($C_0, C_1, C_2, C_3$), and the VNA software applies the model automatically. Using a calibration kit whose polynomial model does not match the physical standard is a common source of error above 10 GHz.

---

### Q4. What is TRL calibration, and why is it preferred over SOLT for on-wafer and fixture-based measurements?

**Answer:**

TRL (Through-Reflect-Line) calibration is a three-standard scheme that establishes the reference plane and error correction at arbitrary locations along a transmission line structure — including directly at probe tips or on-PCB reference planes where ideal lumped SOLT standards cannot be fabricated.

**The three standards:**

| Standard | Description | Requirement |
|---|---|---|
| Through (T) | Direct connection between the two ports, zero length or known length | Serves as the reference impedance |
| Reflect (R) | Highly reflective termination (short or open) at each port | Does not need to be ideal — only needs $|\Gamma| \approx 1$ |
| Line (L) | A section of transmission line with a length different from the Through | Length difference must produce a phase shift of $20°$–$160°$ at the calibration frequency |

**Why TRL is preferred for on-PCB / on-wafer work:**

1. **Reference plane accuracy:** The calibration plane is defined by the transmission line geometry itself — the same medium as the DUT. There is no mismatch between the calibration standard's characteristic impedance and the fixture.

2. **No ideal standards required:** Fabricating a perfect 50 $\Omega$ resistive load on-PCB is difficult. TRL only requires a transmission line and a reflective standard, both of which are naturally compatible with the PCB or wafer medium.

3. **Connector-independent:** SOLT relies heavily on precise connector models. TRL eliminates the connector from the calibrated reference plane entirely.

**Limitation:** The Line standard must be a different length from the Through. At each frequency, the phase difference between Line and Through must remain between $20°$ and $160°$. For a single Line standard this limits the usable bandwidth to roughly one decade of frequency. For broadband calibration, multiple Line standards of different lengths are used (the LRRM or multi-line TRL variants).

**Common mistake:** Choosing a Line length that produces exactly $90°$ phase shift at the centre frequency but falls below $20°$ or above $160°$ at the band edges, causing calibration degradation. Always verify the phase condition across the full frequency range before accepting the calibration.

---

## Tier 2: Intermediate

### Q5. Explain the VNA 12-term error model. Sketch the signal flow graph and explain how the calibration correction is applied mathematically.

**Answer:**

The two-port VNA 12-term error model describes the difference between the VNA's raw measurements and the actual DUT S-parameters using three error terms per measurement path (forward and reverse), giving twelve terms total.

**Forward path error terms ($S_{11}$ and $S_{21}$ measurement):**

- $E_{DF}$: forward directivity
- $E_{SF}$: forward source match
- $E_{RF}$: forward reflection tracking
- $E_{LF}$: forward load match
- $E_{TF}$: forward transmission tracking
- $E_{XF}$: forward isolation

**Reverse path error terms:** mirror of the above with subscript R.

**The correction equations:**

After calibration determines all twelve $E$ terms, the corrected S-parameters are extracted from raw measurements $S_{11m}$, $S_{21m}$, $S_{12m}$, $S_{22m}$ using:

$$S_{11A} = \frac{\left(\frac{S_{11m} - E_{DF}}{E_{RF}}\right)\left(1 + \frac{S_{22m} - E_{DR}}{E_{RR}} \cdot E_{SR}\right) - E_{LF}\left(\frac{S_{21m} - E_{XF}}{E_{TF}}\right)\left(\frac{S_{12m} - E_{XR}}{E_{TR}}\right)}{\Delta}$$

where $\Delta$ is the common denominator involving all four raw measurements and error terms. Similar expressions exist for $S_{21A}$, $S_{12A}$, $S_{22A}$.

**In practice:** This is handled automatically by VNA firmware. The engineer's role is to:

1. Use the correct calibration kit definition matching the physical standards
2. Perform calibration at the correct reference plane (connector face, probe tip, or PCB launch)
3. Verify calibration quality by measuring a known standard and comparing to its model

**Calibration verification:** After SOLT calibration, measuring the Through standard should give $|S_{21}| = 0$ dB and $|S_{11}| < -40$ dB across the band. Residual errors above these levels indicate poor connections or worn standards.

---

### Q6. What is de-embedding? Describe the two-port network subtraction method and its assumptions.

**Answer:**

De-embedding is the mathematical removal of fixture parasitics (connectors, launch pads, cables, SMA footprints) from a combined measurement, to obtain the S-parameters of the DUT alone.

**The problem setup:**

The raw measurement $[S_{meas}]$ contains the cascade of the input fixture $[S_A]$, the DUT $[S_{DUT}]$, and the output fixture $[S_B]$:

$$[S_{meas}] = [S_A] \star [S_{DUT}] \star [S_B]$$

where $\star$ denotes S-parameter cascade (performed conveniently in T-matrix or ABCD-matrix form).

**T-matrix (wave-transfer matrix) de-embedding:**

Convert all matrices to T-matrices (also called wave-cascade matrices):

$$[T_X] = \frac{1}{S_{21,X}}\begin{bmatrix} -\det(S_X) & S_{11,X} \\ -S_{22,X} & 1 \end{bmatrix}$$

The cascade in T-matrix form is a simple matrix product:

$$[T_{meas}] = [T_A] \cdot [T_{DUT}] \cdot [T_B]$$

Therefore:

$$[T_{DUT}] = [T_A]^{-1} \cdot [T_{meas}] \cdot [T_B]^{-1}$$

Convert the result back to S-parameters.

**Assumptions and limitations:**

1. **Fixture characterisation must be accurate.** $[S_A]$ and $[S_B]$ are typically obtained by measuring dedicated "dummy" structures (open, short, or matched Through on the same PCB layer). Any error in the fixture model propagates directly into $[S_{DUT}]$.

2. **Single-mode propagation.** The T-matrix method assumes only the fundamental mode propagates. Multi-mode propagation (e.g., at via transitions or connectors above their cutoff frequency) violates this assumption.

3. **Linear, time-invariant DUT.** S-parameter measurements assume LTI behaviour. Nonlinear DUTs (amplifiers near compression) require large-signal models instead.

4. **Reference plane accuracy.** The de-embedding removes parasitics to the defined reference plane. If the fixture dummy structures do not accurately reproduce the same geometry as the actual fixture, the result is erroneous.

**Common mistake:** Attempting to de-embed a fixture that is resonant within the measurement band. A fixture with a stub resonance at 5 GHz will have $S_{21,A} \approx 0$ at that frequency. Dividing by near-zero in the T-matrix inversion produces wildly incorrect DUT data at and near that frequency.

---

### Q7. How do you determine the characteristic impedance of a PCB trace from a TDR measurement? What sources of error affect the result?

**Answer:**

**The measurement procedure:**

1. Launch a calibrated 50 $\Omega$ step from the TDR. Record the incident voltage $V_i$ (typically $V_i = V_{source}/2$ when the source is matched to the cable).

2. Observe the reflected voltage $V_r$ at the launch point when the step is on the trace under test.

3. Compute the reflection coefficient:

$$\Gamma = \frac{V_r}{V_i}$$

4. Solve for trace impedance:

$$Z_0 = Z_{ref} \cdot \frac{1 + \Gamma}{1 - \Gamma}$$

where $Z_{ref}$ is the TDR's calibrated reference impedance (50 $\Omega$).

**Example:** If $V_i = 250$ mV and the trace reflects $V_r = +25$ mV, then $\Gamma = 0.1$ and $Z_0 = 50 \times 1.1/0.9 = 61.1\ \Omega$.

**Sources of error:**

| Error source | Effect | Mitigation |
|---|---|---|
| Finite rise time of TDR step | Spatial resolution limitation; inductive launches appear capacitive if structure is shorter than $v_p \cdot t_r/2$ | Use faster TDR; apply deconvolution |
| Launch pad and connector parasitics | First few millimetres of measurement contaminated by transition effects | Use a TDR probe tip; apply SOL correction at probe tip |
| Cable reflections | Multiple reflections create standing waves that add to trace reflection | Use high-quality microwave cable; use a short cable |
| Skin effect / dielectric loss | Impedance appears to vary with position along a lossy line (skin effect reduces apparent $Z_0$ at late times) | Measure impedance at the leading edge of the step response, not at the plateau |
| Probe loading | High-impedance probes add capacitance at the launch point; active probes reduce this | Characterise probe loading separately |
| Reference plane uncertainty | If TDR is calibrated to cable end but probe tip is not, absolute impedance is offset | Calibrate directly at the probe tip using a known 50 $\Omega$ reference |

**Key practical point:** Always read the impedance from the first step of the reflected waveform, before multiple reflections and line losses accumulate. A trained eye looks at the leading edge of the plateau for each section.

---

### Q8. A TDR waveform shows the incident voltage is 200 mV. After a trace section, the waveform settles at 220 mV. After a via, it drops to 205 mV. Calculate the impedance of each section.

**Answer:**

**Reference impedance:** $Z_{ref} = 50\ \Omega$. Incident voltage $V_i = 200$ mV.

**Section 1 (trace, settling at 220 mV):**

The total voltage at the launch point during steady-state on a lossless line terminated in impedance $Z_1$ is:

The TDR source is a voltage step $V_s$ behind a series resistor $R_s = Z_{ref}$. The incident wave is $V_i = V_s/2$. The voltage displayed at the launch when looking into impedance $Z_1$ is:

$$V_{display} = V_i \cdot (1 + \Gamma) = V_i \cdot \frac{2Z_1}{Z_{ref} + Z_1}$$

Solving for $Z_1$:

$$\Gamma = \frac{V_{display} - V_i}{V_i} = \frac{220 - 200}{200} = 0.1$$

$$Z_1 = Z_{ref} \cdot \frac{1 + \Gamma}{1 - \Gamma} = 50 \times \frac{1.1}{0.9} = 61.1\ \Omega$$

**The trace section is 61.1 $\Omega$** — higher than the 50 $\Omega$ reference, indicating a slightly narrow trace or elevated height above the reference plane.

**Section 2 (after the via, settling at 205 mV):**

Now the displayed voltage is 205 mV. However, this voltage is referenced to the new "incoming" wave at the via, which is no longer 200 mV — it is the wave that has been transmitted through the first section. For a qualitative assessment in the standard TDR display convention (all voltages referenced to the original $V_i$), the via impedance is read from the change in displayed voltage:

$$\Gamma_{via} = \frac{V_{after} - V_i}{V_i} = \frac{205 - 200}{200} = 0.025$$

$$Z_{via} = 50 \times \frac{1.025}{0.975} = 52.6\ \Omega$$

**Note:** This is an approximation because the TDR display in the second section is affected by the first section's mismatch. For precise multi-section analysis, convert the TDR to frequency-domain S-parameters and use T-matrix de-embedding. The simple calculation above gives the correct answer only when each section's $\Gamma$ is small (which is the case here — both sections are close to 50 $\Omega$).

**Physical interpretation:** The trace at 61 $\Omega$ is over-impedance (too narrow or too far from the reference plane). The via at 52.6 $\Omega$ is approximately matched. In a real design, the 61 $\Omega$ section would need a wider trace or adjusted dielectric height to target 50 $\Omega$.

---

## Tier 3: Advanced

### Q9. Describe the multi-line TRL (ML-TRL) calibration technique. Why does it outperform single-line TRL for broadband measurements?

**Answer:**

**The bandwidth limitation of single-line TRL:**

A single Line standard of electrical length $\ell$ (relative to the Through) is valid only when its phase shift satisfies $20° \le \beta\ell \le 160°$. This constrains the usable bandwidth to:

$$f_{max}/f_{min} = 160°/20° = 8:1$$

A single Line standard from 1 GHz to 8 GHz, for example, requires $\ell$ to give $90°$ phase at $\sqrt{1 \times 8} \approx 2.83$ GHz. At 1 GHz the phase is $\approx 32°$ (just inside the 20° limit) and at 8 GHz it is $\approx 256°$ (far outside). So the practical bandwidth is closer to a 4:1 ratio in favourable geometries.

**Multi-line TRL:**

ML-TRL uses several Line standards of different lengths. At each frequency, the algorithm selects the Line standard whose phase difference with the Through falls within the $20°$–$160°$ window. With $N$ lines of geometrically spaced lengths, the calibrated bandwidth is:

$$\frac{f_{max}}{f_{min}} \approx 8^{N-1}$$

Practically, three Line standards (plus one Through and one Reflect) can cover 50 MHz to 26 GHz.

**The Marks algorithm (IEEE 1996):**

The Marks multi-line TRL algorithm performs a least-squares fit across all valid Line standards simultaneously at each frequency, rather than switching between them. This suppresses random measurement noise by up to $\sqrt{N}$ compared to using a single Line. The algorithm also self-consistently determines the characteristic impedance of the Line standards from their propagation constants, providing a fully traceable impedance reference — no external impedance standard (like a 50 $\Omega$ load) is required.

**Advantages over SOLT for on-PCB work:**

1. **Traceable characteristic impedance:** The calibrated impedance is defined by the PCB transmission line's $Z_0$, not by an external coaxial load whose model may not match the board medium.

2. **No lumped standards required:** All standards are planar transmission-line sections that can be fabricated on the same PCB panel as the DUT.

3. **Superior accuracy above 20 GHz:** SOLT connector models become inaccurate above 20–26 GHz. ML-TRL accuracy improves because more Line standards are available near the measurement frequency.

**Limitation:** ML-TRL requires that all Line standards be fabricated from the same transmission-line medium as the DUT (same PCB layer, same trace width, same dielectric). Any process variation between the calibration structures and the DUT location introduces systematic errors that cannot be calibrated out.

---

### Q10. A 25 Gbps differential channel is characterised on a VNA. Describe the complete measurement, calibration, and de-embedding workflow from raw S4P to channel-ready S4P, including mixed-mode conversion. What parameters are most diagnostic of channel health?

**Answer:**

**Step 1 — Physical measurement setup:**

Use a 4-port VNA (or two-port VNA with a 4-port switch box) with phase-stable cables. Set the frequency range from 100 MHz to at least $2.5 \times 25$ GHz = 62.5 GHz (to the 5th harmonic). Set frequency points to at least 1601 (finer near the Nyquist frequency of 12.5 GHz is useful). Set IF bandwidth to 100–1000 Hz for low noise floor (−80 dB or better).

**Step 2 — Calibration at the SMA connector reference plane:**

Perform 4-port SOLT calibration using a precision calibration kit (e.g., Maury or Rosenberger 3.5 mm kit). This establishes the calibrated reference plane at the connector tips. Store the raw 4-port S-parameters as $[S_{4P,raw}]$.

**Step 3 — Fixture de-embedding:**

The fixture (SMA connectors, microvia transitions, launch pads) is characterised by measuring dedicated Open-Short-Match (or Through) dummy structures on the same board. Compute fixture T-matrices $[T_{fixture,port1..4}]$. Apply de-embedding:

$$[T_{DUT}] = [T_{fixture,in}]^{-1} \cdot [T_{meas}] \cdot [T_{fixture,out}]^{-1}$$

Convert back to S-parameters to obtain $[S_{4P,channel}]$.

**Step 4 — Mixed-mode S-parameter conversion:**

A differential channel uses ports as pairs: $(P1, P2)$ = differential pair at transmitter side, $(P3, P4)$ = at receiver side. The 4-port single-ended $S_{4P}$ is converted to mixed-mode $S_{MM}$ using the transformation matrix:

$$[M] = \frac{1}{\sqrt{2}}\begin{bmatrix} 1 & -1 & 0 & 0 \\ 1 & 1 & 0 & 0 \\ 0 & 0 & 1 & -1 \\ 0 & 0 & 1 & 1 \end{bmatrix}$$

$$[S_{MM}] = [M][S_{4P}][M]^{-1}$$

The result has the standard mixed-mode indexing with mode subscripts D (differential) and C (common):

$$[S_{MM}] = \begin{bmatrix} S_{DD11} & S_{DC11} & S_{DD21} & S_{DC21} \\ S_{CD11} & S_{CC11} & S_{CD21} & S_{CC21} \\ S_{DD12} & S_{DC12} & S_{DD22} & S_{DC22} \\ S_{CD12} & S_{CC12} & S_{CD22} & S_{CC22} \end{bmatrix}$$

**Step 5 — Key diagnostic parameters:**

| Parameter | What it measures | Target (25 Gbps channel) |
|---|---|---|
| $|S_{DD21}|$ | Differential insertion loss | $< -5$ dB at Nyquist (12.5 GHz); within spec mask |
| $|S_{DD11}|$ | Differential return loss | $< -12$ dB up to Nyquist |
| $|S_{CC21}|$ | Common-mode insertion loss | $< -20$ dB (high rejection means EMI is suppressed) |
| $|S_{CD21}|, |S_{DC21}|$ | Mode conversion (differential-to-common, common-to-differential) | $< -30$ dB; indicates pair asymmetry |
| $\angle S_{DD21}$ (group delay) | Differential phase linearity | Variation $< \pm 50$ ps across Nyquist band |
| NEXT, FEXT via single-ended terms | Aggressor coupling | Dependent on channel spacing; verify against standard limit |

**Step 6 — Derived channel metrics:**

Convert $S_{DD21}$ to the time domain using inverse FFT (with appropriate windowing — Hann window balances ripple and resolution) to obtain the differential impulse response. Convolve with a PRBS pattern and verify eye opening.

**Most diagnostic parameter:** $|S_{DD21}|$ at Nyquist is the single most important number for insertion loss budget. Mode conversion $|S_{CD21}|$ is the most diagnostic for layout quality — excess mode conversion indicates an asymmetric differential pair (unequal lengths, unequal widths, or asymmetric via stubs) and predicts EMC and jitter problems.

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| VNA | Vector Network Analyser — measures complex S-parameters over frequency |
| TDR | Time Domain Reflectometer — measures reflected step to characterise impedance vs. distance |
| S-parameter | Scattering parameter relating incident and reflected power waves at network ports |
| SOLT | Short-Open-Load-Through coaxial VNA calibration scheme (12 error terms) |
| TRL | Through-Reflect-Line calibration; reference plane defined by transmission-line medium |
| ML-TRL | Multi-line TRL — broadband calibration using multiple Line standards |
| De-embedding | Mathematical removal of fixture effects from a combined measurement |
| T-matrix | Wave-cascade matrix; enables cascade de-embedding via matrix multiplication |
| Mixed-mode S-parameters | S-parameter set distinguishing differential ($D$) and common ($C$) stimulus/response modes |
| $S_{CD21}$ | Common-to-differential mode conversion; indicates pair asymmetry |
| Reference plane | The physical location at which calibration correction is applied |
| Fringe capacitance | Parasitic capacitance at an open-connector end; must be modelled in SOLT calibration |
| Group delay | $-d\angle S_{21}/d\omega$; frequency derivative of transmission phase; indicates dispersion |
