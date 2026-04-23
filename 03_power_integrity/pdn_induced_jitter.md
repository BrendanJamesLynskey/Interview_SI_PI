# PDN-Induced Jitter

## Overview

Power integrity and signal integrity are not independent disciplines — they couple through the supply rails of transmitters, PLLs, and CDRs. A noisy core or I/O rail modulates the delay of a clock buffer or the threshold of a slicer, producing jitter and eye closure that no amount of channel optimisation can fix. This coupling is the single most common SI/PI interview "bridge" question because it exposes whether a candidate thinks of the two domains as one system or as silos. This document covers the mechanisms by which PDN noise becomes jitter, the widely cited "1 ps/mV" rule of thumb, how to budget PDN noise against a jitter spec, and mitigation techniques.

---

## Tier 1: Fundamentals

### Q1. How does supply-rail noise turn into jitter on a clock or data edge?

**Answer:**

Every active circuit element has a propagation delay that depends on its supply voltage. In a CMOS inverter, the small-signal delay scales approximately as:

$$t_d \propto \frac{C_L V_{DD}}{I_{drive}}, \quad I_{drive} \propto (V_{DD} - V_{TH})^\alpha$$

with $\alpha \approx 1.2$–$1.5$ for modern short-channel devices. A small change $\Delta V_{DD}$ produces a delay change:

$$\Delta t_d \approx -\frac{\partial t_d}{\partial V_{DD}}\,\Delta V_{DD}$$

A noisy rail thus modulates every inverter edge along a clock or data path. Accumulated over a clock tree with $N$ stages, the jitter induced by a supply-noise step $\Delta V_{DD}$ is:

$$\Delta t_{jitter} \approx N \cdot \kappa \cdot \Delta V_{DD}$$

where $\kappa$ is the per-stage supply-sensitivity coefficient (ps/mV). For modern CMOS at advanced nodes, $\kappa$ per stage is on the order of 0.05–0.2 ps/mV. A clock tree of 10–20 stages accumulates to **roughly 1–2 ps of jitter per mV of supply noise** — the widely cited "**1 ps per mV**" industry rule of thumb.

**Mechanisms by which PDN noise becomes jitter:**

1. **Buffer-chain delay modulation:** clock trees and retiming flops shift in time with $V_{DD}$.
2. **PLL/VCO modulation:** a VCO controlled by a charge pump has its free-running frequency perturbed by supply noise. Even with PLL feedback, noise inside the loop bandwidth leaks through.
3. **Slicer-threshold modulation:** the receiver decision circuit compares against a reference derived from $V_{DD}$ or a bandgap; noise on $V_{DD}$ shifts the threshold, converting amplitude noise into time-domain jitter via the signal slope.
4. **Output driver slew-rate modulation:** a driver's rise/fall times depend on $V_{DD}$; slope changes are visible as DCD (duty-cycle distortion) and edge-position jitter.

---

### Q2. State and justify the "1 ps per mV" rule of thumb. When does it fail?

**Answer:**

**Rule of thumb:** every 1 mV of periodic or broadband noise on the VDD rail of a PLL, clock tree, or SerDes I/O cell produces approximately 1 ps of RMS jitter at the output edge.

**Justification:** for a cascade of $N$ CMOS stages each with per-stage sensitivity $\kappa \approx 0.05$–$0.1$ ps/mV, $N = 10$–$20$ gives $\kappa_{total} \approx 1$ ps/mV. The number varies with process node (newer nodes have lower $V_{TH}$ margin and higher sensitivity), clock-tree depth, and the specific circuit being modulated.

**When the rule underestimates jitter:**

1. **Resonant PDN noise at the PLL loop-BW:** at a frequency where the PLL cannot track the noise but the VCO is still modulated, the conversion gain can be much higher than 1 ps/mV.
2. **Slicer noise on a SerDes RX with low signal slope:** when the received eye is shallow (large IL), the threshold-to-time conversion gain $\Delta t = \Delta V/(dV/dt)$ is large.
3. **VCO on a dedicated analog rail with poor PSRR:** a VCO whose supply PSRR is only 20 dB below the loop bandwidth sees direct frequency modulation, generating PJ spurs.

**When the rule overestimates jitter:**

1. **PLL strongly filters the PDN noise band:** if PDN noise is concentrated inside the PLL bandwidth (which typically rejects up to ~1 MHz) and the PLL has good supply rejection, jitter is suppressed.
2. **On-die LDO regulators isolate sensitive cells** from the noisy core rail — modern SoCs commonly wrap PLLs and SerDes front-ends in on-die low-dropout regulators with > 40 dB PSRR up to 100 MHz.

For an interview answer, state the rule, then immediately note that the *frequency spectrum* of the noise matters as much as its RMS amplitude, because different frequency bands are rejected to different degrees by the PLL and on-die regulation.

---

### Q3. A SerDes receiver specifies a total jitter budget of 0.3 UI at 10⁻¹² BER for 10 Gb/s operation. How much PDN noise (in mV) does this translate to?

**Answer:**

**Setup:**

- UI at 10 Gb/s: 100 ps.
- Total jitter budget: 0.3 × 100 = 30 ps peak-to-peak at 10⁻¹² BER.
- Using Dual-Dirac: $TJ_{pp} = DJ_{pp} + 14.07\,\sigma_{RJ}$. Assume $DJ_{pp} = 10$ ps (channel ISI + crosstalk), leaving $14.07\,\sigma_{RJ} \le 20$ ps, so $\sigma_{RJ} \le 1.42$ ps.

**Allocation to PDN noise:**

Jitter sources add in quadrature (for Gaussian RJ):

$$\sigma_{RJ,total}^2 = \sigma_{RJ,clock}^2 + \sigma_{RJ,PDN}^2 + \sigma_{RJ,thermal}^2$$

Allocate, say, 50% of the RJ budget to PDN-induced RJ:

$$\sigma_{RJ,PDN} = \sqrt{0.5} \cdot 1.42 \approx 1.0\ \text{ps rms}$$

Using the 1 ps/mV rule:

$$V_{PDN,noise} \le 1.0\ \text{mV rms}$$

That is, the rail noise on the VCO/clock-tree supply must stay below 1 mV RMS across the noise bandwidth that the PLL does not reject.

For a typical jitter-relevant bandwidth of 10 kHz – 100 MHz (golden-PLL-filtered), 1 mV rms is extremely tight. This is why modern SerDes parts include dedicated on-die LDOs with > 60 dB PSRR and require the board-level rail to deliver only moderate noise (10–30 mV rms) at the BGA balls.

**Interpretation:** A 30-mV swing on the BGA-level VDD rail is routine; converting it safely into 1 mV rms at the VCO gate requires 30 dB of regulation and filtering between the board and the sensitive circuit. Missing that regulation (or mis-regulating at the resonance frequency of the on-die bypass cap network) is a classic source of jitter failure that looks like a signal integrity problem but is a PI failure.

---

## Tier 2: Intermediate

### Q4. Derive the PDN-induced jitter transfer function for a clock buffer chain. How does jitter at the output depend on the frequency of the PDN noise?

**Answer:**

Consider a chain of $N$ identical inverters, each with supply sensitivity $\kappa$ (ps/mV). The clock edge at the output of stage $k$ is:

$$t_k = t_0 + \sum_{i=1}^{k} \kappa \cdot \Delta V_{DD,i}(t_i)$$

where $\Delta V_{DD,i}(t_i)$ is the supply noise at stage $i$ *at the time* stage $i$'s edge propagates. Because the stage-to-stage delay is short (tens of picoseconds) relative to any low-frequency PDN noise, the noise is effectively constant across the chain for frequencies well below 1/(N·$t_d$). The edge displacement at the output is then:

$$\Delta t_{out}(t) \approx N\,\kappa \cdot \Delta V_{DD}(t)$$

**Frequency-domain transfer:**

$$H_{PDN \rightarrow jitter}(f) = N\,\kappa$$

(flat, to first order, for noise slow compared to chain propagation time).

**For high-frequency PDN noise** ($f \gtrsim 1/(N t_d)$), noise during propagation averages out somewhat because each stage sees a different phase of the noise. The effective gain rolls off, but not dramatically — typically by 3–6 dB by 1 GHz.

**PLL filtering:**

Inside a PLL loop with jitter transfer function $JTF(f) \approx 1 / (1 + (f/f_L)^2)$ (simplified first-order), input jitter at $f \gg f_L$ is rejected. PDN noise sensed by the VCO's supply pin that falls at $f \gg f_L$ is therefore attenuated by the PLL output:

$$H_{PDN \rightarrow PLL\ output}(f) \approx K_{VCO \ supply} \cdot |1 - JTF(f)|$$

(where $K_{VCO\ supply}$ is the VCO's supply-pushing coefficient in Hz/V). The worst-case frequency is right at the PLL's loop bandwidth where the rejection is least.

---

### Q5. Explain PSRR (Power Supply Rejection Ratio) for an on-die LDO and why it matters for PDN-induced jitter.

**Answer:**

An on-die LDO (low-dropout regulator) provides a clean local supply for sensitive circuits (PLL, VCO, SerDes front-end). Its PSRR is the ratio of input-rail noise (noisy global VDD) to output-rail noise (clean local supply), in dB:

$$PSRR(f) = 20\log_{10}\left(\frac{V_{in,noise}(f)}{V_{out,noise}(f)}\right)$$

**Typical PSRR curve:**

- Low frequency (< 10 kHz): > 60 dB (regulator loop gain is high, strong rejection).
- Mid frequency (100 kHz – 10 MHz): 40–60 dB (still within loop bandwidth).
- Near LDO loop bandwidth (typically 10–50 MHz for on-die LDOs): 20–30 dB (rejection degrades).
- Above loop bandwidth (100 MHz+): rejection is limited by the LDO's output capacitance. For a miller-compensated LDO with 10 nF output cap, PSRR at 100 MHz may be only 10–20 dB.

**Why it matters:**

The PDN noise at the BGA ball is typically 10–50 mV peak-to-peak integrated over 1 Hz – 1 GHz. To reach the 1 mV rms target at the VCO, the LDO must provide roughly:

$$PSRR_{required} = 20\log_{10}(V_{in}/V_{out}) = 20\log_{10}(30/1) \approx 30\ \text{dB}$$

Averaged over the jitter-relevant band.

**Failure modes:**

1. **PSRR trough at a specific frequency** coinciding with a sharp PDN peak. This is the most common LDO-related jitter failure: the PDN impedance profile peaks at, say, 50 MHz, while the LDO PSRR bottoms out at the same frequency.
2. **LDO dropout under transient load:** a digital core drawing 100 A suddenly drops the global rail by 20 mV; if the LDO has only 30 mV of headroom, it loses regulation and passes the noise through unfiltered.
3. **LDO oscillation** with incorrect output-cap ESR — an LDO designed for 100 m$\Omega$ ESR caps becomes unstable with modern low-ESR MLCCs.

**Design rule:** when specifying the board-level rail noise budget, treat the LDO as providing 30 dB of filtering above 10 MHz and design the board PDN so that the BGA-level rail noise × 1/1000 is below the sensitive-circuit noise target.

---

### Q6. How does a PDN resonance peak at, say, 200 MHz manifest in the time-domain jitter of a SerDes?

**Answer:**

A PDN impedance peak at $f_{AR} = 200$ MHz causes a $\Delta V$ rail excursion whenever the load current has spectral content at that frequency.

**Chain of events:**

1. The digital core draws load current with a broad spectrum; at 200 MHz the spectrum has some amplitude $I(200 \text{ MHz})$.
2. The rail voltage develops a noise component $V(200 \text{ MHz}) = Z_{PDN}(200) \cdot I(200)$.
3. This 200 MHz sinusoid couples to the SerDes VCO or clock tree supply.
4. The clock edges pick up sinusoidal phase modulation (PM) at 200 MHz.
5. The data eye, observed after CDR recovery, shows **periodic jitter (PJ)** at 200 MHz — a discrete sinusoidal spur in the jitter spectrum.

**Jitter spectrum observation:**

On a BERT with a jitter spectrum analyser, PJ appears as a discrete line at 200 MHz. Its amplitude (peak phase deviation) is:

$$\phi_{peak} = 2\pi f_{data} \cdot \kappa \cdot V_{noise,peak}$$

For $f_{data} = 10$ GHz, $\kappa = 1$ ps/mV, $V_{noise,peak} = 5$ mV: $\phi_{peak} = 2\pi \cdot 10^{10} \cdot 10^{-12} \cdot 5 = 0.31$ rad peak ≈ 1.0 ps peak — detectable with ordinary BERT jitter decomposition.

**Debug signature:**

- BERT jitter spectrum shows a spike at 200 MHz.
- Oscilloscope near-field probe on the SerDes VDD rail shows a 5 mV peak ring at 200 MHz.
- EM simulation shows a PDN impedance peak at 200 MHz.

This is one of the cleanest diagnostic chains in SI/PI: the jitter frequency directly matches the PDN resonance frequency, which directly matches the EM simulation peak. If the three line up, the PDN resonance is the confirmed source of the jitter — the fix is in the package or on-die (reduce the anti-resonance peak), not on the channel.

---

## Tier 3: Advanced

### Q7. A 56 Gb/s PAM4 SerDes specifies its maximum tolerated VDD noise as ±5 mV integrated from 10 kHz to 100 MHz. What does this imply for the board-level PDN design, assuming a typical on-die LDO with 30 dB PSRR above 1 MHz?

**Answer:**

**Tracing the budget back to the BGA:**

The LDO filters its input by 30 dB above 1 MHz. So the input-rail noise allowed at the BGA ball is:

$$V_{BGA,rail} \le 5\ \text{mV} \times 10^{30/20} \approx 158\ \text{mV rms (above 1 MHz band)}$$

Below 1 MHz, the LDO provides 60 dB PSRR:

$$V_{BGA,rail} \le 5\ \text{mV} \times 10^{60/20} \approx 5000\ \text{mV (below 1 MHz band — i.e., any reasonable rail will easily satisfy this)}$$

**Translating to $Z_{PDN}$ target at the BGA:**

The BGA-level rail noise is $V_{BGA} = Z_{PDN}(f) \cdot I(f)$. If the worst-case load current spectrum has $I_{peak} \approx 2$ A at the relevant frequencies:

$$Z_{target}(f > 1\ \text{MHz}) = \frac{158\ \text{mV}}{2\ \text{A}} \approx 80\ \text{m}\Omega$$

This is a **loose** target thanks to LDO filtering — the board PI engineer does not need to push below 80 m$\Omega$ in this band.

**Where the tight constraint really lives:**

Note, however, that the load current is not Gaussian white noise — it peaks at specific frequencies (clock harmonics, I/O activity frequencies). If the PDN has a resonance at one of those frequencies, the *local* $V_{noise}$ can be 10× the broadband estimate. So the target is:

$$Z_{PDN}(f_{resonance}) \le 80\ \text{m}\Omega, \text{across all } f \in [1, 100]\ \text{MHz}$$

and the challenge is ensuring **flatness** — no peaks exceeding the target — not that the average is low.

**Additional subtlety — LDO loop-BW trough:**

The LDO PSRR dips near its loop bandwidth. If the LDO's loop BW is 30 MHz, PSRR at 30 MHz may drop to 20 dB. Near that frequency, the $Z_{PDN}$ target at the BGA *tightens* by 10 dB (factor of ~3). The PI engineer must know the LDO's PSRR profile and tighten the board target where the LDO is weakest.

**Interview takeaway:**

The apparent "simple" noise spec at the silicon pin hides a multi-layer story: on-die LDO PSRR profile → required rail noise at BGA → required $Z_{PDN}(f)$ at BGA → required decoupling and plane design → reconciliation with workload current spectrum. Senior PI engineers work this chain end-to-end.

---

### Q8. Describe the relationship between PDN-induced jitter, clock forwarding strategies, and common-clock architectures in PCIe.

**Answer:**

PCIe supports three reference-clock architectures, each with different PDN-to-jitter coupling:

**1. Common REFCLK (CC):** a shared 100 MHz REFCLK is distributed to both TX and RX devices. Jitter on the REFCLK appears correlated at both ends; the receiver's CDR subtracts it out (jitter common to TX and REFCLK cancels).

- **Key SI/PI implication:** REFCLK jitter tolerance is loose because it is cancelled. The PDN of the REFCLK buffer chain can be noisier.

**2. Separate REFCLK (SRIS):** TX and RX have independent 100 MHz clocks. Any jitter on either REFCLK adds directly to the effective link jitter.

- **Implication:** REFCLK PDN is tight — any rail noise at the REFCLK oscillator or buffer translates directly to data jitter.

**3. Data-clocked (no REFCLK):** more common in chip-to-chip fabrics (UCIe, DDR). The data edges carry clock information; the CDR recovers a clock from them. Here, TX-side PDN noise directly jitters the data, and RX-side PDN noise jitters the CDR's own clock — both contribute.

**PDN strategy by architecture:**

| Architecture | REFCLK PDN criticality | TX SerDes PDN | RX SerDes PDN | Comment |
|---|---|---|---|---|
| Common REFCLK (CC) | Low | Medium | Medium | REFCLK jitter is cancelled |
| Separate REFCLK (SRIS) | **High** | Medium | Medium | REFCLK jitter adds to budget |
| Data-clocked | n/a | **High** | **High** | No rejection of TX rail jitter |

**Emerging chiplet interfaces (UCIe):** low-swing, fixed-latency, data-clocked. PDN noise at either die directly modulates data timing. UCIe's tight package integration gives short clock paths (few buffer stages), which reduces $\kappa$ per link to ~0.1 ps/mV — one reason UCIe can tolerate higher package-level rail noise.

**Interview angle:**

A common pitfall is to treat REFCLK PDN as always critical. The reality is architecture-dependent: in a CC system, the REFCLK PDN can be relaxed, but the TX/RX data-rate clock tree PDN still matters. Make sure to identify *which* clock tree's supply is the one feeding the edges that the receiver sees.

---

## Summary: PDN → Jitter Quick Reference

| Mechanism | Where it happens | Typical sensitivity |
|---|---|---|
| Clock-tree delay modulation | Digital clock trees | 0.5–2 ps/mV (total) |
| VCO supply pushing | PLL/VCO | 1–10 MHz/V (free-running) |
| Slicer threshold modulation | SerDes RX | ∝ 1/slope |
| Driver slew modulation | TX output stage | 0.1–0.5 ps/mV on edge |
| Overall "rule of thumb" | Full chain | ≈ **1 ps/mV rms** |

| Frequency band | Who rejects noise | Typical PSRR |
|---|---|---|
| 0 – 1 kHz | VRM loop | 60–80 dB |
| 1 kHz – 1 MHz | PCB caps + on-die LDO | 40–60 dB combined |
| 1 MHz – 100 MHz | On-die LDO | 30–50 dB |
| 100 MHz – 1 GHz | On-die LDO + PLL filter | 10–30 dB |
| > 1 GHz | On-die cap only | 0–10 dB |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Per-stage jitter sensitivity | $\kappa_i \approx \partial t_d/\partial V_{DD}$ |
| Chain jitter | $\Delta t = N\,\kappa \Delta V_{DD}$ |
| Peak PJ from sinusoidal $V_{noise}$ | $\phi_{peak} = 2\pi f_{data}\,\kappa\,V_{noise,peak}$ |
| PSRR | $PSRR = 20\log_{10}(V_{in}/V_{out})$ |
| Rail noise at BGA for given jitter target | $V_{BGA} = V_{silicon}/10^{PSRR/20}$ |
| $Z_{PDN}$ target at BGA | $Z_{target} = V_{BGA,allowed}/I_{peak}(f)$ |
