# Quiz: Transmission Lines

15 multiple-choice questions covering characteristic impedance, reflections, propagation delay, and losses. Questions span three difficulty tiers. Answers with explanations are collected at the end.

---

## Instructions

Select the single best answer for each question. After completing all questions, check your answers against the answer key. For each incorrect answer, read the full explanation before moving on.

Suggested time: 25 minutes.

---

## Questions

### Fundamentals (Q1 -- Q5)

**Q1.** The characteristic impedance Z0 of a lossless transmission line is defined as:

- A) The ratio of the total voltage to the total current at the source end
- B) The ratio of the forward-travelling voltage wave to the forward-travelling current wave
- C) The input impedance of the line when the far end is open-circuited
- D) The geometric mean of the source impedance and load impedance

---

**Q2.** A microstrip trace has a characteristic impedance of 50 Ohm. The dielectric constant (Dk) of the substrate is increased while all physical dimensions are held constant. The characteristic impedance will:

- A) Increase, because the capacitance per unit length decreases
- B) Decrease, because the capacitance per unit length increases
- C) Remain unchanged, because Z0 depends only on geometry
- D) Increase, because the inductance per unit length increases

---

**Q3.** A signal with a 1 V step travels along a 50 Ohm transmission line and reaches a load resistor of 100 Ohm. The reflection coefficient at the load is:

- A) +0.5
- B) -0.5
- C) +0.33
- D) +1.0

---

**Q4.** The propagation delay of a transmission line is primarily determined by:

- A) The characteristic impedance and the source resistance
- B) The physical length and the effective dielectric constant of the medium
- C) The rise time of the driving signal
- D) The loss tangent of the substrate material

---

**Q5.** Which of the following termination schemes places a resistor equal to Z0 at the source end of the transmission line?

- A) Parallel (shunt) termination
- B) Series termination
- C) AC termination
- D) Thevenin termination

---

### Intermediate (Q6 -- Q11)

**Q6.** A 50 Ohm, 1 ns transmission line is driven by a 25 Ohm source and terminated with an open circuit. A 1 V step is launched at t = 0. Which best describes the voltage at the load end at t = 2 ns (one round-trip after the initial wave arrives)?

- A) 1.33 V, then later settles to 1.67 V
- B) 1.67 V immediately, with no further bouncing
- C) 1.33 V immediately, settling to 2.0 V after multiple reflections
- D) 2.0 V, settling to 1.67 V

---

**Q7.** Skin effect causes signal attenuation that scales with frequency as:

- A) Linear with frequency (f)
- B) Square root of frequency (sqrt(f))
- C) Square of frequency (f^2)
- D) Logarithmically with frequency

---

**Q8.** Dielectric loss (loss tangent) causes signal attenuation that scales with frequency as:

- A) Independent of frequency
- B) Square root of frequency (sqrt(f))
- C) Linear with frequency (f)
- D) Square of frequency (f^2)

---

**Q9.** A transmission line stub is a section of unterminated line branching off from the main signal path. What is its primary negative effect on signal integrity?

- A) It increases the characteristic impedance of the main line
- B) It acts as a bandpass filter, boosting certain frequencies
- C) It creates a notch (null) in the frequency response at the frequency where the stub is a quarter-wavelength long
- D) It increases the DC resistance of the signal path

---

**Q10.** For a microstrip trace, increasing the trace width while keeping the dielectric height constant has which primary effect on characteristic impedance and propagation delay per unit length?

- A) Z0 increases; propagation delay per unit length increases
- B) Z0 decreases; propagation delay per unit length is largely unchanged
- C) Z0 increases; propagation delay per unit length decreases
- D) Z0 decreases; propagation delay per unit length increases

---

**Q11.** A transmission line is described as "electrically long" for a signal when:

- A) The physical length exceeds 1 metre
- B) The one-way propagation delay is greater than half the signal rise time
- C) The characteristic impedance is greater than 50 Ohm
- D) The signal frequency exceeds 1 GHz

---

### Advanced (Q12 -- Q15)

**Q12.** A via in a PCB high-speed design passes through an 8-layer board but only connects layers 1 to 4. The remaining via barrel from layers 4 to 8 forms a stub. At which approximate frequency does the first resonant null appear in the insertion loss if the stub length is 500 mils and the effective Dk of the via is 4.0? (1 mil = 25.4 um)

- A) Approximately 2.4 GHz
- B) Approximately 4.7 GHz
- C) Approximately 9.4 GHz
- D) Approximately 18.8 GHz

---

**Q13.** The RLGC model of a transmission line describes distributed resistance (R), inductance (L), conductance (G), and capacitance (C) per unit length. In a practical PCB trace at high frequency, which two parameters typically dominate the loss?

- A) R (skin-effect resistance) and G (dielectric conductance)
- B) L (inductance) and C (capacitance)
- C) R (DC resistance) and G (dielectric conductance)
- D) R (skin-effect resistance) and C (capacitance)

---

**Q14.** A differential pair has a single-ended characteristic impedance Z0_se = 55 Ohm when the traces are far apart. When the traces are brought closer together with tight coupling, which of the following correctly describes the change?

- A) The differential impedance decreases and the common-mode impedance increases
- B) The differential impedance increases and the common-mode impedance decreases
- C) Both differential and common-mode impedances decrease
- D) Both differential and common-mode impedances increase

---

**Q15.** In the telegraphers equations, the condition for a distortionless line (one where all frequency components travel at the same speed and with frequency-independent attenuation per unit length) is:

- A) R/L = G/C
- B) R*C = L*G
- C) R = G and L = C
- D) The line must be lossless (R = 0, G = 0)

---

## Answer Key

| Q  | Answer |
|----|--------|
| 1  | B      |
| 2  | B      |
| 3  | C      |
| 4  | B      |
| 5  | B      |
| 6  | C      |
| 7  | B      |
| 8  | C      |
| 9  | C      |
| 10 | B      |
| 11 | B      |
| 12 | B      |
| 13 | A      |
| 14 | A      |
| 15 | A      |

---

## Detailed Explanations

**Q1 -- Answer: B**

Z0 is defined as Vforward / Iforward, the ratio of the forward-travelling voltage wave to its associated current wave. It is a property of the line geometry and materials, not of the termination. Option A describes the input impedance, which equals Z0 only when the line is matched. Option C describes an open-circuit input impedance, which is purely reactive for a lossless line. Option D describes a geometric mean matching condition, not Z0 itself.

---

**Q2 -- Answer: B**

Z0 = sqrt(L/C) per unit length. Increasing Dk increases the capacitance per unit length (C) because C is proportional to the permittivity. With L unchanged and C larger, Z0 decreases. Option A has the effect backwards. Option C is wrong because geometry alone does not determine Z0 -- the dielectric fills the field region between trace and plane, directly affecting C. Option D is incorrect because L is primarily set by the geometry (trace width, dielectric height), and Dk has minimal effect on L for a microstrip.

---

**Q3 -- Answer: C**

The reflection coefficient Gamma_L = (Z_L - Z0) / (Z_L + Z0) = (100 - 50) / (100 + 50) = 50/150 = +0.33. Option A (+0.5) would be correct for a 150 Ohm load. Option B (-0.5) corresponds to a load less than Z0 (specifically 25 Ohm). Option D (+1.0) is an open circuit.

---

**Q4 -- Answer: B**

Propagation delay per unit length = sqrt(Dk_eff) / c, where c is the speed of light. It depends on the physical length and the effective dielectric constant. Option A is incorrect; source resistance affects the amplitude and rise time of reflections, not the propagation velocity. Option C is incorrect; rise time determines whether the line is electrically long, but does not change the propagation velocity. Option D is incorrect; loss tangent causes frequency-dependent attenuation and phase distortion, but the dominant propagation velocity is set by Dk_eff.

---

**Q5 -- Answer: B**

Series termination places a resistor at the source equal to (Z0 - Rs_driver) so the net source impedance matches Z0. This eliminates re-reflections from the source. The initial launched voltage is only half the final value -- the full voltage appears at the load after the first pass and no reflected wave returns. Option A (parallel/shunt termination) places Z0 at the load, absorbing the wave there. Option C (AC termination) uses an RC network in a shunt configuration. Option D (Thevenin) uses a voltage divider and a shunt resistor combination at the load.

---

**Q6 -- Answer: C**

Step through the lattice diagram. The source voltage divider with Zsource = 25 Ohm and Z0 = 50 Ohm launches a wave of V_inc = 1 * 50/(50+25) = 0.667 V. At the open-circuit load: Gamma_L = (inf - 50)/(inf + 50) = +1.0, so the reflected voltage equals the incident. Voltage at load on first arrival (t = 1 ns) = 0.667 + 0.667 = 1.333 V. The reflected wave travels back to the source where Gamma_S = (25 - 50)/(25 + 50) = -0.333, sending a further -0.222 V wave toward the load. Eventually the line charges to the open-circuit Thevenin voltage = 1.0 * (inf / (25+inf)) = 2.0 V. So the answer is 1.33 V at t = 1 ns, settling toward 2.0 V. Option B is wrong because the first bounce does not immediately produce 1.67 V at the load. Option D reverses the sequence.

---

**Q7 -- Answer: B**

Skin effect confines current to a thin skin depth delta = sqrt(2*rho / (omega*mu)), which decreases as 1/sqrt(f). Because the effective cross-sectional area decreases as 1/sqrt(f), the resistance per unit length increases as sqrt(f), and so the attenuation (proportional to resistance) scales as sqrt(f). Options A, C, and D describe different functional forms that do not match the physics of skin effect.

---

**Q8 -- Answer: C**

Dielectric loss attenuation alpha_d is proportional to frequency times the loss tangent: alpha_d = (pi * f * sqrt(Dk_eff) * tan_delta) / c. The loss tangent (tan_delta) is approximately constant over a wide frequency range for most PCB materials, so dielectric loss attenuation increases linearly with frequency. This is in contrast to skin-effect loss (sqrt(f)), meaning dielectric loss eventually dominates at higher frequencies. Options A, B, and D are incorrect functional relationships.

---

**Q9 -- Answer: C**

An unterminated stub acts as a resonant element. When the stub length equals a quarter wavelength (lambda/4), the open-circuit end reflects with +1 and the round trip creates a short circuit at the stub junction, placing a near-zero impedance across the signal path and creating a notch in insertion loss. This is a critical concern in DDR4/5 fly-by routing and PCB via stubs. Option A is incorrect; the stub is a shunt element, not a series element, and does not directly change Z0. Option B is wrong; it acts as a notch filter, not a bandpass filter. Option D is incorrect; the stub does not carry DC current in a properly AC-coupled line.

---

**Q10 -- Answer: B**

For a microstrip, increasing trace width increases the capacitance per unit length (the trace presents a larger parallel-plate area to the reference plane) while the inductance per unit length decreases. Both effects drive Z0 = sqrt(L/C) downward. The propagation delay per unit length (tpd = sqrt(L*C)) is primarily set by the effective dielectric constant, which changes only slightly with geometry changes; it is largely independent of trace width. Options A and C incorrectly state that Z0 increases. Option D overstates the effect on propagation delay.

---

**Q11 -- Answer: B**

A line is electrically long when the signal's transition time is comparable to or shorter than twice the line's propagation delay (round-trip delay). A common rule of thumb: treat a line as electrically long when its one-way delay exceeds one-sixth of the signal rise time (or equivalently, the round-trip delay exceeds one-third of the rise time). The most physically meaningful definition is that reflections can arrive back at the source before the launched edge has settled, causing distortion. Option A uses physical length without reference to frequency or rise time, which is meaningless in isolation. Options C and D introduce impedance level and frequency thresholds that are not the fundamental criterion.

---

**Q12 -- Answer: A**

The stub resonance frequency f = c / (4 * L_stub * sqrt(Dk_eff)). Converting 500 mils to metres: 500 * 25.4e-6 = 12.7 mm = 0.0127 m. f = 3e8 / (4 * 0.0127 * sqrt(4.0)) = 3e8 / (4 * 0.0127 * 2) = 3e8 / 0.1016 = approximately 2.95 GHz, closest to option A. The practical lesson is that a 500 mil stub in FR4 resonates near 3 GHz, directly degrading PCIe Gen 3/4 and 10G Ethernet. Backdrilling the stub removes this resonance.

---

**Q13 -- Answer: A**

At high frequency, conductor loss is dominated by skin-effect resistance (R scales as sqrt(f)) and dielectric loss is dominated by G (conductance per unit length, which represents energy absorbed by the lossy dielectric and scales linearly with f). Both R and G contribute to insertion loss. Option B (L and C) describes the reactive elements that set Z0 and propagation velocity but are not loss terms. Option C is wrong because DC resistance is negligible at high frequency compared to skin-effect resistance. Option D omits G, which becomes significant at multi-GHz frequencies.

---

**Q14 -- Answer: A**

For a tightly coupled differential pair, the mutual inductance M and mutual capacitance Cm between the traces modify the modal impedances. The differential impedance Z_diff = 2 * Z0_se * (1 - k), where k is the coupling coefficient. As coupling increases (k increases), Z_diff decreases. The common-mode impedance Z_cm = (Z0_se / 2) * (1 + k), which increases with coupling. This is why routing differential pairs closer together (to reduce EMI) also reduces the differential impedance and requires adjustment of trace width to maintain the target impedance. Options B, C, and D misstate the direction of one or both changes.

---

**Q15 -- Answer: A**

The Heaviside condition for a distortionless line is R/L = G/C, equivalently written R*C = L*G. This ensures that the propagation constant gamma = sqrt((R+jwL)(G+jwC)) has a frequency-independent real part (attenuation) and a phase velocity that is independent of frequency (no dispersion). R/L = G/C rearranges to R*C = L*G, so options A and B are algebraically equivalent; the canonical form stated in most textbooks is R/L = G/C (option A). Option C (R = G, L = C) is a special case that would only hold for a line with specific non-physical units. Option D (lossless line) avoids the distortion problem trivially but is physically unrealisable; the question asks about the distortionless condition for a lossy line.
