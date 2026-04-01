# ISI and Equalisation

## Overview

Inter-symbol interference (ISI) is the dominant impairment in high-speed serial channels. A dispersive channel spreads each transmitted bit's energy across adjacent time slots, so that the voltage at any sampling instant is a weighted sum of the current and several surrounding bits. At data rates above a few Gbps on standard PCB traces, the resulting eye closure is so severe that no receiver can recover data without equalisation. Equalisation counteracts the channel's distortion to restore a clean eye. This document covers the physical origin of ISI, TX-side and RX-side equalisers (FFE, CTLE, DFE), their implementation trade-offs, and the Channel Operating Margin (COM) metric used in modern standards.

---

## Tier 1: Fundamentals

### Q1. What is inter-symbol interference? Explain its origin in a dispersive channel.

**Answer:**

**Origin:**

A practical PCB trace behaves as a low-pass filter. Its frequency response $H(f)$ attenuates high-frequency components more than low-frequency ones, due to:

1. **Skin-effect loss:** At frequency $f$, current is confined to a skin depth $\delta = \sqrt{\rho/(\pi f \mu)}$. The effective conductor cross-section decreases, increasing resistance as $\sqrt{f}$.

2. **Dielectric loss:** The substrate loss tangent $D_f$ converts electromagnetic energy to heat. Attenuation scales as $f \cdot D_f$.

The combined insertion loss for a microstrip trace of length $\ell$ (in inches) is approximately:

$$IL(f) \approx -\ell \left(\alpha_{skin}\sqrt{f} + \alpha_{diel} \cdot f\right)\ \text{dB}$$

Typical PCB trace parameters at 10 GHz on FR4 ($D_f = 0.02$):
- $\alpha_{skin} \approx 0.03\ \text{dB}/\text{inch}/\sqrt{\text{GHz}}$
- $\alpha_{diel} \approx 0.04\ \text{dB}/\text{inch/GHz}$

For a 20-inch trace: $IL(10\ \text{GHz}) \approx 20(0.03\times\sqrt{10} + 0.04\times 10) = 20(0.095 + 0.40) \approx 9.9$ dB

**How filtering creates ISI:**

An ideal impulse $\delta(t)$ transmitted into this channel produces a smeared output with significant energy extending beyond one bit period. Mathematically, the received waveform at time $n$ (sampled at baud rate $1/T$) is the convolution:

$$r[n] = \sum_{k=-\infty}^{\infty} h[k] \cdot d[n-k]$$

where $h[k]$ are the sampled channel impulse response coefficients and $d[n-k]$ are the transmitted symbols at time $n-k$. The terms with $k \ne 0$ are the ISI — contributions from adjacent symbols $d[n-k]$ to the current decision at time $n$.

For an ideal single-tap channel ($h[0] = 1$, all other $h = 0$): no ISI, perfect eye.
For a real 20-inch PCB trace at 10 Gbps: dozens of non-zero taps, severely closed eye.

**Common mistake:** Assuming ISI is always due to reflections. ISI from frequency-dependent loss (skin effect and dielectric loss) is different from ISI due to impedance discontinuities. Loss-induced ISI has a smooth, causal spread. Discontinuity-induced ISI produces discrete pre- and post-cursors at the round-trip delays of the discontinuities.

---

### Q2. Define pre-cursor ISI and post-cursor ISI. Which is more common and why?

**Answer:**

**Channel cursor terminology:**

The channel impulse response (CIR), sampled at the baud rate, produces a sequence of taps. The dominant tap (largest magnitude) is the **main cursor**. Taps at earlier times are **pre-cursors**; taps at later times are **post-cursors**.

**Post-cursor ISI:**

Post-cursors arise from the channel's causal spreading: an input impulse continues to produce output for several bit periods after it was transmitted. In a loss-dominated channel, post-cursors decay exponentially. Most of the ISI energy in a typical PCB channel is in post-cursors because the channel's impulse response is approximately a decaying exponential after the main cursor.

A 20-inch PCB trace at 10 Gbps might have normalised CIR taps:

$$h = [\underbrace{0.1}_{pre-2},\ \underbrace{0.2}_{pre-1},\ \underbrace{0.7}_{main},\ \underbrace{0.4}_{post-1},\ \underbrace{0.2}_{post-2},\ \underbrace{0.1}_{post-3},\ ...]$$

Most ISI energy is in the post-cursors $h[1], h[2], h[3]$.

**Pre-cursor ISI:**

Pre-cursors arise from:
1. **Channel non-minimum phase response:** via stubs and reflective discontinuities create pre-echo.
2. **Acausal equaliser tap delay:** if the equaliser delay is not aligned to the true channel main cursor, what appear to be pre-cursors are actually shifted post-cursors.
3. **High-frequency peaking** in the channel or package: creates a leading edge undershoot that appears as a pre-cursor.

Pre-cursors have smaller magnitude than post-cursors in most passive PCB channels. However, via stubs and package resonances can produce significant pre-cursors. The TX FFE (transmit feedforward equaliser) targets pre-cursors and near post-cursors; the RX DFE (decision feedback equaliser) targets post-cursors only.

---

### Q3. What is a TX FFE (Feed-Forward Equaliser)? Explain its operation and limitations.

**Answer:**

**Concept:**

A TX FFE (also called TX de-emphasis or TX emphasis) applies a pre-computed equalisation filter at the transmitter before the signal enters the channel. By pre-distorting the transmitted waveform, it compensates for the channel's frequency-dependent attenuation so that the received waveform is more uniform across frequency.

**Operation:**

The TX FFE implements a finite impulse response (FIR) filter on the transmitted data sequence:

$$x_{eq}[n] = \sum_{k=0}^{N-1} c_k \cdot d[n-k]$$

where $d[n-k]$ are the delayed data symbols (from a shift register) and $c_k$ are the equaliser tap coefficients.

For a 3-tap TX FFE with pre-cursor tap $c_{-1}$, main cursor $c_0$, and post-cursor $c_1$:

$$x_{eq}[n] = c_{-1} d[n+1] + c_0 d[n] + c_1 d[n-1]$$

A typical PCIe Gen 4 TX FFE preset for a moderate-loss channel:

$$[c_{-1},\ c_0,\ c_1] = [-2,\ 24,\ -6]\ \text{(in units of 1/32 of full swing)}$$

The negative post-cursor coefficient $c_1 = -6/32$ de-emphasises the current bit when the previous bit was the same polarity (reducing low-frequency content) and boosts it when the polarity changes (restoring high-frequency content).

**Normalisation constraint:**

TX FFE must not increase the transmitted signal power above the allowed limit. The coefficients are normalised so that the peak voltage is preserved:

$$\sum_{k} |c_k| = 1\ \text{(in linear units)}$$

This is called the zero-forcing constraint with peak power normalisation.

**Limitations:**

1. **Reduces signal swing.** Because the main cursor $c_0 < 1$ (headroom is used for emphasis), the eye height at the transmitter output is reduced. TX FFE trades transmit amplitude for frequency flatness.

2. **Fixed preset.** In many SerDes implementations, TX FFE tap coefficients are programmed at link training time and held constant. They are not adaptive to time-varying channel conditions (temperature, connector aging).

3. **Cannot equalise post-cursor ISI beyond its tap span.** A 3-tap FFE handles only 1 pre-cursor and 1 post-cursor. Long channels with many post-cursor taps require complementary RX equalisation.

4. **Amplifies noise.** Pre-cursor taps in the FFE amplify the spectral content that the channel attenuates — but they also amplify additive noise at those frequencies. This noise enhancement is a fundamental TX FFE penalty.

---

### Q4. What is CTLE (Continuous-Time Linear Equaliser)? Describe its frequency response and where it is applied.

**Answer:**

**Concept:**

CTLE is an analog amplifier at the receiver whose frequency response is designed to be the approximate inverse of the channel's frequency response. Where the channel attenuates (at high frequencies), the CTLE provides gain. The result is a flatter total (channel × CTLE) frequency response.

**Frequency response shape:**

CTLE is typically implemented as a high-frequency peaking amplifier. A single-stage CTLE has a transfer function of the form:

$$H_{CTLE}(s) = A_{DC} \cdot \frac{1 + s/\omega_z}{1 + s/\omega_p}$$

where $\omega_z < \omega_p$. The zero $\omega_z$ provides peaking that starts at $f_z = \omega_z/(2\pi)$; the pole $\omega_p$ limits the gain at very high frequencies to prevent noise amplification. The peak gain is:

$$G_{peak} = A_{DC} \cdot \frac{\omega_p}{\omega_z}$$

For a typical 10 Gbps SerDes CTLE:
- DC gain: $A_{DC} = 0.5$ (−6 dB to not amplify low-frequency noise)
- Zero frequency: $f_z = 1$ GHz
- Pole frequency: $f_p = 5$ GHz
- Peak gain: $0.5 \times 5 = 2.5$ (approximately +8 dB peak above DC)

**Peaking magnitude in dB:**

$$G_{peak,dB} = 20\log_{10}(f_p/f_z) = 20\log_{10}(5) \approx 14\ \text{dB above DC}$$

**CTLE placement:**

CTLE is located at the **receiver**, after the channel but before the clock-data recovery (CDR) and data slicer. This is advantageous because it:
- Operates on the received waveform, adapting to actual channel loss.
- Can be made adaptive: the peaking frequency and magnitude can be digitally programmed during link training using closed-loop adaptation based on error rate feedback.
- Does not affect transmitted power.

**CTLE peaking selection:**

The optimal peaking is the one that maximises the eye opening after CTLE. If the peaking is too high, noise is amplified excessively. If too low, ISI remains. Adaptive algorithms (LMS-based) sweep the DC gain and zero/pole positions to find the optimum.

---

## Tier 2: Intermediate

### Q5. What is a DFE (Decision Feedback Equaliser)? How does it differ from FFE and CTLE, and what is its key advantage?

**Answer:**

**DFE operation:**

A DFE uses *previously decided data bits* to cancel the ISI they cause on the current bit. Once the receiver has made a hard decision on bit $d[n-1]$ (decided it is a 1 or -1), that decision is multiplied by the first post-cursor tap of the channel $h[1]$ and subtracted from the current sample $r[n]$:

$$\hat{r}[n] = r[n] - \sum_{k=1}^{K} f_k \cdot \hat{d}[n-k]$$

where $f_k$ are the DFE tap weights (trained to approximate $h[k]$) and $\hat{d}[n-k]$ are the previous decisions.

Mathematically, the DFE cancels post-cursor ISI using a recursive (IIR-like) structure driven by the slicer output rather than the received analog waveform.

**Key advantage — No noise enhancement:**

Unlike CTLE and FFE, the DFE does not amplify noise. The previous decisions $\hat{d}[n-k]$ are digital values ($\pm 1$), not analog waveforms. The feedback signal is perfectly clean (assuming correct previous decisions). Therefore, the DFE cancels ISI without adding any noise amplification. This makes DFE superior for equalising post-cursor ISI in high-SNR channels.

**Comparison:**

| Property | TX FFE | RX CTLE | RX DFE |
|---|---|---|---|
| Location | Transmitter | Receiver (analog) | Receiver (digital/mixed-signal) |
| ISI targeted | Pre- and post-cursor | All (shaped) | Post-cursor only |
| Noise amplification | Yes | Yes | No |
| Adaptive | Sometimes | Yes | Yes |
| Handles non-linear ISI | No | No | No |

**DFE limitations:**

1. **Cannot cancel pre-cursor ISI.** The DFE is causal: it can only use past decisions, not future ones. Pre-cursor ISI requires FFE or CTLE.

2. **Error propagation.** If a decision is wrong ($\hat{d}[n-k] \ne d[n-k]$), the DFE cancels the wrong ISI, which adds error rather than reducing it. This causes **error bursts** — a single bit error can corrupt several subsequent bits. Proper preamble and training reduces this risk; FEC covers residual burst errors.

3. **First-tap timing.** The first DFE tap $f_1$ must be available within one bit period of the decision — an extremely tight timing constraint at 28 Gbps (35 ps bit period). The first tap is often implemented in the slicer analog domain ("speculative DFE" or "unrolled DFE") to meet timing.

4. **Tap count limits.** Practical DFE implementations have 2–10 taps. Long channels with many post-cursors extending beyond the DFE's span have residual ISI.

---

### Q6. Explain the Channel Operating Margin (COM) metric. What does a COM value of 3 dB mean physically?

**Answer:**

**Definition:**

COM (Channel Operating Margin) is an IEEE-standardised metric (IEEE 802.3 Annex 93A and derived standards) that quantifies how much margin a high-speed serial channel has above the minimum required for a target BER, given a specified equalisation architecture. It is defined as:

$$COM = 20\log_{10}\left(\frac{A_s}{A_n}\right)\ \text{dB}$$

where:
- $A_s$ = signal amplitude at the decision point after equalisation and filtering
- $A_n$ = noise amplitude at the decision point, including ISI residual, crosstalk, jitter, and thermal noise

**Physical meaning of COM = 3 dB:**

$COM = 3$ dB means $A_s / A_n = \sqrt{2} \approx 1.41$. The signal amplitude is 41% larger than the noise amplitude at the decision point. This is the minimum passing threshold defined by the standard (COM $\ge 3$ dB required). Below 3 dB, the channel cannot meet the target BER even with the specified equalisation.

**COM computation inputs:**

1. **S-parameters:** Measured or simulated $S_{dd21}$ (channel insertion loss) and $S_{dd11}$ (return loss) of the complete channel (PCB traces, connectors, vias, packages).
2. **Transmitter parameters:** TX FFE tap count and coefficient limits, transmit amplitude.
3. **Receiver parameters:** CTLE pole/zero locations (or peaking range), DFE tap count and weight limits.
4. **Crosstalk:** $S_{dd21}$ of specified aggressor channels (NEXT and FEXT aggressors).
5. **Package models:** Transmitter and receiver package parasitics (S-parameters or RLC models).
6. **Noise floor:** Specified receiver sensitivity (broadband noise spectral density).

**COM optimisation:**

The COM computation optimises the equaliser coefficients (TX FFE taps, CTLE peaking, DFE tap weights) across all allowed settings and reports the maximum achievable COM. A channel that achieves COM $= 6$ dB has 6 dB more signal margin than noise — comfortable headroom for manufacturing variation.

**Common failure modes in COM:**

- COM fails due to excessive insertion loss (Sdd21 too negative at Nyquist): fix by reducing trace length, using lower-loss material, or upgrading connector.
- COM fails due to via stub resonance (notch in Sdd21): fix by backdrilling or redesigning via geometry.
- COM fails due to excessive crosstalk: fix by increasing trace spacing, improving stackup shielding.

---

### Q7. Describe the link training process for a PCIe Gen 5 transmitter and receiver equaliser. What signals are exchanged and how are the equalisers adapted?

**Answer:**

**PCIe Gen 5 link training overview:**

PCIe performs link training during LTSSM (Link Training and Status State Machine) sequences. Equaliser adaptation occurs in the Polling.Equalization and Config states before entering L0 (operational). At Gen 5 (32 GT/s), both TX FFE and RX CTLE are adapted.

**Training sequence:**

**Phase 1 — TX preset broadcasting (Polling.Equalization):**

The downstream component (RC, root complex) transmits data with TX preset P0 (no emphasis). The upstream component (EP, endpoint) evaluates signal quality and requests a different preset. PCIe Gen 5 defines 11 TX presets (P0–P10), each specifying a combination of [pre-cursor, cursor, post-cursor] TX FFE coefficients.

**Phase 2 — Preset evaluation:**

The EP transmits EQ (equalisation) requests via Training Set (TS) ordered sets. Each TS contains a requested TX preset and feedback about the receiver's eye quality (encoded as a Reject or Accept, or as an 8-bit coefficient request). The RC cycles through presets until the EP accepts one.

**Phase 3 — Coefficient fine-tuning (Configuration.Equalization):**

After preset selection, the RC may request fine adjustments using coeff[+1], coeff[0], coeff[-1] increment/decrement commands. The EP responds with status (FS, GS: fully saturated, geometrically stable). This continues until GS is achieved or a timeout occurs.

**Phase 4 — RX CTLE adaptation:**

RX CTLE adaptation is implementation-specific (not specified by the PCIe PHY layer standard). The PHY IP internally adapts CTLE peaking using a proprietary algorithm, often based on minimising the DFE tap magnitudes (a proxy for residual ISI) or maximising the eye height as measured by on-die eye monitors.

**Adaptive algorithm concept (RX side, proprietary):**

A common adaptive approach:

1. Measure the equalised eye opening using a built-in error detector at several CTLE peaking settings.
2. Select the peaking that maximises eye opening — a coarse sweep.
3. Converge to the optimal setting using gradient descent on the error rate: increase peaking if reducing peaking increases errors, decrease peaking if increasing it increases errors.

**Training robustness considerations:**

- At Gen 5 (32 GT/s), training must complete within 24 ms of power-on or reset. With 11 presets × round-trip latency, the training window constrains how slowly adaptation can converge.
- FEC (Forward Error Correction) is mandatory at Gen 5. The FEC must operate correctly during training, which means the training pattern must be FEC-compatible.

---

## Tier 3: Advanced

### Q8. Derive the minimum mean-square error (MMSE) criterion for a linear equaliser and explain why it outperforms a zero-forcing (ZF) equaliser in the presence of noise.

**Answer:**

**Setup:**

The received signal at baud-rate samples is:

$$r[n] = \sum_{k} h[k] d[n-k] + w[n]$$

where $h[k]$ is the channel impulse response, $d[n]$ are the transmitted symbols ($\mathbb{E}[d^2] = \sigma_d^2$), and $w[n]$ is AWGN with power $\sigma_w^2$.

A linear equaliser applies a filter $c[k]$ (with $K$ taps) to the received signal to produce a decision variable:

$$\hat{d}[n] = \sum_{k=0}^{K-1} c[k] \cdot r[n-k]$$

**Zero-Forcing Equaliser:**

The ZF equaliser sets $C(f) = 1/H(f)$, perfectly inverting the channel. This eliminates all ISI. The MSE on the decision variable (after ZF) is:

$$MSE_{ZF} = \sigma_w^2 \int_{-1/2}^{1/2} \frac{1}{|H(f)|^2} df$$

When $|H(f)|$ is small (deep nulls or high-frequency attenuation), $1/|H(f)|^2$ becomes very large, and noise is amplified massively. ZF becomes useless in the presence of significant noise.

**MMSE Equaliser:**

The MMSE equaliser minimises $\mathbb{E}[|d[n] - \hat{d}[n]|^2]$ — the mean squared error between the true symbol and the equaliser output. Using the Wiener-Hopf equation, the optimal filter in the frequency domain is:

$$C_{MMSE}(f) = \frac{H^*(f)}{|H(f)|^2 + SNR^{-1}}$$

where $SNR = \sigma_d^2 / \sigma_w^2$ is the channel SNR.

The denominator $|H(f)|^2 + SNR^{-1}$ prevents the gain from going to infinity at frequencies where $H(f) \approx 0$. When $|H(f)| \gg SNR^{-1/2}$, the MMSE equaliser approximates ZF. When $|H(f)| \ll SNR^{-1/2}$ (deep null), the MMSE equaliser *reduces* gain, accepting the ISI rather than amplifying noise.

**MMSE achievable MSE:**

$$MSE_{MMSE} = \sigma_d^2 \exp\!\left(-\int_{-1/2}^{1/2} \ln\!\left(1 + SNR \cdot |H(f)|^2\right) df\right)$$

This is always less than or equal to $MSE_{ZF}$, with equality only as $SNR \to \infty$.

**Physical interpretation:**

The MMSE criterion introduces a deliberate trade-off: it accepts a small amount of residual ISI in exchange for not amplifying noise at frequencies where the channel provides little signal. This is exactly the right trade-off for practical links where thermal noise is always present. The noise term $SNR^{-1}$ acts as a regulariser that prevents the equaliser from trying to compensate for nulls that are below the noise floor.

**Implementation:**

MMSE tap weights are computed from the channel matrix $\mathbf{H}$ and noise variance:

$$\mathbf{c}_{MMSE} = \left(\mathbf{H}^H\mathbf{H} + SNR^{-1}\mathbf{I}\right)^{-1}\mathbf{H}^H \mathbf{d}_{target}$$

In practice, SI tools (HSPICE IBIS-AMI, ADS Channel Simulator) implement this numerically using the measured S-parameter impulse response and an assumed SNR derived from the transmitter amplitude and thermal noise floor.

---

### Q9. What is speculative (unrolled) DFE? Why is it needed at 28 Gbps and above, and what are its resource costs?

**Answer:**

**The first-tap timing problem:**

At 28 Gbps, one UI = 35.7 ps. A DFE uses the previous decision to compute the first-tap cancellation term that must be ready for the current sample. The sequence is:

1. Sample $r[n-1]$ at time $(n-1)T$.
2. Apply slicer to produce decision $\hat{d}[n-1]$ — this takes $\sim 15$ ps (comparator delay).
3. Multiply $\hat{d}[n-1]$ by tap weight $f_1$ and subtract from $r[n]$ — this takes $\sim 10$ ps (multiplier + subtractor delay).
4. The corrected sample must be ready before the next sample at time $nT = (n-1)T + 35.7$ ps.

Total required: comparator + multiply + subtract $\approx 25$ ps, in a 35.7 ps window. After subtracting setup time for the second slicer, the timing closure is essentially zero. A conventional DFE cannot meet this timing at 28 Gbps.

**Speculative DFE (also called "lookahead" or "unrolled" DFE):**

Speculative DFE eliminates the timing loop by running two parallel slicers simultaneously — one assuming the previous decision was $+1$, and one assuming it was $-1$ — and selecting the correct result after the actual decision is known.

For the first DFE tap:

$$\hat{r}^+[n] = r[n] - f_1 \cdot (+1) = r[n] - f_1$$
$$\hat{r}^-[n] = r[n] - f_1 \cdot (-1) = r[n] + f_1$$

Both $\hat{r}^+$ and $\hat{r}^-$ are pre-computed before the decision on $d[n-1]$ is made. Two slicers make decisions on both hypotheses simultaneously:

$$\hat{d}^+[n] = \text{sign}(\hat{r}^+[n])$$
$$\hat{d}^-[n] = \text{sign}(\hat{r}^-[n])$$

When the decision on $d[n-1]$ becomes available (its latency is now off the critical path), a multiplexer selects the correct hypothesis output:

$$\hat{d}[n] = \begin{cases} \hat{d}^+[n] & \text{if } \hat{d}[n-1] = +1 \\ \hat{d}^-[n] & \text{if } \hat{d}[n-1] = -1 \end{cases}$$

The mux selection is an XOR and NAND delay, much faster than a full comparator — it fits within the available timing slack.

**Resource costs:**

- **First-tap circuit:** 2× slicer comparators (for $\hat{r}^+$ and $\hat{r}^-$) + 1 MUX per bit stream. This doubles the first-tap hardware.
- **For 2-tap unrolling:** 4× comparators + MUX tree. Each additional unrolled tap doubles the comparator count: 3-tap unrolling requires 8× comparators.
- **Power:** 2–4× power of a conventional DFE first stage. At 28 Gbps SerDes operating at 1 mA/GHz, this is a meaningful power overhead.
- **Tap count limitation:** Most commercial SerDes unroll only the first 1–2 DFE taps. Taps 3 and beyond are implemented as a conventional digital DFE with slower timing requirements (the decision latency is absorbed by the longer post-cursor delay).

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| ISI | Inter-symbol interference; adjacent bit energy corrupting the current sample |
| Main cursor | Dominant tap in the channel impulse response at the sampling instant |
| Pre-cursor | Channel tap at times before the main cursor; energy from future symbols |
| Post-cursor | Channel tap at times after the main cursor; energy from past symbols |
| TX FFE | Transmit feed-forward equaliser; FIR filter applied before channel |
| De-emphasis | TX FFE preset that reduces amplitude of repeated bits |
| CTLE | Continuous-time linear equaliser; analog high-frequency peaking at RX |
| Peaking | CTLE gain above DC level; boosts frequencies attenuated by channel |
| DFE | Decision feedback equaliser; cancels post-cursor ISI using past decisions |
| Error propagation | DFE failure mode: wrong decisions cause burst errors |
| ZF | Zero-forcing equaliser; inverts channel exactly; amplifies noise at nulls |
| MMSE | Minimum mean-square error equaliser; optimal with noise regularisation |
| COM | Channel Operating Margin; IEEE metric in dB; $\ge 3$ dB required |
| LMS | Least mean squares; gradient-descent algorithm for adaptive equalisation |
| Speculative DFE | Unrolled first DFE tap to break critical timing loop at $\ge 28$ Gbps |
| LTSSM | Link Training and Status State Machine (PCIe); controls equaliser adaptation |
| TX preset | PCIe-defined TX FFE coefficient combination; 11 presets for Gen 4/5 |
| Skin-effect loss | $\propto \sqrt{f}$ attenuation from reduced conductor cross-section |
| Dielectric loss | $\propto f$ attenuation from substrate loss tangent $D_f$ |
