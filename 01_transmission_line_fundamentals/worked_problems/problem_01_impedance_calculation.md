# Problem 01: Characteristic Impedance Calculation

## Problem Statement

You are designing a PCB stackup for a mixed-signal board that must carry 50 $\Omega$ single-ended signals on both outer (microstrip) and inner (stripline) layers. You are also routing a 100 $\Omega$ differential pair on an inner layer.

The PCB manufacturer provides the following stackup parameters:

| Parameter | Value |
|---|---|
| Dielectric material | FR4 |
| Dk ($\varepsilon_r$) at 1 GHz | 4.3 |
| Prepreg thickness (dielectric height, outer layers) | $h_{MS} = 4\ \text{mil}$ |
| Core thickness (half-spacing, inner stripline) | $h_{SL} = 5\ \text{mil}$ (core only; total $b = 10\ \text{mil}$ between planes) |
| Copper weight (all signal layers) | 0.5 oz ($t = 0.7\ \text{mil}$) |

**Tasks:**

**(a)** Calculate the trace width required for a 50 $\Omega$ microstrip on the outer layer using the IPC-2141A formula.

**(b)** Calculate the trace width required for a 50 $\Omega$ single-ended stripline using the standard closed-form formula.

**(c)** The PCB manufacturer warns that their etch process has a tolerance of $\pm 0.5$ mil on trace width. What is the worst-case impedance range for each geometry from part (a) and (b)?

**(d)** For the 100 $\Omega$ differential pair on the inner layer, determine the single-ended impedance each trace must have and the edge-to-edge spacing required. Assume tight coupling reduces $Z_{diff}$ by 8% relative to $2 Z_0$ when the gap equals 5 mil.

---

## Solution

### Part (a): 50 $\Omega$ Microstrip Width

**Formula (IPC-2141A):**

$$Z_0^{MS} = \frac{87}{\sqrt{\varepsilon_r + 1.41}} \ln\!\left(\frac{5.98\, h}{0.8\, w + t}\right)$$

Setting $Z_0 = 50\ \Omega$, $\varepsilon_r = 4.3$, $h = 4$ mil, $t = 0.7$ mil and solving for $w$:

**Step 1** — Compute the leading coefficient:

$$\frac{87}{\sqrt{4.3 + 1.41}} = \frac{87}{\sqrt{5.71}} = \frac{87}{2.389} = 36.42$$

**Step 2** — Isolate the logarithm:

$$\ln\!\left(\frac{5.98 \times 4}{0.8w + 0.7}\right) = \frac{50}{36.42} = 1.373$$

**Step 3** — Exponentiate:

$$\frac{23.92}{0.8w + 0.7} = e^{1.373} = 3.948$$

**Step 4** — Solve for $w$:

$$0.8w + 0.7 = \frac{23.92}{3.948} = 6.059$$

$$0.8w = 5.359 \implies \boxed{w_{MS} = 6.7\ \text{mil}}$$

**Sanity check:** For FR4 at 4 mil dielectric height, a 50 $\Omega$ microstrip trace of approximately 6–8 mil is a well-known rule of thumb. The result is consistent.

---

### Part (b): 50 $\Omega$ Stripline Width

**Formula:**

$$Z_0^{SL} = \frac{60}{\sqrt{\varepsilon_r}} \ln\!\left(\frac{4b}{0.67\pi (0.8w + t)}\right)$$

Parameters: $Z_0 = 50\ \Omega$, $\varepsilon_r = 4.3$, $b = 10$ mil (total plane-to-plane spacing), $t = 0.7$ mil.

**Step 1** — Leading coefficient:

$$\frac{60}{\sqrt{4.3}} = \frac{60}{2.074} = 28.93$$

**Step 2** — Isolate the logarithm:

$$\ln\!\left(\frac{4 \times 10}{0.67\pi (0.8w + 0.7)}\right) = \frac{50}{28.93} = 1.728$$

**Step 3** — Exponentiate:

$$\frac{40}{0.67\pi (0.8w + 0.7)} = e^{1.728} = 5.630$$

$$0.67\pi (0.8w + 0.7) = \frac{40}{5.630} = 7.104$$

$$0.8w + 0.7 = \frac{7.104}{0.67\pi} = \frac{7.104}{2.104} = 3.376$$

$$0.8w = 2.676 \implies \boxed{w_{SL} = 3.3\ \text{mil}}$$

**Note on stripline geometry:** The stripline trace is narrower than the microstrip trace (3.3 vs 6.7 mil) for the same 50 $\Omega$ target. This is because the stripline is fully immersed in dielectric (higher effective Dk, lower $Z_0$), requiring a narrower trace to raise $Z_0$ back to 50 $\Omega$. The stripline geometry also has a thinner total dielectric (10 mil plane-to-plane vs. 4 mil for microstrip), which further increases capacitance and requires a narrower trace to compensate.

---

### Part (c): Impedance Tolerance from Etch Variation

The sensitivity of $Z_0$ to trace width is found by the partial derivative of the formula. For a small change $\Delta w$ in trace width around the nominal value:

**Microstrip sensitivity:**

Differentiating the IPC-2141A formula:

$$\frac{\partial Z_0}{\partial w} = -Z_0 \cdot \frac{0.8}{(0.8w + t)} \cdot \frac{1}{\ln(\ldots)}$$

At the nominal point: $\ln(\ldots) = 1.373$ (from Step 2 of part a), $0.8w + t = 6.059$ mil:

$$\frac{\partial Z_0}{\partial w}\Bigg|_{MS} = -50 \times \frac{0.8}{6.059} \times \frac{1}{1.373} = -50 \times 0.1321 \times 0.7283 = -4.81\ \Omega/\text{mil}$$

For $\Delta w = \pm 0.5$ mil: $\Delta Z_0 = -4.81 \times (\pm 0.5) = \mp 2.4\ \Omega$

**Microstrip worst-case range:**

$$50 - 2.4 = 47.6\ \Omega \quad \text{(wide trace)} \quad \text{to} \quad 50 + 2.4 = 52.4\ \Omega \quad \text{(narrow trace)}$$

$$\boxed{Z_0^{MS} = 50\ \Omega \pm 2.4\ \Omega\ (^{+5\%}_{-5\%})} \quad \text{from etch tolerance alone}$$

**Stripline sensitivity:**

At the nominal point for stripline: $\ln(\ldots) = 1.728$, $0.8w + t = 3.376$ mil:

$$\frac{\partial Z_0}{\partial w}\Bigg|_{SL} = -50 \times \frac{0.8}{3.376} \times \frac{1}{1.728} = -50 \times 0.2370 \times 0.5787 = -6.85\ \Omega/\text{mil}$$

For $\Delta w = \pm 0.5$ mil: $\Delta Z_0 = \mp 3.4\ \Omega$

**Stripline worst-case range:**

$$\boxed{Z_0^{SL} = 50\ \Omega \pm 3.4\ \Omega\ (^{+7\%}_{-7\%})} \quad \text{from etch tolerance alone}$$

**Key observation:** The stripline is more sensitive to etch variation than the microstrip (6.85 vs. 4.81 $\Omega$/mil), because the narrower trace geometry ($w = 3.3$ mil) means the same 0.5 mil width change is a larger *fractional* change. A 0.5 mil change on a 3.3 mil trace is 15% variation, whereas on a 6.7 mil trace it is only 7.5%.

**Combined tolerance budget:** The etch tolerance combines with dielectric height variation (typically $\pm 5$%) and Dk variation (typically $\pm 3$%). Adding these in root-sum-square:

$$\sigma_{total} \approx \sqrt{(3.4)^2 + (2.5)^2 + (1.5)^2} \approx 4.6\ \Omega$$

Total $\pm 3\sigma$ stripline range: $50 \pm 14\ \Omega$ — well outside $\pm 10$% without tight process control. This illustrates why controlled-impedance PCBs require the manufacturer to measure and adjust the process to hit the specification.

---

### Part (d): 100 $\Omega$ Differential Pair on Inner Layer

**Setting up the single-ended impedance target:**

The relationship between differential impedance and odd-mode (single-ended) impedance for a coupled pair is:

$$Z_{diff} = 2 Z_{odd}$$

For a loosely coupled pair: $Z_{odd} \approx Z_0$ (single-ended). But the problem states that tight coupling (5 mil edge-to-edge gap) reduces $Z_{diff}$ by 8% relative to $2 Z_0$.

**Target:** $Z_{diff} = 100\ \Omega$ with 8% coupling reduction.

$$2 Z_0 \times (1 - 0.08) = 100\ \Omega$$

$$2 Z_0 \times 0.92 = 100\ \Omega$$

$$Z_0 = \frac{100}{2 \times 0.92} = \frac{100}{1.84} = 54.3\ \Omega$$

Each single-ended trace must target $Z_0 = 54.3\ \Omega$ so that the coupling brings the differential impedance down to 100 $\Omega$.

**Finding the trace width for 54.3 $\Omega$ stripline:**

Using the stripline formula with $b = 10$ mil, $t = 0.7$ mil, $\varepsilon_r = 4.3$:

$$\ln\!\left(\frac{40}{0.67\pi(0.8w + 0.7)}\right) = \frac{54.3}{28.93} = 1.877$$

$$\frac{40}{0.67\pi(0.8w + 0.7)} = e^{1.877} = 6.530$$

$$0.67\pi(0.8w + 0.7) = \frac{40}{6.530} = 6.127$$

$$0.8w + 0.7 = \frac{6.127}{2.104} = 2.912$$

$$w = \frac{2.212}{0.8} = 2.8\ \text{mil}$$

**Summary for differential pair:**

| Parameter | Value |
|---|---|
| Each trace width | 2.8 mil |
| Edge-to-edge spacing | 5 mil |
| Single-ended $Z_0$ target | 54.3 $\Omega$ |
| Coupling reduction factor | 8% |
| Resulting $Z_{diff}$ | $2 \times 54.3 \times 0.92 = 100\ \Omega$ |

**Design notes:**

1. **Verification:** These closed-form estimates provide the starting point. Before submitting to the PCB fabricator, a 2D field solver (e.g., Polar Si9000, Ansys SIwave cross-section tool) should verify both the single-ended and differential impedances simultaneously, since the closed-form coupling correction (8%) is geometry-specific and may not be accurate.

2. **Routing rule:** Maintain constant edge-to-edge spacing throughout the route. Any spacing variation changes the coupling coefficient, causing local $Z_{diff}$ variations that produce a reflected wave. Serpentine sections that increase the spacing reduce coupling and raise $Z_{diff}$ locally.

3. **Common-mode impedance:** The even-mode (common-mode) impedance of this pair is $Z_{even} = Z_0(1 + k)$ where $k$ is the coupling coefficient. Higher coupling (tighter spacing) increases even-mode impedance, improving common-mode noise rejection. This is a desirable property for differential signals carrying common-mode noise from switching supplies or connectors.

---

## Summary of Results

| Geometry | Trace width | Nominal $Z_0$ | Etch-induced range |
|---|---|---|---|
| Microstrip (outer layer) | 6.7 mil | 50 $\Omega$ | 47.6–52.4 $\Omega$ |
| Stripline (inner layer) | 3.3 mil | 50 $\Omega$ | 46.6–53.4 $\Omega$ |
| Diff pair each trace (inner) | 2.8 mil | 54.3 $\Omega$ SE | $Z_{diff}$ = 100 $\Omega$ |

**Key interview takeaways:**

- Stripline is narrower than microstrip for the same $Z_0$ because the full-dielectric environment has higher effective Dk and more capacitance per unit length.
- Stripline is more sensitive to etch variation (per mil) than microstrip because narrower traces are a larger fractional geometry change.
- Differential pair trace widths must be designed to a higher single-ended target to allow for coupling reduction.
- Closed-form formulas provide the starting estimate; field-solver verification is mandatory before production.
