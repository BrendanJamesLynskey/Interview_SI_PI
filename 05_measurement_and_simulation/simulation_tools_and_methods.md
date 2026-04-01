# Simulation Tools and Methods

## Overview

SI/PI simulation spans a wide range of tool types, each appropriate for different physical regimes and questions. Field solvers extract electromagnetic parameters (impedance, capacitance, inductance) that are then used in circuit simulators. Behavioural models such as IBIS and IBIS-AMI bridge the gap between IC vendor characterisation data and system-level simulation. Statistical methods extend single-simulation results to probabilistic eye predictions required for high-speed interface compliance. Understanding when to use each tool — and what its limitations are — is as important as knowing how to operate it.

---

## Tier 1: Fundamentals

### Q1. What is the difference between a 2D field solver and a 3D field solver? When is each appropriate?

**Answer:**

**2D field solvers:**

A 2D field solver solves Maxwell's equations in a cross-sectional plane perpendicular to the direction of wave propagation. It assumes the structure is infinitely long and uniform along the propagation direction. The output is the per-unit-length parameters of the transmission line:

- Resistance per unit length $R'$ ($\Omega$/m)
- Inductance per unit length $L'$ (H/m)
- Capacitance per unit length $C'$ (F/m)
- Conductance per unit length $G'$ (S/m)

These are assembled into the characteristic impedance and propagation constant:

$$Z_0 = \sqrt{\frac{R' + j\omega L'}{G' + j\omega C'}}, \qquad \gamma = \sqrt{(R' + j\omega L')(G' + j\omega C')}$$

**When 2D is appropriate:** Straight, uniform PCB traces away from discontinuities. Stackup impedance calculations. Extraction of microstrip, stripline, and coplanar waveguide parameters. These form the backbone of pre-layout impedance estimation and are accurate to within a few percent of the true 3D result for trace lengths much longer than the trace width.

Examples: Polar SI9000, Ansys 2D Extractor, Cadence Sigrity 2D extraction.

**3D field solvers:**

A 3D field solver solves Maxwell's equations in the full three-dimensional volume of the structure. It accurately captures discontinuities where the 2D assumption breaks down: vias, bends, pads, connectors, differential pair breakouts, and reference plane transitions.

The two dominant 3D approaches are:

| Method | Full name | Key property |
|---|---|---|
| FEM | Finite Element Method | Unstructured mesh; excellent for curved/complex geometry; memory-intensive |
| FDTD | Finite Difference Time Domain | Regular grid; efficient for broadband transient analysis; staircase error for curved surfaces |
| MoM | Method of Moments | Solves only surface currents; efficient for planar structures but limited to homogeneous dielectrics |

**When 3D is required:** Via transitions, connector interfaces, PCB launches (SMA footprints), package-to-board interconnects, differential pair routing through connector pins, and any structure where current paths are not parallel to the PCB surface.

**Rule of thumb:** If the feature size is comparable to or smaller than the trace width, use 3D. If the feature is a long straight trace, 2D is sufficient and computationally much faster.

---

### Q2. What is an IBIS model? What information does it contain and what does it not contain?

**Answer:**

IBIS (I/O Buffer Information Specification) is a behavioural model format standardised by the IBIS Open Forum. It describes the electrical behaviour of an IC's I/O buffers using voltage-current (V-I) and voltage-time (V-t) tables, rather than the transistor-level netlist (which vendors keep confidential).

**What an IBIS model contains:**

1. **V-I tables:** Pull-up and pull-down characteristics for the output driver under different conditions (power, ground, die-to-package connection). These are measured curves (not equations) at a set of voltage and temperature corners.

2. **V-t tables (waveforms):** Rising and falling edge waveforms from which the driver's drive strength, rise time, and slew rate are derived.

3. **Package model:** Series R-L-C parasitic representation of the lead frame or flip-chip bumps from the die pad to the package pin. May be a simple lumped model or a full S-parameter file.

4. **Clamp tables:** ESD protection diode characteristics (power-clamp and ground-clamp) — important for overshoot and undershoot analysis.

5. **Receiver input characteristics:** Differential input voltage thresholds and input impedance curves.

**What an IBIS model does not contain:**

- Transistor-level netlist or process parameters (proprietary information protected)
- Noise or EMI behaviour not captured by the V-I/V-t tables
- Non-quasi-static effects (e.g., output impedance dependence on pattern history — this requires IBIS-AMI)
- Accurate common-mode behaviour for differential drivers (some IBIS models include diff-mode, but common-mode path is often missing)
- Internal supply and ground noise coupling (the Cp/Ls package model is simplified)

**Practical use:** IBIS models are the standard deliverable from IC vendors for system-level SI simulation. They are used in tools such as Ansys HSPICE, Cadence Sigrity, Mentor HyperLynx, and Zuken SI Explorer to simulate driver-channel-receiver interactions. The models are accurate to within 5–15% of transistor-level simulation for typical signalling scenarios.

---

### Q3. What is SPICE, and what are its limitations for modern high-speed SI simulation?

**Answer:**

SPICE (Simulation Program with Integrated Circuit Emphasis) is the foundational circuit simulator, originally developed at UC Berkeley in 1973. It solves the differential-algebraic system of equations describing a circuit's nodal voltages and branch currents in the time domain (transient analysis) or in the frequency domain (AC analysis).

**Core capabilities:**

- Transient analysis: step response, eye diagram simulation, power supply transient
- AC analysis: small-signal frequency response, impedance vs. frequency
- DC operating point: bias point for nonlinear devices

**Modern variants:** HSPICE (Synopsys), Spectre (Cadence), ELDO (Siemens EDA), ngspice (open source).

**Limitations for high-speed SI:**

1. **Transmission line modelling:** SPICE's built-in lossless transmission line (T element) is accurate only for lossless, frequency-independent structures. For accurate lossy line simulation (skin effect, dielectric loss), a frequency-dependent W-element or Touchstone S-parameter block must be used. Setting up W-elements correctly requires external extraction and is error-prone.

2. **Simulation time for long patterns:** A 10 Gbps PRBS-31 pattern has $2^{31} - 1 \approx 2 \times 10^9$ bits. Simulating the full pattern in SPICE is computationally infeasible — statistical methods or bathtub curve extrapolation must substitute.

3. **Package and board parasitic capture:** High-pin-count packages with thousands of parasitics overload SPICE's matrix solver. Reduced-order models or S-parameter imports are needed.

4. **Equalization and DSP in the channel:** TX pre-emphasis, RX CTLE, and DFE are DSP algorithms. Implementing these correctly in SPICE requires behavioural models or Verilog-A, not straightforward device elements.

5. **Statistical/stochastic jitter:** SPICE produces a single deterministic waveform per simulation run. Extracting jitter statistics (random jitter RJ, deterministic jitter DJ) requires many thousands of simulation runs or special post-processing.

These limitations are addressed by IBIS-AMI models running in an AMI simulator engine.

---

## Tier 2: Intermediate

### Q4. What is IBIS-AMI and how does it extend standard IBIS for statistical eye analysis?

**Answer:**

IBIS-AMI (Algorithmic Modelling Interface) extends the standard IBIS format by adding a DLL (or shared library) interface that implements the transmitter and receiver signal processing algorithms in executable code rather than lookup tables. This enables accurate simulation of:

- TX pre-emphasis / de-emphasis (FIR filter at the transmitter)
- RX CTLE (Continuous Time Linear Equalisation — a high-pass analog filter)
- RX DFE (Decision Feedback Equalisation — subtracts ISI from previous decisions)
- CDR (Clock and Data Recovery — tracks phase of incoming data)
- Adaptation algorithms (e.g., LMS adaptation of DFE tap weights)

**The AMI simulation flow:**

AMI uses the concept of an impulse response to represent the channel. The full system impulse response $h(t)$ is computed from:

$$h(t) = h_{TX_{analog}}(t) \ast h_{channel}(t) \ast h_{RX_{analog}}(t)$$

where each term comes from IBIS, S-parameter data, or package models. The AMI algorithm processes this convolved impulse response in two phases:

1. **Init phase:** The Tx and Rx AMI DLLs receive the channel impulse response. The DLLs return modified impulse responses reflecting the effect of equalization. Adaptation algorithms run to convergence during Init, operating on the impulse response rather than bit-by-bit data — enormously faster than time-domain simulation.

2. **GetWave phase:** The DLLs operate sample-by-sample on time-domain waveform data. This captures the DFE's data-dependent behaviour (since DFE decisions depend on symbol history). GetWave is slower but necessary for accurate DFE simulation.

**Statistical eye analysis:**

Using the equalised impulse response from the Init phase, the statistical eye is constructed by superimposing the responses to all possible bit patterns in a PRBS sequence. For $N$-tap ISI, there are $2^N$ possible patterns; each contributes a voltage trajectory to the eye diagram. The eye is built by summing the probability distributions:

$$P_{eye}(V, t) = \sum_{patterns} P_{pattern} \cdot \delta(V - V_{pattern}(t))$$

This produces a probability density function (PDF) of voltage at each sampling time. The BER contour at a given BER level (e.g., $10^{-12}$) is the eye opening boundary at that probability level. This statistical approach generates a bathtub curve without simulating $10^{12}$ bits explicitly.

**Why this matters for compliance:** IEEE and OIF standards (PCIe, CEI, USB4) specify mask compliance using statistical eyes at $10^{-12}$ BER. No deterministic simulation can achieve this — AMI statistical methods are mandatory.

---

### Q5. A colleague runs a frequency-domain field solver on a PCB microstrip and extracts $Z_0$, $\alpha$, and $v_p$ as a function of frequency. Describe how these are used in a channel simulation and what each parameter tells you about channel quality.

**Answer:**

The three extracted parameters characterise the channel's linear, time-invariant (LTI) response:

**Characteristic impedance $Z_0(f)$:**

$$Z_0(f) = \sqrt{\frac{R'(f) + j\omega L'(f)}{G'(f) + j\omega C'(f)}}$$

At low frequencies, $R'(f)$ (DC resistance) dominates and $Z_0$ is high and reactive. At microwave frequencies, $Z_0$ approaches its geometric value ($\approx 50\ \Omega$ for a matched trace). Frequency dependence of $Z_0$ causes frequency-dependent reflections in a system designed for 50 $\Omega$ — even a well-designed trace is not exactly 50 $\Omega$ across all frequencies.

**Attenuation constant $\alpha(f)$ (Np/m or dB/m):**

$$\alpha(f) = \text{Re}[\gamma(f)] = \text{Re}\left[\sqrt{(R' + j\omega L')(G' + j\omega C')}\right]$$

Two physically distinct contributions:

- **Conductor loss** $\alpha_c$: increases as $\sqrt{f}$ due to skin effect constricting current to the surface
- **Dielectric loss** $\alpha_d$: increases linearly with $f$ due to polarisation losses in the dielectric ($\alpha_d \propto f \cdot \tan\delta$)

At high data rates (> 10 Gbps), dielectric loss dominates on long traces. The insertion loss at frequency $f$ for a trace of length $\ell$ is:

$$|S_{21}(f)| = e^{-\alpha(f) \cdot \ell}\ \text{(linear)}\ \equiv -8.686 \cdot \alpha(f) \cdot \ell\ \text{(dB)}$$

**Phase velocity $v_p(f)$:**

$$v_p(f) = \frac{\omega}{\text{Im}[\gamma(f)]} = \frac{1}{\sqrt{L'(f) C'(f)}}$$

For a lossless line, $v_p = c/\sqrt{\varepsilon_{eff}}$ and is frequency-independent. For a lossy line, $v_p$ varies with frequency — this is dispersion. Dispersion causes different frequency components of the signal to arrive at different times, spreading the pulse and closing the eye.

**Using these parameters in channel simulation:**

The extracted RLGC or S-parameter data is imported into the channel simulator as a Touchstone (snp) file. The simulator convolves the channel's transfer function with the driver waveform to produce the received waveform. Key quality indicators:

| Parameter behaviour | Interpretation |
|---|---|
| $\alpha(f)$ rises steeply above 5 GHz | High-loss dielectric; consider Megtron6 or better material |
| $v_p$ strongly frequency-dependent | High dispersion; equalization required |
| $Z_0(f)$ deviates from 50 $\Omega$ by > 5 $\Omega$ | Impedance discontinuities from variable width or adjacent copper |
| $\alpha_c \gg \alpha_d$ at Nyquist | Thin trace or rough surface finish; consider smoother copper |

---

### Q6. What is a Touchstone file (.s2p, .s4p) and how is it used in channel simulation?

**Answer:**

A Touchstone file is an ASCII text format for storing frequency-domain N-port S-parameter data. It is the standard interchange format between field solvers, VNA measurements, and circuit simulators.

**File format structure:**

```
! Comment lines begin with !
# GHz S MA R 50        ! Option line: frequency unit, parameter type, format, reference impedance
! Port ordering: Port 1 = +trace, Port 2 = -trace
1.0    0.03 -12.1    0.89 -5.2    0.89 -5.2    0.03 -12.1   ! freq S11 S21 S12 S22
2.0    0.03 -24.2    0.81 -10.4   0.81 -10.4   0.03 -24.2
...
```

**Format options:**

- **MA:** Magnitude-Angle (most common for S-parameters from VNA)
- **RI:** Real-Imaginary (preferred for numerical processing)
- **DB:** dB-Angle

**How it is used in channel simulation:**

1. **Direct import:** HSPICE, SPECTRE, and ADS accept .s2p files as a two-port element in the netlist. The simulator interpolates between frequency points using rational function fitting (e.g., Vector Fitting) to create a time-domain convolution kernel.

2. **Channel impulse response generation:** The S-parameter data is converted to a causal, passive impulse response $h(t)$ via inverse FFT with appropriate windowing (Hann or Blackman-Harris to suppress ringing from truncated frequency data).

3. **Multi-port channels:** A .s4p file represents a differential channel (4 single-ended ports). The simulator reads all 16 S-parameter entries and may apply mixed-mode conversion internally.

**Passivity and causality enforcement:**

Raw field solver or VNA data often violates passivity ($|S_{21}| > 1$ at some frequencies due to numerical error or noise) or causality (non-zero response before $t=0$ due to insufficient frequency span). Simulators such as Ansys SIwave or Cadence Sigrity apply passivity enforcement algorithms before using the data. Always verify that the S-parameter data is passive and causal before running a channel simulation — a non-passive model will cause the simulator to produce divergent, non-physical results.

**Common mistake:** Using .s2p data with too coarse a frequency resolution for long interconnects. A 30 cm PCB trace has a round-trip delay of approximately 3.5 ns. The frequency resolution must be finer than $1/3.5\ \text{ns} \approx 286\ \text{MHz}$ to capture the trace's resonant structure. Data with 1 GHz resolution will miss all resonances below 1 GHz, producing an incorrect channel impulse response.

---

## Tier 3: Advanced

### Q7. Describe the Vector Fitting algorithm. Why is it used to convert S-parameter data to a SPICE-compatible model, and what can go wrong?

**Answer:**

**The problem Vector Fitting solves:**

SPICE simulators operate in the time domain; they cannot directly use S-parameter data, which is defined only at discrete frequency points. A rational function representation $H(s)$ provides a continuous, analytic transfer function that SPICE can evaluate at any time step:

$$H(s) = d + \sum_{k=1}^{N} \frac{c_k}{s - a_k}$$

where $a_k$ are poles (real or complex-conjugate pairs), $c_k$ are residues, and $d$ is the direct term. This is equivalent to a network of R, L, C, and controlled sources that SPICE can simulate.

**The Vector Fitting procedure (Gustavsen 1999):**

1. **Initial pole selection:** Choose a set of starting poles $\{a_k^{(0)}\}$ distributed across the frequency axis (logarithmically or uniformly). Complex poles come in conjugate pairs.

2. **Linearised least-squares fit:** Rewrite $H(s)$ as:

$$\sigma(s) \cdot H(s) = \tilde{H}(s)$$

where $\sigma(s) = 1 + \sum_k \tilde{c}_k/(s-a_k)$ is an auxiliary function. At the measured frequency points, this is now linear in the unknowns $\{c_k, \tilde{c}_k, d\}$. Solve the overdetermined linear system by least-squares (QR decomposition).

3. **Pole relocation:** The zeros of $\sigma(s)$ are the improved pole locations. Compute them and update $\{a_k\}$.

4. **Iterate:** Repeat Steps 2–3 until pole locations converge (typically 3–10 iterations).

5. **Residue extraction:** With final poles fixed, solve for residues $\{c_k\}$ in one final least-squares step.

6. **Passivity enforcement:** Check that the resulting rational model satisfies $\text{Re}[Y(j\omega)] \geq 0$ (positive real condition for passivity). If violated, apply residue perturbation or the Hamiltonian matrix method to enforce passivity.

7. **SPICE synthesis:** Convert poles and residues to equivalent R-L-C networks using Foster or Cauer network synthesis.

**What can go wrong:**

| Problem | Symptom | Cause |
|---|---|---|
| Too few poles | Fit residual $>$ 0.5 dB at some frequencies | Band too wide, or resonances too sharp for pole count |
| Too many poles | Simulation extremely slow; spurious resonances in unused frequency range | Overfitting; poles are fitting noise rather than signal |
| Unstable poles ($\text{Re}[a_k] > 0$) | Time-domain simulation diverges | Numerical issue in pole relocation step; poles must be moved to left half-plane |
| Non-passive model | Simulator energy grows over time; divergent waveform | Passivity enforcement was skipped or failed |
| Causality violation | Ringing before the stimulus arrives | Frequency data does not extend to DC or to high enough frequency; pre-ringing in the inverse FFT |

**Practical guideline:** For PCIe Gen 5 (32 Gbps) channels, a Vector Fitting model with 30–80 pole pairs per S-parameter entry typically achieves < 0.1 dB fit error across 0.1–50 GHz.

---

### Q8. Explain statistical eye analysis. Describe the relationship between the probability density function (PDF) of eye voltage, the bathtub curve, and the bit error ratio (BER) at $10^{-12}$.

**Answer:**

**From deterministic simulation to statistics:**

A time-domain simulation of a bit sequence produces a set of voltage samples at the sampling instant. If the bit sequence is a PRBS, the samples form a histogram. As the number of simulated bits increases, the histogram approaches the probability density function (PDF) of the eye voltage:

$$f_V(v) = \lim_{N \to \infty} \frac{\text{Number of samples in } [v, v+dv]}{N \cdot dv}$$

In a well-opened eye, $f_V(v)$ has two peaks: one near the logic-1 voltage $V_H$ and one near the logic-0 voltage $V_L$.

**The bathtub curve:**

The BER at a threshold voltage $V_{th}$ is the probability that a 1-bit sample falls below $V_{th}$ (an error on a 1) plus the probability that a 0-bit sample rises above $V_{th}$ (an error on a 0), weighted by symbol probabilities:

$$\text{BER}(V_{th}) = P_1 \int_{-\infty}^{V_{th}} f_{V|1}(v)\,dv + P_0 \int_{V_{th}}^{\infty} f_{V|0}(v)\,dv$$

For equal probability symbols ($P_0 = P_1 = 0.5$):

$$\text{BER}(V_{th}) = \frac{1}{2}\left[\int_{-\infty}^{V_{th}} f_{V|1}(v)\,dv + \int_{V_{th}}^{\infty} f_{V|0}(v)\,dv\right]$$

Plotting $\text{BER}(V_{th})$ versus $V_{th}$ gives the bathtub curve. The curve is U-shaped: high BER at extreme thresholds, minimum BER at the optimal sampling threshold.

**Extrapolation to $10^{-12}$ BER:**

Directly simulating $10^{12}$ bits is impossible (at 32 Gbps, $10^{12}$ bits = 31.25 seconds of data, and each simulated bit takes microseconds). Instead:

1. Simulate a smaller but representative PRBS pattern (e.g., $2^{15}-1$ bits = 32,767 bits).

2. Separate the voltage distribution into deterministic ISI contribution and random noise contribution.

3. Model the random noise as Gaussian with standard deviation $\sigma$ (characterised from the tails of the histogram or from jitter noise measurements).

4. Extrapolate the bathtub curve by convolving the deterministic ISI distribution with the Gaussian:

$$\text{BER}(V_{th}) = \frac{1}{2}\,\text{erfc}\left(\frac{V_{th} - \mu}{\sigma\sqrt{2}}\right)$$

where $\text{erfc}$ is the complementary error function. The $10^{-12}$ eye opening is where this function equals $10^{-12}$, corresponding to approximately $7.03\sigma$ from the mean.

**The Dual-Dirac model:**

For jitter decomposition, the total jitter is modelled as the convolution of a Dirac-delta function (representing deterministic jitter DJ) at $\pm \Delta/2$ and a Gaussian (representing random jitter RJ with standard deviation $\sigma_{RJ}$). The total jitter at $10^{-12}$ BER is:

$$TJ_{10^{-12}} = DJ + 2 \times 7.03\sigma_{RJ} = DJ + 14.07\sigma_{RJ}$$

This is the standard formula for high-speed SerDes jitter budget calculations (used in PCIe, CEI-56G, and similar standards).

**Key insight for interviews:** The $10^{-12}$ eye opening cannot be measured directly — it must be extrapolated. The extrapolation assumes the noise distribution has a Gaussian tail. If the actual distribution has heavier tails (non-Gaussian noise), the real BER at $10^{-12}$ will be worse than predicted. This is why compliance standards mandate characterisation of random and bounded uncorrelated jitter separately.

---

### Q9. Compare Ansys HFSS, Ansys SIwave, and Cadence Sigrity for PCB-level SI/PI simulation. When would you choose each?

**Answer:**

These three tools address different granularity levels within the PCB simulation workflow:

**Ansys HFSS (High Frequency Structure Simulator):**

- Method: 3D FEM
- Best for: Individual 3D structures — connectors, via arrays, BGA ball arrays, chip packages, coaxial transitions
- Accuracy: Highest available for 3D structures; captures all coupling, radiation, and mode conversion
- Limitation: Computationally expensive; not practical for full-board simulation; requires hours to days for complex structures
- Typical use: Extracting a connector or via transition S-parameter for import into a board-level simulator; characterising a critical antenna structure

**Ansys SIwave:**

- Method: 2.5D full-wave solver (specialised for planar multilayer PCB geometries)
- Best for: Full-board PDN analysis, plane resonances, power/ground plane impedance, broad-pattern crosstalk, differential pair insertion loss at the board level
- Accuracy: Good for planar structures; captures plane resonances, via coupling in plane stacks, and broadside coupling between adjacent layers
- Limitation: Less accurate than HFSS for 3D structures (connectors, packages); the 2.5D assumption fails at very high frequencies where radiation from PCB edges becomes significant
- Typical use: Simulate the PDN impedance from the VRM to the BGA balls; assess plane resonances; export S-parameters for a complete channel from a 20-layer board

**Cadence Sigrity (PowerSI, SystemSI, SpeedXP):**

- Method: Mixed — 2.5D full-wave (similar to SIwave) plus SPICE circuit integration
- Best for: System-level simulation integrating board, package, IC models; co-simulation with IBIS-AMI
- Accuracy: Similar to SIwave for the board; advantage is tight integration with Cadence layout tools and IBIS-AMI flow
- Typical use: End-to-end channel simulation (transmitter → package → PCB trace → connector → PCB → package → receiver) with equalization; most commonly used in DDR and SerDes bring-up workflows at companies using Cadence EDA

**Comparison table:**

| Dimension | HFSS | SIwave | Cadence Sigrity |
|---|---|---|---|
| Geometry scope | Single structure | Full board (planar) | Full board + package + IC |
| Physical fidelity | Highest (full 3D) | Good (2.5D) | Good (2.5D) |
| Runtime (typical) | Hours–days | Minutes–hours | Minutes–hours |
| IBIS-AMI integration | Via HFSS-SIwave link | Partial | Full (SystemSI) |
| PDN analysis | Not primary use | Excellent | Good |
| Industry adoption | HFSS used by connector/package vendors | Common in board-level SI teams | Common at Cadence-centric design houses |

**Workflow in practice:** A production SI workflow typically uses all three levels: HFSS for via and connector models, SIwave or Sigrity for board-level S-parameter extraction, and an AMI-capable SPICE engine for full-channel eye simulation. The S-parameter files are imported at each level — no single tool covers the full workflow.

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| 2D field solver | Solves Maxwell's equations in cross-section; yields RLGC per-unit-length parameters |
| 3D field solver | Full 3D EM solution; required for discontinuities (vias, connectors) |
| FEM | Finite Element Method — unstructured meshing; used in HFSS |
| FDTD | Finite Difference Time Domain — regular grid; used in CST Studio |
| IBIS | I/O Buffer Information Specification — behavioural I/O model using V-I/V-t tables |
| IBIS-AMI | IBIS Algorithmic Modelling Interface — adds DLL-based equalization model |
| Init phase | AMI phase that runs equalization adaptation on the channel impulse response |
| GetWave phase | AMI phase that runs DFE sample-by-sample on time-domain waveform |
| Touchstone file | .snp ASCII format for N-port S-parameter data |
| Vector Fitting | Rational function fitting algorithm that converts S-parameter data to poles/residues |
| Passivity | Network property: cannot generate energy; $\text{Re}[Z(j\omega)] \geq 0$ for all $\omega$ |
| Bathtub curve | BER vs. sampling threshold; shows horizontal (voltage) eye opening at target BER |
| Dual-Dirac model | Jitter decomposition: Gaussian (RJ) convolved with two Dirac deltas (DJ) |
| $TJ_{10^{-12}}$ | Total jitter at $10^{-12}$ BER: $DJ + 14.07\sigma_{RJ}$ |
| Statistical eye | Eye diagram built from PDF of all bit pattern responses; enables BER extrapolation |
| SIwave | Ansys 2.5D full-wave PCB solver for board-level SI/PI |
| HFSS | Ansys 3D FEM solver for high-accuracy single-structure simulation |
