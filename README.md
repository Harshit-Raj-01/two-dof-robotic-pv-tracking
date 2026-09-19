# Grid-Connected Solar PV System with Single-Axis Tracking & MPPT Inverter

A complete MATLAB/Simulink dynamic simulation model of a 3-phase grid-connected Solar Photovoltaic (PV) system. This model integrates a single-axis solar tracking control mechanism, a DC-DC Boost Converter driven by a Perturb & Observe (P&O) Maximum Power Point Tracking (MPPT) algorithm, and a 3-phase two-level Inverter synchronized to a $415\text{ V}$, $50\text{ Hz}$ utility grid via Phase-Locked Loop (PLL) and PWM control.

---

## 📌 System Architecture

                              [ SINGLE-AXIS TRACKER ]
                                         │
                                         ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Sun Vector  │───> │  Solar PV    │───> │ DC-DC Boost  │───> │  3-Phase     │───> │   Utility    │
│  (Ramp/PID)  │     │    Array     │     │  (P&O MPPT)  │     │   Inverter   │     │ Grid (415V)  │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘


▲
│ Gate Signals

┌──────────────────────┐

│  PLL & PWM Generator │

└──────────────────────┘


The system comprises 5 key stages:
1. **Single-Axis Solar Tracker:** A closed-loop PID positioning system matching panel orientation to the sun's trajectory to maximize effective irradiance ($G_{\text{eff}}$).
2. **Solar PV Array:** SunPower SPR-305E-WHT-D configuration delivering DC power under dynamic environmental conditions.
3. **DC-DC Boost Converter & MPPT:** Executes the Perturb & Observe algorithm via a custom MATLAB Function block to maintain peak power output.
4. **3-Phase Inverter & RL Filter:** Converts DC-link voltage ($500\text{ V}$) to sinusoidal 3-phase AC power, smoothed using a low-pass RL filter ($2\text{ mH}, 0.1\ \Omega$).
5. **Grid Synchronization & PLL:** Tracks grid voltage phase angle ($\theta$) with a Discrete 3-Phase PLL and generates reference sine waves for the 2-level 3-phase PWM Generator.

---

## ⚙️ Subsystem Parameters

| Stage | Simulink Block | Parameter / Property | Configured Value |
| :--- | :--- | :--- | :--- |
| **Global** | `powergui` | Solver Type / Sample Time | `Discrete` / `5e-6` s ($5\ \mu\text{s}$) |
| **Tracking** | `Ramp` | Slope / Initial Output | `18` / `-90` (sweeps $-90^\circ$ to $+90^\circ$) |
| | `PID Controller` | Proportional ($P$) / Integral ($I$) | `2.0` / `0.5` |
| | `Gain` | Gearbox Reduction Ratio | `0.01` ($1:100$) |
| **PV Array** | `PV Array` | Module Type / String Config | SunPower SPR-305E-WHT-D / $5$ Series, $1$ Parallel |
| **Boost / MPPT**| `Series RLC Branch` | Inductance ($L$) / Capacitance ($C$) | $5\text{ mH}$ / $100\ \mu\text{F}$ |
| | `MATLAB Function` | Algorithm | Perturb & Observe (Duty Cycle step $= 0.001$) |
| **Inverter** | `Series RLC Branch` | DC-Link Bus Capacitance | $2200\ \mu\text{F}$ |
| | `Universal Bridge` | Power Devices / Arms | `IGBT / Diodes` / $3$ Arms |
| | `Three-Phase RLC` | Filter Resistance / Inductance | $0.1\ \Omega$ / $2\text{ mH}$ |
| **Grid & PLL** | `Three-Phase Source` | Phase-to-Phase Voltage / Freq | $415\text{ V RMS}$ / $50\text{ Hz}$ |
| | `Discrete 3-Phase PLL`| Target Frequency | $50\text{ Hz}$ |
| | `PWM Generator` | Carrier Frequency | $5000\text{ Hz}$ ($5\text{ kHz}$) |

---

## 🛠️ Key Implementation Details

### 3-Phase Reference Wave Generation
To avoid vector dimension mismatch errors (`Array element 2 is out-of-bounds`) at the `PWM Generator (Three-phase, Two-level)` input, the scalar phase angle $\theta$ from the `Discrete 3-Phase PLL` is converted into a 3-element reference vector $[V_a, V_b, V_c]^T$ using an inline MATLAB Function block:

```matlab
function Vabc = fcn(angle)
% Transforms scalar PLL angle into balanced 3-phase reference sine waves
Vabc = [sin(angle); 
        sin(angle - 2*pi/3); 
        sin(angle + 2*pi/3)];
```
# 🚀 How to Run
Clone this repository:

```Bash
git clone https://github.com/Harshit-Raj-01/two-dof-robotic-pv-tracking.git
```
- Open MATLAB (R2020b or newer recommended) with Simulink and Simscape Electrical installed.

- Open the model file main_grid_tied_pv.slx (or your file name).

- Run the simulation by pressing Ctrl + T or clicking Run.

- Open the Scope blocks to view tracking performance, DC bus stability, and 3-phase grid AC injection currents.

# 📊 Expected Simulation Outputs
- Scope 1 (Actuator Tracking): Solar panel azimuth angle smoothly tracks the sun vector from −90 to +90 with near-zero error.

- Scope 2 (Effective Irradiance): Holds steady around 1000 W/m^2 due to perpendicular alignment maintained by the tracker.

- Scope 3 (DC-Link Bus Voltage): Stabilizes at 500–600 V DC post-transient phase.

- Scope 4 (Grid Currents & Voltages): Pure 50 Hz sinusoidal 3-phase AC voltage and current waveforms injected in phase with the grid voltage (Unity Power Factor).
