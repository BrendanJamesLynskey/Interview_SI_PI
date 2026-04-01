# PDN Impedance

## Overview

The Power Distribution Network (PDN) is the complete electrical path from the voltage regulator module (VRM) output through PCB planes, vias, package leads, and on-die capacitance to the power pins of every integrated circuit on the board. Its impedance — how that network resists and delays current flow as a function of frequency — determines whether the supply voltage stays within specification when a device demands a sudden change in current.

This document covers the physics of PDN impedance from DC up to several gigahertz: the individual contributors to that impedance, how they interact to create the characteristic impedance profile, and what a well-designed PDN impedance curve looks like in practice.

---

## Tier 1: Fundamentals

### Q1. What is PDN impedance, and why does frequency matter?

**Answer:**

PDN impedance $Z_{PDN}(f)$ is the complex ratio of a small-signal voltage perturbation on a power rail to the current perturbation that caused it, measured as a function of frequency:

$$Z_{PDN}(f) = \frac{\Delta V(f)}{\Delta I(f)}$$

A purely resistive PDN would give a flat $Z(f)$. In practice, the PDN contains resistance, inductance, and capacitance distributed across every element from the VRM to the die, so $Z(f)$ varies dramatically with frequency.

Frequency matters because the switching activity of a digital device generates current demand at many frequencies simultaneously. A processor executing code draws a quasi-DC bias current, generates switching transients with frequency content up to $f_{knee} \approx 0.35 / t_r$ (where $t_r$ is signal rise time), and may exhibit resonant load behaviour at clock harmonics. The PDN must present low impedance across all of these frequencies; if any frequency region has high $|Z_{PDN}|$, the corresponding current demand produces a large voltage deviation $\Delta V = Z \cdot \Delta I$.

**Common mistake:** Characterising a PDN only at DC or at a single frequency (e.g., the switching frequency of a regulator). Failures routinely occur at frequencies where no one checked the impedance.

---

### Q2. What are the three main regions of a PDN impedance profile, and what dominates each?

**Answer:**

A typical PDN impedance magnitude $|Z(f)|$ plotted on a log-log scale has three distinct regions:

**1. Low frequency (roughly DC to a few kilohertz) — VRM dominated**

The VRM feedback loop actively regulates the output voltage. Within its loop bandwidth $f_{BW}$, the closed-loop output impedance is low and falls with decreasing frequency. The dominant element is the VRM's closed-loop output resistance $R_{out,CL}$. Below $f_{BW}$, $|Z| \approx R_{out,CL}$, which may be milliohms for a well-designed supply.

**2. Mid frequency (kilohertz to low megahertz) — bulk and mid-frequency capacitors**

Above the VRM loop bandwidth, the regulator can no longer respond. Bulk electrolytic or polymer capacitors take over. Their impedance falls as $1/(2\pi f C_{bulk})$ at low frequencies, flattens at the capacitor's ESR in the middle, then rises as $2\pi f L_{ESL}$ at higher frequencies. Multiple capacitor stages (bulk, ceramic MLCC) each handle a frequency decade.

**3. High frequency (tens of megahertz to GHz) — parasitic inductance and package/die capacitance**

At high frequencies, the PCB trace inductance, via inductance, package lead inductance, and eventually on-die capacitance dominate. The PDN inductance forms resonant tanks with capacitors, creating peaks and troughs in $|Z(f)|$. Beyond a few hundred megahertz the on-die bypass capacitance, whose ESL is picohenries, provides the last line of defence.

**Schematic summary:**

```
DC           1 kHz        1 MHz        100 MHz      1 GHz
|            |            |            |            |
[VRM loop]   [bulk cap]   [MLCC]       [pkg ind]    [on-die cap]
impedance    impedance    impedance    resonance    impedance
falling      falling      flat/ESR     peaks        rising
```

---

### Q3. Define ESR, ESL, and effective capacitance for a real capacitor. How do they set the impedance of a bypass capacitor?

**Answer:**

A real capacitor deviates from an ideal capacitor. Its complete lumped model is:

$$Z_{cap}(f) = R_{ESR} + j\left(2\pi f L_{ESL} - \frac{1}{2\pi f C}\right)$$

where:
- $C$ — nominal capacitance (farads)
- $R_{ESR}$ — equivalent series resistance: conduction losses in leads, termination, and dielectric
- $L_{ESL}$ — equivalent series inductance: self-inductance of leads, termination pads, and internal structure

**Three frequency zones:**

Below self-resonant frequency (SRF): capacitive — $|Z| \approx 1/(2\pi f C)$, falling at 20 dB/decade.

At SRF: resistive — $|Z| = R_{ESR}$, a minimum. This is the most useful region for bypass filtering.

$$f_{SRF} = \frac{1}{2\pi\sqrt{L_{ESL} \cdot C}}$$

Above SRF: inductive — $|Z| \approx 2\pi f L_{ESL}$, rising at 20 dB/decade.

**Practical numbers for 0402 100 nF MLCC:** $R_{ESR} \approx 20\text{–}50\ m\Omega$, $L_{ESL} \approx 0.5\text{–}1\ nH$, giving $f_{SRF} \approx 20\text{–}50\ MHz$.

**Consequence for PDN design:** a capacitor can only provide effective bypass over roughly one decade centred on its SRF. Above SRF, adding more capacitance of the same type does not help — it adds inductance in parallel and can worsen resonance peaks.

---

### Q4. What is the self-resonant frequency (SRF) of a capacitor, and what happens to impedance above the SRF?

**Answer:**

The SRF is the frequency at which the capacitive reactance $X_C = 1/(2\pi f C)$ equals the inductive reactance $X_L = 2\pi f L_{ESL}$, causing them to cancel:

$$f_{SRF} = \frac{1}{2\pi\sqrt{L_{ESL} \cdot C}}$$

At SRF, $|Z| = R_{ESR}$, its minimum. Below SRF the device behaves as a capacitor; above SRF it behaves as an inductor.

**Above SRF:** The device presents net inductive impedance $Z \approx j \cdot 2\pi f L_{ESL}$. Placing more capacitors of the same type in parallel does not reduce this inductance effectively for frequencies well above SRF — the dominant term is $L_{ESL}/N$ for $N$ capacitors in parallel, but the SRF of the parallel combination barely changes because both $C_{total} = NC$ and $L_{ESL,total} = L_{ESL}/N$, so $f_{SRF}$ stays approximately the same. The minimum impedance (ESR/N) does decrease.

**Practical implication:** To cover higher frequencies, use smaller capacitors with lower $L_{ESL}$ (achieved by using smaller case sizes, or capacitors with lower-inductance terminations such as reverse-geometry or via-in-pad configurations). The rule of thumb is: reduce capacitor case size to reduce ESL, not just to reduce capacitance value.

---

## Tier 2: Intermediate

### Q5. Sketch and explain the complete broadband PDN impedance profile, including all resonance peaks. What causes anti-resonance?

**Answer:**

The full PDN impedance profile on a log-log plot (impedance magnitude vs. frequency) has a characteristic shape with multiple features:

```
|Z|
 |
 |  VRM closed-loop         Anti-resonance peaks
 |  output impedance     /\        /\
 |     \               /  \      /  \
 |      \             /    \    /    \
 |       \           / bulk \  /MLCC  \
 |        \_________/  ESR   \/  ESR   \___
 |                                         \  package
 |                                          \ inductance
 |                                           \
 |                                            ---
 +------------------------------------------------> f
   DC   1kHz   10kHz  100kHz  1MHz   10MHz  100MHz 1GHz
```

**Anti-resonance (resonance peaks upward):** occurs when two adjacent capacitor stages — e.g., a bulk electrolytic and a ceramic MLCC array — are connected in parallel. At some intermediate frequency, the bulk capacitor is above its SRF and behaves inductively, while the MLCC is below its SRF and behaves capacitively. Their parallel combination forms an LC tank circuit:

$$f_{anti-resonance} \approx \frac{1}{2\pi\sqrt{L_{bulk,ESL} \cdot C_{MLCC}}}$$

At anti-resonance, $|Z|$ rises sharply — potentially to hundreds of milliohms — far exceeding the individual impedances of either capacitor stage at that frequency. This is the most common cause of mid-frequency PDN failures.

**Physics:** Two resonators in parallel: one inductive (bulk above SRF), one capacitive (MLCC below SRF). The parallel combination of an inductor and capacitor has maximum (not minimum) impedance at resonance — this is the dual of series resonance.

**Key anti-resonance pairs to watch:**
- VRM output filter inductance vs. bulk capacitors
- Bulk capacitor ESL vs. MLCC capacitance
- MLCC ESL vs. package capacitance
- Package inductance vs. on-die capacitance

**Mitigation:** Ensure the impedance of stage N overlaps sufficiently with stage N+1 so there is no frequency gap where both stages are inductive or both are capacitive simultaneously. Adding damping resistance (e.g., a series resistor with the bulk cap) broadens the impedance peak and lowers its maximum.

---

### Q6. How does the number of PDN capacitors in parallel affect impedance? Derive the impedance for $N$ identical capacitors in parallel.

**Answer:**

For $N$ identical capacitors each with parameters $(C, R_{ESR}, L_{ESL})$ connected in parallel:

$$Z_{parallel}(f) = \frac{Z_{single}(f)}{N} = \frac{1}{N}\left(R_{ESR} + j\left(2\pi f L_{ESL} - \frac{1}{2\pi f C}\right)\right)$$

This gives:
- Total capacitance: $C_{total} = NC$ — lower impedance at low frequencies
- Total ESR: $R_{ESR,total} = R_{ESR}/N$ — lower minimum impedance at SRF
- Total ESL: $L_{ESL,total} = L_{ESL}/N$ — lower impedance at high frequencies

The SRF of the parallel combination:

$$f_{SRF,total} = \frac{1}{2\pi\sqrt{(L_{ESL}/N)(NC)}} = \frac{1}{2\pi\sqrt{L_{ESL} \cdot C}} = f_{SRF,single}$$

The SRF does not change. This is an important result: adding more of the same capacitor type scales all reactive and resistive terms equally, maintaining the same resonant frequency but lowering the impedance uniformly by $1/N$.

**Consequence:** To shift coverage to a higher frequency, you must reduce $L_{ESL}$ (smaller case size, different package). To reduce the minimum impedance at a given frequency, add more capacitors of the same type in parallel. These are independent design levers.

**Worked example:** Four 100 nF 0402 MLCCs in parallel give $C_{total} = 400\ nF$, $R_{ESR,total} = R_{ESR}/4$, $L_{ESL,total} = L_{ESL}/4$. The SRF remains the same as a single capacitor. Below SRF the impedance is $1/(2\pi f \cdot 400\ nF)$ — 4x lower than a single capacitor. Above SRF the inductive slope starts at $L_{ESL}/4$ — also 4x lower. The parallel combination is uniformly better by 12 dB.

---

### Q7. Explain the concept of "spreading inductance" and its effect on PDN impedance at high frequencies.

**Answer:**

Spreading inductance (also called plane spreading inductance) is the partial self-inductance of the current path as it spreads laterally through the PCB power and ground planes from a via to the bulk of the plane.

When a via connects a decoupling capacitor to the power/ground plane pair, the current does not instantly distribute across the entire plane. It must travel radially outward from the via. This spreading path has a finite inductance per unit length. The effective spreading inductance seen at the via depends on:

- The separation (dielectric thickness) between the power and ground planes: $L_{spreading} \propto h$
- The geometry: thinner dielectric = lower spreading inductance
- Frequency: at low frequencies current spreads across the entire plane; at high frequencies current is confined to a small region around the via

**Formula (approximate, for a via on an infinite plane pair):**

$$L_{spreading} \approx \frac{\mu_0 h}{2\pi} \ln\left(\frac{r_{outer}}{r_{via}}\right)$$

where $h$ is the plane separation, $r_{outer}$ is the effective current spreading radius (frequency dependent), and $r_{via}$ is the via radius.

**Practical consequence:** Even if a capacitor's own $L_{ESL}$ is very low (e.g., 200 pH for a 0201 MLCC), the spreading inductance from the via through which it connects to the plane can add 0.5–2 nH, completely dominating the capacitor's own ESL. This is why **via placement and count** for decoupling capacitors matters more than the capacitor's own rated ESL at high frequencies.

**Mitigation:**
- Place capacitors close to the device power pins (minimise spreading distance)
- Use two vias per capacitor (one to power, one to ground) — halves via inductance
- Use via-in-pad (VIP) to minimise trace inductance from pad to via
- Maximise the number of parallel vias by tiling capacitors around the device

---

### Q8. What is the relationship between PDN impedance and the location of the measurement point on the PCB?

**Answer:**

PDN impedance is not a single number — it is a 2-port (or 1-port) quantity that depends on where the current injection and voltage measurement occur.

**Self-impedance $Z_{11}$:** Measured at one port (e.g., a test point near the device power pin). This is the impedance "seen by" the device.

**Transfer impedance $Z_{21}$:** Ratio of voltage at port 2 to current injected at port 1. This characterises how noise current from one aggressor device affects the supply voltage seen by a victim device.

**Location dependence:**

The impedance seen at the die attach depends on the inductance of the path between the measurement point and the die:

$$Z_{at\_die}(f) \approx Z_{measured}(f) + j \cdot 2\pi f \cdot L_{path}$$

At low frequencies, the inductive term $j\omega L_{path}$ is small and location matters little. At high frequencies (tens of MHz and above), even a few nanohenries of path inductance adds significant impedance. A PDN that looks excellent at a test point 2 cm from the BGA may have dramatically higher impedance at the actual die.

**Implication for measurement:** Always characterise PDN impedance as close to the device power pins as possible. For FPGAs and ASICs, the relevant measurement point is at the package pins or, if possible, via a probe connection near the BGA balls. Simulations should model the full path from VRM to die, including package inductance.

---

## Tier 3: Advanced

### Q9. Derive the impedance of a simple two-stage PDN (VRM + single bulk capacitor + single MLCC) and identify all resonance features analytically.

**Answer:**

Model the PDN as three impedances in the current path from VRM to die:

1. VRM output impedance above its loop bandwidth: a series $R_V + j\omega L_V$ (the VRM filter inductor)
2. Bulk capacitor: $Z_{bulk}(\omega) = R_B + j(\omega L_B - 1/\omega C_B)$
3. MLCC: $Z_{MLCC}(\omega) = R_M + j(\omega L_M - 1/\omega C_M)$

The bulk capacitor and MLCC are connected in parallel between the power and ground rails. Their parallel combination $Z_{par}$ is in series with the VRM inductor $L_V$:

$$Z_{par}(\omega) = \frac{Z_{bulk} \cdot Z_{MLCC}}{Z_{bulk} + Z_{MLCC}}$$

For simplified analysis, assume ideal capacitors with only ESL (ignore ESR and treat the capacitors as $L_B - C_B$ and $L_M - C_M$ series tanks in parallel):

Numerator of $Z_{par}$:

$$Z_{bulk} \cdot Z_{MLCC} = \left(j\omega L_B + \frac{1}{j\omega C_B}\right)\left(j\omega L_M + \frac{1}{j\omega C_M}\right)$$

This product has zeros (numerator = 0) at the series resonances of each individual capacitor:

$$\omega_{SRF,B} = \frac{1}{\sqrt{L_B C_B}}, \quad \omega_{SRF,M} = \frac{1}{\sqrt{L_M C_M}}$$

Denominator:

$$Z_{bulk} + Z_{MLCC} = j\omega(L_B + L_M) + \frac{1}{j\omega C_B} + \frac{1}{j\omega C_M}$$

Setting the denominator to zero (parallel resonance = anti-resonance of the parallel combination) gives:

$$j\omega(L_B + L_M) = \frac{1}{j\omega}\left(\frac{1}{C_B} + \frac{1}{C_M}\right) \cdot (-1)$$

$$\omega_{AR}^2 = \frac{C_B + C_M}{L_{total} \cdot C_B \cdot C_M} \approx \frac{1}{L_B \cdot C_M}$$

(using $L_B \gg L_M$ and $C_B \gg C_M$ as typical values: bulk cap has large $C_B$ and large $L_B$; MLCC has small $C_M$ and small $L_M$).

**Result:** The anti-resonance frequency is controlled primarily by the bulk capacitor's ESL and the MLCC's capacitance. The peak impedance at anti-resonance approaches:

$$|Z_{AR}| \approx \sqrt{\frac{L_B}{C_M}}$$

For $L_B = 5\ nH$, $C_M = 10\ \mu F$: $|Z_{AR}| = \sqrt{5\times10^{-9} / 10\times10^{-6}} = \sqrt{0.5\times10^{-3}} \approx 22\ m\Omega$.

Whether this peak is problematic depends on the target impedance for the rail in question.

---

### Q10. How does PCB plane inductance affect PDN impedance at high frequencies, and how do you calculate the partial inductance of a power plane pair?

**Answer:**

At high frequencies (above ~100 MHz), the dominant inductive element in the PDN is no longer discrete component ESL but rather the partial inductance of the PCB power/ground plane pair itself, specifically the inductance of the current loop formed between the device via and the nearest capacitor via, travelling through the plane layers.

**Plane pair inductance model:**

For a rectangular current loop of width $w$, length $\ell$, and plane separation $h$ (all in metres), the partial loop inductance is approximately:

$$L_{plane} \approx \mu_0 \cdot h \cdot \frac{\ell}{w}$$

For a current spreading outward radially (as from a capacitor via), the effective inductance to the centre of the plane at high frequency is:

$$L_{spreading}(f) \approx \frac{\mu_0 h}{2\pi} \ln\left(\frac{d}{r_0}\right)$$

where $d$ is the distance from via to the current return reference, and $r_0$ is the via radius.

**Numerical example:** For $h = 100\ \mu m$ (4 mil dielectric), $d = 5\ mm$, $r_0 = 0.15\ mm$:

$$L_{spreading} = \frac{4\pi\times10^{-7} \times 100\times10^{-6}}{2\pi} \ln\left(\frac{5}{0.15}\right) = \frac{4\times10^{-11}}{1} \times \ln(33.3) \approx 2\times10^{-11} \times 3.5 \approx 1.4\ nH$$

This inductance is comparable to the ESL of several ceramic capacitors — highlighting that plane geometry, not just capacitor count, limits PDN performance above 100 MHz.

**Practical conclusions:**
- Thin dielectrics (2–4 mil) between power and ground planes dramatically reduce $L_{plane}$
- Close coupling of power and return planes is the most effective high-frequency PDN improvement
- A 1 oz copper plane pair at 2 mil separation can provide 100–300 pF/in² of effective capacitance along with very low inductance — this is "free" decoupling from the plane structure itself

---

### Q11. What is the significance of the PDN impedance "floor" set by plane spreading inductance, and how does it limit the achievable flat impedance target?

**Answer:**

Even with unlimited ideal decoupling capacitors placed everywhere, the PDN impedance cannot be reduced below a fundamental floor set by the partial inductance of the current path through the PCB stackup.

**The inductance floor:**

Above a few hundred MHz, the dominant impedance contributor is the inductance of the via-to-via current loop: capacitor via, through the power plane, through the device via, back through the ground plane. This loop inductance is approximately:

$$L_{loop} = \frac{\mu_0 h}{\pi} \ln\left(\frac{d}{\rho}\right)$$

where $d$ is the centre-to-centre distance between capacitor and device vias.

The resulting impedance floor at frequency $f$ is:

$$Z_{floor}(f) = 2\pi f \cdot L_{loop}$$

This is inductive — it rises at 20 dB/decade and cannot be reduced by adding more capacitors of the same placement (adding capacitance lowers the SRF of the combined network but if the current must still travel through $L_{loop}$ to reach the capacitor, the impedance seen at the device does not improve above the SRF of that loop).

**Consequence for target impedance:** If the target impedance is $Z_{target}$ and the plane inductance floor reaches $Z_{target}$ at frequency $f_{cross}$, then for $f > f_{cross}$ the PDN cannot achieve the target with PCB-mounted capacitors alone. The only solution is:

1. Reduce the plane inductance (thinner dielectric, closer power/ground planes, larger plane area)
2. Add on-package or on-die capacitance (which bypasses the package inductance)
3. Reduce the target current step (gate the device, reduce core voltage, accept larger $\Delta V$)

This is why modern high-power ASICs and FPGAs include embedded decoupling capacitors in the package substrate, and why on-die decoupling is a standard feature of advanced-node ICs.

---

### Q12. How does the PDN impedance relate to voltage ripple and power supply noise? Derive the relationship and state the assumptions.

**Answer:**

The time-domain voltage deviation $\Delta v(t)$ on a power rail is related to the current demand $\Delta i(t)$ through the convolution:

$$\Delta v(t) = \Delta i(t) * z(t)$$

where $z(t)$ is the inverse Fourier transform of $Z_{PDN}(f)$.

In the frequency domain:

$$\Delta V(f) = Z_{PDN}(f) \cdot \Delta I(f)$$

**Assumptions:** This is a linear, time-invariant (LTI) model. The PDN is treated as a passive linear network; the device (load) current is treated as an independent current source. These assumptions hold well for small-signal analysis but break down if:
- The device load impedance is comparable to $Z_{PDN}$ (loading effect)
- Large-signal current swings saturate capacitors (voltage coefficient of MLCCs — significant for class II dielectrics)
- The VRM non-linearly clamps or limits current

**Voltage ripple from a known current spectrum:**

If the device current spectrum has a prominent component at frequency $f_0$ with amplitude $|\Delta I(f_0)|$:

$$|\Delta V(f_0)| = |Z_{PDN}(f_0)| \cdot |\Delta I(f_0)|$$

For a step change of $\Delta I_{step}$ amps, the worst-case $\Delta V$ is bounded by the target impedance method (see `target_impedance_method.md`). The peak voltage deviation depends not just on the magnitude of $Z_{PDN}$ but on its phase — a purely inductive PDN will produce a voltage spike on the rising edge of the current step, not a sustained deviation.

**Ripple at switching frequency:** A converter switching at $f_{sw}$ produces current ripple $\Delta i_{L}$ in the inductor. The output voltage ripple is:

$$\Delta V_{ripple} \approx \Delta i_L \cdot R_{ESR,bulk} + \frac{\Delta i_L}{8 f_{sw} C_{bulk}}$$

The first term (ESR-limited) dominates for electrolytic capacitors; the second term (capacitance-limited) dominates for MLCCs with very low ESR.

---

## Summary Table: PDN Impedance Contributors

| PDN Element | Dominant Region | Typical Value | Scaling |
|---|---|---|---|
| VRM closed-loop $R_{out}$ | DC to $f_{BW}$ | 1–10 m$\Omega$ | Improves with loop gain |
| Bulk cap capacitance | 1 kHz–100 kHz | 100–2200 $\mu$F | Lower $Z$ with more $C$ |
| Bulk cap ESR | At SRF | 5–50 m$\Omega$ | Paralleling reduces |
| Bulk cap ESL | Above SRF | 3–20 nH | Determines anti-resonance |
| MLCC capacitance | 100 kHz–10 MHz | 1–100 $\mu$F | More caps or larger values |
| MLCC ESR | At SRF | 5–50 m$\Omega$ | Low for X5R/X7R |
| MLCC ESL | Above SRF | 0.3–1 nH (0402) | Reduced by smaller case |
| PCB via inductance | >10 MHz | 0.3–2 nH | Two vias per cap helps |
| Plane spreading inductance | >100 MHz | 0.5–5 nH | Thin dielectric critical |
| Package inductance | >100 MHz | 0.1–2 nH | Package-specific |
| On-die capacitance | >500 MHz | 1–100 nF | Process dependent |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Capacitor SRF | $f_{SRF} = \dfrac{1}{2\pi\sqrt{L_{ESL} \cdot C}}$ |
| Impedance at SRF | $|Z_{SRF}| = R_{ESR}$ |
| $N$ caps in parallel (ESL) | $L_{total} = L_{ESL}/N$ |
| Anti-resonance frequency | $f_{AR} \approx \dfrac{1}{2\pi\sqrt{L_{B,ESL} \cdot C_{M}}}$ |
| Anti-resonance peak impedance | $|Z_{AR}| \approx \sqrt{L_{B,ESL}/C_{M}}$ |
| Plane spreading inductance | $L \approx \dfrac{\mu_0 h}{2\pi}\ln(d/r_0)$ |
| Plane pair capacitance | $C_{plane} = \varepsilon_0 \varepsilon_r A / h$ |
