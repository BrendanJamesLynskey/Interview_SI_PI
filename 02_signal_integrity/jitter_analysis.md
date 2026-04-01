# Jitter Analysis

## Overview

Jitter is the deviation of a signal's transition edge from its ideal timing position. In high-speed serial links, jitter determines eye width, sets the BER floor, and defines compliance margins for transmitter and receiver testing. Every serial interface standard defines jitter budgets — allocating allowable jitter contributions to the transmitter, channel, and receiver. Understanding jitter decomposition (separating random from deterministic components), the Dual-Dirac model, and total jitter extrapolation to low BER is essential for both design and debug work.

---

## Tier 1: Fundamentals

### Q1. Define jitter and explain the difference between period jitter, cycle-to-cycle jitter, and time-interval error (TIE).

**Answer:**

Jitter is formally defined as the short-term variation of a digital signal's transition timing from its ideal reference position. "Short-term" distinguishes jitter from wander, which refers to long-term frequency drift below approximately 10 Hz.

**Three measurement definitions:**

**Time Interval Error (TIE) — also called Phase Jitter:**

TIE measures the absolute timing displacement of each edge from its ideal position:

$$TIE[n] = t_{actual}[n] - t_{ideal}[n]$$

where $t_{ideal}[n] = n \cdot T_{nom}$ for a nominal period $T_{nom}$. TIE accumulates: a steady-state frequency error appears as a linearly growing TIE. TIE is the most fundamental jitter measure and is used in the Dual-Dirac model and bathtub curve extrapolation.

**Period Jitter ($J_p$):**

The deviation of each clock period from the nominal period:

$$J_p[n] = T[n] - T_{nom} = TIE[n+1] - TIE[n]$$

Period jitter is the first difference of TIE. It removes low-frequency wander and DC phase offset, measuring only the cycle-to-cycle variation. Specification limits (e.g., PCIe REFCLK period jitter) use this measure.

**Cycle-to-Cycle Jitter ($J_{cc}$):**

The difference between successive periods:

$$J_{cc}[n] = T[n+1] - T[n] = J_p[n+1] - J_p[n]$$

This is the second difference of TIE. It is most sensitive to high-frequency jitter components and insensitive to low-frequency phase noise. Used less frequently in digital SI; more common in RF synthesiser characterisation.

**Relationship between definitions:**

| Measurement | Filtering effect | Primary use |
|---|---|---|
| TIE | No filtering (all jitter components) | BER bathtub, total jitter |
| Period jitter | High-pass: suppresses wander | Clock specification compliance |
| Cycle-to-cycle | Bandpass (high-pass twice) | High-frequency noise sensitivity |

**Common mistake:** Using period jitter as a proxy for TIE when extrapolating to BER. Period jitter suppresses the low-frequency jitter components that accumulate and dominate TIE at long time scales. This leads to optimistic BER estimates.

---

### Q2. Describe the jitter decomposition hierarchy. What are the main jitter components and how do they add?

**Answer:**

**Jitter decomposition tree:**

```
Total Jitter (TJ)
├── Deterministic Jitter (DJ)    ← bounded, non-Gaussian
│   ├── Data-Dependent Jitter (DDJ / ISI Jitter)
│   ├── Periodic Jitter (PJ)
│   ├── Duty Cycle Distortion (DCD)
│   └── Bounded Uncorrelated Jitter (BUJ)
└── Random Jitter (RJ)           ← Gaussian, unbounded
```

**Random Jitter (RJ):**

RJ arises from thermal noise (Johnson noise), shot noise, and flicker noise in oscillators and PLLs. Its probability density function is Gaussian with zero mean and standard deviation $\sigma_{RJ}$. Because it is Gaussian and unbounded, it contributes to the Gaussian tails of the total jitter distribution and sets the slope of the BER bathtub walls. RJ is characterised by $\sigma_{RJ}$ only.

**Data-Dependent Jitter (DDJ / ISI Jitter):**

DDJ is timing variation correlated with the preceding data pattern. The mechanism is identical to voltage ISI: frequency-dependent channel loss causes the edge crossing time to depend on whether previous bits were the same or different polarity. For example, after a long run of identical bits, the DC level shifts and the first transition crosses the threshold at a different time than a transition following alternating bits.

DDJ is bounded — only a finite number of prior bits can influence the current edge timing given the channel's finite impulse response length. Its PDF is multi-modal.

**Periodic Jitter (PJ):**

Sinusoidal jitter from a coherent interference source. Common sources: power supply switching noise (SMPS at 1–5 MHz), crosstalk from a synchronous aggressor clock, spread-spectrum modulation components. PJ has a bimodal PDF (arcsin distribution). It is characterised by its peak-to-peak amplitude $PJ_{pp}$.

**Duty Cycle Distortion (DCD):**

Asymmetry between the rise time and fall time causes the 0→1 and 1→0 transitions to cross the threshold at slightly different times. DCD is a constant offset: even transitions are consistently early or late relative to odd transitions. It appears as a bimodal distribution with two peaks separated by the DCD magnitude.

**Bounded Uncorrelated Jitter (BUJ):**

Jitter that is bounded in amplitude but not correlated with the data pattern. Crosstalk from asynchronous aggressors is a common source. BUJ has a bounded but non-Gaussian distribution.

**Total jitter combination:**

For the Dual-Dirac model (standard approximation), the bounded jitter components (DDJ, PJ, DCD) are combined into a single DJ peak-to-peak value:

$$DJ_{pp} = DDJ_{pp} + PJ_{pp} + DCD_{pp}$$

(This is conservative — peak-to-peak addition assumes worst-case simultaneous occurrence of all DJ maxima, which is not always physically realistic.)

Total jitter at a target BER:

$$TJ_{pp}(BER) = DJ_{pp} + 2\alpha(BER) \cdot \sigma_{RJ}$$

where $\alpha(BER) = \sqrt{2}\,\text{erfinv}(1 - BER)$. At BER $= 10^{-12}$: $\alpha \approx 7.03$.

$$TJ_{pp}(10^{-12}) = DJ_{pp} + 14.07 \cdot \sigma_{RJ}$$

---

### Q3. What is the Dual-Dirac jitter model? Explain its assumptions, derivation, and how it is used to extrapolate BER from measured data.

**Answer:**

**Model concept:**

The Dual-Dirac model represents the probability density function of deterministic jitter as two equal-weight delta (Dirac) functions located at $\pm DJ_{pp}/2$ around zero:

$$f_{DJ}(t) = \frac{1}{2}\delta\!\left(t - \frac{DJ_{pp}}{2}\right) + \frac{1}{2}\delta\!\left(t + \frac{DJ_{pp}}{2}\right)$$

The random jitter is represented as a Gaussian with standard deviation $\sigma_{RJ}$:

$$f_{RJ}(t) = \frac{1}{\sigma_{RJ}\sqrt{2\pi}} e^{-t^2/(2\sigma_{RJ}^2)}$$

The total jitter PDF is the convolution:

$$f_{TJ}(t) = f_{DJ}(t) * f_{RJ}(t) = \frac{1}{2}g(t - DJ_{pp}/2, \sigma_{RJ}) + \frac{1}{2}g(t + DJ_{pp}/2, \sigma_{RJ})$$

where $g(t, \sigma)$ is a Gaussian PDF. The total jitter PDF is therefore the sum of two equal-weight Gaussians, one centred at $+DJ_{pp}/2$ and one at $-DJ_{pp}/2$.

**BER formula:**

The BER at a sampling phase offset $\Delta t$ from the eye centre (measured as fraction of one UI) is the probability that a jitter excursion exceeds $\pm(EW/2 - |\Delta t|)$, where $EW$ is the maximum eye width:

$$BER(\Delta t) = \frac{1}{4}\left[\text{erfc}\!\left(\frac{UI/2 - \Delta t - DJ_{pp}/2}{\sigma_{RJ}\sqrt{2}}\right) + \text{erfc}\!\left(\frac{UI/2 + \Delta t - DJ_{pp}/2}{\sigma_{RJ}\sqrt{2}}\right)\right]$$

This produces the characteristic bathtub shape. Each wall is a Gaussian tail; the two walls are offset by $DJ_{pp}$ from the ideal $\pm UI/2$ positions.

**Extracting $DJ_{pp}$ and $\sigma_{RJ}$ from measured data:**

1. Measure BER at several sampling phases on each bathtub wall (in the $10^{-4}$ to $10^{-8}$ range).
2. Convert BER to Q-factor: $Q = \sqrt{2}\,\text{erfinv}(1 - 2 \cdot BER)$.
3. On a Q-scale plot, each wall appears as a straight line. Fit a line to each wall.
4. The slope of each line is $1/\sigma_{RJ}$.
5. The x-intercept (phase where $Q = 0$, i.e., BER = 0.5) is the mean crossing time. The separation between the two intercepts is $DJ_{pp}$.
6. Extrapolate the fitted lines to the target Q (e.g., $Q = 7.03$ for BER $= 10^{-12}$) to find the eye width.

**Limitations of Dual-Dirac:**

- It assumes all DJ combines into two equal-amplitude peaks. Real DJ is multi-modal (multiple distinct ISI patterns create multiple crossing populations). The approximation is conservative for well-behaved channels.
- It assumes RJ is stationary Gaussian. Cyclostationary RJ (noise correlated with the data pattern) violates this assumption.
- For PAM4, the model requires extension: four signal levels create three eye openings, each with independent ISI and noise contributions.

---

## Tier 2: Intermediate

### Q4. Describe a BERT (Bit Error Ratio Tester) measurement setup for characterising jitter in a 10 Gbps serial link. What are the key parameters to configure?

**Answer:**

**BERT system components:**

A standard BERT system for 10 Gbps jitter characterisation includes:

1. **Pattern generator:** Produces the test data pattern (PRBS7, PRBS31, or 200UI compliance patterns) at the target baud rate. Output: differential CML, AC-coupled, into the DUT transmitter or directly to the measurement point.

2. **Error detector:** Receives the link output, applies CDR to recover the clock, and compares bit-by-bit to the expected pattern. Counts error bits.

3. **Phase injector (or sampling phase sweep):** The BERT's error detector includes a variable phase delay on its sampling clock, allowing BER to be measured at different phases across 0 to 1 UI — generating the bathtub curve.

4. **Reference clock source:** Ultra-low jitter reference clock ($\sigma_{RJ} < 100$ fs) fed to the BERT and DUT. A noisy reference clock dominates $\sigma_{RJ}$ and masks the DUT's own jitter.

5. **Channel under test:** PCB traces, connectors, cables, or a loopback through the DUT's SerDes.

**Key configuration parameters:**

| Parameter | Typical value / options | Effect on measurement |
|---|---|---|
| Test pattern | PRBS7, PRBS15, PRBS31 | PRBS31 stresses ISI most; PRBS7 runs faster |
| Baud rate | Must match DUT exactly | ±1 ppm clock tolerance required |
| CDR loop bandwidth | 10 MHz (PCIe), 4 MHz (Ethernet) | Wider BW rejects more phase noise; must match standard |
| Phase step size | 0.001–0.01 UI | Finer steps near walls for accurate extrapolation |
| Dwell time per phase | 10–100 seconds | Sufficient to accumulate $\ge 100$ errors for reliable BER |
| Reference clock | SSCG off or on | SSC must be on if testing SSC-enabled device |

**Bathtub curve generation procedure:**

1. Set phase to 0 (left edge of UI). Measure BER for 30 seconds. Record (phase, BER).
2. Increment phase by 0.01 UI. Repeat.
3. Continue across full UI (100 phase steps for 0.01 UI resolution).
4. Post-process: fit Dual-Dirac model to left and right walls. Extract $DJ_{pp}$ and $\sigma_{RJ}$. Extrapolate to target BER.

**Practical tip:** At BER $< 10^{-8}$, measurements near the bathtub floor are unreliable (very few errors in the dwell time). Rely on extrapolation from the wall region ($10^{-4}$ to $10^{-8}$) for the floor estimate.

---

### Q5. How does spread-spectrum clocking (SSC) affect jitter measurements and the eye diagram?

**Answer:**

**SSC operation:**

PCIe and SATA use spread-spectrum clocking to reduce electromagnetic emissions. The reference clock frequency is modulated at a low rate (31.5–33 kHz for PCIe) with a triangular or sinusoidal profile, spreading the clock spectrum across a band of approximately $\pm 0.5\%$ (PCIe "down-spread" is 0 to $-0.5\%$, avoiding frequency increase above nominal):

$$f_{clk}(t) = f_0\left[1 - 0.005 \cdot m(t)\right]$$

where $m(t)$ is the modulation waveform ($0 \le m \le 1$ for down-spread).

**Effect on TIE and eye diagram:**

The SSC modulation appears as a very low-frequency sinusoidal jitter component. At 33 kHz modulation rate, the peak TIE deviation is:

$$TIE_{peak} = \frac{\Delta f}{f_{mod}} \cdot \frac{1}{2\pi} \approx \frac{0.005 \times 2.5\ \text{GHz}}{33\ \text{kHz}} \cdot \frac{1}{2\pi} \approx 60\ \text{ns peak}$$

This 60 ns of TIE is enormous compared to a 400 ps UI at 2.5 GHz. However, if the receiver's CDR tracks the SSC modulation (which PCIe CDR must, per specification — CDR bandwidth covers the SSC rate), the CDR absorbs the SSC jitter and the effective TIE seen by the decision circuit is only the *residual* jitter above the CDR bandwidth.

**Effect on bathtub curves:**

A BERT measured without CDR tracking the SSC will show a massively wide bathtub from the uncorrected TIE. This is a measurement artefact. Correct procedure: configure the BERT's CDR to track at the same bandwidth specification as the DUT's CDR (typically 1.5–4 MHz for PCIe).

**Effect on jitter decomposition:**

SSC contributes a low-frequency periodic component to PJ. If the jitter spectrum analyser computes PJ in the 10 Hz–40 MHz band (as required for PCIe transmitter compliance), SSC at 33 kHz will show as a PJ spike. The PCIe specification's PJ measurement bandwidth filter (500 kHz to 1.5 MHz high-pass) excludes the SSC frequency, so SSC is not counted as a PJ violation in the TX compliance test. However, SSC must be excluded carefully in any manual jitter decomposition.

---

### Q6. Explain the jitter transfer function (JTF) of a PLL and how it relates to SSC tracking and jitter amplification.

**Answer:**

**PLL jitter transfer:**

A Phase-Locked Loop (PLL) — whether used as a CDR, clock multiplier, or de-jitter circuit — has a characteristic jitter transfer function (JTF) that describes how input jitter at different frequencies is processed:

$$JTF(f) = \frac{H_{PLL}(j2\pi f)}{1 + H_{PLL}(j2\pi f)} = \frac{\phi_{out}(f)}{\phi_{in}(f)}$$

For a second-order charge-pump PLL:

$$JTF(f) = \frac{1 + j(f/f_z)}{1 - (f/f_n)^2 + j\cdot 2\zeta(f/f_n)}$$

where $f_n$ is the natural frequency, $\zeta$ is the damping ratio, and $f_z$ is the loop filter zero.

**JTF shape:**

At low frequencies ($f \ll f_n$): $|JTF| \approx 1$ — the PLL tracks input jitter perfectly.
At high frequencies ($f \gg f_n$): $|JTF| \to 0$ — the PLL rejects high-frequency input jitter.
At the loop bandwidth ($f \approx f_n$): $|JTF|$ can peak above 1 (jitter amplification) for underdamped PLLs ($\zeta < 1/\sqrt{2}$).

**JTF peaking and amplification:**

For $\zeta = 0.5$ (common value), the JTF peak at $f = f_n\sqrt{1 - 2\zeta^2} = 0.707 f_n$ is:

$$|JTF|_{peak} = \frac{1}{2\zeta\sqrt{1 - \zeta^2}} = \frac{1}{2 \times 0.5 \times \sqrt{0.75}} \approx 1.15$$

This 1.15 (1.2 dB) jitter amplification at the loop bandwidth means jitter components near $f_n$ are amplified slightly by the PLL. For a CDR in a receiver, this is acceptable. For a reference clock buffer or jitter cleaner, excessive peaking transfers transmitter jitter components that happen to fall near the loop BW to the recovered clock.

**SSC tracking requirement:**

PCIe requires that the receiver CDR must track SSC modulation (down to 10 Hz per the REFCLK SSC spec). The CDR loop bandwidth must be wide enough to cover 33 kHz SSC rate with $|JTF(33\ \text{kHz})| \approx 1$. PCIe CDR loop BW is specified as 1.5–10 MHz to easily cover 33 kHz.

A narrowband CDR (e.g., 100 kHz loop BW) would attenuate the 33 kHz SSC component, causing the SSC jitter to appear as tracking error (effective TIE) and widen the eye. This is why PCIe CDR bandwidth has a hard lower limit.

---

## Tier 3: Advanced

### Q7. Derive the total jitter formula $TJ = DJ_{pp} + 14.07\,\sigma_{RJ}$ at BER $= 10^{-12}$ from the Dual-Dirac PDF, and explain the significance of the coefficient 14.07.

**Answer:**

**Setup:**

Using the Dual-Dirac model with symmetric DJ ($DJ_{pp}/2$ from eye centre to each crossing cluster), the total jitter PDF at the crossing point is:

$$f_{TJ}(t) = \frac{1}{2} \mathcal{N}\!\left(t;\ +\frac{DJ_{pp}}{2},\ \sigma_{RJ}\right) + \frac{1}{2} \mathcal{N}\!\left(t;\ -\frac{DJ_{pp}}{2},\ \sigma_{RJ}\right)$$

where $\mathcal{N}(t; \mu, \sigma)$ is a Gaussian PDF.

**BER calculation:**

Consider a sampling phase at $+T$ from the eye centre (measuring the right wall of the bathtub). A bit error occurs when a +1 jitter excursion moves the edge past the sampling point:

$$BER(T) = P(|j_{crossing}| > T)$$

For the right wall, only the tail of the Gaussian centred at $+DJ_{pp}/2$ matters at low BER (the contribution from the Gaussian centred at $-DJ_{pp}/2$ is negligible at large $T$):

$$BER(T) \approx \frac{1}{2} \int_T^{\infty} \frac{1}{\sigma_{RJ}\sqrt{2\pi}} e^{-(t - DJ_{pp}/2)^2/(2\sigma_{RJ}^2)} dt$$

$$= \frac{1}{4}\,\text{erfc}\!\left(\frac{T - DJ_{pp}/2}{\sigma_{RJ}\sqrt{2}}\right)$$

The factor $1/4$ accounts for: probability 1/2 that the relevant Gaussian is active, and probability 1/2 that the bit is a 1 (only 1→0 transitions contribute to right-wall BER in this analysis).

**Eye width at target BER:**

Setting $BER(T) = BER_{target}$ and solving for $T$:

$$\frac{T - DJ_{pp}/2}{\sigma_{RJ}\sqrt{2}} = \text{erfinv}(1 - 4 \cdot BER_{target})$$

$$T = \frac{DJ_{pp}}{2} + \sigma_{RJ}\sqrt{2}\,\text{erfinv}(1 - 4 \cdot BER_{target})$$

The eye width is $EW = 1\,UI - 2T$ (subtracting both left and right walls from the full UI):

$$EW = UI - DJ_{pp} - 2\sigma_{RJ}\sqrt{2}\,\text{erfinv}(1 - 4 \cdot BER_{target})$$

Total jitter $TJ = UI - EW$:

$$TJ_{pp} = DJ_{pp} + 2\sqrt{2}\,\text{erfinv}(1 - 4 \cdot BER_{target}) \cdot \sigma_{RJ}$$

**Evaluating the coefficient at BER = $10^{-12}$:**

$$\text{erfinv}(1 - 4 \times 10^{-12}) = \text{erfinv}(1 - 4 \times 10^{-12}) \approx \text{erfinv}(0.999999999996)$$

Using the asymptotic expansion $\text{erfinv}(1 - x) \approx \sqrt{-\ln(\pi x^2/4) / (\pi/2)} \cdot (1 + ...)$, or tabulated Q-function: the argument of erfinv corresponds to Q = 7.03 (since $Q = \sqrt{2}\,\text{erfinv}(1-2p)$ for one-tailed probability $p = 2 \times 10^{-12}$).

Therefore:

$$2\sqrt{2}\,\text{erfinv}(1 - 4 \times 10^{-12}) = 2 \times 7.03 = 14.07$$

(The factor of $\sqrt{2}$ converts between erfinv and the Q function: $Q = \sqrt{2}\,\text{erfinv}$.)

**Physical significance:**

The coefficient 14.07 is how many standard deviations of RJ must be allocated to each side of the eye to achieve BER $= 10^{-12}$. Since random jitter has infinite tails, the RJ contribution to total jitter grows without bound as BER decreases. Specifically:

| BER | RJ multiplier ($2\alpha$) | Formula |
|---|---|---|
| $10^{-6}$ | 9.51 | $TJ = DJ_{pp} + 9.51\,\sigma_{RJ}$ |
| $10^{-9}$ | 12.0 | $TJ = DJ_{pp} + 12.0\,\sigma_{RJ}$ |
| $10^{-12}$ | 14.07 | $TJ = DJ_{pp} + 14.07\,\sigma_{RJ}$ |
| $10^{-15}$ | 15.5 | $TJ = DJ_{pp} + 15.5\,\sigma_{RJ}$ |

The logarithmic growth of the multiplier means that improving BER from $10^{-12}$ to $10^{-15}$ requires only 10% more jitter allocation for the same $\sigma_{RJ}$, which is why FEC-based links (targeting $10^{-4}$ pre-FEC) can tolerate significantly higher $\sigma_{RJ}$.

---

### Q8. You measure a bathtub curve and extract $DJ_{pp} = 50$ ps and $\sigma_{RJ} = 2.5$ ps on a 25 Gbps link (UI = 40 ps). Calculate the total jitter and eye width at BER $= 10^{-12}$. Does this link meet a typical specification requiring EW $\ge 0.25$ UI?

**Answer:**

**Given:**

- Data rate: 25 Gbps, UI = $1/25 \times 10^9 = 40$ ps
- $DJ_{pp} = 50$ ps
- $\sigma_{RJ} = 2.5$ ps
- Target BER = $10^{-12}$

**Total Jitter calculation:**

$$TJ_{pp}(10^{-12}) = DJ_{pp} + 14.07 \cdot \sigma_{RJ}$$

$$TJ_{pp} = 50\ \text{ps} + 14.07 \times 2.5\ \text{ps} = 50 + 35.2 = 85.2\ \text{ps}$$

**Eye width:**

$$EW = UI - TJ_{pp} = 40 - 85.2 = -45.2\ \text{ps}$$

**Immediate problem:** $TJ_{pp} > UI$. Total jitter at $10^{-12}$ exceeds the entire UI, so the eye is completely closed. BER = $10^{-12}$ is not achievable.

**Physical interpretation:** Even if the DJ ($DJ_{pp} = 50$ ps) alone exceeds the entire UI ($40$ ps), the eye is already closed without any random jitter contribution.

**Diagnosing the failure:**

1. **DJ is too large:** $DJ_{pp} = 50$ ps on a 40 ps UI means DJ alone spans more than the full bit period. At least 25 ps of reduction is needed.

2. **$\sigma_{RJ}$ contribution:** Even if DJ were 0, $14.07 \times 2.5 = 35$ ps of RJ penalty would leave only $40 - 35 = 5$ ps eye width (12.5% UI) — marginal.

**Specification check:**

A 0.25 UI eye width requirement at BER $= 10^{-12}$ means:

$$EW_{required} = 0.25 \times 40\ \text{ps} = 10\ \text{ps}$$

For the link to meet this requirement:

$$TJ_{pp} \le UI - EW_{required} = 40 - 10 = 30\ \text{ps}$$

$$DJ_{pp} + 14.07\,\sigma_{RJ} \le 30\ \text{ps}$$

With the measured $\sigma_{RJ} = 2.5$ ps: the maximum allowable $DJ_{pp} = 30 - 35.2 = -5.2$ ps (impossible — negative DJ budget). Even with zero DJ, the RJ contribution alone ($35.2$ ps) exceeds the total jitter budget ($30$ ps).

**Corrective actions required:**

1. **Reduce $\sigma_{RJ}$:** Improve the CDR reference clock quality. If $\sigma_{RJ}$ can be reduced to 1.0 ps, the RJ term becomes $14.07 \times 1.0 = 14.07$ ps, leaving $30 - 14.07 = 15.93$ ps for DJ.

2. **Reduce DJ:** Investigate DDJ (equalize the channel to reduce ISI-induced jitter), reduce DCD (improve transmitter rise/fall symmetry), and identify any PJ sources (power supply filtering).

3. **Apply FEC:** Relax the BER target to $10^{-4}$ pre-FEC with RS FEC providing post-FEC BER $< 10^{-15}$. At BER $= 10^{-4}$, the multiplier is $\approx 8.2$ instead of 14.07: $TJ(10^{-4}) = 50 + 8.2 \times 2.5 = 50 + 20.5 = 70.5$ ps. Still larger than 40 ps UI, but the DJ reduction alone could make this achievable.

---

### Q9. Describe how a real-time oscilloscope performs jitter decomposition using spectral analysis. What is a jitter spectrum and how are RJ, PJ, and DDJ identified in it?

**Answer:**

**Jitter spectrum concept:**

A real-time oscilloscope captures a time record of TIE values for each edge in the captured data sequence. Let $TIE[n]$ be the sequence of $N$ edge timing errors. The **jitter spectrum** is the power spectral density of this sequence:

$$S_{TIE}(f) = \frac{1}{N}\left|\mathcal{F}\{TIE[n]\}\right|^2$$

plotted versus modulation frequency $f_{mod}$ (frequency of jitter modulation, different from the signal frequency $f_{data}$).

**Identifying jitter components in the spectrum:**

**Random Jitter (RJ):**

RJ has a flat (white) spectrum across all modulation frequencies (within the RJ bandwidth). In the jitter spectrum, RJ appears as a noise floor:

$$S_{RJ}(f) = \sigma_{RJ}^2 \cdot T_{UI} = \text{constant}$$

The RJ $\sigma_{RJ}$ is extracted by integrating the flat noise floor:

$$\sigma_{RJ}^2 = \int_{f_{low}}^{f_{high}} S_{noise\_floor}(f)\,df$$

**Periodic Jitter (PJ):**

PJ appears as discrete spectral lines (peaks) at the interference frequency and its harmonics. For a 100 kHz power supply switching noise source: sharp peaks at 100 kHz, 200 kHz, 300 kHz, etc.

PJ magnitude is extracted from the peak power: $PJ_{pp} = 2\sqrt{2 \cdot S_{PJ}(f_{peak}) \cdot \Delta f}$ where $\Delta f$ is the frequency resolution.

**Data-Dependent Jitter (DDJ):**

DDJ is correlated with the data pattern. For a PRBS-N pattern with period $2^N - 1$ symbols, DDJ appears as a comb of spectral lines at multiples of the pattern repetition rate $f_{rep} = f_{data}/(2^N - 1)$. For PRBS7 at 10 Gbps: $f_{rep} = 10\ \text{GHz}/127 \approx 78.7\ \text{MHz}$. DDJ lines appear at 78.7 MHz and harmonics.

**Separation procedure:**

1. Capture 1–10 million edges in a single time record.
2. Compute TIE sequence and its Fourier transform.
3. **Identify and remove PJ lines** by peak-finding in the spectrum. Each PJ peak is a sinusoidal jitter component; its amplitude is $PJ_{pp}/2$ per sideband.
4. **Identify and remove DDJ lines** at multiples of the PRBS repetition rate.
5. **The remaining noise floor is RJ.** Integrate the noise floor to obtain $\sigma_{RJ}$.
6. **DDJ peak-to-peak in time domain:** Inverse-FFT the DDJ spectral lines to recover the DDJ waveform; read its peak-to-peak value.

**Spectral analysis limitations:**

- **Frequency resolution:** With $N$ edges at baud rate $f_B$, the frequency resolution is $\Delta f = f_B/N$. For 1 million edges at 10 Gbps: $\Delta f = 10\ \text{kHz}$. PJ sources below 10 kHz cannot be resolved from the DC bin and will alias into the RJ floor estimate.

- **PRBS length:** If the observation window is not an exact multiple of the PRBS period, spectral leakage spreads the DDJ lines into the RJ floor, contaminating $\sigma_{RJ}$. Use windowing (e.g., Hann window) and ensure an integer number of PRBS periods in the capture.

- **Non-stationarity:** Thermal drift during the capture changes $\sigma_{RJ}$ in a time-varying way. Long captures average over this drift; short captures may reflect a non-representative operating point.

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| Jitter | Deviation of signal transition from ideal timing position |
| TIE | Time Interval Error; absolute phase displacement vs ideal position |
| Period jitter | Deviation of each clock period from nominal; first difference of TIE |
| Cycle-to-cycle jitter | Difference between successive periods; second difference of TIE |
| RJ | Random jitter; Gaussian, unbounded, characterised by $\sigma_{RJ}$ |
| DJ | Deterministic jitter; bounded, multi-modal, has identifiable causes |
| DDJ | Data-dependent jitter; edge timing varies with preceding bit pattern |
| PJ | Periodic jitter; sinusoidal from coherent interference; bimodal PDF |
| DCD | Duty cycle distortion; systematic asymmetry between rise and fall transitions |
| BUJ | Bounded uncorrelated jitter; bounded but not pattern-correlated |
| TJ | Total jitter; $TJ = DJ_{pp} + 14.07\,\sigma_{RJ}$ at BER $= 10^{-12}$ |
| Dual-Dirac | Jitter model: DJ as two equal delta functions $\pm DJ_{pp}/2$; RJ as Gaussian |
| Q-scale | Complementary error function scale on which Gaussian tails appear linear |
| Bathtub curve | BER vs sampling phase; walls give jitter at target BER |
| JTF | Jitter transfer function; PLL's frequency-domain response to input jitter |
| SSC | Spread-spectrum clocking; modulates clock frequency to reduce EMI |
| BERT | Bit Error Ratio Tester; measures BER by direct error counting |
| Jitter spectrum | Power spectral density of TIE sequence; identifies RJ floor and PJ/DDJ components |
| $\alpha$ (multiplier) | $\sqrt{2}\,\text{erfinv}(1-BER)$; RJ scaling factor to target BER |
