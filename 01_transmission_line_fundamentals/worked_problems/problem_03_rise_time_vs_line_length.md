# Problem 03: Rise Time vs. Line Length — When Is a Trace Electrically Long?

## Problem Statement

A digital hardware engineer is designing a PCB for a new SoC-based product. They must decide which of the following traces require transmission-line treatment (controlled impedance, termination strategy, and wave-propagation analysis) and which can be treated as simple lumped capacitors.

**Stackup:** FR4, all traces are stripline. Propagation delay: 175 ps/inch.

The traces under review are:

| Trace ID | Length | Signal | Rise time ($t_r$, 20–80%) | Clock/data rate |
|---|---|---|---|---|
| A | 0.3 inch | I$^2$C clock (SCL) | 100 ns | 400 kHz |
| B | 0.5 inch | SPI clock | 5 ns | 50 MHz |
| C | 2.0 inches | LPDDR5 address | 200 ps | 4267 MT/s |
| D | 0.75 inch | PCIe Gen 4 TX | 30 ps | 16 GT/s |
| E | 8.0 inches | 1 Gb Ethernet (SerDes) | 50 ps | 1.25 Gb/s |
| F | 1.5 inches | 3.3 V power-on reset | 10 ms | N/A (event-driven) |

**Tasks:**

**(a)** Apply the $T_D \geq t_r/6$ rule to classify each trace as electrically long (transmission-line treatment required) or electrically short (lumped treatment acceptable). Show all calculations.

**(b)** For each trace classified as electrically long, state the termination strategy you would recommend and justify the choice.

**(c)** Trace D (PCIe Gen 4, 0.75 inch, 30 ps rise time). The $t_r/6$ rule classifies this trace. Your colleague argues: "0.75 inch is a short trace, no termination needed." Evaluate their argument using both the electrical-length criterion and a physical argument about reflections.

**(d)** Trace F is a slow reset signal with a 10 ms rise time. Your manager asks you to add a 50 $\Omega$ series termination resistor to it anyway "just in case." Evaluate this request and determine whether there is any SI concern.

**(e)** A PCIe Gen 5 trace (rise time 15 ps, 32 GT/s) is being routed on the same board. What is the maximum trace length that can be treated as electrically short? Express in both inches and millimetres.

---

## Solution

### Part (a): Classification Using the $t_r/6$ Rule

**Formula:** The trace is electrically long if $T_D \geq t_r/6$, where $T_D = \text{length} \times 175\ \text{ps/inch}$.

**Trace A — 0.3 inch I$^2$C clock, $t_r = 100$ ns:**

$$T_D = 0.3 \times 175 = 52.5\ \text{ps}$$

$$\frac{t_r}{6} = \frac{100 \times 10^3\ \text{ps}}{6} = 16,667\ \text{ps}$$

$T_D = 52.5\ \text{ps} \ll 16,667\ \text{ps}$. **Electrically SHORT.**

**Trace B — 0.5 inch SPI clock, $t_r = 5$ ns:**

$$T_D = 0.5 \times 175 = 87.5\ \text{ps}$$

$$\frac{t_r}{6} = \frac{5000\ \text{ps}}{6} = 833\ \text{ps}$$

$87.5\ \text{ps} \ll 833\ \text{ps}$. **Electrically SHORT.** ($T_D/t_r = 0.0175$, far below the $1/6 = 0.167$ threshold.)

**Trace C — 2.0 inch LPDDR5 address, $t_r = 200$ ps:**

$$T_D = 2.0 \times 175 = 350\ \text{ps}$$

$$\frac{t_r}{6} = \frac{200}{6} = 33.3\ \text{ps}$$

$350\ \text{ps} \gg 33.3\ \text{ps}$. Ratio $= 10.5$. **Electrically LONG.**

**Trace D — 0.75 inch PCIe Gen 4, $t_r = 30$ ps:**

$$T_D = 0.75 \times 175 = 131\ \text{ps}$$

$$\frac{t_r}{6} = \frac{30}{6} = 5\ \text{ps}$$

$131\ \text{ps} \gg 5\ \text{ps}$. Ratio $= 26.2$. **Electrically LONG.**

**Trace E — 8.0 inch Ethernet SerDes, $t_r = 50$ ps:**

$$T_D = 8.0 \times 175 = 1400\ \text{ps}$$

$$\frac{t_r}{6} = \frac{50}{6} = 8.3\ \text{ps}$$

$1400\ \text{ps} \gg 8.3\ \text{ps}$. Ratio $= 168$. **Electrically LONG.**

**Trace F — 1.5 inch power-on reset, $t_r = 10$ ms:**

$$T_D = 1.5 \times 175 = 262.5\ \text{ps}$$

$$\frac{t_r}{6} = \frac{10 \times 10^9\ \text{ps}}{6} = 1.67 \times 10^9\ \text{ps}$$

$262.5\ \text{ps} \ll 1.67 \times 10^9\ \text{ps}$. **Electrically SHORT** by an enormous margin.

**Classification summary:**

| Trace | $T_D$ (ps) | $t_r/6$ (ps) | $T_D / (t_r/6)$ | Classification |
|---|---|---|---|---|
| A | 52.5 | 16,667 | 0.003 | SHORT |
| B | 87.5 | 833 | 0.105 | SHORT |
| C | 350 | 33.3 | 10.5 | **LONG** |
| D | 131 | 5.0 | 26.2 | **LONG** |
| E | 1400 | 8.3 | 168 | **LONG** |
| F | 262.5 | $1.67 \times 10^9$ | $1.6 \times 10^{-7}$ | SHORT |

---

### Part (b): Termination Strategy for Electrically Long Traces

**Trace C — LPDDR5 address (2 inches, 200 ps rise time):**

LPDDR5 uses a fly-by topology (series bus with point-of-load termination). The address bus is a parallel bus with many loads.

**Recommendation: On-die ODT (On-Die Termination) at the receiving device, plus source impedance matching at the transmitter.**

LPDDR5 specifies that termination is implemented inside the DRAM device (ODT resistors switched on during write cycles) and at the controller (during read cycles). The designer does not add discrete termination resistors. The layout must maintain 40 $\Omega$ trace impedance per JEDEC specification with $\pm 10$% tolerance.

If discrete termination were needed (e.g., legacy DDR4): **parallel termination to $V_{TT} = V_{DDQ}/2$** at the last device in the chain, providing a matched impedance throughout the bus.

**Trace D — PCIe Gen 4 (0.75 inch, 30 ps rise time):**

PCIe uses AC-coupled differential signalling with embedded impedance matching. The PCIe specification requires 85 $\Omega$ differential impedance ($\pm 15$%).

**Recommendation: No discrete termination required.** The PCIe PHY contains built-in on-die termination (typically 50 $\Omega$ per wire, making 100 $\Omega$ differential). The board trace must be a controlled-impedance differential pair routed to the specification; the termination is integrated into the silicon.

The board designer's responsibility is impedance control, not termination component placement. The 0.75 inch length, while electrically long by the criterion, is short relative to the PCIe receiver's equalization capability. Standard PCIe Gen 4 channels specify up to 16 dB insertion loss (including connectors and long traces) — a 0.75 inch stub is negligible.

**Trace E — 1 Gb Ethernet SerDes (8 inches, 50 ps rise time):**

This is a long channel with high loss. 1000BASE-T Ethernet uses 4D-PAM5 modulation with complex DSP at the PHY.

**Recommendation: Controlled impedance (100 $\Omega$ differential), no discrete termination.** The Ethernet PHY integrates on-die 100 $\Omega$ differential termination at both ends. The channel is fully AC-coupled through magnetics (Ethernet isolation transformer). The board designer must:
- Maintain 100 $\Omega$ differential impedance
- Minimise via stubs on the signal path
- Keep differential-to-common mode conversion below specification
- Keep the channel insertion loss below the PHY's equalization budget

---

### Part (c): PCIe Gen 4, 0.75 Inch — Evaluating the Colleague's Argument

**Colleague's argument:** "0.75 inch is a short trace, no termination needed."

**Evaluation — Electrical-length criterion:**

As calculated in Part (a), Trace D has $T_D = 131$ ps and $t_r/6 = 5$ ps. The ratio is 26.2 — the trace is 26 times over the threshold for electrically long behaviour. Describing it as "short" in a SI context is incorrect; it is one of the longest traces relative to its rise time in this problem set.

**Physical argument:**

The reflection coefficient for a mismatched source driving this trace is, for a typical PCIe PHY output impedance of 50 $\Omega$ single-ended (100 $\Omega$ differential) into an 85 $\Omega$ differential line:

$$\Gamma = \frac{85 - 50}{85 + 50} = \frac{35}{135} = +0.26$$

An initial reflected wave of 26% of the incident wave amplitude would return from the load discontinuity after $2 \times 131 = 262$ ps. For a 30 ps rise time signal, the round-trip delay is $262/30 \approx 9$ rise times — the reflected wave is a completely distinct event, not merely a superposition with the rising edge. The reflected pulse would cause:

1. A 26% voltage perturbation at the source after 262 ps — visible as undershoot or overshoot depending on load type
2. A second smaller reflection from the source back toward the load after 524 ps

At 16 GT/s PCIe Gen 4, the bit period is 62.5 ps. A 262 ps echo would arrive 4 bit periods after the original edge — spreading inter-symbol interference across 4 bits. The eye would be significantly degraded.

**However — and this is critical for the interview:** The PCIe Gen 4 PHY contains on-die termination that creates $\Gamma_S \approx 0$ and $\Gamma_L \approx 0$. The PHY is designed so that when driven into its specified impedance trace, no significant reflections occur. The colleague's error is not in the length argument but in thinking that "short" means "no termination needed." The correct statement is: the PCIe PHY already provides integrated termination, so no *additional* discrete components are needed. But the trace must be routed to the specified impedance — a 0.75 inch PCIe trace with wrong impedance (e.g., 65 $\Omega$ due to poor layout) would cause reflections regardless of length.

---

### Part (d): Series Termination on 10 ms Reset Signal

**The manager's request:** Add a 50 $\Omega$ series terminator to the power-on reset line "just in case."

**SI analysis:**

Trace F has $T_D = 262.5\ \text{ps}$ and $t_r = 10\ \text{ms}$. The ratio $T_D/(t_r/6)$ is $1.6 \times 10^{-7}$ — essentially zero. No transmission-line effect exists for this signal. The trace behaves as a lumped capacitance of approximately:

$$C = T_D / Z_0 = 262.5 \times 10^{-12} / 50 = 5.25\ \text{pF}$$

Adding a 50 $\Omega$ series resistor creates an RC filter with the lumped capacitance:

$$\tau = R_S \times C_{trace} = 50 \times 5.25 \times 10^{-12} = 0.26\ \text{ns}$$

This is negligible relative to a 10 ms rise time. The terminator will not cause any SI problem.

**However, there are practical reasons to push back on this request:**

1. **Component cost and assembly:** Adding a 0402 resistor costs approximately $0.002–$0.005 per unit. At high volume, this adds up. Every component is also a potential solder defect and adds to board assembly time.

2. **DC voltage drop:** If the reset line sources or sinks significant current (e.g., driving multiple device reset pins), the 50 $\Omega$ resistor creates a voltage drop. A 10 mA drive current creates a 0.5 V drop — potentially causing the reset to fail to reach the logic-high threshold.

3. **Reset timing:** The RC time constant with load capacitance determines reset release timing. Adding a series resistor delays the signal reaching the threshold on reset assertion.

**Recommendation:** No series termination is needed on the reset line from a signal integrity perspective. The manager's concern is unwarranted and the component adds unnecessary cost and potential functional risk. If the manager's concern is about noise on the reset line (e.g., from switching supply transients), a small RC filter ($R = 100$–$470\ \Omega$, $C = 1$–$10$ nF) as a specific anti-bounce or noise filter is a more appropriate design choice.

---

### Part (e): Maximum Electrically Short Length for PCIe Gen 5

**PCIe Gen 5 parameters:** Data rate 32 GT/s, rise time $t_r = 15$ ps.

**Maximum electrically short length:**

Using the $t_r/6$ criterion: maximum $T_D = t_r/6$.

$$T_D^{max} = \frac{15\ \text{ps}}{6} = 2.5\ \text{ps}$$

**Converting to physical length:**

$$l_{max} = \frac{T_D^{max}}{175\ \text{ps/inch}} = \frac{2.5\ \text{ps}}{175\ \text{ps/inch}} = 0.0143\ \text{inch}$$

$$\boxed{l_{max} = 14.3\ \text{mil} \approx 0.36\ \text{mm}}$$

**Interpretation:**

Any PCIe Gen 5 trace longer than 0.36 mm (14 mil) is electrically long and requires full transmission-line treatment. In practice, this means every single trace on a PCIe Gen 5 board — including the pads, vias, package breakout stubs, and connector transitions — must be treated as a transmission-line discontinuity and properly modelled.

**Comparison across PCIe generations:**

| Generation | Data rate | $t_r$ (approx.) | Max "short" length |
|---|---|---|---|
| Gen 1 | 2.5 GT/s | 250 ps | 238 mil (6 mm) |
| Gen 2 | 5 GT/s | 120 ps | 114 mil (2.9 mm) |
| Gen 3 | 8 GT/s | 60 ps | 57 mil (1.4 mm) |
| Gen 4 | 16 GT/s | 30 ps | 29 mil (0.7 mm) |
| Gen 5 | 32 GT/s | 15 ps | 14 mil (0.36 mm) |
| Gen 6 | 64 GT/s | 8 ps | 7.6 mil (0.19 mm) |

The trend is clear: with each generation doubling the data rate, the maximum lumped-circuit trace length halves. At PCIe Gen 6, even the pad dimensions (typically 20–30 mil) exceed the electrically-short threshold. This is why PCIe Gen 6 transitions to PAM-4 (Pulse Amplitude Modulation with 4 levels) to reduce the symbol rate — the Nyquist frequency of 16 GHz allows the signal integrity community slightly more electrical length per symbol than a 32 Gbaud NRZ scheme would require.

---

## Key Takeaways

1. **The $t_r/6$ rule is based on rise time, not data rate.** Always determine rise time from the signalling standard or the driver's datasheet before applying the rule.

2. **Slow signals are never electrically long.** I$^2$C, SPI at 50 MHz, and slow control signals do not require transmission-line treatment regardless of trace length on a typical PCB.

3. **High-speed SerDes and DDR signals are always electrically long.** Even short traces (0.75 inch) require impedance control and proper termination strategy at PCIe Gen 4 speeds and beyond.

4. **Integration of termination into silicon ICs changes the designer's task.** Modern high-speed ICs (PCIe PHY, DDR DRAM) include on-die termination. The board engineer's job shifts from placing discrete termination components to achieving precise controlled impedance in the trace and via geometry.

5. **PCIe Gen 5/6 represents a paradigm shift.** When no trace on the board is electrically short, the entire channel design — from package to package — must be treated as a distributed microwave structure, not a circuit of lumped elements.
