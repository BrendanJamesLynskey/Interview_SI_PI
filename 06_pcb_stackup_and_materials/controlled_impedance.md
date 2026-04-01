# Controlled Impedance

## Overview

Controlled impedance refers to the practice of designing, specifying, and manufacturing PCB traces such that their characteristic impedance $Z_0$ matches the target value within a defined tolerance. High-speed digital interfaces depend on controlled impedance to minimise reflections, maintain signal integrity, and comply with bus standards. This document covers the geometry of common trace types (microstrip and stripline), closed-form impedance equations, the effect of manufacturing variation, and the trade-offs inherent in impedance control over a production run.

---

## Tier 1: Fundamentals

### Q1. What is characteristic impedance, and why must PCB traces be designed to a target value?

**Answer:**

Characteristic impedance $Z_0$ is the ratio of voltage to current in a travelling wave on an infinitely long (or perfectly terminated) transmission line. It is set entirely by the distributed inductance $L'$ (per unit length) and capacitance $C'$ (per unit length) of the line:

$$Z_0 = \sqrt{\frac{L'}{C'}}$$

It has nothing to do with the line's length or the signal's frequency (for a lossless line).

**Why controlled impedance is required:**

When a signal travelling on a transmission line reaches a discontinuity — a change in $Z_0$ — a portion of the energy is reflected back toward the source. The reflection coefficient is:

$$\Gamma = \frac{Z_L - Z_0}{Z_L + Z_0}$$

For a source at 50 Ω driving a 60 Ω trace: $\Gamma = 10/110 = 0.09$, or 9% of the voltage is reflected. For a 75 Ω trace: $\Gamma = 25/125 = 0.20$, or 20% reflected. Reflections cause:

- **Signal distortion:** Reflections return to the source, reflect again, and appear as glitches or level errors at the receiver.
- **Inter-symbol interference (ISI):** Reflected energy from one bit period contaminates subsequent bits.
- **Radiation:** The impedance mismatch creates a site from which the signal radiates as an electromagnetic disturbance.

Most high-speed standards specify a characteristic impedance target:

| Interface | Target $Z_0$ |
|---|---|
| Single-ended (PCIe, DDR, USB) | 50 Ω |
| Differential pair (LVDS, PCIe, USB diff) | 100 Ω differential |
| DDR4/DDR5 single-ended | 40–50 Ω (depending on topology) |
| Ethernet 100BASE-T | 100 Ω |
| RF/coax | 50 Ω or 75 Ω |

**Tolerance:** Most digital standards accept ±10% (i.e., 45–55 Ω for a 50 Ω target). Some analog and RF applications require ±5% or tighter.

---

### Q2. State the closed-form impedance equation for a microstrip trace. Identify each parameter and explain how increasing the trace width affects impedance.

**Answer:**

**Microstrip geometry:**

A microstrip consists of a trace of width $W$ and thickness $T$ on the surface of a dielectric of height $h$, backed by a continuous reference plane.

```
    ┌────────────┐  ← Trace (width W, thickness T)
    └────────────┘
    ││ dielectric ││  ← height h
════════════════════  ← Reference plane (GND)
```

**Approximate closed-form impedance (IPC-2141A equation):**

$$Z_0 = \frac{87}{\sqrt{\epsilon_r + 1.41}} \ln\left(\frac{5.98h}{0.8W + T}\right)$$

Valid for $W/h < 1$ (narrow traces). For wider traces, the full Hammerstad-Jensen model is preferred.

**Parameters:**

| Parameter | Symbol | Typical range |
|---|---|---|
| Trace width | $W$ | 50–300 µm |
| Trace thickness | $T$ | 17–35 µm (½ oz or 1 oz copper) |
| Dielectric height | $h$ | 50–250 µm |
| Relative permittivity | $\epsilon_r$ | 3.5–4.5 |
| Target impedance | $Z_0$ | 50 Ω (typical) |

**Effect of increasing $W$:**

Increasing $W$ increases the capacitance per unit length $C'$ (more overlap area between trace and plane) while decreasing the inductance per unit length $L'$ (wider conductor). Since $Z_0 = \sqrt{L'/C'}$, a wider trace has **lower characteristic impedance**. This is the dominant sensitivity:

$$\frac{\partial Z_0}{\partial W} < 0 \quad \text{(monotonically decreasing)}$$

Approximately, for a microstrip near 50 Ω:

$$\frac{\Delta Z_0}{Z_0} \approx -0.6 \times \frac{\Delta W}{W}$$

A 10% increase in trace width causes approximately a 6% decrease in impedance. For a 50 Ω target, a $+10\%$ width increase produces $\approx 47$ Ω — within the ±10% tolerance.

**Numerical example:**

Target: $Z_0 = 50$ Ω on FR4 ($\epsilon_r = 4.2$), $h = 100$ µm, $T = 18$ µm.

Using the IPC equation:
$$50 = \frac{87}{\sqrt{4.2 + 1.41}} \ln\left(\frac{598}{0.8W + 18}\right)$$

$$50 = \frac{87}{\sqrt{5.61}} \ln\left(\frac{598}{0.8W + 18}\right) = 36.72 \ln\left(\frac{598}{0.8W + 18}\right)$$

$$\ln\left(\frac{598}{0.8W + 18}\right) = \frac{50}{36.72} = 1.362$$

$$\frac{598}{0.8W + 18} = e^{1.362} = 3.904$$

$$0.8W + 18 = \frac{598}{3.904} = 153.2 \implies W = \frac{153.2 - 18}{0.8} = 169 \text{ µm}$$

For a 50 Ω microstrip on FR4 with 100 µm prepreg and 1 oz copper: trace width $\approx$ **169 µm** (approximately 6.7 mil).

---

### Q3. What is a stripline, and how does its impedance equation differ from microstrip?

**Answer:**

**Stripline geometry:**

A stripline is a trace buried between two reference planes (typically two ground planes). Unlike microstrip, the trace is entirely embedded in the dielectric. There is no air-dielectric interface.

```
════════════════════  ← Upper reference plane
    ┌────────────┐   ← Trace (width W, thickness T)
    └────────────┘
    ││ dielectric ││  ← total height b between planes
════════════════════  ← Lower reference plane
```

**Closed-form impedance (for centred symmetric stripline):**

$$Z_0 = \frac{60}{\sqrt{\epsilon_r}} \ln\left(\frac{4b}{0.67\pi(0.8W + T)}\right)$$

where $b$ is the total dielectric height between the two reference planes.

**Differences from microstrip:**

| Property | Microstrip | Stripline |
|---|---|---|
| Reference planes | 1 (below) | 2 (above and below) |
| Dielectric environment | Partial air, partial dielectric | Fully embedded in dielectric |
| Effective $\epsilon_r$ | $\epsilon_{eff} < \epsilon_r$ (air reduces average) | $= \epsilon_r$ (no air) |
| Radiation | Higher (no upper shielding) | Lower (enclosed between planes) |
| Impedance formula coefficient | 87 / $\sqrt{\epsilon_r + 1.41}$ | 60 / $\sqrt{\epsilon_r}$ |
| Trace width for 50 Ω | Wider (lower $\epsilon_{eff}$) | Narrower (higher $\epsilon_r$) |
| Crosstalk | Higher (field extends upward into air) | Lower (fields confined between planes) |

**Effect of asymmetric stripline:**

If the trace is not centred between the two planes (distance $h_1$ to the upper plane, $h_2$ to the lower plane, $h_1 \neq h_2$), the impedance can be approximated using the Wadell asymmetric formula. The key point is that the closer reference plane dominates the capacitance: a trace 50 µm from the upper plane and 150 µm from the lower plane will have most of its return current in the upper plane.

**Numerical example:**

Target: $Z_0 = 50$ Ω stripline, $\epsilon_r = 3.7$ (Megtron 6), $b = 200$ µm, $T = 18$ µm.

$$50 = \frac{60}{\sqrt{3.7}} \ln\left(\frac{4 \times 200}{0.67\pi(0.8W + 18)}\right) = 31.18 \ln\left(\frac{800}{2.104(0.8W + 18)}\right)$$

$$\ln\left(\frac{800}{2.104(0.8W + 18)}\right) = \frac{50}{31.18} = 1.604$$

$$\frac{800}{2.104(0.8W + 18)} = e^{1.604} = 4.973$$

$$2.104(0.8W + 18) = 160.9 \implies 0.8W + 18 = 76.5 \implies W = 72.9 \text{ µm}$$

For a 50 Ω stripline on Megtron 6 with 200 µm total dielectric: trace width $\approx$ **73 µm** (approximately 2.9 mil).

---

## Tier 2: Intermediate

### Q4. What manufacturing variations affect controlled impedance, and how is each budgeted?

**Answer:**

The as-fabricated impedance of a PCB trace deviates from the designed value due to process variations in four main parameters: trace width, dielectric thickness, dielectric constant, and trace thickness. Each has a different variation profile and magnitude.

**1. Trace width variation ($\Delta W$):**

Trace width is controlled by the photolithographic etch process. The designed width in the Gerber file undergoes etch factor correction (the trace is drawn slightly wider to compensate for etch undercut). Typical width variation after etching:

| Copper weight | Width variation |
|---|---|
| ½ oz (18 µm) | ±15–25 µm |
| 1 oz (35 µm) | ±20–30 µm |
| 2 oz (70 µm) | ±40–60 µm |

For a 100 µm trace: ±20 µm represents ±20% of width — a dominant variation source. For a 200 µm trace: ±20 µm is ±10%.

**Impedance sensitivity:**

$$\frac{\Delta Z_0}{Z_0} \approx -\frac{1}{2} \cdot \frac{\Delta W}{W - W_{eff}}$$

where $W_{eff}$ accounts for trace thickness geometry. For a 50 Ω, 170 µm microstrip with ±20 µm width variation:

$$\Delta Z_0 \approx \pm \frac{20}{170} \times 0.6 \times 50 \approx \pm 3.5 \text{ Ω} \quad (\pm 7\%)$$

**2. Dielectric thickness variation ($\Delta h$):**

Prepreg thickness varies due to resin flow during lamination. Typical variation:

- Prepreg (standard): ±10–15% of nominal thickness
- Core (tight tolerance): ±5–8% of nominal

Impedance sensitivity to $h$ (microstrip, $Z_0 \propto \ln(h)$ approximately):

$$\frac{\Delta Z_0}{Z_0} \approx +\frac{\Delta h}{h \cdot \ln(5.98h/(0.8W+T))}$$

For $h = 100$ µm, ±15% variation: $\Delta h = \pm 15$ µm.

$$\Delta Z_0 \approx \pm \frac{15}{100 \times 1.36} \times 50 \approx \pm 5.5 \text{ Ω} \quad (\pm 11\%)$$

**3. Dielectric constant variation ($\Delta\epsilon_r$):**

$D_k$ varies with resin content (resin-to-glass ratio), which varies by ±5–10% across a panel. For FR4:

| Condition | $D_k$ variation |
|---|---|
| Lot-to-lot | ±0.2–0.4 |
| Within panel | ±0.1–0.15 |

Impedance sensitivity:

$$\frac{\Delta Z_0}{Z_0} \approx -\frac{1}{2}\frac{\Delta \epsilon_r}{\epsilon_r + 1.41}$$

For FR4, $\epsilon_r = 4.2$, $\Delta\epsilon_r = \pm 0.3$:

$$\Delta Z_0 \approx \pm \frac{0.3}{2 \times 5.61} \times 50 \approx \pm 1.3 \text{ Ω} \quad (\pm 2.6\%)$$

**4. Trace thickness variation ($\Delta T$):**

Copper foil thickness varies by ±10–15% from nominal. The plated copper adds additional thickness variation of ±3–5 µm from electroless/electrolytic plating.

Effect on impedance is typically the smallest of the four sources: $|\Delta Z_0| \lesssim 1$–$2\%$ for ±10% thickness variation on standard traces.

**RSS (Root Sum Square) combined tolerance:**

Assuming independent variations:

$$\Delta Z_{total} = \sqrt{(\Delta Z_W)^2 + (\Delta Z_h)^2 + (\Delta Z_{\epsilon})^2 + (\Delta Z_T)^2}$$

$$= \sqrt{3.5^2 + 5.5^2 + 1.3^2 + 0.8^2} \approx \sqrt{12.25 + 30.25 + 1.69 + 0.64} \approx \sqrt{44.8} \approx 6.7 \text{ Ω}$$

This is ±13.5% — exceeding the typical ±10% specification. This illustrates why fabricators do not guarantee ±10% without process controls. Achieving ±10% requires either:

- Using controlled prepreg (tighter $h$ tolerance)
- Reducing copper weight (thinner traces etch more uniformly)
- Coupon-based measurement and trace width adjustment per panel

---

### Q5. How does differential impedance differ from twice the single-ended impedance? What geometric factors cause the difference?

**Answer:**

**Differential impedance definition:**

For a differential pair driven with signals $+V/2$ and $-V/2$ (equal and opposite), the differential impedance is:

$$Z_{diff} = 2 Z_0 \left(1 - k_{em}\right)$$

where $k_{em}$ is the electromagnetic coupling coefficient between the two traces. Equivalently:

$$Z_{diff} = 2(Z_{self} - Z_{mutual})$$

where $Z_{self}$ is the single-ended impedance of one trace and $Z_{mutual}$ is the mutual impedance between the traces.

**Why $Z_{diff} \neq 2 Z_0$:**

When two traces are close together and carry differential signals, the electric and magnetic fields from each trace partially cancel in the region between the traces. This reduces the effective inductance (mutual inductance subtracts from self-inductance) and increases the effective capacitance (mutual capacitance adds to self-capacitance). Both effects reduce $Z_{diff}$ below $2Z_0$.

**Coupling dependence on trace spacing:**

The coupling coefficient $k_{em}$ increases as the trace separation $s$ decreases relative to the trace height $h$ above the reference plane:

$$k_{em} \approx \frac{s/h}{a + b \cdot (s/h)^2} \quad (\text{approximate empirical form})$$

For a tightly coupled pair ($s = 0.1$ mm, $h = 0.1$ mm):

$$Z_{diff} \approx 2 \times 50 \times (1 - 0.15) = 85 \text{ Ω}$$

Increasing the spacing to $s = 0.3$ mm (3× the dielectric height):

$$Z_{diff} \approx 2 \times 50 \times (1 - 0.03) = 97 \text{ Ω}$$

**Practical consequence — designing for 100 Ω differential:**

To achieve $Z_{diff} = 100$ Ω, the single-ended impedance of each trace must be slightly above 50 Ω (to compensate for the coupling reduction). The designer must specify both the trace width and the trace gap in the impedance control note:

- **Tight coupling (s = 1× trace width):** Single-ended $Z_0 \approx 55$–60 Ω per trace to achieve $Z_{diff} = 100$ Ω
- **Loose coupling (s = 3× trace width or more):** Single-ended $Z_0 \approx 50$–52 Ω per trace

A common mistake is designing a differential pair to have each trace at exactly 50 Ω single-ended and expecting the differential impedance to be 100 Ω. In practice, it will be 90–97 Ω depending on the gap — a 3–10% miss.

**Manufacturing implication:** Differential impedance is more sensitive to the gap $s$ than single-ended impedance is to width $W$. If the etch process narrows both traces and the gap simultaneously, the coupling increases and $Z_{diff}$ may decrease more than the reduction in $Z_{self}$ alone would predict.

---

### Q6. Describe how a fabricator validates controlled impedance on a production panel. What is an impedance coupon and how is it measured?

**Answer:**

**Impedance coupon:**

An impedance coupon is a set of representative transmission-line test structures fabricated on the same panel as the production boards. The coupon contains traces with identical stackup, trace width, and geometry to the production impedance-controlled nets. After fabrication, the coupon is measured before the panels are shipped or routed.

**Coupon structure:**

A minimum impedance coupon for a 6-layer board contains:

- One microstrip test trace (L1), 150–200 mm long
- One microstrip test trace (L6), 150–200 mm long
- One or two stripline test traces (L3 or L4), 150–200 mm long
- Each trace terminated with a short or open at one end (or left unterminated, measured TDR)

The trace width on the coupon exactly matches the production trace width specification for each controlled-impedance layer.

**Measurement method — TDR (Time-Domain Reflectometry):**

The coupon is measured using a TDR test system (e.g., Tektronix/Keysight TDR module):

1. A fast step voltage (rise time ≤ 35 ps) is launched from a 50 Ω source through a coaxial probe to the coupon trace.
2. The TDR waveform shows the reflected voltage vs. time. On a uniform 50 Ω trace, the reflection is flat (no reflected wave). Any deviation from flat shows a local impedance variation.
3. The impedance at any point on the trace is read from the TDR voltage level:
   $$Z(t) = Z_0 \cdot \frac{1 + \rho(t)}{1 - \rho(t)}$$
   where $\rho(t) = V_{reflected}(t) / V_{incident}$.
4. The fabricator reads the average TDR impedance over the middle region of the trace (avoiding launch and termination artefacts) and compares it to the target ±10% window.

**IPC-2141A coupon requirements:**

- Coupon trace length: minimum 75 mm usable (IPC-2141A clause 4.7)
- Measurement averaging: over 20–60% of the trace length
- TDR rise time: ≤ 35 ps to resolve trace features
- Calibration: through a 50 Ω reference at the probe interface

**What the coupon does NOT guarantee:**

The coupon measures the trace impedance on a specific panel at a specific coupon location. It does not directly verify:
- Impedance uniformity across the entire panel
- Impedance on individual signal nets (which may have corners, vias, and voids that alter local geometry)
- Consistency across panel lots

For critical applications, some designers specify in-circuit TDR measurement of production signal traces, or use statistical sampling of multiple coupons per panel lot.

---

## Tier 3: Advanced

### Q7. A DDR5 design requires 40 Ω single-ended traces on L1 (microstrip) and 40 Ω traces on L3 (stripline). The stackup uses Megtron 6 ($\epsilon_r = 3.7$). Calculate the required trace widths for both layers, given: L1 prepreg $h = 85$ µm, $T = 18$ µm; L3 total dielectric $b = 180$ µm, $T = 18$ µm. Then determine the impedance sensitivity to a ±20 µm width variation.

**Answer:**

**Step 1 — L1 microstrip at 40 Ω:**

Using the IPC closed-form equation:

$$40 = \frac{87}{\sqrt{\epsilon_r + 1.41}} \ln\left(\frac{5.98h}{0.8W + T}\right) = \frac{87}{\sqrt{3.7 + 1.41}} \ln\left(\frac{5.98 \times 85}{0.8W + 18}\right)$$

$$= \frac{87}{\sqrt{5.11}} \ln\left(\frac{508.3}{0.8W + 18}\right) = 38.50 \ln\left(\frac{508.3}{0.8W + 18}\right)$$

$$\ln\left(\frac{508.3}{0.8W + 18}\right) = \frac{40}{38.50} = 1.039$$

$$\frac{508.3}{0.8W + 18} = e^{1.039} = 2.827$$

$$0.8W + 18 = \frac{508.3}{2.827} = 179.8 \implies W_{L1} = \frac{179.8 - 18}{0.8} = 202 \text{ µm}$$

**L1 microstrip trace width for 40 Ω: 202 µm (approximately 8.0 mil).**

**Step 2 — L3 stripline at 40 Ω:**

Using the stripline equation:

$$40 = \frac{60}{\sqrt{3.7}} \ln\left(\frac{4 \times 180}{0.67\pi(0.8W + 18)}\right) = 31.18 \ln\left(\frac{720}{2.104(0.8W + 18)}\right)$$

$$\ln\left(\frac{720}{2.104(0.8W + 18)}\right) = \frac{40}{31.18} = 1.283$$

$$\frac{720}{2.104(0.8W + 18)} = e^{1.283} = 3.608$$

$$2.104(0.8W + 18) = \frac{720}{3.608} = 199.6 \implies 0.8W + 18 = 94.87 \implies W_{L3} = \frac{94.87 - 18}{0.8} = 96.1 \text{ µm}$$

**L3 stripline trace width for 40 Ω: 96 µm (approximately 3.8 mil).**

Note: The stripline trace is approximately half the width of the microstrip trace for the same impedance target, primarily because the stripline has full $\epsilon_r = 3.7$ (no air correction) and a fully enclosed geometry.

**Step 3 — Impedance sensitivity to ±20 µm width variation:**

**L1 microstrip sensitivity:**

At the design point $W_{L1} = 202$ µm, evaluate:

$$\frac{dZ_0}{dW}\bigg|_{W=202} \approx -\frac{Z_0}{W_{eff}} \times \frac{0.8}{0.8W + T} \times \frac{1}{\ln\left(\frac{5.98h}{0.8W+T}\right)}$$

where $W_{eff} = 0.8W + T = 0.8 \times 202 + 18 = 179.6$ µm.

Numerically:

$$\frac{dZ_0}{dW} \approx -\frac{40}{202} \times \frac{0.8}{\ln(508.3/179.8)} = -0.198 \times \frac{0.8}{1.039} = -0.152 \text{ Ω/µm}$$

For $\Delta W = \pm 20$ µm:

$$\Delta Z_0^{L1} = \pm 20 \times 0.152 = \pm 3.0 \text{ Ω} \quad (\pm 7.5\%)$$

**L3 stripline sensitivity:**

At $W_{L3} = 96$ µm:

$$\frac{dZ_0}{dW}\bigg|_{W=96} \approx -\frac{40}{96} \times \frac{0.8}{\ln(720/(2.104 \times 94.87))} = -0.417 \times \frac{0.8}{1.283} = -0.260 \text{ Ω/µm}$$

For $\Delta W = \pm 20$ µm:

$$\Delta Z_0^{L3} = \pm 20 \times 0.260 = \pm 5.2 \text{ Ω} \quad (\pm 13\%)$$

**Key insight:** The 96 µm stripline is significantly more sensitive to width variation than the 202 µm microstrip because the same ±20 µm absolute variation represents ±21% of the trace width (vs. ±10% for microstrip). For the ±10% impedance tolerance specification:

- L1 microstrip: ±7.5% from width alone — marginal but within tolerance.
- L3 stripline: ±13% from width alone — **exceeds the ±10% tolerance** from width variation alone before any dielectric variation is added.

**Corrective action for L3:** Either widen the trace to reduce the width sensitivity (but this lowers impedance, requiring a thinner dielectric), or specify tighter etch tolerance for this layer (e.g., ±10 µm instead of ±20 µm), or use the field solver to optimise the geometry accounting for the full variation stack.

---

### Q8. An 8-layer board is being designed for PCIe Gen4. The fabricator's process achieves ±10% impedance tolerance. The PCIe specification allows ±15% from the nominal 85 Ω differential (odd-mode). Derive the nominal target impedance to centre the distribution and maximise the yield margin.

**Answer:**

**PCIe Gen4 differential impedance specification:**

PCIe Base Specification (Gen4) specifies the differential via/trace impedance as 85 Ω ±15% for the mated card-edge connector and PCB channel. The tolerance band is:

$$Z_{diff,min} = 85 \times 0.85 = 72.25 \text{ Ω}$$
$$Z_{diff,max} = 85 \times 1.15 = 97.75 \text{ Ω}$$

**Fabrication process distribution:**

Assume the fabricator's ±10% tolerance represents a ±3σ process capability (i.e., $\sigma_{fab} = Z_{nominal}/30$ per side, or equivalently the process standard deviation for the impedance distribution is approximately $Z_{nominal} \times 0.10 / 3 = Z_{nominal}/30$).

For a nominal design target $Z_{target}$, the fabricated impedance is normally distributed (approximately):

$$Z \sim \mathcal{N}(Z_{target}, \sigma^2) \quad \text{where } \sigma = Z_{target} \times \frac{0.10}{3}$$

**Yield condition:**

For 100% yield (6σ coverage):

$$Z_{target} - 3\sigma \geq Z_{diff,min} \quad \text{and} \quad Z_{target} + 3\sigma \leq Z_{diff,max}$$

Substituting $\sigma = 0.0333 Z_{target}$:

**Lower bound:**
$$Z_{target}(1 - 0.10) \geq 72.25 \implies Z_{target} \geq 80.3 \text{ Ω}$$

**Upper bound:**
$$Z_{target}(1 + 0.10) \leq 97.75 \implies Z_{target} \leq 88.9 \text{ Ω}$$

**Feasible target range:** $80.3 \text{ Ω} \leq Z_{target} \leq 88.9 \text{ Ω}$.

**Optimal target — centering for maximum margin:**

To maximise the margin equally from both specification limits:

$$Z_{target,optimal} = \frac{Z_{diff,min} + Z_{diff,max}}{2} \times \frac{1}{1} = \frac{72.25 + 97.75}{2} = 85 \text{ Ω}$$

Wait — this is the spec nominal. But the spec nominal and the process optimal nominal coincide here only because the spec tolerance (±15%) is larger than the process capability (±10%). In general, if the process variation were biased (e.g., fabrication tends to produce slightly lower impedance due to systematic over-etch), the optimal nominal would be shifted upward.

**Practical adjustment:**

If the fabricator reports a systematic process offset — for example, all measured impedances run 3% below the target — the designer specifies a nominal of:

$$Z_{target} = \frac{85}{1 - 0.03} = 87.6 \text{ Ω}$$

This pre-compensates for the systematic offset, centring the actual distribution at 85 Ω.

**Yield calculation (design target = 85 Ω, ±10% fabrication, ±15% spec):**

Process $3\sigma$ limits: 85 ± 8.5 Ω = [76.5, 93.5] Ω.
Spec limits: [72.25, 97.75] Ω.

Since the $3\sigma$ process limits are entirely within the spec limits, all parts within $3\sigma$ pass. Yield = 99.73% for a $\pm3\sigma$ process.

$z_{lower} = (76.5 - 72.25) / 2.83 = 1.50$ (σ from lower spec to lower 3σ edge). Yield loss below lower spec: $\Phi(-3 - 1.50) = \Phi(-4.5) \approx 3.4 \times 10^{-6}$ — negligible.

**Conclusion:** With ±15% spec and ±10% fabrication capability (3σ), the design is operating with substantial yield margin and no centring adjustment is needed at 85 Ω nominal. The design team should nonetheless verify the fabricator process distribution is centred (not systematically biased) through incoming acceptance testing of impedance coupons.

---

## Quick Reference: Controlled Impedance Equations

**Microstrip (IPC-2141A approximation, $W/h < 1$):**

$$\boxed{Z_0 = \frac{87}{\sqrt{\epsilon_r + 1.41}} \ln\left(\frac{5.98h}{0.8W + T}\right)}$$

**Symmetric stripline:**

$$\boxed{Z_0 = \frac{60}{\sqrt{\epsilon_r}} \ln\left(\frac{4b}{0.67\pi(0.8W + T)}\right)}$$

**Differential microstrip (approximate):**

$$Z_{diff} \approx 2 Z_0 \left[1 - 0.347 e^{-2.9s/h}\right]$$

**Differential stripline (approximate):**

$$Z_{diff} \approx 2 Z_0 \left[1 - 0.374 e^{-2.72s/b}\right]$$

where $s$ is the edge-to-edge gap between the two traces.

**Key sensitivities (approximate, near 50 Ω):**

| Parameter | Sensitivity |
|---|---|
| Trace width +10% | $Z_0$ decreases ~6% |
| Dielectric height +10% | $Z_0$ increases ~4% |
| $\epsilon_r$ +10% | $Z_0$ decreases ~5% |
| Trace thickness +10% | $Z_0$ decreases ~1% |

**Manufacturing tolerance summary:**

| Control level | $Z_0$ tolerance achievable |
|---|---|
| Standard FR4, no impedance control | ±25–35% |
| Standard impedance control (FR4) | ±10% |
| Tight impedance control (low-loss material) | ±7–8% |
| High-accuracy (RF/microwave, specialised fab) | ±5% |
