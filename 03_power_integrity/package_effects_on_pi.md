# Package Effects on Power Integrity

## Overview

The package is the middle tier of every PDN: between the PCB-level network (VRM, bulk capacitors, plane pair, mid-frequency MLCCs) and the on-die capacitance lies the package PDN, with its own inductance, resistance, capacitance, and resonant structure. This middle tier is the choke point that decouples the PCB's effective bandwidth from the die's needs. PCB capacitors, no matter how perfectly placed, cannot help the die above the frequency at which the package inductance dominates the impedance seen looking into the package from the die. On-package decoupling capacitors (OPD caps) are the answer: they sit *inside* the package to bypass package-induced inductance and bridge the impedance gap between PCB and die. This document covers package PDN structure, the bandwidth limits the package imposes on PCB-level PI work, the design and selection of on-package decoupling, and chip-package-board (CPB) co-design.

---

## Tier 1: Fundamentals

### Q1. Where does the package fit in the PDN topology, and what does it contribute electrically?

**Answer:**

The PDN is conventionally drawn as three concentric tiers, each with its own dominant frequency band:

```
DC ─────────────────────────────────────────────────────────► high f

   ┌────────────┐   ┌──────────────┐   ┌──────────────┐
   │    PCB     │ → │   PACKAGE    │ → │     DIE      │
   │ (VRM, bulk │   │ (planes,     │   │ (on-chip cap │
   │  + MLCC)   │   │  vias, OPD)  │   │  + decap)    │
   └────────────┘   └──────────────┘   └──────────────┘
   DC – ~100 MHz    ~30 MHz – ~1 GHz   ~500 MHz – tens of GHz
```

The package contributes the following electrical elements between the PCB power pad and the die power bump:

1. **Vertical via inductance** through the package substrate from each BGA ball to the corresponding power plane.
2. **Power/ground plane pair inductance and capacitance** within the package substrate.
3. **Bump (C4) inductance** from the package power planes to the die power pads.
4. **Resistance** of all of the above (typically tens of milliohms for a high-pin-count power net).
5. **On-package decoupling capacitors** (if fitted) — discrete or integrated capacitors mounted on the substrate.

A typical lumped equivalent for a single power net through the package, looking from the BGA ball to the die bump, is:

$$Z_{pkg}(\omega) \approx j\omega(L_{ball} + L_{plane} + L_{bump}) + R_{pkg}\,\Big\|\,\frac{1}{j\omega C_{pkg,planes} + Y_{OPD}(\omega)}$$

For a modern flip-chip BGA with hundreds of power balls in parallel, total $L_{pkg}$ from BGA balls to die bumps is typically **30–200 pH**, and parasitic plane capacitance is **0.5–5 nF**. These small values are misleading: divided by *number of power pins in parallel* they represent the *total* package PDN inductance, so per-pin inductance is much higher (order 5–50 nH), which is why the number of power balls and their distribution matters so much.

---

### Q2. What are typical values of package PDN inductance, and why does this number dominate PI design above ~100 MHz?

**Answer:**

The total package PDN inductance $L_{pkg,PDN}$ seen from the die looking down into the package depends on:

- **Number of power balls** ($N_p$): inductance scales as $1/N_p$ for parallel paths.
- **Length of via from ball to plane.**
- **Pitch and thickness of power/ground plane pair** (closer planes = lower inductance).
- **Number of bumps** from plane to die.

Representative numbers for modern flip-chip BGAs:

| Package class | Power balls | Bumps | $L_{pkg,PDN}$ ball-to-die |
|---|---|---|---|
| Small (< 100 balls), single-die | 5–20 | ~50 | 200–800 pH |
| Mid (200–500 balls), low-power SoC | 30–80 | 200–600 | 80–300 pH |
| Large (1000+ balls), high-perf CPU/GPU | 200–500+ | 5000+ | 20–80 pH |
| HBM stack on interposer | 100–500 | 1000s | 30–100 pH |

**Why this dominates above ~100 MHz:**

The impedance contributed by the package inductance alone is $|Z_{L,pkg}| = 2\pi f L_{pkg,PDN}$. For $L_{pkg,PDN} = 100\ pH$:

| Frequency | $|Z_{L,pkg}|$ |
|---|---|
| 1 MHz | 0.6 m$\Omega$ |
| 10 MHz | 6 m$\Omega$ |
| 100 MHz | 63 m$\Omega$ |
| 1 GHz | 630 m$\Omega$ |

A PDN target impedance of 1–5 m$\Omega$ at the die is routine for a modern high-current part. Above ~100 MHz the package inductance alone *exceeds* the target impedance — meaning **no PCB-level capacitor, however ideal, can lower the die-side impedance below this floor**. The only way to reduce $Z_{die,seen}$ above the package corner is to put capacitance on the *die side* of the package inductance — i.e., on-package decoupling caps and on-die capacitance.

**Common mistake:** Spending iteration after iteration adding more PCB MLCCs to fix die-side noise above 100 MHz. The PCB PI engineer cannot fix this; the solution lives inside the package (OPD) or on the die.

---

### Q3. What is on-package decoupling (OPD), and why is it needed in addition to PCB and on-die capacitance?

**Answer:**

On-package decoupling refers to capacitors physically mounted on or embedded in the **package substrate**, between the PCB BGA balls and the die bumps. They sit inside the package, electrically *between* the package's vertical inductance and the die.

**Why OPD is needed:**

Consider the PDN as a chain:

```
[VRM + bulk] → L_pcb → [PCB MLCC] → L_pkg_ball → [OPD] → L_pkg_bump → [on-die cap] → die
```

Each inductance in the chain forms a low-pass barrier. PCB MLCCs are isolated from the die by the full $L_{pkg}$, and on-die capacitance is small and expensive. OPD caps fill the gap by:

1. **Providing low impedance in the 30–500 MHz band**, where PCB MLCCs are already inductive and on-die capacitance is not yet dominant.
2. **Bypassing the package vertical inductance**, allowing transient currents to flow locally on the package without propagating down to the PCB.
3. **Damping package PDN resonances** that arise between $L_{pkg}$ and the parallel combination of PCB caps and on-die caps.

**Where OPD lives physically:**

- **Bottom side of package substrate (LSC — land-side capacitors):** Discrete MLCCs (0201 or 01005) soldered to the bottom of the package substrate, in the void between BGA balls. Adds 200–1000 nF total in a typical CPU/GPU package.
- **Top side of substrate (between die and lid):** Less common; constrained by die-mounting area.
- **Embedded in substrate (IPD — Integrated Passive Device):** Thin-film capacitors built into the substrate layers themselves. Very low ESL but low capacitance density.
- **Silicon/IPDIC capacitors flip-chipped next to the die:** Dedicated thin-film silicon capacitors (deep-trench MOS or MIM) bonded as additional dies on the package. High capacitance density (μF in mm² area) and low ESL (< 50 pH).

**Without OPD, two failure modes occur:**

1. The die-side impedance has a large peak at the package PDN resonance frequency (typically 50–200 MHz), causing rail noise spikes when the load draws current at that frequency.
2. The total PDN cannot meet $Z_{target}$ above ~50 MHz, forcing either reduced clock speed, larger guard bands, or excessive on-die decoupling area (which is silicon-area expensive).

OPD is therefore not optional for any high-performance digital part; the question is only what type and how much.

---

### Q4. What types of capacitors are used for on-package decoupling, and how do they compare?

**Answer:**

**1. Land-Side Capacitors (LSC) — discrete MLCCs:**

Standard 01005 or 0201 ceramic MLCCs (typically X5R or X7R, 10–470 nF) soldered to the bottom of the package substrate, in the unused area between BGA balls.

| Property | Typical value |
|---|---|
| Capacitance per part | 10–470 nF |
| ESL | 100–300 pH (better than PCB-mounted by ~5×) |
| ESR | 10–30 m$\Omega$ |
| Effective frequency band | 30–500 MHz |
| Cost per part | ~$0.001 |

LSCs are the workhorse of OPD because they are cheap, well understood, and easy to add or remove during package design iterations.

**2. Embedded Capacitors / IPD:**

Thin-film capacitors fabricated as part of the package substrate stackup, typically using a thin high-$\epsilon_r$ dielectric (e.g., BaTiO$_3$, 0.5–5 µm thick) sandwiched between substrate copper layers.

| Property | Typical value |
|---|---|
| Capacitance density | 0.1–10 nF/mm² |
| ESL | 5–50 pH (very low — direct connection to plane) |
| Effective frequency band | 100 MHz – 2 GHz |
| Cost premium | Significant — adds substrate complexity |

Their advantage is extremely low ESL because there is no via or solder joint separating the capacitor from the planes.

**3. Silicon Decoupling Capacitors (IPDIC / DTC / MIM):**

A separate small silicon die fabricated specifically as a high-density capacitor, flip-chipped onto the package substrate next to the main die.

- **Deep-Trench MOS Capacitors (DTC):** Etched trenches in silicon increase the surface area; capacitance density 100–500 nF/mm². Used in high-performance CPUs and GPUs.
- **MIM (Metal-Insulator-Metal):** Lower density (~10 nF/mm²) but very low ESR and ESL; used in RFIC packages.

| Property | Typical value |
|---|---|
| Capacitance per chip | 100 nF – 10 µF |
| ESL | 10–80 pH |
| ESR | 1–10 m$\Omega$ |
| Effective frequency band | 50 MHz – 5 GHz |
| Cost | High (extra silicon process) |

Silicon decap is the highest-performance OPD option and is now standard on flagship CPUs, GPUs, and AI accelerators.

**4. On-Die Capacitance (for context):**

On-die decoupling — MOS-cap, MOM-cap, deep-trench — sits on the same die as the load logic. ESL is essentially zero (picohenries) because there is no off-die connection. Effective up to several gigahertz.

| Property | Typical value |
|---|---|
| Capacitance density | 5–30 nF/mm² (MOS) up to 200 nF/mm² (DTC) |
| ESL | 1–10 pH |
| Effective frequency band | 500 MHz – 20 GHz |
| Silicon area cost | Very high (1 nF can occupy 0.05 mm² of die area) |

**Comparison summary:**

| Tier | Capacitor type | Effective $f$ band | ESL |
|---|---|---|---|
| PCB | Bulk + MLCC | DC – 30 MHz | 0.5–5 nH |
| Package | LSC MLCC | 30–500 MHz | 100–300 pH |
| Package | Embedded/IPD | 100 MHz – 2 GHz | 5–50 pH |
| Package | Silicon decap | 50 MHz – 5 GHz | 10–80 pH |
| Die | On-die | 500 MHz – 20 GHz | 1–10 pH |

The selection is dictated by the impedance and frequency that must be supported and by the budget for cost, area, and power.

---

## Tier 2: Intermediate

### Q5. Quantitatively, how does the package's bandwidth limit what PCB-level PI work can achieve? Derive the maximum PCB-cap-effective frequency from the package inductance.

**Answer:**

This is one of the most important questions in PCB-level PI engineering and is the core reason OPD exists.

**Setup:**

Consider a single power net. The PCB engineer adds capacitance $C_{PCB}$ at the BGA edge with effective ESL $L_{PCB}$ (the cap's own ESL plus its mounting inductance). Looking from the *die* through the package and into the PCB cap, the path is:

```
die ──[L_pkg]── BGA ball ──[L_PCB]── [C_PCB] ── ground
```

The PCB cap's effective behaviour at the die is what matters. Decompose by frequency:

- At low $f$, $C_{PCB}$ has high reactance and dominates: $|Z| \approx 1/(\omega C_{PCB})$ — capacitive, decreasing with $f$.
- At intermediate $f$, $|Z|$ flattens at the PCB-cap ESR.
- Above the **mounted SRF** of the PCB cap, the inductance $L_{PCB}$ dominates the cap branch: $|Z| \approx \omega L_{PCB}$ — rising.
- Above some still-higher $f$, the *package* inductance $L_{pkg}$ in series above the cap dominates, and the PCB cap looks like a short far below the ground plane: it can no longer affect the die-side impedance.

The boundary frequency above which the PCB cap is **invisible from the die** is:

$$f_{PCB,limit} \approx \frac{1}{2\pi}\cdot\frac{Z_{target}}{L_{pkg}}$$

This is derived from the requirement that the package inductance alone must be lower than $Z_{target}$ for the PCB cap to make a meaningful difference at the die. Above this frequency, $|Z_{L,pkg}| > Z_{target}$, so even *zero-impedance PCB capacitance* would not pull the die-seen impedance below the target.

**Numerical example:**

Modern CPU package: $L_{pkg} = 50\ \text{pH}$, $Z_{target} = 2\ \text{m}\Omega$:

$$f_{PCB,limit} = \frac{1}{2\pi} \cdot \frac{2\times10^{-3}}{50\times10^{-12}} \approx 6.4\ \text{MHz}$$

**That is, above approximately 6 MHz, no PCB-side decoupling can help the die meet a 2 m$\Omega$ target.** Every PI improvement at the die above this frequency must come from OPD or on-die capacitance.

**Looser target, modest package:** $L_{pkg} = 500\ \text{pH}$, $Z_{target} = 50\ \text{m}\Omega$:

$$f_{PCB,limit} = \frac{1}{2\pi} \cdot \frac{50\times10^{-3}}{500\times10^{-12}} \approx 16\ \text{MHz}$$

PCB still only helps up to ~16 MHz. Anything above is a package-level problem.

**Practical implications for PCB-level PI:**

1. **Set the PCB PI bandwidth realistically.** The PCB PI engineer's job ends near 10–50 MHz for most modern parts; above that, you are wasting effort optimising MLCCs that the die cannot see.
2. **Coordinate with the package team.** If the die-side impedance is failing at 200 MHz, you cannot fix it with PCB caps. Push back to silicon/package for more OPD or more on-die capacitance.
3. **Use the right tools.** PCB-only PI simulators that exclude package and on-die models will produce optimistic results at the die. Always include the vendor-supplied package PDN model in any sign-off simulation.

This bandwidth split between PCB and package is the most common source of misallocated effort in PI engineering.

---

### Q6. Describe the "frequency handoff" between PCB capacitors, on-package decoupling, and on-die capacitance. Where are the typical crossover points and how is anti-resonance managed at each handoff?

**Answer:**

Each PDN tier handles a frequency band, and each adjacent pair of tiers has a *handoff* — a frequency region where one tier's effectiveness is fading and the next is taking over. Each handoff is potentially an anti-resonance point.

**Tier 1 → Tier 2: PCB MLCC → Package OPD handoff (~10–100 MHz)**

- Above the PCB MLCC's mounted SRF, the PCB cap branch becomes inductive ($\omega L_{PCB,mounted}$).
- The OPD branch becomes the lower-impedance path.
- **Anti-resonance:** $L_{PCB,mounted}$ in parallel with $C_{OPD}$ across the package L. Peak at:

$$f_{AR,1} = \frac{1}{2\pi\sqrt{(L_{PCB,mounted} + L_{pkg,upper}) \cdot C_{OPD,total}}}$$

For $L = 1\ nH$, $C_{OPD} = 200\ nF$: $f_{AR} \approx 11\ MHz$. Peak impedance $|Z| \approx \sqrt{L/C} = \sqrt{10^{-9}/200\times10^{-9}} \approx 70\ m\Omega$.

Mitigation: increase $C_{OPD}$, lower $L_{PCB,mounted}$, or rely on the ESR of either stage to damp the peak.

**Tier 2 → Tier 3: Package OPD → on-die cap handoff (~200 MHz – 1 GHz)**

- Above OPD SRF, OPD branch is inductive.
- On-die capacitance takes over.
- **Anti-resonance:** $L_{OPD,mounted} + L_{pkg,bump}$ in parallel with $C_{on-die}$. Peak typically in the 100–500 MHz region.

This is often the **dominant PDN noise peak** at the die and the most difficult to suppress because the only available damping is OPD ESR (small for ceramic) and the spreading resistance of die metal layers.

Mitigation: silicon decap with controllable ESR, distributed on-die decap, current-shaping in the load (die-side controlled di/dt).

**Crossover-point rules of thumb:**

| Handoff | Typical $f_{cross}$ | Dominant inductance |
|---|---|---|
| Bulk → MLCC (PCB only) | 100 kHz – 1 MHz | Bulk cap ESL + PCB trace |
| PCB MLCC → OPD | 10 MHz – 100 MHz | Mounted MLCC ESL + via |
| OPD → on-die | 200 MHz – 1 GHz | OPD ESL + bump array |

**Key observation:** Each handoff has a peak whose magnitude is $\sqrt{L/C}$. To keep the peak low, you want the *inductance between the two stages to be small* and *the next-stage capacitance to be large*. Both knobs are physical-design problems (more bumps, more balls, denser OPD, lower-ESL OPD types).

**Visualising the impedance profile:**

```
|Z|  ▲
 1 Ω │                                            ╱╲    ← package L only (no OPD)
     │                              ╱╲          ╱   ╲
100 mΩ                  ╱╲       ╱╲   ╲      ╱      ╲
     │             ╲ ╱   ╲     ╱   ╲    ╲   ╱        ╲
10 mΩ │  ╲       ╲ ╱       ╲ ╱    AR    AR            ╲
     │     ╲   ╲ ╱           ╳    ╳                     ╲
 1 mΩ │       ╳ VRM   PCB MLCC   OPD       on-die         
     │      ╱   ╲   ╱                                          
100 µΩ│   ╱       ╲                                            
     └──┬─────┬─────┬─────┬─────┬─────┬─────┬─── f
        DC   1k   100k   10M  100M   1G   10G
```

A flat impedance at $Z_{target}$ across the band is the goal; the anti-resonance peaks at each handoff are the obstacles.

---

### Q7. How do you compute the OPD capacitance required for a given target impedance and package inductance?

**Answer:**

The OPD must satisfy two constraints simultaneously:

**Constraint 1: Sufficient capacitance at the lowest OPD-relevant frequency.**

Above $f_{PCB,limit}$, OPD must hold $|Z| \leq Z_{target}$. At the low edge of the OPD band, the OPD presents capacitive reactance:

$$|Z_{OPD,cap}| = \frac{1}{2\pi f \cdot C_{OPD,total}} \leq Z_{target}$$

$$C_{OPD,total} \geq \frac{1}{2\pi f_{PCB,limit} \cdot Z_{target}}$$

For $f_{PCB,limit} = 6\ \text{MHz}$, $Z_{target} = 2\ \text{m}\Omega$:

$$C_{OPD} \geq \frac{1}{2\pi \times 6\times10^6 \times 2\times10^{-3}} \approx 13\ \mu\text{F}$$

For a CPU-class package this is achievable as ~30–60 LSC parts of 470 nF each, or ~3 silicon decap dice.

**Constraint 2: Sufficient damping (ESR) to keep the anti-resonance peak below $Z_{target}$.**

The OPD–to–on-die anti-resonance peak (without ESR damping) is:

$$|Z_{AR}|_{undamped} \approx \sqrt{\frac{L_{OPD,mounted} + L_{pkg,bump}}{C_{on-die}}}$$

To keep this below $Z_{target}$, either:

- Add enough on-die capacitance: $C_{on-die} \geq L/(Z_{target}^2)$. For $L = 50\ pH$, $Z_{target} = 2\ m\Omega$: $C_{on-die} \geq 12.5\ nF$ (challenging on a small die).
- Add ESR to the OPD: target $R_{OPD,total} \approx \sqrt{L/C_{on-die}}$ for critical damping. This is part of why some OPD parts intentionally use higher-ESR dielectrics or include a series resistor.
- Reduce $L$ — bump density and OPD-to-die proximity matter more than capacitance value once minimum cap is met.

**Constraint 3: Number of OPD parts required.**

Once total capacitance is chosen, divide into individual parts based on:

- Available area on the package substrate.
- Per-part ESL (lower with smaller case sizes).
- Per-part voltage derating.

Place parts on **the bottom side directly under the BGA**, distributed across the power net to minimise spreading inductance from any one bump.

**Iteration in practice:**

The PI engineer rarely has full freedom on all three parameters. Typical workflow:

1. Vendor publishes target $Z(f)$ profile and the on-die decap fabric is fixed by silicon design.
2. Package designer selects OPD type (LSC vs silicon decap) based on cost/perf trade.
3. PI engineer places OPD, simulates, checks the AR peak against $Z_{target}$.
4. If AR violates target, options are (a) add OPD to lower SRF, (b) increase on-die decap (silicon design ECO), (c) accept the violation and constrain workload current spectra. Option (c) sometimes wins.

---

## Tier 3: Advanced

### Q8. How does the package contribute to PDN resonance, and what tools are used to verify package-level PI before silicon tape-out?

**Answer:**

Package-level PDN resonance arises wherever a closed loop forms between an inductance and a capacitance in the package substrate. The dominant resonant structures are:

**1. Plane-pair cavity resonance:**

The power and ground planes of the package form a thin parallel-plate cavity. As covered for PCB plane resonance, modes occur at:

$$f_{m,n} = \frac{c}{2\sqrt{\epsilon_r}} \sqrt{(m/a)^2 + (n/b)^2}$$

Smaller package planes mean higher fundamental mode (3–10 GHz typical), but Q can be high without distributed damping.

**2. OPD anti-resonance:**

Already covered above — between OPD and on-die capacitance, mediated by bump-array inductance.

**3. Bump-array resonance:**

The 2-D periodic bump array supporting current flow from package planes into die exhibits Floquet (periodic structure) resonances. Typically these appear above 10 GHz but are increasing in importance as core current density grows.

**4. Lid/heatspreader cavity resonance:**

A metal lid above the die forms a resonant cavity that can couple into the package PDN through bumps and bond wires (in lidded wirebond parts). Mostly above 5 GHz.

**Verification workflow (pre-tape-out package PI sign-off):**

1. **3-D EM extraction of package PDN.** Tools: Ansys SIwave/Q3D/HFSS, Cadence Sigrity PowerSI/PowerDC/Clarity 3D, Mentor HyperLynx PI. The package geometry — substrate layers, planes, vias, bumps, BGA — is meshed and a Touchstone or SPICE PDN model is extracted.

2. **Concatenate with PCB PDN model.** The PCB-side PI extraction is already in hand; the two are joined at the BGA ball ports and the on-die capacitance and load model are attached at the bump-side ports.

3. **Frequency-domain $|Z(f)|$ analysis.** Sweep the impedance seen at the die from DC to ~10× the highest expected current spectrum frequency. Identify peaks. Verify all peaks lie below $Z_{target}(f)$.

4. **Time-domain transient simulation.** Apply realistic current excitation (extracted from RTL/gate-level activity vector or from a behavioural workload model) to the on-die ports. Compute the resulting $V_{rail}(t)$ and check it stays inside the rail tolerance.

5. **Workload-aware verification.** Modern tools support "spectrum-aware" checks where the measured or simulated current spectrum is multiplied by $|Z(f)|$ to predict noise spectrum, integrated to give RMS noise and checked against worst-case allowable.

6. **Power-aware SI co-simulation.** With OPD and bump models, signal and PDN are simulated together to expose ground-bounce-induced jitter and SSN (simultaneous-switching noise).

**Sign-off rule:** $|Z_{PDN}(f)| \leq Z_{target}(f)$ at the die ports across the full spectrum the load can excite, including all anti-resonance peaks, with margin (typically 30–50%) for process and temperature variation.

---

### Q9. What is chip-package-board (CPB) co-design, and how does it change the traditional partitioning of PI responsibilities?

**Answer:**

Traditional PI partitioning treats each tier as a separate engineering domain:

- Silicon team designs on-die capacitance and the load (di/dt profile).
- Package team designs OPD placement and substrate planes.
- Board team designs PCB caps, planes, and VRM.

Each delivers a model (S-parameters, lumped network) at its boundary, and adjacent teams treat the other as a fixed black box. This worked when PDN bandwidth requirements were modest; it breaks when the three tiers must jointly meet $Z_{target}$ across a band that no single tier can cover.

**CPB co-design** is the integrated workflow where chip, package, and board PI are designed and verified together, with each team able to influence the others' design choices.

**Workflow:**

1. **Define $Z_{target}(f)$ at the die** based on workload current spectrum and rail tolerance.
2. **Allocate the impedance budget** across PCB/package/die at each frequency band:
   - DC to ~1 MHz: VRM dominates.
   - 1–30 MHz: PCB MLCCs.
   - 30 MHz – 500 MHz: OPD.
   - 500 MHz – multi-GHz: on-die.
3. **Build a unified PDN model** incorporating all three tiers, with editable parameters (number of OPD parts, on-die cap area, PCB cap stages).
4. **Co-optimise across tiers.** Reducing die area for capacitance might be enabled by increasing OPD; reducing PCB cap count might be enabled by tighter VRM bandwidth.
5. **Trade-off analysis.** Cost/performance trades become explicit: silicon decap is silicon-area expensive; OPD is package-cost expensive; on-die regulation is design-effort expensive. CPB co-design lets the system architect choose the cheapest combination meeting the impedance target.

**Tools:**

- Ansys RedHawk-CPS, Cadence Sigrity Aurora/Topology Explorer, Synopsys PrimePower – chip-aware tools that link load models from RTL/gate-level simulation back into PDN analysis.
- Industry-standard CPM (Chip Power Model) and CPM-X formats for distributing chip-side current and capacitance models to package and board teams without revealing IP.

**Why CPB matters:**

For modern multi-core CPUs, GPUs, and AI accelerators, transient currents of 100–500 A with $dI/dt$ in the 100 A/ns range create voltage noise that no single tier can absorb. Only a co-designed PDN — with VRM bandwidth, PCB caps, OPD, on-die regulation, and load-line management all engineered together — meets the target. CPB co-design is now standard practice in any high-performance digital programme.

**Implication for the engineer's role:**

The classical division between "PCB PI guy" and "package PI guy" is dissolving. A senior PI engineer is expected to reason fluently across all three tiers, understand the constraints each imposes on the others, and participate in the trade-off discussions that allocate the impedance budget across the system.

---

### Q10. A real-world debug scenario: the die-side rail noise exceeds spec at 150 MHz. Walk through the diagnosis and the candidate fixes, identifying which can be done at PCB level and which require package or silicon changes.

**Answer:**

**Symptom:** Die-side rail $V_{DD,core}$ shows peak-to-peak noise of 80 mV at 150 MHz; spec is 50 mV. Failure occurs only under heavy compute load.

**Step 1 — Identify the impedance peak.**

Run a frequency-domain PDN simulation of the full PDN (PCB + package + on-die) and plot $|Z(f)|$ at the die. Confirm a peak at 150 MHz with magnitude exceeding $Z_{target}$.

**Step 2 — Locate the peak in the topology.**

The 150 MHz region is well above $f_{PCB,limit}$ for any reasonable $L_{pkg}$. So the peak is between OPD and on-die capacitance, likely the OPD–on-die anti-resonance:

$$f_{AR,2} = \frac{1}{2\pi \sqrt{(L_{OPD} + L_{pkg,bump}) \cdot C_{on-die}}}$$

For $L = 75\ pH$ and $C_{on-die} = 15\ nF$: $f_{AR} \approx 150\ \text{MHz}$. Confirms the peak is the OPD/on-die anti-resonance.

**Step 3 — Candidate fixes, in order of cost:**

| Fix | Effective? | Where? | Cost |
|---|---|---|---|
| Add more PCB MLCCs | **No** — far above $f_{PCB,limit}$ | PCB | Free in design, no benefit |
| Move PCB caps closer to BGA | **No** — same reason | PCB | Free, no benefit |
| Tighten on-die termination | Possibly — reduces excitation at AR freq | Silicon | High — silicon ECO |
| Add OPD parts (more LSC) | Yes — lowers $f_{AR}$ and adds damping | Package | Medium — package re-spin |
| Switch OPD from MLCC to silicon decap | Yes — lower ESL, higher density | Package | High — substrate redesign |
| Add on-die capacitance | Yes — moves AR down, reduces peak | Silicon | Very high — silicon area |
| Spectral shape the load (firmware/clock-gating) | Yes — avoids exciting 150 MHz | Workload | Medium — system effort |
| Lower core current via DVFS at risk frequencies | Yes — reduces $\Delta I$ at 150 MHz | Silicon/firmware | Low — performance impact |

**Step 4 — Realistic resolution:**

In production, the most common fix is *workload shaping* (clock gating, randomised stall insertion, DVFS step rates) combined with *adding OPD on the next package re-spin*. Adding PCB capacitors is **the wrong fix** but is, in practice, the first thing inexperienced PI engineers try.

**Lessons:**

1. Always identify *which* tier owns a given impedance peak before attempting fixes.
2. Above the package L-dominated frequency, PCB-level changes cannot help — the only routes are package-level (OPD), silicon-level (on-die cap), or workload-level (load shaping).
3. Fixing high-frequency die-side noise costs silicon mask sets and package redesigns; the time to address it is during initial CPB co-design, not after silicon comes back.

---

## Summary: PCB / Package / Die PI Quick Reference

| Frequency band | Dominant tier | Adjustable by |
|---|---|---|
| DC – 1 kHz | VRM (PCB) | PCB designer |
| 1 kHz – 1 MHz | Bulk caps (PCB) | PCB designer |
| 1 MHz – 30 MHz | MLCC (PCB) | PCB designer |
| **30 MHz – 500 MHz** | **OPD (package)** | **Package designer (PCB cannot help)** |
| 500 MHz – multi-GHz | On-die caps | Silicon designer |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Package L impedance | $|Z_{L,pkg}| = 2\pi f L_{pkg}$ |
| Max PCB-cap-effective frequency | $f_{PCB,limit} \approx Z_{target} / (2\pi L_{pkg})$ |
| Required OPD capacitance | $C_{OPD} \geq 1 / (2\pi f_{PCB,limit} \cdot Z_{target})$ |
| OPD/on-die anti-resonance frequency | $f_{AR} = 1 / (2\pi\sqrt{(L_{OPD}+L_{bump})\cdot C_{on-die}})$ |
| OPD/on-die anti-resonance peak | $|Z_{AR}| \approx \sqrt{(L_{OPD}+L_{bump})/C_{on-die}}$ |
| Critical damping ESR for AR peak | $R_d \approx \sqrt{L/C_{on-die}}$ |
| Package plane resonance | $f_{m,n} = (c/2\sqrt{\epsilon_r})\sqrt{(m/a)^2+(n/b)^2}$ |
