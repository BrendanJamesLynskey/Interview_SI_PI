# Oscilloscope Probing

## Overview

Correct probing technique is one of the most frequently underestimated skills in SI/PI engineering. Even a state-of-the-art oscilloscope produces meaningless results if the probe introduces excessive loading, bandwidth limiting, or ground-path inductance. This section covers the physics of probe loading, the trade-offs between probe types, differential probing for high-speed channels, and the systematic approach to knowing when your measurement is trustworthy. These topics appear frequently in hands-on SI interviews at hardware companies and in board bring-up discussions.

---

## Tier 1: Fundamentals

### Q1. What is probe bandwidth, and how does it limit your measurement?

**Answer:**

Probe bandwidth is the frequency at which the probe's attenuation of the measured signal reaches $-3$ dB relative to its DC passband gain. A probe with 1 GHz bandwidth attenuates a 1 GHz sine wave by 3 dB (reduces amplitude to 70.7% of the true value) and attenuates higher frequencies more severely.

**The bandwidth-rise time relationship:**

The system bandwidth $BW_{system}$ is determined by the bandwidth of every element in the signal chain: the probe, the oscilloscope input amplifier, and the cable between them. For a cascade of first-order systems (Gaussian approximation):

$$BW_{system} \approx \frac{1}{\sqrt{\frac{1}{BW_{probe}^2} + \frac{1}{BW_{scope}^2}}}$$

The corresponding system rise time is:

$$t_{r,system} = \frac{0.35}{BW_{system}}$$

The measured rise time $t_{r,meas}$ relates to the true signal rise time $t_{r,signal}$ by:

$$t_{r,signal} = \sqrt{t_{r,meas}^2 - t_{r,system}^2}$$

**Practical rule:** The measurement system bandwidth should be at least three to five times the highest frequency of interest in the signal. For a 10 Gbps signal (Nyquist 5 GHz, 3rd harmonic 15 GHz), a 20–30 GHz probe and scope are needed for accurate time-domain waveform capture.

**Example:** A 2 GHz oscilloscope used with a 500 MHz probe on a 1 GHz signal:

$$BW_{system} = \frac{1}{\sqrt{1/4 + 1/0.25}} \times 10^{-9} \approx 447\ \text{MHz}$$

The system will smear a 700 ps transition to approximately:

$$t_{r,system} = \frac{0.35}{447\ \text{MHz}} \approx 783\ \text{ps}$$

A true 700 ps rise time would appear as $\sqrt{783^2 + 700^2} \approx 1.05$ ns — a 50% error. The measurement is fundamentally unreliable.

---

### Q2. What is probe loading, and how does a passive 10x probe affect the circuit under test?

**Answer:**

Probe loading is the change in circuit behaviour caused by connecting the probe. Every probe presents an impedance $Z_{probe}$ in parallel with the measurement node, altering the node voltage, current, and frequency response.

**Passive 10x probe equivalent circuit:**

A standard 10:1 passive probe is modelled as:

$$Z_{probe} = \left(R_{tip} + j\omega L_{ground}\right) \| \left(\frac{1}{j\omega C_{cable}} \| R_{comp}\right)$$

Typical values for a 500 MHz 10x passive probe:

| Element | Typical Value | Effect |
|---|---|---|
| Tip resistance $R_{tip}$ | 9 M$\Omega$ (series with 1 M$\Omega$ scope input → 10 M$\Omega$ total) | DC loading: minimal on most circuits |
| Ground lead inductance $L_{ground}$ | 20–100 nH (for a 5–10 cm ground lead) | Creates resonance with tip capacitance; causes ringing |
| Tip capacitance $C_{tip}$ | 8–15 pF | Loads high-impedance nodes at AC; limits bandwidth |
| Cable capacitance $C_{cable}$ | 100 pF | Attenuated 10x to effective 10 pF at the probe tip |

**Impact of tip capacitance:**

The probe tip capacitance $C_{tip}$ in parallel with the source impedance $R_s$ forms a low-pass filter:

$$f_{-3dB,loading} = \frac{1}{2\pi R_s C_{tip}}$$

For a 50 $\Omega$ transmission-line node and $C_{tip} = 10$ pF: $f_{-3dB} = 318$ MHz. The probe limits the measurement bandwidth at this node to 318 MHz regardless of the oscilloscope's bandwidth.

**Impact of ground lead inductance:**

The ground lead inductance $L_{ground}$ in series with $C_{tip}$ creates a series resonant circuit. The resonant frequency for $L_{ground} = 40$ nH and $C_{tip} = 10$ pF is:

$$f_{res} = \frac{1}{2\pi\sqrt{L_{ground} C_{tip}}} = \frac{1}{2\pi\sqrt{40 \times 10^{-9} \times 10 \times 10^{-12}}} \approx 252\ \text{MHz}$$

This resonance appears as a prominent peak in the probe's frequency response followed by a steep roll-off — the classic "ringing" observed on fast edge measurements with long ground leads.

**Mitigation:** Use the shortest possible ground lead. Replace the alligator-clip ground with a coaxial barrel adapter or spring-tip ground. For signals above 500 MHz, use an active probe.

---

### Q3. What is the difference between a high-impedance passive probe, a high-impedance active probe, and a 50 $\Omega$ matched passive probe?

**Answer:**

**High-impedance passive probe (standard 10x):**

- Input impedance: 10 M$\Omega$ parallel with 10–15 pF
- Bandwidth: 300 MHz – 1.5 GHz (depending on quality)
- Loading on 50 $\Omega$ nodes: significant above 200–300 MHz
- Use case: general logic, power supply measurements, slow analog signals
- Advantage: inexpensive, rugged, wide dynamic range

**High-impedance active probe:**

- Input impedance: 100 k$\Omega$ to 1 M$\Omega$ parallel with 0.1–1 pF
- Bandwidth: 1–30 GHz (powered by a probe supply from the oscilloscope)
- Loading: minimal — even at 50 $\Omega$ nodes, 0.5 pF loads to $f_{-3dB} = 1/(2\pi \times 50 \times 0.5 \times 10^{-12}) \approx 6.4$ GHz
- Use case: high-speed single-ended signals, IC test points, probing without dedicated SMA connectors
- Limitation: small dynamic range ($\pm 1$–2 V typical); sensitive to physical handling; expensive

**50 $\Omega$ passive probe (terminated):**

- Input impedance: 50 $\Omega$ (matched to the cable and scope input)
- Bandwidth: DC to 3–12 GHz depending on design
- Loading: 50 $\Omega$ in parallel with the source — this terminates the line, potentially changing circuit behaviour at unterminated nodes
- Use case: 50 $\Omega$ systems (RF coaxial, SMA connectors on PCB), measurements where a terminated load is acceptable
- Advantage: flat frequency response, no resonances, low noise
- Limitation: cannot probe high-impedance nodes; the 50 $\Omega$ load draws significant current ($V/50$) from the circuit

**Selection guide:**

| Signal environment | Best probe type |
|---|---|
| Power rails (< 100 MHz ripple) | 10x passive |
| Logic signals < 500 MHz | 10x passive |
| High-speed single-ended (500 MHz – 6 GHz) | High-impedance active |
| Signals at 50 $\Omega$ SMA connectors | 50 $\Omega$ passive (terminated) |
| Differential pairs on PCB at GHz | Differential active probe or two-channel active probe with math |

---

## Tier 2: Intermediate

### Q4. Describe how differential probing works and why it is preferred over measuring two single-ended signals and computing their difference in software.

**Answer:**

**Differential probing with a dedicated differential probe:**

A differential active probe has two input tips (P+ and P−) with a matched internal amplifier that subtracts the two signals:

$$V_{out} = A_{DM} \cdot (V_+ - V_-) + A_{CM} \cdot \frac{V_+ + V_-}{2}$$

where $A_{DM}$ is the differential gain and $A_{CM}$ is the common-mode gain. The Common Mode Rejection Ratio (CMRR) is:

$$\text{CMRR} = 20 \log_{10}\left(\frac{A_{DM}}{A_{CM}}\right) \quad \text{[dB]}$$

A good differential probe achieves 40–60 dB CMRR at 1 GHz. This means common-mode noise (ground bounce, power supply coupling) is rejected by 100–1000x before the signal enters the oscilloscope.

**Why the two-channel software subtraction method fails at high speed:**

When two separate single-ended channels measure $V_+$ and $V_-$ independently and compute $V_{diff} = V_+ - V_-$ in the oscilloscope's math function, the CMRR depends entirely on how well the two acquisition channels are matched. Typical mismatch sources:

1. **Channel skew:** Even 1 ps of time offset between the two channels creates a differential error at high frequencies. For a 100 ps edge, 1 ps skew represents 1% of the transition time — visible as a glitch on the computed difference.

2. **Gain mismatch:** 0.5% gain difference between two scope channels produces $\text{CMRR} = 20\log_{10}(200/1) = 46$ dB at DC, but CMRR degrades with frequency as the response curves diverge.

3. **Cable length mismatch:** Two probe cables of different lengths introduce differential delay, acting as skew.

**Quantitative comparison:**

For a 1 V common-mode signal and CMRR = 40 dB:

- Differential probe: common-mode noise contribution to output = $1\ \text{V}/100 = 10\ \text{mV}$
- Two-channel subtraction with 1% gain mismatch: common-mode noise contribution = $1\ \text{V} \times 0.01 = 10\ \text{mV}$ at DC, degrading to 50–100 mV at GHz frequencies

**Practical conclusion:** For signals below 200 MHz with $< 500$ mV common-mode noise, two-channel subtraction is acceptable. Above 500 MHz or with significant ground bounce, a dedicated differential probe is mandatory for reliable results.

---

### Q5. How do you measure power supply ripple on a low-voltage rail (e.g., 1.0 V at 50 mV ripple) accurately with an oscilloscope?

**Answer:**

This is a practical test that is commonly posed in interviews for PI engineers. The challenge is that the ripple is 5% of the rail voltage — the scope must be set to high sensitivity (e.g., 10 mV/div) while the DC offset is 1.0 V. At 10 mV/div, the 1.0 V offset consumes 100 divisions of range, which exceeds the scope's dynamic range.

**Method 1 — AC coupling (simple, low-frequency ripple):**

Switch the scope input to AC coupling. The DC component is blocked by an internal capacitor. Set the sensitivity to 10–20 mV/div. This works for ripple above $f_{-3dB,AC} = 1/(2\pi R_{input} C_{AC}) \approx 10$ Hz (for 1 M$\Omega$ input and $10\ \mu$F AC cap). It fails for very low-frequency load transients or switching ripple below ~1 kHz.

**Method 2 — DC offset compensation with DC coupling (preferred for broadband):**

Use DC coupling with the oscilloscope's vertical offset control to shift the 1.0 V DC level to mid-screen, then zoom the sensitivity to 10 mV/div. This avoids any bandwidth penalty from the AC coupling capacitor. The trade-off is that the input amplifier must handle the full 1.0 V DC + ripple within its range — verify that the input amplifier is not saturated at the coarse setting before changing sensitivity.

**Method 3 — External DC block:**

Use a high-quality coaxial DC block (rated to the required bandwidth) at the probe input. This is functionally equivalent to AC coupling but uses an external component with better-characterised frequency response, no scope-internal bandwidth limitation, and a precise corner frequency.

**Method 4 — Power rail probe:**

Dedicated power rail probes (e.g., Tektronix TPR1000, Keysight N7020A) are designed exactly for this measurement. They present a high input impedance (typically 50 k$\Omega$ || 0.9 pF), have low tip inductance (minimising ground resonance), and have built-in DC offset capability to accommodate 1–60 V rails while measuring millivolt-level ripple.

**Probing technique for minimum noise:**

- Use a coaxial SMA connection to the power plane if available, not an alligator clip
- Keep the probe ground as short as physically possible (< 5 mm using a spring-tip or SMD grab clip)
- Avoid probing at through-hole component leads — these have significant lead inductance that adds to the measured waveform

**What to measure:** Ripple frequency (should match $f_{sw}$ or $2 f_{sw}$ for the converter topology), peak-to-peak amplitude, rise/fall time of transient events, and the settling behaviour after a load step.

---

### Q6. What is probe input derating, and what happens to bandwidth and input impedance as frequency increases?

**Answer:**

**Bandwidth derating with attenuation ratio:**

Many high-impedance active probes offer selectable attenuation ratios (e.g., 2:1 and 10:1) to accommodate different signal amplitudes. Increasing attenuation reduces noise floor but may reduce bandwidth:

- At 2:1 attenuation, tip capacitance is lower but input impedance is also lower
- At 10:1 attenuation, the additional series resistance increases noise floor but reduces loading

Always verify the probe's datasheet bandwidth at the specific attenuation setting being used.

**Input impedance versus frequency:**

Even a high-impedance probe (e.g., 1 M$\Omega$ || 1 pF at DC) presents decreasing impedance with frequency. The reactive component dominates:

$$|Z_{probe}(f)| = \left|\frac{1}{j\omega C_{in}} \| R_{in}\right| = \frac{R_{in}}{\sqrt{1 + (2\pi f R_{in} C_{in})^2}}$$

For $R_{in} = 100\ \text{k}\Omega$ and $C_{in} = 1\ \text{pF}$:

$$|Z_{probe}| = 100\ \text{k}\Omega \text{ at DC}$$
$$|Z_{probe}| = \frac{100\ \text{k}\Omega}{\sqrt{1 + (2\pi \times 1.6\ \text{GHz} \times 100\ \text{k}\Omega \times 1\ \text{pF})^2}} \approx 100\ \Omega \text{ at 1.6 GHz}$$

At 10 GHz, a 1 pF probe input has $|Z_{probe}| = 1/(2\pi \times 10 \times 10^9 \times 1 \times 10^{-12}) \approx 16\ \Omega$. The probe loads the circuit comparably to a 16 $\Omega$ resistor at 10 GHz — this is significant even for a "high-impedance" active probe.

**Practical implication:** For signals above 5 GHz, even active probes should be considered to have meaningful loading effects. The safest practice for characterising GHz-range signals is to design dedicated 50 $\Omega$ SMA launch pads into the PCB and use a VNA or sampling oscilloscope with a matched connector.

---

## Tier 3: Advanced

### Q7. A colleague measures a 10 Gbps NRZ signal with a 16 GHz active probe and 33 GHz oscilloscope and reports a measured rise time of 40 ps. They claim the true rise time is approximately the same. Evaluate this claim rigorously.

**Answer:**

**Step 1 — Calculate system bandwidth:**

Assume the probe and oscilloscope have Gaussian roll-offs:

$$BW_{system} = \frac{1}{\sqrt{(1/16\ \text{GHz})^2 + (1/33\ \text{GHz})^2}}$$

$$BW_{system} = \frac{1}{\sqrt{3.906 \times 10^{-18} + 0.918 \times 10^{-18}}} = \frac{1}{\sqrt{4.824 \times 10^{-18}}} \approx 14.4\ \text{GHz}$$

**Step 2 — Calculate system rise time contribution:**

$$t_{r,system} = \frac{0.35}{14.4\ \text{GHz}} \approx 24\ \text{ps}$$

**Step 3 — Extract true signal rise time:**

$$t_{r,signal} = \sqrt{t_{r,meas}^2 - t_{r,system}^2} = \sqrt{40^2 - 24^2} = \sqrt{1600 - 576} = \sqrt{1024} = 32\ \text{ps}$$

**Conclusion:** The claim is wrong. The system rise time contributes 24 ps, so the true signal rise time is approximately 32 ps, not 40 ps — a 20% error. The measured value of 40 ps significantly overstates the true transition time.

**Is 32 ps physically reasonable for a 10 Gbps transmitter?**

A 10 Gbps NRZ transmitter with a nominal UI of 100 ps and 20–80% rise time specification of 20–30% UI would have $t_r \approx 20$–30 ps. A measured 32 ps (after deconvolution) is plausible for a slower driver or a channel with some bandwidth limiting.

**Additional concern:** The 16 GHz probe bandwidth means the probe passes only:

$$BW_{probe}/f_{signal,fundamental} = 16\ \text{GHz}/5\ \text{GHz} = 3.2\times$$

This is right at the minimum 3x rule for reasonable waveform fidelity. The probe is attenuating the 3rd harmonic (15 GHz) by:

$$A_{3H} = \frac{1}{\sqrt{1 + (15/16)^2}} = \frac{1}{\sqrt{1.879}} \approx 0.73\ (-2.7\ \text{dB})$$

The 3rd harmonic is suppressed by $-2.7$ dB, which removes some of the "squareness" from the measured eye. The recommended practice for a 10 Gbps signal is a 25–30 GHz probe, not 16 GHz.

---

### Q8. During PCB bring-up, you observe unexpected ringing on a 3.3 V CMOS output measured with a 10x passive probe. Describe a systematic approach to determine whether the ringing is real or a measurement artefact.

**Answer:**

This is one of the most common diagnostic challenges in lab work. The approach is:

**Step 1 — Characterise the measurement system.**

Measure the probe + scope bandwidth combination. For a typical 500 MHz 10x passive probe on a 1 GHz oscilloscope: $BW_{system} \approx 447$ MHz, $t_{r,system} \approx 783$ ps.

At 3.3 V CMOS switching speeds (typically $t_r \approx 2$–5 ns), the 447 MHz system bandwidth is adequate to resolve the signal's fundamental transitions without adding distortion. This means ringing is not automatically attributable to the probe.

**Step 2 — Change the ground lead length.**

The principal artefact source in passive probing is the ground lead's inductive resonance. Short the ground lead from 10 cm to 2 cm (using a spring-tip ground spring or coaxial barrel). If the ringing amplitude decreases significantly or the ringing frequency increases, the ringing was dominated by ground lead inductance and is a measurement artefact.

Ground lead resonant frequency: $f_{res} = 1/(2\pi\sqrt{LC})$. Shortening the lead from 10 cm ($L \approx 100$ nH) to 2 cm ($L \approx 20$ nH) shifts $f_{res}$ from $\approx 159$ MHz to $\approx 355$ MHz and reduces the ringing amplitude proportionally.

**Step 3 — Move the probe to a different point on the same net.**

If the ringing is real, it will appear at every point on the net (with different amplitude depending on impedance). If it disappears or changes dramatically when probing just a few millimetres away, it is a probe-to-PCB interaction artefact.

**Step 4 — Use a different probe type.**

Replace the passive probe with a high-impedance active probe (e.g., 1 pF input capacitance). Reduce the capacitive loading by 10x. If the ringing disappears or changes significantly, the passive probe's 10–15 pF input capacitance was causing a resonance with the source impedance or line inductance — an artefact.

**Step 5 — Simulate the measurement.**

Model the driver's output impedance (typically 10–50 $\Omega$ for CMOS), the PCB trace (microstrip inductance and capacitance), and the passive probe ($R_{tip}$, $C_{tip}$, $L_{ground}$) in a SPICE simulator. If the simulation with the probe model matches the measured waveform, and the simulation without the probe shows clean transitions, the ringing is a measurement artefact. If the simulation without the probe also shows ringing, the ringing is real (caused by trace reflections or load capacitance).

**Step 6 — Check the PCB layout.**

If Steps 2–5 confirm the ringing is real, investigate:
- Unterminated transmission line: the CMOS output is driving a trace longer than $t_r/(2 \times t_{pd})$, causing reflections
- Missing or wrong termination resistor
- Via stubs causing resonance
- Coupling from an adjacent switching signal

**Decision matrix:**

| Observation | Likely conclusion |
|---|---|
| Ringing disappears with shorter ground lead | Ground lead inductance artefact |
| Ringing frequency increases with shorter ground lead | Ground lead resonance (artefact) |
| Ringing unchanged with active probe substitution | Real signal ringing |
| Ringing only appears during probe contact | Probe loading artefact |
| Ringing at specific frequency matching trace round-trip delay | Real reflection from unterminated line |

---

### Q9. Describe the application of the oscilloscope's built-in de-embedding feature. What mathematical operations does it perform, and when would you trust (or distrust) its results?

**Answer:**

Modern high-end oscilloscopes (Keysight Infiniium UXR, Tektronix DPO70000SX) include software de-embedding that corrects for probe and cable frequency response in real time. The operation is:

**Mathematical operation:**

1. Load a characterisation file (S1P or S2P) describing the probe's or fixture's frequency response $H(f)$ over the bandwidth of interest.

2. The oscilloscope computes $H^{-1}(f)$ — the inverse frequency response (equalisation filter).

3. A finite-impulse response (FIR) filter with coefficients derived from $H^{-1}(f)$ is applied to every captured waveform in real time:

$$V_{de-embedded}(t) = V_{measured}(t) \ast h^{-1}(t)$$

where $h^{-1}(t)$ is the inverse impulse response.

4. For S2P fixture de-embedding (e.g., removing a PCB launch pad), the oscilloscope applies the equivalent of T-matrix inversion, extracting the signal as it would appear before the fixture.

**When to trust de-embedding results:**

- The probe's S-parameters were measured under the same conditions (bias, temperature, orientation) as the actual measurement
- The de-embedding bandwidth is within the characterisation range — do not extrapolate
- The probe's magnitude response is $> -20$ dB at all frequencies being de-embedded; inverting a severely attenuated response amplifies noise
- The FIR filter length is sufficient to represent the frequency span; verify the filter's group delay does not exceed the capture window

**When to distrust de-embedding:**

1. **Deep notches in the probe S-parameters:** If the probe has a notch (e.g., from a resonance) at $f_0$, $H(f_0) \approx 0$ and $H^{-1}(f_0) \to \infty$. The de-embedding filter amplifies noise by an enormous factor at $f_0$. The result near that frequency is noise, not signal.

2. **Probe loading changes the source signal:** De-embedding mathematically removes the probe's filter response, but it cannot recover the change in the source signal caused by the probe's loading impedance. If the probe's 10 pF loads a 500 $\Omega$ source, the source voltage is reduced. De-embedding cannot restore the missing voltage — it only removes the probe's passband roll-off.

3. **Mismatch between characterisation conditions and use conditions:** If the probe S-parameters were measured in a 50 $\Omega$ coaxial fixture but the probe is used on a PCB trace with a 1 mm spring-tip connection, the reflection environment is different and the characterised S1P does not match actual behaviour.

4. **Non-linear behaviour in the probe amplifier:** Active probe amplifiers can compress for large signals. No frequency-domain de-embedding corrects for non-linear distortion. Verify the probe is operating in its linear range before applying de-embedding.

**Best practice:** After de-embedding, always perform a sanity check. Measure a calibrated signal (e.g., a 10 GHz sine wave from a signal generator at a known power level) and verify the de-embedded result matches the expected amplitude to within ±0.5 dB. If not, the characterisation file or the de-embedding process has an error.

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| Probe bandwidth | Frequency at which probe attenuation reaches $-3$ dB |
| Tip capacitance | Capacitance at the probe tip that loads high-impedance nodes; limits AC bandwidth |
| Ground lead inductance | Inductance of the probe ground lead; forms resonant circuit with tip capacitance |
| CMRR | Common Mode Rejection Ratio — ratio of differential gain to common-mode gain; expressed in dB |
| 10x passive probe | Standard resistive divider probe with 10 M$\Omega$ || 10–15 pF loading |
| Active probe | FET-input amplifier probe with $< 1$ pF loading; powered by oscilloscope probe supply |
| Differential probe | Two-input probe measuring $V_+ - V_-$ with high CMRR; preferred for differential signal measurement |
| 50 $\Omega$ probe | Terminated probe matched to 50 $\Omega$ systems; flat frequency response but loads circuit |
| System rise time | Combined rise time of scope + probe: $t_{r,sys} = 0.35/BW_{sys}$ |
| De-embedding | Software removal of probe/cable frequency response from a captured waveform |
| Power rail probe | Specialised probe for measuring millivolt ripple on DC rails with large offset voltages |
| AC coupling | Internal capacitor blocks DC to allow high-sensitivity AC ripple measurement; limits low-frequency bandwidth |
