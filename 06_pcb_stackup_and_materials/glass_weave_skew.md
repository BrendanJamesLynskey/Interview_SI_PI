# Glass-Weave Skew (Fiber-Weave Effect)

## Overview

PCB laminate is not a homogeneous dielectric. It is a composite: woven glass fibre bundles (high $D_k$, roughly 6) embedded in a resin matrix (low $D_k$, roughly 3). Traces that happen to run directly over a glass bundle see a different effective $D_k$ than traces that run over a resin-rich region, and the propagation velocity differs accordingly. In a differential pair, if one conductor rides over glass while the other rides over resin, the intra-pair skew accumulates along the trace length, converts differential signal into common mode, and degrades the eye. This is the **fiber-weave effect** (also called glass-weave skew or woven-glass skew). Above ~28 Gb/s, it is one of the leading causes of channel asymmetry, and mitigating it is a required design topic on any SerDes role. This document covers the physics, quantification, and mitigation of fiber-weave skew. Key reference: Bert Simonovich's extensive blog and DesignCon papers (blog.lamsimenterprises.com).

---

## Tier 1: Fundamentals

### Q1. What is the fiber-weave effect and why does it cause differential pair skew?

**Answer:**

**PCB laminate construction:**

A PCB core or prepreg consists of woven glass fibre cloth (E-glass, style coded e.g. 1080, 2116, 3313) impregnated with an epoxy (or PPE, or PTFE) resin. The glass has $D_k \approx 6$; the resin has $D_k \approx 3$. The macroscopic "laminate $D_k$" that appears in datasheets is a volume-weighted average, typically 3.5–4.5.

**At the scale of a trace (a few mils wide), the dielectric is not homogeneous:**

- Trace routed directly over a glass bundle: local dielectric environment has $D_k$ biased toward the glass value (~5–6).
- Trace routed over a resin-rich window (the "knuckle" between fibre bundles): local dielectric biased toward the resin value (~3).

The two conductors of a differential pair, separated by a few mils, can independently happen to run over glass or resin. If the pair runs parallel to the fibre warp (or weft) direction, and the two conductors sit on different glass/resin regions *for long stretches*, they see different $D_k$ and therefore different propagation velocities:

$$v_p = \frac{c}{\sqrt{D_{k,eff}}}$$

**Skew accumulation:**

Per unit length, the delay difference between "over-glass" and "over-resin" is:

$$\Delta t_d \approx T_{pd,0}\left(\sqrt{D_{k,glass}} - \sqrt{D_{k,resin}}\right)/\sqrt{D_{k,avg}}$$

For typical FR4 numbers ($D_{k,glass} = 6$, $D_{k,resin} = 3$, $D_{k,avg} = 4.3$): $\Delta t_d \approx T_{pd,0} \times 0.15$ — that is, **roughly 15% of the nominal propagation delay difference**.

For a 10-inch trace with nominal $T_{pd} = 170$ ps/inch → 1.7 ns total delay, the worst-case fibre-weave-induced skew is up to 0.15 × 1700 ps = **255 ps**. On a 28 Gb/s differential channel (UI = 35 ps), this is **7× the UI** — catastrophic.

Real-world fibre-weave skew is usually much smaller because the trace does not consistently stay over the same region for the full length — it crosses many glass bundles. Measured values are typically 5–30 ps for 10 inches of FR4-1080 glass style, still significant at high data rates.

---

### Q2. Why is fiber-weave skew a bigger problem at 28 Gb/s and above than at 10 Gb/s?

**Answer:**

Fiber-weave skew is a fixed time (in picoseconds per inch) for a given stackup and routing direction; it does not scale with data rate.

At 10 Gb/s:
- UI = 100 ps.
- 10 inches of FR4 might introduce 10–20 ps of fibre-weave skew.
- Skew as fraction of UI: 10–20% — noticeable but not catastrophic. Compensation by design margin.

At 28 Gb/s:
- UI = 35 ps.
- Same 10 inches of FR4 still introduces 10–20 ps.
- Skew as fraction of UI: 30–60% — channels routinely fail compliance.

At 56 Gb/s PAM4 (28 Gbaud):
- UI (symbol) = 36 ps, but the eye-width budget for PAM4 is much tighter.
- Even 5 ps of skew converts significant differential energy to common mode.

At 112 Gb/s PAM4 (56 Gbaud):
- UI (symbol) = 18 ps.
- Fibre-weave skew of even 2 ps per inch is intolerable; mitigation is mandatory.

**The general rule:** fibre-weave skew matters when it approaches 0.1 UI. This threshold is crossed at ~14 Gb/s for long channels; by 28 Gb/s, mitigation is standard; by 56+ Gb/s, mitigation is mandatory.

---

### Q3. What is the effect of fibre-weave skew on a differential channel in measurable terms?

**Answer:**

**1. Mode conversion (differential → common mode):**

Intra-pair skew directly produces mode conversion. The magnitude:

$$|S_{cd21}|(f) \approx \sin(\pi f \Delta t_{skew})$$

For $\Delta t_{skew} = 10$ ps, at 14 GHz: $|S_{cd21}| = \sin(\pi \cdot 14 \cdot 10 / 1000) = \sin(0.44) \approx 0.42$ (−7.5 dB) — that is, −7.5 dB of differential energy converts to common mode at Nyquist. This is a *large* amount and directly fails most SerDes compliance tests.

**2. Intra-pair skew degrades the eye:**

At the receiver slicer, the P and N legs arrive at different times, creating a skewed differential edge. The edge rate is slower, the eye height shrinks, and the BER floor rises.

**3. Common-mode signal couples to EMI:**

The converted common-mode signal flows on cable shields, system ground returns, and connector shells — causing radiated EMI in the tens of MHz to a few GHz range, frequencies that regulatory EMC testing specifically monitors.

**4. Aggravated crosstalk sensitivity:**

A partially mode-converted signal is more susceptible to common-mode interference from aggressor traces on nearby layers, compounding the eye closure.

---

## Tier 2: Intermediate

### Q4. List the main mitigation techniques for fibre-weave skew.

**Answer:**

In order of typical effectiveness and cost:

**1. Spread-glass fabric (Mechanical weave "spreading", a.k.a. "Mech Sub" or "flat glass"):**

Standard glass cloths have knuckles (regions where warp and weft bundles cross and the resin pocket is thickest). Spread-glass (or "square-weave") cloths mechanically flatten the bundles so the glass distribution is more uniform — the "windows" between bundles are smaller.

- Standard 1080 glass has ~50 µm resin windows.
- Spread 1080 ("1080 spread" or "2113 spread") has ~10–20 µm windows.
- "Mech spread" or ultra-spread versions approach homogeneity.

Spread glass can reduce fibre-weave skew by **2–5×** compared to standard glass. It is widely available from all major laminate vendors (Panasonic Megtron 6/7N, Isola I-Tera, Rogers Tachyon) typically at a 5–15% cost premium.

**2. Multi-ply stackup — stack multiple glass plies with different weaves:**

Combining two glass cloths of different weave styles (e.g., 1080 + 3313) ensures that the glass pattern seen by the trace is more random. Even if one ply has a resin-rich line under the P conductor, the other ply's pattern is different and the average is more homogeneous. Cost: slight (extra ply). Reduction: **1.5–2×** depending on ply combination.

**3. Dk-matched glass and resin:**

Use a resin system with higher intrinsic $D_k$ and a glass cloth with lower intrinsic $D_k$ so the contrast is reduced. For example, some Panasonic Megtron products use low-$D_k$ glass (NE-glass with $D_k \approx 4.4$ rather than E-glass at 6.0) and resin with $D_k \approx 3.5$. The contrast ratio drops from 2:1 to ~1.25:1.

Reduction: **2–4×** vs standard E-glass. Cost: moderate to high.

**4. Zig-zag (rotated) trace routing:**

Routing the trace at a small angle (10–20°) relative to the board weave direction ensures that any one conductor of the pair does not stay over a single glass bundle for more than a few mm. Each conductor averages over many glass bundles, and their averages converge.

Reduction: **3–5×**, depending on angle and length. Cost: **zero** (layout-only). The main drawbacks are slightly longer trace length (minor), and routing must not align with existing board grid in dense designs.

**5. Panel rotation during fabrication:**

Asking the fab to rotate the panel 10° relative to the trace direction achieves the same effect as zig-zag routing but without changing the Gerber. Panel rotation increases raw material waste by a few percent (the fab must cut rectangular boards from a rotated rectangle). Typical fab premium: ~5%.

**6. Diagonal routing:**

Routing the pair at 45° to the glass direction maximises the number of glass bundle crossings per unit length. Similar effect to zig-zag but simpler for the layout tool.

**7. Symmetric stackup and signal direction selection:**

If glass weave direction is known, route the most critical pairs perpendicular (or at 45°) to the warp direction. This is sometimes done only for the most critical lanes when full-board mitigation is too expensive.

---

### Q5. How do you measure fibre-weave skew in a fabricated board?

**Answer:**

**Test structure:**

A fibre-weave test coupon is a series of identical differential pairs routed in different directions (parallel to warp, parallel to weft, 45°) and at multiple locations on a single panel. Each pair terminates in probe launches for TDR/TDT or VNA measurement.

**Measurement procedure:**

1. **TDR on each single-ended leg:** measure the one-way propagation delay on P and on N conductors of a single pair. The difference is the intra-pair skew for that trace.
2. **Repeat over many traces (same weave direction, different board locations):** the *statistical spread* of skew values across many traces is the fibre-weave skew distribution. The maximum observed skew is the worst-case that design must handle.
3. **Compare orientations:** traces parallel to the weave have the largest skew; 45° traces have the smallest. The ratio quantifies the mitigation benefit.
4. **VNA $S_{cd21}$ across frequency:** mode conversion correlates with intra-pair skew. $|S_{cd21}|$ peaks at $f = 1/(2\Delta t_{skew})$.

**Typical measured values (10-inch differential pair, standard FR4 1080 glass):**

| Orientation | Median skew | Worst-case skew |
|---|---|---|
| Parallel to weave | 15 ps | 35 ps |
| Perpendicular to weave | 8 ps | 20 ps |
| 10° rotated | 5 ps | 10 ps |
| 45° diagonal | 2 ps | 5 ps |

For 28 Gb/s links these would translate into $|S_{cd21}|$ of $-3$ dB (parallel weave) to $-25$ dB (diagonal) — a 20+ dB improvement from routing direction alone.

---

### Q6. Which glass fabric styles are used in modern high-speed PCB stackups, and what are their fibre-weave characteristics?

**Answer:**

Glass cloth styles are identified by a 4-digit IPC-4412 code. The lower digits generally indicate thinner, denser weaves with fewer resin windows — better for SI.

| Style | Thickness | Warp ×weft yarn | Typical use | Fibre-weave severity |
|---|---|---|---|---|
| 106 | 1.4 mil | Very fine, dense | Thin prepreg | Very low (bundles very small) |
| 1078 | 2.1 mil | 53 × 53 yarn/in | Thin prepreg | Low |
| 1080 | 2.5 mil | 60 × 47 yarn/in | Common high-speed prepreg | **Medium** (largest resin windows) |
| 2113 | 2.9 mil | 60 × 56 yarn/in | Medium prepreg | Medium |
| 2116 | 4.0 mil | 60 × 58 yarn/in | General purpose | Medium |
| 3313 | 3.2 mil | 60 × 62 yarn/in | Core and prepreg | Medium-low |
| 7628 | 6.8 mil | 44 × 32 yarn/in | Thick cores | High (large bundles) |

**"Spread glass" versions** of each style have knuckles flattened and are preferred for high-speed designs. When a laminate datasheet says "spread glass 1078" or "Mech 2113", it means spread-weave.

**Modern high-speed stackup practice:**

- Use 1067 or 1078-spread prepreg for signal-adjacent layers.
- Avoid 7628 glass in signal layers (use it only in thick core for mechanical reasons).
- For the most demanding designs (>56 Gb/s), combine multiple thin spread-glass plies for a homogeneous composite dielectric.

---

## Tier 3: Advanced

### Q7. A 56 Gb/s PAM4 channel shows a 4 dB $S_{cd21}$ peak at 10 GHz. Estimate the fibre-weave skew and propose mitigation.

**Answer:**

**Estimate skew from $S_{cd21}$:**

$|S_{cd21}|(f) \approx \sin(\pi f \Delta t)$. With $|S_{cd21}| = 10^{-4/20} = 0.63$ at $f = 10$ GHz:

$$\Delta t = \frac{\arcsin(0.63)}{\pi \cdot 10\ \text{GHz}} = \frac{0.68}{\pi \cdot 10^{10}} \approx 22\ \text{ps}$$

For a 10-inch trace, this is 2.2 ps/inch of fibre-weave skew. Worst-case for standard 1080 glass parallel to weave direction.

**Is 22 ps acceptable at 56 Gb/s PAM4?**

UI = 36 ps. 22 ps of skew is 0.6 UI — the differential signal is unrecognisable to the receiver. **This channel fails.**

**Mitigation, in order of cost-effectiveness:**

1. **Route at 45° to weave (zig-zag):** should reduce skew to ~2–5 ps. Biggest impact, zero cost.
2. **Switch to spread-glass prepreg (1080-spread or 1067):** reduces skew by 2–3×. Cost: ~10% of PCB cost.
3. **Multi-ply stackup (1067 + 2113):** averages weave patterns. Further 1.5× reduction. Small cost impact.
4. **$D_k$-matched glass-resin system:** switch to Megtron 7N or similar with low $D_k$ contrast. Significant cost increase.

A combined approach (spread-glass 1067 prepreg + 10° rotation + multi-ply) should bring skew below 2 ps/inch → total 2–3 ps over 10 inches. At 56 Gb/s, $|S_{cd21}|$ at 10 GHz would drop to about $-30$ dB — well within compliance.

**Production consideration:**

Budget 2–3 weeks for PCB fab re-spin to introduce the weave mitigation, and verify with a new test coupon before declaring fixed.

---

### Q8. Does fibre-weave skew affect single-ended signals and power/ground planes?

**Answer:**

**Single-ended signals:**

Yes — the local $D_k$ variation changes the propagation velocity along a single trace. The effect manifests as:

1. **Trace-to-trace $D_k$ variation across the board** → impedance tolerance degrades (traces on glass-rich regions have lower $Z_0$ than traces on resin-rich regions).
2. **Delay tolerance between nominally-matched single-ended groups** (e.g., DDR DQ lanes) — length-matched traces may have different propagation delays if they happen to run over different dielectric mixtures.

For most single-ended designs, this second-order effect is absorbed into the impedance tolerance budget (typically ±10%) and is rarely a root cause of failures. But for DDR5 at high speeds, it contributes to the system-level skew budget and may force tighter length-match.

**Power/ground planes:**

Generally unaffected at PI-relevant frequencies because the plane dimensions are much larger than the glass weave period. The plane cavity mode frequencies depend on the average $D_k$, which is well-defined at the plane scale. Local $D_k$ variations wash out when the field is distributed across a large plane area.

**Exceptions:**

- **Very small plane islands or moats:** a 1-cm² isolated power region may see noticeable $D_k$ variance, but in practice this is dwarfed by the variability of the decoupling network.
- **High-frequency plane-pair transmission-line analysis** (for EBG or specific-geometry plane designs) above ~20 GHz — here the $D_k$ granularity starts to matter.

For ordinary PI work up to 5 GHz, ignore fibre-weave effects on planes and focus them on signals.

---

## Summary: Fibre-Weave Skew Quick Reference

| Data rate | Problem severity | Typical mitigation |
|---|---|---|
| ≤ 10 Gb/s | Minor, absorbed by margin | None required |
| 10–25 Gb/s | Notable | Spread glass or rotation |
| 25–56 Gb/s | Critical | Spread + rotation + multi-ply |
| 56–112 Gb/s | Mandatory mitigation | All of the above + low-$D_k$-contrast glass |
| > 112 Gb/s | Designed from outset | Hybrid-bond / glass-free tech (interposer) |

---

## Key Formulas Reference

| Quantity | Formula |
|---|---|
| Propagation velocity | $v_p = c/\sqrt{D_{k,eff}}$ |
| Per-inch skew worst case | $\Delta t/L \approx T_{pd}[(\sqrt{D_{k,glass}} - \sqrt{D_{k,resin}})/\sqrt{D_{k,avg}}]$ |
| Mode conversion from skew | $|S_{cd21}|(f) \approx \sin(\pi f \Delta t)$ |
| Mitigation ratio, rotation by $\theta$ | $\Delta t_{rot} \approx \Delta t_0 \cdot \cos(\theta)$ averaged over many periods |
| Acceptable skew (design rule) | $\Delta t \le 0.05$ UI (aggressive), $\le 0.1$ UI (workable) |
