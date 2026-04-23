# Package Effects on Signal Integrity

## Overview

The IC package sits between the silicon die and the PCB and is part of every signal channel — every transmitted bit traverses bond wires or copper bumps, package routing layers, and BGA balls before reaching the PCB. From an SI perspective the package is an extension of the transmission line system: it adds parasitic inductance and capacitance, introduces an impedance discontinuity, contributes its own crosstalk and reflections, and at multi-gigahertz signalling rates can be the dominant loss element in the channel. This document covers how the package degrades signal integrity, how its parasitics are characterised and modelled, and the design techniques (flip-chip, controlled-impedance package routing, BGA pin assignment, package S-parameter extraction) used to manage these effects in modern high-speed designs.

---

## Tier 1: Fundamentals

### Q1. What constitutes the "package" in an electrical sense, and what parasitic elements does it add to a signal path?

**Answer:**

Electrically, the package is the entire interconnect structure between the active silicon and the PCB pad. For a typical organic flip-chip BGA package the signal path is:

```
die pad → micro-bump (C4) → RDL/under-bump metal →
package routing layer (Cu trace) → via → BGA ball pad → solder ball
```

For an older wire-bonded QFN/QFP/BGA package the path is:

```
die pad → bond wire (Au or Cu) → lead frame / package trace →
package terminal (lead or BGA ball)
```

Every one of these elements is electrically short compared to a PCB trace but is no longer negligible at high speeds. The lumped parasitic elements added per signal pin are:

| Element | Bond-wire BGA | Flip-chip BGA |
|---|---|---|
| Lead/wire inductance | 2–8 nH | 0.1–0.5 nH (bump + trace) |
| Lead/pad capacitance to ground | 0.3–1 pF | 0.1–0.3 pF |
| Series resistance | 50–200 m$\Omega$ | 20–80 m$\Omega$ |
| Mutual inductance to neighbour | 0.5–2 nH | 0.05–0.3 nH |

These contributions are *per pin* and concentrated in a few millimetres of vertical extent. They form a low-pass filter at the channel input and output, modify the local characteristic impedance, and (because they are shared along densely packed rows of pins) couple signals to one another.

**Common mistake:** Treating the package as ideal because "it's only a few mm long". A 5 nH bond wire driven by a 50 ps edge produces $V = L\,dI/dt = 5\times10^{-9}\times(20\times10^{-3}/50\times10^{-12}) = 2\ \text{V}$ of ground bounce per amp of switching current — large enough to corrupt nearby signals.

---

### Q2. How does package series inductance affect signal rise time and the observed waveform at the receiver?

**Answer:**

Package series inductance $L_{pkg}$ in the signal path forms a low-pass filter with the line impedance $Z_0$ and the load capacitance $C_L$ (or with the receiver input). The single-pole response from a series-$L$ network driving a $Z_0$ line has a $-3\ \text{dB}$ corner at:

$$f_{3dB} \approx \frac{Z_0}{2\pi L_{pkg}}$$

For $Z_0 = 50\ \Omega$ and $L_{pkg} = 2\ nH$: $f_{3dB} \approx 4\ \text{GHz}$.

The corresponding rise-time degradation through a single-pole filter:

$$t_{r,filter} \approx \frac{0.35}{f_{3dB}} = \frac{0.35 \times 2\pi L_{pkg}}{Z_0}$$

For the same $L_{pkg} = 2\ nH$: $t_{r,filter} \approx 88\ ps$. If the driver edge is 50 ps and the package adds 88 ps of filtering, the output edge slows to:

$$t_{r,out} = \sqrt{t_{r,driver}^2 + t_{r,filter}^2} = \sqrt{50^2 + 88^2} \approx 101\ ps$$

The package alone has roughly doubled the effective rise time. This shows up at the eye-diagram level as reduced eye height (high-frequency loss) and increased ISI jitter.

**Reflection effect:** A series inductor on a $Z_0$ line also produces a positive reflection. The reflection coefficient seen looking into a series $L$ on a $Z_0$ line is approximately:

$$\Gamma(\omega) \approx \frac{j\omega L_{pkg}/2}{Z_0 + j\omega L_{pkg}/2}$$

At low frequencies $\Gamma \rightarrow 0$; at high frequencies $\Gamma \rightarrow 1$. The package therefore looks like a reflective high-pass discontinuity to TDR — visible as a sharp positive spike on the impedance trace at the package transition.

---

### Q3. What is the difference between bond-wire and flip-chip packaging from a signal integrity perspective?

**Answer:**

**Bond-wire packaging** (QFN, QFP, leaded BGA, older wire-bond BGA): the die is glued face-up onto the package substrate, and signals exit from peripheral pads on the die via thin (25–50 µm) gold or copper bond wires that arc over to landing pads on the substrate or lead frame. Bond wires are 1–3 mm long, present 1–3 nH/mm inductance, and have significant mutual coupling to neighbouring wires.

**Flip-chip packaging** (organic flip-chip BGA, FC-CSP): the die is flipped face-down and joined to the package substrate by a dense array of solder bumps (C4) or copper pillars distributed over the entire die surface. The vertical interconnect is approximately 100 µm tall, presents 0.05–0.2 nH per bump, and the area-array layout breaks up the long parallel-wire crosstalk paths characteristic of bond wires.

| Property | Bond wire | Flip chip |
|---|---|---|
| Per-pin series $L$ | 2–8 nH | 0.1–0.5 nH |
| Per-pin mutual $L$ | 0.5–2 nH | 0.05–0.2 nH |
| Pin-out flexibility | Peripheral only | Area array |
| Useful signalling BW | ≤ 5 GHz | 25–50 GHz+ |
| Fabrication cost | Low | High |
| Die size scaling | Limited by perimeter | Scales with area |

**SI consequences:**

- **Achievable data rate:** Bond-wire packages cap practical NRZ data rates around 5–10 Gb/s per lane. Flip-chip is required for PCIe Gen 4+, DDR5, and all SerDes above ~10 Gb/s.
- **Power and signal pin separation:** With bond wires, supply and return pins must be paired carefully because each bond wire's $L$ contributes to ground bounce. Flip-chip C4s are short and permit dense interleaving of power and ground pins, dramatically reducing simultaneous-switching noise.
- **Crosstalk profile:** Bond wires have long parallel runs in air and exhibit strong inductive coupling between adjacent wires, especially for wires that run side by side over long spans. Flip-chip bumps are point connections with much shorter coupling length.

For interview purposes: any high-speed standard above ~5 Gb/s is, in practice, a flip-chip part. Specifying a bond-wire package for a multi-gigabit interface is an immediate red flag.

---

### Q4. What is "ball-out" or pin-out, and why is the assignment of signals to package balls a critical SI design choice?

**Answer:**

The ball-out (or pin-map) is the physical assignment of die signals to package balls in the BGA grid. It determines:

1. **Which signals are adjacent on the package**, controlling near-end and far-end crosstalk through the package routing.
2. **The escape routing pattern on the PCB**, controlling how cleanly signals can be fanned out on the board without unnecessary layer transitions or via-induced parasitics.
3. **The proximity of each signal to a return-current ground ball**, controlling loop inductance.
4. **The physical length of the package trace** from die to ball, controlling per-channel skew.

**Key SI principles for ball-out design:**

- **Signal–ground ball ratio:** For high-speed signals, a minimum of 1 ground ball per signal (1:1) is the starting point. For SerDes lanes above 25 Gb/s, production practice is typically **2:1 (ground:signal) or higher** — each differential pair is surrounded by ground balls on all non-pair sides, so the ratio rises to 3–4 grounds per P/N pair. Each signal ball should have ground balls as immediate orthogonal neighbours to provide a low-inductance return path.
- **Differential pairs adjacent:** True (P) and complement (N) of a differential pair are placed on adjacent balls, and the pair is surrounded by ground balls on remaining sides.
- **Aggressor isolation:** Single-ended high-speed signals (e.g., DDR DQ) are spaced apart by ground or low-activity signals to limit crosstalk on the package and during PCB escape.
- **Power/ground placement:** Power and ground balls are distributed across the centre of the BGA (under the die area) to provide low-inductance current paths to the PDN, not concentrated at the periphery where they would force long current loops through the package.
- **Length matching:** Signals belonging to the same source-synchronous group (e.g., DDR DQ\[7:0\] of a single byte) are placed on balls that yield similar in-package routing lengths to minimise package-induced skew. Vendor datasheets publish per-pin package delays for exactly this purpose.

A poorly chosen ball-out cannot be fixed by the PCB designer; it forces compromises at every layer of the system. Modern high-speed silicon vendors devote significant SI effort to ball-out optimisation and publish package-level S-parameters or .pkg models so customers can verify channel performance before PCB layout begins.

---

## Tier 2: Intermediate

### Q5. How does package crosstalk arise, and how does it differ from PCB-level crosstalk?

**Answer:**

Package crosstalk arises in three distinct regions of the package, each with its own coupling mechanism:

**1. Bond-wire / bump array crosstalk (vertical interconnect):**

Adjacent bond wires (or, less severely, adjacent C4 bumps) share magnetic flux. The mutual inductance $L_m$ between two parallel wires of length $\ell$, diameter $d$, separation $s$ is approximately:

$$L_m \approx \frac{\mu_0 \ell}{2\pi}\left[\ln\left(\frac{2\ell}{s}\right) - 1\right]$$

For $\ell = 2\ mm$, $s = 200\ \mu m$: $L_m \approx 0.4\ nH$. The induced voltage on a victim wire is $V_{xtalk} = L_m\,dI/dt$. With $dI/dt = 100\ mA / 50\ ps = 2\times10^9\ A/s$: $V_{xtalk} \approx 0.8\ V$. This is huge by SI standards — bond-wire crosstalk is the dominant SI limiter for legacy packages above 1 Gb/s.

**2. Package-trace crosstalk (horizontal routing):**

Inside the package substrate, signals are routed on copper layers similar to a miniature PCB. Coupling here behaves exactly like PCB crosstalk (NEXT and FEXT, capacitive plus inductive coupling) but at a smaller scale: trace spacings are 25–75 µm rather than hundreds of micrometres, dielectric heights are 25–50 µm, and dielectric materials (e.g., ABF for organic substrates) have $D_k \approx 3.0$–$3.5$, lower than FR-4. The smaller geometry means coupling is concentrated over shorter physical lengths, but the coupling per unit length can be similar to PCB.

**3. Via and BGA-ball-array crosstalk (vertical exit):**

The BGA ball field acts as a 2-D array of vertical interconnects. Adjacent balls couple inductively in the same way as adjacent bond wires. This is mitigated by careful ball-out (interleaving grounds) and by limiting ball pitch (smaller pitch reduces $L_m$ but increases manufacturing difficulty).

**Differences from PCB crosstalk:**

| Property | Package crosstalk | PCB crosstalk |
|---|---|---|
| Coupling length | Short (1–5 mm) | Long (cm to m) |
| Coupling per unit length | High (tight pitch) | Lower (looser pitch) |
| Dominant mechanism | Inductive (especially bond wires) | Capacitive + inductive comparable |
| Fix in design | Ball-out, in-package routing | Spacing, ground guards |
| Fix at customer level | None — frozen by package | Partial — can adjust trace spacing |

**Implication:** Package crosstalk is a *fixed property* of the chosen part. The customer cannot improve it after package selection. This makes vendor-supplied package S-parameter or model data essential for system-level SI sign-off.

---

### Q6. The package introduces an impedance discontinuity between the on-die transmission line and the PCB trace. How does this discontinuity affect the channel, and how do designers control it?

**Answer:**

A signal travelling from the die driver to the PCB encounters at least three impedance regions:

1. **On-die interconnect:** $Z_0 \approx 30$–$60\ \Omega$ depending on metal layer and process.
2. **Package routing:** $Z_0 \approx 40$–$80\ \Omega$, controlled by trace geometry and dielectric.
3. **PCB trace:** $Z_0 = 50\ \Omega$ single-ended or 85–100 $\Omega$ differential, by spec.

At each interface (die-to-package via the bump/wire and package-to-PCB via the BGA ball) the local impedance changes abruptly, causing reflections.

**Reflection magnitude:**

The reflection coefficient at a transition from $Z_1$ to $Z_2$ is:

$$\Gamma = \frac{Z_2 - Z_1}{Z_2 + Z_1}$$

A typical bond-wire transition presents a series inductance that, at the knee frequency, can momentarily raise the local impedance by 20–40 $\Omega$, producing $\Gamma \approx 0.2$–$0.3$ — that is, 20–30% reflection. Without compensation, this produces ringing, eye closure, and ISI.

**Control techniques:**

1. **Controlled-impedance package routing:** Package substrates are designed with stripline or microstrip layers of known $D_k$ and trace geometry, just like a PCB, and target $Z_0 = 50\ \Omega$ (or the differential equivalent) within ±10%. Modern flip-chip BGA substrates routinely achieve this for all signals.

2. **Bump and ball compensation:** The pad and via under each bump or ball can be sized to tune the local capacitance, partially cancelling the inductance of the vertical transition. This is done in 3-D EM simulation by sweeping pad diameter, antipad clearance, and stub length to flatten the TDR profile through the transition.

3. **Reference-plane placement:** A solid ground reference layer immediately below the package signal layer keeps the local $Z_0$ controlled and provides a tight return path. Discontinuities in the reference plane (gaps, splits, ferrite beads in the wrong place) inside the package cause large impedance bumps; modern packages avoid this by design.

4. **Length-matched, shielded differential pairs:** For differential signalling, the P and N pads, bumps, traces, vias and balls are designed as a coupled pair throughout the package, with ground planes above and below and ground bumps/balls around. This preserves differential mode impedance and minimises mode conversion (differential-to-common) through the discontinuity.

5. **Co-simulation with the channel:** The package S-parameters (or .pkg model) are concatenated with the on-die circuit and the PCB channel in a single time-domain or frequency-domain simulation, ensuring the package-induced reflections are accounted for in the eye margin budget.

**Common mistake:** Designing the PCB channel to a $Z_0$ specification independently of the package, then assuming the system works because both sub-systems are "controlled". A package with an uncompensated impedance bump will close the eye by 100–200 mV regardless of how well the PCB is designed — and the only fix is at the package level.

---

### Q7. What are package S-parameters, how are they extracted, and how are they used in channel simulation?

**Answer:**

Package S-parameters are an N-port frequency-domain description of the package as a passive linear network, where the ports are the die-side bumps/pads and the board-side balls. For an N-pin signal interface (e.g., a 32-bit DDR data bus), the package model has $2N$ ports — $N$ on the die side and $N$ on the board side — and the S-parameter matrix captures every transmission, reflection, and crosstalk path through the package.

**Extraction methods:**

1. **3-D EM simulation:** The package geometry (substrate layers, traces, vias, bumps, balls) is meshed and solved with a full-wave EM solver (HFSS, CST, Clarity 3D). This is the standard method for advanced packages and produces broadband Touchstone (.s2p, .s4p, … .sNp) files good from DC to 50+ GHz.
2. **Vector network analyser (VNA) measurement:** A test fixture lands probes on the package balls and either probes or wirebonds to the die-side pads. Used for correlation and for parts where a 3-D model is unavailable.
3. **Hybrid approach:** Critical structures (bumps, vias, BGA exits) are 3-D EM simulated and concatenated with 2-D extracted package traces. Faster than full 3-D for large packages with many signals.

**How models are delivered to customers:**

- **Touchstone (.sNp) file:** Standard frequency-domain S-parameter format for the entire signal interface.
- **IBIS package model (.pkg):** Lumped R-L-C per-pin model embedded in the IBIS file. Suitable only for low-speed signalling where a single L–C lump is adequate.
- **IBIS-ISS or SPICE subcircuit:** A distributed R-L-C-coupled-line model exported from the EM simulator. More accurate than the .pkg lump for high-speed signals.
- **Power-aware models (BIRD-compliant):** Couple signal and PDN behaviour through shared package planes and bumps — needed for ground-bounce-aware simulation.

**Use in channel simulation:**

The full channel concatenation for an end-to-end SI sign-off is:

```
TX I/O (IBIS-AMI) → TX-side package S-params → PCB channel S-params (TX side)
→ connector/cable → PCB channel S-params (RX side) → RX-side package S-params
→ RX I/O (IBIS-AMI)
```

The simulator cascades the S-parameters (with renormalisation to the system port impedance), feeds the resulting channel model to the AMI behavioural transmitter and receiver models, and produces the eye diagram, BER bathtub, and jitter decomposition at the receiver decision point.

**Practical considerations:**

- Package S-parameters must be **passive** and **causal**; non-passive Touchstone files cause time-domain simulators to diverge. Modern post-processors enforce passivity.
- The model must extend at least **3× the knee frequency** of the signal to capture rise-time content. For a 25 Gb/s NRZ signal with $f_{knee} \approx 17.5\ \text{GHz}$, the model should be valid to at least 50 GHz.
- For PAM-4 signalling, model fidelity must extend further because the three eyes have different SNR sensitivities and the channel response near $f_{Nyquist}/2$ matters more than for NRZ.

---

## Tier 3: Advanced

### Q8. How do 2.5D interposer and 3D-stacked packages change SI considerations compared to traditional flip-chip BGA?

**Answer:**

Traditional flip-chip BGA places a single die on an organic substrate. **2.5D** packaging (silicon interposer) and **3D** stacking (die-on-die with TSVs) change the topology fundamentally.

**2.5D (silicon interposer):**

Multiple dice (e.g., a CPU/GPU and several HBM stacks) are flip-chipped onto a silicon interposer roughly 100 µm thick. The interposer provides high-density wiring (sub-µm linewidths versus tens of µm in organic substrates) and through-silicon vias (TSVs) connecting the interposer top side to the C4 bumps on its bottom side. The interposer is then flip-chipped onto a conventional organic package substrate.

**SI implications:**

- **Die-to-die interconnect on the interposer is short (mm-scale) and dense.** This enables wide, slow, parallel buses (e.g., HBM 1024-bit interface at 1–6 Gb/s/pin) with very low energy per bit and very low latency. Die-to-die SI on the interposer is essentially a controlled-impedance microstrip problem at small scale; loss is negligible because traces are 1–10 mm.
- **Signals leaving the interposer to the board still see the conventional flip-chip package transition.** External SerDes performance at the package-to-board boundary is unchanged from a regular flip-chip part.
- **TSVs introduce a new vertical discontinuity.** Each TSV is approximately 50–100 µm long with 5–10 µm diameter, presenting roughly 30–80 pH series inductance and 50–200 fF shunt capacitance. The local impedance dip can be tuned by TSV pitch and dielectric thickness.

**3D stacking (HBM, hybrid bonding):**

Multiple dice are stacked vertically and connected by TSVs (HBM uses TSVs through every die layer except the top) or by direct copper-to-copper hybrid bonding (sub-µm pitch, near-zero parasitic). The result is essentially zero interconnect delay between stacked layers.

**SI implications:**

- **Inter-layer SI is dominated by TSV parasitic L and C, not by transmission-line loss.** Channels are too short for transmission-line behaviour; lumped RLC is the appropriate model.
- **Crosstalk between TSVs is the dominant SI concern**, especially in high-density TSV arrays. EM simulation with detailed TSV geometry is essential.
- **Thermal coupling matters for SI** because junction temperatures of stacked dice rise faster than single-die packages, and many SI parameters (driver strength, on-die termination, jitter) drift with temperature.

**Comparison summary:**

| Topology | Inter-die SI domain | Channel reach | Dominant parasitic |
|---|---|---|---|
| Single-die flip-chip | N/A | 5 mm (in-package) + cm (PCB) | C4 bump + package trace |
| 2.5D interposer | Interposer microstrip | mm | Interposer trace + TSV |
| 3D stacked (TSV) | TSV array | < 100 µm | TSV inductance, capacitance |
| 3D hybrid bonded | Direct Cu | < 1 µm | Negligible |

The trend is unmistakable: as packaging technology advances, the bandwidth bottleneck migrates from the package interior outward to the package-to-board interface, and ultimately to the PCB itself.

---

### Q9. What are package resonances, where do they occur in the signal path, and how do they couple to signal integrity?

**Answer:**

A package contains multiple structures that behave as electromagnetic resonators above 1 GHz. The dominant ones are:

**1. Power/ground plane resonance in the package substrate:**

The package substrate's power and ground planes form a resonant cavity, just like PCB plane resonance, but at higher frequencies because the planes are smaller. For a rectangular plane pair of dimensions $a \times b$ filled with dielectric of $\epsilon_r$, the resonant modes are at:

$$f_{m,n} = \frac{c}{2\sqrt{\epsilon_r}} \sqrt{\left(\frac{m}{a}\right)^2 + \left(\frac{n}{b}\right)^2}$$

For a 25 mm × 25 mm package substrate with $\epsilon_r = 3.5$: $f_{1,0} \approx 3.2\ \text{GHz}$. Modes pile up above this frequency.

**SI coupling:** Switching currents on the power plane excite these modes. The standing-wave voltage on the plane couples to nearby signal traces through shared reference planes, producing periodic "bumps" in the channel transfer function and corresponding ringing in the time-domain response.

**Mitigation:** Dense, distributed on-package decoupling capacitors damp these modes. Use of multiple thin power/ground plane pairs (lower characteristic impedance per pair) reduces the Q of each mode.

**2. Bump/ball array resonance:**

A periodic 2-D array of bumps or balls connected to power and ground planes acts as a 2-D filter structure. Periodic loading produces stop bands where the structure becomes highly resonant. For typical BGA pitches (0.4–1.0 mm) the first stop band falls in the 5–20 GHz range — above the most demanding signal bandwidths today but encroaching as data rates rise.

**3. Antipad / via resonance:**

The antipad clearance around a signal via in a power plane creates a small resonant cavity. The first resonance is approximately:

$$f_{antipad} \approx \frac{c}{2\sqrt{\epsilon_r}\,d_{antipad}}$$

For $d_{antipad} = 0.5\ mm$, $\epsilon_r = 3.5$: $f_{antipad} \approx 160\ \text{GHz}$. Currently above signal bands of interest, but it is a real consideration in millimetre-wave packaging.

**4. Cavity resonance under the lid (for lidded packages):**

A metal lid or heatspreader above the die creates a small cavity. At high frequencies it resonates and can couple radiated emissions back onto sensitive signals or oscillator pins.

**Identification and verification:**

- Package S-parameters $S_{11}$, $S_{21}$, and the transfer impedance $Z_{21}$ from a power port to a signal port show characteristic peaks at resonance frequencies.
- 3-D EM simulation of the full package, including bumps, vias, plane shapes, decoupling capacitors, and (where applicable) lid, predicts the resonance spectrum.
- Time-domain TDR of fabricated packages reveals reflections corresponding to resonant structures; spectral content of the ringing matches the resonance frequencies.

---

### Q10. How is the package modelled inside an IBIS-AMI channel simulation, and what are the limits of the IBIS-style lumped package model?

**Answer:**

IBIS (I/O Buffer Information Specification) provides a behavioural description of an I/O cell — V/I curves, switching waveforms, and a simple package parasitic block. The package block in IBIS 5.0 and earlier is a **single-segment lumped R-L-C network per pin** with optional matrix elements describing pin-to-pin coupling.

**The IBIS .pkg model:**

```
[Package]
| variable    typ        min        max
R_pkg        20m        15m        30m
L_pkg        2.5n       2.0n       3.0n
C_pkg        0.4p       0.3p       0.5p
```

This describes each pin as a series R–L with shunt C, and is adequate for two regimes:

- Signalling well below the first package resonance (rule of thumb: $f_{signal} < f_{resonance}/10$).
- Power-pin modelling where only the gross inductance matters for ground-bounce calculations.

**Where the lumped model fails:**

- For SerDes above ~3 Gb/s, the package transmission-line behaviour, distributed coupling, and reference-plane effects matter; a lumped R-L-C per pin produces optimistic eye margins and underpredicts crosstalk.
- For DDR4 and faster, the rise-time content extends to 5+ GHz and the lumped model misses package resonance and inter-pin coupling.

**Modern alternatives within IBIS-AMI flow:**

- **Package model in S-parameter form:** IBIS 6.0 introduced the `[External Circuit]` and `[Package Model]` keywords that allow embedding a Touchstone S-parameter file (or SPICE subcircuit) as the package model. The simulator then concatenates the S-parameters with the AMI Tx/Rx and the PCB S-parameters.
- **BIRD 95 / 116 / 158** etc.: industry-standard IBIS extensions adding power-aware models that couple signal and PDN behaviour through the package.

**Practical workflow:**

For 25 Gb/s SerDes channel simulation:

1. Vendor extracts package S-parameters from 3-D EM simulation, delivers .s4p (per differential pair, both ends).
2. PCB channel is extracted as .s4p (also per pair) from the board EM simulator.
3. The channel simulator (e.g., ADS, HSPICE, SystemVue, custom Python flow) concatenates: AMI Tx → TX package .s4p → PCB .s4p → RX package .s4p → AMI Rx.
4. The output is BER bathtub, eye diagram, and jitter/noise decomposition at the receiver slicer.

The package's contribution to the total SerDes channel loss budget can easily be 1–3 dB at Nyquist for a flip-chip BGA, and the package's contribution to the residual ISI and crosstalk is often the swing factor that determines whether a channel meets compliance.

---

## Summary: Package SI Quick Reference

| Concern | Bond-wire BGA | Flip-chip BGA | Mitigation |
|---|---|---|---|
| Lead inductance | 2–8 nH/pin | 0.1–0.5 nH/pin | Choose flip-chip for > 5 Gb/s |
| Crosstalk | Inductive, severe | Inductive, mild | Ground interleaving, ball-out |
| Impedance discontinuity | Large positive bump | Compensated to ±10% | Controlled-Z package routing |
| Effective bandwidth | ≤ 5 GHz | 25–50 GHz+ | Package selection |
| Resonances | Above 5 GHz | Above 3 GHz (substrate planes) | OPD caps damp Q |
| Modelling fidelity needed | Lumped RLC OK | Distributed S-parameters required | IBIS 6.0 + Touchstone |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Package L–Z low-pass corner | $f_{3dB} = Z_0 / (2\pi L_{pkg})$ |
| Rise-time degradation by $L_{pkg}$ | $t_{r,filter} = 0.35 \cdot 2\pi L_{pkg} / Z_0$ |
| Two-wire mutual inductance | $L_m \approx (\mu_0 \ell / 2\pi)[\ln(2\ell/s) - 1]$ |
| Reflection at $Z_1 \rightarrow Z_2$ transition | $\Gamma = (Z_2 - Z_1)/(Z_2 + Z_1)$ |
| Package plane resonance | $f_{m,n} = (c/2\sqrt{\epsilon_r})\sqrt{(m/a)^2 + (n/b)^2}$ |
| Channel rise-time addition (RSS) | $t_{r,out} = \sqrt{t_{r,driver}^2 + t_{r,filter}^2}$ |
| Knee frequency (signal-derived) | $f_{knee} \approx 0.35 / t_r$ |
