# Reflections and Matching

## Overview

Signal reflections are the dominant cause of waveform distortion in high-speed digital systems. Any impedance discontinuity — a stub, a via, a connector, a package transition, or a mismatched termination — sends a reflected wave back toward the source and a transmitted wave forward to the load. Understanding how to calculate, predict, and prevent reflections is central to high-speed PCB design and a core topic in every SI engineering interview.

This document covers the reflection coefficient, lattice (bounce) diagrams, and all practical termination strategies used in production designs: source termination, parallel (DC) termination, AC termination, and Thevenin termination.

---

## Tier 1: Fundamentals

### Q1. What is the reflection coefficient, and how is it derived?

**Answer:**

The reflection coefficient $\Gamma$ at a discontinuity is the ratio of the reflected voltage wave amplitude to the incident voltage wave amplitude at that point.

**Derivation at a load:**

At the load end of a transmission line with characteristic impedance $Z_0$ terminated in load impedance $Z_L$, the boundary condition requires that the total voltage and current are continuous:

$$V_{total} = V^+ + V^- \quad \text{(superposition of forward and reflected waves)}$$
$$I_{total} = I^+ - I^- = \frac{V^+}{Z_0} - \frac{V^-}{Z_0}$$

Ohm's law at the load:

$$Z_L = \frac{V_{total}}{I_{total}} = \frac{V^+ + V^-}{\frac{V^+ - V^-}{Z_0}} = Z_0 \cdot \frac{V^+ + V^-}{V^+ - V^-}$$

Solving for $V^-/V^+$:

$$Z_L (V^+ - V^-) = Z_0 (V^+ + V^-)$$

$$V^-(Z_L + Z_0) = V^+(Z_L - Z_0)$$

$$\boxed{\Gamma_L = \frac{V^-}{V^+} = \frac{Z_L - Z_0}{Z_L + Z_0}}$$

**Special cases:**

| Load condition | $Z_L$ | $\Gamma_L$ | Physical result |
|---|---|---|---|
| Matched | $Z_L = Z_0$ | 0 | No reflection |
| Open circuit | $Z_L = \infty$ | $+1$ | Full reflection, same polarity |
| Short circuit | $Z_L = 0$ | $-1$ | Full reflection, inverted polarity |
| Resistive (high) | $Z_L > Z_0$ | $0 < \Gamma < +1$ | Partial positive reflection |
| Resistive (low) | $Z_L < Z_0$ | $-1 < \Gamma < 0$ | Partial negative reflection |
| Capacitive | $Z_L = 1/j\omega C$ | Frequency-dependent | Step: negative then recovering |
| Inductive | $Z_L = j\omega L$ | Frequency-dependent | Step: positive then decaying |

**Source reflection coefficient:**

At the source end, the reflection coefficient is calculated with respect to the source impedance $Z_S$ (Thevenin equivalent of the driver):

$$\Gamma_S = \frac{Z_S - Z_0}{Z_S + Z_0}$$

---

### Q2. What is a lattice diagram (also called a bounce diagram)? Draw the structure and explain how it predicts waveforms at arbitrary points on a transmission line.

**Answer:**

A lattice diagram (Bergeron diagram or bounce diagram) is a graphical tool for tracking voltage and current waves as they reflect back and forth on a transmission line. It accounts for every reflection event in chronological order.

**Structure:**

```
Source (z=0)                          Load (z=L)
    |                                      |
t=0 |------- V1 = V_inc ----------------->|
    |                                      |
 TD |<------- V2 = Γ_L · V1 -------------|
    |                                      |
2TD |------- V3 = Γ_S · V2 ------------>|
    |                                      |
3TD |<------- V4 = Γ_L · V3 -------------|
    |
(time increases downward)
```

Where $TD$ is the one-way propagation delay of the line.

**Algorithm:**

1. The driver launches an initial wave $V_1 = V_{source} \cdot Z_0 / (Z_S + Z_0)$ (voltage divider between $Z_S$ and $Z_0$).
2. $V_1$ arrives at the load after time $TD$. The reflected wave is $V_2 = \Gamma_L \cdot V_1$.
3. $V_2$ arrives at the source after another $TD$. The re-reflected wave is $V_3 = \Gamma_S \cdot V_2$.
4. Continue until the waves become negligible (system settles to DC steady state).

**Voltage at any node:**

The voltage at a point is the superposition of all waves that have arrived at that point. At the load:

$$V_{load}(t) = V_1 + V_2 + V_4 + V_6 + \ldots \quad \text{(only forward-arriving waves at the load)}$$

At steady state, the voltage at every point settles to the DC value determined by the resistive divider: $V_{dc} = V_{source} \cdot Z_L / (Z_S + Z_L)$.

**Example — high-impedance CMOS load, low-impedance driver:**

$Z_0 = 50\ \Omega$, $Z_S = 10\ \Omega$, $Z_L = 1\ \text{k}\Omega$, $V_{source} = 3.3$ V step.

$$V_1 = 3.3 \times \frac{50}{10 + 50} = 2.75\ \text{V}$$

$$\Gamma_L = \frac{1000 - 50}{1000 + 50} = 0.905$$, so $V_2 = 0.905 \times 2.75 = 2.49$ V

At load after 1TD: $V = V_1 + V_2 = 5.24$ V (overshoot!)

$$\Gamma_S = \frac{10 - 50}{10 + 50} = -0.667$$, so $V_3 = -0.667 \times 2.49 = -1.66$ V

At source after 2TD: the re-reflected wave reduces the overshoot. The system oscillates and eventually settles to $3.3 \times 1000/1010 \approx 3.27$ V.

This illustrates why unterminated CMOS inputs with low-impedance CMOS drivers are prone to severe ringing.

---

### Q3. What is source termination? When does it work correctly, and what are its limitations?

**Answer:**

**Definition:**

Source termination (also called series termination) places a resistor $R_S$ in series with the driver output such that the total source impedance equals the line characteristic impedance:

$$R_S + Z_{driver} = Z_0$$

where $Z_{driver}$ is the output impedance of the driver IC.

**How it works:**

With source termination, $\Gamma_S = 0$. The driver launches a wave of amplitude $V_1 = V_{supply}/2$ (since the line and the series resistor form a 1:1 divider). This wave travels to the load, reflects with $\Gamma_L \approx +1$ (high-impedance CMOS load), and the reflected wave $V_2 = V_1$ returns to the source. At the load, the voltage steps to $V_1 + V_2 = V_{supply}$ — the correct final value.

When the reflected wave reaches the source, it encounters $\Gamma_S = 0$ and is absorbed. No further reflections occur. The waveform at the load is a clean single step to the final value after exactly one $TD$.

**Condition for clean operation:**

The load must be high impedance ($\Gamma_L \approx +1$). If intermediate loads (other receivers tapped onto the line) are present, they see only $V_{supply}/2$ until the reflected wave returns from the far end. This takes $2 \times TD$ from the source or equivalently one round-trip time. During this time, intermediate loads may not see a valid logic level — a condition called "initial undershoot" or "initial half-voltage."

**Limitations:**

1. **Only correct for single far-end load (point-to-point topology).** Branched or multi-drop topologies cause the midpoint loads to see only half the final voltage for one round trip — not suitable for synchronous buses.

2. **Adds series resistance in signal path.** The RC time constant with the line capacitance and any stubs can slow the rise time slightly. For long lines or high-capacitance loads, a well-calculated $R_S$ value is essential.

3. **Driver output impedance varies.** CMOS driver output impedance varies with supply voltage, temperature, and process corner. The resistor must be chosen to match the nominal impedance; worst-case mismatch causes residual reflections.

4. **DC current:** Series termination does not provide a DC load path. The line floats to the logic level, which is usually desirable for CMOS (minimises static power).

---

### Q4. Explain parallel (DC) termination and Thevenin termination. Compare their power consumption and noise immunity.

**Answer:**

**Parallel (DC) termination:**

A single resistor $R_T = Z_0$ is connected from the signal line to a voltage rail at the load end. The reference voltage is typically ground (GND) or a positive supply ($V_{TT}$).

- If terminated to GND: $R_T$ to GND provides a matched impedance but pulls the signal low. The driver must source current $V_{high}/Z_0$ continuously to maintain a high level. Static power = $V_{high}^2 / Z_0$. For a 3.3 V, 50 $\Omega$ system: $P = 3.3^2/50 = 218$ mW per line — impractical for wide buses.

- If terminated to $V_{TT} = V_{supply}/2$: Zero static current when line is at $V_{TT}$, but the driver must sink/source current for high and low levels respectively. Power is lower on average but $V_{TT}$ requires an additional supply.

**Thevenin termination:**

Two resistors ($R_1$ from supply, $R_2$ to GND) form a voltage divider that presents the characteristic impedance $Z_0$ and a mid-supply bias voltage $V_{TT} = V_{supply} \times R_2/(R_1 + R_2)$.

Matching condition: $R_1 \| R_2 = Z_0$, so:

$$\frac{R_1 R_2}{R_1 + R_2} = Z_0$$

For a $V_{TT} = V_{supply}/2$ bias with $Z_0 = 50\ \Omega$:

$$R_1 = R_2 = 100\ \Omega$$

$$R_1 \| R_2 = 50\ \Omega \quad \checkmark$$

**Comparison table:**

| Property | Series (source) | Parallel to GND | Parallel to $V_{TT}$ | Thevenin |
|---|---|---|---|---|
| Termination point | Source end | Load end | Load end | Load end |
| Static power (high state) | None | $V_{high}^2 / Z_0$ | $(V_{high}-V_{TT})^2/Z_0$ | $V_{supply}^2 / (R_1+R_2)$ |
| Additional supply | No | No (or $V_{TT}$) | Yes ($V_{TT}$) | No |
| Multi-drop support | Poor | Yes | Yes | Yes |
| Signal integrity | Half-voltage mid-line | Clean step everywhere | Clean step everywhere | Clean step everywhere |
| Noise immunity | High (no DC bias) | Good | Good | Good |
| Component count | 1 resistor | 1 resistor | 1 resistor + $V_{TT}$ reg | 2 resistors |

**Practical guidance:** Thevenin termination is frequently used for DDR memory and other parallel buses because it provides load-end termination (clean signal everywhere on the net) without requiring a dedicated $V_{TT}$ supply at the terminator, though it dissipates more power than a single parallel termination to $V_{TT}$.

---

### Q5. What is AC termination? What problem does it solve compared to DC parallel termination?

**Answer:**

**AC termination** places a resistor in series with a capacitor from the signal line to GND at the load end:

$$Z_T(j\omega) = R_T + \frac{1}{j\omega C}$$

At high frequencies (above the AC termination corner frequency $f_c = 1/(2\pi R_T C)$), the capacitor is a short circuit and $Z_T \approx R_T = Z_0$ — the line sees a matched termination. At DC and low frequencies, the capacitor is open and the line sees no DC load.

**Problem it solves:**

DC parallel termination to GND forces the driver to source $V_{high}/Z_0$ of current continuously when the line is at logic high. For a 3.3 V, 50 $\Omega$ terminator, this is 66 mA per line. On a 32-bit bus, this is over 2 A of static current — completely impractical.

AC termination eliminates this static current. The capacitor blocks DC while providing a matched termination at signal frequencies.

**Design of AC termination:**

The corner frequency must be low enough that the terminator is effective at the signal's fundamental frequency:

$$f_c = \frac{1}{2\pi R_T C} \ll f_{signal}$$

For a 100 MHz signal with $R_T = 50\ \Omega$:

$$C \gg \frac{1}{2\pi \times 50 \times 10^8} = 32\ \text{pF}$$

Typical values: $C = 100$–$470$ pF. The RC time constant must also be long enough that the capacitor does not discharge significantly during a single bit period — the capacitor must act as a charge reservoir rather than a reactive filter.

**Limitation:** If the signal has significant low-frequency content (long runs of the same bit, as in DC-balanced codes), the capacitor voltage tracks the average signal voltage and the termination becomes less effective for slow transitions. This is why AC termination is preferred for AC-coupled or 8b/10b encoded interfaces rather than for DC-balanced buses.

---

## Tier 2: Intermediate

### Q6. A 50 $\Omega$ transmission line is driven by a source with $Z_S = 10\ \Omega$ and terminated with $Z_L = 50\ \Omega$. The propagation delay is 1 ns. The source switches from 0 V to 1 V at $t = 0$. Construct the lattice diagram and determine the voltage at the load as a function of time.

**Answer:**

**Reflection coefficients:**

$$\Gamma_S = \frac{Z_S - Z_0}{Z_S + Z_0} = \frac{10 - 50}{10 + 50} = \frac{-40}{60} = -0.667$$

$$\Gamma_L = \frac{Z_L - Z_0}{Z_L + Z_0} = \frac{50 - 50}{50 + 50} = 0$$

Since $\Gamma_L = 0$, there are no reflections from the load. The load is perfectly matched.

**Initial launched wave:**

$$V_1 = V_{source} \cdot \frac{Z_0}{Z_S + Z_0} = 1.0 \times \frac{50}{60} = 0.833\ \text{V}$$

**Lattice diagram:**

```
t=0:    V1 = +0.833 V launched toward load
t=1 ns: V1 arrives at load. V_load = 0.833 V. Γ_L = 0, no reflection.
```

The system settles immediately. The DC steady state is:

$$V_{dc} = V_{source} \cdot \frac{Z_L}{Z_S + Z_L} = 1.0 \times \frac{50}{60} = 0.833\ \text{V}$$

**Result:** $V_{load} = 0$ for $0 \le t < 1$ ns, then steps to $0.833$ V for $t \ge 1$ ns. No ringing or overshoot. This is the ideal matched-load case — the transmission line simply delays the signal by its propagation delay.

**Note on the voltage:** The final load voltage is $50/(10+50) \times 1\ \text{V} = 0.833$ V, not 1.0 V, because the source resistance forms a resistive divider. If the specification requires 1 V at the load with a 10 $\Omega$ source, the source voltage must be set to $V_{source} = 1.0 \times 60/50 = 1.2$ V, or the source impedance must be reduced.

---

### Q7. Compare and contrast the waveforms produced by series termination versus parallel termination on a point-to-point 50 $\Omega$ line with a high-impedance CMOS load. Include mid-line waveforms in your analysis.

**Answer:**

**Setup:** $Z_0 = 50\ \Omega$, $V_{supply} = 3.3$ V, driver output impedance $Z_{driver} = 7\ \Omega$, load is CMOS ($Z_L = 10\ \text{k}\Omega \approx \infty$).

**Series termination:** $R_S = 50 - 7 = 43\ \Omega$ placed in series at the source.

Total source impedance: $7 + 43 = 50\ \Omega$. $\Gamma_S = 0$.

Initial wave: $V_1 = 3.3 \times 50/100 = 1.65$ V.

- Source voltage: steps immediately to 1.65 V at $t = 0$
- Mid-line: sees 1.65 V at $t = TD/2$ (half the propagation delay)
- Load: sees 1.65 V at $t = TD$, then the reflected wave $V_2 = \Gamma_L \times 1.65 \approx 1.65$ V arrives simultaneously and adds, making $V_{load} = 3.3$ V at $t = TD$
- Mid-line: sees reflected wave at $t = 3TD/2$, voltage steps from 1.65 V to 3.3 V
- Source: $\Gamma_S = 0$, reflected wave absorbed at $t = 2TD$, no further events

Mid-line waveform for series termination:
```
V
3.3 |                        ____
1.65|         ____
0   |__________________
    0    TD/2  TD   3TD/2  2TD
```

The mid-line sees only half the final voltage from $t = TD/2$ until $t = 3TD/2$ — a total of one full round-trip delay of ambiguous voltage. This is the "initial undershoot" problem for intermediate loads.

**Parallel termination:** $R_T = 50\ \Omega$ to GND at the load end, no series resistor.

Source impedance is still $Z_S = 7\ \Omega$. $\Gamma_S = (7-50)/(7+50) = -0.754$. $\Gamma_L = 0$ (matched load).

Initial wave: $V_1 = 3.3 \times 50/(7+50) = 2.89$ V.

- Load: sees 2.89 V at $t = TD$. Since $\Gamma_L = 0$, no reflection. Final value = 2.89 V? No — the DC steady state is $3.3 \times 50/(7+50) = 2.89$ V, which is the correct final value.
- Mid-line: sees 2.89 V at $t = TD/2$ and remains there (no reflection to observe after $TD$)

Mid-line waveform for parallel termination:
```
V
3.3 |
2.89|         ____________________
0   |__________
    0    TD/2  TD
```

Every point on the line settles to the correct voltage in one direction of travel — no round-trip wait required.

**Summary of waveform comparison:**

| Property | Series termination | Parallel termination |
|---|---|---|
| Source overshoot | None | Ringing if $\Gamma_S \neq 0$ |
| Load overshoot | None | None (matched) |
| Mid-line during transition | Half voltage for 1 round trip | Full voltage after 1 $TD$ |
| Multi-drop suitability | Poor | Good |
| Static power | Zero | $V_{high}^2/Z_0$ |
| Number of components | 1 per signal | 1 per signal |

---

### Q8. A signal with a rise time of 200 ps is launched onto a 50 $\Omega$ line. At the midpoint of the line there is a 1 pF capacitive stub. Calculate the reflection coefficient produced by the stub and describe the waveform distortion.

**Answer:**

**Reflection from a shunt capacitor:**

A shunt capacitance $C_{stub}$ at a point on a $Z_0 = 50\ \Omega$ line represents a localised impedance discontinuity. In the time domain, the reflected voltage waveform from a shunt capacitor is:

$$v_{reflected}(t) = -\frac{Z_0}{2} C_{stub} \frac{dv_{incident}}{dt}$$

This is a differentiator: the reflection is proportional to the derivative of the incident waveform.

**Magnitude estimate:**

For a ramp with slope $dV/dt = V_{swing}/t_r = 1.0/200\times10^{-12} = 5 \times 10^9$ V/s:

$$V_{reflected,peak} = -\frac{50}{2} \times 1\times10^{-12} \times 5\times10^9 = -0.125\ \text{V}$$

For a 1.0 V swing, this is a $-12.5$% reflection — significant.

**In the frequency domain** (valid when the stub is electrically small):

The impedance of the shunt capacitor is $Z_{stub} = 1/(j\omega C)$. The reflection coefficient is:

$$\Gamma(\omega) = \frac{Z_{stub} \| Z_0 - Z_0}{Z_{stub} \| Z_0 + Z_0}$$

For $Z_{stub} \gg Z_0$ (low frequency):

$$Z_{stub} \| Z_0 \approx Z_0 \left(1 - j\omega Z_0 C\right)$$

$$\Gamma \approx \frac{-j\omega Z_0 C / 2}{1} = -\frac{j\omega Z_0 C}{2}$$

$$|\Gamma| = \frac{\omega Z_0 C}{2} = \frac{2\pi f \times 50 \times 1\times10^{-12}}{2} = \pi f \times 50 \times 10^{-12}$$

At $f = 1$ GHz (corresponding to a 200 ps rise time): $|\Gamma| = \pi \times 10^9 \times 50 \times 10^{-12} = 0.157$ (-16 dB) — a substantial reflection.

**Waveform effect:**

The transmitted signal experiences a local low-pass filtering effect (the capacitor slows the edge). The reflected wave is a negative-going pulse coinciding with the incident edge. Observers upstream see a small negative glitch. The transmitted waveform has its rise time degraded by approximately:

$$\Delta t_r \approx 2.2 \times Z_0 C_{stub} = 2.2 \times 50 \times 10^{-12} = 110\ \text{ps}$$

This 110 ps degradation added in quadrature to the original 200 ps rise time gives:

$$t_{r,degraded} = \sqrt{200^2 + 110^2} \approx 228\ \text{ps}$$

**Design rule:** Via stub capacitances, test point pads, and connector pin capacitances all act as shunt capacitors. At 5+ Gb/s data rates, even 0.2 pF of parasitic capacitance produces measurable reflections. Via backdrilling removes the unused stub below the layer transition, reducing $C_{stub}$ by 50–80%.

---

## Tier 3: Advanced

### Q9. For a double-terminated transmission line (matched at both ends), derive the total transfer function from source to load and show why it is simply a delay.

**Answer:**

**Setup:** Source $V_S(s)$ in series with $Z_S = Z_0$, transmission line of characteristic impedance $Z_0$ and length $l$, terminated at the load by $Z_L = Z_0$.

**Transmission line ABCD matrix:**

For a lossless line of electrical length $\theta = \beta l$ (where $\beta = \omega/v_p$):

$$\begin{pmatrix} V_1 \\ I_1 \end{pmatrix} = \begin{pmatrix} \cos\theta & jZ_0\sin\theta \\ j\sin\theta/Z_0 & \cos\theta \end{pmatrix} \begin{pmatrix} V_2 \\ I_2 \end{pmatrix}$$

**Transfer function derivation:**

With source impedance $Z_S = Z_0$ and load impedance $Z_L = Z_0$, and using the voltage wave analysis:

$\Gamma_S = 0$ and $\Gamma_L = 0$. The wave launched is:

$$V^+(s) = V_S(s) \cdot \frac{Z_0}{Z_S + Z_0} = \frac{V_S(s)}{2}$$

At the load, there is no reflection, so:

$$V_{load}(s) = V^+(s) \cdot e^{-\gamma l}$$

For a lossless line: $\gamma = j\beta = j\omega/v_p$, so $e^{-\gamma l} = e^{-j\omega l/v_p} = e^{-j\omega TD}$.

$$V_{load}(s) = \frac{V_S(s)}{2} e^{-sT_D}$$

where $s = j\omega$ and $T_D = l/v_p$ is the propagation delay.

**Transfer function:**

$$H(s) = \frac{V_{load}(s)}{V_S(s)} = \frac{1}{2} e^{-sT_D}$$

The factor of $1/2$ arises because the source resistance $Z_S = Z_0$ forms a voltage divider — half the source voltage is dropped across $Z_S$. The $e^{-sT_D}$ term is a pure delay.

**In the time domain:**

$$v_{load}(t) = \frac{1}{2} v_S(t - T_D)$$

The double-terminated transmission line is a perfect delay element — it delays the signal by $T_D$ with no waveform distortion, no overshoot, and no ringing. The factor of $1/2$ is the matched-termination insertion loss.

**Physical interpretation:** When $\Gamma_S = \Gamma_L = 0$, the launched wave travels from source to load, arrives with amplitude $V_S/2$, and is fully absorbed. The line stores no energy after the transient has passed.

---

### Q10. You are debugging a 10 Gb/s SerDes channel and observe periodic resonance spikes in the insertion loss $|S_{21}|$ every 5 GHz. The channel contains two connectors in series with flying leads estimated at 5 cm each. Explain the cause of the resonance and calculate the expected resonant frequency. What termination strategy would you use to damp the resonance?

**Answer:**

**Root cause — unterminated stub resonance:**

Flying leads (pigtail wires from connector to PCB pad) are short, partially unterminated transmission lines. If their ends are not matched, they form open-circuit stubs. An open-circuit stub of electrical length $\lambda/4$ appears as a short circuit at the driving point — creating a series null in $|S_{21}|$.

**Resonant frequency calculation:**

Electrical length $\theta = \lambda/4$ at resonance means the physical length equals one-quarter wavelength:

$$l = \frac{\lambda}{4} = \frac{v_p}{4 f_{res}}$$

For a flying lead (wire in air, $\varepsilon_{eff} \approx 1$, $v_p = c = 3 \times 10^8$ m/s):

$$f_{res} = \frac{v_p}{4l} = \frac{3 \times 10^8}{4 \times 0.05} = \frac{3 \times 10^8}{0.2} = 1.5\ \text{GHz}$$

But the observation is resonance spikes every 5 GHz. Two possibilities:

1. The resonances at multiples of $f_0 = 1.5$ GHz are present, but the first null appears at $3 f_0 = 4.5 \approx 5$ GHz (fundamental mode was suppressed or not measured; higher-order modes dominate).

2. Alternatively, re-examine the actual stub length. If $l = 1.5$ cm:

$$f_{res} = \frac{3 \times 10^8}{4 \times 0.015} = 5\ \text{GHz}$$

This is the most likely interpretation: the physically active stub length is approximately 1.5 cm (perhaps the connector housing reduces the effective flying lead to a pin stub of that length), and the $\lambda/4$ resonance is at 5 GHz.

**Why the spikes are periodic:** The stub resonates at $f_0, 3f_0, 5f_0, \ldots$ (odd harmonics of $\lambda/4$). Even harmonics are not null points (at even multiples, the stub presents an open circuit at its drive point, which is equivalent to a local open and passes signal normally). The 5 GHz, 15 GHz, 25 GHz series of spikes is therefore expected.

**Impact on 10 Gb/s channel:**

The fundamental data rate of 10 Gb/s (NRZ) has significant spectral content at $f_{Nyquist} = 5$ GHz. A notch at exactly 5 GHz collapses the eye opening and makes the link non-functional regardless of equalization depth.

**Termination strategies:**

1. **Reduce stub length (best option):** Replace flying leads with controlled-impedance flex cable or direct PCB-to-PCB press-fit connectors. Eliminate the air wire entirely. This removes the resonance rather than damping it.

2. **Ferrite or resistive loading:** A small resistor (10–22 $\Omega$) in series with the stub damps the $Q$ of the resonance, broadening and reducing the notch. The resistor adds insertion loss but replaces the sharp null with a gradual slope.

3. **Backdrilling (if on PCB):** If the stub is a via barrel below the layer connection point, backreaming removes the stub, eliminating the resonance.

4. **Equalization:** CTLE (Continuous-Time Linear Equalization) in the receiver can boost high-frequency loss, but a notch at the Nyquist frequency is typically too deep to equalize reliably. More than 6–8 dB notch depth exceeds practical CTLE range.

**Lesson:** Uncontrolled stub lengths are catastrophic at data rates above 5 Gb/s. Every mechanical interface (connector, cable termination, test point) must be treated as a potential $\lambda/4$ resonator at the signal's Nyquist frequency.

---

## Quick Reference: Reflections and Termination

| Quantity | Formula |
|---|---|
| Reflection coefficient (load) | $\Gamma_L = (Z_L - Z_0)/(Z_L + Z_0)$ |
| Reflection coefficient (source) | $\Gamma_S = (Z_S - Z_0)/(Z_S + Z_0)$ |
| Initial launched wave | $V_1 = V_S \cdot Z_0/(Z_S + Z_0)$ |
| Transmitted voltage at load | $V_T = V_{inc}(1 + \Gamma_L) = 2V_{inc} Z_L/(Z_L + Z_0)$ |
| Series termination value | $R_S = Z_0 - Z_{driver}$ |
| Thevenin termination | $R_1 \| R_2 = Z_0$, $V_{TT} = V_{supply} \cdot R_2/(R_1+R_2)$ |
| AC termination corner | $f_c = 1/(2\pi R_T C) \ll f_{signal}$ |
| Stub resonant frequency | $f_{res} = v_p / (4l)$ for $\lambda/4$ open stub |
| Capacitive stub reflection | $v_{refl}(t) = -(Z_0/2) C \cdot dv_{inc}/dt$ |

| Termination type | Load-end matched | Eliminates source reflections | No static power |
|---|---|---|---|
| Series (source) | No | Yes | Yes |
| Parallel (DC, to GND) | Yes | No | No |
| Parallel (DC, to $V_{TT}$) | Yes | No | Partial |
| Thevenin | Yes | No | No |
| AC (RC) | Yes (AC only) | No | Yes (AC only) |
