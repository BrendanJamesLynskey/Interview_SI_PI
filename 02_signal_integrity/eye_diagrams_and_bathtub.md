# Eye Diagrams and Bathtub Curves

## Overview

The eye diagram and its derivative, the bathtub curve, are the two most important graphical tools in high-speed serial link analysis. An eye diagram is a superposition of many bit periods overlaid in time, revealing the combined effect of every impairment — intersymbol interference, jitter, noise, and reflections — in a single plot. The bathtub curve extracts the eye's statistical structure and projects it to the bit error ratios (BER) that matter in production systems. Every serial interface standard — PCIe, USB, Ethernet, MIPI, SATA — specifies compliance in terms of eye diagram masks and bathtub BER targets.

---

## Tier 1: Fundamentals

### Q1. What is an eye diagram and how is it constructed?

**Answer:**

An eye diagram is produced by capturing a long bit stream, synchronising to the bit clock, and overlaying successive 1 UI (unit interval) windows of the waveform on top of each other. The result looks like one or more eye-shaped openings.

**Construction steps:**

1. Transmit a long pseudo-random bit sequence (PRBS) or a specification-mandated test pattern (e.g., SJ, SSC-modulated for PCIe).
2. Capture the waveform with an oscilloscope (hardware) or simulate the channel impulse response convolved with the transmit waveform (software).
3. Divide the waveform into segments of exactly 1 UI length, triggered on the recovered clock edge.
4. Overlay all segments on a 2D plot with axes: horizontal = time (0 to 1 UI), vertical = voltage.

**What the eye reveals:**

- **Horizontal opening (eye width):** The time window within each UI that is free of transitions — the sampling window available to the receiver. Reduced by jitter.
- **Vertical opening (eye height):** The voltage separation between the logic-1 and logic-0 levels at the optimal sampling point. Reduced by noise, ISI, and attenuation.
- **Eye crossings:** The transition region. Asymmetric crossings at voltages other than $V_{mid}$ indicate duty cycle distortion or DC offset.
- **Eye closure:** Any waveform traces passing through the geometric centre of the eye indicate ISI-induced closure. A "closed eye" means the channel introduces enough memory that some bit sequences collapse the eye to zero opening.

**Common mistake:** Assuming a "good-looking" eye at the transmitter is sufficient. The receiver eye (after the channel) is what matters for BER. A 30 dB insertion-loss channel can completely close a 25 Gbps eye even if the transmit eye is textbook-perfect.

---

### Q2. Define eye height, eye width, and the optimal sampling point. How are they measured?

**Answer:**

**Eye Height ($E_H$):**

The vertical opening of the eye at the optimal sampling time $t_{opt}$. Formally:

$$E_H = \mu_1(t_{opt}) - \sigma_1(t_{opt}) \cdot k - \left[\mu_0(t_{opt}) + \sigma_0(t_{opt}) \cdot k\right]$$

where $\mu_1, \sigma_1$ are the mean and standard deviation of the voltage distribution of logic-1 samples, $\mu_0, \sigma_0$ for logic-0, and $k$ is the number of standard deviations corresponding to the target BER ($k \approx 7.03$ for BER $= 10^{-12}$ in a Gaussian distribution).

In practical oscilloscope measurements, eye height is read directly from the waveform histogram at the eye centre, or defined as the $V1_{low} - V0_{high}$ separation at the nominal sampling point, where $V1_{low}$ and $V0_{high}$ are the 3-sigma boundary values of the respective populations.

**Eye Width ($E_W$):**

The horizontal opening of the eye at a threshold voltage midway between the logic-0 and logic-1 levels:

$$E_W = t_{right}(V_{mid}) - t_{left}(V_{mid})$$

where $t_{left}$ and $t_{right}$ are the rightmost and leftmost edge crossings at $V_{mid}$, measured at the target BER. Eye width is directly related to total jitter (TJ): a wider eye means less jitter.

**Optimal sampling point:**

The point $(t_{opt}, V_{th})$ that maximises the eye opening and minimises BER. It is found by:

1. Scanning the threshold voltage and finding the point midway between the noise-free level distributions.
2. Scanning the sampling phase and finding the phase with maximum eye height.

In practice, CDR (clock-data recovery) circuits track the optimal phase dynamically. The optimal point is not always the geometric centre of the eye if the transition distributions are asymmetric.

---

### Q3. What is a BER bathtub curve and what information does it contain?

**Answer:**

A BER bathtub curve is a plot of bit error ratio as a function of sampling phase, typically swept from 0 to 1 UI across one bit period. The curve has a characteristic bathtub shape:

- **Left and right walls:** BER is high (near 0.5) when the sampling phase is in the eye crossing region — nearly every bit is sampled during a transition.
- **Flat bottom:** BER drops to a minimum in the centre of the eye where transitions rarely occur. For a well-designed link, this minimum may be below $10^{-15}$ — too low to measure directly.

**Reading the bathtub curve:**

```
BER
1e-1  |##               ##|
1e-3  | ###           ### |
1e-6  |   ###       ###   |
1e-9  |     ##     ##     |
1e-12 |      #_____#      |
      +--------------------+
      0    0.25  0.5  0.75  1.0 UI
```

At a target BER (e.g., $10^{-12}$), draw a horizontal line across the curve. The intersections with the left and right walls define the **eye width at BER = $10^{-12}$**. This is the total jitter (TJ) at that BER: $TJ = UI - E_W$.

**Voltage bathtub:**

The analogous plot sweeps threshold voltage at fixed optimal timing phase. The voltage bathtub gives eye height at BER, from which the noise margin can be extracted.

**Key limits:**

| Interface | BER target | Typical TJ limit |
|---|---|---|
| PCIe Gen 4/5 | $10^{-12}$ | 0.3 UI |
| USB 3.2 Gen 2 | $10^{-12}$ | 0.25 UI |
| 100G Ethernet (PAM4) | $10^{-6}$ (pre-FEC) | Per-level margin |
| SATA III | $10^{-12}$ | 0.25 UI |

---

### Q4. What is eye mask testing and why is it used for compliance?

**Answer:**

An eye mask is a geometric region in the eye diagram (a hexagon or polygon) that no valid waveform trace is permitted to enter. Compliance testing verifies that the measured eye diagram has zero hits within the mask.

**Mask structure:**

A typical NRZ eye mask contains three regions: one in the centre of the eye (forbidden zone between 1s and 0s) and two at the left and right edges (forbidden transition zones). The centre polygon is defined by:
- Horizontal span: $\pm X_1 / 2$ around the UI centre
- Vertical span: $\pm Y_1 / 2$ around the threshold voltage

**Why masks are used instead of eye height/width alone:**

1. **Standardisation.** A mask provides an unambiguous pass/fail boundary that every test lab can apply identically, regardless of how they define "eye height."

2. **Multi-dimensional.** The mask simultaneously constrains voltage margin and timing margin in a single test.

3. **Worst-case capture.** By running a long PRBS31 or PRBS15 pattern and checking mask hits, all ISI patterns are exercised — not just the ideal alternating-bit pattern.

4. **Transmit vs. receive.** Transmit-side masks define minimum acceptable signal quality at the transmitter output. Receive-side masks define minimum quality at the receiver input after the channel.

**Statistical masks:**

Modern standards (PCIe Gen 5, CEI-112G) use **statistical eye masks** checked at a target BER rather than zero-hit geometric masks. The statistical mask accounts for unbounded jitter distributions (random jitter) that can *always* produce a hit if enough bits are captured. The mask is enforced at a specified BER (e.g., the mask must not be violated at $10^{-12}$ extrapolated BER), not by direct hit-count.

---

## Tier 2: Intermediate

### Q5. Explain the difference between a real-time oscilloscope eye diagram and an equivalent-time sampling oscilloscope eye diagram. When would you choose each?

**Answer:**

**Real-Time Oscilloscope (RTO):**

An RTO samples the waveform continuously at a fixed high sample rate (e.g., 256 GSa/s for a 100 GHz instrument). Every sample is captured in sequence; no samples are missed.

Advantages:
- Captures transient and sporadic events (rogue bits, burst errors) that occur only a few times in millions of bits.
- Enables jitter decomposition in the time record itself (jitter track analysis).
- Can correlate simultaneous channels.
- Pattern-trigger capability: trigger on a specific bit sequence to capture SSC-induced eye distortion.

Limitations:
- Noise floor is higher than equivalent-time: $\sim 1\text{–}3$ mV$_{rms}$ for a broadband amplifier vs. 0.1–0.5 mV$_{rms}$ for sampled.
- Bandwidth limited to $\sim 100$ GHz (as of 2025).
- Expensive for bandwidths above 33 GHz.

**Equivalent-Time Sampling Oscilloscope (ETS/DSA):**

An ETS samples asynchronously at a slow rate and assembles the waveform from samples taken at different phases of successive bit periods. Requires a repetitive, stable signal.

Advantages:
- Much lower noise floor: $< 0.5$ mV$_{rms}$, enabling accurate eye height measurement.
- Bandwidth exceeding 100 GHz is achievable.
- Better vertical accuracy for noise margin budgeting.

Limitations:
- Cannot capture non-repetitive events. If the jitter has long-term drift, the ETS will capture it incorrectly.
- Requires phase reference (recovered clock) — clock recovery circuitry adds its own jitter.
- No ability to correlate with simultaneous data channels.

**Decision guide:**

| Scenario | Preferred instrument |
|---|---|
| Characterising a new chip transmitter output quality | ETS (lower noise floor) |
| Debugging intermittent link errors | RTO (captures transients) |
| Jitter decomposition on a locked pattern | Either; RTO preferred if SSC is present |
| 112 Gbps PAM4 compliance test | RTO (modern standards require it) |
| Via/connector discontinuity characterisation | VNA + TDR (not oscilloscope) |

---

### Q6. Define jitter in the context of an eye diagram. What is the difference between deterministic and random jitter as they appear in an eye diagram?

**Answer:**

Jitter is the deviation of a signal transition from its ideal timing position. In the eye diagram, jitter manifests as horizontal spreading of the transition edges. The distribution of edge timing positions determines the eye width at a given BER.

**Deterministic Jitter (DJ):**

Bounded peak-to-peak jitter with a characteristic multi-modal probability density function (PDF). It has identifiable causes and a finite maximum amplitude. It does *not* contribute to the Gaussian tails of the timing distribution.

In the eye diagram: DJ produces distinct, separated clusters of edge crossings rather than a smooth continuous distribution. Specific DJ types:
- **Data-Dependent Jitter (DDJ/ISI jitter):** Edge timing depends on the preceding bit sequence. Caused by frequency-dependent loss rolling off high-frequency content. Appears as the eye having multiple discrete crossing levels.
- **Periodic Jitter (PJ):** Sinusoidal modulation from a coherent interference source (power supply switching, crosstalk from a clock). Produces a bimodal crossing distribution.
- **Duty Cycle Distortion (DCD):** Systematic asymmetry between rise and fall times. Shifts the crossing point between 0→1 and 1→0 transitions.

**Random Jitter (RJ):**

Unbounded Gaussian-distributed jitter from thermal noise, shot noise, and other white-noise sources. Its PDF has infinite tails — given enough bits, an arbitrarily large jitter deviation will eventually occur. This property means RJ cannot be characterised by peak-to-peak measurement; it must be described by its standard deviation $\sigma_{RJ}$.

In the eye diagram: RJ causes smooth, continuous Gaussian edge distributions. The BER contribution of RJ at the eye crossing dominates the bathtub wall slope.

**Combined effect on BER:**

$$TJ_{pp}(BER) = DJ_{pp} + 2\sqrt{2} \cdot \text{erfinv}\left(1 - BER\right) \cdot \sigma_{RJ}$$

At BER $= 10^{-12}$, the RJ multiplier $2\sqrt{2}\,\text{erfinv}(1-10^{-12}) \approx 14.07$.

---

### Q7. Explain how ISI closes the eye diagram. Using a simple 3-tap FIR channel model, calculate the ISI-induced eye closure for a PRBS pattern.

**Answer:**

Inter-symbol interference (ISI) arises because a dispersive channel spreads a single bit's energy across multiple adjacent bit periods. Previous (and future, in non-causal representations) bits leave residual energy that adds to or subtracts from the current bit's received amplitude, causing its eye level to depend on the surrounding data pattern.

**Simple 3-tap channel model:**

Represent the channel impulse response as three discrete taps sampled at the baud rate:

$$h = [h_{-1},\ h_0,\ h_{+1}] = [-0.15,\ 0.80,\ -0.15]$$

where $h_0$ is the main cursor (the current bit), $h_{-1}$ is the pre-cursor (energy from the previous bit leaking into the current sampling time), and $h_{+1}$ is the post-cursor. The values are normalised: $\sum|h_i| = 1.1$ — a mildly lossy channel.

**Eye voltage calculation:**

For NRZ signalling with bit values $d \in \{+1, -1\}$ (mapped from $\{1, 0\}$), the received sample at time $n$ is:

$$r[n] = h_{-1} d[n+1] + h_0 d[n] + h_{+1} d[n-1]$$

Enumerate all $2^2 = 4$ two-bit patterns for the interfering bits $(d[n+1], d[n-1])$:

| $d[n+1]$ | $d[n-1]$ | ISI term | Received level (d[n]=+1) | Received level (d[n]=-1) |
|---|---|---|---|---|
| +1 | +1 | $h_{-1}(+1)+h_{+1}(+1) = -0.30$ | $+0.80-0.30=+0.50$ | $-0.80-0.30=-1.10$ |
| +1 | -1 | $h_{-1}(+1)+h_{+1}(-1) = 0.00$ | $+0.80+0.00=+0.80$ | $-0.80+0.00=-0.80$ |
| -1 | +1 | $h_{-1}(-1)+h_{+1}(+1) = 0.00$ | $+0.80+0.00=+0.80$ | $-0.80+0.00=-0.80$ |
| -1 | -1 | $h_{-1}(-1)+h_{+1}(-1) = +0.30$ | $+0.80+0.30=+1.10$ | $-0.80+0.30=-0.50$ |

**Eye height:**

The worst-case logic-1 level is $+0.50$ (pattern 1,d[n]=+1,1) and the worst-case logic-0 level is $-0.50$ (pattern -1,d[n]=-1,-1):

$$E_H = 0.50 - (-0.50) = 1.00$$

Without ISI ($h$ were an ideal single spike $[0, 1, 0]$), the eye height would be 2.00 (full swing from -1 to +1). ISI has reduced the eye height to 50% of its ideal value.

**Eye closure in dB:**

$$EC = 20\log_{10}\left(\frac{2.0}{1.0}\right) = 6\ \text{dB}$$

This 6 dB eye closure from a mild 3-tap channel illustrates why equalisation is mandatory at data rates above ~5 Gbps on typical PCB channels where pre- and post-cursors of -0.15 or worse are common at the Nyquist frequency.

---

### Q8. Describe the procedure for de-embedding fixture effects from an eye diagram measurement.

**Answer:**

Production eye diagram measurements include the test fixture (probes, SMA connectors, cable segments, PCB launch structures), which introduce additional insertion loss, return loss, and skew that are not part of the device under test (DUT). De-embedding removes these fixture contributions.

**Procedure:**

**Step 1 — Characterise the fixture with S-parameters.**

Fabricate a thru calibration structure that electrically connects the two fixture ends without the DUT (or use a known reference DUT). Measure the thru's S-parameters ($S_{thru}$). Fabricate open and short standards if full 2-port fixture characterisation is needed.

**Step 2 — Compute the DUT S-parameters.**

Using the measured total S-parameters $S_{meas}$ (fixture + DUT + fixture) and the fixture model $S_{fix}$:

$$S_{DUT} = S_{fix}^{-1} \cdot S_{meas} \cdot S_{fix}^{-1}$$

In practice, this is done in ABCD (transmission) matrix form since ABCD matrices cascade by multiplication rather than this pseudo-inverse form:

$$\mathbf{T}_{DUT} = \mathbf{T}_{fix,left}^{-1} \cdot \mathbf{T}_{meas} \cdot \mathbf{T}_{fix,right}^{-1}$$

**Step 3 — Convert back to time domain.**

Apply IFFT to the de-embedded $S_{DUT}(f)$ to recover the DUT impulse response. Convolve with the transmit waveform to reconstruct the de-embedded eye diagram.

**Step 4 — Validate.**

Cross-check by re-embedding: $S_{fix,left} \cdot S_{DUT} \cdot S_{fix,right}$ should reproduce $S_{meas}$ within VNA measurement noise.

**Common pitfalls:**

- **Port extension errors:** If the fixture reference plane is not precisely calibrated to the DUT pads, residual delay shifts the eye horizontally.
- **Asymmetric half-fixture de-embedding:** For fixtures where the left and right halves are not identical (different PCB traces, connectors), each half must be characterised separately.
- **Non-passive de-embedded result:** If the computed $S_{DUT}$ violates passivity ($|S_{21}| > 1$), the fixture model is inaccurate. This produces artificially large eye openings.

---

## Tier 3: Advanced

### Q9. Explain how a BER bathtub curve is extrapolated from a finite measurement to BER levels below $10^{-9}$. What assumptions does this extrapolation require, and where can it break down?

**Answer:**

**The measurement problem:**

At 25 Gbps, measuring BER $= 10^{-15}$ directly requires at least $10^{16}$ bits — approximately 400,000 seconds of test time. This is impractical. Instead, the bathtub curve is extrapolated from measurements at higher BER (say $10^{-4}$ to $10^{-8}$, each measurable in seconds) down to the target BER.

**Extrapolation methodology:**

The bathtub curve near each wall is dominated by the Gaussian tail of random jitter:

$$BER(t) = \frac{1}{2}\,\text{erfc}\left(\frac{t - t_{cross}}{\sigma_{RJ}\sqrt{2}}\right)$$

where $t_{cross}$ is the nominal crossing time and $\sigma_{RJ}$ is the RJ standard deviation.

On a log-BER scale, the Gaussian tail is linear in $(t - t_{cross})$ when plotted on a complementary error function (Q-scale) axis. The procedure is:

1. Measure BER at several sampling phases near each wall edge, in the $10^{-3}$ to $10^{-8}$ range.
2. Fit a straight line to each wall on the Q-scale plot.
3. Extrapolate each line to the target BER.
4. The intersection of the extrapolated left and right walls defines the eye width at the target BER.

**Required assumptions:**

1. **RJ is Gaussian and stationary.** The extrapolation is valid only if $\sigma_{RJ}$ is constant across BER levels. Non-stationary noise (burst errors, temperature drift) violates this.

2. **Deterministic jitter is constant.** DJ shifts the crossing mean but does not change the Gaussian tail slope. If DJ has a distribution (e.g., SSC sinusoidal modulation), the effective crossing time is no longer a single point and simple linear extrapolation overestimates the eye width.

3. **Single jitter source on each wall.** If two distinct jitter mechanisms (e.g., two PRBS-correlated patterns) create two crossing populations, the tail has two superimposed Gaussians with different slopes. The extrapolated line from the measured range may not represent the true tail.

4. **No correlated bit patterns beyond the PRBS length.** A PRBS7 pattern may fail to exercise all ISI patterns relevant to a real data stream.

**Where extrapolation breaks down:**

- **Crosstalk aggressors with unknown data patterns.** Crosstalk from an asynchronous aggressor creates a bimodal jitter distribution. The two tails can have very different slopes, and measuring only the combined tail produces an incorrect extrapolation.
- **Spread-spectrum clocking (SSC).** SSC modulates the bit clock frequency ±0.5%, creating a slowly-varying systematic jitter. This adds a deterministic but quasi-random component that broadens the tail without being a true Gaussian.
- **Low-frequency wander.** Long-timescale phase wander (thermal, supply voltage drift) produces non-stationary Gaussian noise. The measured $\sigma_{RJ}$ depends on measurement duration and does not extrapolate simply.

**Dual-Dirac model:**

The industry-standard extrapolation model (used in PCIe, USB, SATA) is the Dual-Dirac model, which represents DJ as two delta functions at $\pm DJ_{pp}/2$ and RJ as a Gaussian of width $\sigma_{RJ}$. The resulting BER curve is:

$$BER(t) = \frac{1}{4}\left[\text{erfc}\!\left(\frac{t + DJ_{pp}/2}{\sigma_{RJ}\sqrt{2}}\right) + \text{erfc}\!\left(\frac{-t + DJ_{pp}/2}{\sigma_{RJ}\sqrt{2}}\right)\right]$$

This is discussed further in the Jitter Analysis document.

---

### Q10. A PAM4 eye diagram has three eye openings. Explain how eye diagram metrics and BER calculations differ from NRZ, and what the key compliance parameters are for 112 Gbps PAM4 (IEEE 802.3ck).

**Answer:**

**PAM4 vs NRZ eye structure:**

PAM4 (4-level Pulse Amplitude Modulation) encodes 2 bits per symbol using four voltage levels: {-3, -1, +1, +3} (normalised). This produces three vertically stacked eye openings (Eye 0, Eye 1, Eye 2) where NRZ produces one. The symbol rate is half the bit rate: 112 Gbps PAM4 operates at 56 GBaud.

**Key differences from NRZ:**

**1. Voltage margin:**

Each of the three eyes has approximately 1/3 the voltage margin of an NRZ eye at the same swing. Specifically, the eye height of each sub-eye is:

$$E_{H,PAM4} \approx \frac{E_{H,NRZ}}{3}$$

assuming equal noise. This makes PAM4 much more sensitive to noise and crosstalk.

**2. Eye linearity:**

The three eye openings should ideally be identical in height. Level asymmetry (non-linearity) causes unequal eye heights, which raises BER preferentially in the smaller eyes. Metrics:

- **RLM (Relative Level Mismatch):** Deviation of the inner signal levels from their ideal positions $(\pm 1/3 \cdot V_{swing})$. A RLM of $> 0.95$ is typically required.
- **VECP (Vertical Eye Closure Penalty):** dB penalty on eye height due to ISI-induced eye closure across all three eyes.

**3. BER target:**

IEEE 802.3ck (112G PAM4) specifies:
- Pre-FEC BER $\le 2.4 \times 10^{-4}$ (KR4 KP4 FEC correctable)
- Post-FEC BER $< 10^{-15}$ after RS(544,514) FEC

The pre-FEC target is 8 orders of magnitude higher than an uncoded NRZ link. This relaxed target enables operation with much higher channel loss, relying on FEC to correct residual errors.

**4. COM (Channel Operating Margin):**

IEEE 802.3ck uses COM as its primary channel fitness metric. COM is defined as:

$$COM = 20\log_{10}\left(\frac{A_{signal}}{A_{noise+ISI}}\right) \text{ dB}$$

COM $\ge 3$ dB is required for a compliant channel. The COM computation accounts for:
- Transmitter EQ (TX FFE taps)
- CTLE frequency response
- DFE tap count and weight limits
- Crosstalk from specified aggressors
- Package parasitics
- VNA-measured channel S-parameters

**5. Eye mask for 112G PAM4:**

The compliance mask is applied to each of the three eye openings independently. The mask for the centre eye (Eye 1) is typically the most constraining due to ISI asymmetry.

**Measurement setup for 112 Gbps PAM4:**

- 256 GSa/s or higher real-time oscilloscope (equivalent-time insufficient due to non-repetitive ISI patterns)
- Hardware DSP de-skew and reference clock recovery (REFCLK or recovered from data)
- Software equalisation (CTLE + 2-tap DFE) applied to the captured waveform before eye analysis
- CDR loop filter bandwidth per standard specification (typically 10 MHz)

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| Eye diagram | Overlay of bit periods showing the combined effect of all channel impairments |
| Eye height | Vertical opening at the optimal sampling phase; bounded by noise and ISI |
| Eye width | Horizontal opening at the threshold voltage; bounded by jitter |
| UI (Unit Interval) | Duration of one bit period; $UI = 1/f_{baud}$ |
| BER bathtub | Plot of BER vs. sampling phase; width at target BER gives timing margin |
| Eye mask | Forbidden region in eye diagram; violations = compliance failure |
| ISI | Inter-symbol interference; energy from adjacent bits distorts current sample |
| DJ | Deterministic jitter; bounded, multi-modal, has identifiable causes |
| RJ | Random jitter; Gaussian, unbounded, characterised by $\sigma_{RJ}$ |
| TJ | Total jitter; $TJ = DJ_{pp} + k\,\sigma_{RJ}$ at a given BER |
| ETS | Equivalent-time sampling oscilloscope; low noise, requires repetitive signal |
| RTO | Real-time oscilloscope; captures transients, higher noise floor |
| De-embedding | Mathematical removal of fixture S-parameters to isolate DUT response |
| PAM4 | 4-level amplitude modulation; 3 eye openings, used at 56 GBaud and above |
| COM | Channel Operating Margin; IEEE metric combining ISI, EQ, and noise |
| RLM | Relative Level Mismatch; linearity metric for PAM4 signal levels |
| Dual-Dirac | Jitter extrapolation model: DJ as two delta functions + Gaussian RJ |
