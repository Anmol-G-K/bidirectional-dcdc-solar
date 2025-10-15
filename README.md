# Design and Implementation of a 4-Switch Bidirectional Synchronous Buck-Boost DC-DC Converter for Solar Charging Lights

**Authors:**  
Anmol Govindarajpuram Krishnan, Aryan Jaljith, Mauli Rajguru  
Department of Electrical and Electronics Engineering  
Amrita Vishwa Vidyapeetham, Coimbatore, India  
Roll Numbers: CB.EN.U4EEE23103, CB.EN.U4EEE23105, CB.EN.U4EEE23120  

---

## Abstract

This report presents the design and implementation of a bidirectional four-switch synchronous buck-boost DC-DC converter for standalone solar battery charging applications. The converter enables efficient bidirectional power flow between a photovoltaic (PV) source and a 12 V lead-acid battery.  

A microcontroller-based control system employs PWM switching integrated with MPPT and PID control for optimal power transfer and voltage regulation. The system performance was validated through MATLAB Simscape simulations and hardware prototyping, achieving a peak efficiency of **83%**.  

The converter operates in buck and buck-boost modes during PV-to-battery charging, and in buck, boost, and buck-boost modes during battery discharge. The proposed design demonstrates effective power management and high reliability for compact, renewable energy storage systems under varying solar irradiation conditions.

---

The repository includes two primary files related to this project:

- **bidirection_buck_boost.slx** — the main MATLAB/Simulink model implementing the four-switch bidirectional buck-boost converter.  
- **MPPT_plot.slx** — a supporting file used for plotting and verifying the MPPT (Maximum Power Point Tracking) characteristics of the photovoltaic array.

These files together form the complete modeling and analysis setup for the converter design.

---
## Keywords

`DC-DC converter`, `bidirectional converter`, `buck-boost converter`, `solar charging`, `MPPT algorithm`, `four-switch topology converter`, `solar battery charging`

---

## 1. Introduction

The growing demand for renewable energy has accelerated the development of efficient power conversion, especially for harvesting and storing solar energy. Standalone photovoltaic systems require reliable DC-DC converters capable of managing power flow between solar panels, energy storage devices, and loads under varying electrical and environmental conditions.

Conventional power management systems, particularly in existing solar-powered lights and search lamps, often rely on separate unidirectional converters for charging and discharging or employ isolated topologies. **Such designs are bulky, complex, and suffer from increased costs and power losses**, lacking the dynamics required for modern energy storage.

To address these limitations, this project introduces a **bidirectional four-switch synchronous buck–boost DC–DC converter** for high-power solar lighting systems. This topology offers a compact and efficient alternative that enables bidirectional energy flow — both battery charging (from solar) and discharging (to a lighting load) — using a single converter, which significantly **reduces hardware complexity**.

For extracting optimal power from photovoltaic modules whose output characteristics vary with irradiation and temperature, the system uses a PID feedback control strategy, eliminating the need for manual switching.

This project presents the simulated design and hardware implementation of the converter, which interfaces a Monocrystalline Silicon PV module (31.0 V, 8.08 A at maximum power) with a 12 V, 100 Ah lead-acid battery. An STM32 Nucleo G474RE microcontroller implements the PWM switching and PID control for optimal power management. The design was **validated through MATLAB Simscape simulation and hardware testing**, demonstrating stable operation in buck, boost, and buck–boost modes for both charging and discharging operation.

---

## 2. Background Theory

### 2.1 DC–DC Converter Principles
DC–DC converters are power electronic circuits that convert a fixed DC voltage into a different DC voltage level. They operate by periodically switching energy storage elements such as inductors and capacitors. The two most basic types are the **buck (step-down)** and **boost (step-up)** converters.

**Buck Converter:** Output voltage is lower than input voltage.
\[
V_{out,buck} = D \times V_{in}
\]

**Boost Converter:** Output voltage is higher than input voltage.
\[
V_{out,boost} = \frac{V_{in}}{1 - D}
\]

### 2.2 Bidirectional Operation
A **bidirectional DC–DC converter** allows power flow in both directions — from source to storage (buck) and storage to load (boost). This is crucial for systems like electric vehicles and solar charging setups.

### 2.3 Synchronous Switching
Synchronous converters replace diodes with MOSFETs to minimize losses due to diode voltage drops. The two switches operate complementarily, enabling bidirectional current flow with reduced conduction loss.

### 2.4 Non-Isolated Converters
Non-isolated converters do not use galvanic isolation. They are compact, efficient, and ideal for low-to-medium voltage systems like solar-battery setups. The input and output share a common ground.

---

## 3. Proposed Converter Topology and Operating Principles

### 3.1 Four-Switch Buck-Boost Topology
![Converter Topology](images/Kicad_Diag.png)  
*Figure 1: Four-switch buck-boost converter circuit diagram.*

The converter includes four MOSFETs (Q1–Q4), an inductor (L), and capacitors (Cin, Cout). Depending on voltage conditions, it operates in buck, boost, or buck-boost modes.

### 3.2 Operating Modes

#### PV-to-Battery Charging Mode

##### Buck Mode (Vpv > Vbat)
When PV voltage exceeds battery voltage:
\[
M_{buck} = \frac{V_{BAT}}{V_{PV}} = D
\]
![Buck Mode](images/Screenshot 2025-10-14 105855.png)

##### Boost Mode (Vpv < Vbat)
\[
M_{boost} = \frac{V_{BAT}}{V_{PV}} = \frac{1}{1 - D}
\]
This mode was not implemented for simplicity since it is rarely needed.

##### Buck-Boost Mode (Vpv ≈ Vbat)
\[
M_{buck-boost} = \frac{V_{BAT}}{V_{PV}} = \frac{D}{1 - D}
\]
![Buck-Boost Mode](images/Screenshot 2025-10-14 110415.png)

#### Battery-to-Load Discharge Mode
Similar transitions occur in reverse:

- **Boost Mode:** (Vbat < Vload)  
  ![Boost Discharge](images/Screenshot 2025-10-14 110011.png)

- **Buck Mode:** (Vbat > Vload)  
  ![Buck Discharge](images/Screenshot 2025-10-14 110034.png)

### 3.3 Continuous Conduction Mode (CCM)
To maintain stability:
\[
L_{crit,buck-boost} = \frac{1}{\Delta i_l f_s} \cdot \frac{V_o}{1+\frac{V_o}{V_s}}
\]

### 3.4 Mode Transition and Deadzone Elimination
Traditional systems suffer from deadzones between buck and boost operation. This design avoids it via:
- Unified control algorithm
- Smooth mode transitions
- Buck-boost overlap mode

![Deadzone](images/deadtime.png)

---

## 4. Component Design and Selection

### 4.1 Photovoltaic Module
**Model:** Mitsubishi Electric PV-MLU250HC  
- 250 W, 31 V, 8.08 A  
- Voc = 37.6 V, Isc = 8.79 A  
- Monocrystalline silicon  
- Temp coefficient: -0.454 %/°C  

### 4.2 Battery
**Model:** RS PRO 727-0427  
- 12 V, 100 Ah sealed lead-acid AGM  
- Float voltage: 13.5–13.8 V  
- Max charge current: 30 A  
- Operating: -15 °C to 50 °C  

### 4.3 Load LEDs

#### LUXEON K2 (LXK2PW14V00)
- Forward voltage: 3.72 V @ 1 A  
- 120 lm at 1 A  
- Thermal resistance: 9 °C/W  

#### LUXEON HL1Z (L1HZ-5780100000000)
- Forward voltage: 2.8 V @ 350 mA  
- 148 lm typical luminous flux  
- Thermal resistance: 3 °C/W  

### 4.4 Inductor Design
Design target: 15 % ripple.  
\[
L_{max} = 59.629 \mu H
\]
Selected: **100 µH toroidal core**  
Peak current: 21.5 A, RMS: 20 A  
Use Litz wire to reduce skin effect.

### 4.5 Capacitor Design
\[
C_{in} = \frac{I_o D}{8 f_s \Delta V_{in}}
\]
Selected: Cin = 100 µF, Cout = 500 µF.

### 4.6 Power Switch (MOSFET)
**Model:** IRFZ44Z  
- Vds = 100 V, Id = 20 A  
- Rds(on) = 17.5 mΩ  
- Losses (at 100 °C): 11.75 W per switch.

### 4.7 Gate Driver
**Driver:** TLP250  
- Peak output current: 2 A  
- Built-in dead-time  
- Frequency: up to 500 kHz  
- Dead time: 500 ns chosen  
![Gate Driver](images/Screenshot 2025-10-06 025431.png)

---

## 5. Control Strategies

### 5.1 Microcontroller-Based Control
**Controller:** STM32 Nucleo G474RE (ARM Cortex-M4)  
- 75 kHz PWM frequency  
- 4 PWM outputs with dead-time control  

### 5.2 MPPT Implementation
MPPT algorithm tracks maximum power via real-time PV I–V and P–V analysis.  

![MPPT I-V](images/MPPT_VI.png)  
![MPPT P-V](images/MPPT_PV.png)

### 5.3 PID Regulation
PID controllers regulate PV voltage and current ensuring battery stability and smooth transitions between modes.

### 5.4 Mode Selection Logic
Automatic switching between:
- Buck (Vpv > Vbat + 2 V)
- Buck-Boost (Vpv ≈ Vbat)
- Boost (discharge mode)

---

## 6. Simulation Results

### 6.1 MATLAB Simscape Model
![Simscape Model](images/matlab_ckt.png)

---

### 6.2 Charging Mode

#### Buck Operation
- Irradiance: 1000 W/m²  
  - η = 87.38 %  
  - Vin = 30.96 V, Iin = 8.08 A  
  - Vout = 13.35 V  
  ![1000W Buck](images/Input_Output_1000.png)  
  ![Switching Buck](images/Charging_Switching_1000.png)

- Irradiance: 600 W/m² → η = 81.95 %  
  ![600W Buck](images/Input_Output_600.png)

- Irradiance: 400 W/m² → η = 87.07 %  
  ![400W Buck](images/Input_Output_400.png)

#### Buck-Boost Operation
- Irradiance: 1000 W/m² → η = 74.54 %  
  ![Buck-Boost IO](images/Input_Output_buck_boost.png)  
  ![Buck-Boost Switching](images/Switching_buck_boost.png)

---

### 6.3 Discharge Mode

#### Buck (LUXEON K2)
- η = 81 %  
- Vin = 11.74 V, Iout = 7.18 A  
![Discharge Buck IO](images/Input_Output_Buck.png)  
![Discharge Buck Switching](images/Discharging_Switching_Buck.png)

#### Boost (LUXEON HL1Z)
- η = 83.2 %  
- Vin = 11.74 V, Vout = 14.24 V  
![Discharge Boost IO](images/Input_Output_Boost.png)  
![Discharge Boost Switching](images/Discharging_Switching_Boost.png)

---

## 7. Hardware Implementation

### 7.1 Setup
![Hardware Setup](images/hardware setup 2.jpeg)
![Deadtime](images/Deadtime_Practical.jpg)

### 7.2 Inductor
Hand-wound toroidal core, 26 AWG copper wire.  
![Toroid](images/toroid wound inductor.jpeg)

### 7.3 Hardware Results

**Charging Buck Mode:**  
Input: 24 V → Output: 12.21 V  
![Charging Buck Output](images/Hardware_Output_Charging_Buck_With load.jpeg)  
![Charging Buck Oscilloscope](images/Charging_Buck_Practical.jpg)

**Discharging Boost Mode:**  
Input: 12 V → Output: 14.71 V  
![Boost Output](images/Hardware_Output_Disharging_Boost_With load.jpeg)  
![Boost Oscilloscope](images/Discharging_Boost_Practical.jpg)

> **Note:**  
> The hardware implementation phase is still ongoing. Additional refinements, including closed-loop control integration, performance measurements, and PCB realization, are under active development and will be updated in future revisions of this project.

---

## 8. Challenges
- Configuring Simscape PV and battery blocks for real-world parameters.  
- Missing datasheet parameters for accurate simulation.  
- Time constraints limited depth of component-level optimization.  
- Open-loop control was used in hardware for simplicity; closed-loop requires additional MCU abstraction layers and signal conditioning.

---

## 9. Future Scope
- PCB design and SiC MOSFET integration.  
- Hardware protection (OVP/OCP/Thermal).  
- IoT-based data monitoring.  
- Expansion to off-grid renewable systems and field lighting.

---

## 10. Conclusion
A 4-switch bidirectional synchronous buck-boost DC-DC converter was designed, simulated, and implemented for high-power solar lighting systems.  

- **Charging efficiency:** 74–82 %  
- **Discharging efficiency:** 81–83.2 %  

The system successfully demonstrated stable bidirectional operation and reliable mode transitions.

---

## 11. Acknowledgment
The authors thank **Dr. Mohanrajan S.R.** for his guidance and the **Department of Electrical and Electronics Engineering, Amrita Vishwa Vidyapeetham**, for laboratory support and resources.

---

## 12. References
1. M. H. Rashid, *Power Electronics: Circuits, Devices, and Applications*, 4th Ed., Pearson, 2014.  
2. N. Mohan, T. M. Undeland, W. P. Robbins, *Power Electronics: Converters, Applications, and Design*, 3rd Ed., Wiley, 2003.  
3. R. W. Erickson, D. Maksimovic, *Fundamentals of Power Electronics*, 3rd Ed., Springer, 2020.  
4. X. Li et al., “Four-Switch Buck–Boost Converter Based on Model Predictive Control With Smooth Mode Transition Capability,” *IEEE Trans. Ind. Electronics*, 68(10), 2021.  
5. J.-K. Shiau et al., “Circuit Simulation for Solar Power Maximum Power Point Tracking with Different Buck-Boost Converter Topologies,” *Energies*, 7(8), 2014.  
6. S. Das, “Bi-Directional Converter Topology for Solar-Battery Charging,” *Int. Res. J. Modern Eng. Technol. Sci.*, 2(9), 2020.  
7. Texas Instruments, “Test Report: PMP21529 4-Switch Buck-Boost Bi-directional DC-DC Converter Reference Design,” Application Report TIDT046, 2018.  

---
