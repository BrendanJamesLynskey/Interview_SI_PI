# Quiz: Measurement and Simulation

15 multiple-choice questions covering VNA, TDR, oscilloscope probing, simulation methods, and simulation-to-measurement correlation. Questions span three difficulty tiers. Answers with explanations are collected at the end.

---

## Instructions

Select the single best answer for each question. After completing all questions, check your answers against the answer key. For each incorrect answer, read the full explanation before moving on.

Suggested time: 25 minutes.

---

## Questions

### Fundamentals (Q1 -- Q5)

**Q1.** A Vector Network Analyser (VNA) measures:

- A) Time-domain voltage waveforms at the device under test ports
- B) The complex (magnitude and phase) S-parameters of a device by sweeping frequency and measuring the ratio of reflected and transmitted waves to the incident wave
- C) The impedance of a component by applying a DC current and measuring the resulting voltage
- D) The noise figure of an amplifier by comparing output noise to a known noise source

---

**Q2.** A Time Domain Reflectometer (TDR) sends a fast-rise-time step into a transmission line and observes reflections. A reflected step in the positive direction (same polarity as the incident) indicates:

- A) A short circuit or capacitive discontinuity
- B) An inductive discontinuity or impedance higher than Z0
- C) A matched termination
- D) An inductive discontinuity or impedance lower than Z0

---

**Q3.** Before making measurements with a VNA, the instrument must be calibrated. What does a SOLT (Short-Open-Load-Thru) calibration correct for?

- A) Thermal drift inside the VNA hardware
- B) The systematic errors in the VNA's directional couplers, test cables, connectors, and adapters, referred to as the error model (12-term for 2-port)
- C) Random noise in the VNA measurement
- D) Impedance mismatches between the DUT and the 50 Ohm reference impedance

---

**Q4.** When probing a high-speed signal on a PCB with a passive 10:1 oscilloscope probe, the probe tip introduces additional:

- A) Resistance that attenuates the signal by 10:1
- B) Capacitance (typically 10-15 pF) and inductance (from the ground lead) that form a resonant circuit, loading the circuit and limiting bandwidth
- C) Gain, amplifying the signal before the oscilloscope input
- D) Impedance matching that improves signal fidelity at high frequencies

---

**Q5.** In a SPICE simulation of a transmission line, what does the "TD" (time delay) parameter specify?

- A) The time constant of the transmission line's RC decay
- B) The one-way propagation delay of the transmission line (the time for a signal to travel from one end to the other)
- C) The rise time of the signal launched into the line
- D) The period of the highest-frequency component to be simulated

---

### Intermediate (Q6 -- Q11)

**Q6.** De-embedding is a post-processing technique applied to VNA measurements. Its primary purpose is:

- A) To increase the VNA's dynamic range by reducing measurement noise
- B) To mathematically remove the effects of fixtures, cables, or PCB launch structures from the measured S-parameters, revealing the S-parameters of the DUT alone
- C) To convert S-parameters measured at 50 Ohm reference impedance to a different reference impedance
- D) To interpolate S-parameter data to a finer frequency grid for SPICE simulation

---

**Q7.** A TDR measurement of a PCB microstrip trace shows the characteristic impedance rising from 50 Ohm to 65 Ohm and then returning to 50 Ohm over a 200 ps region. The most likely cause is:

- A) A via transition in the trace
- B) A section of the trace where the reference plane has a void or gap (split plane), reducing the capacitance per unit length and raising Z0
- C) A connector with high-impedance pins
- D) A short circuit to an adjacent trace

---

**Q8.** The 3 dB bandwidth of a single-pole oscilloscope system is related to rise time (10%-90%) by the well-known approximation:

- A) BW = rise_time / 0.35
- B) BW = 0.35 / rise_time
- C) BW = rise_time * 0.35
- D) BW = 0.5 / rise_time

---

**Q9.** In an electromagnetic (EM) field solver used for SI simulation, what is the difference between a 2D cross-section solver and a 3D full-wave solver?

- A) 2D solvers are less accurate for all geometries; 3D solvers are always preferable
- B) A 2D solver computes the per-unit-length RLGC parameters of a uniform transmission line cross-section (valid for long, uniform traces), while a 3D solver models discontinuities (vias, pads, connectors, bends) where the field varies in all three dimensions
- C) 2D solvers model signal and power integrity together, while 3D solvers model only signal integrity
- D) 3D solvers can only simulate passive structures up to 10 GHz

---

**Q10.** A VNA is used to measure a 4-port S-parameter model of a differential via on a PCB. The measurement shows S11 = -8 dB at 8 GHz. This indicates:

- A) The via is well-matched and has only 8 dB of insertion loss
- B) The return loss at port 1 is -8 dB, meaning approximately 16% of the incident power is reflected -- a significant impedance discontinuity
- C) The via has 8 dB of gain due to resonance in the barrel
- D) The measurement is invalid because vias cannot be measured with a 4-port VNA

---

**Q11.** When correlating simulation to measurement for a high-speed channel, a common source of discrepancy is the PCB dielectric constant (Dk) used in simulation differing from the actual Dk of the board material. If the measured channel delay is longer than simulated, this suggests:

- A) The actual Dk is lower than the value used in simulation
- B) The actual Dk is higher than the value used in simulation, because propagation delay scales as sqrt(Dk) and a higher Dk slows the wave
- C) The trace is physically longer than modelled
- D) The simulation does not include skin-effect losses, which slow the signal

---

### Advanced (Q12 -- Q15)

**Q12.** A 4-port VNA is used to measure the S-parameters of a differential PCB channel from 100 MHz to 40 GHz. After measurement, the data is to be used in a channel simulation. The engineer wants to check the passivity of the S-parameter model (that it does not generate energy). Which condition must be satisfied for a passive network?

- A) The imaginary parts of all S-parameters must be zero
- B) All S-parameters must have magnitudes less than 0 dB (|Sij| < 1 for all i, j)
- C) The singular values of the S-parameter matrix at each frequency must all be less than or equal to 1 (the S-matrix must be contractive)
- D) The real parts of all diagonal S-parameters (S11, S22, S33, S44) must be positive

---

**Q13.** When performing TDR measurements on a differential pair, the "odd mode" TDR and "even mode" TDR are performed by launching stimuli with specific polarities. How are these modes excited in a two-port TDR system?

- A) Odd mode: launch into one trace with the other left open; Even mode: launch into one trace with the other shorted to ground
- B) Odd mode: launch equal and opposite (differential) signals on both traces simultaneously; Even mode: launch equal and same-polarity (common-mode) signals on both traces simultaneously
- C) Odd mode: terminate both traces with 50 Ohm to ground; Even mode: terminate both traces with 100 Ohm differentially
- D) The TDR system cannot distinguish between odd and even modes without a balun

---

**Q14.** A simulation engineer is building a channel model for a PCIe Gen 5 link and needs to validate the model against board-level VNA measurements. The simulated IL at 16 GHz is -22 dB but the measured IL is -28 dB. After verifying trace length and geometry, the most systematic approach to root-cause the 6 dB discrepancy is:

- A) Adjust the simulation source resistance until the simulation matches measurement
- B) Isolate each loss contributor: re-measure Dk and Df of the actual PCB material at 16 GHz, verify the connector and via models, and check whether skin-effect roughness correction has been applied in the trace loss model
- C) Add a 6 dB pad to the simulation output to force correlation
- D) Accept the discrepancy as measurement error and proceed with the simulation

---

**Q15.** A signal integrity engineer is using a time-domain reflectometer (TDR) to characterise a microstrip trace. The TDR has a step rise time of 35 ps (10-90%). The spatial resolution of the TDR measurement (the minimum resolvable feature size along the line, in millimetres) for a microstrip on FR4 (Dk_eff = 3.8) is approximately:

- A) 35 mm
- B) 7 mm
- C) 3.2 mm
- D) 1.6 mm

---

## Answer Key

| Q  | Answer |
|----|--------|
| 1  | B      |
| 2  | B      |
| 3  | B      |
| 4  | B      |
| 5  | B      |
| 6  | B      |
| 7  | B      |
| 8  | B      |
| 9  | B      |
| 10 | B      |
| 11 | B      |
| 12 | C      |
| 13 | B      |
| 14 | B      |
| 15 | C      |

---

## Detailed Explanations

**Q1 -- Answer: B**

A VNA sweeps frequency and applies a known incident signal at each port, measuring the complex (magnitude and phase) ratio of reflected and transmitted waves. This yields the complete S-parameter matrix. It is inherently a frequency-domain instrument. Option A describes an oscilloscope (time-domain voltage waveforms). Option C describes an LCR meter or precision impedance analyser (which applies DC or low-frequency AC, not swept RF signals). Option D describes a noise figure analyser (Y-factor method using a calibrated noise source).

---

**Q2 -- Answer: B**

The reflection coefficient Gamma = (Z_L - Z0) / (Z_L + Z0). A positive reflection (same polarity as the incident step) means Gamma > 0, which requires Z_L > Z0. Inductive discontinuities (e.g., a via stub, a narrow trace, an inductive connector pin) have impedance that increases with frequency, producing a positive reflection overshoot. A higher-impedance region (wider trace on a different reference plane distance) also gives a positive reflection. Option A is incorrect -- a short or capacitive discontinuity gives a negative reflection (Gamma < 0). Option C (matched termination) gives zero reflection. Option D reverses the polarity for an inductor.

---

**Q3 -- Answer: B**

SOLT calibration uses four known calibration standards (Short, Open, Load = 50 Ohm, and Thru) to characterise and remove the systematic errors of the measurement system from the reference plane to the calibration standards. This includes directivity errors, source match errors, load match errors, and transmission tracking errors in the VNA hardware and test cables -- the 12-term error model for a 2-port system. Option A (thermal drift) is a hardware problem; calibration corrects for systematic errors at a point in time and should be repeated if temperature changes significantly. Option C (random noise) cannot be corrected by calibration; it is reduced by averaging. Option D describes an impedance transformation, which is a separate mathematical operation (renormalisation).

---

**Q4 -- Answer: B**

A passive 10:1 probe has a tip capacitance of typically 10-15 pF and a ground lead with 10-50 nH of inductance. Together they form a series resonant circuit with the trace impedance. This creates a resonance peak followed by roll-off, limiting the useful bandwidth to typically a few hundred MHz. The capacitive loading also affects the circuit under test. Option A is incorrect -- the 10:1 attenuation is achieved by a resistive divider (typically 9 MOhm probe tip + 1 MOhm scope input), not a change in the source resistance of the probe. Option C is incorrect -- passive probes never have gain. Option D is the opposite of what happens; passive probes degrade signal fidelity at high frequencies due to loading.

---

**Q5 -- Answer: B**

In SPICE (and most circuit simulators), the TD parameter for a transmission line model specifies the one-way propagation delay -- the time for the electromagnetic wave to travel from port 1 to port 2. It is used in the lossless (Berkeleigh) or lossy transmission line models to define the electrical length. Option A (RC time constant) is relevant for a lossy cable model but is not what TD defines in a T-line model. Option C (rise time) is a property of the source, not the transmission line. Option D (signal period) is unrelated to the TD parameter.

---

**Q6 -- Answer: B**

De-embedding removes the effect of the test fixture (PCB launch pads, SMA connectors, via transitions, edge-launch structures) from the raw VNA measurement, mathematically moving the calibration reference plane from the connector interface to the DUT ports. This is done by measuring the fixture's S-parameters separately (or characterising it with a 2x-thru or Open-Short method) and applying the inverse S-matrix transformation. Option A (dynamic range) is not improved by de-embedding -- it requires better hardware (lower noise floor, higher power). Option C describes renormalisation (re-referencing S-parameters to a different impedance). Option D describes interpolation, which is a separate data processing step.

---

**Q7 -- Answer: B**

A TDR step rising above Z0 indicates an impedance higher than Z0. A plane void or split in the reference plane (often caused by a power/ground plane split under the trace, routed across a plane split to a different voltage domain) reduces the capacitance per unit length because the electric field cannot terminate on the distant plane. With lower C and largely unchanged L, Z0 = sqrt(L/C) increases. The impedance rise is gradual (not instantaneous) because the split affects the spreading field region gradually. Option A (via transition) typically shows a short, sharp inductive spike (positive) followed by a capacitive dip (negative). Option C (high-impedance connector pin) would be a localised step, not a gradual rise. Option D (short to adjacent trace) would produce a sharp negative reflection.

---

**Q8 -- Answer: B**

The relationship between bandwidth and rise time for a single-pole (first-order) system is BW = 0.35 / rise_time, where rise_time is measured as 10% to 90% of the step response. This is derived from the relationship between the time constant (tau) and the pole frequency: tau = 1/(2*pi*BW), and rise_time = 2.2*tau for a single pole. Combining: rise_time = 2.2 / (2*pi*BW) = 0.35 / BW. This formula is used to estimate the measurement system's bandwidth needed to faithfully reproduce a signal with a given rise time. Options A, C, and D rearrange or alter the constants incorrectly.

---

**Q9 -- Answer: B**

A 2D cross-section solver (like a 2D FEM or method-of-moments solver) solves the Laplace/quasi-TEM equations in the plane perpendicular to the direction of propagation, assuming an infinitely long, uniform structure. It yields the per-unit-length L, C, R, and G matrices accurately for straight, uniform traces but cannot model 3D features. A 3D full-wave solver (FDTD, FEM, MoM) solves Maxwell's equations in all three dimensions and is required for discontinuities: vias, pads, SMA connectors, bends, transitions, and anything with a feature that varies along the direction of propagation. Option A is incorrect; 2D solvers are extremely accurate for what they model (uniform cross-sections). Option C is incorrect; EM solvers simulate passive structures for SI or PI, not both simultaneously in a single tool. Option D is incorrect; 3D solvers are used up to tens or hundreds of GHz.

---

**Q10 -- Answer: B**

S11 = -8 dB means the return loss is 8 dB. In linear magnitude: |S11| = 10^(-8/20) = 0.398. Power reflected = |S11|^2 = 0.158 = approximately 16%. An S11 of -8 dB is poor (a good SI channel target is S11 < -20 dB). It indicates a significant impedance discontinuity at the via. Option A confuses S11 (return loss) with S21 (insertion loss) and the magnitude is wrong for a "well-matched" interpretation. Option C is wrong; S-parameters of a passive network cannot exceed 0 dB (unless there is measurement error). Option D is incorrect; via structures are routinely characterised with 4-port (or 2-port) VNA measurements.

---

**Q11 -- Answer: B**

Propagation delay per unit length = sqrt(Dk_eff) / c. If the measured delay is longer than simulated, the wave is travelling slower, meaning the actual Dk is higher. A higher Dk means more capacitance per unit length (greater charge storage in the dielectric), which slows the wave. Option A has this backwards -- lower Dk speeds up the wave and would result in shorter measured delay. Option C (trace physically longer than modelled) is possible but the question asks about Dk discrepancy specifically. Option D is incorrect; skin-effect loss increases attenuation and changes rise time, but has a negligible effect on the delay of the signal's primary edge.

---

**Q12 -- Answer: C**

Passivity of an S-parameter network is confirmed by checking that the singular values of the S-matrix are all <= 1 at every frequency point. This is the mathematically rigorous condition: the S-matrix must be contractive (does not amplify). For a 2-port network, this simplifies to: |S11|^2 + |S21|^2 <= 1 and |S22|^2 + |S12|^2 <= 1. For larger port counts, singular value decomposition (SVD) of the full S-matrix is required. Option A is incorrect -- imaginary parts of S-parameters are non-zero for any reactive network; requiring them to be zero would restrict to pure resistive networks. Option B is necessary but not sufficient -- individual S-parameter magnitudes less than 1 do not guarantee passivity for all combinations of input. Option D describes a condition on the real part of Z-parameters, not S-parameters.

---

**Q13 -- Answer: B**

The differential (odd) mode of a pair is excited by equal and opposite signals: V1 = +V, V2 = -V. The common (even) mode is excited by equal and same-polarity signals: V1 = +V, V2 = +V. In a dual-port TDR capable of time-correlated simultaneous stimulus, the two channels launch the appropriate waveforms. The odd-mode TDR reveals the differential impedance (Z_diff = 2 * Z_odd), and the even-mode TDR reveals the common-mode impedance (Z_cm = Z_even / 2). Option A (one trace open, one driven) is a single-ended TDR measurement, not a modal measurement -- it gives a superposition of odd and even modes. Option C describes termination schemes, not stimulus conditions. Option D is incorrect; baluns convert between single-ended and differential and are not required for a dual-port TDR.

---

**Q14 -- Answer: B**

A 6 dB discrepancy between simulation and measurement at 16 GHz is substantial and must be root-caused, not patched. The systematic approach is: (1) verify material properties -- Dk and Df (loss tangent) at the actual operating frequency using a dedicated material characterisation coupon; many designers use nominal datasheet Dk/Df at 1 GHz but actual Df increases with frequency for FR4 and other lossy materials; (2) check conductor roughness -- skin-effect roughness (Huray, Hemispherical, or Groisse model) adds significant loss at high frequencies and is often omitted in basic simulations; (3) validate via and connector models with independent measurements. Option A (adjusting source resistance) changes the source match, not insertion loss. Option C (adding a 6 dB pad) forces correlation without understanding the cause and would invalidate the model. Option D is not an engineering approach -- a 6 dB error at 16 GHz in a PCIe Gen 5 channel is design-critical.

---

**Q15 -- Answer: C**

Spatial resolution of a TDR is determined by the rise time of the TDR step and the propagation velocity in the medium. The spatial resolution (half round-trip, since the TDR displays a one-way distance) is: delta_x = (v_prop * t_rise) / 2. Propagation velocity in FR4 with Dk_eff = 3.8: v = c / sqrt(3.8) = 3e8 / 1.949 = approximately 1.539e8 m/s. For a 35 ps rise time: delta_x = (1.539e8 * 35e-12) / 2 = (5.386e-3) / 2 = 2.69 mm, which is approximately 3.2 mm (option C, the closest answer). The factor of 2 arises because the TDR measures round-trip time but the display is calibrated to one-way distance, so two features separated by delta_x produce step edges separated by 2*delta_x / v_prop in time -- the resolution in time is t_rise, corresponding to delta_x in distance. Option A (35 mm) ignores propagation velocity and time-to-distance conversion entirely. Option B (7 mm) omits the factor of 2. Option D (1.6 mm) would require approximately 10 ps rise time.
