# Simple_InGaAs_Photodetector
This repository documents how the photodetector was built, starting from developing a electronic circuit and a PCB as well as assembly and tests (ongoing)
# 6 GHz InGaAs Photodetector PCB and schematics

## Technical Specifications

| Parameter | Specification |
| :--- | :--- |
| **RF Bandwidth** | DC up to 6 GHz |
| **Characteristic Impedance** | 50 Ω (Matched Coplanar Waveguide with Ground) |
| **Bias Voltage ($V_{CC}$)** | ~12 VDC (Nominal) |
| **RF Output Interface** | Edge-Launch SMA Connector |
| **Power Supply** | 3 LiPo 3.7V Batteries via 2x4 Pin Header |
| **Design Software** | KiCad v9.0

---

## System Architecture & Circuit Design

The circuit implements a high-bandwidth, low-noise reverse-biased InGaAs photodetector topology designed to extract fast electrical transients from optical signals.

### 1. Photodiode Biasing Topology (`PD1`)
The photodiode (`PD1`) is configured in a reverse-bias configuration:
* **Cathode (Pin 1):** Connected to the filtered +12V DC bias line.
* **Anode (Pin 2):** Connected directly to the 50 Ω RF transmission line leading to the SMA output.
* **Case(Pin 3):** Directly tied to Ground to provide structural shielding.

**Why:** Applying a +12V reverse bias across the photodiode significantly widens its internal depletion region. This minimizes the junction capacitance ($C_j$), which is the primary limiting factor for high-speed photodetector bandwidth. The electrical bandwidth is governed by:

$$f_{3\text{dB}} \approx \frac{1}{2\pi \cdot R_L \cdot C_j}$$

Where $R_L$ is the 50 Ω load resistor internal to the connected measurement equipment (e.g., oscilloscope or spectrum analyzer). Minimizing $C_j$ ensures crisp pulse responses and maximum frequency extension up to 6 GHz.

### 2. Multi-Tier Power Decoupling & Low-Pass Filter Network
To prevent high-frequency RF currents from leaking into the power supply rail and to insulate the photodiode from power supply ripple, an aggressive, multi-stage cascading low-pass filter is implemented:
* **Bulk Storage (`C1`, `C2`):** A **10 µF (`C1`)** and a **6.8 µF (`C2`)** capacitor flank a **1 kΩ (`R1`)** series isolation resistor. This network forms an RC low-pass filter ($f_c \approx 23.4\text{ Hz}$) that squashes low-frequency power supply noise and ripple.
* **RF Decoupling Array (`C3`, `C4`, `C5`):** Three ultra-low Equivalent Series Resistance (ESR) and Equivalent Series Inductance (ESL) surface-mount capacitors are placed in parallel directly adjacent to the photodiode cathode: **47 pF (`C3`)**, **10 pF (`C4`)**, and **2 pF (`C5`)**. 

*Why:** Single capacitors possess self-resonant frequencies (SRF) above which they behave inductively. By staggering capacitance values down to 2 pF, the board maintains a broad, ultra-low impedance return path to ground across the entire multi-gigahertz spectrum, ensuring the photodiode sees a perfectly stable, noise-free DC bias.

---

## PCB Design & RF Considerations

At 6 GHz, electrical traces cannot be treated as simple lumped-element wires; they must be managed as **distributed-element transmission lines**. 

### 1. 50 Ω Impedance-Matched Transmission Line
The RF signal path connecting the photodiode anode to the SMA connector uses a **Coplanar Waveguide with Ground (CPWG)** geometry. 
* **Geometry:** A central signal trace surrounded on the same layer by continuous ground planes, separated by a precise air gap (0.3mm from each side), with a solid reference ground plane on the layer directly below.
* **Impedance Target:** Exactly **50 Ω** at 6 GHz. Trace width carefully designed since trace length needed to be more than λ/10 due to mechanical constraints.
* **Design Execution:** The width of the central trace and the width of the ground clearances are precisely computed based on the PCB stackup's dielectric constant ($\epsilon_r$), substrate height (FR4 or specialized high-frequency laminate), and copper thickness. This precise geometric control prevents impedance discontinuities, minimizing signal reflections and return loss ($S_{11}$).

### 2. Via Fence & Stiching
The PCB layout features a highly dense matrix of ground via stitching:
* **Via Fencing:** Rows of closely spaced via stitches flank both sides of the CPWG transmission line. This bounds the electromagnetic fields within the dielectric, suppresses parasitic substrate modes, and prevents cross-coupling or radiation loss. The distance between each via and from the edge of the trace was carefully calculated and respectively equals, 2 mm and 1.95mm (measured from centre of the via to centre of the path)
* **Resonance Suppression:** The entire board area is stitched with vias to tightly couple all ground planes together. This eliminates structural ground loops and prevents the PCB substrate from acting as a resonant cavity at microwave frequencies.
* **Mounting Hole Shielding:** All 4 mechanical mounting holes are encircled by dense via rings, binding the chassis ground firmly to the internal layer grounds to eliminate localized electromagnetic radiation.

### 3. Component Layout & RF Footprints
* **Linear Signal Flow:** The design maintains a perfectly straight. Right-angle bends are entirely avoided, as they introduce parasitic capacitive columns that degrade high-frequency signals.
* **Decoupling Proximity:** The sub-nanofarad capacitors (`C3`, `C4`, `C5`) are placed immediately adjacent to the photodiode pins using minimal trace lengths to mitigate the parasitic trace inductance that would otherwise invalidate their high-frequency filtering performance.
* **Missing Solder Mask:**  Peeling back the mask, ensures that the 6 GHz signal experiences predictable 50 $\Omega$ impedance, minimal signal attenuation, and maximum propagation speed by utilizing air as the primary upper dielectric.

---

## Component Bill of Materials (BOM)

| Reference | Qty | Value | Package | Description |
| :--- | :--- | :--- | :--- | :--- |
| **PD1** | 1 | InGaAs Photodiode | 3-Pin Package | High-bandwidth Photodiode (6 GHz) |
| **J1** | 1 | CON-SMA-EDGE-SRP | Edge-Mount SMA | 50 Ω Female RF Edge-Launch Connector |
| **J2** | 1 | 2x4 Header (2.54mm) | Thru-Hole | Power/GND Distribution Interface |
| **R1** | 1 | 1 kΩ | SMD 0402/0201 | Thick/Thin Film Isolation Resistor |
| **C1** | 1 | 10 µF | SMD 0805/1206 | Ceramic Bulk Capacitor |
| **C2** | 1 | 6.8 µF | SMD 0805 | Ceramic Bulk Filter Capacitor |
| **C3** | 1 | 47 pF | SMD 0402/0201 | Ultra-Low ESR High-Q RF Capacitor |
| **C4** | 1 | 10 pF | SMD 0402/0201 | Ultra-Low ESR High-Q RF Capacitor |
| **C5** | 1 | 2 pF | SMD 0402/0201 | Ultra-Low ESR High-Q RF Capacitor |

## PCB and Schematic
<img width="1604" height="1242" alt="image" src="https://github.com/user-attachments/assets/d1fddcc6-1b44-4de7-8a1d-2c7da6d707b5" />
<img width="1849" height="530" alt="image" src="https://github.com/user-attachments/assets/e41e9eec-206c-4ead-85ae-2dd7a8bd9c87" />



