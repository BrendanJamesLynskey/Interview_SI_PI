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

$$\Gamma_L = \frac{Z_L - Z_0}{Z_L + Z_0} = \frac{4000 - 50}{4000 + 50} = \frac{3950}{4050} = +0.975 \approx +1.0$$

For all practical purposes, the open-circuit load reflects the wave with full positive coefficient. This is the expected result for a high-impedance CMOS input.

**At the source:**

$$\Gamma_S = \frac{Z_S - Z_0}{Z_S + Z_0} = \frac{8 - 50}{8 + 50} = \frac{-42}{58} = -0.724$$

The source has a negative reflection coefficient because $Z_S < Z_0$ — the driver is "harder" than the line. Reflected waves arriving back at the source are partially absorbed and partially re-reflected with inverted polarity.

---

### Part (b): Initial Launched Wave

The driver (Thevenin equivalent: $V_{DD} = 3.3$ V, series resistance $Z_S = 8\ \Omega$) drives the line. The initial wave amplitude is given by the voltage divider between $Z_S$ and $Z_0$:

$$V_1 = V_{DD} \times \frac{Z_0}{Z_S + Z_0} = 3.3 \times \frac{50}{8 + 50} = 3.3 \times \frac{50}{58} = 3.3 \times 0.862 = \mathbf{2.845\ \text{V}}$$

The source node voltage immediately after launch is given by the voltage divider: $V_{source}(0^+) = V_{DD} \times Z_0/(Z_S + Z_0) = 2.845$ V. The source node instantly rises to 2.845 V when the wave is launched.

---

### Part (c): Lattice Diagram

The one-way delay is $TD = 2$ ns. Events occur at multiples of $TD$.

```
SOURCE (z=0)                                    LOAD (z=L)
    Γ_S = −0.724                                Γ_L = +0.975

t = 0 ns:
    [V_source = 2.845 V]
    V1 = +2.845 V  ─────────────────────────────>

t = 2 ns:
    V1 arrives at load. V_load = V1(1 + Γ_L) = 2.845 × 1.975 = 5.619 V
    Reflected wave: V2 = Γ_L × V1 = 0.975 × 2.845 = +2.775 V
                   <─────────────────────────────

t = 4 ns:
    V2 arrives at source. V_source += V2(1 + Γ_S) = 2.775 × (1 − 0.724) = 0.765 V
    V_source total = 2.845 + 0.765 = 3.610 V
    Re-reflected wave: V3 = Γ_S × V2 = −0.724 × 2.775 = −2.009 V
    V3 ──────────────────────────────────────────>

t = 6 ns:
    V3 arrives at load. V_load += V3(1 + Γ_L) = −2.009 × 1.975 = −3.969 V
    V_load total = 5.619 − 3.969 = 1.651 V
    Reflected wave: V4 = Γ_L × V3 = 0.975 × (−2.009) = −1.960 V
                   <─────────────────────────────

t = 8 ns:
    V4 arrives at source. V_source += V4(1 + Γ_S) = −1.960 × 0.276 = −0.541 V
    V_source total = 3.610 − 0.541 = 3.070 V
    Re-reflected wave: V5 = Γ_S × V4 = −0.724 × (−1.960) = +1.419 V
    V5 ──────────────────────────────────────────>

t = 10 ns:
    V5 arrives at load. V_load += V5(1 + Γ_L) = 1.419 × 1.975 = 2.803 V
    V_load total = 1.651 + 2.803 = 4.454 V
    (continues oscillating, converging toward 3.293 V DC = 3.3 × 4000/4008)
```

---

### Part (d): Voltage at the Load vs. Time

Reading from the lattice diagram above, and tracking all contributions:

| Time | Event | $V_{load}$ |
|---|---|---|
| $0 \leq t < 2$ ns | No wave has arrived | **0 V** |
| $t = 2$ ns | $V_1$ arrives at load | **+5.62 V** (overshoot of 2.32 V = 70% above $V_{DD}$!) |
| $2 < t < 6$ ns | Load holds | **5.62 V** |
| $t = 6$ ns | $V_3$ arrives at load | **+1.65 V** (undershoot of 1.65 V below $V_{DD}$) |
| $6 < t < 10$ ns | Load holds | **1.65 V** |
| $t = 10$ ns | $V_5$ arrives at load | **+4.45 V** |

**Does the waveform ring above or below supply?**

Yes — severely. At $t = 2$ ns, the load voltage overshoots to **5.62 V**, which is 70% above $V_{DD} = 3.3$ V. For a 3.3 V CMOS input with absolute maximum ratings of 3.6–4.5 V (depending on process), this level is likely to cause latch-up or permanent gate oxide damage. This is the classic consequence of an unterminated low-impedance CMOS driver: the nearly-matched source launches a near-full-swing wave that reflects with $\Gamma_L \approx +1$ and doubles at the open-circuit load.

The waveform alternates: overshoot at $t = 2$ ns, undershoot at $t = 6$ ns, overshoot at $t = 10$ ns, etc. The oscillation amplitude decays as $|\Gamma_S \Gamma_L|^n$ where $n$ is the round-trip number. With $|\Gamma_S \Gamma_L| = 0.724 \times 0.975 = 0.706$, the oscillation damps over approximately $1/(1-0.706) \approx 3.4$ round trips, or about 14 ns. The DC steady-state value is:

$$V_{dc} = V_{DD} \times \frac{Z_L}{Z_S + Z_L} = 3.3 \times \frac{4000}{4008} \approx 3.293\ \text{V}$$

---

### Part (e): Series Termination ($R_S = 42\ \Omega$, total source impedance = 50 $\Omega$)

**New source impedance:** $Z_S' = 8 + 42 = 50\ \Omega$

**New reflection coefficients:**

$$\Gamma_S' = \frac{50 - 50}{50 + 50} = 0$$

$$\Gamma_L = +0.975 \approx +1.0 \quad \text{(unchanged)}$$

**Initial launched wave:**

$$V_1' = 3.3 \times \frac{50}{50 + 50} = 3.3 \times 0.5 = \mathbf{1.65\ \text{V}}$$

**Lattice diagram (series terminated):**

```
t = 0 ns:   V1' = +1.65 V  ────────────────────────>
t = 2 ns:   V1' arrives at load.
            V_load = 1.65 × (1 + 0.975) = 1.65 × 1.975 = 3.259 V ≈ 3.3 V
            V2' = Γ_L × 1.65 = 0.975 × 1.65 = +1.609 V
                               <────────────────────────
t = 4 ns:   V2' arrives at source.
            Γ_S' = 0. Wave is fully absorbed. No re-reflection.
            V_source += V2'(1 + Γ_S') = 1.609 × 1 = 1.609 V
            V_source total = 1.65 + 1.609 = 3.26 V ≈ 3.3 V
```

**Load voltage vs. time (series terminated):**

| Time | Event | $V_{load}$ |
|---|---|---|
| $0 \leq t < 2$ ns | No wave has arrived | **0 V** |
| $t = 2$ ns | $V_1'$ arrives; reflects with $\Gamma_L \approx +1$ | **3.26 V** |
| $t > 2$ ns | No further reflections ($\Gamma_S' = 0$) | **3.26 V** (settled) |

**Comparison:**

| Property | Unterminated | Series terminated |
|---|---|---|
| Initial load overshoot | 5.62 V (+70%) | 3.26 V (no overshoot) |
| Load voltage settling time | ~14 ns (multiple ring cycles) | ~2 ns (single step) |
| Source launch voltage | 2.845 V (for 4 ns) | 1.65 V (mid-voltage for 4 ns) |
| Risk to CMOS input | High (likely damage) | None |
| Series resistor dissipation | N/A | $P = \frac{(V_{DD}/2)^2}{R_S} \times \text{duty} \approx 0$ (no DC path) |

**Intermediate node note:** With series termination, every point on the line (not just the far end) sees 1.65 V from $t = 0$ until the reflected wave returns from the far end. A midpoint probe at $l/2$ would show 1.65 V from $t = 1$ ns until $t = 3$ ns, then 3.26 V after that. If an intermediate load is present at the midpoint, it sees only half-voltage for the 2 ns round-trip window. This is the primary limitation of series termination for multi-drop buses.

---

## Key Takeaways

1. **An unterminated low-impedance driver into a high-impedance CMOS load produces dangerous voltage overshoot.** In this example, the load overshoot was 5.62 V — 70% above $V_{DD}$. This is a real failure mode that destroys ICs.

2. **Series termination ($\Gamma_S = 0$) eliminates all re-reflections.** After one round trip, the waveform is settled and clean. The only cost is a 2 ns window where intermediate nodes see half-voltage.

3. **The lattice diagram is the standard tool for predicting reflections.** It is expected in an interview that you can set up and step through the diagram for a simple three-event sequence.

4. **Reflection coefficient magnitude product $|\Gamma_S \Gamma_L|$ determines the damping rate.** In this example, $0.724 \times 0.975 = 0.706$, meaning each round trip retains 70.6% of the previous amplitude. Convergence is slow without termination.

5. **Rule of thumb for CMOS loads:** With $Z_S \ll Z_0$ and $Z_L \gg Z_0$, the first overshoot at the load is approximately $2 V_{DD} Z_0/(Z_S + Z_0) \approx V_{DD} \times (2 - 2Z_S/Z_0) \to 2 V_{DD}$ for $Z_S \to 0$. Always check whether this exceeds the maximum input voltage rating.
