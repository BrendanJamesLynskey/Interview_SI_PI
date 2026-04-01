# Problem 01: Target Impedance Calculation

## Problem Statement

You are designing the PDN for the core power rail of a mid-range FPGA placed on a PCIe add-in card. The FPGA datasheet specifies the following for the core supply rail:

- Nominal voltage: $V_{nom} = 0.95\ V$
- Maximum operating current: $I_{max} = 12\ A$
- Voltage tolerance: $\pm 3\%$
- Maximum current slew rate: $dI/dt = 2\ A/ns$

The board-level VRM is a 4-phase buck converter switching at 600 kHz per phase, with an achievable loop bandwidth of approximately 150 kHz. The VRM output filter uses per-phase inductors of 470 nH.

You have available the following passive components:

| Part | Type | Value | $R_{ESR}$ | $L_{ESL}$ | Case |
|---|---|---|---|---|---|
| A | Polymer electrolytic | 470 µF | 8 m$\Omega$ | 4 nH | SMD D-case |
| B | X5R MLCC | 22 µF | 3 m$\Omega$ | 0.9 nH | 0805 |
| C | X7R MLCC | 4.7 µF | 8 m$\Omega$ | 0.6 nH | 0402 |
| D | X7R MLCC | 100 nF | 35 m$\Omega$ | 0.5 nH | 0402 |

**Tasks:**

**(a)** Calculate the target impedance for this power rail, justifying your voltage budget allocation.

**(b)** Determine the frequency range over which the PDN must meet this target impedance.

**(c)** Calculate the minimum required bulk capacitance and minimum required number of bulk capacitors (Part A).

**(d)** Determine how many Part B MLCCs are needed to meet the target impedance at their SRF.

**(e)** Check the anti-resonance peak between the bulk capacitor stage (Part A) and the mid-frequency MLCC stage (Part B). Does it violate the target impedance? Propose a mitigation if needed.

**(f)** Summarise the full PDN design, listing the component count per stage, SRF coverage, and impedance at SRF for each stage.

---

## Solution

### Part (a): Target Impedance

**Voltage budget allocation:**

Total tolerance: $\pm 3\%$ of 0.95 V = $\pm 28.5\ mV$.

Allocate this across PDN contributors (worst-case sum approach):

| Contributor | Allocation |
|---|---|
| VRM DC accuracy | 8 mV |
| PCB IR drop ($I_{max} \times R_{trace}$ at 2 m$\Omega$ estimated trace) | 24 mV — too high with 12 A |

Re-evaluate: at 12 A, even 2 m$\Omega$ of plane resistance contributes $12 \times 0.002 = 24\ mV$, consuming most of the budget. For a PCIe card with wide planes and short trace to the FPGA, the DC resistance should be $< 0.5\ m\Omega$, giving:

| Contributor | Allocation |
|---|---|
| VRM DC accuracy ($\pm$1%) | 9.5 mV |
| PCB IR drop (0.5 m$\Omega \times 12\ A$) | 6 mV |
| Thermal drift | 3 mV |
| **AC PDN ripple** | **10 mV** |
| Total (sum) | 28.5 mV ✓ |

**Target impedance:**

$$\boxed{Z_{target} = \frac{\Delta V_{AC}}{I_{max}} = \frac{10\ mV}{12\ A} = 0.83\ m\Omega}$$

This is a very tight target, consistent with a high-current FPGA core rail. Note that using RSS allocation instead:

$$\Delta V_{AC} \approx \sqrt{28.5^2 - 9.5^2 - 6^2 - 3^2} = \sqrt{812 - 90 - 36 - 9} = \sqrt{677} \approx 26\ mV$$

$$Z_{target,RSS} = \frac{26\ mV}{12\ A} \approx 2.2\ m\Omega$$

In practice, a PDN designer would use the RSS-derived value of approximately **2 m$\Omega$** to keep the design tractable. We will use $Z_{target} = 2\ m\Omega$ for the remainder of this problem.

---

### Part (b): Frequency Range

**Lower bound — VRM loop bandwidth:**

$$f_{low} = f_{BW} = 150\ kHz$$

**Upper bound — current slew rate:**

The fastest current transient has a rise time determined by the slew rate and current step:

$$t_{rise} = \frac{\Delta I}{dI/dt} = \frac{12\ A}{2\ A/ns} = 6\ ns$$

Knee frequency:

$$f_{high} = \frac{0.35}{t_{rise}} = \frac{0.35}{6\times10^{-9}} \approx 58\ MHz$$

**Required coverage range:**

$$\boxed{f_{low} = 150\ kHz \text{ to } f_{high} = 58\ MHz}$$

Note: for safety margin, extend to 100 MHz in the design.

---

### Part (c): Minimum Bulk Capacitance and Capacitor Count

**Minimum bulk capacitance to cover from $f_{low}$ with impedance $\leq Z_{target}$:**

$$C_{bulk,min} = \frac{1}{2\pi f_{low} \cdot Z_{target}} = \frac{1}{2\pi \times 150\times10^3 \times 2\times10^{-3}}$$

$$C_{bulk,min} = \frac{1}{2\pi \times 300} = \frac{1}{1885} \approx 530\ \mu F$$

**Alternative check using transient energy argument:**

The VRM responds in $\tau = 1/f_{BW} = 1/150\ kHz = 6.67\ \mu s$. The charge demanded:

$$Q = I_{max} \times \tau = 12 \times 6.67\times10^{-6} = 80\ \mu C$$

Bulk capacitance to limit droop to $\Delta V_{AC} = 26\ mV$:

$$C_{bulk} \geq \frac{Q}{\Delta V_{AC}} = \frac{80\times10^{-6}}{26\times10^{-3}} \approx 3.1\ mF$$

The energy method gives a much larger requirement — this is the dominant constraint because of the high current and tight tolerance. Use $C_{bulk,min} = 3.1\ mF$.

**Number of Part A capacitors:**

Part A: 470 µF polymer, $R_{ESR} = 8\ m\Omega$.

Capacitance constraint:

$$N_A \geq \frac{3.1\ mF}{470\ \mu F} = 6.6 \rightarrow N_A = 7$$

ESR constraint at SRF:

$$R_{ESR,total} = \frac{8\ m\Omega}{7} = 1.14\ m\Omega < Z_{target} = 2\ m\Omega \quad \checkmark$$

$$\boxed{N_A = 7\ \text{Part A polymer capacitors in parallel}}$$

Total bulk capacitance: $7 \times 470\ \mu F = 3.29\ mF$, satisfying the 3.1 mF requirement.

SRF of Part A:

$$f_{SRF,A} = \frac{1}{2\pi\sqrt{L_{ESL,A} \times C_A}} = \frac{1}{2\pi\sqrt{4\times10^{-9} \times 470\times10^{-6}}} = \frac{1}{2\pi\sqrt{1.88\times10^{-12}}}$$

$$= \frac{1}{2\pi \times 1.37\times10^{-6}} \approx 116\ kHz$$

The bulk stage has SRF at 116 kHz — just below the VRM loop bandwidth. This is expected: the bulk capacitors are primarily energy storage, with their SRF below the VRM crossover. Above SRF they are inductive (the effective ESL of the parallel bank is $4/7 \approx 0.57\ nH$).

---

### Part (d): Mid-Frequency MLCC Count (Part B)

Part B: 22 µF 0805 MLCC, $R_{ESR} = 3\ m\Omega$, $L_{ESL} = 0.9\ nH$.

**SRF of single Part B:**

$$f_{SRF,B} = \frac{1}{2\pi\sqrt{0.9\times10^{-9} \times 22\times10^{-6}}} = \frac{1}{2\pi\sqrt{1.98\times10^{-14}}} = \frac{1}{2\pi \times 140.7\times10^{-9}} \approx 1.13\ MHz$$

However, note the X5R voltage coefficient derating. At 0.95 V on a capacitor rated for 6.3 V (typical for 0805 22 µF X5R), the bias is 15% of rating. At this bias level, capacitance drops to approximately 85% of nominal: $C_{eff} \approx 18.7\ \mu F$.

$$f_{SRF,B,derated} = \frac{1}{2\pi\sqrt{0.9\times10^{-9} \times 18.7\times10^{-6}}} \approx 1.22\ MHz$$

**Capacitance needed at $f_{SRF,B}$ to meet $Z_{target}$:**

$$C_{MLCC,min} = \frac{1}{2\pi f_{SRF,B} \cdot Z_{target}} = \frac{1}{2\pi \times 1.13\times10^6 \times 2\times10^{-3}} = \frac{1}{14200} \approx 70\ \mu F$$

Number of Part B required from capacitance:

$$N_B \geq \frac{70\ \mu F}{18.7\ \mu F} = 3.7 \rightarrow N_B = 4$$

**ESR constraint:**

$$R_{ESR,B,total} = \frac{3\ m\Omega}{4} = 0.75\ m\Omega < 2\ m\Omega \quad \checkmark$$

$$\boxed{N_B = 4\ \text{Part B 22 µF 0805 MLCCs}}$$

Total MLCC capacitance (derated): $4 \times 18.7\ \mu F = 74.8\ \mu F$.

SRF of parallel bank (unchanged): $f_{SRF,B,parallel} \approx 1.22\ MHz$.

---

### Part (e): Anti-Resonance Check — Bulk vs. MLCC Stage

The parallel combination of Part A (bulk, above its SRF → inductive) and Part B (MLCC, below its SRF → capacitive) can create an anti-resonance peak.

**Effective ESL of Part A parallel bank:**

$$L_{A,eff} = \frac{L_{ESL,A}}{N_A} = \frac{4\ nH}{7} = 0.571\ nH$$

**Total effective capacitance of Part B bank (derated):**

$$C_{B,total} = 4 \times 18.7\ \mu F = 74.8\ \mu F$$

**Anti-resonance frequency:**

$$f_{AR} = \frac{1}{2\pi\sqrt{L_{A,eff} \cdot C_{B,total}}} = \frac{1}{2\pi\sqrt{0.571\times10^{-9} \times 74.8\times10^{-6}}}$$

$$= \frac{1}{2\pi\sqrt{4.27\times10^{-14}}} = \frac{1}{2\pi \times 206.7\times10^{-9}} \approx 770\ kHz$$

**Anti-resonance peak impedance (lossless approximation):**

$$|Z_{AR}| \approx \sqrt{\frac{L_{A,eff}}{C_{B,total}}} = \sqrt{\frac{0.571\times10^{-9}}{74.8\times10^{-6}}} = \sqrt{7.63\times10^{-6}} \approx 2.76\ m\Omega$$

**Assessment:**

$|Z_{AR}| \approx 2.76\ m\Omega > Z_{target} = 2\ m\Omega$ — this peak **violates the target impedance**.

**Mitigation — damping resistor:**

Add a series resistor $R_d$ to one of the bulk capacitors to add damping. The optimal damping value:

$$R_d \approx |Z_{AR}| - R_{ESR,A,total} = 2.76\ m\Omega - \frac{8\ m\Omega}{7} = 2.76 - 1.14 = 1.62\ m\Omega$$

In practice, replace one Part A capacitor (say, capacitor A7) with a series combination of Part A + $2\ m\Omega$ surface-mount resistor. The damped peak impedance:

$$|Z_{AR,damped}| \approx \frac{R_{ESR,A,total} + R_d}{2} \approx \frac{1.14 + 1.62}{2} \approx 1.38\ m\Omega < 2\ m\Omega \quad \checkmark$$

**Alternative:** Add one more Part B capacitor ($N_B = 5$):

$$|Z_{AR,5B}| = \sqrt{\frac{0.571\times10^{-9}}{5 \times 18.7\times10^{-6}}} = \sqrt{\frac{0.571\times10^{-9}}{93.5\times10^{-6}}} = \sqrt{6.11\times10^{-6}} \approx 2.47\ m\Omega$$

Still above target — the damping resistor approach is more effective here.

---

### Part (f): Full PDN Design Summary

| Stage | Part | Count | $f_{SRF}$ | $R_{ESR,total}$ | $C_{total}$ | Coverage |
|---|---|---|---|---|---|---|
| Bulk | A (470 µF polymer) | 7 | 116 kHz | 1.14 m$\Omega$ | 3.29 mF | DC to ~150 kHz |
| Mid-freq | B (22 µF 0805 MLCC) | 4 | 1.22 MHz | 0.75 m$\Omega$ | 75 µF | 500 kHz to 5 MHz |
| High-freq | C (4.7 µF 0402 MLCC) | TBD | ~3.1 MHz | — | — | 2–20 MHz |
| VHF | D (100 nF 0402 MLCC) | TBD | ~22.5 MHz | — | — | 10–100 MHz |

**For Part C (4.7 µF 0402, $f_{SRF} \approx 3.1\ MHz$):**

Number needed (ESR criterion): $N_C = R_{ESR,C}/Z_{target} = 8/2 = 4$.

**For Part D (100 nF 0402, $f_{SRF} \approx 22.5\ MHz$):**

Number needed (ESR criterion): $N_D = R_{ESR,D}/Z_{target} = 35/2 = 17.5 \rightarrow 18$.

**Final PDN Bill of Materials:**

| Stage | Part | Count | Notes |
|---|---|---|---|
| Bulk | 470 µF polymer (A) | 7 | One with 2 m$\Omega$ series damping resistor |
| Mid-freq | 22 µF 0805 MLCC (B) | 4 | X5R, 6.3 V rated |
| High-freq | 4.7 µF 0402 MLCC (C) | 4 | X7R |
| VHF | 100 nF 0402 MLCC (D) | 18 | X7R, placed adjacent to BGA power pins |

---

## Common Mistakes and Exam Tips

**Mistake 1: Forgetting to allocate the voltage budget**

Applying $\Delta V_{max}$ as the full tolerance without subtracting DC contributors leads to an unrealistically tight or too-relaxed target impedance. Always split the budget.

**Mistake 2: Not derating MLCC capacitance**

Failing to apply voltage coefficient derating leads to a PDN that looks fine on paper but has insufficient capacitance at the operating voltage. Use 80–70% of nominal for X5R/X7R at typical operating voltages.

**Mistake 3: Checking only ESR without checking anti-resonance**

Meeting the target at each SRF does not guarantee the target is met at all frequencies. The anti-resonance check between stages is mandatory and is frequently overlooked.

**Mistake 4: Computing $f_{SRF}$ of the parallel bank instead of the individual capacitor**

When $N$ identical capacitors are in parallel, $f_{SRF}$ is unchanged (both $C_{total} = NC$ and $L_{ESL,total} = L/N$, so $f_{SRF} = 1/(2\pi\sqrt{(L/N)(NC)})$ is unchanged). Only the impedance at SRF changes (ESR/N).

**Exam insight:** If asked to justify $Z_{target}$, always mention both the tolerance percentage and the current step — the question implicitly requires both. A PDN question without specifying $I_{step}$ is underspecified.
