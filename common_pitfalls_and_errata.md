# Common SI/PI Pitfalls and Errata

## Overview

This document is a curated list of SI/PI misconceptions, formula errors, spec misquotes, and design traps that repeatedly appear in textbooks, blogs, and even seasoned engineers' work. Each entry is a "gotcha" — something that looks plausible, passes quick inspection, and causes real failures. Interviewers use these as discriminators between candidates who learned SI/PI from a book and candidates who have worked through lab failures. Most items cite the senior source that corrected the record.

---

## PI Pitfalls

### 1. "Target impedance alone is sufficient for PDN design."

**Wrong because:** $|Z(f)| \le Z_{target}$ at every frequency is a necessary condition but not a sufficient one. Sharp resonant peaks at $Z_{target}$ can cause rail noise to exceed $\Delta V_{max}$ when the load current has periodic content at the resonance — the "rogue wave" phenomenon (Smith & Sandler, *Signal Integrity Journal*, 2019). Always add a **flatness constraint** (peak-to-minimum ratio ≤ 6 dB) and budget for periodic-excitation build-up.

### 2. "Use the MLCC's nameplate capacitance in the PDN simulation."

**Wrong because:** Class-II MLCCs (X5R, X7R) have significant DC-bias derating — a 10 µF 0402 X5R rated 10 V running at 1 V bias may deliver only 2–5 µF; at its rated voltage only 1–3 µF. Always use the manufacturer's S-parameter model (Murata SimSurfing, TDK SEAT) or apply a 50–80% derating factor. Simulating with nameplate values over-predicts PDN capability by 2–5×.

### 3. "Adding more PCB caps fixes high-frequency rail noise."

**Wrong because:** above the package-inductance-limited frequency (typically 10–50 MHz), PCB caps cannot see the die — the package's vertical inductance creates a hard bandwidth limit. The fix is on-package decoupling (OPD) or on-die capacitance, not more PCB MLCCs. See `03_power_integrity/package_effects_on_pi.md` for the derivation.

### 4. "Plane resonance only matters near the first mode."

**Wrong because:** higher-order modes pile up fast and are Q-dominated. The second, third, and fourth modes often overlap within a factor of 2 of the first, producing a dense forest of peaks rather than isolated resonances. Simulate the full modal spectrum across the operating band.

### 5. "Load current is a single step of amplitude $I_{max}$."

**Wrong because:** real load currents are periodic (clock harmonics, activity patterns). The spectrum concentrates at discrete frequencies. Designing for a step is conservative for broadband noise but ignores resonant build-up at specific frequencies. Characterise the workload current spectrum before setting the PDN target.

### 6. "ESR is bad — minimise it."

**Wrong because:** ESR provides damping. Zero-ESR caps cause high-Q anti-resonance peaks. Mixed-ESR designs (a few higher-ESR caps deliberately included) are superior for flatness. Polymer aluminium caps with naturally high ESR (5–30 m$\Omega$) are the classic "damper" in a staged PDN.

### 7. "The VRM's closed-loop output impedance is low everywhere below its loop BW."

**Wrong because:** even within the VRM loop bandwidth, compensator design can produce peaks. Worse, under transient load the VRM can slew-rate saturate and briefly lose regulation, spiking output impedance. Use NISM (Non-Invasive Stability Measurement, Picotest) to characterise the VRM under realistic load — passive measurement is not enough.

---

## SI Pitfalls

### 8. "Route high-speed traces 3× the trace width apart and crosstalk is solved."

**Wrong because:** 3W rules are rough guidelines only. Actual coupling depends on dielectric thickness, trace geometry, coupled length, and rise time. For stripline at 56 Gb/s, 3W may be nowhere near adequate; for microstrip with short coupled segments at 1 Gb/s, 1W may suffice. Always simulate with a field solver.

### 9. "NEXT and FEXT add at the receiver."

**Wrong because:** NEXT energy travels back toward the aggressor's source, not the victim's receiver. FEXT travels forward with the victim signal. At the victim's receiver, only FEXT and a reflected fraction of NEXT (from aggressor-side mismatches) appear. They do not simply add.

### 10. "Insertion loss is the only metric that matters for channel quality."

**Wrong because:** a channel with low IL but high return loss still produces large BER via reflections. A channel with flat IL but notches at specific frequencies (stubs, fibre-weave) also fails despite good "average" loss. Modern compliance uses **COM (Channel Operating Margin)** — a statistical metric that captures IL, RL, crosstalk, jitter, and equaliser capability together.

### 11. "Differential signalling is immune to common-mode noise."

**Wrong because:** differential is only immune if (a) the P and N legs are perfectly matched and (b) the receiver has infinite CMRR. Real receivers have 30–60 dB CMRR, and real pairs have skew/asymmetry that converts common-mode noise into differential. $S_{cd21}$ captures this; it must be < −20 dB for clean operation.

### 12. "Glass-weave skew does not matter because the PCB $D_k$ is an average."

**Wrong because:** the macroscopic $D_k$ is an average, but traces see the *local* $D_k$ at the scale of their cross-section. A trace over a glass bundle sees $D_k \approx 5–6$; over a resin window sees $D_k \approx 3$. Intra-pair skew accumulates over trace length. For 25 Gb/s and above, glass-weave mitigation is mandatory.

### 13. "A VNA with DC-to-20 GHz sweep is fine for any 25 Gb/s channel."

**Wrong because:** 25 Gb/s NRZ has significant spectral content out to $3 \times f_{Nyq}$ = 37.5 GHz (rise time contains higher harmonics). A 20 GHz sweep truncates important frequency content and causes aliasing in the time-domain IFFT. **Measurement bandwidth must exceed 2–3× the signalling Nyquist**.

### 14. "Treat S-parameter files as pure data — they are what the simulator uses."

**Wrong because:** S-parameter files can be non-causal, non-passive, and non-reciprocal due to measurement imperfections. Simulators feed directly from them, and non-physical file properties produce non-physical simulation results (pre-cursor ringing, divergence). Always validate causality, passivity, and reciprocity before simulating.

### 15. "CTLE peaking gain is the absolute gain at the peak."

**Wrong because:** CTLE "peaking" specifically means the *gain relative to DC*. A CTLE with DC gain −6 dB and peak absolute gain +8 dB has a peaking of 14 dB. Confusing absolute vs. relative gain leads to incorrect CTLE selection in link simulation.

---

## Measurement Pitfalls

### 16. "A scope with 20 GHz bandwidth can measure a 25 Gb/s signal."

**Wrong because:** Nyquist for 25 Gb/s NRZ is 12.5 GHz, but the rise-time content extends well beyond. Scope bandwidth should be 3–5× Nyquist to accurately capture the edge. Real 25 Gb/s compliance needs 50 GHz scopes.

### 17. "Two-channel software subtraction gives the same result as a differential probe."

**Wrong because:** channel-to-channel skew (even 1 ps) and gain mismatch (0.5% or worse) destroy CMRR. A 1% gain mismatch gives only 40 dB CMRR; above 500 MHz the mismatch grows with frequency. A dedicated differential probe achieves 40–60 dB CMRR across a much wider bandwidth.

### 18. "TDR resolution is set by the step rise time — faster step, better resolution."

**Wrong because:** TDR spatial resolution is set by the system bandwidth, which is the smaller of the source rise time and the measurement system's bandwidth. A 20 GHz sampling scope cannot extract features smaller than $\sim$7 mm on FR4, regardless of the step rise time.

### 19. "2X-thru de-embedding removes all fixture effects."

**Wrong because:** 2X-thru works only if the 2X-thru coupon is *truly identical* to the fixture. Different trace widths, different routing layers, or different connectors destroy the assumption. Use IEEE P370 with quality metrics to detect and correct these issues.

---

## Specification and Industry Pitfalls

### 20. "PCIe Gen 4 maximum channel IL is 36 dB."

**Wrong because:** this number appears in some secondary sources but the PCIe base spec channel budgets at 8 GHz are mask-based and typically ~28 dB total for the CEM reference channel. Always consult the current PCI-SIG base specification for the authoritative frequency-dependent mask.

### 21. "LPDDR5 WCK runs at 2× the data rate."

**Wrong because:** LPDDR5 WCK is a DDR (double-data-rate) clock that captures data on both edges. Its frequency is at or below the data rate (typically the data rate /2) depending on the WCK:CK ratio selected. The phrase "2× data rate" is a common misconception.

### 22. "FR4 has $D_f = 0.022$."

**Wrong because:** "FR4" is a class of materials ranging from $D_f = 0.015$ (high-Tg low-loss FR4) to 0.025 (cheap commodity FR4 at low frequencies). Modern "standard FR4" at 10 GHz has $D_f \approx 0.017$–$0.020$. Using 0.022 uniformly overestimates dielectric loss in most designs.

### 23. "Ethernet 100GBASE-CR4 is PAM4."

**Wrong because:** 100GBASE-CR4 is 4 lanes × 25 Gbps **NRZ** (IEEE 802.3bj). The 2-lane PAM4 variant is 100GBASE-CR2 (rare in practice). Be careful with standard names; many permutations exist with subtle encoding differences.

### 24. "PCIe SSC adds jitter that the CDR must reject."

**Wrong because:** the CDR *must track* the SSC (standard requires CDR loop BW covers 33 kHz SSC rate). If the CDR rejects SSC instead of tracking, the data edges drift relative to the CDR sampling clock and cause BER degradation. The CDR loop bandwidth is specified precisely so SSC stays inside the tracking bandwidth.

---

## Design Pitfalls

### 25. "Add bulk decoupling, mid-frequency decoupling, and high-frequency decoupling — that covers everything."

**Wrong because:** the classical three-tier cap strategy often leaves gaps at tier-to-tier handoff frequencies where anti-resonance peaks. Use frequency-overlapping stages (five or six cap values, not three) and verify with an impedance simulation.

### 26. "Place bulk caps near the VRM to reduce the distance the VRM must drive."

**Wrong because:** bulk caps are primarily loop-isolators for VRM loop dynamics; their placement affects VRM stability, not high-frequency PDN. The caps that matter for decoupling at tens-of-MHz and above are the MLCCs near the load, not the bulk. Bulk cap placement is governed by VRM compensation, not "distance to load".

### 27. "A dense ground fill on unused board area improves SI."

**Wrong because:** floating ground fills (not connected to the ground plane via many stitch vias) are **antennas** that couple noise between signals. They can degrade SI rather than improve it. Either stitch the fill densely (every 5 mm) or do not add it.

### 28. "Route the most critical signals on an outer layer for easy debug access."

**Wrong because:** outer-layer microstrip has air as one reference dielectric, causing asymmetric loss and mode conversion. It is also more susceptible to external EMI and more sensitive to solder-mask and conformal-coat variations. High-speed critical signals belong on inner stripline layers.

### 29. "Differential impedance is just twice the single-ended impedance."

**Wrong because:** differential impedance is $Z_{diff} = 2Z_0(1 - k)$ where $k$ is the coupling coefficient (0 for un-coupled, positive for edge-coupled pairs). For typical PCB differential pairs with 4–10 mil spacing, $k \approx 0.1$–$0.2$, so $Z_{diff} = 80$–$90\ \Omega$ for nominally 50 Ω single-ended, not 100 Ω.

### 30. "Skin-depth scaling is linear with frequency."

**Wrong because:** skin depth $\delta = 1/\sqrt{\pi f \mu \sigma}$ scales as $1/\sqrt{f}$, not $1/f$. AC resistance $R_s \propto \sqrt{f}$, and conductor loss $\alpha_c \propto \sqrt{f}$. A 10× frequency increase gives $\sqrt{10} \approx 3.16$× loss, not 10×.

---

## Quick-Reference Sanity Checks

Before reporting a number, verify:

| Claim type | Sanity check |
|---|---|
| Conductor loss at 10 GHz for 50Ω stripline | Expected ~0.15–0.30 dB/inch |
| Dielectric loss at 10 GHz for FR4 | Expected ~0.8–1.0 dB/inch |
| Dielectric loss at 10 GHz for Megtron 6 | Expected ~0.15–0.25 dB/inch |
| PCB propagation velocity (FR4) | Expected 6.5–7 in/ns stripline |
| Typical PCIe Gen 5 via | Expected ~0.3 dB (backdrilled) to ~1.5 dB (not) |
| Typical SRF of 100 nF 0402 MLCC (mounted) | Expected 10–20 MHz |
| Typical package inductance per power pin | Expected 200 pH (flip-chip) to 5 nH (bond wire) |
| Typical on-die VDD sensitivity | Expected ~1 ps/mV (clock tree) |
| Typical PCB FR4 $D_k$ at 10 GHz | Expected 4.0–4.3 |

If a calculation produces a result 10× off any of these rules, **recheck the units** — almost always the error is a decimal place, a conversion (mil ↔ mm, GHz ↔ Hz), or a dropped factor of $2\pi$.

---

## Key References for Deeper Reading

- **Eric Bogatin**, *Signal and Power Integrity — Simplified* (3rd ed., Pearson 2017). The canonical textbook; "20 Rules for Engineers" articles in *Signal Integrity Journal*.
- **Howard Johnson, Martin Graham**, *High-Speed Digital Design: A Handbook of Black Magic* (1993) and *High-Speed Signal Propagation* (2003). The fundamental SI references.
- **Larry Smith, Eric Bogatin**, *Principles of Power Integrity for PDN Design — Simplified* (Prentice Hall, 2017). Target impedance and flatness.
- **Steve Sandler**, *Power Integrity: Measuring, Optimizing, and Troubleshooting Power Related Parameters* (McGraw-Hill, 2014). Measurement and NISM.
- **Yuriy Shlepnev** (Simberian), DesignCon papers on S-parameter quality metrics.
- **Bert Simonovich** (LAMSIM), blog posts on fibre-weave, Cannonball-Huray, and stackup design.
- **IEEE P370-2020**, *Electrical Characterization of Printed Circuit Board and Related Interconnects at Frequencies up to 50 GHz*.
- **PCI-SIG**, PCIe Base Specification revisions 4.0 through 7.0.
- **JEDEC**, JESD79 (DDR4), JESD79-5 (DDR5), JESD209-5 (LPDDR5), JESD270-4 (HBM4).
