# Quiz: Signal Integrity

15 multiple-choice questions covering S-parameters, eye diagrams, crosstalk, ISI, equalisation, and jitter. Questions span three difficulty tiers. Answers with explanations are collected at the end.

---

## Instructions

Select the single best answer for each question. After completing all questions, check your answers against the answer key. For each incorrect answer, read the full explanation before moving on.

Suggested time: 25 minutes.

---

## Questions

### Fundamentals (Q1 -- Q5)

**Q1.** S21 in a two-port network is defined as:

- A) The ratio of the reflected wave at port 1 to the incident wave at port 1, with port 2 matched
- B) The ratio of the transmitted wave at port 2 to the incident wave at port 1, with port 2 matched
- C) The ratio of the transmitted wave at port 1 to the incident wave at port 2, with port 1 matched
- D) The ratio of forward voltage to backward voltage measured at port 2

---

**Q2.** An eye diagram is constructed by:

- A) Plotting the bit error rate versus the decision threshold voltage
- B) Overlaying many segments of a time-domain waveform, each one or two bit periods long, triggered on the bit clock
- C) Plotting signal amplitude versus frequency on a logarithmic scale
- D) Measuring the voltage at the receiver input and plotting it against the transmitted bit pattern

---

**Q3.** Near-end crosstalk (NEXT) is measured:

- A) At the far end of the victim line, in the same direction as the aggressor signal
- B) At the near end of the victim line, at the same end as the aggressor source
- C) At the mid-point of the victim line
- D) Across the termination resistor at the far end of the victim line

---

**Q4.** Inter-symbol interference (ISI) is primarily caused by:

- A) Noise coupling from adjacent power planes
- B) Frequency-dependent attenuation and phase distortion in the channel that causes energy from one bit to spread into adjacent bit periods
- C) Impedance mismatches at via transitions
- D) Ground bounce in the transmitter package

---

**Q5.** The S11 parameter of a well-matched 50 Ohm transmission line, measured at its input, should ideally be:

- A) 0 dB (S11 = 1 in linear)
- B) Very large and positive in dB
- C) Very small and negative in dB (approaching negative infinity for a perfect match)
- D) Exactly -6 dB

---

### Intermediate (Q6 -- Q11)

**Q6.** A channel has an insertion loss of -20 dB at the Nyquist frequency. A receiver uses a 4-tap decision feedback equaliser (DFE). Which statement best describes what the DFE is doing?

- A) Boosting high-frequency content in the transmit waveform before the channel
- B) Using decisions on already-decoded bits to subtract their ISI contribution from the current bit's sample
- C) Applying a high-pass filter at the receiver input to compensate the channel's low-pass response
- D) Adding pre-emphasis to the transmitted signal to flatten the channel response

---

**Q7.** In an eye diagram, the "eye height" is primarily degraded by:

- A) Random jitter only
- B) Deterministic jitter only
- C) Noise, crosstalk, ISI, and reflections, all of which reduce the voltage margin
- D) The baud rate exceeding the channel bandwidth

---

**Q8.** Far-end crosstalk (FEXT) in a PCB transmission line:

- A) Is always larger in magnitude than NEXT for microstrip traces
- B) Is the coupled noise observed at the far end of the victim, and travels in the same direction as the aggressor signal
- C) Cancels out completely in a differential pair
- D) Is independent of the coupling length between aggressor and victim traces

---

**Q9.** The bathtub curve in serial link analysis plots:

- A) Bit error rate (BER) versus sample time position within the unit interval (UI), revealing the timing margin at a target BER
- B) Insertion loss versus frequency for the channel
- C) Jitter amplitude versus jitter frequency
- D) Eye height versus eye width simultaneously on a single plot

---

**Q10.** Continuous-time linear equalisation (CTLE) at the receiver differs from a feed-forward equaliser (FFE) at the transmitter primarily because:

- A) CTLE operates in the frequency domain by applying a high-frequency boost, while FFE applies pre-computed tap weights in the time domain at the transmitter
- B) CTLE can only compensate for random jitter, while FFE compensates for deterministic jitter
- C) CTLE is applied before the channel and FFE is applied after the channel
- D) CTLE uses analog circuitry while FFE is always implemented digitally

---

**Q11.** Total jitter (TJ) at a given BER target is commonly decomposed into:

- A) TJ = Random Jitter (RJ) + Deterministic Jitter (DJ), where both are measured peak-to-peak
- B) TJ = RJ_rms * Q_factor + DJ_pp, where Q_factor is determined by the target BER and accounts for the unbounded Gaussian tails of RJ
- C) TJ = DJ_pp * BER_target
- D) TJ = sqrt(RJ^2 + DJ^2), treating both as Gaussian distributions

---

### Advanced (Q12 -- Q15)

**Q12.** A 4-port S-parameter measurement is taken on a differential pair. The mixed-mode S-parameters are desired. Which mixed-mode parameter describes the conversion of a differential stimulus at port 1 into a common-mode response at port 2?

- A) Sdd21 (differential-to-differential transmission)
- B) Scc21 (common-mode-to-common-mode transmission)
- C) Sdc21 (differential stimulus at port 1, common-mode response at port 2)
- D) Scd21 (common-mode stimulus at port 1, differential response at port 2)

---

**Q13.** A 25 Gbps NRZ serial link uses a channel with the following measured properties: IL at 12.5 GHz (Nyquist) = -18 dB, IL at 6.25 GHz = -12 dB, IL at 1 GHz = -3 dB. The channel's loss profile is approximately:

- A) Dominated by dielectric loss (linear with frequency)
- B) Dominated by skin-effect loss (proportional to sqrt(f))
- C) Dominated by resistive DC loss (flat with frequency)
- D) A mixture where skin-effect loss dominates below 5 GHz and dielectric loss dominates above 5 GHz

---

**Q14.** In a PAM4 signalling scheme compared to NRZ at the same data rate, which statement is correct?

- A) PAM4 requires the same SNR as NRZ because both carry the same data rate
- B) PAM4 halves the baud rate, reducing ISI and channel loss, but requires approximately 9.5 dB more SNR for the same BER, and has 3 eye openings instead of 1
- C) PAM4 doubles the baud rate, which halves the required channel bandwidth
- D) PAM4 improves noise margin compared to NRZ because the symbol spacing is larger with 4 levels

---

**Q15.** A receiver's CDR (clock and data recovery) circuit loses lock intermittently on a 10 Gbps SERDES link. The eye diagram shows a closed eye at the target BER, but the random jitter appears low. The most likely cause is:

- A) Excessive random jitter from the transmitter PLL
- B) A large deterministic jitter component, specifically spread-spectrum clocking (SSC) or data-dependent jitter from ISI exceeding the CDR's tracking bandwidth
- C) Insufficient pre-emphasis at the transmitter
- D) The oscilloscope sample rate is too low to resolve the jitter accurately

---

## Answer Key

| Q  | Answer |
|----|--------|
| 1  | B      |
| 2  | B      |
| 3  | B      |
| 4  | B      |
| 5  | C      |
| 6  | B      |
| 7  | C      |
| 8  | B      |
| 9  | A      |
| 10 | A      |
| 11 | B      |
| 12 | C      |
| 13 | A      |
| 14 | B      |
| 15 | B      |

---

## Detailed Explanations

**Q1 -- Answer: B**

S-parameter notation Sij means: wave entering port j, wave measured at port i. S21 = b2/a1, with all ports other than port 1 terminated in the reference impedance (matched, so no re-reflections). It describes forward insertion gain or loss. Option A describes S11 (return loss / reflection). Option C describes S12 (reverse transmission). Option D is not standard S-parameter definition.

---

**Q2 -- Answer: B**

An eye diagram is formed by triggering an oscilloscope (or performing a post-processing equivalent) on the recovered clock and overlaying successive 1UI or 2UI segments of the data waveform on top of one another. The resulting "eye" opening indicates timing and voltage margins. Option A describes a BER vs. threshold sweep, which produces a bathtub curve or a voltage bathtub. Option C describes a frequency-domain spectrum. Option D is not a real measurement technique.

---

**Q3 -- Answer: B**

NEXT (near-end crosstalk) is observed at the same physical end of the coupled line pair as the aggressor source. The victim signal exits the near end in the opposite direction to the aggressor. In a PCB context, NEXT is most problematic in bi-directional bus topologies. FEXT (option A) is measured at the far end. Options C and D do not correspond to standard crosstalk measurement points.

---

**Q4 -- Answer: B**

ISI arises because real channels are bandlimited: they attenuate high frequencies more than low frequencies, and introduce frequency-dependent phase shifts. A transmitted bit's energy smears over multiple subsequent bit periods at the receiver. Option A describes power supply noise coupling, a different mechanism. Option C (via discontinuities) causes reflections, which are a different form of linear distortion (though they can contribute to ISI). Option D (ground bounce) is a non-linear switching noise effect.

---

**Q5 -- Answer: C**

S11 represents the reflected power. A perfect match (Z_in = Z0 = 50 Ohm) means zero reflection, so S11 approaches 0 in linear scale and negative infinity in dB. In practice, a return loss better than -20 dB (|S11| < 0.1 linear) is considered good. Option A (0 dB) means all power is reflected, indicating a complete mismatch (open or short). Option B makes no physical sense. Option D (-6 dB, |S11| = 0.5 linear) corresponds to only 25% power transmission, indicating a very poor match.

---

**Q6 -- Answer: B**

A DFE uses previously decoded bits (which are known, assuming correct decisions) to subtract their ISI contributions from the current sample. Because it operates on already-decoded data rather than the raw analog signal, it does not amplify noise. This is its key advantage over a linear equaliser at high loss. Option A describes transmitter pre-emphasis or feed-forward equalisation (FFE), which operates before the channel. Option C describes a CTLE (continuous-time linear equaliser) at the receiver. Option D also describes transmitter pre-emphasis.

---

**Q7 -- Answer: C**

Eye height reduction (vertical eye closure) is caused by all mechanisms that reduce the voltage margin at the decision point: random noise, crosstalk coupling, ISI from channel loss and dispersion, and reflections from impedance mismatches. Option A is too narrow -- random jitter degrades eye width (horizontal), not eye height directly. Option B is also incomplete -- deterministic jitter closes the eye width, not height directly. Option D is partially true (bandwidth limiting does close the eye), but the question asks about the general category of degradations.

---

**Q8 -- Answer: B**

FEXT travels in the same direction as the aggressor (forward direction along the victim trace) and is observed at the far end of the victim line. Option A is incorrect; FEXT is generally smaller than NEXT for PCB microstrip at typical trace separations, because the FEXT coupling involves a cancellation between the capacitive and inductive coupling terms that subtracts, while NEXT coupling terms add. Option C is incorrect; differential signalling reduces mode-conversion crosstalk but does not eliminate FEXT between two differential pairs. Option D is incorrect; FEXT is strongly dependent on the coupled length (it accumulates over the coupled length for a forward-travelling wave).

---

**Q9 -- Answer: A**

The bathtub curve plots BER as a function of the sampling time offset within the unit interval. The "tub" shape reveals the timing margin: the flat bottom at the optimal sampling point represents the lowest BER, and the BER rises steeply at the edges of the UI. The width of the tub at the target BER (e.g., 1e-12) defines the timing margin. Option B describes an insertion loss plot (S21 vs. frequency). Option C describes jitter spectrum analysis. Option D describes what a 2D eye contour plot does, not the bathtub curve.

---

**Q10 -- Answer: A**

CTLE is an analog filter (typically a peaking amplifier or zero-pole network) placed at the receiver that boosts high-frequency content to compensate for the channel's low-pass behaviour. FFE applies a set of delay-and-sum tap weights in the time domain, either at the transmitter (as pre-emphasis/de-emphasis) or at the receiver. The key difference is that CTLE is a continuous-time analog boost applied after the channel, while transmitter FFE shapes the transmit waveform before the channel. Option B is wrong; neither equaliser specifically targets jitter types in that way. Option C reverses the locations. Option D is not always true; CTLE uses analog circuitry but FFE can be implemented in analog or digital circuits.

---

**Q11 -- Answer: B**

The standard JEDEC/industry method for total jitter decomposition is TJ(BER) = DJ_pp + 2 * Q(BER) * RJ_rms, where Q(BER) is the Q-factor corresponding to the target BER (for BER = 1e-12, Q ≈ 7.03). This accounts for the fact that RJ follows a Gaussian distribution with unbounded tails -- the peak-to-peak RJ contribution grows logarithmically with the number of accumulated bits (i.e., it depends on BER). Option A is incorrect because simply adding peak-to-peak values ignores the statistical nature of RJ. Option C omits RJ entirely. Option D incorrectly treats DJ as Gaussian; DJ is deterministic and bounded.

---

**Q12 -- Answer: C**

Mixed-mode S-parameters use a two-letter mode prefix: the first letter is the mode of the response port, the second is the mode of the stimulus port. "d" = differential mode, "c" = common mode. Sdc21 = response in common mode at port 2 due to differential stimulus at port 1. This is the mode conversion term and is particularly important for EMC and noise rejection. Option A (Sdd21) is pure differential-to-differential transmission (the desired signal path). Option B (Scc21) is common-mode transmission. Option D (Scd21) is the opposite mode conversion: common-mode input producing differential output at the far end.

---

**Q13 -- Answer: A**

Examining the loss values: at 1 GHz the loss is -3 dB; at 6.25 GHz (6.25x higher) it is -12 dB (4x higher loss); at 12.5 GHz (12.5x higher) it is -18 dB (6x higher loss). Dielectric loss scales linearly with frequency, so a 12.5x increase in frequency would give 12.5x more dielectric loss in dB. Skin-effect loss scales as sqrt(f), so a 12.5x increase would give sqrt(12.5) ≈ 3.5x more loss in dB. The observed ratio from 1 GHz to 12.5 GHz is 18/3 = 6x, which is between sqrt(12.5) = 3.5 (skin) and 12.5 (dielectric). At these frequencies and typical FR4 loss tangents, the predominance of dielectric loss is consistent. The approximately linear increase in dB with frequency confirms dielectric loss dominance.

---

**Q14 -- Answer: B**

PAM4 encodes 2 bits per symbol, so the baud rate is halved compared to NRZ for the same bit rate. This reduces ISI and channel loss at the Nyquist frequency. However, PAM4 has 3 eye openings between 4 voltage levels. The voltage spacing between adjacent levels is 1/3 of the full signal swing (versus 1/2 for NRZ). The required SNR is approximately 9.5 dB higher (20*log10(3) ≈ 9.5 dB) for the same BER on an AWGN channel. Option A ignores the SNR penalty. Option C gets the baud rate change backwards (it halves, not doubles). Option D is the opposite -- PAM4 reduces the eye height per level, not improves it.

---

**Q15 -- Answer: B**

CDR lockout with a closed eye and low RJ points to a deterministic jitter mechanism. Spread-spectrum clocking (SSC) modulates the clock frequency (typically +0/-0.5% for PCIe) to reduce EMI, creating a periodic jitter component that may exceed the CDR's tracking bandwidth at certain modulation rates. Data-dependent jitter (DDJ) from ISI can also cause the eye to close in a pattern-specific way that looks like low RJ but causes burst errors on specific patterns. Option A (high RJ from PLL) would appear as high RJ in the measurement. Option C (pre-emphasis) would show a large, symmetrically degraded eye, not pattern-specific closure. Option D is a measurement artefact concern but would not cause actual CDR lock loss in hardware.
