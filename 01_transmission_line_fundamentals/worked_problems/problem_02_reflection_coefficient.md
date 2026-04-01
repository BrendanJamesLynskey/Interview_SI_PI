# Problem 02: Reflection Coefficient and Lattice Diagram

## Problem Statement

A CMOS driver with output impedance $Z_S = 8\ \Omega$ drives a 50 $\Omega$ transmission line ($Z_0 = 50\ \Omega$) that is 2 ns long (one-way propagation delay $TD = 2$ ns). The load is a CMOS receiver input with $Z_L = 4\ \text{k}\Omega$ (effectively open circuit for this analysis).

The driver switches from 0 V to $V_{DD} = 3.3$ V at $t = 0$.

**No series or parallel termination is present.**

**Tasks:**

**(a)** Calculate the reflection coefficient at the source ($\Gamma_S$) and at the load ($\Gamma_L$).

**(b)** Calculate the amplitude of the initial wave launched by the driver.

**(c)** Construct a lattice diagram showing at least four reflection events and the voltage at each node (source and load) after each event.

**(d)** What is the voltage at the load at $t = 1$ ns, $t = 2$ ns, $t = 3$ ns, $t = 5$ ns, and $t = 8$ ns? Does the waveform ring above or below the supply voltage?

**(e)** A 42 $\Omega$ series resistor is added in series at the driver output ($Z_S + R_S = 50\ \Omega$). Repeat parts (b)–(d) for the series-terminated case and explain the difference.

---

## Solution

### Part (a): Reflection Coefficients

**At the load:**

$$\Gamma_L = \frac{Z_L - Z_0}{Z_L + Z_0} = \frac{4000 - 50}{4000 + 50} = \frac{3950}{4050} = +0.988 \approx +1.0$$

For all practical purposes, the open-circuit load reflects the wave with full positive coefficient. This is the expected result for a high-impedance CMOS input.

**At the source:**

$$\Gamma_S = \frac{Z_S - Z_0}{Z_S + Z_0} = \frac{8 - 50}{8 + 50} = \frac{-42}{58} = -0.724$$

The source has a negative reflection coefficient because $Z_S < Z_0$ — the driver is "harder" than the line. Reflected waves arriving back at the source are partially absorbed and partially re-reflected with inverted polarity.

---

### Part (b): Initial Launched Wave

The driver (Thevenin equivalent: $V_{DD} = 3.3$ V, series resistance $Z_S = 8\ \Omega$) drives the line. The initial wave amplitude is given by the voltage divider between $Z_S$ and $Z_0$:

$$V_1 = V_{DD} \times \frac{Z_0}{Z_S + Z_0} = 3.3 \times \frac{50}{8 + 50} = 3.3 \times \frac{50}{58} = 3.3 \times 0.862 = \mathbf{2.845\ \text{V}}$$

The voltage at the source node immediately after launch:

$$V_{source}(0^+) = V_{DD} - V_1 \times \frac{Z_S}{Z_0} = V_1 + V_1 \times 0 = V_1$$

Wait — use the simpler form: $V_{source} = V_{DD} \times Z_0/(Z_S + Z_0)$. This gives $V_{source}(0^+) = 2.845$ V. The source node instantly rises to 2.845 V (the voltage divider) when the wave is launched.

---

### Part (c): Lattice Diagram

The one-way delay is $TD = 2$ ns. Events occur at multiples of $TD$.

```
SOURCE (z=0)                                    LOAD (z=L)
    Γ_S = −0.724                                Γ_L = +0.988

t = 0 ns:
    [V_source = 2.845 V]
    V1 = +2.845 V  ─────────────────────────────>

t = 2 ns:
    V1 arrives at load. V_load = V1(1 + Γ_L) = 2.845 × 1.988 = 5.656 V
    Reflected wave: V2 = Γ_L × V1 = 0.988 × 2.845 = +2.811 V
                   <─────────────────────────────

t = 4 ns:
    V2 arrives at source. V_source += V2(1 + Γ_S) = 2.811 × (1 − 0.724) = 0.776 V
    V_source total = 2.845 + 0.776 = 3.621 V
    Re-reflected wave: V3 = Γ_S × V2 = −0.724 × 2.811 = −2.035 V
    V3 ──────────────────────────────────────────>

t = 6 ns:
    V3 arrives at load. V_load += V3(1 + Γ_L) = −2.035 × 1.988 = −4.046 V
    V_load total = 5.656 − 4.046 = 1.610 V
    Reflected wave: V4 = Γ_L × V3 = 0.988 × (−2.035) = −2.011 V
                   <─────────────────────────────

t = 8 ns:
    V4 arrives at source. V_source += V4(1 + Γ_S) = −2.011 × 0.276 = −0.555 V
    V_source total = 3.621 − 0.555 = 3.066 V
    Re-reflected wave: V5 = Γ_S × V4 = −0.724 × (−2.011) = +1.456 V
    V5 ──────────────────────────────────────────>

t = 10 ns:
    V5 arrives at load. V_load += V5(1 + Γ_L) = 1.456 × 1.988 = 2.894 V
    V_load total = 1.610 + 2.894 = 4.504 V
    (continues oscillating, converging toward 3.285 V DC = 3.3 × 4000/4058)
```

---

### Part (d): Voltage at the Load vs. Time

Reading from the lattice diagram above, and tracking all contributions:

| Time | Event | $V_{load}$ |
|---|---|---|
| $0 \leq t < 2$ ns | No wave has arrived | **0 V** |
| $t = 2$ ns | $V_1$ arrives at load | **+5.66 V** (overshoot of 2.36 V = 71% above $V_{DD}$!) |
| $2 < t < 6$ ns | Load holds | **5.66 V** |
| $t = 6$ ns | $V_3$ arrives at load | **+1.61 V** (undershoot of 1.69 V below $V_{DD}$) |
| $6 < t < 10$ ns | Load holds | **1.61 V** |
| $t = 10$ ns | $V_5$ arrives at load | **+4.50 V** |

**Does the waveform ring above or below supply?**

Yes — severely. At $t = 2$ ns, the load voltage overshoots to **5.66 V**, which is 71% above $V_{DD} = 3.3$ V. For a 3.3 V CMOS input with absolute maximum ratings of 3.6–4.5 V (depending on process), this level is likely to cause latch-up or permanent gate oxide damage. This is the classic consequence of an unterminated low-impedance CMOS driver: the nearly-matched source launches a near-full-swing wave that reflects with $\Gamma_L \approx +1$ and doubles at the open-circuit load.

The waveform alternates: overshoot at $t = 2$ ns, undershoot at $t = 6$ ns, overshoot at $t = 10$ ns, etc. The oscillation amplitude decays as $|\Gamma_S \Gamma_L|^n$ where $n$ is the round-trip number. With $|\Gamma_S \Gamma_L| = 0.724 \times 0.988 = 0.715$, the oscillation damps over approximately $1/(1-0.715) \approx 3.5$ round trips, or about 14 ns. The DC steady-state value is:

$$V_{dc} = V_{DD} \times \frac{Z_L}{Z_S + Z_L} = 3.3 \times \frac{4000}{4008} \approx 3.297\ \text{V}$$

---

### Part (e): Series Termination ($R_S = 42\ \Omega$, total source impedance = 50 $\Omega$)

**New source impedance:** $Z_S' = 8 + 42 = 50\ \Omega$

**New reflection coefficients:**

$$\Gamma_S' = \frac{50 - 50}{50 + 50} = 0$$

$$\Gamma_L = +0.988 \approx +1.0 \quad \text{(unchanged)}$$

**Initial launched wave:**

$$V_1' = 3.3 \times \frac{50}{50 + 50} = 3.3 \times 0.5 = \mathbf{1.65\ \text{V}}$$

**Lattice diagram (series terminated):**

```
t = 0 ns:   V1' = +1.65 V  ────────────────────────>
t = 2 ns:   V1' arrives at load.
            V_load = 1.65 × (1 + 0.988) = 1.65 × 1.988 = 3.280 V ≈ 3.3 V
            V2' = Γ_L × 1.65 = 0.988 × 1.65 = +1.630 V
                               <────────────────────────
t = 4 ns:   V2' arrives at source.
            Γ_S' = 0. Wave is fully absorbed. No re-reflection.
            V_source += V2'(1 + Γ_S') = 1.630 × 1 = 1.630 V
            V_source total = 1.65 + 1.630 = 3.28 V ≈ 3.3 V
```

**Load voltage vs. time (series terminated):**

| Time | Event | $V_{load}$ |
|---|---|---|
| $0 \leq t < 2$ ns | No wave has arrived | **0 V** |
| $t = 2$ ns | $V_1'$ arrives; reflects with $\Gamma_L \approx +1$ | **3.28 V** |
| $t > 2$ ns | No further reflections ($\Gamma_S' = 0$) | **3.28 V** (settled) |

**Comparison:**

| Property | Unterminated | Series terminated |
|---|---|---|
| Initial load overshoot | 5.66 V (+71%) | 3.28 V (no overshoot) |
| Load voltage settling time | ~14 ns (multiple ring cycles) | ~2 ns (single step) |
| Source launch voltage | 2.845 V (mid-voltage for 2 ns) | 1.65 V (mid-voltage for 4 ns) |
| Risk to CMOS input | High (likely damage) | None |
| Series resistor dissipation | N/A | $P = \frac{(V_{DD}/2)^2}{R_S} \times \text{duty} \approx 0$ (no DC path) |

**Intermediate node note:** With series termination, every point on the line (not just the far end) sees 1.65 V from $t = 0$ until the reflected wave returns from the far end. A midpoint probe at $l/2$ would show 1.65 V from $t = 1$ ns until $t = 3$ ns, then 3.28 V after that. If an intermediate load is present at the midpoint, it sees only half-voltage for the 2 ns round-trip window. This is the primary limitation of series termination for multi-drop buses.

---

## Key Takeaways

1. **An unterminated low-impedance driver into a high-impedance CMOS load produces dangerous voltage overshoot.** In this example, the load overshoot was 5.66 V — 71% above $V_{DD}$. This is a real failure mode that destroys ICs.

2. **Series termination ($\Gamma_S = 0$) eliminates all re-reflections.** After one round trip, the waveform is settled and clean. The only cost is a 2 ns window where intermediate nodes see half-voltage.

3. **The lattice diagram is the standard tool for predicting reflections.** It is expected in an interview that you can set up and step through the diagram for a simple three-event sequence.

4. **Reflection coefficient magnitude product $|\Gamma_S \Gamma_L|$ determines the damping rate.** In this example, $0.724 \times 0.988 = 0.715$, meaning each round trip retains 71.5% of the previous amplitude. Convergence is slow without termination.

5. **Rule of thumb for CMOS loads:** With $Z_S \ll Z_0$ and $Z_L \gg Z_0$, the first overshoot at the load is approximately $V_{DD} \times (2 - Z_S/Z_0) \approx 2 V_{DD}$ for $Z_S \to 0$. Always check whether this exceeds the maximum input voltage rating.
