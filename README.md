# Signal Integrity and Power Integrity Interview Preparation

[![Signal Integrity](https://img.shields.io/badge/Topic-Signal%20Integrity%20%26%20Power%20Integrity-blue)](https://github.com/BrendanJamesLynskey/Interview_SI_PI)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Description

This repository provides comprehensive interview preparation materials for signal integrity (SI) and power integrity (PI) engineering roles. Topics covered include transmission line fundamentals, S-parameters, eye diagrams, PDN design methodologies, high-speed interface standards (DDR, PCIe, USB, Ethernet), measurement techniques using VNA and TDR, and PCB stackup optimization. Each section includes detailed explanations, worked problems, and practical design guidelines.

## Table of Contents

- [01 Transmission Line Fundamentals](#01-transmission-line-fundamentals)
- [02 Signal Integrity](#02-signal-integrity)
- [03 Power Integrity](#03-power-integrity)
- [04 High-Speed Interfaces](#04-high-speed-interfaces)
- [05 Measurement and Simulation](#05-measurement-and-simulation)
- [06 PCB Stackup and Materials](#06-pcb-stackup-and-materials)
- [07 Quizzes](#07-quizzes)
- [How to Use](#how-to-use)
- [Contributing](#contributing)
- [Related Repositories](#related-repositories)

## 01 Transmission Line Fundamentals

Foundational concepts for understanding signal propagation on PCB traces and cables.

- [Characteristic Impedance](01_transmission_line_fundamentals/characteristic_impedance.md)
- [Reflections and Matching](01_transmission_line_fundamentals/reflections_and_matching.md)
- [Propagation Delay and Dielectric](01_transmission_line_fundamentals/propagation_delay_and_dielectric.md)
- [Lossy Lines, Skin Effect, and Dielectric Loss](01_transmission_line_fundamentals/lossy_lines_skin_effect_dielectric.md)
- [Worked Problems](01_transmission_line_fundamentals/worked_problems/)
  - [Problem 01: Impedance Calculation](01_transmission_line_fundamentals/worked_problems/problem_01_impedance_calculation.md)
  - [Problem 02: Reflection Coefficient](01_transmission_line_fundamentals/worked_problems/problem_02_reflection_coefficient.md)
  - [Problem 03: Rise Time vs Line Length](01_transmission_line_fundamentals/worked_problems/problem_03_rise_time_vs_line_length.md)

## 02 Signal Integrity

Analysis and optimization of signal quality in high-speed digital designs.

- [S-Parameters](02_signal_integrity/s_parameters.md)
- [Eye Diagrams and Bathtub Curves](02_signal_integrity/eye_diagrams_and_bathtub.md)
- [Crosstalk: Near-End and Far-End](02_signal_integrity/crosstalk_near_end_far_end.md)
- [ISI and Equalization](02_signal_integrity/isi_and_equalisation.md)
- [Jitter Analysis](02_signal_integrity/jitter_analysis.md)
- [Worked Problems](02_signal_integrity/worked_problems/)
  - [Problem 01: Insertion Loss Budget](02_signal_integrity/worked_problems/problem_01_insertion_loss_budget.md)
  - [Problem 02: Crosstalk Estimation](02_signal_integrity/worked_problems/problem_02_crosstalk_estimation.md)
  - [Problem 03: Eye Diagram Interpretation](02_signal_integrity/worked_problems/problem_03_eye_diagram_interpretation.md)

## 03 Power Integrity

Design and analysis of power distribution networks for stable voltage supply.

- [PDN Impedance](03_power_integrity/pdn_impedance.md)
- [Target Impedance Method](03_power_integrity/target_impedance_method.md)
- [Decoupling Capacitor Placement](03_power_integrity/decoupling_capacitor_placement.md)
- [Plane Resonance](03_power_integrity/plane_resonance.md)
- [VRM and PMIC Design](03_power_integrity/vrm_and_pmic_design.md)
- [Worked Problems](03_power_integrity/worked_problems/)
  - [Problem 01: Target Impedance](03_power_integrity/worked_problems/problem_01_target_impedance.md)
  - [Problem 02: Capacitor Selection](03_power_integrity/worked_problems/problem_02_capacitor_selection.md)
  - [Problem 03: PDN Resonance Debug](03_power_integrity/worked_problems/problem_03_pdn_resonance_debug.md)

## 04 High-Speed Interfaces

Standard interfaces and compliance requirements for modern computing and communication systems.

- [DDR4, DDR5, and LPDDR5](04_high_speed_interfaces/ddr4_ddr5_lpddr5.md)
- [PCIe Gen 4, Gen 5, and Gen 6](04_high_speed_interfaces/pcie_gen4_gen5_gen6.md)
- [USB3 and USB4](04_high_speed_interfaces/usb3_usb4.md)
- [Ethernet and SerDes](04_high_speed_interfaces/ethernet_and_serdes.md)
- [Worked Problems](04_high_speed_interfaces/worked_problems/)
  - [Problem 01: DDR Routing Rules](04_high_speed_interfaces/worked_problems/problem_01_ddr_routing_rules.md)
  - [Problem 02: PCIe Loss Budget](04_high_speed_interfaces/worked_problems/problem_02_pcie_loss_budget.md)
  - [Problem 03: SerDes Channel Compliance](04_high_speed_interfaces/worked_problems/problem_03_serdes_channel_compliance.md)

## 05 Measurement and Simulation

Techniques and tools for validating SI/PI designs through measurement and simulation.

- [VNA and TDR](05_measurement_and_simulation/vna_and_tdr.md)
- [Oscilloscope Probing](05_measurement_and_simulation/oscilloscope_probing.md)
- [Simulation Tools and Methods](05_measurement_and_simulation/simulation_tools_and_methods.md)
- [Correlation: Simulation to Measurement](05_measurement_and_simulation/correlation_sim_to_measurement.md)
- [Worked Problems](05_measurement_and_simulation/worked_problems/)
  - [Problem 01: TDR Interpretation](05_measurement_and_simulation/worked_problems/problem_01_tdr_interpretation.md)
  - [Problem 02: De-Embedding](05_measurement_and_simulation/worked_problems/problem_02_de_embedding.md)
  - [Problem 03: Stackup Optimization](05_measurement_and_simulation/worked_problems/problem_03_stackup_optimisation.md)

## 06 PCB Stackup and Materials

PCB design considerations for controlled-impedance transmission lines.

- [Stackup Design](06_pcb_stackup_and_materials/stackup_design.md)
- [Materials: Dk and Df](06_pcb_stackup_and_materials/materials_dk_df.md)
- [Via Design and Backdrilling](06_pcb_stackup_and_materials/via_design_and_backdrilling.md)
- [Controlled Impedance](06_pcb_stackup_and_materials/controlled_impedance.md)
- [Worked Problems](06_pcb_stackup_and_materials/worked_problems/)
  - [Problem 01: Stackup for DDR5](06_pcb_stackup_and_materials/worked_problems/problem_01_stackup_for_ddr5.md)
  - [Problem 02: Via Stub Resonance](06_pcb_stackup_and_materials/worked_problems/problem_02_via_stub_resonance.md)
  - [Problem 03: Material Selection](06_pcb_stackup_and_materials/worked_problems/problem_03_material_selection.md)

## 07 Quizzes

Self-assessment quizzes covering key concepts.

- [Quiz: Transmission Lines](07_quizzes/quiz_transmission_lines.md)
- [Quiz: Signal Integrity](07_quizzes/quiz_signal_integrity.md)
- [Quiz: Power Integrity](07_quizzes/quiz_power_integrity.md)
- [Quiz: High-Speed Interfaces](07_quizzes/quiz_high_speed_interfaces.md)
- [Quiz: Measurement](07_quizzes/quiz_measurement.md)

## How to Use

1. **Sequential Learning**: Start with Section 01 (Transmission Line Fundamentals) to build foundational understanding. Progress through sections in order, as later sections build on earlier concepts.

2. **Worked Problems**: Each section includes 3 worked problems with detailed solutions. Attempt problems independently before reviewing solutions to reinforce learning.

3. **Quizzes**: After completing a section, take the corresponding quiz to assess comprehension. Use quiz results to identify gaps requiring further review.

4. **Interview Preparation**: Focus on understanding physical principles, not just formulas. Be prepared to:
   - Derive key equations from first principles
   - Explain trade-offs in design decisions
   - Discuss measurement techniques and their limitations
   - Work through calculation-based problems under time constraints

5. **Supplementary Study**: Reference the related repositories for broader PCB and power supply context.

## Contributing

Contributions are welcome. To contribute:

1. Create a new branch with a descriptive name
2. Add or edit content following the existing markdown style
3. Ensure all links are relative and correctly formatted
4. Submit a pull request with a clear description of changes

Contributions should maintain professional tone and focus on technical accuracy and clarity.

## Related Repositories

- [Interview_PCB_Electronics](https://github.com/BrendanJamesLynskey/Interview_PCB_Electronics) — General PCB design, schematic review, and electronics fundamentals
- [Interview_Power_Supply_Design](https://github.com/BrendanJamesLynskey/Interview_Power_Supply_Design) — Power supply topologies, control theory, and converter design

---

**License**: MIT (2025 Brendan James Lynskey)
