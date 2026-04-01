# S-Parameters

## Overview

Scattering parameters (S-parameters) are the standard language for characterising high-frequency networks. Unlike impedance or admittance matrices, S-parameters are defined in terms of travelling voltage waves rather than open- or short-circuit terminal conditions, making them practically measurable with a Vector Network Analyser (VNA) at RF and microwave frequencies. In signal integrity work, S-parameters appear everywhere: channel characterisation, connector modelling, via analysis, differential pair simulation, and compliance testing for PCIe, USB, DDR, and Ethernet standards.

This document covers single-ended S-parameters, mixed-mode (differential) S-parameters, and their interpretation in the context of high-speed serial links.

---

## Tier 1: Fundamentals

### Q1. What are S-parameters, and why are they used instead of Z- or Y-parameters at high frequencies?

**Answer:**

S-parameters (scattering parameters) describe a linear network by relating the amplitudes of incoming and outgoing travelling voltage waves at each port. For a two-port network with a reference impedance $Z_0$ (typically 50 $\Omega$):

$$b_1 = S_{11} a_1 + S_{12} a_2$$
$$b_2 = S_{21} a_1 + S_{22} a_2$$

where:
- $a_n$ = normalised incident wave amplitude at port $n$
- $b_n$ = normalised reflected wave amplitude at port $n$

**Why not Z- or Y-parameters at high frequencies:**

Z-parameters require open-circuit termination ($I = 0$) and Y-parameters require short-circuit termination ($V = 0$) at ports not under stimulus. At high frequencies, these conditions are impossible to achieve — parasitic inductance prevents a true short and parasitic capacitance prevents a true open. Any residual mismatch introduces large measurement errors.

S-parameters require only that the non-stimulated ports are terminated in the reference impedance $Z_0$. A 50 $\Omega$ termination is easily realised with a matched load at microwave frequencies, making S-parameters directly and accurately measurable with a VNA.

**Common mistake:** Treating S-parameters as inherently tied to 50 $\Omega$. The reference impedance $Z_0$ is a parameter of the S-parameter set, not a physical law. Signal integrity simulations often use $Z_0 = 100\ \Omega$ differential or $Z_0 = 85\ \Omega$ to match the channel's characteristic impedance. A Touchstone file header specifies the reference impedance.

---

### Q2. Define S11, S21, S12, and S22 for a two-port network and give a physical interpretation of each.

**Answer:**

All S-parameters are defined with all ports terminated in the reference impedance $Z_0$. The subscript convention is $S_{output,input}$ — first index is the output port (where the wave exits), second index is the input port (where the wave enters).

**S11 — Input Reflection Coefficient:**

$$S_{11} = \frac{b_1}{a_1}\bigg|_{a_2=0}$$

Ratio of the wave reflected from port 1 to the wave incident on port 1, when port 2 is terminated in $Z_0$.

*Physical meaning:* How well port 1 is matched to $Z_0$. A perfect match gives $S_{11} = 0$ ($-\infty$ dB). A short circuit gives $S_{11} = -1$ ($0$ dB at $180°$). Also called **return loss** (with sign convention: $RL = -20\log_{10}|S_{11}|$ dB, positive values indicate loss of the reflected signal relative to the incident signal).

**S21 — Forward Transmission (Insertion Loss):**

$$S_{21} = \frac{b_2}{a_1}\bigg|_{a_2=0}$$

Ratio of the wave arriving at port 2 to the wave incident on port 1.

*Physical meaning:* How much signal is transmitted from input to output. For a passive channel, $|S_{21}| \le 1$ always. Expressed as **insertion loss** in dB: $IL = -20\log_{10}|S_{21}|$ dB (positive values; a larger number means more loss). A 3 dB insertion loss means half the power is transmitted.

**S12 — Reverse Transmission:**

$$S_{12} = \frac{b_1}{a_2}\bigg|_{a_1=0}$$

*Physical meaning:* Signal travelling from port 2 to port 1 — the reverse direction. For a reciprocal network (passive, linear, no magnetic materials), $S_{12} = S_{21}$ exactly. Non-reciprocal elements (amplifiers, circulators, isolators) break this symmetry.

**S22 — Output Reflection Coefficient:**

$$S_{22} = \frac{b_2}{a_2}\bigg|_{a_1=0}$$

*Physical meaning:* Match quality at port 2. Analogous to S11 but looking into the output port.

---

### Q3. What is insertion loss and return loss? Convert between linear and dB representations. What values are acceptable in a PCIe Gen 4 channel?

**Answer:**

**Insertion Loss (IL):**

The reduction in signal magnitude due to passing through the channel. Defined as:

$$IL_{dB} = -20\log_{10}|S_{21}| = 20\log_{10}\left(\frac{|a_1|}{|b_2|}\right)$$

The negative sign ensures IL is a positive number for a passive network (since $|S_{21}| \le 1$, $\log_{10}|S_{21}| \le 0$).

Conversion examples:
| $|S_{21}|$ (linear) | $IL$ (dB) |
|---|---|
| 1.0 | 0 dB (lossless) |
| 0.707 | 3.0 dB |
| 0.5 | 6.0 dB |
| 0.316 | 10 dB |
| 0.1 | 20 dB |

**Return Loss (RL):**

The ratio of reflected power to incident power:

$$RL_{dB} = -20\log_{10}|S_{11}|$$

A larger positive return loss value means *less* reflection (better match). A return loss of 20 dB means 1% of incident power is reflected.

Conversion examples:
| $|S_{11}|$ (linear) | $RL$ (dB) | Mismatch severity |
|---|---|---|
| 0.0 | $\infty$ | Perfect match |
| 0.1 | 20 dB | Excellent |
| 0.316 | 10 dB | Acceptable |
| 1.0 | 0 dB | Total reflection |

**PCIe Gen 4 (16 GT/s) channel limits:**

| Parameter | Limit | Notes |
|---|---|---|
| Max insertion loss at Nyquist (8 GHz) | 36 dB | Total channel including package, via, connector, PCB trace |
| Return loss (each end) | $\le -6$ dB up to 8 GHz | $|S_{11}| \le 0.5$ |
| Differential insertion loss (Sdd21) | $\le 36$ dB at 8 GHz | Measured in mixed-mode |

The 36 dB limit for PCIe Gen 4 is quite aggressive and typically requires low-loss laminate materials ($D_f < 0.004$) and careful via design with backdrilling.

---

### Q4. Explain impedance mismatch and derive the reflection coefficient $\Gamma$ in terms of impedance.

**Answer:**

When a transmission line with characteristic impedance $Z_0$ is terminated in a load $Z_L \ne Z_0$, the boundary condition at the termination requires both voltage and current continuity. The incident wave cannot be fully absorbed by $Z_L$, so a reflected wave is generated.

**Derivation:**

At the termination, total voltage $V = V^+ + V^-$ and total current $I = (V^+ - V^-)/Z_0$. The load requires $V = Z_L I$:

$$V^+ + V^- = Z_L \cdot \frac{V^+ - V^-}{Z_0}$$

$$Z_0(V^+ + V^-) = Z_L(V^+ - V^-)$$

$$V^-(Z_0 + Z_L) = V^+(Z_L - Z_0)$$

$$\Gamma = \frac{V^-}{V^+} = \frac{Z_L - Z_0}{Z_L + Z_0}$$

This is the voltage reflection coefficient. It is complex-valued in general (frequency-dependent $Z_L$).

**Special cases:**

| Termination | $Z_L$ | $\Gamma$ | $S_{11}$ interpretation |
|---|---|---|---|
| Matched | $Z_0$ | 0 | No reflection |
| Open circuit | $\infty$ | +1 | Full positive reflection |
| Short circuit | 0 | -1 | Full negative reflection |
| Capacitor at DC | $\infty$ | +1 | Reflects like an open |
| Inductor at DC | 0 | -1 | Reflects like a short |

**Relationship to S11:**

For a two-port network driven from a $Z_0$ source and terminated in $Z_0$, $S_{11} = \Gamma$ at the input port. In general, $S_{11}$ includes contributions from internal reflections throughout the network, not just the termination.

**Common mistake:** Applying the formula $\Gamma = (Z_L - Z_0)/(Z_L + Z_0)$ to a frequency-varying channel and forgetting that $Z_L$ is itself a function of frequency. The correct approach is to measure or simulate $S_{11}(f)$ across the full bandwidth.

---

### Q5. What is a Touchstone file? What do the columns represent in a 2-port .s2p file?

**Answer:**

A Touchstone file (.sNp, where N is the port count) is the standard text format for storing N-port S-parameter data as a function of frequency. It is defined by the Touchstone 1.0 and 2.0 specifications published by the IPC and used natively by Keysight ADS, Cadence AWR, Ansys HFSS, and most SI tools.

**File structure for a 2-port (.s2p) file:**

```
! Optional comment lines start with '!'
# Hz S MA R 50
! Frequency format: Hz
! Parameter type: S (S-parameters, could also be Z, Y, H, G)
! Data format: MA (Magnitude-Angle), could be RI (Real-Imaginary) or DB (dB-Angle)
! Reference impedance: 50 ohms

1e6   0.9999  -0.01   0.0001  89.9   0.0001  89.9   0.9999  -0.01
1e7   0.9990  -0.1    0.001   89.5   0.001   89.5   0.9990  -0.1
...
```

**Column order for a 2-port .s2p with MA format:**

| Col 1 | Col 2 | Col 3 | Col 4 | Col 5 | Col 6 | Col 7 | Col 8 | Col 9 |
|---|---|---|---|---|---|---|---|---|
| Freq | $|S_{11}|$ | $\angle S_{11}$ | $|S_{21}|$ | $\angle S_{21}$ | $|S_{12}|$ | $\angle S_{12}$ | $|S_{22}|$ | $\angle S_{22}$ |

The column order always follows: $S_{11}, S_{21}, S_{12}, S_{22}$ — not the matrix row-by-row order that a mathematician might expect.

**Touchstone 2.0 additions:** Mixed-mode (differential) port definitions, port impedance per-port (not a single $R$ value), and noise parameter inclusion.

---

## Tier 2: Intermediate

### Q6. What are mixed-mode S-parameters? Define Sdd21, Scc21, Sdc21, and Scd21 and explain what each tells you about a differential channel.

**Answer:**

Mixed-mode S-parameters recast a differential pair's 4-port single-ended S-parameter matrix into a 2-port matrix using differential (D) and common-mode (C) wave definitions. The mixed-mode representation directly characterises how a differential channel handles differential signals and common-mode signals.

**Wave definitions** for a differential pair with ports 1+ and 1- (left side) and 2+ and 2- (right side):

$$a_D = \frac{a_{1+} - a_{1-}}{\sqrt{2}}, \quad a_C = \frac{a_{1+} + a_{1-}}{\sqrt{2}}$$

The $\sqrt{2}$ normalisation preserves power. The mixed-mode S-matrix is:

$$\begin{bmatrix} b_{D1} \\ b_{C1} \\ b_{D2} \\ b_{C2} \end{bmatrix} = \begin{bmatrix} S_{dd11} & S_{dc11} & S_{dd12} & S_{dc12} \\ S_{cd11} & S_{cc11} & S_{cd12} & S_{cc12} \\ S_{dd21} & S_{dc21} & S_{dd22} & S_{dc22} \\ S_{cd21} & S_{cc21} & S_{cd22} & S_{cc22} \end{bmatrix} \begin{bmatrix} a_{D1} \\ a_{C1} \\ a_{D2} \\ a_{C2} \end{bmatrix}$$

**Parameter interpretations:**

**Sdd21** — Differential-to-Differential Forward Transmission:

The primary channel metric. It is the insertion loss of the differential signal through the channel, referenced to a 100 $\Omega$ differential system ($Z_0 = 50\ \Omega$ per conductor). This is what compliance tests check. For a well-designed differential pair, $|S_{dd21}|$ should approach the ideal single-ended $|S_{21}|$.

**Scc21** — Common-Mode-to-Common-Mode Forward Transmission:

How efficiently common-mode noise passes through the channel. For a well-balanced differential pair, common-mode signals should be attenuated by the receiver's CMRR. If $|S_{cc21}|$ is large (close to 0 dB), common-mode noise propagates freely — this is not a compliance failure by itself but worsens EMI and susceptibility.

**Sdc21** — Common-Mode-to-Differential Conversion (Mode Conversion):

A common-mode input on the left side generates a differential output on the right. This is the critical **mode conversion** or **balanced-to-unbalanced** parameter. A non-zero $S_{dc21}$ means common-mode noise entering the channel (from power supply noise, crosstalk) appears as differential noise at the receiver — directly degrading the eye diagram. Tight trace-to-trace length matching, symmetric stackup, and careful connector pin assignment minimise $S_{dc21}$.

**Scd21** — Differential-to-Common-Mode Conversion:

A differential signal on the left generates common-mode output on the right. This is an EMI metric: it quantifies how much the channel converts differential signal energy into common-mode radiation. Skew in connector pins, asymmetric via transitions, and unequal trace widths all contribute to $S_{cd21}$.

**Relationship to single-ended matrix** (for a balanced network):

$$S_{dd21} = \frac{S_{21} - S_{23} - S_{41} + S_{43}}{2}$$

(using port numbering 1+, 1-, 2+, 2- = ports 1, 2, 3, 4 respectively in a 4-port single-ended matrix)

---

### Q7. A differential channel measurement shows Sdd21 = -28 dB at the Nyquist frequency and Sdc21 = -25 dB at the same frequency. Is this a problem? How would you diagnose the root cause?

**Answer:**

**Assessing Sdd21 = -28 dB:**

For PCIe Gen 3 (8 GT/s, Nyquist 4 GHz), the maximum allowed insertion loss is 20 dB. At 28 dB loss, the channel fails the PCIe Gen 3 budget and is marginal even for some equaliser implementations. For USB 3.2 Gen 2 (10 GT/s), the limit is 12 dB — this channel fails badly. However, context matters: with equalisers (CTLE + DFE), links often operate at 30+ dB of IL. The question is whether COM (Channel Operating Margin) remains positive after equalisation.

**Assessing Sdc21 = -25 dB:**

Mode conversion of -25 dB is concerning. The ratio of mode-converted noise to the differential signal is:

$$\frac{|S_{dc21}|}{|S_{dd21}|} = \frac{10^{-25/20}}{10^{-28/20}} = \frac{0.0562}{0.0398} \approx 1.4\ (3\ \text{dB})$$

The mode-converted signal is 3 dB *larger* than the attenuated differential signal. This means common-mode noise is being amplified relative to the differential signal through the channel, which will degrade the signal-to-noise ratio at the receiver significantly.

A general guideline is that $|S_{dc21}|$ should be at least 10–20 dB below $|S_{dd21}|$ for the mode conversion to be a minor contributor.

**Diagnostic approach:**

1. **TDR/TDT on each single-ended trace separately.** Measure propagation delay on each leg. A delta-T between the two legs directly creates skew and mode conversion. The mode-conversion magnitude scales as $\sin(\pi \Delta T / T_{bit})$.

2. **Examine Sdd11 and Scc11.** If $S_{dd11}$ shows a sharp resonance (notch in $S_{dd21}$ corresponding to a peak in $S_{dd11}$), there is a discontinuity — likely a via, connector pin, or impedance step. Asymmetric discontinuities (different on + and - legs) are the primary mode-conversion sources.

3. **Compare S21 for each single-ended path** (port 1+ to 2+ vs port 1- to 2-). Any asymmetry in these two paths directly contributes to $S_{dc21}$.

4. **Check connector pin assignment.** Adjacent pins in the same row cause pin field skew. Best practice assigns differential pairs to opposite-side pins (e.g., row A and row B) of the same column.

5. **Via geometry.** A via with an uneven anti-pad or asymmetric stub backdrilling exhibits asymmetric capacitance loading, converting mode.

---

### Q8. Explain the concept of time-domain reflectometry (TDR) and its relationship to S11 measurements.

**Answer:**

**TDR principle:**

A TDR launches a fast-edge step function into the device under test and measures the reflected waveform as a function of time. Each impedance discontinuity along the transmission line reflects a portion of the incident wave. The round-trip delay to the discontinuity is $t = 2d/v_p$, where $d$ is the distance and $v_p$ is the propagation velocity. Knowing $v_p$ (from the dielectric constant), the TDR trace maps directly to the physical layout: peaks and dips in the reflected waveform correspond to capacitive and inductive discontinuities at known distances.

**Mathematical relationship to S11:**

S11 in the frequency domain and the TDR waveform in the time domain are a Fourier transform pair (with appropriate normalisation):

$$s_{11}(t) = \mathcal{F}^{-1}\{S_{11}(f)\} \cdot 2$$

In practice, a VNA measures $S_{11}(f)$ across a frequency range $[f_1, f_2]$ and the time-domain transform is computed by inverse FFT with appropriate windowing. This is called **Time-Domain Reflectometry via VNA (TDR-VNA)** and has superior dynamic range compared to a hardware TDR instrument because the VNA uses a narrowband receiver at each frequency point.

**Bandwidth and spatial resolution:**

The spatial resolution of TDR is limited by the rise time of the step (hardware TDR) or the maximum measurement frequency (VNA-based TDR):

$$\Delta x_{min} = \frac{v_p}{2 \cdot BW} = \frac{c}{2 \sqrt{\varepsilon_r} \cdot f_{max}}$$

For $\varepsilon_r = 4.0$ (FR4), $f_{max} = 20$ GHz:

$$\Delta x_{min} = \frac{3 \times 10^8}{2 \times \sqrt{4} \times 20 \times 10^9} = \frac{3 \times 10^8}{8 \times 10^{10}} \approx 3.75\ \text{mm}$$

Features closer than 3.75 mm cannot be resolved individually at 20 GHz. This is why backdrilling stubs shorter than ~4 mm are difficult to characterise with standard VNA equipment.

---

### Q9. What is causality in the context of S-parameters and why does it matter for simulation?

**Answer:**

**Causality requirement:**

A physical system cannot respond before it is stimulated. In the time domain, the impulse response $h(t) = 0$ for $t < 0$. In frequency-domain terms, this means the real and imaginary parts of $S_{21}(f)$ are Hilbert transform pairs (Kramers-Kronig relations).

**Why causality matters for simulation:**

SI simulators perform channel simulations using convolution of the input signal with the channel's impulse response. If the S-parameter model is non-causal:

1. **Pre-cursor ringing** appears before the signal edge — an artefact, not a physical phenomenon. This inflates pre-cursor ISI estimates.

2. **Eye diagram pessimism or optimism** — depending on how the non-causality manifests, eye height and width predictions can be wrong by 10–20%.

3. **Passivity violations** may accompany non-causality in measured data, causing convergence failures in SPICE-based tools.

**Common causes of non-causal S-parameter data:**

1. **Phase reference errors** in VNA calibration. The port phase reference does not align with the physical port, causing an apparent negative time delay.

2. **Truncated frequency sweep.** If measurement starts at too high a frequency (e.g., 10 MHz instead of DC), the abrupt low-frequency cut-off causes ringing in the IFFT.

3. **Sparse frequency sampling.** Insufficient frequency points cause aliasing in the time domain.

**Checking and correcting causality:**

```python
import numpy as np

# Given S21 complex data at frequencies f_data
# Compute IFFT to get impulse response
h = np.fft.irfft(S21_complex)
time = np.fft.rfftfreq(len(h), d=(f_data[1]-f_data[0]))

# Check for pre-cursor energy
t0_idx = np.argmax(np.abs(h))  # main arrival
pre_energy = np.sum(np.abs(h[:t0_idx])**2)
total_energy = np.sum(np.abs(h)**2)
print(f"Pre-cursor energy fraction: {pre_energy/total_energy:.4f}")
# Values > 0.01 (1%) indicate non-trivial causality issues
```

Correction methods include enforcing the Hilbert relation via minimum-phase reconstruction or applying a time shift to the $S_{21}$ phase to move pre-cursor energy to $t \ge 0$.

---

## Tier 3: Advanced

### Q10. Derive the mixed-mode transformation matrix. Starting from a 4-port single-ended S-matrix, show how to compute Sdd21.

**Answer:**

**Setup:**

Label the four single-ended ports as:
- Port 1: positive terminal, left (input+)
- Port 2: negative terminal, left (input-)
- Port 3: positive terminal, right (output+)
- Port 4: negative terminal, right (output-)

The 4×4 single-ended S-matrix $\mathbf{S}$ relates incident waves $\mathbf{a} = [a_1, a_2, a_3, a_4]^T$ to reflected waves $\mathbf{b} = [b_1, b_2, b_3, b_4]^T$:

$$\mathbf{b} = \mathbf{S}\,\mathbf{a}$$

**Mixed-mode wave definitions:**

Define differential (D) and common-mode (C) waves at each end:

$$a_{D1} = \frac{a_1 - a_2}{\sqrt{2}}, \quad a_{C1} = \frac{a_1 + a_2}{\sqrt{2}}$$
$$a_{D2} = \frac{a_3 - a_4}{\sqrt{2}}, \quad a_{C2} = \frac{a_3 + a_4}{\sqrt{2}}$$

This defines a transformation matrix $\mathbf{M}$:

$$\mathbf{a}_{mm} = \mathbf{M}\,\mathbf{a}_{se}, \quad \mathbf{a}_{mm} = [a_{D1}, a_{C1}, a_{D2}, a_{C2}]^T$$

$$\mathbf{M} = \frac{1}{\sqrt{2}}\begin{bmatrix} 1 & -1 & 0 & 0 \\ 1 & +1 & 0 & 0 \\ 0 & 0 & 1 & -1 \\ 0 & 0 & 1 & +1 \end{bmatrix}$$

The inverse transformation is $\mathbf{M}^{-1} = \mathbf{M}^T$ (since $\mathbf{M}$ is orthogonal, $\mathbf{M}\mathbf{M}^T = \mathbf{I}$).

**Mixed-mode S-matrix:**

$$\mathbf{S}_{mm} = \mathbf{M}\,\mathbf{S}_{se}\,\mathbf{M}^T$$

**Extracting Sdd21:**

$S_{dd21}$ is the element in row 3, column 1 of $\mathbf{S}_{mm}$ (output differential = row, input differential = column, with the ordering D1, C1, D2, C2):

Expanding the matrix product for the (3,1) element:

$$S_{dd21} = \frac{1}{2}\left(S_{31} - S_{32} - S_{41} + S_{42}\right)$$

**Physical check:** If the network is perfectly balanced and symmetric ($S_{31} = S_{42}$, $S_{41} = S_{32} = 0$ for a lossless differential line), then:

$$S_{dd21} = \frac{1}{2}(S_{31} - 0 - 0 + S_{31}) = S_{31}$$

which confirms that $S_{dd21}$ equals the single-ended through transmission of one conductor in the ideal case — as expected, since in a balanced differential system the two conductors each carry half the differential signal.

**Python implementation:**

```python
import numpy as np

def single_ended_to_mixed_mode(S_se):
    """
    Convert 4x4 single-ended S-matrix to 4x4 mixed-mode S-matrix.
    Port ordering convention:
      Single-ended: [1+, 1-, 2+, 2-] = indices [0, 1, 2, 3]
      Mixed-mode output: [Dd1, Cc1, Dd2, Cc2] = indices [0, 1, 2, 3]
    """
    M = (1/np.sqrt(2)) * np.array([
        [ 1, -1,  0,  0],
        [ 1,  1,  0,  0],
        [ 0,  0,  1, -1],
        [ 0,  0,  1,  1],
    ], dtype=complex)

    S_mm = M @ S_se @ M.T
    return S_mm

# Example: extract Sdd21 from frequency-swept data
# S_se_f has shape (N_freq, 4, 4)
def extract_Sdd21(S_se_f):
    Sdd21 = []
    for S in S_se_f:
        S_mm = single_ended_to_mixed_mode(S)
        Sdd21.append(S_mm[2, 0])  # row=2 (D output), col=0 (D input)
    return np.array(Sdd21)

# Verify with simple symmetric case
S_symmetric = np.array([
    [0.05,  0.0,  0.9,  0.0],
    [0.0,  0.05,  0.0,  0.9],
    [0.9,  0.0,  0.05, 0.0],
    [0.0,  0.9,  0.0,  0.05],
], dtype=complex)

S_mm = single_ended_to_mixed_mode(S_symmetric)
print(f"Sdd21 = {S_mm[2,0]:.4f}")  # Expected: 0.9
print(f"Scc21 = {S_mm[3,1]:.4f}")  # Expected: 0.9
print(f"Sdc21 = {S_mm[2,1]:.4f}")  # Expected: 0.0 (no mode conversion)
```

Expected output:
```
Sdd21 = 0.9000+0.0000j
Scc21 = 0.9000+0.0000j
Sdc21 = 0.0000+0.0000j
```

---

### Q11. A 4-port S-parameter measurement of a differential pair shows a pronounced notch in Sdd21 at 7.5 GHz. Describe the systematic investigation you would perform to identify and fix the root cause.

**Answer:**

A notch (sharp minimum) in $|S_{dd21}|$ at a specific frequency indicates a resonant cancellation — energy at that frequency is being reflected or diverted rather than transmitted. The primary suspects are:

**Step 1 — Correlate with S11 (resonance vs. absorption).**

- If $|S_{dd11}|$ shows a *peak* at 7.5 GHz (high reflection), the cause is a reflective discontinuity acting as a resonant filter — typically a via stub or cavity resonance.
- If $|S_{dd11}|$ shows no peak (remains small), the energy is being dissipated or coupled elsewhere — suspect a crosstalk-induced null or mode-conversion resonance.

**Step 2 — Calculate resonant stub length.**

Via stubs are the most common cause of sharp notches. The stub resonates at:

$$f_{notch} = \frac{v_p}{4 \cdot L_{stub}} = \frac{c}{4 L_{stub} \sqrt{\varepsilon_{eff}}}$$

Solving for stub length at 7.5 GHz with $\varepsilon_{eff} = 4.0$:

$$L_{stub} = \frac{c}{4 f_{notch} \sqrt{\varepsilon_{eff}}} = \frac{3 \times 10^8}{4 \times 7.5 \times 10^9 \times 2} = 5\ \text{mm}$$

Check whether the PCB has vias with unbackdrilled stubs of approximately 5 mm. Given typical PCB layer counts, a 12-layer board with signal on layer 3 and the via continuing to layer 12 through a 2.4 mm board thickness gives a stub on the order of $2.4 \times (12-3)/12 \approx 1.8$ mm — adjust for actual board and routing layer.

**Step 3 — TDR analysis to localise.**

Perform a TDR (via VNA IFFT) on the differential channel. The notch frequency corresponds to a time-domain feature at:

$$t_{stub} = \frac{2 L_{stub}}{v_p} = \frac{2 \times 5 \times 10^{-3}}{1.5 \times 10^8} \approx 67\ \text{ps}$$

A reflection peak at 67 ps after the main through-path arrival identifies the stub location. Cross-reference with the cumulative propagation delay to locate the via on the board.

**Step 4 — Mode-conversion check.**

Plot $|S_{dc21}|$. If the notch in $S_{dd21}$ is accompanied by a peak in $|S_{dc21}|$ at the same frequency, the differential signal energy is being converted to common mode at the resonance — indicating an asymmetric discontinuity (one via stub longer than the other, offset anti-pads, etc.).

**Step 5 — Fix options in priority order:**

1. **Backdrilling:** Remove the via stub. Reduces via inductance and capacitance simultaneously. Most effective fix; typically restores the notch from -25 dB to $\le -3$ dB.

2. **Via-in-pad:** Move to via-in-pad geometry which eliminates the stub entirely.

3. **Equalisation:** If a PCB rework is not feasible, configure RX CTLE boost at 7.5 GHz to compensate for the notch. However, a resonant null deeper than -20 dB may exceed the CTLE's ability to compensate and will leave residual ISI.

4. **Board redesign routing layer:** Route the critical signal on an inner layer closer to the connector, shortening the via stub.

---

## Quick Reference: Key Terms

| Term | Definition |
|---|---|
| S-parameter | Scattering parameter relating travelling wave amplitudes at network ports |
| S11 | Input reflection coefficient; relates to return loss |
| S21 | Forward transmission; relates to insertion loss |
| S12 | Reverse transmission; equals S21 for reciprocal networks |
| S22 | Output reflection coefficient |
| Insertion Loss | $IL = -20\log_{10}|S_{21}|$ dB; larger value = more loss |
| Return Loss | $RL = -20\log_{10}|S_{11}|$ dB; larger value = better match |
| $\Gamma$ | Voltage reflection coefficient; $\Gamma = (Z_L - Z_0)/(Z_L + Z_0)$ |
| Touchstone (.sNp) | Standard file format for N-port S-parameter data |
| Mixed-mode | Transformation of single-ended S-params into differential/common-mode basis |
| Sdd21 | Differential insertion loss; primary channel quality metric |
| Scc21 | Common-mode transmission through channel |
| Sdc21 | Mode conversion: common-mode in, differential out; EMI and noise metric |
| Scd21 | Mode conversion: differential in, common-mode out; EMI radiation metric |
| TDR | Time-Domain Reflectometry; localises impedance discontinuities |
| Via stub resonance | Notch in S21 caused by quarter-wave resonance of unterminated via barrel |
| Causality | Physical requirement that $h(t) = 0$ for $t < 0$; must hold for valid SI models |
