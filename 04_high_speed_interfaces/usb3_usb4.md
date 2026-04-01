# USB 3.x and USB4

## Overview

USB (Universal Serial Bus) has evolved from a low-speed peripheral interface into a high-performance multi-protocol interconnect. USB 3.x (SuperSpeed) operates at 5, 10, and 20 Gbps per lane using NRZ differential signalling with 8b/10b or 128b/132b encoding. USB4 (based on the Thunderbolt 3/4 architecture) supports up to 40 Gbps over two-lane channels and can tunnel PCIe, DisplayPort, and USB protocols simultaneously.

From a signal integrity perspective, USB 3.x and USB4 share many characteristics with other high-speed serial links (controlled impedance, equalization, cable/connector loss budgets) but introduce unique challenges: the Type-C connector and cable must support multiple protocols and orientations, retimer placement is critical at higher speeds, and the combined USB-C power delivery and data channel must be co-designed.

---

## Tier 1: Fundamentals

### Q1. Describe the USB 3.x SuperSpeed physical layer. How does it differ from USB 2.0 at the electrical level?

**Answer:**

USB 3.x (SuperSpeed) is a separate physical layer from USB 2.0 (Low Speed/Full Speed/Hi-Speed), sharing only the connector and cable. The two physical layers coexist on the same cable but use entirely different signalling.

**USB 2.0 physical layer (for comparison):**

- Single-ended signalling: D+ and D− referenced to ground
- Half-duplex (shared bus)
- Maximum speed: 480 Mbps (Hi-Speed)
- Cable impedance: 90 Ω differential
- No scrambling; uses NRZI encoding with bit stuffing

**USB 3.x SuperSpeed physical layer:**

- **Differential signalling:** SSTX+/SSTX- (transmit) and SSRX+/SSRX- (receive) — four wires total for full-duplex operation
- **Full-duplex:** Separate unidirectional TX and RX pairs; simultaneous bidirectional data flow
- **Speeds:**
  - USB 3.2 Gen 1 (formerly USB 3.0): 5 Gbps, 8b/10b encoding → 4 Gbps effective
  - USB 3.2 Gen 2 (formerly USB 3.1 Gen 2): 10 Gbps, 128b/132b encoding → 9.7 Gbps effective
  - USB 3.2 Gen 2×2: 20 Gbps (two lanes, each 10 Gbps)
- **Cable impedance:** 90 Ω differential (±15%)
- **PCB impedance:** 90 Ω differential (typical; some specifications allow 85–95 Ω)
- **Scrambling:** LFSR-based data scrambling for spectral whitening and reduced EMI
- **Equalization:** Tx de-emphasis (2-tap FIR), Rx CTLE in the PHY

**The critical difference:**

USB 3.x uses a dedicated equalization and clock recovery architecture. The 5 Gbps physical layer uses a 5 GHz clock; the 10 Gbps layer uses a 10 GHz clock. Both use embedded clock (no separate clock signal — the receiver CDR recovers the clock from the data stream). This contrasts with source-synchronous interfaces like DDR where a companion clock signal is transmitted.

**Connector pins used:**

In USB Type-A/B connectors, USB 3.x adds four additional pins (SSTX+/-, SSRX+/-) alongside the original USB 2.0 pins. In USB Type-C, USB 3.x is carried on the SuperSpeed TX/RX pins with two sets for cable orientation reversibility.

---

### Q2. What are the PCB routing requirements for USB 3.2 Gen 2 (10 Gbps) traces? State impedance targets, length constraints, and spacing rules.

**Answer:**

USB 3.2 Gen 2 operates at 10 Gbps with a Nyquist frequency of 5 GHz and bit period of 100 ps. The routing requirements balance controlled impedance, signal integrity, and EMI.

**Impedance:**

| Trace configuration | Differential impedance | Single-ended impedance |
|---|---|---|
| PCB trace (differential stripline) | 90 Ω ± 10% | 45 Ω ± 10% |
| PCB trace (differential microstrip) | 90 Ω ± 10% | 45 Ω ± 10% |
| USB cable | 90 Ω ± 15% | — |
| Connector (USB Type-C) | 85–95 Ω | — |

The target is 90 Ω differential to match the cable impedance. A 5 Ω mismatch at the connector or via creates a reflection coefficient of:

$$\Gamma = \frac{Z_L - Z_0}{Z_L + Z_0} = \frac{95 - 90}{95 + 90} \approx 0.027 = -31 \text{ dB}$$

This is acceptable; larger mismatches (> 10 Ω) degrade return loss enough to affect USB 3.2 Gen 2 compliance.

**Length constraints:**

- **Maximum trace length (PCB only, no cable):** Approximately 8–12 inches (200–300 mm) on low-loss laminate at 10 Gbps. On FR4, maximum recommended trace is ~4–6 inches (100–150 mm) before signal integrity margin is consumed.
- **Intra-pair skew:** Maximum ±5 mil (approximately ±0.12 mm, which corresponds to ~0.8 ps of skew at $t_{pd}$ = 6.6 ps/mm). USB 3.2 Gen 2 specifications allow maximum intra-pair skew of 5 ps at the package launch.
- **Inter-pair skew (SSTX vs SSRX):** No strict requirement since TX and RX are independent; CDR handles any length difference between TX and RX paths.

**Spacing rules:**

- Minimum spacing between the TX and RX differential pairs: 3× the trace width (3W rule) to control FEXT. At 10 Gbps (Nyquist 5 GHz), FEXT is significant for spacings < 3W.
- Spacing from USB 3.x SuperSpeed pairs to USB 2.0 D+/D- signals: ≥ 20 mil (to prevent 2.0 signal noise from coupling into the SuperSpeed pair).
- Spacing from other high-speed differential pairs (HDMI, PCIe, SATA): ≥ 20 mil minimum; preferably separate layers or with ground pour isolation.

**Layer assignment:**

Differential stripline (inner layers with adjacent ground planes) is strongly preferred for USB 3.2 Gen 2:
- Lower EMI compared to microstrip (fields contained between ground planes)
- Lower dielectric loss variation (buried microstrip in homogeneous dielectric)
- Better FEXT isolation from adjacent signals on the same layer

**Via guidelines:**

Every via on a USB 3.x SuperSpeed trace introduces a capacitive stub and impedance discontinuity. Minimise via counts. Where vias are unavoidable:
- Use small-diameter vias (8–10 mil drill on a 4–6 mil stackup transition) to minimise capacitance
- Add antipad (clearance opening) in all power and ground planes to reduce via-to-plane capacitance
- For 10 Gbps, consider backdrilling on boards thicker than 1.6 mm
- Route TX and RX pairs through the same via pattern/location to maintain symmetry

---

### Q3. What is a USB retimer, and when is it required in a USB 3.x or USB4 design?

**Answer:**

A USB retimer is an active device inserted into the USB SuperSpeed data path that:

1. Receives the incoming degraded differential signal
2. Recovers the clock from the data stream using an internal CDR
3. Re-times (regenerates) the data to remove accumulated jitter, ISI, and noise
4. Retransmits the signal with a fresh eye opening and low output jitter

A retimer is electrically transparent to the USB protocol — it forwards all data without inspecting or modifying packet content. It is distinct from a USB hub (which processes USB transactions) and from a re-driver (which only amplifies without CDR-based re-timing).

**When a retimer is required:**

1. **Long cable assemblies:** USB4 cables carry 40 Gbps over 2 m passive cables. The cable loss at 20 GHz (Nyquist for 40 Gbps) may exceed the receiver's equalization range. A retimer at the cable end restores signal quality before delivery to the host or device.

2. **Docking station and hub designs:** A docking station routes USB4 from the host upstream port, through the dock PCB, to multiple downstream device ports. The combined PCB trace + connector + PCB trace chain may exceed 30 cm at 20 Gbps, requiring a retimer to restore signal integrity.

3. **Server chassis and rack cabling:** USB Type-C is used in server management interfaces (BMC, debug ports). Traces routed around a server motherboard plus a front-panel cable may require retiming.

4. **USB4 with PCIe tunneling:** When USB4 is tunnelling PCIe Gen 3 (8 GT/s per lane equivalent) at 40 Gbps aggregate, the link budget requirements are tight. Retimers are commonly placed at both the host and device end of a 40 Gbps USB4 link exceeding 50 cm total PCB path length.

**Retimer placement rule of thumb:**

Insert a retimer when the channel insertion loss at the Nyquist frequency exceeds the receiver's CTLE equalization range. For USB 3.2 Gen 2 (10 Gbps), this is typically when $|IL(5\text{ GHz})| > 15$ dB. For USB4 Gen 3 (20 Gbps per lane), this is when $|IL(10\text{ GHz})| > 12$ dB.

**Retimer power and EMI impact:**

A retimer adds approximately 0.1–0.3 W of power per lane and introduces 1–5 ns of additional latency (one CDR lock period). The retimer itself is a switching noise source; its power supply must be carefully decoupled (100 nF + 10 nF at each VCC pin, with inductance-minimising placement).

---

### Q4. Describe the USB Type-C connector from a signal integrity perspective. What are the key challenges it introduces?

**Answer:**

The USB Type-C connector was designed to be:
- **Reversible** (same connector regardless of cable orientation)
- **Multi-protocol** (carries USB 2.0, USB 3.x SuperSpeed, USB4, Thunderbolt, DisplayPort, and power)
- **Compact** (24 pins in a 8.3 mm × 2.5 mm footprint)

**Pin assignment for USB 3.x/USB4 (orientation A):**

| Pin | Signal | Description |
|---|---|---|
| A2 | SSTXp1 | SuperSpeed TX+ (lane 1) |
| A3 | SSTXn1 | SuperSpeed TX- (lane 1) |
| A10 | SSRXp1 | SuperSpeed RX+ (lane 1) |
| A11 | SSRXn1 | SuperSpeed RX- (lane 1) |
| B2 | SSTXp2 | SuperSpeed TX+ (lane 2, for USB 3.2 Gen 2×2 and USB4) |
| B3 | SSTXn2 | SuperSpeed TX- (lane 2) |
| B10 | SSRXp2 | SuperSpeed RX+ (lane 2) |
| B11 | SSRXn2 | SuperSpeed RX- (lane 2) |
| A6/B6 | CC1/CC2 | Configuration Channel (orientation detection, power negotiation) |

**Signal integrity challenges:**

1. **Connector impedance discontinuity:** The USB Type-C receptacle contacts have a nominal impedance of 90 Ω differential, but the actual insertion loss from a production connector at 10 GHz is 1–3 dB. At 20 GHz (USB4), the connector loss increases to 3–6 dB for economy-grade connectors. Connector selection has a major impact on the channel loss budget.

2. **Crosstalk between protocols:** The TX/RX SuperSpeed pairs are routed adjacent to USB 2.0 D+/D- pins in the connector housing. At 5–10 GHz, the adjacent USB 2.0 pins present capacitive loading that degrades SuperSpeed signal quality. Ground pins between the signal groups mitigate this, but the connector's physical pitch limits isolation.

3. **Lane muxing for orientation reversibility:** Because the cable can be inserted in either orientation, the host controller must route either the A-side or B-side SuperSpeed lanes to the PHY based on the cable orientation detected on CC1/CC2. This mux path (often implemented as a CrossBar Mux chip or within the USB Type-C retimer) adds parasitic capacitance and impedance discontinuities to both sets of lanes even when only one set is in use.

4. **Second-tier connector (plug) mating quality:** USB Type-C plug contacts are very small (0.5 mm pitch). Contact resistance and wipe length affect both the 50 Ω single-ended termination and the contact resonance. A low-quality cable plug can contribute 1–2 dB of additional loss at 10 GHz compared to a high-quality cable assembly.

5. **PCB footprint design:** The PCB footprint for the USB Type-C receptacle must maintain the 90 Ω differential impedance through the SMD pads. The large SMD pad area of the SuperSpeed contacts increases parasitic capacitance. Coplanar waveguide structures or RF-optimised footprints can reduce this to under 0.5 dB of additional loss per connector footprint.

---

## Tier 2: Intermediate

### Q5. A USB4 Gen 3×2 system (40 Gbps) uses two lanes at 20 Gbps per lane. The host PCB has 180 mm of trace, a USB Type-C connector, and a 500 mm passive cable. Estimate whether a retimer is needed.

**Answer:**

**USB4 Gen 3 per-lane parameters:**
- Data rate per lane: 20 Gbps
- Encoding: 128b/132b → Nyquist frequency = $20 \times (128/132) / 2 \approx 9.7$ GHz ≈ 10 GHz (Nyquist)
- BER target: $10^{-12}$

**Channel component loss estimate at 10 GHz:**

**1. Host PCB trace (180 mm ≈ 7.1 inches):**

Assuming a mid-range laminate (Dk = 3.8, Df = 0.010 at 10 GHz, typical mid-loss):

Conductor loss at 10 GHz: $\alpha_c \approx 0.5 \times \sqrt{10/1} \approx 1.58$ dB/inch
Dielectric loss at 10 GHz: $\alpha_d \approx 0.8$ dB/inch
Total: $\approx 2.38$ dB/inch

$$IL_{PCB} \approx 7.1 \times 2.38 \approx -16.9 \text{ dB}$$

**2. USB Type-C PCB receptacle connector (host side):**

$$IL_{connector,host} \approx -2.5 \text{ dB}$$

**3. Passive USB4 cable (500 mm ≈ 19.7 inches):**

USB4 passive cables use 32 AWG or 30 AWG conductors. At 10 GHz, a 500 mm passive USB4 cable typically has:

$$IL_{cable} \approx -8 \text{ to } -12 \text{ dB at 10 GHz}$$

Using the USB4 specification's cable model: $IL_{cable} \approx -10$ dB at 10 GHz for a 500 mm cable.

**4. USB Type-C plug (device end connector):**

$$IL_{connector,device} \approx -2.0 \text{ dB}$$

**5. Via transitions (2 per board, backdrilled):**

$$IL_{vias} \approx 2 \times (-0.5) = -1.0 \text{ dB}$$

**Total channel insertion loss:**

$$IL_{total} \approx -16.9 - 2.5 - 10.0 - 2.0 - 1.0 = -32.4 \text{ dB at 10 GHz}$$

**Comparison to receiver equalization capability:**

A typical USB4 Gen 3 receiver CTLE has approximately 15–20 dB of equalization range at 10 GHz. The channel loss of -32.4 dB far exceeds the CTLE range.

**Conclusion: A retimer is required.** In fact, for a 40 Gbps USB4 link with a 500 mm passive cable, two retimers are typically used:
1. One at the host end of the cable (after the host PCB routing)
2. One at the device end (before the device PCB routing)

The retimer at the host end reduces the effective channel seen by the cable to only the cable + device connector + device PCB, which is within the cable-end receiver's equalization range. The retimer at the device end restores the signal before delivery to the device controller.

**Active cable alternative:** For 500 mm and longer, an active USB4 cable assembly with integrated retimers in the cable plugs is an alternative to separate retimer chips on the host PCB. This moves the equalization inside the cable assembly and simplifies the host PCB design.

---

### Q6. Explain the USB 3.x link training sequence and how equalization presets are selected. What is the significance of the Polling.LFPS state?

**Answer:**

USB 3.x link training establishes the operating speed, equalization coefficients, and CDR lock before entering U0 (active link) state. The key training states are:

**Polling.LFPS (Low Frequency Periodic Signalling):**

LFPS is the USB 3.x mechanism for waking up the link and initiating training. LFPS uses a 250 MHz square wave burst (burst duration: 800 ns to 1.5 μs; repeat period: 5 μs to 100 μs) on the SuperSpeed TX pair. This low-frequency signal is detectable even when the SuperSpeed CDR is not yet locked, and it is robust against cable loss at high frequencies.

LFPS is used to:
- Wake the link from a low-power state (U1/U2/U3)
- Begin the Polling sequence for initial link establishment
- Request a speed change (e.g., downgrade from Gen 2 to Gen 1 if training fails)

**Polling.RxEq (Receiver Equalization):**

In this state, the transmitter cycles through all available equalization presets, sending a known training sequence (TS1/TS2 ordered sets) with each preset. The receiver measures the signal quality for each preset using a built-in eye monitor or signal integrity metric and identifies the best preset.

For USB 3.2 Gen 2 (10 Gbps), the presets include:
- No de-emphasis (P0)
- -3.5 dB post-cursor (common for intermediate channels)
- -6 dB post-cursor (common for long traces or cables)
- Combinations of pre-cursor and post-cursor emphasis

The receiver signals its preferred preset back to the transmitter using TS1 ordered sets in the reverse direction.

**Configuration.Idle:**

After equalization negotiation, both sides verify CDR lock by exchanging TS1 ordered sets. If the CDR lock and error rate pass the criteria, the link enters U0 (active).

**Training failure and speed downgrade:**

If training fails at 10 Gbps after multiple attempts (typically 3–5 retries), the link falls back to 5 Gbps (Gen 1) and repeats training. This automatic speed downgrade means a marginal channel may still enumerate as USB 3.2 Gen 1 rather than failing entirely — but at half the intended bandwidth. A system that trains at Gen 1 instead of Gen 2 should trigger a PCB/cable SI investigation.

**Significance of LFPS for SI engineers:**

LFPS compliance requires that the 250 MHz burst signal pass through the channel (including any connector, cable, and retimer) with sufficient amplitude for detection. A channel that passes LFPS detection but fails SuperSpeed training at 10 Gbps indicates a channel with acceptable low-frequency response but excessive high-frequency loss — the classic signature of a high-loss laminate or connector design.

---

### Q7. Compare the USB4 and Thunderbolt 4 physical layer specifications. From a board design perspective, are they interchangeable?

**Answer:**

USB4 is based on the Intel Thunderbolt 3 specification, which Intel contributed to the USB-IF standardisation process. The result is a close but not identical set of specifications.

**Key comparison:**

| Parameter | USB4 Gen 3×2 | Thunderbolt 4 |
|---|---|---|
| Max speed | 40 Gbps (2 × 20 Gbps) | 40 Gbps (2 × 20 Gbps) |
| USB4 Gen 2×2 (20 Gbps) | Mandatory support | Optional (Thunderbolt 4 mandates 40 Gbps) |
| PCIe tunneling | Optional (PCIe Gen 2 minimum) | Mandatory (PCIe Gen 3 at 32 Gbps) |
| DisplayPort tunneling | Optional | Mandatory (DisplayPort 2.0) |
| Power delivery | USB PD 3.0 (up to 100 W) | Minimum 15 W host-to-device; 100 W when connected to power adapter |
| Daisy-chaining | Not specified | Up to 6 devices |
| Intel certification | Not required | Required (Intel certification and silicon) |
| Cable requirement | Passive to 500 mm (Gen 2), active for Gen 3 at 500 mm | Active cable required for 40 Gbps at ≥ 500 mm |

**Physical layer equivalence:**

At the physical layer (signals on the USB Type-C connector), USB4 Gen 3×2 and Thunderbolt 4 use identical electrical specifications: 90 Ω differential, same loss budget, same equalization requirements. A PCB designed and validated for USB4 Gen 3×2 is electrically compatible with Thunderbolt 4 if the host controller implements Thunderbolt 4 firmware.

**Board design implications:**

1. **Host controller selection:** Thunderbolt 4 requires an Intel-certified host controller (currently only Intel's own JHL8540/JHL8040 or Maple Ridge/Goshen Ridge silicon). USB4 Gen 3 can be implemented with third-party PHYs (e.g., Realtek, ASMedia, Renesas). The PHY pinout and PCB escape routing differ between vendors.

2. **Retimer requirements:** Thunderbolt 4 certification requires retimers to be Thunderbolt certified as well. Not all USB4 retimers are Thunderbolt 4 certified. Using an uncertified retimer may pass USB4 validation but fail Thunderbolt 4 certification.

3. **Protocol mux complexity:** A USB4/Thunderbolt 4 host requires a mux that can route either USB3 SuperSpeed, USB4, or Thunderbolt 4 lanes to the USB Type-C connector. This mux (CrossBar Mux or integrated in the retimer) adds 1–3 dB of loss and is a major signal integrity element that must be characterized.

**Practical answer:** For a board that must support both USB4 and Thunderbolt 4, design and validate to the Thunderbolt 4 physical layer requirements (tighter) and use a Thunderbolt-certified retimer. USB4-only designs can use less expensive components, but the routing is identical.

---

## Tier 3: Advanced

### Q8. A USB 3.2 Gen 2 design fails enumeration intermittently. The SSTX and SSRX pairs pass impedance testing (88–92 Ω differential). The failure rate is 1-in-20 insertions. Describe a systematic debug approach.

**Answer:**

A 1-in-20 insertion failure rate with nominal impedance measurements suggests the fault is not a systematic loss or impedance mismatch, but a marginal condition that depends on variables that change between insertions: contact alignment, cable orientation, or mechanical variation.

**Step 1 — Characterise the failure mode precisely.**

- Does the failure manifest as no-device-detected (no Vbus detection or LFPS), or as speed-downgrade (enumerates at USB 3.2 Gen 1 instead of Gen 2)?
- Is the failure orientation-dependent? Test with both cable orientations (if Type-C). If one orientation fails more frequently, the fault is in the lane mux path for one orientation.
- Does the failure reproduce with a different cable? If yes, the fault is on the PCB. If no, the fault is cable-related.

**Step 2 — Inspect physical connector integrity.**

A 1-in-20 insertion failure pattern is characteristic of:

- **Bent or damaged contact in the USB Type-C receptacle:** Inspect the receptacle under magnification. A bent contact causes intermittent connection or high contact resistance.
- **Solder joint quality on the receptacle pins:** Perform X-ray inspection on the SuperSpeed contact pins (A2, A3, A10, A11 and B2, B3, B10, B11). Cold solder joints on high-pitch SMD contacts are a common failure mode.
- **Insufficient wiping length:** The USB Type-C contact wipe specification requires a minimum contact engagement length. Mechanical tolerance stack-up in the housing can cause marginal wipe in some insertions.

**Step 3 — Measure LFPS on failure insertions.**

Using a real-time oscilloscope, capture the LFPS waveform on a failed enumeration:

- **No LFPS:** The failure is at the physical connection level (contact resistance or connector mechanical fault). The SuperSpeed PHY cannot generate LFPS if VCC is not delivered to the contacts.
- **LFPS present but no training progress:** The LFPS is passing but TS1/TS2 ordered sets are failing. This points to a marginal high-frequency path (not a connector mechanical problem).

**Step 4 — Measure SSTX output at the connector pins.**

With the link attempting to enumerate, capture the SSTX± waveform at the USB Type-C receptacle pads (not at the host IC pin). Look for:

- **Amplitude variation:** If the voltage swing varies by > 20% between insertions, there is a contact resistance problem.
- **Common-mode noise injection:** Asymmetric contact resistance on the +/- contacts converts differential signal to common-mode and back. This is visible as mode conversion in S-parameter measurements.

**Step 5 — Measure S-parameters in failure configuration.**

If the failure can be reproduced with a cable fixture, measure S21 and S11 of the SSTX path in the failing insertion state. Compare to a passing insertion. A difference of > 2 dB at 5 GHz in the failing state confirms a contact-related cause.

**Step 6 — Root cause and corrective actions:**

| Root Cause | Evidence | Fix |
|---|---|---|
| Damaged USB Type-C receptacle contacts | Physical inspection, intermittent contact resistance | Replace receptacle; add mechanical protection (cover, guard ring) |
| Marginal solder joints on receptacle | X-ray, occasional open/high-R on fail | Rework solder profile; add paste volume or use press-fit connector |
| Lane mux chip marginal for one cable orientation | Orientation-specific failure | Replace mux chip; check mux manufacturer's EQ settings for that channel |
| Cable quality variation | Failure with some cables, not others | Specify higher-quality cable; add retimer if cable loss is marginal |
| Board-level signal integrity just under COM limit | COM simulation shows 0–1 dB margin | Increase board-level trace quality; add retimer chip |

---

### Q9. Design the PCB routing strategy for a USB4 Gen 3×2 (40 Gbps) host controller with an integrated retimer, connecting to a USB Type-C receptacle. Specify layer assignment, impedance, spacing, and via treatment.

**Answer:**

**Design context:**

The path from host controller to USB Type-C connector must achieve < 12 dB insertion loss at 10 GHz (Nyquist for 20 Gbps per lane) for each of the two lanes, including the retimer's insertion. The retimer is on the PCB between the host controller and the connector.

**Physical topology:**

```
Host SoC ─── [PCB trace A] ─── [Retimer] ─── [PCB trace B] ─── [USB Type-C receptacle]
```

Each section (A and B) is a high-speed differential channel that must be individually designed.

**Section A — Host SoC to Retimer:**

Typical length: 20–50 mm (BGA escape + routing to retimer).

- **Layer assignment:** Inner stripline layer, with dedicated reference planes above and below. Avoid outer microstrip layers (higher radiation, more susceptible to surface contamination effects at 10 GHz).
- **Impedance:** 90 Ω differential ±10%, verified with field solver. For a 4-layer board with 0.2 mm dielectric height, a trace width of 0.10 mm with 0.10 mm gap gives approximately 90 Ω differential.
- **Intra-pair skew:** Length-match TX+ and TX- within ±2 mm (±13 ps maximum, though tighter is better).
- **Via treatment:** Minimize via count. Where BGA escape requires vias, use 0.2 mm drill, 0.35 mm pad, 0.5 mm antipad, and backdrill if board thickness > 1.0 mm.

**Section B — Retimer to USB Type-C receptacle:**

Typical length: 10–30 mm (retimer to connector).

- **Layer assignment:** Maintain same inner layer as Section A if possible, to avoid additional layer transitions.
- **Impedance:** Same 90 Ω target. The USB Type-C receptacle footprint pads are wider (0.3–0.4 mm for SMD pads), which increases capacitance and drops impedance. Taper the trace width from 0.10 mm up to the pad width to reduce the abrupt impedance step.
- **USB 2.0 D+/D- isolation:** Keep D+/D- (which run adjacent to SuperSpeed pins in the Type-C connector pin array) at least 20 mil from any SuperSpeed trace. Use a ground fill between them if space permits.
- **CC1/CC2 routing:** The CC1/CC2 lines carry the orientation detection signal. Route them on the same or adjacent layer with enough spacing from SuperSpeed lanes to prevent crosstalk from 5 V PD negotiation signals.

**Lane mux placement (if not integrated in retimer):**

If a separate CrossBar Mux chip is used for orientation reversibility:
- Place it between the connector and the retimer, as close to the connector as possible (to shorten the non-retimed path).
- The mux introduces 1–2 dB of insertion loss; include in the loss budget.
- The mux's power supply (typically 1.8 V) requires 100 nF + 10 nF decoupling within 1 mm of each VCC pin.

**Retimer PCB placement:**

- Place the retimer as close to the host SoC as practical (not at the connector end), so that the retimer's equalized output drives the shorter Section B trace to the connector.
- Retimer VCC pins: 100 nF + 10 nF per pin, within 1 mm of pin.
- Retimer reference clock: Many USB4 retimers require a 19.2 MHz or 24 MHz reference clock. Route this as a matched-impedance trace (50 Ω single-ended) with series termination at the driver. Keep it away from SuperSpeed pairs (> 30 mil spacing).

**Stackup recommendation for a 6-layer board:**

```
Layer 1 (top):    Power planes, connector landing pads (SMD)
Layer 2:          USB4 SuperSpeed differential pairs (stripline)
Layer 3:          Ground plane (reference for Layer 2)
Layer 4:          Ground plane (reference for Layer 5)
Layer 5:          Signal routing, clocks, control lines
Layer 6 (bottom): Power delivery, USB 2.0 lines
```

The USB4 SuperSpeed pairs are sandwiched between ground planes on Layers 2 and 3, minimising EMI and crosstalk.

**Loss budget verification:**

| Component | Estimated loss at 10 GHz |
|---|---|
| Section A trace (40 mm, low-loss laminate) | -2.5 dB |
| BGA escape vias (2 × backdrilled) | -1.0 dB |
| Retimer insertion | -1.5 dB |
| Section B trace (25 mm, low-loss laminate) | -1.5 dB |
| Via to connector layer (1 × backdrilled) | -0.5 dB |
| USB Type-C receptacle | -2.0 dB |
| **Total** | **-9.0 dB** |

This is within the 12 dB budget, leaving 3 dB of margin for manufacturing variation and laminate property variation with temperature.

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| SuperSpeed | Marketing name for USB 3.x physical layer (5/10/20 Gbps); contrasted with USB 2.0 Hi-Speed (480 Mbps) |
| SSTX / SSRX | SuperSpeed Transmit / Receive differential pairs in USB 3.x |
| LFPS | Low Frequency Periodic Signalling; 250 MHz burst used for wake-up and link initiation in USB 3.x |
| Retimer | Active signal regeneration device with CDR; re-times data to remove jitter and extend reach |
| Re-driver | Passive or linear amplifier that boosts signal without CDR; no jitter cleanup |
| Lane mux / CrossBar | Chip that routes two physical SuperSpeed lane sets to a single controller based on cable orientation |
| USB4 Gen 3×2 | USB4 at 20 Gbps per lane × 2 lanes = 40 Gbps aggregate; identical physical layer to Thunderbolt 4 |
| Thunderbolt 4 | Intel-certified protocol using same physical layer as USB4 Gen 3; mandates PCIe and DisplayPort tunnelling |
| 128b/132b | USB 3.2 Gen 2 line encoding; 4 sync bits per 132-bit block → 3% overhead |
| TS1 / TS2 | Training Sequences 1 and 2; ordered sets used during link training to negotiate equalization and speed |
| CDR | Clock and Data Recovery; PLL-based circuit in receiver that extracts the clock from the incoming data stream |
| U0/U1/U2/U3 | USB 3.x power states: U0 = active, U1/U2 = low latency power save, U3 = suspended |
| CC1/CC2 | Configuration Channel pins in USB Type-C; used for orientation detection and USB PD power negotiation |
| USB PD | USB Power Delivery; negotiation protocol for voltages up to 48 V and currents up to 5 A (240 W max) |
| Passive cable | USB4 cable with no active components; limited to USB4 Gen 2×2 (20 Gbps) at lengths > 500 mm |
| Active cable | USB4 cable with integrated retimers in the plugs; supports 40 Gbps at cable lengths up to 2 m |
