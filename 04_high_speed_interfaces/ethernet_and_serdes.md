# Ethernet PHY and SerDes Architecture

## Overview

Ethernet has evolved from 10 Mbps coaxial bus to 400 Gbps and beyond on optical fiber and copper backplanes. The physical layer encoding, SerDes architecture, and reference clocking scheme are tightly coupled: together they determine achievable bandwidth, link latency, power consumption, and the maximum channel loss budget that the system can tolerate.

This document covers the SerDes (Serializer/Deserializer) architecture from PLL to slicer, the reference clocking topologies used in data-centre and telecom equipment, and the specific physical layer requirements for 10GbE, 25GbE, and 100GbE over copper backplane and direct-attach copper (DAC) cables. Understanding these topics is essential for hardware engineers at companies building network ASICs, switch silicon, data centre servers, and high-performance computing interconnect.

---

## Tier 1: Fundamentals

### Q1. What is a SerDes, and what are its primary functional blocks?

**Answer:**

A SerDes (Serializer/Deserializer) is the silicon circuit that converts between the wide parallel data bus inside a chip and the narrow (typically 1-bit) high-speed serial stream on the physical channel. Every high-speed serial interface — PCIe, Ethernet, USB3, SATA, Fibre Channel — is built around SerDes blocks.

**Transmit path (Serializer):**

1. **Parallel-to-serial shift register:** Accepts $N$-bit parallel data from the digital logic core and outputs one bit per high-speed clock cycle. For a 25 Gbps SerDes with a 64-bit parallel bus, the serialization ratio is 64:1; the parallel clock runs at $25\text{ Gbps} / 64 = 390.6\text{ MHz}$.

2. **Transmitter PLL (TX PLL):** Multiplies the reference clock (e.g., 156.25 MHz) up to the full line rate. For 25 GbE: $156.25 \text{ MHz} \times 160 = 25 \text{ GHz}$. The TX PLL must achieve low phase noise to minimise transmitter jitter contribution.

3. **Transmitter equalizer (Tx EQ):** An FIR pre-emphasis filter that applies de-emphasis (typically 2–4 post-cursor taps plus 1 pre-cursor tap) to compensate for channel high-frequency loss.

4. **Output driver:** A differential current-mode driver (CML — Current-Mode Logic) that delivers the signal to the channel. CML drivers typically provide 400–1000 mV differential peak-to-peak swing with programmable amplitude.

**Receive path (Deserializer):**

1. **Continuous-Time Linear Equalizer (CTLE):** A high-pass analog filter that compensates the channel's low-pass roll-off before the slicer.

2. **Decision Feedback Equalizer (DFE):** A feedback digital filter that cancels post-cursor ISI by subtracting previously decided symbol values multiplied by channel tap weights.

3. **Slicer:** A high-speed comparator that makes a binary (NRZ) or ternary/quaternary (PAM4) decision on each incoming symbol.

4. **Clock and Data Recovery (CDR):** Extracts the transmitter's embedded clock from the received data transitions, aligning the sampling clock to the eye centre. CDR implementations range from linear (bang-bang) PLLs to sophisticated digital CDR with fractional-N correction.

5. **Serial-to-parallel shift register:** Widens the recovered bit stream back to the parallel bus width.

**Key performance parameters:**

| Parameter | Typical Value (25 Gbps NRZ) |
|---|---|
| Jitter (RJ, rms) | < 0.5 ps rms (TX output) |
| Jitter (TJ, pk-pk at $10^{-12}$) | < 0.3 UI = 12 ps |
| CTLE range | 8–20 dB at 12.5 GHz |
| DFE tap count | 3–8 post-cursor taps |
| Reference clock frequency | 156.25 MHz (25GbE), 161.1328125 MHz (100GbE FlexE) |
| Lock time (CDR) | 1–50 μs depending on CDR type |

---

### Q2. What is the reference clock architecture for 10GbE, 25GbE, and 100GbE? Why is clock frequency and distribution critical for Ethernet?

**Answer:**

Ethernet physical layers use a specific reference clock frequency dictated by the line rate and encoding:

**10GbE (10GBASE-R):**

- Line rate: 10.3125 Gbps (after 64b/66b overhead: $10 \times 10.3125/10 = 10.3125$)
- Reference clock: 156.25 MHz
- PLL multiplication: $156.25 \text{ MHz} \times 66 = 10312.5 \text{ MHz} = 10.3125 \text{ Gbps}$

**25GbE (25GBASE-R, IEEE 802.3by):**

- Line rate: 25.78125 Gbps ($25 \times 64/66 \times 66/64 = 25.78125$)
- Reference clock: 156.25 MHz
- PLL multiplication: $156.25 \text{ MHz} \times 165 = 25781.25 \text{ MHz} = 25.78125 \text{ Gbps}$

**100GbE (100GBASE-R4, four lanes each at 25.78125 Gbps):**

- Same reference clock as 25GbE: 156.25 MHz
- Four SerDes lanes, each at 25.78125 Gbps

**400GbE (400GBASE-R8, eight lanes at 53.125 Gbps PAM4):**

- Reference clock: 156.25 MHz
- Each lane: PAM4 at 26.5625 Gbaud → effective bit rate 53.125 Gbps

**Why reference clock quality is critical:**

The SerDes PLL multiplies the reference clock by a large integer (165× for 25GbE). Phase noise on the reference clock is multiplied by the same factor:

$$\mathcal{L}_{VCO}(f) = \mathcal{L}_{REF}(f) + 20\log_{10}(N)$$

For $N = 165$: $20\log_{10}(165) \approx 44$ dB of phase noise multiplication. A reference oscillator with -130 dBc/Hz at 10 kHz offset produces a VCO phase noise of approximately $-130 + 44 = -86$ dBc/Hz at 10 kHz offset from the 25 GHz carrier.

This translates directly to integrated jitter. For 25GbE, the IEEE 802.3by specification requires total reference clock jitter < 1 ps peak-to-peak (10 Hz–1.5 MHz bandwidth). This requires a TCXO or crystal oscillator (not a silicon-based spread-spectrum clock) with specified close-in phase noise.

**Reference clock distribution:**

On a multi-lane ASIC or switch chip, a single 156.25 MHz reference clock is distributed to all SerDes banks. The clock distribution tree must maintain phase alignment between SerDes blocks to within 50 ps (for 25GbE) to avoid inter-lane skew on multi-lane interfaces (100GbE, 400GbE).

**Common mistake:** Using a spread-spectrum clock (SSC) for the SerDes reference. SSC intentionally modulates the reference clock frequency by ±0.5% at 30–33 kHz to spread EMI. However, this modulation creates significant extra jitter at the CDR tracking bandwidth edge. IEEE 802.3 Ethernet specifications do not permit SSC on the reference clock for 10GbE and above.

---

### Q3. Explain 64b/66b encoding used in 10GbE and above. Why was it chosen over 8b/10b?

**Answer:**

**8b/10b (used in 1GbE, PCIe Gen 1/2, USB 3.2 Gen 1):**

8b/10b maps each 8-bit data byte to a 10-bit code word from a predefined table. The encoding guarantees:
- DC balance: each code word has equal or nearly equal numbers of 1s and 0s
- Transition density: at least 2 transitions per 10-bit period, ensuring CDR functionality
- Special control characters (K-codes) for in-band signalling (Comma, SOF, EOF, Idle)

The overhead is $\frac{10-8}{10} = 20\%$ — meaning 20% of link bandwidth is consumed by encoding alone.

**64b/66b (used in 10GbE, 25GbE, 100GbE):**

64b/66b maps 64 bits of data plus a 2-bit synchronisation header into a 66-bit code word:

```
[2-bit sync header][64-bit payload]
  01 = data block
  10 = control/ordered-set block
```

Properties:
- **Overhead:** $\frac{66-64}{66} = 3.03\%$ — drastically lower than 8b/10b
- **DC balance and transition density:** Maintained through LFSR-based scrambling of the 64-bit payload (the sync header is not scrambled). The scrambler ensures the data stream appears pseudo-random with adequate transitions for CDR.
- **Block synchronisation:** The CDR synchronises to the 2-bit header pattern by locking to the position where alternating 01/10 headers appear. Comma-alignment is replaced by this block-boundary detection.

**Why 64b/66b is preferred at 10 Gbps and above:**

The 20% overhead of 8b/10b means a 10 Gbps physical link delivers only 8 Gbps of user data. To achieve 10 Gbps user throughput with 8b/10b would require a 12.5 Gbps line rate, which increases power consumption, requires higher-bandwidth channel, and was not practical with 2000s-era CMOS technology. 64b/66b's 3% overhead allows a 10 Gbps user data rate with only a 10.3125 Gbps line rate — a much more achievable target.

**128b/130b (PCIe Gen 3+, USB 3.2 Gen 2):**

128b/130b extends the principle further: 128 bits of data plus a 2-bit sync header in 130 bits → $\frac{2}{130} = 1.54\%$ overhead. Ethernet standardized on 64b/66b early and has retained it for consistency with existing framing.

---

### Q4. What is an Ethernet PHY and how does it relate to the SerDes in an ASIC?

**Answer:**

In Ethernet, the Physical Layer (PHY) specification is divided into sub-layers defined by IEEE 802.3:

```
MAC layer (Media Access Control)
     |
PCS (Physical Coding Sublayer)   ← 64b/66b encoding, lane distribution
     |
PMA (Physical Medium Attachment) ← SerDes TX/RX, CDR, equalization
     |
PMD (Physical Medium Dependent)  ← Specific to medium: copper, optical fiber, backplane
     |
Physical medium (cable, PCB trace, fiber)
```

**The SerDes is the PMA sublayer.** The PCS layer above it handles 64b/66b encoding/decoding, scrambling, and (for multi-lane interfaces) the AM (Alignment Marker) based lane deskew. The PCS feeds the serialized bit stream to the PMA's serializer; the PMA's deserializer feeds the recovered bits back to the PCS for decoding.

**Internal PHY (integrated) vs external PHY chip:**

Many data-centre switch ASICs integrate the SerDes (PMA) on-die but use an external retimer or gearbox chip at the port. This is because:

1. The switch ASIC's SerDes is optimised for chip-to-chip distances (10–30 cm backplane)
2. An external PHY chip provides additional equalization range and handles optical module driving (for SFP+, QSFP28 interfaces)
3. External PHY chips often include KR4 auto-negotiation, forward error correction (clause 74 RS-FEC), and diagnostic eye monitor capabilities not integrated in the switch ASIC's SerDes

For direct-attach copper (DAC) cables, the switch ASIC's SerDes may connect directly to the DAC cable (no external PHY), relying on the SerDes's own equalization to handle the cable's 5–8 dB/m loss.

---

## Tier 2: Intermediate

### Q5. A 25GBASE-KR (25 Gbps backplane Ethernet) link fails to achieve auto-negotiation. The transmitter's output eye measures a good opening at the near end. Describe the systematic debug approach.

**Answer:**

25GBASE-KR (KR = backplane, 1 lane) uses 25.78125 Gbps NRZ over a PCB backplane trace. Auto-negotiation (AN) and link training are defined in IEEE 802.3by Clause 72.

**Step 1 — Verify LFPS/AN link pulses at the far end.**

KR auto-negotiation uses a base-page exchange at 2.5 Gbps (10× lower than the full data rate) over the same physical pair. If AN is failing:

- Probe the transmitter output at the far-end (receiver) connector.
- Verify that the AN link pulses arrive with amplitude > -5 dBm into 50 Ω.
- If no AN pulses arrive at the far end: check trace continuity, via connectivity, and connector engagement.

**Step 2 — Check the channel loss at the training pattern frequency.**

KR link training uses a $\pm$ PRBS-11 pattern at full line rate. If auto-negotiation passes but link training fails (link does not progress beyond TRAIN state):

- Measure $|S_{21}|$ at 12.89 GHz (Nyquist for 25.78 Gbps).
- 25GBASE-KR specification defines a maximum channel insertion loss of -28.1 dB at 12.89 GHz (Clause 93, Annex 93A channel model).
- If $|S_{21}| < -28.1$ dB: the channel is out of spec. Options are to shorten the trace, upgrade laminate, or add a retimer.

**Step 3 — Examine link training coefficient exchange.**

KR link training (Clause 72) uses a coefficient update protocol where each end requests changes to the remote transmitter's de-emphasis coefficients:

- Request HOLD, INCREMENT, DECREMENT for c(-1) (pre-cursor), c(0) (main cursor), c(+1) (post-cursor)
- The remote transmitter applies each change and the local receiver evaluates the eye quality
- Training terminates when both ends signal STATUS_REQUEST

If training never converges:
- Read the link training debug registers from the PHY (if accessible via MDIO).
- Identify which coefficient is stuck at maximum or minimum: a coefficient that can never be satisfied indicates the channel loss is outside the training range.
- Check whether the pre-cursor or post-cursor direction of the channel response is unusual (e.g., a channel with very large pre-cursor reflection is rare and may confuse the training algorithm).

**Step 4 — Differential-mode return loss.**

A channel with high $|S_{11}|$ (poor return loss) can fail link training even with acceptable $|S_{21}|$. Reflections cause constructive interference that creates ripple in the frequency response, which makes the channel look band-limited at certain frequencies. Measure $|S_{11}|$ over 1–15 GHz. A peak $|S_{11}| > -5$ dB at any frequency below Nyquist warrants investigation.

Common return loss culprits: non-backdrilled vias, connector impedance mismatch, trace width transitions, or a DUT input impedance that is not well-matched to 100 Ω differential.

**Step 5 — Mode conversion (differential to common mode).**

Backplane designs with impedance asymmetry between the + and - conductors of a differential pair convert differential signal to common mode. The mode conversion (Sdc21) degrades the effective differential signal amplitude and can cause CDR lock issues. Use a 4-port VNA to measure $|S_{dc21}|$ (differential-in, common-mode-out at far end). A large $|S_{dc21}|$ indicates routing asymmetry or an unbalanced connector.

---

### Q6. Describe the IEEE 802.3cd / 802.3bs 100GbE and 400GbE PAM4 physical layer architecture. How does PAM4 change the SerDes design requirements?

**Answer:**

**100GbE PAM4 (100GBASE-CR4, 100GBASE-DR, 100GBASE-FR):**

IEEE 802.3cd defines 100GbE using 2 lanes at 53.125 Gbps PAM4 per lane. Each lane:

- Symbol rate: 26.5625 Gbaud
- Encoding: 2 bits/symbol (PAM4)
- Line rate equivalent: 53.125 Gbps per lane
- FEC: Clause 91 RS(544,514) or Clause 108 RS(272,258) mandatory

**400GbE PAM4 (400GBASE-CR8, 400GBASE-DR4):**

8 lanes at 53.125 Gbps PAM4. Same per-lane architecture as 100GbE PAM4 but 8× the lane count.

**How PAM4 changes SerDes design:**

**1. Four-level output driver:**

The TX output driver must generate four distinct voltage levels precisely. Non-linearity in the driver causes unequal eye openings:
- If the upper two levels (1→2, 2→3) have larger amplitude than the lower levels (0→1), the three eyes are unequal in height.
- IEEE 802.3 specifies a maximum level mismatch of ±1 dB (eye height uniformity).
- The driver must use a calibrated DAC output, not a simple differential pair.

**2. Three-level slicer:**

The PAM4 receiver needs two slicers (or three threshold comparators) to distinguish four levels. The three thresholds must be precisely aligned to the centres of the three eye openings. A threshold offset of $\delta$ from the ideal value degrades the BER by:

$$\Delta BER \approx \frac{\delta}{\sigma_{noise} \cdot V_{eye}} \times 10^{-SNR/20}$$

Threshold calibration is performed during startup using a built-in test pattern (PRBS-13Q for PAM4).

**3. More complex equalizer:**

The DFE for PAM4 must handle inter-symbol interference with four amplitude levels. The tap weights are the same as for NRZ DFE (they cancel the impulse response tail), but the DFE must operate with 3 independent threshold levels. The DFE summer must handle 4 distinct input levels plus the multi-tap feedback accurately.

**4. Tight reference voltage requirement:**

The PAM4 receiver's three slicer thresholds are derived from an analog reference. Any noise on the reference voltage propagates directly into threshold uncertainty. Typical PAM4 receivers require reference noise < 5 mV rms to meet BER targets, compared to ~15 mV for NRZ.

**5. CDR complexity:**

PAM4 CDR must track to symbol boundaries that are defined by transitions between any of 16 possible symbol pairs (4 × 4). The CDR bang-bang phase detector is modified for PAM4: the error signal is proportional to the distance between the received sample and the ideal level, rather than simply whether the sample is above or below a single threshold.

**6. Mandatory FEC:**

NRZ 25GbE can operate without FEC on short copper channels. PAM4 at 53 Gbps mandates FEC (Clause 91 RS(544,514)) because the raw BER without FEC is $~10^{-4}$ to $10^{-6}$, far above the required post-FEC BER of $10^{-15}$.

---

### Q7. A 25GbE SerDes design uses a fractional-N PLL for the VCO. Explain the jitter implications and the advantage over an integer-N PLL for this application.

**Answer:**

**Integer-N PLL:**

An integer-N PLL multiplies the reference frequency by a fixed integer $N$:

$$f_{VCO} = N \times f_{REF}$$

For 25.78125 Gbps (25GbE) with a 156.25 MHz reference: $N = 165$ (integer). This works exactly for 25GbE.

For 10.3125 Gbps with 156.25 MHz reference: $N = 66$ (integer). Also exact.

Integer-N PLLs are simple and have low-noise VCO design because there is no fractional divide in the feedback path. The dominant jitter sources are the reference clock phase noise (multiplied by $N$), the PLL charge pump noise, and the VCO free-running noise.

**Fractional-N PLL:**

A fractional-N PLL achieves a non-integer multiplication ratio by rapidly alternating the divide ratio between $N$ and $N+1$ (or higher integers) using a delta-sigma modulator. The average division ratio is fractional:

$$f_{VCO} = (N + \Delta) \times f_{REF}, \quad 0 \le \Delta < 1$$

For example, a VCO at 12.890625 GHz (Nyquist for 25.78125 Gbps) with a 25 MHz reference: $N + \Delta = 12890625/25000 = 515.625$. An integer-N PLL would require a 25 MHz reference to produce exactly this frequency, but a fractional-N PLL can achieve it with a 25 MHz reference and $\Delta = 0.625$.

**Jitter implications:**

Fractional-N PLLs introduce **fractional spurs** — periodic phase modulation at the delta-sigma modulator's operating frequency. These spurs appear as deterministic jitter at specific frequencies:

$$f_{spur} = \Delta \times f_{REF} = 0.625 \times 25 \text{ MHz} = 15.625 \text{ MHz}$$

This deterministic jitter component is bounded (BDJ) rather than unbounded Gaussian (RJ). For Ethernet, a fractional spur at 15.625 MHz with amplitude 0.05 UI = 2 ps would contribute ~4 ps peak-to-peak DJ to the total jitter budget, consuming a significant fraction of the IEEE 802.3by allotment.

**Advantage of fractional-N for Ethernet SerDes:**

The primary advantage is **reference clock flexibility**. Modern system-on-chips support many protocols (PCIe, SATA, USB, Ethernet) with different reference clock requirements. A fractional-N PLL allows all SerDes to share a single reference clock (e.g., 100 MHz) rather than requiring a separate 156.25 MHz oscillator for Ethernet.

The disadvantage (fractional spurs) is mitigated by:
1. High-order delta-sigma modulators that spread spur energy across a wider bandwidth
2. PLL bandwidth selection that attenuates fractional spurs before they reach the VCO
3. Careful selection of the fractional ratio to place spurs outside the CDR tracking bandwidth

**For 25GbE with a 156.25 MHz reference, an integer-N PLL (N=165) is the optimal choice** — no fractional spurs, lowest jitter. The fractional-N approach is preferred when the reference clock cannot be set to 156.25 MHz.

---

## Tier 3: Advanced

### Q8. Derive the COM-equivalent metric for a 25GBASE-KR backplane channel. Given the following parameters, determine whether the channel passes: $|S_{21}| = -18$ dB at 12.89 GHz, CTLE range = 15 dB, DFE = 5 taps, integrated FEXT = -35 dBV, RJ = 0.3 ps rms, and TX amplitude = 500 mV differential peak.

**Answer:**

This problem uses a simplified COM-like approach rather than the full MATLAB COM script, but the method captures the key factors.

**Step 1 — Channel-equalized signal amplitude at slicer input.**

The transmitter launches a signal with amplitude $A_{TX} = 500/2 = 250$ mV (single-sided, since differential peak-to-peak = 500 mV means ±250 mV).

After the channel with $|S_{21}| = -18$ dB at 12.89 GHz, the signal is attenuated. However, the CTLE then boosts it back. Assuming the CTLE is set to fully compensate the channel loss at Nyquist (15 dB of CTLE boost compensates 15 dB of the 18 dB channel loss), there is 3 dB of residual attenuation at Nyquist:

$$A_{after-CTLE} = 250 \text{ mV} \times 10^{-3/20} \approx 250 \times 0.708 \approx 177 \text{ mV}$$

The DFE cancels post-cursor ISI after the CTLE. With 5 DFE taps well-trained, the residual ISI at the slicer input is typically < 10 mV rms for a well-behaved channel.

**Step 2 — Noise contributions at the slicer.**

**RJ from jitter:**

Timing jitter of $\sigma_J = 0.3$ ps rms translates to amplitude uncertainty via the eye slope:

$$\sigma_{amp,jitter} = \sigma_J \times \frac{dV}{dt}\bigg|_{transition}$$

The transition slew rate for a 25 GHz channel with 40 ps rise time (10-90%):

$$\frac{dV}{dt} = \frac{0.8 \times 354 \text{ mV}}{40 \times 10^{-12}} \approx 7.08 \text{ V/ns}$$

$$\sigma_{amp,jitter} = 0.3 \times 10^{-12} \times 7.08 \times 10^9 = 2.1 \text{ mV rms}$$

**FEXT noise:**

Integrated FEXT = -35 dBV → $V_{FEXT} = 10^{-35/20} = 17.8$ mV peak. Assuming FEXT is approximately Gaussian distributed at the slicer (multiple aggressor superposition), $\sigma_{FEXT} \approx V_{FEXT} / 3 \approx 5.9$ mV rms.

**Receiver thermal noise:**

A typical 25GbE receiver has an equivalent input noise of -60 dBm in a 12.89 GHz bandwidth = $10^{-6}$ W into 50 Ω → $V_{noise} = \sqrt{10^{-6} \times 50} = 7.07$ mV rms.

**Total noise (RSS):**

$$\sigma_{total} = \sqrt{\sigma_{jitter}^2 + \sigma_{FEXT}^2 + \sigma_{rx}^2} = \sqrt{2.1^2 + 5.9^2 + 7.07^2} \approx \sqrt{4.4 + 34.8 + 50.0} \approx \sqrt{89.2} \approx 9.4 \text{ mV rms}$$

**Step 3 — Compute SNR and COM equivalent.**

$$SNR = 20\log_{10}\left(\frac{A_{after-CTLE}}{\sigma_{total}}\right) = 20\log_{10}\left(\frac{177}{9.4}\right) = 20\log_{10}(18.8) \approx 25.5 \text{ dB}$$

**Step 4 — Compare to minimum required SNR.**

For NRZ at $BER = 10^{-15}$, the required $Q$-factor is approximately 8.0 ($20\log_{10}(8.0) = 18.1$ dB).

$$COM_{equivalent} = SNR - SNR_{required} = 25.5 - 18.1 = +7.4 \text{ dB}$$

**Conclusion:** COM equivalent ≈ **+7.4 dB**, which is well above the 0 dB threshold. This channel passes.

**Important caveat:** This simplified analysis ignores several effects that the full COM script includes (non-stationary noise, channel impedance variation with frequency, higher-order ISI from the channel pulse response, transmitter jitter spectrum shaping). The actual COM value from a full MATLAB simulation may differ by ±2 dB, but the channel is unlikely to fail given a +7.4 dB first-order margin.

---

### Q9. A 100GbE PAM4 system using 100GBASE-DR (1 lane, 106.25 Gbps) over single-mode optical fiber is showing elevated pre-FEC BER (measured BER = $5 \times 10^{-5}$, specification requires $\leq 2.4 \times 10^{-4}$ pre-FEC for RS(544,514)). After FEC, the link appears clean. An SI engineer notices the eye diagram shows unequal eye heights. Describe the investigation and corrective action.

**Answer:**

**Initial characterisation:**

100GBASE-DR uses PAM4 at 53.125 Gbps (26.5625 Gbaud) over a single-mode fiber up to 500 m. The optical transmitter is a directly modulated DML or EML driven by the SerDes through a TIA (transimpedance amplifier) in the optical module (QSFP-DD or OSFP form factor).

Unequal PAM4 eye heights indicate one of:
1. **Transmitter non-linearity** (DAC or driver output stage non-linear)
2. **Level spacing miscalibration** (the four PAM4 levels are not equally spaced)
3. **Optical modulator non-linearity** (DML chirp or EML bandwidth limitation)
4. **Receiver non-linearity** (photodetector or TIA saturation)

**Step 1 — Localise the fault: electrical vs optical.**

If the measurement is made at the optical eye (optical sampling oscilloscope after the photodetector): the non-linearity could be in the laser driver, the DML/EML modulator response, or the photodetector.

If the measurement is made at the electrical eye of the SerDes output (before the optical module): the non-linearity is in the SerDes DAC or output driver.

Procedure: Measure the PAM4 eye diagram at both the SerDes electrical output and at the optical output. Compare the eye height ratios.

**Step 2 — Quantify level mismatch.**

Use the eye diagram to measure the four voltage levels $V_0, V_1, V_2, V_3$. Compute the three eye openings:

$$h_1 = V_1 - V_0 - \sigma_{noise,0-1}$$
$$h_2 = V_2 - V_1 - \sigma_{noise,1-2}$$
$$h_3 = V_3 - V_2 - \sigma_{noise,2-3}$$

If $h_1 \ne h_2 \ne h_3$ beyond a ±5% tolerance, there is level mismatch. The ratio of the smallest to largest eye height provides the penalty:

$$\text{Level mismatch penalty} \approx 20\log_{10}\left(\frac{h_{min}}{h_{max}}\right) \text{ dB}$$

**Step 3 — Check DAC calibration.**

Most PAM4 SerDes devices include a built-in PAM4 level calibration register that adjusts the four DAC output levels independently. If the levels are at factory defaults and the transmitter output is non-ideal:

- Access the vendor-specific SerDes CSR (Control and Status Registers) via MDIO.
- Read the TX_LEVEL[0..3] registers.
- Run the built-in calibration routine (if supported) or manually adjust levels until eye heights are equalized.

**Step 4 — Optical module characterisation.**

A DML (directly modulated laser) has a non-linear P-I curve: the power output vs drive current relationship is linear only in the laser's operating region. If the drive current extends into the non-linear region at level 3 (highest amplitude), the top eye is compressed.

Check: Reduce the transmitter swing (lower TX amplitude setting). If the eye height ratio improves, the laser is being driven into non-linearity. The operating point must be adjusted to the linear region.

**Step 5 — FEC error pattern analysis.**

Even though the post-FEC link appears clean, examine the FEC error statistics:
- **Uniform symbol errors across all four level transitions:** Thermal noise floor or jitter is the dominant impairment — equalization is required.
- **Errors concentrated at the middle eye transitions (1→2 and 2→1):** The middle eye is smaller than outer eyes, confirming non-linear level spacing.
- **Burst errors correlated with temperature:** Thermal effects on the optical module are compressing one eye.

**Corrective action:**

If the root cause is optical non-linearity: adjust the TX bias current and modulation amplitude settings in the optical module's EEPROM (SFF-8636 or CMIS registers for QSFP-DD). Most QSFP-DD modules expose TX_APC (automatic power control) and TX_BIAS registers accessible via I2C/I3C.

If the root cause is SerDes DAC non-linearity: apply the SerDes vendor's recommended calibration procedure and verify with an updated eye diagram capture.

---

### Q10. Compare the CDR architectures (bang-bang CDR vs linear CDR) in the context of 25GbE and 100GbE SerDes. What are the jitter tolerance implications for each?

**Answer:**

**Bang-Bang CDR (BB-CDR):**

The bang-bang CDR uses a binary phase detector: on each data transition, it determines only whether the sampling clock arrived early (before the transition) or late (after the transition). The phase correction is a fixed step size $\Delta\phi_{step}$ regardless of the magnitude of the phase error.

The BB-CDR loop dynamics:

$$\phi_{n+1} = \phi_n \pm \Delta\phi_{step} \cdot \text{sign}(error_n)$$

**Characteristics:**

- **Self-noise (dither jitter):** Even when locked, the BB-CDR continuously oscillates between $\pm\Delta\phi_{step}$ because the binary phase detector has no zero-output state. This intrinsic dither is typically 0.1–0.5 UI peak-to-peak and represents deterministic jitter (DJ) generated by the CDR itself.
- **Lock time:** Fast lock due to large phase steps, typically 1–10 μs.
- **Noise bandwidth:** The effective CDR loop bandwidth is approximately $f_{BW} \approx \Delta\phi_{step} \times f_{data} / (2\pi)$. For a step of 0.01 UI at 25 Gbps: $f_{BW} \approx 39.8 \text{ MHz}$.
- **Jitter tolerance (JTOL):** Bang-bang CDRs have good high-frequency jitter tolerance but poor low-frequency jitter tolerance (below the loop bandwidth). IEEE 802.3 Ethernet JTOL specifications require tolerance of 0.65 UI peak-to-peak at low frequencies (< 1 MHz) — a BB-CDR with loop bandwidth 40 MHz meets this easily.

**Linear CDR:**

The linear CDR uses an analog phase-frequency detector (or Alexander detector in digital implementations) that produces an error signal proportional to the phase error magnitude. The feedback loop is a conventional Type-I or Type-II PLL:

$$H(s) = \frac{K_{VCO} \cdot F(s)}{s}$$

where $F(s)$ is the loop filter transfer function.

**Characteristics:**

- **No dither jitter:** The linear CDR naturally settles to a stable operating point with zero steady-state error (Type-II PLL). This eliminates the intrinsic dither jitter that limits BB-CDR.
- **Lock time:** Slower initial lock (must ramp VCO frequency before fine phase lock), typically 10–100 μs.
- **Precise bandwidth control:** The -3 dB bandwidth can be set to a specific value (e.g., 10 MHz) with a stable loop filter design.
- **Jitter tolerance:** Better low-frequency JTOL than BB-CDR for the same loop bandwidth. The JTOL curve is a smooth roll-off at the loop bandwidth rather than the abrupt step of BB-CDR.

**Application to 25GbE and 100GbE:**

| CDR type | 25GBASE-R application | 100GBASE-R PAM4 |
|---|---|---|
| BB-CDR | Widely used; simpler implementation; self-noise tolerable at 25G | Less suitable; dither jitter is proportionally larger at 53 Gbps (UI = 18.8 ps) |
| Linear CDR | Used in premium 25G PHYs for lower intrinsic jitter | Preferred for 100G PAM4; lower dither jitter critical for already-reduced PAM4 SNR |

For PAM4 at 53 Gbps, the UI is 37.7 ps. A BB-CDR dither jitter of 0.2 UI = 7.5 ps would consume a significant fraction of the PAM4 timing margin. Linear CDR with an intrinsic jitter of < 0.5 ps rms is strongly preferred for 100G+ PAM4 SerDes.

**IEEE 802.3 JTOL testing:**

The jitter tolerance test (IEEE 802.3, Clause 93A for 25GbE) applies a sinusoidal jitter $J_{test}(f)$ to the transmitter and verifies that the receiver achieves BER < $10^{-12}$ while tolerating the injected jitter. The JTOL mask specifies:

$$JTOL(f) \ge \begin{cases} 0.65 \text{ UI pp} & f < 4.8 \text{ MHz} \\ 0.65 \times (4.8/f) \text{ UI pp} & 4.8 \text{ MHz} < f < 640 \text{ MHz} \end{cases}$$

This mask shape is defined to match the expected CDR tracking capability. A BB-CDR with insufficient loop bandwidth may fail the JTOL mask in the transition region (4.8–640 MHz).

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| SerDes | Serializer/Deserializer; converts between parallel chip-internal bus and high-speed serial channel |
| PMA | Physical Medium Attachment; the SerDes sub-layer in IEEE 802.3 |
| PCS | Physical Coding Sublayer; handles 64b/66b encoding, scrambling, lane alignment |
| CDR | Clock and Data Recovery; PLL-based circuit recovering the TX clock from embedded transitions in the data |
| BB-CDR | Bang-Bang CDR; binary phase detector; simple but introduces self-noise (dither jitter) |
| CTLE | Continuous-Time Linear Equalizer; analog high-pass filter at RX for ISI cancellation |
| DFE | Decision Feedback Equalizer; feedback digital filter cancelling post-cursor ISI |
| PAM4 | Pulse Amplitude Modulation 4-level; encodes 2 bits/symbol; 9.5 dB SNR penalty vs NRZ |
| 64b/66b | Ethernet line encoding: 64 data bits + 2 sync header bits; 3% overhead |
| 8b/10b | Legacy line encoding; 20% overhead; used in 1GbE, PCIe Gen1/2 |
| KR | Backplane Ethernet physical layer (e.g., 10GBASE-KR, 25GBASE-KR) |
| FEC | Forward Error Correction; mandatory for PAM4 Ethernet; typically RS(544,514) |
| DAC | Direct Attach Copper; twinaxial cable assembly connecting switch ASIC port to server NIC |
| QSFP-DD | Quad Small Form Factor Double Density; 8-lane optical/active module; carries 400GbE |
| TIA | Transimpedance Amplifier; converts optical photodetector current to voltage in optical receivers |
| Reference clock | Low-frequency precision oscillator (e.g., 156.25 MHz) used by TX PLL; must be low phase noise |
| Jitter tolerance (JTOL) | Maximum sinusoidal jitter the receiver can tolerate while maintaining BER < $10^{-12}$ |
| RS(544,514) | Reed-Solomon FEC code used in IEEE 802.3; corrects up to 15 symbol errors per codeword |
| KP4 FEC | Clause 91 RS(544,514); mandatory for 100GbE and 400GbE PAM4 interfaces |
| Eye mask | Pass/fail region defined in the eye diagram; signal must not violate the mask at specified BER |
