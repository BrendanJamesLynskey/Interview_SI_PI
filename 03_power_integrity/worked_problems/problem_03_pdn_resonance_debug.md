# Problem 03: Debugging a PDN Resonance Issue on a Real Board

## Problem Statement

You are a power integrity engineer on a team that has just received first-pass hardware for a new PCIe Gen 4 network adapter card. The card uses a Xilinx UltraScale+ FPGA with a 0.85 V core supply and a 1.8 V I/O supply. The board has been designed with what appears to be an adequate PDN, but initial bring-up reveals the following failures:

**Observed symptoms:**

1. The FPGA core occasionally generates uncorrectable ECC errors during sustained PCIe traffic. The errors are intermittent and dependent on traffic pattern.
2. Probing the 0.85 V core rail with an oscilloscope shows periodic voltage spikes of approximately 60–80 mV amplitude. The spikes have a dominant spectral component at 312 MHz when measured with a spectrum analyser connected to the power rail via a probe.
3. The 1.8 V I/O rail is clean — no anomalies observed.
4. The errors disappear when the FPGA clock frequency is reduced from 250 MHz to 200 MHz.
5. A thermal camera shows the FPGA at 72°C junction temperature under load, within specification.

**PDN design on the board:**

- 0.85 V core supply: 3-phase buck VRM, 500 kHz per phase, loop bandwidth approximately 120 kHz
- Bulk decoupling: six 470 µF polymer capacitors placed near the FPGA (within 20 mm)
- Mid-frequency decoupling: eight 10 µF 0402 X5R MLCCs placed around the BGA periphery
- High-frequency decoupling: sixteen 100 nF 0402 X7R MLCCs placed around the BGA periphery
- Board dimensions: 100 mm × 60 mm
- PCB stackup: 8-layer board, 1.6 mm total thickness; power and ground planes are layers 5 and 6, separated by 4 mil (102 µm) of FR4
- FR4 dielectric constant: $\varepsilon_r = 4.2$ (at 1 GHz)

**Your available debug tools:**

- 2-channel oscilloscope, 2 GHz bandwidth
- Spectrum analyser
- VNA (vector network analyser), 3 GHz maximum frequency
- Soldering station and spare components

**Tasks:**

**(a)** Based on the symptoms, form a hypothesis about the root cause of the PDN failure. What is the likely mechanism linking the 312 MHz spectral component to the observed errors?

**(b)** Calculate the power plane resonant frequencies for the 0.85 V power plane (assuming it occupies approximately 80 mm × 55 mm of the board area) and determine if a resonant mode coincides with the 312 MHz frequency.

**(c)** Explain why the errors disappear when the clock frequency is reduced from 250 MHz to 200 MHz.

**(d)** Describe the measurement procedure to confirm the PDN resonance hypothesis using the VNA. What impedance profile do you expect to see?

**(e)** Calculate the target impedance for the 0.85 V core rail. Assume the FPGA datasheet specifies a maximum current step of 8 A and the tolerance is $\pm 3\%$.

**(f)** Identify three independent design changes that would address the root cause. For each, predict its effect on the resonance.

**(g)** After implementing a fix and re-measuring, you observe that the 312 MHz resonance peak has been reduced from 45 m$\Omega$ to 8 m$\Omega$. Does this resolve the issue? Show your work.

---

## Solution

### Part (a): Root Cause Hypothesis

**Symptom analysis:**

The dominant 312 MHz spectral component on the core rail, combined with errors that depend on clock frequency (disappear at 200 MHz, present at 250 MHz), strongly suggests a **power plane cavity resonance excited by the FPGA's switching current at a harmonic of the clock frequency**.

**Mechanism:**

1. The FPGA running at 250 MHz generates current switching events at 250 MHz and all harmonics: 500 MHz, 750 MHz, etc. However, the current transients from internal logic switching and PCIe data patterns may have spectral content concentrated at specific frequencies.

2. The 312 MHz spectral component is $\approx 1.25 \times 250\ MHz$ — not an exact harmonic. However, DDR interfaces, SERDES clocking, and internal PLL harmonics can create beat frequencies. More likely: the FPGA's internal fabric switching frequency creates a significant harmonic at or near 312 MHz, which corresponds to the 5th harmonic of 62.5 MHz (a common reference/SERDES frequency in PCIe Gen 4) or the secondary clock domain inside the FPGA.

3. The 312 MHz spectral component coincides with a **power plane resonance mode** that presents high impedance. The switching current at 312 MHz flows through this high impedance, producing a large $\Delta V = Z_{resonance} \times \Delta I$ — the 60–80 mV spike observed.

4. **ECC errors:** The core voltage spike exceeding the $\pm 3\%$ tolerance (25.5 mV) on the 0.85 V rail causes timing violations in the FPGA fabric — setup time failures or hold time failures in sequential logic, appearing as uncorrectable bit errors.

5. **Clock frequency dependence:** The harmonic of 250 MHz that coincides with the 312 MHz resonance does not exist at 200 MHz harmonics — explaining why reducing the clock eliminates the resonance excitation.

**Primary hypothesis:** A power plane cavity resonance at or near 312 MHz is being excited by a specific harmonic of the FPGA operating frequency. The resonance peak presents an impedance well above the target, causing voltage deviation that exceeds the core tolerance.

---

### Part (b): Plane Resonance Frequency Calculation

**Given:**
- Power plane dimensions: $a = 80\ mm$, $b = 55\ mm$
- Dielectric constant: $\varepsilon_r = 4.2$
- Phase velocity in FR4: $v_{ph} = c/\sqrt{\varepsilon_r} = 3\times10^8/\sqrt{4.2} = 3\times10^8/2.049 = 1.464\times10^8\ m/s$

**Resonant frequency formula:**

$$f_{mn} = \frac{v_{ph}}{2}\sqrt{\left(\frac{m}{a}\right)^2 + \left(\frac{n}{b}\right)^2}$$

**Mode (1, 0) — dominant mode along the long dimension:**

$$f_{10} = \frac{1.464\times10^8}{2 \times 0.080} = \frac{1.464\times10^8}{0.160} = 915\ MHz$$

**Mode (0, 1) — dominant mode along the short dimension:**

$$f_{01} = \frac{1.464\times10^8}{2 \times 0.055} = \frac{1.464\times10^8}{0.110} = 1.331\ GHz$$

**Mode (1, 1):**

$$f_{11} = \frac{1.464\times10^8}{2}\sqrt{\left(\frac{1}{0.080}\right)^2 + \left(\frac{1}{0.055}\right)^2}$$

$$= 0.732\times10^8 \times \sqrt{156.25 + 330.6} = 0.732\times10^8 \times \sqrt{486.9} = 0.732\times10^8 \times 22.07$$

$$= 1.615\times10^9\ Hz = 1.615\ GHz$$

**Mode (2, 0):**

$$f_{20} = \frac{1.464\times10^8}{2} \times \frac{2}{0.080} = \frac{1.464\times10^8}{0.080} = 1.830\ GHz$$

**Mode (2, 1):**

$$f_{21} = \frac{1.464\times10^8}{2}\sqrt{\left(\frac{2}{0.080}\right)^2 + \left(\frac{1}{0.055}\right)^2} = 0.732\times10^8\sqrt{625 + 330.6}$$

$$= 0.732\times10^8 \times 30.9 = 2.262\ GHz$$

**Summary of resonant modes:**

| Mode $(m,n)$ | Resonant frequency |
|---|---|
| (1, 0) | 915 MHz |
| (0, 1) | 1331 MHz |
| (1, 1) | 1615 MHz |
| (2, 0) | 1830 MHz |
| (2, 1) | 2262 MHz |

**Assessment:** None of the calculated modes falls near 312 MHz. The fundamental resonance of this plane is at 915 MHz.

**Revised hypothesis:** The 312 MHz feature is not a plane resonance of the full power plane. It must be one of:

1. **A resonance of a smaller power plane segment** — if the power plane has cutouts, slots, or is divided into sub-regions by anti-pads around vias, a smaller effective cavity dimension can produce a lower resonant frequency. For $f_{mn} = 312\ MHz$ as a (1,0) mode: $a = v_{ph}/(2 \times f) = 1.464\times10^8/(2\times312\times10^6) = 235\ mm$ — too large. For a (2,0) mode: $a = v_{ph}/f = 1.464\times10^8/312\times10^6 = 469\ mm$ — also too large.

2. **An anti-resonance between the bulk capacitor stage ESL and the MLCC stage capacitance** — more likely at 312 MHz given the capacitor values on this board:

$$f_{AR} = \frac{1}{2\pi\sqrt{L_{bulk,eff} \times C_{MLCC,total}}}$$

For 6 × 470 µF polymer caps with $L_{ESL} = 4\ nH$ each: $L_{bulk,eff} = 4/6 = 0.667\ nH$

For 8 × 10 µF X5R MLCCs (derated to 8.5 µF each): $C_{MLCC} = 68\ \mu F$

$$f_{AR,1} = \frac{1}{2\pi\sqrt{0.667\times10^{-9} \times 68\times10^{-6}}} = \frac{1}{2\pi \times 6.73\times10^{-9}} \approx 23.6\ MHz$$

For 8 × 10 µF MLCC ($L_{MLCC,eff} = 0.9/8 = 0.1125\ nH$) and 16 × 100 nF ($C_{HF} = 1.6\ \mu F$):

$$f_{AR,2} = \frac{1}{2\pi\sqrt{0.1125\times10^{-9} \times 1.6\times10^{-6}}} = \frac{1}{2\pi \times 13.4\times10^{-9}} \approx 11.9\ MHz$$

Neither anti-resonance is at 312 MHz. The actual 312 MHz source requires measurement to confirm.

**Most likely actual cause at 312 MHz:** A resonance between the package inductance of the FPGA ($\sim$1 nH for a large BGA) and the remaining on-die capacitance, or between the high-frequency capacitor ESL and plane capacitance. The VNA measurement (Part d) will resolve this.

---

### Part (c): Why Errors Disappear at 200 MHz

The key is that the 312 MHz spectral component in the supply current must arise from the FPGA's switching activity at a specific harmonic of the clock.

**At 250 MHz clock:** The 5th harmonic is $5 \times 62.5\ MHz = 312.5\ MHz \approx 312\ MHz$. The reference clock for PCIe Gen 4 is 100 MHz; the internal PLL can produce reference clocks at fractional multiples. More directly: $250\ MHz \times 1.25 = 312.5\ MHz$, consistent with a beat frequency between the 250 MHz FPGA clock and a 62.5 MHz SERDES reference clock ($312.5 = 250 + 62.5$).

**At 200 MHz clock:** The 5th harmonic of 40 MHz (potential internal division) is 200 MHz; the beat frequency $200 + 62.5 = 262.5\ MHz$ — no harmonic falls at 312 MHz. None of the harmonics of 200 MHz ($200, 400, 600, \ldots$ MHz) or the beat frequencies with 100 MHz SERDES ($300, 500, \ldots$ MHz) coincide with the 312 MHz resonance.

**Physical mechanism:** The FPGA contains thousands of simultaneously switching flip-flops. The current demand from clock-synchronous switching events has spectral peaks at clock harmonics. When one of these harmonics excites a PDN resonance, it drives that resonance continuously (a periodic excitation at the resonance frequency, not a one-time step). The steady-state voltage oscillation amplitude builds up to:

$$|\Delta V_{ss}| = |Z_{PDN}(f_{resonance})| \times |\hat{I}(f_{resonance})|$$

where $|\hat{I}|$ is the current amplitude at the resonant frequency. If $|Z_{PDN}| \gg Z_{target}$ at 312 MHz, the voltage oscillation exceeds the tolerance budget.

At 200 MHz, the same resonance still exists in the PDN, but no switching harmonic of the FPGA lands at 312 MHz — the resonance is not excited, so no large voltage oscillation builds up.

---

### Part (d): VNA Measurement Procedure

**Goal:** Characterise the 1-port self-impedance $Z_{11}(f)$ of the 0.85 V PDN at the FPGA power pin location, from 1 MHz to 1 GHz.

**Equipment setup:**

1. Power down the board (VRM off; measure passive PDN impedance).
2. Connect the VNA port 1 to the FPGA BGA 0.85 V power pin test point using a coaxial probe or an SMA connector soldered to a test pad near the BGA.
3. Set VNA to 1-port S11 measurement mode.
4. Calibrate to the probe tip (OSL calibration — Open, Short, Load — at the measurement point).
5. Convert S11 to $Z_{11}$ (most VNAs do this directly with a "Z" display mode).
6. Sweep from 1 MHz to 1 GHz; use at least 1001 frequency points.

**Expected impedance profile:**

```
|Z| (dBΩ)
  |
  |  Bulk cap SRF        Mid-freq MLCC       HF MLCC      Package
  |  (anti-resonance)     SRF               SRF          resonance
  |        /\          /\             /\              /\
  |       /  \        /  \           /  \            /  \
  |      /    \______/    \_________/    \__________/    \
  |_____/                                                  \
  |                                                         \
  +-----------------------------------------------------------> f
    1MHz    10MHz    100MHz    300MHz    500MHz    1GHz
```

Key features to look for:
- Anti-resonance peaks at ~23 MHz and ~12 MHz (from Part b analysis)
- A peak at 312 MHz (confirming the hypothesis)
- Plane resonance peaks above 900 MHz

**What confirms the hypothesis:**

A peak in $|Z_{11}|$ at 312 MHz that exceeds $Z_{target}$. The peak height quantifies the severity.

**What refutes the hypothesis:**

No peak at 312 MHz in the impedance plot. In that case, the 312 MHz spectral component on the supply rail may be radiated coupling from the SERDES transceiver noise, not a PDN resonance.

---

### Part (e): Target Impedance Calculation

**Given:**
- $V_{nom} = 0.85\ V$, tolerance $\pm 3\%$
- $\Delta I = 8\ A$

**Voltage budget:**

$\Delta V_{total} = 0.85 \times 3\% = 25.5\ mV$

Allocate: DC accuracy (6 mV), IR drop (4 mV), thermal (2 mV), AC PDN ripple (13.5 mV).

$$\boxed{Z_{target} = \frac{\Delta V_{AC}}{I_{step}} = \frac{13.5\ mV}{8\ A} \approx 1.69\ m\Omega}$$

Round to **$Z_{target} = 1.7\ m\Omega$** for design purposes.

The observed peak at 312 MHz was measured at 45 m$\Omega$ (Part g states). This is $45/1.7 = 26.5$ times above the target — an extreme violation. Even the 60–80 mV supply spikes observed on the scope are consistent: for a 312 MHz current component of even 1 mA amplitude, $\Delta V = 45\ m\Omega \times 1\ mA = 45\ \mu V$ — too small. For the observed 60–80 mV, the current component must be:

$$\Delta I_{312MHz} = \frac{60\ mV}{45\ m\Omega} = 1.33\ A$$

This is plausible for an FPGA with 8 A total current if 1.33/8 = 16% of the switching activity is correlated at 312 MHz.

---

### Part (f): Three Independent Design Fixes

**Fix 1: Add additional high-frequency MLCCs targeted at 300–400 MHz**

Use 0201 100 nF X7R capacitors with lower ESL ($L_{ESL} \approx 0.35\ nH$, $f_{SRF} \approx 27\ MHz$), or 10 nF 0201 C0G capacitors ($f_{SRF} \approx 95\ MHz$). Place them via-in-pad directly at the BGA power pins to minimise path inductance.

**Effect on resonance:** Adding capacitance at 312 MHz loads the resonating structure (whether plane or package resonance) and reduces the Q-factor. The impedance at resonance is inversely proportional to the total damping in the circuit:

$$|Z_{peak}| \approx \frac{Z_0}{Q} \times \frac{1}{1 + N_{added} \cdot C_{added} \cdot Z_0 \cdot \omega_{res}}$$

More accurately: each capacitor below its SRF at 312 MHz (i.e., whose SRF is above 312 MHz) presents a capacitive impedance that short-circuits the resonance. Adding 20 × 10 nF 0201 MLCCs ($f_{SRF} \approx 90\ MHz < 312\ MHz$ — they are inductive at 312 MHz, so this won't help directly). Better: add 20 × 1 nF 0201 capacitors with $f_{SRF} = 1/(2\pi\sqrt{0.3\times10^{-9}\times1\times10^{-9}}) \approx 290\ MHz$ — these are capacitive at 312 MHz and will bypass the resonance directly.

**Predicted effect:** Reduce peak impedance from 45 m$\Omega$ to approximately 5–10 m$\Omega$ (estimate: 5x to 10x reduction from loading the resonance with 20 shunt capacitors).

**Fix 2: Add a lossy damping element near the resonance**

Place a 10 m$\Omega$ to 20 m$\Omega$ thin-film resistor in series with a 1 nF capacitor (forming an RC snubber) from the 0.85 V plane to the ground plane, directly under the FPGA BGA (back-side placement). The RC snubber absorbs energy at the resonant frequency.

RC snubber design: choose the series RC to have an impedance of approximately $|Z_{resonance}|/2$ at 312 MHz:

$$Z_{snubber}(312\ MHz) \approx \frac{1}{2\pi \times 312\ MHz \times 1\ nF} = 0.51\ \Omega$$

This is much higher than the required 45 m$\Omega$ target — so for a 45 m$\Omega$ resonance peak, the snubber capacitor needs to be much larger:

$$C_{snubber} = \frac{1}{2\pi \times 312\ MHz \times 45\ m\Omega} = \frac{1}{2\pi \times 312\times10^6 \times 0.045} = \frac{1}{88.2\times10^6} \approx 11.3\ nF$$

Use $C_{snubber} = 10\ nF$ with $R_{snubber} = 45\ m\Omega$ (a thin-film resistor chip). The snubber damps the resonance by providing a dissipative parallel path.

**Predicted effect:** Reduce peak from 45 m$\Omega$ to less than 5 m$\Omega$ directly at 312 MHz. Narrow-band highly effective at the target frequency.

**Fix 3: Modify the PCB stackup to reduce plane spreading inductance**

Move the 0.85 V power plane pair closer together in the stackup (reduce dielectric thickness from 4 mil to 2 mil). This:
- Reduces spreading inductance (halving $h$ halves $L_{spreading}$)
- Increases plane capacitance (doubling for the same area)
- Reduces the characteristic impedance of the plane cavity, lowering the peak height

The plane capacitance per unit area doubles:

$$C_{plane,2mil} = \varepsilon_0 \varepsilon_r \frac{A}{h/2} = 2 \times C_{plane,4mil}$$

**Predicted effect:** Shifting the plane inductance characteristic downward shifts all resonant features higher in frequency and reduces their impedance by up to 6 dB (2x). It also provides more distributed bypass capacitance, reducing the Q of existing resonances. This is a long-term, fundamental fix but requires a PCB respin.

---

### Part (g): Post-Fix Verification

**Given:** After fix implementation, the 312 MHz resonance peak has been reduced from 45 m$\Omega$ to 8 m$\Omega$.

**Target impedance from Part (e):** $Z_{target} = 1.7\ m\Omega$.

**Assessment:** $8\ m\Omega > 1.7\ m\Omega$. The fix has reduced the peak by a factor of $45/8 = 5.6\times$ (15 dB), but the remaining peak still **exceeds the target impedance by a factor of $8/1.7 = 4.7\times$**.

**Predicted remaining voltage spike:**

The current component at 312 MHz that caused the original 60–80 mV spike with 45 m$\Omega$ impedance:

$$\Delta I_{312MHz} = \frac{60\ mV}{45\ m\Omega} = 1.33\ A$$

After fix, with 8 m$\Omega$ at 312 MHz:

$$\Delta V_{new} = 8\ m\Omega \times 1.33\ A = 10.6\ mV$$

This is less than the $\pm 3\%$ tolerance ($\pm 25.5\ mV$) if the full tolerance is available for AC noise. From Part (e), the allocated AC budget is 13.5 mV.

$$10.6\ mV < 13.5\ mV \quad \checkmark$$

**Conclusion:** The fix **may resolve the operational failure** even though the impedance technically still exceeds $Z_{target}$, because:
1. The voltage spike (10.6 mV) is now within the allocated AC budget
2. The $Z_{target}$ calculation used conservative assumptions (full $\Delta I = 8\ A$ as the step, but the actual 312 MHz spectral component is only 1.33 A)

However, this conclusion relies on the 1.33 A estimate being accurate. To fully certify the design:
1. Measure the voltage spike amplitude directly: expect approximately 10–12 mV
2. Run the full traffic pattern that previously generated ECC errors and verify the error count
3. Continue reducing the PDN impedance to below $Z_{target}$ if errors persist

**Recommended further action:**

Implement Fix 2 (RC snubber tuned to 312 MHz) as an additional rework to reduce the peak from 8 m$\Omega$ to $< 2\ m\Omega$. This can be done on the existing board without a respin, by adding a back-side snubber component.

---

## Lessons Learned (Examiner Notes)

This problem illustrates several real-world PDN debugging scenarios:

1. **Spectral analysis of supply noise is more informative than amplitude alone.** Knowing the frequency of the noise peak guides the root-cause diagnosis and the fix.

2. **Plane resonances are NOT always the culprit** at high frequencies. Anti-resonances between capacitor stages, package resonances, and SERDES coupling are equally common. Always calculate the plane resonance frequencies first to confirm or rule out the hypothesis.

3. **The target impedance is a design requirement, not just a guideline.** A factor of 4.7× above target is a serious failure. The fact that the design nominally "survived" with the partial fix is due to conservative budget allocation, not good PDN engineering.

4. **Clock frequency dependence is a diagnostic fingerprint.** If the failure disappears at a different clock frequency, a harmonic of the clock is exciting a PDN resonance. The crossing frequency identifies which harmonic — narrowing the debug to a specific frequency range.

5. **VNA measurement is the definitive PDN debug tool.** Oscilloscope measurements give time-domain amplitude but limited spectral resolution. VNA impedance plots from 1 MHz to 1 GHz immediately reveal all resonance peaks and anti-resonances, allowing direct comparison against the target impedance curve.
