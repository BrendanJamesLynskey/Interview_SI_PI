# Digital Signal Spectrum and Knee Frequency

## Overview

A digital signal — whether a clock edge, a data transition on a bus, or a current transient drawn by a switching gate — is defined in the time domain by its amplitude and edge timing. But the physical systems it must traverse (transmission lines, PDNs, connectors, package parasitics) respond as a function of frequency. To reason about whether a channel, a decoupling network, or a measurement instrument is "fast enough" for a given digital signal, the engineer must translate time-domain edges into frequency-domain spectra.

The central concept is the **digital knee frequency** (Howard Johnson, *High-Speed Digital Design*): a single figure of merit derived from the edge rise time that bounds the frequency range over which the digital signal's spectral content is significant. Above the knee, the signal's energy drops off rapidly; below the knee, full amplitude must be preserved. The knee frequency is independent of the clock rate — it is set by the edge rate alone.

This document derives the spectrum of digital signals from Fourier analysis, defines the knee frequency and its common approximations, and shows how the concept unifies signal integrity and power integrity bandwidth calculations.

---

## Tier 1: Fundamentals

### Q1. What is the frequency spectrum of an ideal square wave?

**Answer:**

An ideal (zero rise time) square wave with period $T$ and 50% duty cycle, toggling between $-V/2$ and $+V/2$, has the Fourier series:

$$x(t) = \frac{4V}{\pi}\sum_{n=1,3,5,\ldots}^{\infty} \frac{1}{n}\sin(2\pi n f_0 t)$$

where $f_0 = 1/T$ is the fundamental frequency.

**Key properties:**

- **Odd harmonics only.** For a symmetric 50% duty cycle square wave, all even harmonics vanish.
- **$1/n$ amplitude rolloff.** The $n$-th harmonic has amplitude $(4V)/(\pi n)$ — a 20 dB/decade rolloff in amplitude, equivalent to 40 dB/decade in power.
- **Infinite bandwidth.** The spectrum extends to infinity. No real channel can pass all these harmonics, which is why an ideal square wave is physically impossible.

**Numerical example — a 1 V, 1 GHz ideal square wave:**

| Harmonic | Frequency | Amplitude | dB below fundamental |
|---|---|---|---|
| 1 (fundamental) | 1 GHz | $4/\pi \approx 1.27$ V | 0 dB |
| 3 | 3 GHz | $1.27/3 \approx 0.42$ V | $-9.5$ dB |
| 5 | 5 GHz | $1.27/5 \approx 0.25$ V | $-14.0$ dB |
| 7 | 7 GHz | $1.27/7 \approx 0.18$ V | $-16.9$ dB |
| 9 | 9 GHz | $1.27/9 \approx 0.14$ V | $-19.1$ dB |

**Practical implication:** Even if a channel passes only the fundamental and third harmonic of a 1 GHz clock, the recovered waveform is recognisable as a rounded square wave; if it passes up to the ninth harmonic, the waveform has visible but damped ringing. This is why oscilloscope bandwidth is often specified at "3× to 5× the fundamental frequency" for clean square-wave capture.

---

### Q2. How does finite rise time change the spectrum?

**Answer:**

A real digital signal has a finite rise time $t_r$ (conventionally the 10% to 90% transition time). The waveform is no longer a pure square wave; it is a trapezoidal pulse.

The Fourier transform of a periodic trapezoidal pulse is the product of two factors:

$$|X_n| \propto \left|\frac{\sin(\pi n f_0 \tau)}{\pi n f_0 \tau}\right| \cdot \left|\frac{\sin(\pi n f_0 t_r)}{\pi n f_0 t_r}\right|$$

The first $\text{sinc}$ factor (pulse width $\tau$) gives nulls at multiples of $1/\tau$ — this is the duty-cycle factor, present for any pulse train. The second $\text{sinc}$ factor (rise time $t_r$) is the one that matters for rise-time-limited bandwidth: it applies a low-pass roll-off that begins around $f \approx 1/(\pi t_r)$ and imposes an additional 20 dB/decade attenuation above that corner.

**The result:** The trapezoidal pulse spectrum has two regions:

- **Low frequency ($f \lesssim 1/(\pi t_r)$):** spectrum resembles the ideal square wave — odd harmonics falling at 20 dB/decade ($1/n$).
- **High frequency ($f \gtrsim 1/(\pi t_r)$):** the additional sinc from the rise time kicks in — total rolloff becomes 40 dB/decade ($1/n^2$).

This transition frequency — where the rolloff slope doubles — is the **knee frequency**. It is the natural break point in the spectrum of any trapezoidal digital signal.

---

### Q3. What is the digital knee frequency, and what is the standard formula?

**Answer:**

The knee frequency $f_{knee}$ is the frequency above which the spectral content of a digital signal drops off rapidly, set by the signal's rise time. It is the engineering bandwidth of the signal — the frequency above which the channel need not faithfully transmit, because the signal itself contains little energy there.

**Howard Johnson's standard approximation:**

$$\boxed{f_{knee} \approx \frac{0.5}{t_r}}$$

where $t_r$ is the 10%–90% rise time.

**Alternative approximation:**

$$f_{knee} \approx \frac{0.35}{t_r}$$

This form assumes a Gaussian-like edge (e.g., the output of a linear first-order low-pass filter); the 0.5 form assumes a more aggressive spectral cutoff closer to a pure trapezoid. Both are used in industry. The 0.35 form is standard for oscilloscope bandwidth specification:

$$f_{-3\,dB,scope} \approx \frac{0.35}{t_{r,scope}}$$

**Rule of thumb:** For a 1 ns rise time, $f_{knee} \approx 350$ MHz to 500 MHz. For a 100 ps rise time, $f_{knee} \approx 3.5$ GHz to 5 GHz.

**Numerical examples:**

| Rise time ($t_r$, 10–90%) | $f_{knee} = 0.5/t_r$ | $f_{knee} = 0.35/t_r$ |
|---|---|---|
| 10 ns | 50 MHz | 35 MHz |
| 1 ns | 500 MHz | 350 MHz |
| 500 ps | 1.0 GHz | 700 MHz |
| 200 ps | 2.5 GHz | 1.75 GHz |
| 100 ps | 5.0 GHz | 3.5 GHz |
| 50 ps | 10.0 GHz | 7.0 GHz |
| 20 ps | 25.0 GHz | 17.5 GHz |

Both formulas give the same order of magnitude. In conservative SI analysis, use the larger value (0.5/$t_r$) to ensure margin.

---

### Q4. Why is the knee frequency the "engineering bandwidth" of a digital signal?

**Answer:**

The knee frequency is the practical upper bound of the frequency range over which a channel must faithfully preserve amplitude and phase. Above $f_{knee}$, two things happen that make higher-frequency channel performance increasingly unimportant:

1. **The signal's spectral energy has dropped well below the fundamental.** Amplitude at $10 f_{knee}$ is typically 40 dB (100×) lower than at $f_{knee}$. Distortion or attenuation at these frequencies has little observable effect on the time-domain waveform.

2. **Noise and crosstalk from other sources typically exceed the signal's own content at these frequencies.** Passing more channel bandwidth than needed admits more noise without adding useful signal.

**Implication for channel design:** If $f_{knee} = 2$ GHz (corresponding to $t_r = 250$ ps), the channel must be characterised and must behave well from DC to approximately $2 \times f_{knee} = 4$ GHz (one octave of margin). Above this, S-parameter behaviour can be degraded without degrading the transmitted eye. This is why SERDES channel compliance masks typically specify insertion loss budgets at multiples of the Nyquist rate, not at arbitrary high frequencies.

**Implication for PDN design:** If the fastest current transient has $t_r = 1$ ns, the PDN must present low impedance from the VRM loop bandwidth up to approximately $f_{knee} = 500$ MHz. Above this, the current demand has negligible spectral content and even a high-impedance PDN produces small voltage perturbations.

**The common pitfall:** Engineers new to high-speed design often assume the bandwidth requirement is set by the clock frequency $f_{clk}$, not by the rise time. A 100 MHz clock with 200 ps edges has a knee frequency of 2.5 GHz — the channel must support GHz-class performance even though the clock is slow. Modern CMOS logic frequently has rise times disproportionate to its clock frequency: a device clocked at 50 MHz may have sub-nanosecond edges, creating PDN and SI challenges at frequencies far above the clock rate.

---

## Tier 2: Intermediate

### Q5. Derive the knee frequency from the trapezoidal pulse spectrum.

**Answer:**

A single trapezoidal pulse of amplitude $V$, pulse width $\tau$ (measured at 50% amplitude), and rise/fall time $t_r$ has the Fourier transform:

$$|X(f)| = V\tau \cdot \left|\frac{\sin(\pi f \tau)}{\pi f \tau}\right| \cdot \left|\frac{\sin(\pi f t_r)}{\pi f t_r}\right|$$

**Behaviour at low frequency** ($f \ll 1/\tau$): both sinc functions are close to 1, so $|X(f)| \approx V\tau$.

**First rolloff region** ($1/\tau \lesssim f \lesssim 1/(\pi t_r)$): the pulse-width sinc envelope decays as $1/f$ (–20 dB/decade). The rise-time sinc is still close to 1.

**Second rolloff region** ($f \gtrsim 1/(\pi t_r)$): the rise-time sinc also begins to decay as $1/f$, adding another –20 dB/decade. Net rolloff is $1/f^2$ (–40 dB/decade).

The transition between the two slopes occurs approximately where the rise-time sinc first deviates significantly from unity. Setting $\pi f t_r = 1$ gives:

$$f_{knee} = \frac{1}{\pi t_r} \approx \frac{0.318}{t_r}$$

Different conventions for how to locate the "true" break point give different numerical coefficients — 0.318, 0.35, and 0.5 all appear in the literature. The underlying physics is unchanged: the knee is set by the rise time, with a coefficient of order $\pi^{-1}$ to $1/2$.

**Conservative design rule:** Use the largest common coefficient (0.5) when specifying channel bandwidth or PDN coverage to ensure margin.

---

### Q6. How do the knee frequency and the clock frequency relate, and why is the clock frequency not the right bandwidth metric?

**Answer:**

The clock frequency $f_{clk}$ determines the *repetition rate* of edges. The knee frequency $f_{knee} = 0.5/t_r$ determines the *spectral extent* of each edge. These are independent — a given rise time sets the knee regardless of how often the edges repeat.

**Two limiting cases illustrate the independence:**

**Case A: A 100 MHz clock with 10 ns edges ($t_r = 10$ ns)**
- $f_{clk} = 100$ MHz
- $f_{knee} = 0.5/10\,\text{ns} = 50$ MHz

The knee is *below* the fundamental. The clock is nearly a sine wave — its harmonics are heavily attenuated by the slow edge. Channel requirements: roughly DC to 100–150 MHz for moderate fidelity.

**Case B: A 100 MHz clock with 200 ps edges ($t_r = 200$ ps)**
- $f_{clk} = 100$ MHz
- $f_{knee} = 0.5/200\,\text{ps} = 2.5$ GHz

The knee is 25× above the fundamental. The clock waveform is a near-ideal square wave with odd harmonics reaching well into the GHz range. Channel requirements: DC to several GHz even though the clock rate is only 100 MHz.

**Engineering conclusion:** If you are handed a signal described only by its clock rate, you cannot yet answer "what is the required channel bandwidth?" You must first ask about the edge rate. Modern logic families (LVDS, HSTL, SSTL) have edge rates determined by the output driver, not by the clock — a slow clock driven by a fast driver still creates GHz-class SI and PI challenges.

**Common mistake:** Computing bandwidth from the data rate (e.g., "1 Gb/s NRZ, so I need 1 GHz of channel"). NRZ has a Nyquist rate of half the bit rate, but the rise-time-limited knee frequency is usually much higher than that. A 1 Gb/s NRZ signal with 100 ps edges has a knee of 5 GHz — five times the Nyquist rate — and the channel must be characterised over that range.

---

### Q7. How does the knee-frequency concept transfer from signal integrity to power integrity?

**Answer:**

In signal integrity, the knee frequency describes the spectral extent of a voltage edge launched onto a transmission line. In power integrity, the analogous concept describes the spectral extent of a current transient drawn by a switching load.

**SI perspective:**

A voltage edge with rise time $t_r$ has spectral content up to $f_{knee} = 0.5/t_r$. The channel (trace, vias, connectors) must present low insertion loss and a flat phase response from DC to approximately this frequency.

**PI perspective:**

When gates switch, they draw a transient current from the power rail. The current waveform is trapezoidal (approximately), with a rise time set by gate-level charging dynamics. Let $t_{r,I}$ be the current transient rise time; then the current transient has spectral content up to $f_{knee,I} = 0.5/t_{r,I}$. The PDN must present impedance below $Z_{target}$ from DC (actually VRM loop bandwidth) up to approximately this frequency.

**Crucial difference:** Current transient rise time is typically faster than signal edge rise time. A CMOS output driver switching its load has its current transient set not by the output edge rate but by the internal gate network charging — the current demand pulse can have rise times of tens of picoseconds even when the external signal edge is 200 ps or slower. This is why PDN design frequently deals with knee frequencies well above the corresponding SI channel requirements. A processor with 100 ps output edges may demand current with 30 ps rise times, requiring PDN response out to 15+ GHz at the die level.

**Applied formula for PI bandwidth:**

$$f_{high,PDN} = \frac{0.5}{t_{r,I}} \quad \text{(upper frequency for which } Z_{PDN} \leq Z_{target}\text{)}$$

See [Target Impedance Method](../03_power_integrity/target_impedance_method.md) for how this frequency bounds PDN design.

---

### Q8. What is the difference between 10–90% and 20–80% rise time, and how do the conversions affect knee-frequency calculations?

**Answer:**

Rise time is the time for a signal to traverse a specified fraction of its final step amplitude. Two conventions are common:

- **10%–90% rise time ($t_{r,10-90}$):** standard for oscilloscope specifications and most textbook formulas
- **20%–80% rise time ($t_{r,20-80}$):** used in some SERDES compliance specifications and IBIS models

For a linear first-order ($RC$) response, the conversion is:

$$t_{r,10-90} = 2.2 \tau, \quad t_{r,20-80} = 1.386 \tau$$

$$\frac{t_{r,10-90}}{t_{r,20-80}} = \frac{2.2}{1.386} \approx 1.59$$

For a Gaussian-like response:

$$\frac{t_{r,10-90}}{t_{r,20-80}} \approx 1.52$$

**Practical conversion:** $t_{r,10-90} \approx 1.5 \times t_{r,20-80}$ (approximately).

**Impact on knee frequency:** If a datasheet quotes $t_{r,20-80} = 100$ ps, the equivalent $t_{r,10-90} \approx 150$ ps, and $f_{knee} \approx 0.5/150\,\text{ps} = 3.3$ GHz (not 5 GHz). Always confirm which definition the source uses — a factor of 1.5 in knee frequency can mean the difference between an adequate and an inadequate channel specification.

**Rule of thumb:** When in doubt, use the faster (shorter) rise-time number; this produces a higher knee frequency and a more conservative bandwidth requirement.

---

## Tier 3: Advanced

### Q9. How does the spectrum of a repetitive clock differ from the spectrum of random NRZ data, and what does this imply for PDN and SI design?

**Answer:**

A repetitive clock at frequency $f_{clk}$ has a discrete-line spectrum: all energy is concentrated at $f_{clk}, 3 f_{clk}, 5 f_{clk}, \ldots$ (plus minor content at even harmonics if duty cycle is imperfect). Individual spectral lines can be quite tall, but the gaps between them contain no energy at all.

**Random NRZ data** (scrambled, 50% mark density) at bit rate $f_b$ has a continuous spectrum with a $\text{sinc}^2(f T_b)$ envelope (where $T_b = 1/f_b$). The spectrum has its first null at $f = f_b$, peaks at $f = f_b/2$ (the Nyquist frequency for NRZ), and declines thereafter. Energy is distributed continuously across frequency.

**PDN design implication:**

For a repetitive clock, PDN resonances that fall between clock harmonics are harmless — the load current has no energy at those frequencies to excite them. PDN resonances at or near clock harmonics are dangerous — sustained sinusoidal current at the resonant frequency causes amplification (see [target impedance method, resonance excitation](../03_power_integrity/target_impedance_method.md)).

For random data, every frequency within the signal bandwidth has some excitation. PDN resonances anywhere in the band matter. There is no "safe gap" to hide them in.

**SI design implication:**

For a repetitive clock, channel nulls (due to, for example, via stub resonances) between harmonics have no effect on fidelity. A null exactly at a harmonic strips that harmonic from the recovered signal, causing recognisable distortion. For random data, all channel anomalies contribute to ISI across the signal bandwidth.

**Practical consequence:** Clock-distribution PDN design can tolerate peaked impedance profiles if the peaks miss the clock harmonics. Data-bus PDN design cannot. Server and memory-bus PDNs typically carry random-data-like current spectra and must meet the target impedance across a continuous band.

---

### Q10. How do data encoding schemes (8b/10b, 64b/66b, PAM4) modify the signal spectrum and the resulting knee-frequency analysis?

**Answer:**

Line coding changes the statistical properties of the transmitted signal and therefore its spectrum. Three common cases:

**NRZ with DC balancing (8b/10b):**

Ensures approximate DC balance and bounded run length. The spectrum retains the $\text{sinc}^2$ shape of random NRZ but with suppressed low-frequency content (AC-coupled compatibility). Peak frequency remains at $f_b/2$; knee-frequency analysis proceeds with the same rise-time-based $f_{knee} = 0.5/t_r$.

**Scrambled NRZ (64b/66b, PCIe Gen 3+):**

Produces whitened spectrum without preamble-style DC balance. Spectral content extends to full Nyquist at $f_b/2$, with full-amplitude content right up to that frequency. Channel requirements are stricter than for 8b/10b because the edge of the spectrum carries nearly full signal energy. Knee-frequency analysis is unchanged but margin to the Nyquist is tighter.

**PAM4 (PCIe Gen 6, 400GBase, DDR6):**

Transmits 2 bits per symbol via 4 amplitude levels. Symbol rate is half the bit rate, so the Nyquist is at $f_b/4$ — half that of NRZ for the same bit rate. However:

- Rise-time requirements are similar to NRZ at the symbol rate, so $f_{knee}$ is roughly unchanged.
- Signal-to-noise ratio requirements are 9.5 dB higher (3× noise on each amplitude level), tightening eye-height budgets.
- Linear channel response matters more than with NRZ — non-linearities that were negligible at 2 levels now create level-squashing errors at 4 levels.

**Engineering consequence for PAM4:** The headline "half the Nyquist frequency" is misleading for channel design. The required channel bandwidth, driven by rise time, is comparable to NRZ at the same bit rate. The real gain is at the equalisation and noise-margin level, not the channel-bandwidth level.

---

### Q11. What is the minimum channel bandwidth required to preserve a digital signal with a specified rise time, and how much margin should be included?

**Answer:**

Two commonly cited criteria relate channel bandwidth to signal rise time:

**Criterion 1: The 3 dB bandwidth must equal or exceed the knee frequency.**

$$f_{-3\,dB,channel} \geq f_{knee} = \frac{0.5}{t_r}$$

This preserves the fundamental and lower harmonics with minimal attenuation, yielding a recognisable but somewhat rounded waveform.

**Criterion 2: Channel bandwidth must be $2 \times f_{knee}$ for full-fidelity preservation.**

$$f_{-3\,dB,channel} \geq 2 f_{knee} = \frac{1}{t_r}$$

This ensures that the third harmonic of a square wave falls within the channel passband, preserving the sharp edge profile.

**Channel rise-time rule:**

Channel and signal rise-time addition is approximately root-sum-squared:

$$t_{r,output}^2 \approx t_{r,input}^2 + t_{r,channel}^2$$

where $t_{r,channel}$ is the channel's own 10–90% step response. For the output rise time to degrade by less than 10% relative to the input:

$$t_{r,channel} \leq 0.46 \cdot t_{r,input}$$

Equivalently, the channel's bandwidth must satisfy:

$$f_{-3\,dB,channel} \geq \frac{0.35}{t_{r,channel}} = \frac{0.35}{0.46 \cdot t_{r,input}} \approx \frac{0.76}{t_{r,input}} \approx 1.5 f_{knee}$$

**Practical recommendation:** For instrumentation (oscilloscope), target $3 \times f_{knee}$. For data channel design, target $1.5 \times f_{knee}$ as a minimum with additional margin for process and temperature variation. For PDN design, $f_{knee,I}$ of the current transient is already a conservative upper bound.

---

## Summary Table: Knee Frequency for Common Signal Types

| Signal | Typical $t_r$ | $f_{knee} = 0.5/t_r$ | Notes |
|---|---|---|---|
| TTL (74LS) | 15 ns | 33 MHz | Legacy logic |
| CMOS 3.3 V | 1–2 ns | 250–500 MHz | Standard FPGA I/O |
| LVDS | 300 ps | 1.7 GHz | SSTL/LVDS fast logic |
| DDR4 DQ | 150 ps | 3.3 GHz | Data I/O edges |
| PCIe Gen 3 (8 GT/s) | 60 ps | 8.3 GHz | SERDES output |
| PCIe Gen 5 (32 GT/s) | 20 ps | 25 GHz | Modern SERDES |
| Internal gate current transient | 30–100 ps | 5–17 GHz | PDN die-level |

---

## Cross-References

- [Propagation Delay and Dielectric](propagation_delay_and_dielectric.md) — transmission-line treatment threshold uses rise time
- [S-Parameters](../02_signal_integrity/s_parameters.md) — channel characterisation up to $f_{knee}$
- [ISI and Equalisation](../02_signal_integrity/isi_and_equalisation.md) — channel-induced degradation of rise time
- [Target Impedance Method](../03_power_integrity/target_impedance_method.md) — applies knee frequency to PDN bandwidth
- [PDN Impedance](../03_power_integrity/pdn_impedance.md) — PDN must cover up to current-transient knee frequency

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Knee frequency (standard) | $f_{knee} = 0.5 / t_r$ |
| Knee frequency (Gaussian edge) | $f_{knee} = 0.35 / t_r$ |
| Trapezoidal pulse spectrum | $|X(f)| \propto \text{sinc}(\pi f \tau) \cdot \text{sinc}(\pi f t_r)$ |
| 10–90% to 20–80% conversion | $t_{r,10-90} \approx 1.5 \cdot t_{r,20-80}$ |
| Channel rise-time addition | $t_{r,out}^2 = t_{r,in}^2 + t_{r,channel}^2$ |
| Scope bandwidth from rise time | $f_{-3\,dB} = 0.35 / t_{r,scope}$ |
