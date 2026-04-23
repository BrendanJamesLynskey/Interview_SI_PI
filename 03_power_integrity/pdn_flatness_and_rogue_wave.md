# PDN Flatness and the Rogue-Wave Problem

## Overview

The target-impedance method — pick $Z_{target} = \Delta V_{max}/I_{max}$, and design the PDN so that $|Z(f)| \le Z_{target}$ — is the cornerstone of classical PDN design. But "$|Z(f)| \le Z_{target}$ at every frequency" is **not sufficient** to guarantee that rail noise stays below $\Delta V_{max}$ in the time domain. A PDN with sharp anti-resonant peaks can violate the rail tolerance even though each peak sits exactly at $Z_{target}$ — because a repeating current waveform with the right period will resonantly build up a much larger voltage than $Z_{target}$ × I predicts. This is the **rogue-wave problem** (Larry Smith & Steve Sandler, Signal Integrity Journal 2019): a meeting of the target impedance spec that still fails in the lab. This document covers why, what "PDN flatness" means, how to measure it, and how to budget for it.

---

## Tier 1: Fundamentals

### Q1. What is the classical target-impedance method, and why is it not sufficient?

**Answer:**

**Classical method:**

Given a maximum transient current $I_{max}$ and a maximum allowed rail voltage excursion $\Delta V_{max}$ (e.g., 3% of $V_{DD}$), define:

$$Z_{target} = \frac{\Delta V_{max}}{I_{max}}$$

Then design the PDN so that $|Z_{PDN}(f)| \le Z_{target}$ across the "meaningful" frequency range (DC to the load's rise-time knee).

**Why it is not sufficient:**

The method treats the load as a step of current: $\Delta V = Z \cdot \Delta I$. That is correct for a single aperiodic transient. But real load currents are:

1. **Periodic** at clock frequencies and clock sub-multiples — every cycle draws the same current shape.
2. **Narrowband** — the current spectrum concentrates at discrete harmonics.

If the PDN has an impedance peak at exactly the frequency of one of those harmonics, the rail voltage at that frequency is:

$$V_{rail}(f_0) = |Z(f_0)| \cdot I(f_0)$$

evaluated *per-harmonic*. Because the harmonic content of a periodic waveform can be much larger than a single step's Fourier content, the *voltage* at the resonant frequency can dwarf what the time-domain step analysis predicts. Even at $Z_{target}$, a narrowband periodic excitation pumps energy into the resonance over many cycles, building up to a steady-state amplitude far larger than a single step response would show.

**Numerical example:**

A PDN with $|Z| = 5$ m$\Omega$ flat plus a 20-m$\Omega$ sharp resonant peak at 50 MHz, excited by a square current waveform at 50 MHz with amplitude 1 A peak-to-peak. The fundamental Fourier component of a 50-MHz square wave is $(4/\pi) \cdot (1 A / 2) \approx 0.64$ A peak.

Rail voltage at 50 MHz: $V = 20\ \text{m}\Omega \times 0.64\ A = 12.7$ mV peak.

Compared to the stepwise estimate $\Delta V = 5\ \text{m}\Omega \times 1\ A = 5$ mV, the actual peak rail voltage is **~2.5× larger** — because the resonance amplifies the single harmonic.

If the target was 10 mV, the PDN *meets* $|Z(f)| \le 20$ m$\Omega$ at the peak (if $Z_{target} = 20$ m$\Omega$) but *fails* the 10 mV rail spec. Hence the necessity of a **flatness** requirement on top of the magnitude requirement.

---

### Q2. Define "PDN flatness" and explain why it matters.

**Answer:**

**PDN flatness** is the peak-to-minimum ratio (or the Q-factor) of the $|Z_{PDN}(f)|$ profile across the frequency band of interest:

$$\text{Flatness} = \frac{|Z|_{peak}}{|Z|_{avg}}$$

or, equivalently, the decibel range over which $|Z(f)|$ varies:

$$\Delta Z_{dB} = 20\log_{10}\left(\frac{|Z|_{peak}}{|Z|_{min}}\right)$$

A PDN with $\Delta Z_{dB} = 3$ dB is very flat (peak is 1.4× the minimum); $\Delta Z_{dB} = 20$ dB is highly peaked (peak is 10× the minimum) and prone to rogue-wave build-up.

**Why it matters:**

1. **Narrowband excitation amplification:** as derived above, a periodic load current at the resonant frequency builds up a voltage larger than the target-impedance prediction.
2. **Coincidence failure:** any load with spectral content at the peak causes a noise spike. Given that real silicon generates current at every clock harmonic from 10 MHz to multi-GHz, the chance that *some* harmonic lines up with a sharp peak is high.
3. **Product-lot variability:** small changes in component tolerance (MLCC bias derating, ESR variation, layout tolerances) shift the resonance frequency by 10–30%. A PDN that passes lab measurement at the nominal peak may fail in another unit where the peak shifts onto a harmonic.

**Flatness target for a modern PDN:**

Industry best practice (Larry Smith, Steve Sandler, DesignCon papers) is to target $\Delta Z_{dB} \le 6$ dB across the frequency band where the load has significant current spectrum. For high-performance designs (AI accelerators, HPC CPUs), $\Delta Z_{dB} \le 3$ dB is the aspiration.

**Common mistake:** designing the PDN to $|Z| \le Z_{target}$ and believing that is sufficient. Always additionally check flatness and, where possible, simulate with workload-representative current waveforms rather than a single step.

---

### Q3. What is a "rogue wave" in PDN context, and where did the term come from?

**Answer:**

A **rogue wave** in PDN context is a rail voltage excursion that is much larger than naive $Z_{target} \cdot I_{max}$ predicts, arising when a periodic load current resonates with a PDN impedance peak. The term is borrowed from oceanography (rogue ocean waves that are much taller than the surrounding sea state) and was popularised for PDN design by Larry Smith and Steve Sandler in *Principles of Power Integrity for PDN Design — Simplified* (Prentice Hall, 2017) and in *Signal Integrity Journal* articles ("Target Impedance Is Not Enough", 2019).

**Mechanism:**

A resonant PDN stores energy in a tank circuit (the inductance between two decoupling stages against the capacitance of those stages). If the load current drives energy into that tank at the resonance frequency — even at moderate amplitude — the tank accumulates energy over multiple cycles until the stored energy is limited by damping. The resulting voltage amplitude can be orders of magnitude higher than a single-shot $I \cdot Z$ prediction.

**Mathematical description:**

For a series-RLC model of one PDN stage, excited by sinusoidal current $I(t) = I_0 \sin(\omega t)$:

$$V(t) = |Z(\omega)| I_0 \sin(\omega t + \phi)$$

At the resonant frequency $\omega_0$: $|Z(\omega_0)| \approx R$ (ESR of the cap).

At the anti-resonance frequency $\omega_{AR}$ (where two adjacent stages interact in parallel): $|Z(\omega_{AR})| \approx \sqrt{L/C}$, which can be 10× the ESR.

Over $N$ cycles at the anti-resonance, the tank voltage amplitude reaches:

$$V_{ss} \approx |Z(\omega_{AR})| \cdot I_0$$

in steady state (roughly $Q \times$ the first-cycle amplitude, where $Q$ is the quality factor). For a high-Q peak, this is much larger than the stepwise estimate.

**Practical test:**

In sensitive lab measurements, drive the PDN with a tone generator at the suspected anti-resonance frequency and measure the rail noise. If the rail noise is much larger than $Z_{target} \cdot I_{excitation}$, the PDN has a rogue wave at that frequency.

---

## Tier 2: Intermediate

### Q4. How do you design a PDN for flatness rather than just magnitude? What techniques damp anti-resonance peaks?

**Answer:**

**Design techniques for flatness:**

**1. Use capacitors with deliberately higher ESR ("ESR-tuned" or "controlled-ESR" caps).**

A cap with high ESR has low Q. Placing a low-Q cap at the anti-resonance frequency damps the peak. Polymer aluminium electrolytics (POSCAPs, OS-CONs) have naturally high ESR (5–30 m$\Omega$) and damp low-frequency anti-resonance peaks better than MLCCs. For this reason, mixing electrolytics with MLCCs at the bulk stage is standard practice, not a sign of poor design.

**2. Deliberate ESR damping resistors.**

Add a small series resistor (5–30 m$\Omega$) with a bulk capacitor to raise its ESR. This damps the anti-resonance between bulk and MLCC stages at the cost of increasing DC rail drop.

**3. Staggered capacitor values that overlap in frequency.**

Rather than a sharp transition from one cap type to the next, use overlapping stages: e.g., 100 µF + 22 µF + 4.7 µF + 1 µF + 220 nF + 47 nF. Adjacent stages' SRFs overlap, smoothing the impedance profile and reducing anti-resonance peaks between stages.

**4. Reduce mounted ESL of the first "bridging" cap.**

The anti-resonance peak between stages is $|Z_{AR}| = \sqrt{L/C}$ where $L$ is the ESL of the upper (lower-frequency) stage and $C$ is the capacitance of the lower (higher-frequency) stage. Reducing $L$ (by using polymer-type bulk caps, or adding more bulk caps in parallel) directly lowers the peak.

**5. On-die regulation.**

An on-die LDO or low-dropout LDO with high PSRR provides an almost-infinite flatness at the die side — the LDO actively rejects noise regardless of PDN impedance shape. Modern SoCs use this extensively.

**Spectral workload awareness:**

Use simulation or measurement to characterise the load current spectrum under representative workloads. If the current spectrum has a pronounced tone at 100 MHz, the PDN target at 100 MHz is tighter than at other frequencies. Fidelity here avoids over-designing where it does not matter.

---

### Q5. What is the "Non-Invasive Stability Measurement" (NISM) and how is it used for PDN validation?

**Answer:**

NISM is a measurement technique pioneered by Steve Sandler (Picotest) for characterising PDN impedance with a **running live system** (not a passive impedance measurement). It measures the closed-loop output impedance of a regulated rail *as the actual load drives transient current*, capturing resonance peaks that only appear when the regulator is loaded.

**Why NISM is different from passive VNA measurement:**

A passive VNA measurement stimulates the rail with a small RF tone and measures the response. This captures the PDN's passive impedance with the regulator's closed-loop behaviour approximated as a linear passive element. For many designs this is sufficient. But:

1. A VRM under transient load can behave non-linearly (slew-rate limited, saturated output stage), causing the in-situ impedance to differ from the passive VNA-extracted value.
2. Digital load modulates the PDN — inductive mutual coupling from load current modulates the measured $Z(f)$ in subtle ways.
3. Under real workload, any intermodulation or non-linear regulation shows up as rail noise at frequencies the passive measurement would not predict.

NISM injects the stimulus *concurrent with* the live load and extracts the impedance from the rail's response. Picotest publishes NISM reference designs (OMICRON-Lab Bode 100 VNA with a coupling network) that are widely used in PI labs.

**When NISM matters:**

- High-performance digital parts where rail noise specs are tight.
- Validating that the operating-condition PDN impedance matches the design-intent PDN impedance.
- Confirming whether a mystery jitter spike at a specific frequency corresponds to a PDN resonance that only appears under load.

**When simpler measurements suffice:**

- Early-stage board bring-up where passive VNA measurement is adequate.
- Commercial consumer products where the jitter and rail noise budgets are loose.

---

### Q6. Walk through a PDN noise budget that includes flatness, AVP droop, SSO, and AC ripple.

**Answer:**

A senior-grade PDN budget looks like this:

| Component | Budget (mV) | Mechanism |
|---|---|---|
| DC tolerance | 20 | DC VRM set-point tolerance |
| AVP (adaptive voltage positioning) droop | 15 | Intentional load-line droop under load |
| Transient response (VRM loop) | 20 | VRM cannot respond faster than $1/f_{BW}$ |
| AC ripple | 5 | Switching-frequency ripple from buck converter |
| **Rail noise — quasi-static** | **60** | Sum of above (time-domain total) |
| PDN anti-resonance "rogue wave" | 10 | Flatness failure at one harmonic |
| Simultaneous switching output (SSO) | 10 | L·dI/dt ground bounce on I/O supplies |
| Plane-cavity noise | 3 | Plane resonance coupling |
| **Rail noise — AC** | **23** | Sum of dynamic contributions |
| **Total worst-case rail noise** | **83 mV** | |

For a 1.0 V nominal rail with 5% tolerance (±50 mV): this budget *exceeds* spec by 33 mV — the design would fail. The PI engineer must shrink one of the lines:

- Tighten the VRM (better AVP, wider loop BW)
- Improve decoupling to reduce the rogue wave (add OPD, damp the peak)
- Reduce SSO by improving package ball assignment

The **insight** is that the "rogue wave" line, which the target-impedance method alone does not produce, is a significant fraction of the dynamic budget. Ignoring it is the most common cause of "passed simulation but failed system test" PDN failures.

---

## Tier 3: Advanced

### Q7. How do you estimate the steady-state rogue-wave voltage for a PDN with a known anti-resonance, driven by a periodic load at that frequency?

**Answer:**

Model the PDN at the anti-resonance as a parallel-RLC tank with peak impedance $|Z_{peak}|$ and quality factor $Q$:

$$|Z_{peak}| = Q \cdot \sqrt{L/C}$$

Assume a periodic load current with fundamental at the anti-resonance frequency and amplitude $I_1$ (Fourier fundamental, peak value). Steady-state rail voltage amplitude at that frequency:

$$V_{peak} = |Z_{peak}| \cdot I_1 = Q \sqrt{L/C} \cdot I_1$$

**Numerical example:**

- PDN tank: $L = 1$ nH, $C = 100$ nF → $\sqrt{L/C} = 3.2$ m$\Omega$.
- Q = 10 (high-Q peak) → $|Z_{peak}| = 32$ m$\Omega$.
- Load current fundamental: 1 A peak at the resonance frequency.
- Steady-state rail voltage: $V = 32$ m$\Omega$ × 1 A = 32 mV peak.

If $Z_{target}$ had been set at 30 m$\Omega$ assuming a single-step load, the rail stays within 30 mV only for a single pulse. The periodic excitation at the resonance builds up to 32 mV — slightly above target. For higher-Q tanks or larger current, the multiplier grows linearly with $Q$.

**Damping the peak:**

To reduce the rogue wave, reduce $Q$. For a parallel-RLC tank, $Q = R/\sqrt{L/C}$ where $R$ is the equivalent parallel resistance. Adding ESR to the capacitor (raising $R$ makes the tank *more* lossy, lower $Q$) — but in a parallel tank, $Q$ is sometimes defined with $R$ as series equivalent, in which case $Q = 1/(R\sqrt{L/C})$ and higher $R$ lowers $Q$. Either way, adding dissipation damps the peak.

**Practical damping calculation:**

Target $Q \le 3$ for a well-damped PDN (flatness $\le 10$ dB). For $\sqrt{L/C} = 3.2$ m$\Omega$, need $R \le 10$ m$\Omega$ of parallel dissipation.

---

### Q8. Case study: a production board passes all passive PDN $|Z|$ measurements against $Z_{target}$ but shows mysterious rail noise of 80 mV during stress testing. Walk through the diagnostic approach.

**Answer:**

**Initial observation:** Rail spec is 30 mV; measured rail noise under stress test is 80 mV. VNA-measured $|Z|$ profile is within 10 m$\Omega$ everywhere (target is 10 m$\Omega$), so the passive measurement agrees with the design intent.

**Hypothesis 1: Workload current spectrum has resonance.**

Capture the rail noise waveform on an oscilloscope under stress test. FFT the waveform to identify dominant frequency components.

- If the noise peaks at a specific frequency — say 75 MHz — check whether $|Z(75\ \text{MHz})|$ is close to the target. If yes but the noise is much larger than $Z \cdot I_{DC}$, the culprit is a **narrowband periodic load** at 75 MHz resonating with the PDN.
- Load spectrum: common culprits include DDR access pattern (memory refresh at specific rates), clock harmonics, RSSO from core activity.

**Hypothesis 2: Rogue-wave build-up at anti-resonance.**

Check the $|Z|$ profile for any anti-resonance peak, even if it is below target. A sharp 9 m$\Omega$ peak at 75 MHz that the passive VNA measures as "passing" will nevertheless build up a rogue wave under sustained narrowband excitation.

**Hypothesis 3: Non-linear VRM behaviour.**

Verify with NISM measurement (active impedance with running load). If the NISM-measured $|Z|$ differs from the passive VNA at any frequency, the VRM is behaving non-linearly under load — possible slew-rate saturation or compensator instability.

**Hypothesis 4: Current spectrum was under-estimated.**

Reconcile the design-time $I_{peak}$ estimate with the measured $I$ under workload. Modern CPUs/GPUs often exceed datasheet $I_{typ}$ by 30–100% during AVX-like workloads; the PDN was sized for the typical case and over-stressed in practice.

**Hypothesis 5: Anti-resonance mismatch with simulation.**

Validate the PDN simulation against the measured $|Z|$. Often the measurement differs from simulation by 10–30% in peak location due to MLCC DC-bias derating, connector tolerances, or missing parasitics in the sim model.

**Resolution order:**

1. Measure rail noise FFT to identify frequency(ies) of concern.
2. Measure $|Z|$ (VNA or NISM) at those frequencies.
3. Adjust PDN (add damping, more decap, better ball assignment) to flatten any peaks in the problem band.
4. Re-stress-test.

**Lesson for interview:** A production PDN failure that cannot be explained by passive $|Z|$ meeting target is almost always a **flatness/rogue-wave/spectrum** problem. The classical $Z_{target}$ spec, taken alone, is a necessary but not sufficient condition.

---

## Summary: Target Impedance Is Necessary But Not Sufficient

| Classical method | Senior method |
|---|---|
| $|Z(f)| \le Z_{target}$ | $|Z(f)| \le Z_{target}$ **and** flatness $\le 6$ dB |
| Step-response $\Delta V$ budget | Step **and** periodic-excitation budget |
| Passive VNA measurement | Passive + NISM (live-load) measurement |
| Single $I_{max}$ value | Workload current spectrum |
| Lump all anti-resonance into "margin" | Explicit rogue-wave line in the budget |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Target impedance | $Z_{target} = \Delta V_{max}/I_{max}$ |
| Anti-resonance peak impedance | $|Z_{AR}| = Q \sqrt{L/C}$ |
| PDN flatness | $\Delta Z_{dB} = 20\log_{10}(|Z|_{peak}/|Z|_{min})$ |
| Rogue-wave steady-state voltage | $V_{ss} = |Z_{peak}| \cdot I_1\ \text{(fundamental)}$ |
| Damping resistor for $Q \le 3$ | $R_d \approx \sqrt{L/C} \cdot 3$ (series-equivalent) |
