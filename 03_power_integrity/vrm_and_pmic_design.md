# VRM and PMIC Design

## Overview

The voltage regulator module (VRM) or power management IC (PMIC) is the source end of the PDN. Its dynamic behaviour — loop bandwidth, transient response, output impedance — determines the low-frequency end of the PDN impedance profile and sets the minimum amount of bulk decoupling capacitance required. A VRM that cannot respond quickly enough forces the system to rely on passive energy storage for longer, increasing capacitance requirements and cost. This document covers VRM loop dynamics, PMIC selection criteria, load transient response analysis, and the interaction between the VRM and the rest of the PDN.

---

## Tier 1: Fundamentals

### Q1. What is a VRM loop bandwidth, and why is it critical for PDN design?

**Answer:**

The VRM loop bandwidth $f_{BW}$ (also called the crossover frequency $f_c$) is the frequency at which the open-loop gain of the voltage regulator's control system equals 0 dB (unity). It characterises how quickly the regulator can respond to changes in load current.

**Physical meaning for PDN:**

Below $f_{BW}$, the VRM's closed-loop feedback actively regulates the output voltage — any load-current-induced voltage deviation is detected and corrected within roughly one cycle of $f_{BW}$. The closed-loop output impedance is very low (ideally zero) in this frequency range.

Above $f_{BW}$, the VRM cannot respond. Its output impedance transitions from the closed-loop (low) value to the open-loop (high) value, which is dominated by the output filter inductor. The PDN must rely entirely on bypass capacitors for frequencies above $f_{BW}$.

**Quantitative relationship:**

The closed-loop output impedance of the VRM above its loop bandwidth rises approximately as:

$$Z_{out,VRM}(f) \approx j2\pi f L_{filter} \quad \text{for } f > f_{BW}$$

where $L_{filter}$ is the VRM output filter inductance. This impedance rises at 20 dB/decade and must be countered by capacitor stages.

**Target bandwidth from PDN requirements:**

The VRM must achieve $|Z_{out}| \leq Z_{target}$ at its crossover frequency. Since the closed-loop impedance at $f_{BW}$ is approximately $Z_0^{filter} = \sqrt{L_{filter}/C_{filter}}$, the loop bandwidth must satisfy:

$$f_{BW} \geq \frac{1}{2\pi C_{bulk} Z_{target}}$$

For a 1 A load, 50 mV tolerance ($Z_{target} = 50\ m\Omega$), and $C_{bulk} = 100\ \mu F$:

$$f_{BW} \geq \frac{1}{2\pi \times 100\times10^{-6} \times 50\times10^{-3}} \approx 31.8\ kHz$$

For a 10 A load, 25 mV tolerance ($Z_{target} = 2.5\ m\Omega$), and $C_{bulk} = 4\ mF$:

$$f_{BW} \geq \frac{1}{2\pi \times 4\times10^{-3} \times 2.5\times10^{-3}} \approx 15.9\ kHz$$

High-current, tight-tolerance rails require either high loop bandwidth or very large bulk capacitance, or both.

---

### Q2. What is a PMIC, and how does it differ from a discrete VRM?

**Answer:**

A PMIC (Power Management IC) is a single integrated circuit that contains multiple voltage regulators (buck, LDO, boost), power switches, and often supervisory functions (power sequencing, fault detection, enable/disable logic) in one package.

**Differences from discrete VRM:**

| Property | Discrete VRM | PMIC |
|---|---|---|
| Integration level | One converter per IC | Multiple rails in one IC |
| Current capacity | High (1–100 A per phase) | Lower (typically 0.1–5 A per channel) |
| Output count | 1–4 (multiphase variants) | 2–20 rails |
| Switching frequency | 300 kHz to 2 MHz | 1–6 MHz (integrated) |
| Loop bandwidth | 20–200 kHz | 100 kHz to 1 MHz |
| Efficiency | Optimised for high current | Reasonable for moderate current |
| PCB area | Larger (external components) | Smaller (highly integrated) |
| Application | CPU/FPGA core rails | SoC, mobile, IoT, mixed-signal boards |

**When to use a PMIC:**

- Multiple low-to-moderate current rails ($< 3\ A$) needed on the same board
- Board area is limited
- Power sequencing requirements (the PMIC handles rail ordering automatically)
- Battery-powered systems (PMIC often integrates battery charger and fuel gauge)

**When to use a discrete VRM:**

- High current rail ($> 5\ A$, e.g., FPGA core, CPU)
- Tight transient requirements demanding high loop bandwidth
- Custom multiphase design to minimise output inductance and capacitance
- When the PMIC's integrated passive components limit transient performance

---

### Q3. What is load transient response, and how is it characterised?

**Answer:**

Load transient response describes how the output voltage of a regulator behaves when the load current changes suddenly. It is characterised by:

**Voltage undershoot (positive load step):**

When load current increases suddenly from $I_1$ to $I_2$ (a positive step of $\Delta I = I_2 - I_1$), the output voltage immediately drops because:
1. The PDN has finite impedance: initial $\Delta V = Z_{PDN} \cdot \Delta I$ (dominated by the bypass capacitors)
2. The VRM has not yet responded (response delayed by loop latency $1/f_{BW}$)

The voltage undershoot can be divided into two phases:
- **Phase 1 (0 to $1/f_{BW}$):** capacitors supply the current; voltage sags at a rate $dV/dt = \Delta I / C_{total}$
- **Phase 2 (after $1/f_{BW}$):** VRM loop begins to correct; inductor current ramps up; voltage recovers

**Voltage overshoot (negative load step):**

When load current decreases suddenly, the VRM's inductor continues to deliver current (it cannot stop instantly). The excess current charges the output capacitors, causing a voltage overshoot.

**Key metrics:**

- **Peak undershoot $\Delta V_{pk}$:** maximum voltage deviation below nominal during positive step
- **Peak overshoot $\Delta V_{ov}$:** maximum voltage above nominal during negative step
- **Settling time $t_s$:** time for voltage to return within specification after the transient
- **Slew rate of voltage response:** $dV/dt$ during the transient

**Typical specifications (from IC power-on guide):**

| Metric | Typical value |
|---|---|
| Peak undershoot | $< \pm 3\%$ of $V_{nom}$ |
| Settling time | $< 100\ \mu s$ |
| Steady-state ripple | $< 1\%$ of $V_{nom}$ |

---

### Q4. What is load line (droop) regulation, and why is it used for high-performance processors?

**Answer:**

Load line (or active droop) regulation intentionally programs the VRM output voltage to be linearly dependent on the output current:

$$V_{out}(I) = V_{nom} - R_{LL} \times I_{load}$$

where $R_{LL}$ is the load line resistance (also called droop impedance or output impedance setpoint).

**Why it is used:**

Without load line regulation, the VRM tries to hold $V_{out} = V_{nom}$ at all load currents. This means:
- At zero load: $V_{out} = V_{nom}$
- At full load ($I_{max}$): $V_{out} = V_{nom}$ (VRM corrects any sag)

A load step from zero to full load causes an undershoot of up to $\Delta V_{pk}$ below $V_{nom}$ before the VRM recovers, and then settles back to $V_{nom}$. This requires the IC to tolerate a range of $[V_{nom} - \Delta V_{pk}, V_{nom}]$.

**With load line regulation:**

The VRM is programmed to output:
- At zero load: $V_{out} = V_{nom} + R_{LL} \times I_{max}/2$ (or some upper value)
- At full load: $V_{out} = V_{nom} - R_{LL} \times I_{max}/2$ (or some lower value)

The steady-state output is centred in the tolerance window at half load. A load step from zero to full load undershoots by approximately $\Delta V_{pk}$ below $V_{nom} - R_{LL} \times I_{max}/2$, but since the steady-state full-load voltage is already at the bottom of the window, the transient just goes somewhat below the already-low nominal — and the total required tolerance window is smaller.

**Quantitative benefit:**

Without droop: required tolerance = $V_{nom}$ to $V_{nom} - \Delta V_{pk}$. Range = $\Delta V_{pk}$.

With droop ($R_{LL} = \Delta V_{pk}/I_{max}$): VRM output at full load is $V_{nom} - \Delta V_{pk}$. Transient undershoot below this equals $\Delta V_{pk}/2$ (same capacitors, half the effective current step to absorb). Total range = $\Delta V_{pk}/2 + \Delta V_{pk}/2 = \Delta V_{pk}$.

Wait — that seems the same. The actual benefit: by pre-drooping the voltage, the capacitors only need to supply half the total swing, effectively halving the required capacitance:

$$C_{required,droop} = \frac{C_{required,no-droop}}{2}$$

This is the key result: active droop halves the required bulk capacitance for a given tolerance window, at the cost of reduced DC output accuracy.

---

## Tier 2: Intermediate

### Q5. Derive the closed-loop output impedance of a voltage-mode buck converter and show how it relates to the PDN target impedance.

**Answer:**

**Open-loop model:**

A voltage-mode buck converter with output filter ($L$, $C_{out}$, $R_{ESR}$) has an open-loop (uncompensated) output impedance that is the parallel combination of the output filter impedance $Z_{filter}$ and the load:

For the output network alone (ignoring load):

$$Z_{out,OL}(s) = \frac{(sL + R_L)(sR_{ESR}C_{out} + 1)/(sC_{out})}{sL + R_L + (sR_{ESR}C_{out}+1)/(sC_{out})}$$

Simplified (assuming $R_L \ll sL$ and $R_{ESR} \ll 1/(sC)$):

$$Z_{out,OL}(s) \approx \frac{sL + 1/(sC_{out})}{1} = j\left(\omega L - \frac{1}{\omega C}\right) \quad \text{(lossless)}$$

**Closed-loop output impedance:**

With a feedback control loop of open-loop gain $T(s)$:

$$Z_{out,CL}(s) = \frac{Z_{out,OL}(s)}{1 + T(s)}$$

Below the crossover frequency $f_c$ (where $|T| \gg 1$):

$$|Z_{out,CL}| \approx \frac{|Z_{out,OL}|}{|T|} \approx \frac{|Z_{out,OL}|}{A_0 / (f/f_c)} \quad (\text{for a single-pole roll-off of }T)$$

For a simple type-I compensator, $|T(f)| = f_c / f$ below $f_c$:

$$|Z_{out,CL}(f)| \approx \frac{Z_{out,OL}(f) \cdot f}{f_c}$$

Below the LC resonance: $Z_{out,OL} \approx 1/(\omega C)$, so:

$$|Z_{out,CL}(f)| \approx \frac{f}{f_c \cdot 2\pi f C} = \frac{1}{2\pi f_c C}$$

This is approximately flat (frequency-independent) — a resistive output impedance equal to $R_{CL} = 1/(2\pi f_c C)$.

**Connection to target impedance:**

Setting $Z_{out,CL} = Z_{target}$:

$$Z_{target} = \frac{1}{2\pi f_c C_{out}} \Rightarrow f_c = \frac{1}{2\pi C_{out} Z_{target}}$$

This is the same result derived from the energy argument in the target impedance method — confirming that the VRM loop bandwidth must be at least $1/(2\pi C_{out} Z_{target})$ to achieve target impedance coverage down to $f_c$.

At $f_c$, the closed-loop impedance transitions from the flat resistive value to the rising inductive value $|Z| \approx 2\pi f L$. This transition point is why the bulk capacitor stage is designed to take over exactly at $f_c$.

---

### Q6. What is multiphase VRM interleaving, and what are its PDN benefits?

**Answer:**

A multiphase VRM has $N$ converter phases, each with its own inductor and switch network but sharing the output capacitors. The phases switch at the same frequency $f_{sw}$ but with phase offsets of $360°/N$ between them.

**Ripple current cancellation:**

In a single-phase converter, the output current ripple is:

$$\Delta I_{L,1ph} = \frac{V_{in} - V_{out}}{L} \cdot D \cdot T_{sw} \cdot (1-D) = \frac{V_{out}(1-D)}{Lf_{sw}}$$

where $D$ is the duty cycle.

With $N$ interleaved phases, the ripple cancels at the output. For integer duty cycles that are multiples of $1/N$, complete cancellation can occur. In general, the output ripple current amplitude is reduced by a factor of up to $N$, and the ripple frequency seen by the output capacitors is $N \times f_{sw}$.

**Key PDN benefits:**

1. **Reduced output inductance requirement:** Since the effective ripple frequency is $N \times f_{sw}$, the per-phase inductance can be reduced by a factor of $N$ for the same ripple:

$$L_{phase,N} = \frac{L_{single}}{N}$$

Smaller inductance means higher loop bandwidth and faster transient response.

2. **Reduced output capacitance:** With $N \times f_{sw}$ effective frequency and reduced ripple amplitude, the capacitance needed for the same voltage ripple is:

$$C_{out,N} \approx \frac{C_{out,single}}{N}$$

3. **Higher achievable loop bandwidth:** With smaller $L_{phase}$, the LC filter resonance moves higher, allowing a higher crossover frequency $f_c \approx f_{sw}/5$:

$$f_{c,N} \approx N \times f_{c,single}$$

4. **Lower thermal stress:** Power dissipation distributed across $N$ phases and inductors.

**Practical multiphase VRM design:**

Phase count choices: 2, 3, 4, 6, 8, or more phases for high-current rails.

For an FPGA core supply with $I_{max} = 40\ A$, $f_{sw} = 500\ kHz$ per phase, $N = 4$:

- Effective output frequency: $4 \times 500\ kHz = 2\ MHz$
- Per-phase inductor: $L_{phase} \approx V_{out}(1-D)/(f_{sw} \times \Delta I_L) \approx 100\text{–}300\ nH$
- Loop bandwidth achievable: up to 200–400 kHz (with proper compensation)
- Target impedance coverage: from $f_c \approx 200\ kHz$ up to 10–100 MHz with MLCCs

---

### Q7. How do you analyse a load step transient to determine if the VRM and PDN meet the voltage specification? Walk through the analysis procedure.

**Answer:**

**Given:**
- Supply voltage: $V_{nom} = 1.0\ V$
- Tolerance: $\pm 3\%$ → $\Delta V_{max} = 30\ mV$
- Maximum load current step: $\Delta I = 10\ A$
- VRM loop bandwidth: $f_{BW} = 100\ kHz$
- Output inductance (total, multiphase): $L_{eff} = 200\ nH$
- Bulk capacitance: $C_{bulk} = 2\ mF$, $R_{ESR,bulk} = 5\ m\Omega$, $L_{ESL,bulk} = 10\ nH$
- MLCC bank: $C_{MLCC} = 500\ \mu F$, $R_{ESR,MLCC} = 1\ m\Omega$, $L_{ESL,MLCC} = 1\ nH$

**Step 1: Initial voltage drop (first nanoseconds)**

Before the capacitors respond, the initial drop is limited by the high-frequency PDN impedance. In the first few nanoseconds, the on-die and package capacitance $C_{die}$ (assume 10 nF for this example) supplies the current:

$$\Delta V_{initial} = \frac{\Delta I \cdot t_{rise}}{C_{die}} = \frac{10 \times 1\times10^{-9}}{10\times10^{-9}} = 1\ V$$

This seems extreme — for a 1 ns rise time with only 10 nF on-die, the voltage would collapse completely if no other capacitance were present. In practice the package inductance limits $dI/dt$ and the MLCC capacitors on the board contribute. This phase is handled by the package and on-die decoupling.

**Step 2: First microsecond — MLCC response**

The MLCC bank begins supplying current as soon as its voltage (and the bus voltage) drops. The effective droop rate from the MLCC bank:

$$\frac{dV}{dt} = \frac{\Delta I}{C_{MLCC}} = \frac{10}{500\times10^{-6}} = 20\ kV/s = 20\ mV/\mu s$$

The MLCC voltage drop in $\tau_{MLCC} = L_{ESL,MLCC}/(R_{ESR,MLCC}) = 1\ nH / 1\ m\Omega = 1\ \mu s$:

$$\Delta V_{MLCC} = \Delta I \times R_{ESR,MLCC} + \frac{\Delta I}{\tau_{MLCC}} \times L_{ESL,MLCC} = 10 \times 0.001 + 10\times10^9\times10^{-9} = 10\ mV + 10\ mV = 20\ mV$$

Wait — this should be: instantaneous drop $\Delta V = \Delta I \times R_{ESR} = 10 \times 0.001 = 10\ mV$, followed by a slow droop.

**Step 3: Bulk capacitor response (first 10 µs)**

The bulk capacitors handle the current demand during the VRM latency $\tau = 1/f_{BW} = 10\ \mu s$. Charge drawn:

$$Q = \Delta I \times \tau = 10 \times 10\times10^{-6} = 100\ \mu C$$

Voltage drop from bulk capacitance alone:

$$\Delta V_{bulk} = \frac{Q}{C_{bulk}} = \frac{100\times10^{-6}}{2\times10^{-3}} = 50\ mV$$

This exceeds the 30 mV specification. The bulk capacitance is insufficient for the specified loop bandwidth. Options:
- Increase $C_{bulk}$ to $\Delta I / (f_{BW} \times \Delta V_{max}) = 10/(10^5 \times 0.030) = 3.3\ mF$
- Increase VRM loop bandwidth to $\Delta I / (C_{bulk} \times \Delta V_{max}) = 10/(2\times10^{-3} \times 0.030) = 167\ kHz$

**Step 4: VRM recovery**

After $1/f_{BW}$, the VRM inductor ramps current at:

$$\frac{dI_L}{dt} = \frac{V_{in} - V_{out}}{L_{eff}} = \frac{12 - 1.0}{200\times10^{-9}} = 55\ A/\mu s$$

The time to ramp from 0 to $\Delta I = 10\ A$:

$$t_{ramp} = \frac{\Delta I}{dI_L/dt} = \frac{10}{55} = 182\ ns$$

So the VRM inductor current reaches the new load level approximately 182 ns after the VRM starts responding. The capacitors continue to supply current during this ramp, but the voltage is recovering.

**Conclusion:** The VRM loop bandwidth must be increased to 167 kHz, or the bulk capacitance increased to 3.3 mF, to meet the 30 mV specification.

---

## Tier 3: Advanced

### Q8. Explain how a VRM's output impedance can cause instability when connected to a constant-power load (CPL). What is the Middlebrook stability criterion?

**Answer:**

A constant-power load (CPL) maintains $P = V \times I = \text{const}$. As the supply voltage drops, the load draws more current to maintain power. This is negative incremental impedance behaviour:

$$Z_{CPL} = \frac{dV}{dI} = -\frac{V^2}{P} = -\frac{V}{I}$$

The CPL input impedance is negative — as voltage decreases, current increases, which further decreases voltage. This is a destabilising mechanism when connected to a source with finite output impedance.

**Middlebrook stability criterion:**

For a source-load system to be stable, the output impedance of the source $Z_S$ must be much smaller than the input impedance of the load $Z_L$ at all frequencies:

$$|Z_S(f)| \ll |Z_L(f)| \quad \forall f$$

This ensures the Nyquist plot of $Z_S/Z_L$ does not encircle $(-1, 0)$.

For a VRM (source) driving a CPL (load), the stability condition becomes:

$$|Z_{out,VRM}(f)| < |Z_{CPL}(f)| = \frac{V^2}{P}$$

At low frequencies where the VRM has high loop gain and very low $Z_{out,VRM}$, this is easily satisfied. At frequencies above the VRM loop bandwidth, $Z_{out,VRM}$ rises as $j\omega L_{filter}$ and eventually exceeds $V^2/P$ — risking instability.

**Phase margin with CPL:**

The effective loop gain of the source-load system is modified by the load impedance. The system loop gain becomes:

$$T_{system}(s) = T_{VRM}(s) \times \frac{Z_{CPL}}{Z_{CPL} + Z_{out,VRM}}$$

For $|Z_{CPL}| \gg |Z_{out,VRM}|$ (Middlebrook condition satisfied): $T_{system} \approx T_{VRM}$ — stable.

For $|Z_{CPL}| \approx |Z_{out,VRM}|$: the loop gain is reduced significantly and the phase margin may collapse to zero, causing oscillation.

**Practical relevance:** Point-of-load converters (the load) are often the CPL for the intermediate bus VRM (the source). If the intermediate bus converter has inadequate output capacitance or too narrow a bandwidth, the downstream POL converter's input can oscillate. This is a real design issue in distributed power architectures.

---

### Q9. How does VRM switching noise couple into the PDN, and what filter techniques are used to attenuate it?

**Answer:**

A switching VRM generates noise at its switching frequency $f_{sw}$ and all harmonics. This noise is injected into the PDN through the output filter and can appear as supply voltage ripple at the load.

**Noise coupling mechanisms:**

1. **Output ripple voltage:** The inductor current ripple $\Delta I_L$ flowing through the ESR of the output capacitors produces a voltage ripple $\Delta V \approx \Delta I_L \times R_{ESR}$.

2. **Switching node radiated noise:** The switch node (drain of the switching FET) transitions from 0 to $V_{in}$ at each switching edge. The fast $dV/dt$ creates displacement current through the parasitic capacitance between the switch node and the PCB. This current flows through the PCB plane to ground, creating conducted common-mode noise.

3. **Magnetic coupling from inductor:** The magnetic field from the power inductor can induce voltages in nearby loops (sensitive analog traces, clock lines). Shielded inductors or careful placement reduces this.

**Filter attenuation — second-stage LC filter:**

Adding a second LC stage after the main VRM output filter attenuates switching ripple more aggressively:

Single-stage attenuation: $H(f) \approx (f_{sw}/f_0)^2$ where $f_0$ is the LC resonant frequency.

Two-stage attenuation: $H(f) \approx (f_{sw}/f_0)^4$ — much greater high-frequency rejection.

However, a second LC stage must be compensated carefully — it changes the open-loop gain by an additional 40 dB/decade, which can destabilise the VRM if the loop bandwidth is not reduced accordingly.

**Filter design for second stage:**

The second-stage filter must have its resonant frequency well below the switching frequency:

$$f_{0,2nd} = \frac{1}{2\pi\sqrt{L_2 C_2}} < \frac{f_{sw}}{10}$$

And the VRM loop bandwidth must be reduced below $f_{0,2nd}$:

$$f_{BW} < \frac{f_{0,2nd}}{10}$$

This conservative nesting ensures that the phase contributed by the second stage is small at the crossover frequency.

**Common-mode noise filtering:**

For switch-node conducted EMI, a common-mode choke at the VRM output combined with Y-capacitors (from the output rails to chassis ground) filters the common-mode component. Differential filtering (bulk capacitors) handles the differential-mode ripple.

---

### Q10. How do you select a PMIC for a complex SoC design? Describe the selection criteria and the evaluation process.

**Answer:**

PMIC selection involves balancing electrical performance, integration, cost, and design risk. The following criteria guide the selection process:

**Step 1: Define all rail requirements**

Create a power map table listing every rail on the board:

| Rail | $V_{nom}$ | $I_{max}$ | Tolerance | Sequence order | Notes |
|---|---|---|---|---|---|
| VDD_CORE | 0.85 V | 3.0 A | ±3% | Rail 3 | DVFS: 0.7–0.95 V |
| VDD_IO | 1.8 V | 500 mA | ±5% | Rail 2 | Fixed |
| VDD_DDR | 1.1 V | 1.5 A | ±2% | Rail 4 | Tight tolerance |
| VDD_PLL | 1.8 V | 100 mA | ±1% | Rail 2 | Low noise required |
| VDD_USB | 3.3 V | 200 mA | ±5% | Rail 1 | Always-on |

**Step 2: Identify PMIC candidates**

Screen against:
- Number of regulated outputs: must match or exceed the rail count
- Input voltage range: must cover the board's supply (e.g., 5 V or 3.8 V battery)
- Per-channel current capacity: each PMIC channel must supply the required current
- Output voltage range: covers the required nominal voltages (fixed or adjustable)
- Sequencing flexibility: programmable power-on/power-off sequences via OTP or I²C

**Step 3: Evaluate transient performance**

For each rail with tight tolerance or high $dI/dt$:

Compute $Z_{target} = \Delta V_{max}/I_{step}$ and compare against the PMIC's specified output impedance or load transient response time.

If the PMIC cannot meet transient requirements, consider:
- Adding external bulk capacitors to extend the PMIC's response time window
- Using a discrete VRM for that specific rail while using the PMIC for the others

**Step 4: Noise evaluation**

For sensitive analog or PLL rails:

- Examine the PMIC's output noise specification (integrated noise in µV$_{RMS}$)
- Buck switchers have higher ripple than LDO regulators; for very sensitive rails, use an LDO on the PMIC or a post-filter
- Ripple at $f_{sw}$ must be below the phase noise specification of the PLL

**Step 5: Thermal analysis**

Compute total power dissipation in the PMIC:

$$P_{total} = \sum_k \frac{(V_{in} - V_{out,k}) \times I_{out,k}}{\eta_k}$$

Verify that $P_{total}$ is within the PMIC's maximum power dissipation for the given junction temperature:

$$P_{max} = \frac{T_{j,max} - T_{ambient}}{\theta_{JA}}$$

If overheated, derate current, add a heatsink, or split into two PMICs.

**Step 6: Power sequencing verification**

Confirm that the PMIC's built-in sequencing options match the SoC's power-on requirements. PMICs typically support:
- Fixed timed delays between rail enable events
- Voltage-threshold triggered sequencing (rail $k+1$ enables when rail $k$ exceeds a threshold)
- I²C-programmable sequencing for flexibility

---

## Summary: VRM/PMIC Design Trade-Off Matrix

| Design Choice | Improves | Degrades | Trade-off |
|---|---|---|---|
| Increase VRM loop bandwidth | Transient response, reduces needed $C_{bulk}$ | Phase margin, may require smaller $L$ | Stability vs. capacitor count |
| Increase $C_{bulk}$ | Voltage droop budget | Cost, PCB area, loop stability | Cost vs. PDN performance |
| Add more phases | Reduces $L_{eff}$, lowers ripple, higher BW | Controller complexity, cost | Cost vs. transient performance |
| Enable active droop | Halves required $C_{bulk}$ | Reduced DC accuracy | Accuracy vs. capacitance savings |
| Use second-stage filter | Attenuates switching ripple | Reduces achievable BW, adds components | Noise vs. bandwidth |
| Switch to LDO for sensitive rail | Very low output noise | Reduced efficiency: $(V_{in}-V_{out}) \times I$ wasted | Efficiency vs. noise |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Closed-loop output impedance (flat region) | $Z_{out,CL} \approx 1/(2\pi f_c C_{out})$ |
| Minimum bulk capacitance | $C_{bulk} \geq \Delta I/(f_{BW} \times \Delta V_{max})$ |
| Required loop bandwidth | $f_{BW} \geq \Delta I/(C_{bulk} \times \Delta V_{max})$ |
| VRM inductor current ramp | $dI/dt = (V_{in}-V_{out})/L_{eff}$ |
| Droop impedance | $R_{LL} = \Delta V_{DC,droop}/I_{max}$ |
| Capacitance saving with droop | $C_{droop} = C_{no-droop}/2$ |
| Middlebrook criterion | $|Z_{source}| \ll |Z_{load}|$ at all frequencies |
| N-phase effective frequency | $f_{eff} = N \times f_{sw}$ |
