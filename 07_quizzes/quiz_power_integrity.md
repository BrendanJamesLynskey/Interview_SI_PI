# Quiz: Power Integrity

15 multiple-choice questions covering PDN impedance, target impedance, decoupling capacitor selection and placement, and plane resonance. Questions span three difficulty tiers. Answers with explanations are collected at the end.

---

## Instructions

Select the single best answer for each question. After completing all questions, check your answers against the answer key. For each incorrect answer, read the full explanation before moving on.

Suggested time: 25 minutes.

---

## Questions

### Fundamentals (Q1 -- Q5)

**Q1.** The target impedance of a power distribution network (PDN) is defined as:

- A) The impedance of the voltage regulator module (VRM) at DC
- B) The maximum allowable PDN impedance over the frequency range of interest, calculated from the allowed voltage ripple and the maximum transient current step
- C) The characteristic impedance of the power planes in the PCB
- D) The parallel combination of all decoupling capacitors on the board

---

**Q2.** Why does a real decoupling capacitor have a self-resonant frequency (SRF)?

- A) Because the PCB power plane creates a resonant cavity at high frequencies
- B) Because the capacitor package and PCB via have parasitic inductance (ESL) that resonates with the capacitor's capacitance
- C) Because the capacitor's dielectric constant varies with applied voltage
- D) Because the capacitor absorbs energy at the frequency where its reactance equals its resistance (ESR)

---

**Q3.** Below its self-resonant frequency, a real decoupling capacitor behaves primarily as:

- A) A resistor (ESR dominated)
- B) An inductor (ESL dominated)
- C) A capacitor (capacitance dominated)
- D) An open circuit

---

**Q4.** The VRM (voltage regulator module) in a PDN is typically effective at which frequency range?

- A) DC to a few hundred kHz (limited by its control loop bandwidth)
- B) 1 MHz to 100 MHz (its switching frequency range)
- C) Above 1 GHz only
- D) It is equally effective at all frequencies

---

**Q5.** In a PDN impedance profile, a well-designed PDN should ideally:

- A) Have a flat impedance profile from DC to the highest frequency of interest, at or below the target impedance
- B) Have the lowest possible impedance only at DC
- C) Have sharp resonant peaks below the target impedance
- D) Have high impedance above 1 GHz because current transients at those frequencies are negligible

---

### Intermediate (Q6 -- Q11)

**Q6.** A processor draws a maximum current step of 2 A with an allowed supply voltage droop of 50 mV. The required target impedance is:

- A) 100 mOhm
- B) 25 mOhm
- C) 50 mOhm (purely resistive)
- D) 4 mOhm

---

**Q7.** An anti-resonance peak in a PDN impedance profile typically occurs when:

- A) The VRM loop bandwidth is too low
- B) Two decoupling capacitors in the network are at their individual self-resonant frequencies simultaneously
- C) The inductive reactance of one capacitor group meets the capacitive reactance of another capacitor group at the same frequency, creating a parallel LC resonance with a high impedance peak
- D) The power plane is too thin, increasing its resistance

---

**Q8.** Placing a decoupling capacitor physically closer to a load IC reduces which parasitic element most significantly?

- A) The capacitor's ESR
- B) The capacitor's intrinsic ESL (within the package)
- C) The mounting inductance (via and trace inductance between capacitor and IC power pins)
- D) The capacitor's capacitance value

---

**Q9.** The power plane capacitance in a PCB (formed by the power and ground planes) is most effective at which frequency range?

- A) DC to 1 kHz (it replaces the VRM at low frequencies)
- B) 100 MHz to several GHz (it provides very low inductance charge storage for fast transients)
- C) It is effective only at a single resonant frequency
- D) 1 kHz to 100 kHz (the same range as bulk capacitors)

---

**Q10.** The spreading inductance of a power plane is the inductance associated with current spreading from a via to the plane. What is its primary effect on PDN performance?

- A) It increases the effective capacitance of the plane
- B) It creates an inductive component in series with the plane capacitance, raising the PDN impedance at high frequencies and limiting the effective frequency range of the plane capacitance
- C) It reduces the EMI radiated from the power planes
- D) It is beneficial because it filters high-frequency noise

---

**Q11.** A designer wants to reduce the PDN impedance in the 100 MHz to 500 MHz range. The most effective approach is typically:

- A) Add more bulk electrolytic capacitors near the VRM
- B) Increase the VRM switching frequency
- C) Add multiple small ceramic capacitors (e.g., 100 nF, 0402 or 0201 package) with low ESL, distributed close to the load IC power pins
- D) Increase the number of PCB layers to add more copper to the power plane

---

### Advanced (Q12 -- Q15)

**Q12.** Power plane resonance in a PCB creates standing waves that depend on the plane geometry. For a rectangular power-ground plane pair with dimensions L x W and effective Dk of 4.0, the lowest resonant frequency (dominant mode) is:

- A) f = c / (2 * L * sqrt(Dk)), where L is the longest dimension
- B) f = c / (4 * L * sqrt(Dk)), where L is the longest dimension
- C) f = c * sqrt(Dk) / (2 * L), which gives a higher frequency for larger Dk
- D) f = c / (L * sqrt(Dk)), without the factor of 2

---

**Q13.** In a PDN simulation, the impedance is plotted from 1 MHz to 3 GHz. A sharp resonant peak is observed at 850 MHz that exceeds the target impedance. Which measurement or analysis would best identify whether this is a plane resonance or a capacitor anti-resonance?

- A) Measure the ESR of each decoupling capacitor with an LCR meter
- B) Compare the resonant frequency against predicted plane resonance frequencies from the plane geometry and Dk, and check whether the peak disappears when capacitors are removed from the simulation
- C) Increase the VRM bandwidth to suppress the peak
- D) Measure the S11 at the power pin with a VNA and compare to a simulation without planes

---

**Q14.** The concept of "PDN noise budget" in a multi-rail power system requires the designer to consider:

- A) Only the highest-power rail because lower power rails have negligible noise
- B) Each rail independently, because coupling between rails does not occur through a shared PDN
- C) Coupling between rails through shared return paths, simultaneous switching noise (SSN) from multiple ICs on shared planes, and the contribution of each rail's noise to the total supply variation at sensitive loads
- D) The thermal noise of the decoupling capacitors, which limits the minimum achievable noise floor

---

**Q15.** A power integrity engineer is tasked with meeting a target impedance of 10 mOhm from DC to 1 GHz for a CPU VDD rail. The VRM handles DC to 500 kHz. A SPICE model of the PDN shows the impedance exceeds 10 mOhm at 450 MHz with a 35 mOhm peak. The phase of the impedance at 450 MHz is nearly +90 degrees (inductive). Which corrective action is most appropriate?

- A) Add more 10 uF capacitors near the VRM to boost the mid-frequency response
- B) Increase the number of PCB layers to reduce plane resistance
- C) Add 100 nF ceramic capacitors (0201 package) directly under the CPU package at the power delivery vias, targeting their SRF to around 450 MHz to create a local impedance minimum
- D) Replace the VRM with a higher-bandwidth unit to extend its regulation range to 450 MHz

---

## Answer Key

| Q  | Answer |
|----|--------|
| 1  | B      |
| 2  | B      |
| 3  | C      |
| 4  | A      |
| 5  | A      |
| 6  | B      |
| 7  | C      |
| 8  | C      |
| 9  | B      |
| 10 | B      |
| 11 | C      |
| 12 | A      |
| 13 | B      |
| 14 | C      |
| 15 | C      |

---

## Detailed Explanations

**Q1 -- Answer: B**

Target impedance Z_target = delta_V / delta_I, where delta_V is the maximum allowed voltage ripple (as a fraction of the supply voltage, e.g., 5% of Vdd) and delta_I is the maximum transient current step. It is a design constraint, not a measured property of the VRM or planes alone. Option A (VRM DC impedance) is a component property, not the system target. Option C (plane characteristic impedance) is a different concept entirely. Option D describes the resultant capacitor bank impedance, not the target.

---

**Q2 -- Answer: B**

Every physical capacitor has series inductance (ESL) from its internal structure, the package leads or pads, PCB vias, and connecting traces. This ESL forms a series LC circuit with the capacitance. The self-resonant frequency is SRF = 1 / (2*pi*sqrt(L*C)). Below SRF, C dominates; above SRF, L dominates. Option A describes power plane resonance, a different phenomenon. Option C (DC bias effect) is the piezoelectric/ferroelectric voltage coefficient of ceramic capacitors, which reduces effective capacitance but does not create SRF. Option D describes the ESR frequency, which is not the same as SRF.

---

**Q3 -- Answer: C**

Below the SRF, the capacitive reactance (1/2*pi*f*C) exceeds the inductive reactance (2*pi*f*ESL), so the component presents net capacitive impedance -- it decreases with frequency. This is the desired decoupling region. Option B describes behaviour above SRF. Option A (ESR dominated) is only accurate right at the SRF, where the reactances cancel. Option D (open circuit) is the behaviour of an inductor at very high frequency.

---

**Q4 -- Answer: A**

The VRM's output voltage regulation is controlled by a feedback loop. The bandwidth of this loop (typically 50 kHz to a few hundred kHz for a standard buck converter) limits how fast the VRM can respond to current transients. Above its loop bandwidth, the VRM output impedance rises and it cannot suppress noise. Option B is incorrect; the switching frequency (typically 300 kHz to several MHz) is not the regulation bandwidth. Option C is wrong -- above 1 GHz the VRM is ineffective and decoupling capacitors must handle transients. Option D is incorrect; the VRM becomes inductive (high impedance) above its loop bandwidth.

---

**Q5 -- Answer: A**

The goal of PDN design is to maintain the impedance at or below the target impedance across all frequencies where the load generates significant current transients. A flat profile means no frequencies are unaddressed. Option B ignores high-frequency transients from digital logic switching. Option C is wrong; resonant peaks that remain below target impedance are acceptable, but peaks above target impedance cause supply voltage excursions. Option D is incorrect; modern high-speed digital ICs generate significant current switching events at frequencies well above 1 GHz.

---

**Q6 -- Answer: B**

Z_target = delta_V / delta_I = 50 mV / 2 A = 25 mOhm. This is the maximum allowable PDN impedance at any frequency at which the current transient has significant spectral content. Option A (100 mOhm = 50 mV / 0.5 A) uses the wrong current. Option C (50 mOhm) uses delta_I = 1 A. Option D (4 mOhm) has no basis in the given numbers.

---

**Q7 -- Answer: C**

Anti-resonance occurs at the frequency where the inductive reactance of one capacitor group (which has passed its SRF and is now inductive) equals the capacitive reactance of another capacitor group (which is below its SRF). This creates a parallel resonant circuit with a high impedance peak -- the opposite of the desired behaviour. Option A (VRM bandwidth) creates a rising impedance below the VRM's bandwidth limit, not a sharp peak. Option B is partially right in setup but wrong in mechanism -- two capacitors at their individual SRFs are not the cause; it is the interaction between one inductive group and one capacitive group. Option D (thin plane) increases resistive loss uniformly.

---

**Q8 -- Answer: C**

The total inductance in the current path from a capacitor's charge store to the IC power pin includes: (1) the capacitor's intrinsic ESL, and (2) the mounting inductance (via to capacitor pad, PCB trace from capacitor to via, via to power plane, plane spreading, IC package via). Placing the capacitor closer to the IC primarily reduces the trace length and thus the mounting inductance, which is a controllable variable. The intrinsic ESL (option B) is fixed by the capacitor package and cannot be changed by placement. ESR (option A) is a material property. Capacitance (option D) does not change with placement.

---

**Q9 -- Answer: B**

Power plane capacitance is formed by the dielectric between the power and ground planes, which are very close together (typically 100 um or less). This proximity gives very low inductance (essentially the inductance of a few via transitions), making the plane capacitance effective at high frequencies -- typically 100 MHz to several GHz, depending on board dimensions and Dk. Option A is wrong; plane capacitance is far too small in absolute value to replace bulk capacitors at DC. Option C is wrong; plane capacitance is a distributed element, not a lumped resonant element (though the planes do resonate). Option D is incorrect; that is the range where bulk electrolytics operate.

---

**Q10 -- Answer: B**

Current flowing into a via must spread radially outward through the power plane to reach multiple capacitors or load devices. This spreading path has a small but non-negligible inductance (spreading inductance). It creates an inductive impedance in series with the plane capacitance, raising the effective SRF of the plane-capacitor system and limiting how high in frequency the plane capacitance is useful. Option A is incorrect -- spreading inductance does not increase capacitance. Option C is incorrect -- spreading inductance has little effect on EMI radiation directly. Option D is incorrect; inductance in series with a decoupling element raises impedance, which is harmful, not beneficial.

---

**Q11 -- Answer: C**

The 100 MHz to 500 MHz range is above the effective frequency of bulk capacitors and VRM, and below the frequency where power planes become fully effective. This "mid-frequency gap" is best filled by ceramic capacitors in small packages (0402 or 0201) with low ESL, placed near the IC. Their SRF can be placed in this frequency range, providing a low impedance point. Multiple values in parallel spread the effective low-impedance band. Option A (bulk electrolytic) is effective only up to ~1 MHz. Option B (VRM switching frequency) is unrelated to the decoupling frequency. Option D (more copper layers) reduces DC resistance marginally but does not address 100-500 MHz decoupling.

---

**Q12 -- Answer: A**

A rectangular plane pair acts as a 2D resonant cavity. The resonant frequencies are given by f_mn = (c / (2*sqrt(Dk))) * sqrt((m/L)^2 + (n/W)^2), where m and n are integers. The dominant (lowest) mode is the (1,0) mode: f_10 = c / (2*L*sqrt(Dk)), where L is the longest dimension and n=0. Option B uses a factor of 4, which applies to a quarter-wavelength resonator (e.g., a stub open at one end and short at the other), not a plane pair. Option C incorrectly puts sqrt(Dk) in the numerator, which would give higher frequency for higher Dk -- the opposite of the correct relationship (higher Dk slows the wave, lowering resonance frequency). Option D omits the factor of 2.

---

**Q13 -- Answer: B**

To distinguish a plane resonance from a capacitor anti-resonance: (1) calculate the plane resonance frequencies from the plane dimensions and Dk; (2) run the simulation without capacitors to see if the peak is present (it would be if it is a plane resonance); (3) add capacitors back and check if the peak is created by the capacitor addition (indicating an anti-resonance between capacitor groups). Option A (LCR meter on capacitors) can characterise capacitor parasitics but does not directly identify the source of the system-level PDN peak. Option C (increase VRM bandwidth) is a corrective action, not diagnostic. Option D (VNA S11 measurement) can validate the simulation but does not directly differentiate between the two causes without the analysis step.

---

**Q14 -- Answer: C**

In a real system, power rails share PCB return plane layers, and the return currents from multiple switching loads mix in the ground reference. Simultaneous switching of multiple ICs causes ground bounce and supply rail noise that couples between ICs via the shared PDN. Designers must account for the worst-case simultaneous switching scenario and the supply voltage sensitivity of each load. Option A is incorrect -- low-power rails can supply noise-sensitive analog or I/O circuits that are more sensitive to rail noise than high-power digital rails. Option B is incorrect because rail isolation is never perfect -- the shared return path is a fundamental coupling mechanism. Option D is incorrect; thermal noise of capacitors is negligibly small compared to switching noise.

---

**Q15 -- Answer: C**

The impedance is inductive at 450 MHz and peaks at 35 mOhm. This means the PDN is under-capacitanced at that frequency -- the capacitors providing coverage below 450 MHz have already become inductive, and the next capacitor group's SRF is too high. The solution is to add capacitors whose SRF is at or near 450 MHz: 0201 100 nF ceramic capacitors placed at the CPU package power delivery vias will resonate near this frequency and create a local impedance minimum to bring the peak below 10 mOhm. Option A (10 uF near VRM) addresses the range below ~10 MHz and will not help at 450 MHz. Option B (more PCB layers) is irrelevant to a mid-frequency inductive impedance. Option D (higher bandwidth VRM) is impractical -- extending VRM bandwidth to 450 MHz is not feasible with standard switching regulators and would not be the right approach; the issue is local decoupling, not VRM bandwidth.
