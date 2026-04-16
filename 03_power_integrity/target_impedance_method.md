# Target Impedance Method

## Overview

The target impedance method is the primary quantitative framework for PDN design. Rather than designing from the supply side outward and hoping the result is "good enough", it works backward from a specified voltage tolerance on a power rail: given a maximum allowed voltage deviation and an expected maximum current step, it calculates the maximum PDN impedance that the network must not exceed anywhere in frequency. The entire PDN design — VRM bandwidth, bulk capacitor selection, MLCC count and placement — then flows from satisfying that impedance target.

This document covers the derivation of target impedance, its assumptions and limitations, how to extend it to a voltage ripple budget, and how real PDN design iterates around this method.

---

## Tier 1: Fundamentals

### Q1. What is the target impedance, and what is the fundamental formula?

**Answer:**

The target impedance $Z_{target}$ is the maximum allowable PDN impedance at any frequency, derived from the power rail's voltage tolerance and the maximum expected current transient:

$$Z_{target} = \frac{\Delta V_{max}}{I_{max}}$$

where:
- $\Delta V_{max}$ — the maximum allowed voltage deviation from the nominal rail voltage (volts)
- $I_{max}$ — the maximum expected current step that the load can demand (amperes)

**Example:** A 1.0 V core rail with $\pm$5% tolerance and a device that can step its current demand by 10 A:

$$\Delta V_{max} = 1.0\ V \times 5\% = 50\ mV$$

$$Z_{target} = \frac{50\ mV}{10\ A} = 5\ m\Omega$$

**Physical meaning:** If $|Z_{PDN}(f)| \leq Z_{target}$ for all frequencies where the load generates significant current, then the resulting voltage deviation satisfies $|\Delta V| \leq \Delta V_{max}$ at all those frequencies.

**Design implication:** The PDN must present an impedance at or below $Z_{target}$ across the entire frequency range of interest — typically from the VRM loop bandwidth up to at least the frequency corresponding to the fastest current transient in the system.

---

### Q2. How do you determine $\Delta V_{max}$ and $I_{max}$ for a real power rail?

**Answer:**

**Determining $\Delta V_{max}$:**

The allowable voltage deviation comes from the component datasheet or the system power supply specification:

$$\Delta V_{max} = V_{nom} \times \text{tolerance \%}$$

Common specifications:
- Core FPGA/ASIC rails: typically $\pm$3% to $\pm$5% of nominal
- DDR memory rails: $\pm$2% to $\pm$3% (tight, because they are not regulated on the IC)
- I/O rails: $\pm$5% to $\pm$10% (more relaxed)
- Analog/RF supplies: sometimes $\pm$1% or tighter

**Practical note:** Do not use the full tolerance budget for PDN noise alone. Budget it across contributors: DC accuracy of the VRM, thermal drift, PCB voltage drop under DC current, and AC ripple. A conservative allocation for AC PDN ripple might be 50% of the total tolerance. For example, if the total tolerance is $\pm$50 mV, allocate $\pm$25 mV to PDN AC ripple and use that as $\Delta V_{max}$.

**Determining $I_{max}$:**

This is the maximum current step the load can demand in a single transient event. Sources of this information:

1. **Device datasheet:** many FPGA and processor datasheets specify $dI/dt$ or a maximum current step magnitude
2. **Activity analysis:** estimate the fraction of logic that can switch simultaneously in the worst case
3. **Measurement:** capture the current waveform on existing hardware with a current probe
4. **Simulation:** SPICE or power integrity simulation with a behavioural current source model

**Conservative approach:** Use the device's total maximum current rating as $I_{max}$, assuming all current is demanded as a step. This is pessimistic but safe. In practice, not all current switches simultaneously — a detailed activity-based analysis gives a smaller (more achievable) target.

**Example for a mid-range FPGA:**
- Core voltage: 0.95 V, tolerance $\pm$3% → $\Delta V_{max} = 28.5\ mV$ (use 25 mV for PDN budget)
- Maximum core current: 8 A
- $Z_{target} = 25\ mV / 8\ A = 3.1\ m\Omega$

---

### Q3. What frequency range must the PDN impedance satisfy the target impedance?

**Answer:**

The target impedance must be satisfied from $f_{low}$ to $f_{high}$:

**Lower frequency bound $f_{low}$:**

Below the VRM loop bandwidth $f_{BW}$, the VRM actively regulates voltage and the closed-loop output impedance is much lower than $Z_{target}$. Therefore the target impedance effectively applies from the VRM loop bandwidth upward:

$$f_{low} \approx f_{BW,VRM}$$

Typical switching regulator loop bandwidths: 20–200 kHz for board-level VRMs; up to 1–5 MHz for integrated or multiphase point-of-load regulators.

**Upper frequency bound $f_{high}$:**

The upper bound is set by the fastest current transient in the system, specifically the "knee frequency" of the load current waveform:

$$f_{high} \approx \frac{0.35}{t_{rise}}$$

where $t_{rise}$ is the 10–90% rise time of the fastest current transient (not signal edge — current transient rise time).

For a digital ASIC with a clock edge triggering simultaneous switching, the current transient may have a rise time of 100 ps to a few nanoseconds:
- $t_r = 1\ ns \Rightarrow f_{high} \approx 350\ MHz$
- $t_r = 100\ ps \Rightarrow f_{high} \approx 3.5\ GHz$

Above $f_{high}$, the load current has negligible spectral content, so even if $|Z_{PDN}|$ rises above $Z_{target}$, no significant voltage deviation results. At those frequencies, on-die capacitance and package capacitance typically handle the demand.

**Practical design range:** For most FPGA and ASIC boards, the PDN must meet target impedance from roughly 100 kHz to 100–500 MHz. Above 100 MHz, PCB-mounted capacitors become less effective and the design relies on package/die capacitance.

---

### Q4. What are the key assumptions of the target impedance method, and when do they break down?

**Answer:**

**Assumption 1: The load is an ideal current source**

The method treats the IC as a current source that injects $\Delta I$ into the PDN. In reality, the IC input impedance is finite and can load the PDN. If the device's own impedance is comparable to $Z_{PDN}$, the actual voltage deviation is lower than $Z_{PDN} \cdot \Delta I$ predicts (the device "self-damps"). The current-source model is conservative — it overestimates $\Delta V$.

**Assumption 2: The PDN is linear and time-invariant**

MLCC capacitors (especially class II dielectrics X5R, X7R, Y5V) have significant voltage coefficient: their capacitance decreases with applied DC bias. A 100 nF X5R MLCC at 1.0 V bias may measure only 60–70 nF. This shifts the SRF upward and raises the impedance above what a linear model predicts. Always derate MLCC capacitance by the voltage coefficient when computing $f_{SRF}$.

**Assumption 3: The current step is instantaneous (worst case)**

The formula $Z_{target} = \Delta V_{max} / I_{max}$ implicitly assumes a step function in current, which has a flat frequency spectrum. A real current transient with finite rise time $t_r$ has a spectrum that rolls off above $0.35/t_r$. If the actual transient is known to have a specific rise time, a less conservative target can be used:

$$Z_{target}(f) = \frac{\Delta V_{max}}{I_{max}} \quad \text{for } f \leq 0.35/t_r$$

$$Z_{target}(f) = \frac{\Delta V_{max}}{I(f)} \quad \text{for } f > 0.35/t_r$$

where $I(f)$ is the actual current spectrum (which is falling).

**Assumption 4: Single worst-case current step**

The method addresses the amplitude of a single step. It does not account for periodic switching (which can cause resonant buildup if the switching frequency coincides with a PDN resonance) or correlated simultaneous switching from multiple devices.

**When the method breaks down:**
- Strongly inductive PDN with resonances sharply above $Z_{target}$: the transient response shows overshoot/undershoot even if the impedance magnitude barely touches $Z_{target}$
- Very high $dI/dt$ (GaN FETs switching in tens of nanoseconds): the frequency content extends to hundreds of MHz where the assumption of PCB-capacitor coverage fails
- On-die power management: some processors manage their own $dI/dt$ to reduce PDN demands — the datasheet "worst-case" $I_{max}$ may be pessimistic

---

## Tier 2: Intermediate

### Q5. Derive the target impedance from the ripple voltage budget. How does the voltage ripple budget allocate tolerances across contributors?

**Answer:**

**Voltage ripple budget derivation:**

The total voltage at the power pin of a device must stay within $[V_{nom} - \Delta V_{allowed}, V_{nom} + \Delta V_{allowed}]$. The total deviation has multiple contributors that must be summed (in worst case, all add coherently; in practice, they are root-sum-squared or statistically allocated):

$$\Delta V_{total} = \Delta V_{DC} + \Delta V_{line} + \Delta V_{thermal} + \Delta V_{AC}$$

where:
- $\Delta V_{DC}$ — VRM output regulation accuracy (DC setpoint error)
- $\Delta V_{line}$ — DC resistance drop through PCB traces and planes: $\Delta V_{line} = I_{DC} \times R_{PCB}$
- $\Delta V_{thermal}$ — VRM output change with temperature
- $\Delta V_{AC}$ — AC ripple from switching transients (what the PDN method addresses)

**Worst-case allocation example:**

| Contributor | Allowance |
|---|---|
| VRM DC accuracy | 10 mV |
| PCB resistance drop | 5 mV |
| Thermal drift | 5 mV |
| **AC PDN ripple** | **25 mV** |
| **Total (sum)** | **45 mV** |

For a $\pm$50 mV tolerance budget, the 45 mV sum leaves 5 mV margin.

**RSS allocation (statistical, less conservative):**

$$\Delta V_{total,RSS} = \sqrt{\Delta V_{DC}^2 + \Delta V_{line}^2 + \Delta V_{thermal}^2 + \Delta V_{AC}^2}$$

Using values above: $\Delta V_{RSS} = \sqrt{100 + 25 + 25 + 625} = \sqrt{775} \approx 27.8\ mV$

RSS allocation gives more headroom for PDN ripple — sometimes the AC allowance is increased to $\pm$40 mV when using RSS.

**Target impedance from the AC allowance:**

$$Z_{target} = \frac{\Delta V_{AC}}{I_{step,max}}$$

The smaller the AC allowance, the tighter the target impedance. This is why tight voltage tolerances on modern ICs (often $\pm$3%) drive increasingly difficult PDN requirements.

---

### Q6. How do you account for multiple capacitor stages in meeting the flat target impedance? Describe the design cascade methodology.

**Answer:**

The flat target impedance method requires that $|Z_{PDN}(f)| \leq Z_{target}$ at every frequency from $f_{BW,VRM}$ to $f_{high}$. Achieving this with a single capacitor type is impossible because of SRF limitations. The design uses a cascade of stages, each responsible for a frequency decade:

**Design cascade procedure:**

**Stage 0: VRM coverage**
The VRM must provide $|Z_{PDN}| \leq Z_{target}$ up to its loop bandwidth $f_{BW}$. This sets the minimum requirement on VRM loop bandwidth:

$$f_{BW} \geq \frac{1}{2\pi} \cdot \frac{Z_{target}}{L_{filter}}$$

where $L_{filter}$ is the VRM output filter inductance. If the VRM cannot achieve $Z_{target}$ at $f_{BW}$ with reasonable bandwidth, the filter inductance must be reduced.

**Stage 1: Bulk capacitance (covers $f_{BW}$ to ~100 kHz)**

Required capacitance to hold impedance below $Z_{target}$ at the VRM loop bandwidth crossover:

$$C_{bulk,min} = \frac{1}{2\pi f_{BW} \cdot Z_{target}}$$

Select bulk capacitors (electrolytic or polymer) with total $C > C_{bulk,min}$ and ESR $< Z_{target}$.

**Stage 2: Mid-frequency ceramics (covers ~100 kHz to ~10 MHz)**

The SRF of the MLCC array must fall within this range. Required capacitance:

$$C_{MLCC,min} = \frac{1}{2\pi f_{1} \cdot Z_{target}}$$

where $f_1$ is the lower end of this stage's coverage range. Select MLCCs such that the parallel ESR $< Z_{target}$ and the SRF array covers the required band.

**Stage 3: Small ceramics near device (covers ~10 MHz to ~500 MHz)**

Small-value, small-case-size (0201 or 01005) MLCCs placed immediately adjacent to device power pins. Each has low $L_{ESL}$ (~0.3–0.5 nH) giving SRF in the 50–300 MHz range.

**Anti-resonance check:**

After defining each stage, check all adjacent-stage pairs for anti-resonance:

$$f_{AR} \approx \frac{1}{2\pi\sqrt{L_{N,ESL} \cdot C_{N+1}}}$$

$$|Z_{AR}| \approx \sqrt{\frac{L_{N,ESL}}{C_{N+1}}}$$

If $|Z_{AR}| > Z_{target}$, add series damping (a resistor in series with the bulk stage) or increase the overlap between stages.

**Design iteration:** In practice, a power integrity simulation tool (Ansys SIwave, Cadence PowerDC/Clarity, or a SPICE model of the PDN) is used to verify that the full impedance profile stays below $Z_{target}$. The analytical cascade design is the starting point.

---

### Q7. What is the "inductance-limited" regime of PDN design, and what does it imply about capacitor count?

**Answer:**

As the design pushes to higher frequencies, an important transition occurs: the dominant impedance contributor shifts from capacitance (which can be increased by adding more capacitors) to inductance (which is fixed by geometry and placement).

**Capacitance-limited regime** (lower frequencies):

$|Z| = 1/(2\pi f C_{total})$. Doubling the number of capacitors halves the impedance. Target impedance can always be met by adding more capacitors of the appropriate type. This is the regime where "add more decaps" is a valid fix.

**Inductance-limited regime** (above the frequency where $L_{ESL,total}$ dominates):

$|Z| = 2\pi f \cdot (L_{ESL}/N)$. Below a certain threshold, adding capacitors helps because it reduces $L_{ESL,total} = L_{ESL}/N$. However, there is a crossover point where the *spreading inductance* through the PCB planes becomes the dominant series inductance, not the capacitor's own $L_{ESL}$.

Above this crossover, adding more capacitors with identical placement does nothing — they all connect through the same inductive path to the device. The impedance is then set entirely by:

$$Z_{floor}(f) = 2\pi f \cdot L_{spreading}$$

**Critical frequency:** The transition occurs at the frequency where placing a capacitor one additional millimetre further from the device adds more inductance than having no capacitor at that location. For a 2 mil (50 µm) dielectric, the plane inductance per millimetre of path is roughly:

$$L_{per\_mm} = \mu_0 \cdot h / w \approx 4\pi\times10^{-7} \times 50\times10^{-6} / 1\times10^{-3} \approx 0.06\ pH/mm/mm$$

This is small but accumulates over centimetres.

**Design implication:** When calculations show that meeting $Z_{target}$ requires an impractically large number of capacitors (hundreds or thousands), the PDN is in the inductance-limited regime. The correct solution is:

1. Reduce board-level inductance (thinner dielectric, more via pairs)
2. Move capacitors closer (or inside the BGA footprint, if allowed)
3. Use on-package decoupling capacitors
4. Accept a higher $Z_{target}$ by relaxing the voltage tolerance or reducing $I_{step}$

---

### Q8. How does the target impedance method extend to multiple power rails that share a ground reference?

**Answer:**

When multiple power rails share a common ground plane, noise from one rail can couple to adjacent rails through the shared return path. This introduces a transfer impedance consideration beyond the self-impedance target.

**Transfer impedance $Z_{21}$:**

The voltage deviation on victim rail 2 caused by a current step on aggressor rail 1:

$$\Delta V_2(f) = Z_{21}(f) \cdot \Delta I_1(f)$$

If the rails share a ground plane with finite impedance $Z_{GND}$, then:

$$Z_{21}(f) \approx Z_{GND}(f)$$

This is typically small (milliohms) at DC but can rise at high frequencies if the ground return path between the two rails has significant inductance.

**Multi-rail target impedance procedure:**

For each rail $k$ with voltage tolerance $\Delta V_{k,max}$ and maximum current step $\Delta I_k$:

1. Set self-impedance target: $Z_{kk,target} = \Delta V_{k,max} / \Delta I_k$
2. Set cross-coupling budget: allocate a fraction $\alpha$ of $\Delta V_{k,max}$ to noise coupling from other rails: $\Delta V_{k,cross} = \alpha \cdot \Delta V_{k,max}$
3. Require: $Z_{jk}(f) \cdot \Delta I_j \leq \Delta V_{k,cross}$ for all aggressors $j \neq k$

**Practical approach:** For most designs, cross-rail coupling through the PDN is a second-order effect compared to self-impedance. The primary concern is adjacent power islands on the same ground plane where high-current switching regulators share the ground return with sensitive analog or DDR supplies. In such cases:
- Use separate ground pour areas or stitching capacitors to control current paths
- Simulate the transfer impedance between sensitive rails and switching nodes
- Verify that the floor of the common ground return impedance is below the cross-coupling budget

---

## Tier 3: Advanced

### Q9. Derive the relationship between VRM loop bandwidth and the maximum bulk capacitance size, showing the stability trade-off.

**Answer:**

A switching regulator with an output capacitor $C_{bulk}$ and output filter inductor $L$ forms a resonant LC output filter. The converter's control loop must be designed so that the phase margin remains positive at the crossover frequency.

**Output filter transfer function (voltage mode, simplified):**

$$G_{filter}(s) = \frac{1/LC}{s^2 + s(R_{ESR}/L) + 1/(LC)}$$

This has a double pole at $\omega_0 = 1/\sqrt{LC}$ and a zero at $\omega_{ESR} = 1/(R_{ESR} C)$.

**Loop stability constraint:**

The crossover frequency $f_c$ (where the loop gain $|T(j\omega_c)| = 1$) must be lower than the switching frequency $f_{sw}$ by at least a factor of 5–10:

$$f_c \leq \frac{f_{sw}}{5}$$

For voltage mode control, the compensator must provide phase boost to overcome the 180° phase shift from the double pole. The maximum achievable loop bandwidth while maintaining 45° phase margin is approximately:

$$f_c \leq \frac{1}{2\pi} \cdot \frac{1}{\sqrt{LC}} \cdot \left(\text{function of ESR and compensation}\right)$$

**The bulk capacitance constraint:**

The target impedance requires:

$$C_{bulk} \geq \frac{1}{2\pi f_c \cdot Z_{target}}$$

Substituting a maximum $f_c = f_{sw}/5$ for a 500 kHz switcher ($f_c \leq 100\ kHz$):

$$C_{bulk} \geq \frac{1}{2\pi \times 100\ kHz \times Z_{target}}$$

For $Z_{target} = 5\ m\Omega$: $C_{bulk} \geq \frac{1}{2\pi \times 10^5 \times 5\times10^{-3}} = \frac{1}{3142} \approx 318\ \mu F$

**The tension:** Increasing $C_{bulk}$ lowers the LC double-pole frequency, making the loop more difficult to compensate and reducing achievable bandwidth. Reducing $L$ raises the double-pole frequency (improving compensability) but increases switching current ripple. The design must balance:

$$f_c \propto \frac{1}{\sqrt{LC_{bulk}}} \quad \text{(loop bandwidth from LC resonance)}$$

$$Z_{target} \propto \frac{1}{C_{bulk} \cdot f_c} \quad \text{(target impedance from bulk capacitance)}$$

Combining: $Z_{target} \propto \sqrt{L/C_{bulk}}$ — target impedance is the characteristic impedance of the LC filter. To achieve very low $Z_{target}$, you need a high $C/L$ ratio, which requires either very large capacitance or very small inductance. Modern multiphase regulators achieve both by using many parallel phases with small per-phase inductors.

---

### Q10. How do you use the target impedance method to size a multi-phase VRM? Walk through the phase count, inductance, and capacitance determination.

**Answer:**

A multi-phase VRM interleaves $N$ converter phases, each switching at $f_{sw}$. The effective switching frequency seen by the output capacitors is $N \cdot f_{sw}$, which raises the achievable loop bandwidth and reduces ripple current.

**Step 1: Determine required output impedance at DC**

From the datasheet load regulation requirement:

$$R_{out,DC} = \frac{\Delta V_{DC,max}}{I_{max}}$$

For $\Delta V_{DC} = 10\ mV$, $I_{max} = 40\ A$: $R_{out,DC} = 0.25\ m\Omega$. This requires either a high DC gain in the control loop or active droop/load line compensation.

**Step 2: Target impedance drives loop bandwidth**

$$f_{BW,min} = \frac{1}{2\pi} \cdot \frac{Z_{target}}{L_{phase}/N}$$

where $L_{phase}$ is the per-phase inductor. Rearranging for required inductance:

$$L_{phase} = \frac{N \cdot Z_{target}}{2\pi \cdot f_{BW,target}}$$

For $N = 4$, $Z_{target} = 5\ m\Omega$, $f_{BW} = 200\ kHz$:

$$L_{phase} = \frac{4 \times 5\times10^{-3}}{2\pi \times 200\times10^3} = \frac{0.02}{1.257\times10^6} \approx 15.9\ nH$$

This is a very small inductor, consistent with high-current integrated voltage regulators.

**Step 3: Bulk capacitance to bridge VRM response time**

The capacitance needed to supply the load during the VRM response time $\tau = 1/f_{BW}$:

$$Q = I_{step} \cdot \tau = I_{step} / f_{BW}$$

The voltage sag during this time:

$$\Delta V = Q / C_{bulk} \leq \Delta V_{max}$$

$$C_{bulk} \geq \frac{I_{step}}{f_{BW} \cdot \Delta V_{max}}$$

For $I_{step} = 20\ A$, $f_{BW} = 200\ kHz$, $\Delta V_{max} = 25\ mV$:

$$C_{bulk} \geq \frac{20}{2\times10^5 \times 25\times10^{-3}} = \frac{20}{5000} = 4\ mF$$

This confirms the need for large total bulk capacitance (e.g., four 1 mF polymer capacitors).

**Step 4: MLCC count for mid-frequency coverage**

Each frequency decade above the bulk SRF up to the device's knee frequency requires MLCC coverage. For a 100 MHz knee frequency with MLCCs having $f_{SRF} \approx 10\ MHz$ and $R_{ESR} = 20\ m\Omega$ each:

$$N_{MLCC} = \frac{R_{ESR,single}}{Z_{target}} = \frac{20\ m\Omega}{5\ m\Omega} = 4$$

Minimum 4 MLCCs in parallel to bring ESR at SRF below $Z_{target}$. In practice, more are used to achieve lower impedance and provide margin against anti-resonance peaks.

---

### Q11. What are the limitations of the scalar target impedance method for modern ICs with complex current profiles?

**Answer:**

The scalar target impedance method ($Z_{target} = \Delta V_{max} / I_{max}$) applies a single flat impedance constraint across the entire frequency band. Modern ICs violate several of its key assumptions in ways that require more sophisticated analysis.

**Limitation 1: Frequency-dependent current spectrum**

Real device current spectra are not flat. A device with a 1 GHz clock generates current at 1 GHz, 2 GHz, 3 GHz, etc. but with an envelope that falls off at higher harmonics. A flat $Z_{target}$ is excessively conservative at high frequencies. A better formulation uses a frequency-shaped target:

$$Z_{target}(f) = \frac{\Delta V_{max}}{|\hat{I}(f)|}$$

where $|\hat{I}(f)|$ is the current spectral envelope of the actual device.

**Limitation 2: Simultaneous resonances**

If a PDN has a resonance peak exactly at a harmonic of the device clock or memory bus frequency, the PDN acts as an amplifier for that specific frequency component. Even if the peak impedance only slightly exceeds $Z_{target}$, the sustained resonant excitation can build up to a larger voltage amplitude than the single-step analysis predicts. This requires a periodic-steady-state or frequency-domain analysis rather than a step-response analysis.

**Limitation 3: On-die power management (DVFS)**

Dynamic voltage and frequency scaling (DVFS) changes the supply voltage and clock frequency in real time. The worst-case current step may occur during a DVFS transition, not during steady-state operation at maximum frequency. The effective $Z_{target}$ during a transition may be different from the steady-state value.

**Limitation 4: Die-level voltage regulator impedance**

Modern server processors and FPGAs include on-die or in-package voltage regulators (integrated voltage regulators, IVR). These have their own loop dynamics that interact with the board-level PDN. The simple $Z_{target}$ calculation ignores this interaction. The stability of the IVR depends on the impedance it sees looking into the board PDN.

**Limitation 5: Non-linear capacitor behaviour**

MLCC capacitance varies with DC bias (voltage coefficient), temperature, and age. Under full DC bias, X5R capacitors can lose 50–80% of their nominal capacitance. This shifts $f_{SRF}$ and raises the minimum impedance, potentially violating the target. Design must derate capacitor values accordingly.

**Modern approach:** PDN design for advanced ICs combines the target impedance method as a first-cut tool with full-wave 3D EM simulation of the package and PCB, time-domain power integrity simulation with actual device current profiles (from chip simulation or measurement), and on-die PDN models extracted from the silicon designers.

---

### Q12. The target impedance formula $Z_{target} = \Delta V_{max}/I_{max}$ is really a statement about steady-state sinusoidal excitation. Explain what this means and why the formula can still be applied to time-domain current transients.

**Answer:**

The target impedance is defined in the frequency domain as a bound on the magnitude of the PDN's small-signal impedance:

$$|Z_{PDN}(f)| \leq Z_{target} \quad \text{for all } f \in [f_{low}, f_{high}]$$

Impedance, as a concept, is a **steady-state sinusoidal** quantity. Writing $Z_{PDN}(f) = \Delta V(f) / \Delta I(f)$ presumes that we are injecting a continuous-wave sinusoidal current $\Delta I(f) e^{j 2\pi f t}$ and measuring the resulting sinusoidal voltage perturbation $\Delta V(f) e^{j 2\pi f t}$ at the same frequency, after any transient has decayed. It is not a direct statement about time-domain step or pulse excitation.

**Why the time-domain interpretation still holds (approximately):**

For a linear, time-invariant PDN, the time-domain voltage response to any current waveform $i(t)$ is the inverse Fourier transform:

$$\Delta v(t) = \mathcal{F}^{-1}\{ Z_{PDN}(f) \cdot I(f) \}$$

Peak time-domain voltage is bounded by:

$$|\Delta v(t)|_{peak} \leq \int_{-\infty}^{\infty} |Z_{PDN}(f)| \cdot |I(f)|\, df$$

If $|Z_{PDN}(f)| \leq Z_{target}$ across the full band where $I(f)$ has significant energy, and if $|I(f)| \leq I_{max}$ as a scalar-bound approximation, then the peak time-domain voltage is controlled. But this bound is **loose** — it is a triangle-inequality upper limit, and the actual peak may be much smaller, or in some cases much larger, than the scalar product $Z_{target} \cdot I_{max}$.

**The hidden assumption:**

The formula $Z_{target} = \Delta V_{max}/I_{max}$ implicitly treats $I_{max}$ as the peak of a current spectrum that is approximately flat within the band of interest — which is true for a clean step function. A step's spectrum is $|I(f)| = I_{max}/(2\pi f)$, falling at 20 dB/decade from DC. After accounting for the $1/f$ spectral rolloff, the formula gives:

$$|\Delta v(t)|_{peak,step} \sim Z_{target} \cdot I_{max}$$

only when $|Z_{PDN}|$ rises with frequency at 20 dB/decade or less — which happens to be true for a well-designed flat PDN. So the approximation works in practice *because* good PDN profiles are flat, not because the formula is tight for arbitrary impedance shapes.

**Practical conclusion:**

The target impedance method is a **sufficient** (not tight) design criterion. Meeting $|Z_{PDN}(f)| \leq Z_{target}$ everywhere in band guarantees the time-domain voltage stays within spec for any single current step of magnitude $\leq I_{max}$ with rise time slower than $1/(2\pi f_{high})$. However, it does *not* automatically guarantee robustness against sustained periodic current excitation — especially at PDN resonant frequencies (see Q13).

---

### Q13. Describe the worst-case real-world current waveform that fully excites a PDN resonance, and explain why a resonant peak modestly above $Z_{target}$ can still cause a spec violation that a single-step analysis does not predict.

**Answer:**

The target impedance method implicitly models the load current as a single step (or a single isolated transient). A sustained, *periodic* current excitation with a harmonic landing exactly on a PDN resonance is a fundamentally worse excitation than a single step — and it is commonplace in real systems.

**The worst-case excitation:**

A periodic square-wave current drawn at frequency $f_{res}$ (or any integer sub-harmonic whose odd-numbered harmonics align with $f_{res}$), where $f_{res}$ is the frequency of a PDN impedance peak.

Periodic excitation at a resonant frequency, applied over many cycles, drives the resonator to its **steady-state amplitude**, which for a parallel-LC resonance with quality factor $Q$ is:

$$|\Delta V|_{steady-state} = Q \cdot Z_{DC,avg} \cdot I_{harmonic}$$

In impedance terms, $|Z_{PDN}(f_{res})| = Q \cdot Z_{avg}$ — the quality factor itself sets how high the resonant peak rises above the surrounding impedance. A $Q = 10$ resonance driven by a current harmonic equal to $0.1 \cdot I_{max}$ generates the *same* voltage perturbation as a broadband current of amplitude $I_{max}$ against a flat $Z_{target}$. In other words, a modest resonant peak can be as damaging as a broadband excitation at ten times the spectral amplitude.

**Why this is worse than a single step:**

A single current step has its spectral energy spread across all frequencies from DC to $f_{knee} = 0.5/t_r$. Only the small fraction of spectral energy that lies near $f_{res}$ excites the resonance, and only briefly — the resonator's ringing decays with time constant $\tau = Q/(2\pi f_{res})$. Peak voltage is bounded and the transient is short.

A periodic excitation continuously pumps energy into the resonator cycle after cycle, until steady-state is reached after approximately $Q$ cycles. The resonator does not ring *once*; it rings *indefinitely* at the full steady-state amplitude.

**Concrete example:**

Consider a PDN with:
- Flat impedance $|Z| = 3\,\text{m}\Omega$ across most of the band
- Anti-resonant peak $|Z|_{peak} = 15\,\text{m}\Omega$ (a $5\times$ rise) at $f_{res} = 200\,\text{MHz}$, with $Q \approx 5$
- $Z_{target} = 5\,\text{m}\Omega$

**Single-step analysis:** A 10 A current step contains approximately $10 \cdot (1/2\pi \cdot 0.01)$ A of energy in a 1% band around 200 MHz — call it 0.16 A. The resonance briefly rings at $0.16 \cdot 15\,\text{m}\Omega = 2.4\,\text{mV}$. Below $\Delta V_{max} = 50\,\text{mV}$, so the single-step criterion suggests acceptable behaviour.

**Periodic-excitation analysis:** Now assume the device has a 200 MHz on-die regulator or memory interface generating a repetitive 1 A current harmonic at exactly 200 MHz. Steady-state voltage:

$$|\Delta V|_{steady-state} = 1\,\text{A} \cdot 15\,\text{m}\Omega = 15\,\text{mV}$$

Now suppose the harmonic grows to 3.5 A (common for high-activity data phases):

$$|\Delta V|_{steady-state} = 3.5\,\text{A} \cdot 15\,\text{m}\Omega = 52.5\,\text{mV} \quad (> \Delta V_{max})$$

**Spec violated**, even though the resonant peak was only 3× above $Z_{target}$ and the exciting current (3.5 A) was only 35% of the single-step $I_{max}$. The resonance amplifies what would otherwise be a modest, in-band current into a full-amplitude violation.

**Why this happens in real systems:**

1. **Clock harmonics align with PDN resonances.** If a digital bus switches at 400 MHz with 50% duty cycle, its current spectrum has strong content at 400 MHz, 1.2 GHz, 2.0 GHz, etc. Any PDN resonance landing on one of these frequencies gets fully excited.

2. **DDR memory bus activity.** During write bursts, synchronous switching of many DQ/DQS lines generates current spectra with strong content at the data rate and its harmonics.

3. **On-die switching regulator noise.** Integrated voltage regulators on modern SoCs switch at frequencies (tens to hundreds of MHz) that fall within the band where the board PDN has resonances.

4. **Correlated simultaneous switching (SSO/SSN).** When many I/O drivers switch together at the bus clock rate, the aggregate current has a periodic spectral content that can hit PDN peaks.

**Design mitigation:**

Because periodic excitation at PDN resonances is fundamentally worse than a single step, defensive PDN design demands:

1. **Lower the Q of anti-resonant peaks** via intentional damping (series resistance in bulk capacitor stage, ferrite beads, or resistive terminations). Lower Q spreads the peak over a wider frequency range and reduces its height.

2. **Shift resonances away from known harmonics.** If simulation shows a 400 MHz peak and the system runs a 400 MHz bus, add bypass capacitors to shift the peak to 300 MHz or 500 MHz.

3. **Time-domain PI simulation with realistic current profiles.** Feed an actual or modelled device current waveform (not a step) into a full PDN model. This catches resonance-driven violations that frequency-domain impedance checks miss.

4. **Tighten the target impedance.** Some engineering shops apply a factor of 2–5× margin on $Z_{target}$ specifically to cover periodic-excitation amplification that scalar analysis underestimates.

**Common interview pitfall:** A candidate who answers that "as long as $|Z_{PDN}(f_{res})| < Z_{target}$, the design is fine" has only demonstrated single-step sufficiency. A strong answer acknowledges that $Z_{target}$ is a frequency-domain sinusoidal bound and that the time-domain worst case is sustained resonant excitation, which is independently bounded only by PDN Q.

---

## Summary: Target Impedance Design Flow

```
1. Identify power rail: V_nom, tolerance, I_step_max
          |
          v
2. Allocate voltage budget: delta_V_AC = fraction of total tolerance
          |
          v
3. Calculate Z_target = delta_V_AC / I_step_max
          |
          v
4. Determine frequency range: f_low = VRM BW, f_high = 0.35/t_rise
          |
          v
5. Design VRM: loop BW >= f_low, output Z <= Z_target at f_low
          |
          v
6. Design bulk caps: C >= 1/(2*pi*f_low*Z_target), ESR < Z_target
          |
          v
7. Design MLCC stages: cover f_low to f_high with SRF overlap
          |
          v
8. Check anti-resonance peaks between stages
          |
          v
9. Simulate full PDN impedance; iterate if peaks exceed Z_target
          |
          v
10. Verify with board measurement (VNA 2-port or 1-port)
```

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Target impedance | $Z_{target} = \Delta V_{max} / I_{step}$ |
| Minimum bulk capacitance | $C_{min} = 1 / (2\pi f_{BW} \cdot Z_{target})$ |
| Minimum MLCC count (ESR) | $N = R_{ESR,single} / Z_{target}$ |
| VRM bandwidth requirement | $f_{BW} \geq 1/(2\pi C_{bulk} Z_{target})$ |
| Current step knee frequency | $f_{knee} = 0.35 / t_{rise}$ |
| Anti-resonance check | $|Z_{AR}| = \sqrt{L_{bulk,ESL}/C_{MLCC}}$ must be $< Z_{target}$ |
| Multi-phase inductor sizing | $L_{phase} = N \cdot Z_{target} / (2\pi f_{BW})$ |
