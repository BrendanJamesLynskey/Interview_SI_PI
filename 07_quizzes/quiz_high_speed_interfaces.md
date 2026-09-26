# Quiz: High-Speed Interfaces

15 multiple-choice questions covering DDR4/5, PCIe, USB, Ethernet, and SERDES. Questions span three difficulty tiers. Answers with explanations are collected at the end.

---

## Instructions

Select the single best answer for each question. After completing all questions, check your answers against the answer key. For each incorrect answer, read the full explanation before moving on.

Suggested time: 25 minutes.

---

## Questions

### Fundamentals (Q1 -- Q5)

**Q1.** DDR4 SDRAM uses which signalling standard for its data lines?

- A) LVDS (Low Voltage Differential Signalling)
- B) LVCMOS (Low Voltage CMOS)
- C) POD12 (Pseudo Open Drain, 1.2 V); SSTL-style signalling is used for the command/address bus
- D) LVPECL (Low Voltage Positive Emitter-Coupled Logic)

---

**Q2.** PCIe (PCI Express) uses which fundamental signalling topology for each lane?

- A) Single-ended parallel bus with a shared clock
- B) AC-coupled differential serial links, one pair for transmit and one pair for receive (full duplex)
- C) LVDS differential parallel bus with source-synchronous clocking
- D) DC-coupled differential serial links with embedded clock

---

**Q3.** USB 3.2 Gen 2x2 achieves its maximum data rate of 20 Gbps by using:

- A) A single differential pair operating at 20 Gbps
- B) Two differential pairs, each operating at 10 Gbps, simultaneously in the same direction
- C) Four differential pairs, each at 5 Gbps
- D) A differential pair at 10 Gbps combined with 128b/132b encoding efficiency

---

**Q4.** The ODT (on-die termination) feature in DDR interfaces:

- A) Provides impedance matching inside the DRAM die, eliminating the need for discrete termination resistors on the PCB for data lines
- B) Controls the operating temperature of the DRAM die
- C) Sets the output drive strength of the memory controller
- D) Provides ECC (error correction) for data integrity

---

**Q5.** A PCIe Gen 5 link operates at 32 GT/s (gigatransfers per second) per lane. Using 128b/130b encoding, the approximate usable data bandwidth per lane (one direction) is closest to:

- A) 32 Gbps
- B) 25.6 Gbps
- C) 31.5 Gbps
- D) 16 Gbps

---

### Intermediate (Q6 -- Q11)

**Q6.** In a DDR4 PCB layout, the address and command bus is routed as a fly-by topology rather than a star (T-branch) topology. What is the primary reason?

- A) Fly-by routing reduces the total trace length, lowering propagation delay
- B) Fly-by topology allows the memory controller to apply command/address training to compensate for the propagation delay differences to each DRAM, enabling reliable operation at DDR4 speeds where stub resonances from T-branches would degrade signal quality
- C) Star topology requires more PCB layers
- D) Fly-by routing reduces the power consumption of the memory interface

---

**Q7.** PCIe requires AC coupling capacitors on every differential pair. What is the primary function of these coupling capacitors?

- A) To filter high-frequency noise and reduce EMI
- B) To block DC bias differences between devices, allowing each device to set its own common-mode voltage independently and preventing DC current flow through the channel
- C) To set the transmission line characteristic impedance
- D) To compensate for the attenuation of the channel at high frequencies

---

**Q8.** The SERDES (serialiser/deserialiser) CDR (clock and data recovery) circuit recovers the clock from the incoming serial data stream. Which condition is required for reliable CDR lock?

- A) The data stream must include an embedded clock signal on a separate wire
- B) The data stream must have a minimum transition density, typically ensured by line coding (e.g., 8b/10b or 128b/130b scrambling)
- C) The CDR input must be synchronised to an external reference clock at the baud rate
- D) The data stream must use Manchester encoding to guarantee transitions on every bit boundary

---

**Q9.** 100GBASE-KR4 Ethernet uses how many serial lanes, and at what per-lane data rate?

- A) 1 lane at 100 Gbps
- B) 10 lanes at 10 Gbps each
- C) 4 lanes at 25 Gbps each
- D) 4 lanes at 28 Gbps each with overhead

---

**Q10.** In a DDR5 PCB design, the data bus width per sub-channel is 32 bits (vs. 64 bits for DDR4). One significant board-level implication of this change is:

- A) DDR5 requires twice as many address lines per channel
- B) DDR5 operates with two independent 32-bit sub-channels per DIMM slot, improving memory parallelism but requiring the memory controller to manage two separate command/address/data buses per slot, increasing routing complexity
- C) DDR5 eliminates the need for on-die termination (ODT)
- D) DDR5 uses a star topology for all signals, unlike DDR4's fly-by

---

**Q11.** PCIe Gen 4 specifies a maximum channel insertion loss of approximately 28 dB at the Nyquist frequency (8 GHz). If a PCIe Gen 4 channel budget consists of a package, a PCB trace, a connector, and a second PCB trace on the card, and the package contributes 3 dB, the connector contributes 2 dB, and each PCB trace contributes loss at a rate of 1 dB/inch at 8 GHz, what is the maximum total PCB trace length (sum of both PCB traces) allowed?

- A) 28 inches
- B) 23 inches
- C) 25 inches
- D) 20 inches

---

### Advanced (Q12 -- Q15)

**Q12.** A USB4 Version 2.0 link in symmetric mode carries 80 Gbps in each direction over two lanes (40 Gbps per lane) on a USB Type-C cable. It uses PAM3 signalling, packing 11 bits into 7 ternary symbols. The approximate symbol rate per lane is:

- A) 80 Gbaud
- B) 40 Gbaud
- C) 25.6 Gbaud
- D) 20 Gbaud

---

**Q13.** In a DDR4 system, write levelling is a training procedure that adjusts the DQS (data strobe) timing relative to the clock at each DRAM. Why is this training needed?

- A) To compensate for the voltage offset of the DQS differential comparator in each DRAM
- B) To compensate for the different flight times from the memory controller to each DRAM along the fly-by topology, which cause the clock to arrive at different phases at each DRAM position
- C) To calibrate the on-die termination (ODT) resistance value
- D) To measure the bit error rate of each DQ lane and disable faulty bits

---

**Q14.** A 400G Ethernet implementation uses 400GBASE-DR4, which uses 4 lanes of 100 Gbps each over single-mode fibre. Each 100 Gbps lane uses PAM4 (2 bits per symbol) at 53.125 Gbaud, after 256b/257b transcoding and RS(544,514) FEC. What property of the 256b/257b encoding scheme makes it preferable to 64b/66b at these data rates?

- A) 256b/257b has higher DC-balance, which is important for AC-coupled optical links
- B) 256b/257b has lower encoding overhead (1/257 ≈ 0.39% vs. 2/66 ≈ 3.03%), meaning more of the raw baud rate carries useful data
- C) 256b/257b requires less complex scrambling logic in the SERDES
- D) 256b/257b adds more redundancy for forward error correction

---

**Q15.** In a PCIe Gen 5 or Gen 6 channel compliance analysis, the link training and status state machine (LTSSM) includes a Loopback state. A debug engineer observes that a link fails to exit the Detect state and never progresses to Polling. The most likely root causes are:

- A) The receiver CTLE peaking is set too high, over-equalising the compliance pattern
- B) The transmitter's receiver-detection circuit is not seeing the far-end receiver termination through the AC-coupling capacitors, typically because of an open in the pair (unsoldered capacitor, broken trace or via), a short, or a link partner that is unpowered or has its termination disabled
- C) The CDR has locked to the wrong frequency harmonic
- D) The spread-spectrum clocking (SSC) is enabled but the CDR cannot track the SSC modulation rate

---

## Answer Key

| Q  | Answer |
|----|--------|
| 1  | C      |
| 2  | B      |
| 3  | B      |
| 4  | A      |
| 5  | C      |
| 6  | B      |
| 7  | B      |
| 8  | B      |
| 9  | C      |
| 10 | B      |
| 11 | B      |
| 12 | C      |
| 13 | B      |
| 14 | B      |
| 15 | B      |

---

## Detailed Explanations

**Q1 -- Answer: C**

DDR4 uses SSTL-12 (Stub Series Terminated Logic for 1.2 V) for some signals and POD12 (Pseudo Open Drain 1.2 V) for data (DQ) and data strobe (DQS) signals in JEDEC-standard DDR4. POD12 uses a unidirectional pull-up to the VDDQ rail, reducing power compared to SSTL push-pull. Option A (LVDS) is not used: DQS and CK are differential pairs, but they use the same POD/SSTL levels as the single-ended lines, not LVDS. Option B (LVCMOS) is too slow for DDR4 data rates. Option D (LVPECL) is used in high-speed clock distribution, not DDR DRAM interfaces.

---

**Q2 -- Answer: B**

PCIe uses AC-coupled differential serial links. AC coupling (via 75-265 nF capacitors on each line) allows each device to set its own common-mode bias independently. The differential pair for each direction is physically separate -- one pair for TX and one pair for RX per lane -- providing full-duplex communication without a shared medium. Option A (parallel bus with shared clock) describes older parallel bus architectures (e.g., PCI). Option C (LVDS parallel with source-synchronous clock) is used by some parallel standards like LVDS camera interfaces. Option D (DC-coupled differential serial) is incorrect -- PCIe mandates AC coupling at the card edge connector.

---

**Q3 -- Answer: B**

USB 3.2 Gen 2x2 uses two SuperSpeed differential pairs simultaneously in the same direction (called "dual-lane" operation), each running at USB 3.2 Gen 2 speed (10 Gbps), for a combined 20 Gbps. This is enabled by the USB Type-C connector which has two TX and two RX differential pairs. Option A is incorrect -- no current USB implementation runs a single pair at 20 Gbps. Option C (four pairs at 5 Gbps) is not an actual USB 3.2 mode. Option D mixes USB 3.2 Gen 2 and encoding -- 128b/132b is used but the 20 Gbps rate comes from the dual lanes, not just encoding efficiency.

---

**Q4 -- Answer: A**

ODT (on-die termination) incorporates the termination resistors inside the DRAM silicon. This eliminates the board-level Thevenin or parallel termination resistors that were required for SDRAM and DDR1, reduces PCB component count, and allows the termination to be dynamically enabled and disabled to save power when a rank is not being accessed. Option B is incorrect -- that describes thermal management or a temperature sensor (thermistor). Option C describes drive strength calibration (ZQ calibration), a related but distinct feature. Option D describes ECC, which is a data reliability feature, not termination.

---

**Q5 -- Answer: C**

PCIe Gen 5 line rate is 32 GT/s (32 Gbps NRZ). The 128b/130b encoding carries 128 bits of payload for every 130 bits transmitted. Efficiency = 128/130 = 98.46%. Usable bandwidth = 32 Gbps * (128/130) ≈ 31.5 Gbps per lane, per direction. Option A (32 Gbps) ignores encoding overhead. Option B (25.6 Gbps) incorrectly applies 8b/10b overhead (80% efficiency), which PCIe used only up to Gen 2. Option D (16 Gbps) is the PCIe Gen 4 line rate, not Gen 5.

---

**Q6 -- Answer: B**

At DDR4 data rates (up to 3200 MT/s), T-branch (star) topologies create stub resonances. The stub from the branch point to each undriven DRAM creates a notch in the frequency response, degrading signal quality. Fly-by topology avoids stubs by routing through each DRAM in sequence. The increasing propagation delay to each DRAM along the chain is compensated by write levelling training, where each DRAM independently calibrates its DQS-to-CK timing relationship. Option A is incorrect; fly-by is generally longer than a centralised star. Option C is incorrect; the layer count is determined by other routing requirements. Option D is unsupported -- the topology choice has minimal direct effect on power consumption.

---

**Q7 -- Answer: B**

AC coupling capacitors in PCIe (and other SERDES standards) block DC and allow the transmitter and receiver to independently set their differential common-mode voltage levels. Without AC coupling, a DC voltage difference between two devices' common modes would drive current through the interconnect and possibly damage input circuits. Option A (filtering) is a side effect but not the primary purpose -- the capacitors are placed in the signal path specifically to allow common-mode independence. Option C is incorrect; the impedance is set by the differential pair geometry, not the coupling capacitors. Option D describes equalisation (CTLE or FFE), not coupling capacitors.

---

**Q8 -- Answer: B**

A CDR extracts clock information from transitions in the data stream. If long runs of identical bits occur (many consecutive 1s or 0s), there are no transitions, and the CDR's PLL will drift. Line coding (8b/10b guarantees at most 5 consecutive identical symbols; 128b/130b uses scrambling to statistically guarantee transition density) ensures sufficient transitions for CDR lock. Option A is incorrect -- SERDES interfaces are specifically designed to embed the clock in the data, not use a separate clock wire (which would require matched routing and scaling poorly). Option C contradicts the purpose of CDR (to recover the clock from data, not require an external reference). Option D (Manchester encoding) was used in older standards (10BASE-T Ethernet, CAN) and doubles the baud rate -- modern high-speed SERDES does not use Manchester encoding.

---

**Q9 -- Answer: C**

100GBASE-KR4 is defined in IEEE 802.3bj and uses 4 lanes, each running at 25.78125 GBd NRZ (25 Gbps of data after 64b/66b coding), for 100 Gbps aggregate. In the port name, "K" indicates backplane and "R" the 64B/66B (scrambled) PCS coding; KR4 also uses RS(528,514) FEC. Option A (1 lane at 100 Gbps) describes the later single-lane 100GBASE-KR1/CR1 (IEEE 802.3ck). Option B (10 lanes at 10 Gbps) describes 100GBASE-CR10/SR10 (IEEE 802.3ba). Option D (4 lanes at 28 Gbps) approximately describes OIF CEI-28G or proprietary implementations; the IEEE rate for 100GBASE-KR4 is 25.78125 GBd per lane.

---

**Q10 -- Answer: B**

DDR5 splits each 64-bit channel into two independent 32-bit sub-channels. Each sub-channel has its own command/address bus, data bus, and DQS. This doubles the number of independent memory operations per clock cycle but requires the memory controller to manage twice as many buses per DIMM slot compared to DDR4. This increases the memory controller I/O count and PCB routing complexity significantly. Option A is incorrect -- the address width per sub-channel is similar to DDR4; the total effective address width may be comparable. Option C is incorrect; DDR5 retains and expands ODT capabilities. Option D is incorrect; DDR5 still uses fly-by topology for command/address.

---

**Q11 -- Answer: B**

Total budget = 28 dB. Fixed losses: package = 3 dB, connector = 2 dB. Remaining for PCB traces = 28 - 3 - 2 = 23 dB. At 1 dB/inch, maximum combined trace length = 23 inches. Option A (28 inches) ignores the package and connector contributions. Option C (25 inches) subtracts only the package. Option D (20 inches) over-subtracts. This type of loss budget calculation is fundamental to PCIe channel compliance analysis.

---

**Q12 -- Answer: C**

USB4 Version 2.0 reaches 80 Gbps per direction with two lanes of 40 Gbps each (and an asymmetric mode with three lanes, 120 Gbps, one way). It uses PAM3: each symbol carries log2(3) = 1.58 bits at most, and the 11-bits-in-7-symbols mapping (3^7 = 2187 ≥ 2^11 = 2048) carries 11/7 = 1.57 bits per symbol. At 25.6 GBd that is 25.6 × 11/7 = 40.2 Gbps per lane (Keysight and Synopsys USB4 v2.0 overviews). Option A would be one bit per symbol for the whole link; option B is NRZ at 40 Gbps per lane; option D is PAM4 (2 bits per symbol), which USB4 v2.0 does not use.

---

**Q13 -- Answer: B**

In a fly-by topology, the clock signal is routed from the memory controller and passes through each DRAM position in sequence. Each DRAM sees the clock at a progressively later phase (longer propagation delay). The data strobe (DQS) for writes is generated by the memory controller and must be aligned to the clock as seen at each DRAM, not as seen at the controller. Write levelling trains the DQS delay in the controller by sending training patterns and having each DRAM report the phase relationship, allowing the controller to set per-DRAM DQS delays. Option A describes a voltage offset calibration (VREF training), a different procedure. Option C describes ZQ calibration. Option D describes a post-silicon screening test, not a standard training procedure.

---

**Q14 -- Answer: B**

Encoding overhead is the fraction of raw bandwidth consumed by the encoding header/overhead bits. For 64b/66b: overhead = 2/66 = 3.03%, leaving 97% efficiency. For 256b/257b: overhead = 1/257 = 0.39%, leaving 99.61% efficiency. At 400G and beyond, every fraction of a percent of overhead directly translates to line rate. 256b/257b was adopted specifically to reduce this overhead. Option A is incorrect -- 256b/257b uses scrambling (not balanced coding) to achieve spectral distribution; DC balance is maintained differently and is not the motivation for choosing 256b/257b over 64b/66b. Option C is incorrect -- 256b/257b uses a more complex scrambler, not less. Option D is incorrect -- 256b/257b does not add FEC redundancy; FEC is handled by a separate FEC layer (e.g., Reed-Solomon).

---

**Q15 -- Answer: B**

In the Detect state each transmitter checks whether a receiver is electrically present: it steps its common-mode voltage and times how fast the line charges. A receiver's termination, seen through the AC-coupling capacitor, makes the line charge slowly; an open line charges quickly. If no receiver is detected, the link never moves to Polling (where bit lock, symbol lock and speed negotiation occur). The failure is therefore caused by an open (unsoldered or missing capacitor, broken trace, cold joint, missing via), a short, or a link partner whose termination is not present. Channel insertion loss does not affect this low-frequency test. Option A (CTLE setting) only matters once data is being received, after Detect. Option C (CDR locking to wrong harmonic) would appear as a link that reaches Polling or Configuration but then fails -- not a Detect failure. Option D (SSC with CDR) would also appear as a failure after Detect, not during.
