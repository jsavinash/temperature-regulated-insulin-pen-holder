# Engineering Specification & Roadmap: Prototype to Mass Production
## Temperature-Regulated Insulin Cooling Case (Active & Passive Hybrid)

**Document Reference:** ENG-SPEC-TRIPC-001  
**Project:** Temperature-Regulated Insulin Pen & Cartridge Cooling Carrier  
**Engineering Disciplines:** Thermal Engineering, Electronics & Embedded Systems, Mechanical & Manufacturing Engineering  
**Version:** 1.0  
**Status:** Approved for Engineering Execution  

---

## Executive Summary & Engineering Scope

This document translates the high-level Product Requirement Document (`insulin_cooling_case_prd.md`) into an end-to-end, multi-disciplinary engineering blueprint. It covers the physics calculations, mechanical design, electrical circuitry, control algorithms, manufacturing transitions, and test validation protocols required to bring both **Prototype** and **Mass Production** units to fruition.

```
       +-------------------------------------------------------+
       |                  SYSTEM ARCHITECTURE                  |
       +-------------------------------------------------------+
                                  |
         +------------------------+------------------------+
         |                                                 |
+------------------+                             +--------------------+
|  Active Carrier  |                             |  Passive Carrier   |
|  (Peltier / TEC) |                             |     (Bio-PCM)      |
+------------------+                             +--------------------+
  - USB-C 5V/3A Power                              - 4°C Phase-Change Gel
  - Filtered DC Buck-Driven TEC                    - Vacuum Insulation Flask
  - Micro Heatsink + MagLev Blower                 - Pure Conduction Sleeve
  - Dual NTC + Closed-loop PID                     - 24-48h Autonomous Hold
  - Flush Cap OLED & Low-Power MCU
```

---

## 1. Thermal Engineering Analysis & Thermodynamic Modeling

### 1.1 First-Principles Heat Transfer Analysis

Insulin (specifically rapid-acting formulations like insulin aspart/Fiasp, NovoRapid, Humalog) denatures irreversibly if frozen ($T \le 0^\circ\text{C}$) or degraded if exposed to heat ($T > 30^\circ\text{C}$). The clinical storage window is strictly:
$$\mathbf{2.0^\circ\text{C} \le T_{\text{chamber}} \le 8.0^\circ\text{C}}$$

#### Ambient Boundary Conditions
* Max Ambient Temperature ($T_{\text{amb}}$): $40.0^\circ\text{C}$ (313.15 K)
* Target Internal Core Temperature ($T_{\text{int}}$): $4.0^\circ\text{C}$ (277.15 K)
* Design Temperature Delta ($\Delta T_{\text{system}}$): $36.0\,\text{K}$

#### Chamber Heat Influx Equation
The thermal influx into the storage cavity $Q_{\text{in}}$ comprises conductive, convective, and radiative paths:
$$Q_{\text{in}} = Q_{\text{vacuum}} + Q_{\text{cap\_leak}} + Q_{\text{rad\_leak}} + Q_{\text{internal\_loss}}$$

1. **Vacuum Flask Body ($Q_{\text{vacuum}}$):**
   * Double-walled stainless steel cylinder: Outer OD $\approx 65\,\text{mm}$, Inner ID $\approx 36\,\text{mm}$, Length $\approx 180\,\text{mm}$.
   * Surface Area $A \approx 0.025\,\text{m}^2$.
   * Vacuum level $< 10^{-3}\,\text{Torr}$ ($< 0.133\,\text{Pa}$), virtually eliminating gas conduction and convection ($h_{\text{conv}} \approx 0$, $k_{\text{gas}} \approx 0$).
   * Radiative transfer through vacuum jacket:
     $$Q_{\text{rad}} = \frac{\sigma A (T_{\text{amb}}^4 - T_{\text{int}}^4)}{\frac{1}{\varepsilon_{\text{outer}}} + \frac{1}{\varepsilon_{\text{inner}}} - 1}$$
     With internal copper/silver plating or low-emissivity coating on inner wall ($\varepsilon \approx 0.03$ to $0.05$):
     $$Q_{\text{rad}} \approx \frac{(5.67 \times 10^{-8})(0.025)(313.15^4 - 277.15^4)}{\frac{1}{0.04} + \frac{1}{0.04} - 1} \approx \frac{(1.4175 \times 10^{-9})(9.623 \times 10^9 - 5.895 \times 10^9)}{49} \approx 0.108\,\text{W}$$

2. **Thermal Bridging at the Neck/Cap ($Q_{\text{cap\_leak}}$):**
   * The structural connection between the inner flask and outer shell at the neck forms a solid conductive path.
   * Stainless Steel neck thickness $t = 0.5\,\text{mm}$, circumference $C = \pi \times 36\,\text{mm} \approx 113.1\,\text{mm}$, conduction path length $L = 15\,\text{mm}$.
   * Stainless steel thermal conductivity $k_{\text{SS}} \approx 16\,\text{W/(m}\cdot\text{K)}$.
   * $A_{\text{cond}} = 113.1\,\text{mm} \times 0.5\,\text{mm} = 5.65 \times 10^{-5}\,\text{m}^2$.
   $$Q_{\text{neck\_cond}} = k_{\text{SS}} \cdot A_{\text{cond}} \cdot \frac{\Delta T}{L} = 16 \times 5.65 \times 10^{-5} \times \frac{36}{0.015} \approx 2.17\,\text{W}$$
   *(Note: This highlights that unmitigated neck conduction is the single largest thermal parasite! Design mitigation with thin-wall hydroforming or an engineered polymer thermal break is mandatory).*
   * Adding an engineered PEEK/POM polymer thermal break ($k \approx 0.25\,\text{W/(m}\cdot\text{K)}$) reduces this parasitic bridge to:
     $$Q_{\text{neck\_optimized}} \approx 0.35\,\text{W}$$

3. **Cap Insulation Conduction:**
   * Expanded Polypropylene (EPP) or Polyurethane (PU) core in cap ($k \approx 0.033\,\text{W/(m}\cdot\text{K)}$), diameter $36\,\text{mm}$, thickness $25\,\text{mm}$:
     $$Q_{\text{cap}} = 0.033 \times \frac{\pi (0.018)^2}{0.025} \times 36 \approx 0.048\,\text{W}$$

4. **Total Passive Parasitic Influx ($Q_{\text{total\_leak}}$):**
   $$Q_{\text{leak}} = 0.108 + 0.35 + 0.048 + 0.15\,(\text{seal/convection margin}) \approx \mathbf{0.656\,\text{W}} \approx \mathbf{0.7\,\text{W}}$$

---

### 1.2 Passive PCM Core Sizing (4°C Bio-Gel)

To sustain $T_{\text{int}} \le 8^\circ\text{C}$ for **48 hours** under continuous $40^\circ\text{C}$ ambient:
* Total Energy Ingress over 48h ($E_{\text{ingress}}$):
  $$E_{\text{ingress}} = Q_{\text{leak}} \times t = 0.70\,\text{W} \times (48 \times 3600\,\text{s}) = 120,960\,\text{J} \approx 121\,\text{kJ}$$
* Selected Phase Change Material: Hydrocarbon/bio-based organic PCM or inorganic salt-hydrate gel with melting phase transition $T_m = 4.0^\circ\text{C}$ to $5.0^\circ\text{C}$.
  * Latent heat of fusion ($\Delta H_f$): $\approx 200\,\text{kJ/kg}$ ($200\,\text{J/g}$).
  * Specific heat capacity liquid/solid: $c_p \approx 2.1\,\text{kJ/(kg}\cdot\text{K)}$.
* Mass of PCM required ($m_{\text{PCM}}$):
  $$m_{\text{PCM}} = \frac{E_{\text{ingress}}}{\Delta H_f} = \frac{121\,\text{kJ}}{200\,\text{kJ/kg}} \approx 0.605\,\text{kg} = 605\,\text{g (for pure 48h)}$$
* For a compact form factor ($24\,\text{hour}$ hold rating):
  $$m_{\text{PCM\_24h}} \approx \mathbf{302\,\text{g}} \quad (\text{Volume} \approx 320\,\text{mL})$$
* **Thermal Anti-Freezing Buffer:** A 1.5 mm high-density closed-cell elastomeric sleeve or anodized aluminum spacer prevents glass cartridges from touching the sub-zero boundary layer during initial insertion from the freezer.

---

### 1.3 Active Thermoelectric Cooling (TEC / Peltier) Physics

For the active lid module, the thermoelectric cooler acts as a heat pump operating against the temperature gradient.

#### Thermoelectric Governing Equations
The net cooling capacity at the cold junction ($Q_c$) is:
$$Q_c = 2 N \left[ \alpha I T_c - \frac{1}{2} I^2 R - K (T_h - T_c) \right]$$
Where:
* $\alpha$: Seebeck coefficient $(\text{V/K})$
* $R$: Electrical resistance $(\Omega)$
* $K$: Thermal conductance $(\text{W/K})$
* $N$: Number of semiconductor couples
* $I$: Operating DC current $(\text{A})$
* $T_c$: Cold side temperature $(277\,\text{K} \approx 4^\circ\text{C})$
* $T_h$: Hot side heatsink junction temperature $(313\,\text{K} + \Delta T_{\text{sink}} \approx 323\,\text{K} \approx 50^\circ\text{C})$

#### Total Heat to Dissipate on Hot Side ($Q_h$)
$$Q_h = Q_c + P_{\text{elec}} = Q_c + (V \times I)$$

With maximum steady-state thermal load from enclosure and active electronics $Q_c \approx 1.2\,\text{W}$:
* Selected Module: Custom/micro TEC (e.g., $25\,\text{mm} \times 25\,\text{mm}$ or $30\,\text{mm} \times 30\,\text{mm}$, 5V nominal, $I_{\max} = 2.0\,\text{A}$).
* Operating Point: Under 5V/1.2A ($P_{\text{elec}} = 6.0\,\text{W}$), typical COP at $\Delta T = 45\,\text{K}$ is $\approx 0.20 - 0.25$.
$$Q_c = 6.0\,\text{W} \times 0.22 = 1.32\,\text{W} \quad (\ge 1.2\,\text{W required})$$
$$Q_h = 1.32\,\text{W} + 6.0\,\text{W} = \mathbf{7.32\,\text{W}}$$

#### Hot-Side Heat Exchanger (Heatsink + Micro Blower)
* Allowable heatsink temp rise above ambient: $\Delta T_{\text{sink}} = T_h - T_{\text{amb}} = 50^\circ\text{C} - 40^\circ\text{C} = 10^\circ\text{C}$.
* Target thermal resistance of heatsink assembly ($R_{\text{th}}$):
  $$R_{\text{th\_sink}} \le \frac{\Delta T_{\text{sink}}}{Q_h} = \frac{10^\circ\text{K}}{7.32\,\text{W}} \approx \mathbf{1.36\,\text{K/W}}$$
* To achieve $1.36\,\text{K/W}$ within a compact $\varnothing 55\,\text{mm} \times 25\,\text{mm}$ active cap volume:
  * Dense skived-fin copper or aluminum heatsink (fin thickness $0.3\,\text{mm}$, fin pitch $1.0\,\text{mm}$, 32 fins).
  * High-static-pressure centrifugal blower: $30\,\text{mm} \times 30\,\text{mm} \times 7\,\text{mm}$ or $40\,\text{mm} \times 10\,\text{mm}$ MagLev blower delivering $\ge 1.8\,\text{CFM}$ with noise $\le 22\,\text{dBA}$.

---

## 2. Mechanical & Structural Engineering

```
+-------------------------------------------------------------+
|                MECHANICAL CROSS-SECTION EXPLODED            |
+-------------------------------------------------------------+

 [ ACTIVE CAP ]
   +-- Polycarbonate / ABS Top Cover with OLED Window
   +-- Centrifugal Micro-Blower (Exhaust Ducts Radial)
   +-- Copper Cold-Plate / Heat-Pipe / Skived Aluminum Fin Sink
   +-- Thermal Isolation Gasket (Silicone Shore 40A)
   +-- Peltier Module (25x25mm) clamped at 1.2 MPa with TIM
   +-- Aluminum Conduction Cold-Plug (Inserts into chamber)
   +-- Multi-lead Acme / Buttress Thread with Double Silicone O-rings

 [ FLASK CHASSIS ]
   +-- Outer Vacuum Shell (SUS304 / SUS316 Stainless Steel, 0.6mm)
   +-- High Vacuum Chamber (< 0.05 Pa) + Barium Sputter Getter
   +-- Inner Stainless Steel Wall (0.5mm, Copper Mirror Plated)
   +-- PEEK Neck Thermal Decoupling Collar

 [ INTERNAL STORAGE INSERT ]
   +-- CNC Extruded 6061-T6 Anodized Aluminum Thermal Core
   +-- Dual Chamber Machining:
       |-- Chamber A: FlexTouch Pen (150 mm L x 16.5 mm ID)
       |-- Chamber B: 3 mL Penfill Cartridge (65 mm L x 11.5 mm ID)
   +-- Anti-Rattle Overmolded Silicone Spring Ribs
   +-- Spring-loaded Cartridge Ejector Pin
```

### 2.1 Dimensional Mapping & Fits
| Component | Nominal Dimension | Tolerance | Fit Type / Notes |
| :--- | :--- | :--- | :--- |
| **FlexTouch Pen Chamber** | $\varnothing 16.5\,\text{mm} \times 155\,\text{mm}$ | $+0.2/-0.0\,\text{mm}$ | Clearance fit with silicone axial ribs for friction grip |
| **3 mL Cartridge Chamber** | $\varnothing 11.5\,\text{mm} \times 68\,\text{mm}$ | $+0.1/-0.0\,\text{mm}$ | Precision slip fit; internal silicone bottom shock pad |
| **Flask Outer Body** | $\varnothing 68.0\,\text{mm} \times 195\,\text{mm}$ | $\pm 0.25\,\text{mm}$ | Fits automobile cup holders and flight backpack side pockets |
| **Vacuum Annulus** | $3.5\,\text{mm}$ gap | $\pm 0.3\,\text{mm}$ | Maintained via ceramic/borosilicate standoffs |
| **Cold-Plug to Core Interface** | $\varnothing 24.0\,\text{mm} \times 12\,\text{mm}$ | $H7/h6$ | Spring-loaded face mating with Arctic MX-6 or Phase-Change TIM |

### 2.2 Material Specifications & Surface Treatment
1. **Outer Shell & Inner Shell:**
   * Material: Deep-drawing grade **AISI 304 or 316L Stainless Steel**.
   * Thickness: $0.6\,\text{mm}$ outer wall, $0.5\,\text{mm}$ inner wall.
   * Finish: Outer: PVD (Physical Vapor Deposition) Titanium-Gray or satin electropolished matte. Inner: Mirror silver/copper plating on vacuum side ($\varepsilon \le 0.03$).
2. **Cap & Thermal Break:**
   * Neck Isolation Ring: **PEEK 450G** or **POM (Acetal/Delrin)**. Prevents steel-to-steel conduction bridge.
   * Cap Outer Housing: Injection-molded **PC/ABS (Sabic Cycoloy)** with soft-touch matte finish.
   * Internal Core: **6061-T6 Aluminum Alloy**, hard anodized (Type III, MIL-A-8625, $25\,\mu\text{m}$ coating thickness) for chemical neutrality and corrosion protection against condensation.
3. **Elastomers & Gaskets:**
   * Medical-grade liquid silicone rubber (LSR, Shore 40A to 50A, USP Class VI, ISO 10993 compliant).

### 2.3 Acoustic & Airflow Ducting Design
* Air intake: Perforated annular intake ring along lower bezel of cap.
* Air exhaust: Directed radial directional exhaust ports angled at $45^\circ$ backward, preventing hot exhaust recirculation into intake vents.
* Fan mounting: Suspended on four silicone elastomeric isolators (Shore 30A) to decouple blade and motor acoustic vibration from the stainless steel flask resonator. Noise floor: $< 22\,\text{dBA}$ at 1 meter.

---

## 3. Electrical & Embedded Systems Engineering

### 3.1 Hardware Block Diagram

```
+-------------------------------------------------------------------------------+
|                             ELECTRICAL ARCHITECTURE                           |
+-------------------------------------------------------------------------------+

  [ USB-C 5V/3A Power Input ]
             |
             +---> CC1/CC2 Pull-down (5.1k) (Legacy 5V) + CH224K PD Controller
             |
             +---> High-Efficiency Synchronous Buck-Boost / Battery Manager
             |
             +---> Low-Noise LDO (3.3V) ----> MCU (STM32L412 / nRF52840)
             |                                    |
             |                                    +---> I2C: 0.91" OLED Display
             |                                    +---> I2C: TMP117 (Chamber NTC)
             |                                    +---> ADC: Heatsink NTC (10k 1%)
             |                                    +---> GPIO: Buzzer / Alert LED
             |                                    +---> PWM: Fan Tach & Speed Control
             |
             +---> Synchronous Buck DC-DC (Filtered for TEC)
                       |
                       +---> Filtered Smooth DC (0.5V - 4.5V) ---> Peltier TEC Module
```

### 3.2 Detailed Circuit Subsystems

#### A. Power Ingress & Regulation
* **Port:** USB Type-C Receptacle (16-pin hybrid SMT/TH with IP67 gasket).
* **Power Capability:** Handles 5V at up to 3.0A (15W) or 9V via USB Power Delivery (PD) negotiation using an on-board sink controller (WCH **CH224K** or STMicro **STUSB4500**).
* **System Rail Protection:** Texas Instruments **TPS2595** eFuse or discrete reverse-polarity P-channel MOSFET with over-voltage clamping ($V_{\text{clamp}} = 5.8\,\text{V}$ for 5V rail).

#### B. Peltier Module Driver Architecture
> [!IMPORTANT]
> **Peltier COP Preservation Rule:** Driving a Thermoelectric Cooler directly with raw square-wave PWM induces severe alternating thermal stress, acoustic whistle, and drastically degrades cooling efficiency (COP drops by 30-50%).
> The active driver MUST deliver **ripple-filtered smooth DC**.

* **Topology:** High-efficiency Synchronous Buck Converter with dedicated current-feedback control loop (e.g., **TI TPS62840** or **Texas Instruments TPS54302** configured in constant-current / adjustable analog voltage mode via DAC/filtered PWM reference).
* **Output Ripple:** $< 50\,\text{mV}$ peak-to-peak.
* **Output Range:** $0.5\,\text{V}$ to $4.5\,\text{V}$ dynamic DC output, $0 - 2.0\,\text{A}$.

#### C. Sensor Subsystem & Anti-Freezing Protection
* **Chamber Sensor:** **TI TMP117** (Factory-calibrated NIST traceable, $\pm 0.1^\circ\text{C}$ accuracy across $-20^\circ\text{C}$ to $+50^\circ\text{C}$, 16-bit resolution, low power $3.5\,\mu\text{A}$).
* **Secondary Heatsink Sensor:** Murata $10\,\text{k}\Omega \pm 1\%$ NTC Thermistor embedded in the hot-side copper baseplate to monitor heatsink saturation.
* **Hardware Interlock (Failsafe):** Dual-comparator or analog thermal supervisor (**TI LM2903** or thermal switch). If the cold plate reaches $\le 1.8^\circ\text{C}$, a hardware MOSFET disconnects TEC power immediately—independent of MCU firmware.

#### D. Microcontroller & Human-Machine Interface (HMI)
* **MCU:** **STM32L412KBU6** (Ultra-low-power ARM Cortex-M4, 128KB Flash, QFN32) or **Nordic nRF52840** (if Bluetooth Low Energy telemetry to iOS/Android companion app is desired).
* **Display:** $0.91''$ Mono OLED ($128 \times 32$ pixels, SSD1306, I2C, sleep current $< 10\,\mu\text{A}$) recessed under a scratch-resistant polycarbonate lens.
* **Visual/Audible Alerts:**
  * Two-color Status Ring LED (Blue: Stable $2-8^\circ\text{C}$, Amber: Cooling down, Red: Temperature alarm / Fan stall).
  * Piezo transducer (85 dBA buzzer for over/under-temperature audible alarm).

---

### 3.3 Firmware Architecture & Control Theory

The firmware operates on a deterministic state machine managed by FreeRTOS or a cyclic executive bare-metal scheduler.

```
       +-------------------------------------------------------+
       |                  FIRMWARE STATE MACHINE               |
       +-------------------------------------------------------+
                                  |
                               [ BOOT ]
                                  |
                       [ POST & Sensor Health ]
                                  |
             +--------------------+--------------------+
             |                                         |
     (Sensors OK)                             (Sensor Fault)
             v                                         v
     [ STATE: COOLING DOWN ]                  [ STATE: HARD FAULT ]
   PID: Aggressive P-term                     Buzzer Alert + Lockout
             |
    (T_chamber reaches 5.0°C)
             v
     [ STATE: STEADY-STATE ]
    Anti-Windup PID Active
    Fan speed throttled (< 18 dBA)
             |
    +--------+--------+
    |                 |
(T < 2.2°C)       (T > 8.0°C for > 300s)
    v                 v
[ EMERGENCY     [ EXCURSION ALARM ]
 TEC CUTOFF ]    Flash Red LED + Beep
```

#### PID Loop Implementation (Pseudo-C)
```c
// Closed-Loop Temperature Controller Parameters
#define T_TARGET_C          4.5f    // Target setpoint in Celsius
#define T_FREEZE_LIMIT_C    2.0f    // Absolute safety lower threshold
#define T_HIGH_LIMIT_C      8.0f    // Upper excursion threshold

typedef struct {
    float Kp, Ki, Kd;
    float integral;
    float prev_error;
    float output_min;
    float output_max;
} PID_Controller_t;

PID_Controller_t tec_pid = {
    .Kp = 1.85f,
    .Ki = 0.045f,
    .Kd = 0.62f,
    .integral = 0.0f,
    .prev_error = 0.0f,
    .output_min = 0.0f,
    .output_max = 1.0f  // 0% to 100% Buck Voltage Command
};

float PID_Update(PID_Controller_t *pid, float setpoint, float measured, float dt) {
    float error = measured - setpoint; // Positive if measured > setpoint -> need more cooling
    
    // Proportional
    float p_out = pid->Kp * error;
    
    // Integral with Anti-Windup Clamping
    pid->integral += error * dt;
    if (pid->integral > 10.0f) pid->integral = 10.0f;
    if (pid->integral < -10.0f) pid->integral = -10.0f;
    float i_out = pid->Ki * pid->integral;
    
    // Derivative
    float derivative = (error - pid->prev_error) / dt;
    float d_out = pid->Kd * derivative;
    pid->prev_error = error;
    
    float output = p_out + i_out + d_out;
    
    // Output saturation limits
    if (output > pid->output_max) output = pid->output_max;
    if (output < pid->output_min) output = pid->output_min;
    
    return output;
}

void Safety_Supervisory_Task(float T_chamber, float T_heatsink) {
    // Hardware anti-freeze protection
    if (T_chamber <= T_FREEZE_LIMIT_C) {
        Buck_Disable_TEC();
        Set_Alarm(ALARM_UNDER_TEMP);
    }
    // Heatsink thermal runaway protection
    if (T_heatsink >= 65.0f) {
        Buck_Disable_TEC();
        Fan_Set_Max();
        Set_Alarm(ALARM_OVERHEAT_HEATSINK);
    }
}
```

---

## 4. Phase-by-Phase Roadmap: Prototype to Mass Production

```mermaid
flowchart LR
    A["Phase 1:<br/>Benchtop Rig (PoC)"] --> B["Phase 2:<br/>Alpha Prototype (Works-Like)"]
    B --> C["Phase 3:<br/>EVT / Beta Prototype (Looks-Like/Works-Like)"]
    C --> D["Phase 4:<br/>DVT / Pre-Production Validation"]
    D --> E["Phase 5:<br/>PVT & Mass Production (MP)"]
```

---

### Phase 1: Proof-of-Concept (PoC) & Thermal Benchtop Rig
* **Goal:** Validate thermal heat load, Peltier sizing, PID stability, and heatsink performance on a breadboard.
* **Mechanical:**
  * Off-the-shelf vacuum insulated thermos ($\approx 350\,\text{mL}$ / $500\,\text{mL}$ stainless steel flask).
  * 3D printed cap (FDM/PETG or SLA resin) with cutouts for fan, heatsink, and cable glands.
  * Turn a simple cylinder of 6061 aluminum on a manual lathe to simulate the internal cartridge sleeve.
* **Electronics:**
  * Microcontroller: Arduino Nano, RP2040 Pico, or ESP32 dev kit.
  * Driver: Lab bench DC power supply or off-the-shelf adjustable buck converter module (LM2596 with external op-amp current control).
  * Sensors: 2x DS18B20 digital probes or 10k NTC thermistors wired into ADC.
  * Actuators: 30x30mm 12V/5V Peltier module + small CPU/GPU heatsink and 40mm fan.
* **Deliverables:**
  * Temperature logging data (40°C oven or heat-box test) verifying internal temperature falls from 25°C to 4°C within 30 minutes and maintains steady state.
  * Initial PID coefficient tuning.

---

### Phase 2: Functional Alpha Prototype ("Works-Like / Fits-Like")
* **Goal:** Integrate all electronics and custom mechanics into the target travel form factor.
* **Mechanical:**
  * Custom CNC machined 6061-T6 aluminum internal core featuring dual slots (150 mm FlexTouch + 65 mm 3 mL cartridge).
  * Cap enclosure: SLA/SLS printed in tough Nylon PA12 with threaded engagement and silicone O-ring seals.
  * Heatsink: Custom CNC milled skived fin copper baseplate + aluminum fin array.
* **Electronics:**
  * First-spin Custom PCB (Alpha Rev 1): 2-layer FR4 board fitting the cap diameter ($\varnothing 48\,\text{mm}$ round circular PCB).
  * On-board USB-C input, STM32 MCU, filtered buck regulator for TEC, TMP117 sensor on a rigid-flex tail running down to the aluminum chamber.
  * OLED display mounted flush in cap bezel.
* **Testing:**
  * Environmental chamber soak test: 24h at $40^\circ\text{C}$ external ambient with USB-C power bank supply.
  * Measure total power consumption ($W$) and battery life on a standard $10,000\,\text{mAh}$ power bank (target $\ge 8 - 12\,\text{hours}$ autonomous runtime).

---

### Phase 3: Engineering Validation Test (EVT) / Beta Prototype
* **Goal:** Transition from lab prototype parts to tooling-ready components with production-representative materials.
* **Mechanical:**
  * Vacuum Flask: Custom tooling for deep-drawn double-wall SUS304/SUS316 with high-vacuum furnace brazing and barium getter activation.
  * Neck: Injection-molded PEEK/Delrin thermal decoupling ring ultrasonic-welded or press-fit with laser-welded retention collar.
  * Cap: Soft-tooling / aluminum-tooling injection molded PC/ABS with ultrasonic brass threaded inserts.
  * Liquid Silicone Rubber (LSR) injection mold for anti-rattle internal pen grips.
* **Electronics:**
  * Production PCB (Rev 2.0): 4-layer FR4 (Signal-GND-PWR-Signal) with ENIG finish, fully certified automotive/medical-grade passives.
  * Full hardware failsafe interlocks tested and validated against simulated MCU freeze.
* **Deliverables:**
  * Complete 2D GD&T drawing package and 3D CAD STEP assembly models.
  * Verification of thermal drop resistance (1.2m drop test onto concrete per MIL-STD-810H).

---

### Phase 4: Design Validation Test (DVT) & Certification
* **Goal:** Verify that the product meets all regulatory, safety, and reliability standards under volume manufacturing processes.
* **Key Tests:**
  1. **Thermal Uniformity Mapping:** 12-channel thermocouple logging along the full length of the insulin pen to ensure $\Delta T_{\text{axial}} < 1.0^\circ\text{C}$ and no point reaches $< 2.0^\circ\text{C}$ (freezing prevention).
  2. **Ingress Protection (IP54 / IP65):** Dust ingress test and oscillating water spray test on cap USB-C port with silicone seal.
  3. **Acoustic Chamber Test:** Anechoic chamber acoustic test verifying $< 22\,\text{dBA}$ at full fan load.
  4. **Drop & Shock:** 26 drops from $1.2\,\text{m}$ (faces, edges, corners) without vacuum breach or optical OLED failure.
  5. **Regulatory Submissions:**
     * **EMC/EMI:** FCC Part 15 Subpart B (Class B) and EN 55032 / EN 55035 for CE compliance.
     * **Environmental:** RoHS 3 (Directive 2015/863) and REACH compliance.
     * **TSA / FAA Aviation:** Battery-free active model powered externally conforms to carry-on regulations; PCM bio-gel validated non-toxic/non-hazardous (SDS documentation package).

---

### Phase 5: Production Validation Test (PVT) & Mass Production (MP)
* **Tooling & Fixturing:**
  * Multi-cavity hardened steel injection molds (P20 or H13) for cap components.
  * Automated rotary laser welding cell for vacuum flask seams.
  * End-of-Line (EOL) Automated Test Equipment (ATE):
    * Automated vacuum retention helium leak test ($< 1 \times 10^{-9}\,\text{mbar}\cdot\text{L/s}$).
    * Electrical continuity, flash programming, and 15-minute quick chill thermal delta verification.
* **Yield & Quality Targets:**
  * First Pass Yield (FPY) $\ge 98.5\%$.
  * Thermal Cpk $\ge 1.33$ on chamber steady-state temperature ($4.0^\circ\text{C} \pm 1.5^\circ\text{C}$).

---

## 5. Bill of Materials (BOM) & Cost Architecture

### Estimated Unit Cost Breakdown (Active Model, 5,000 unit batch)

| Item | Component Description | Material / Vendor | Cost (USD) |
| :--- | :--- | :--- | :--- |
| **M1** | Double-wall Vacuum Flask Assembly | SUS304 / Copper coated / Laser-welded | $6.50 |
| **M2** | Dual-Chamber Thermal Aluminum Core | Extruded & CNC 6061-T6, Hard Anodized | $3.20 |
| **M3** | Active Cap Plastic Housing & Bezels | Injection Molded PC/ABS (3 parts) | $1.80 |
| **M4** | Thermal Decoupler & Internal Grippers | PEEK Collar + Overmolded LSR Silicone | $1.40 |
| **M5** | Heatsink Assembly | Skived Fin Copper/Alu Composite | $2.10 |
| **E1** | Thermoelectric Cooler (TEC) | $25 \times 25\,\text{mm}$ Custom Micro-Peltier | $2.80 |
| **E2** | Silent Micro Blower | $30 \times 30 \times 7\,\text{mm}$ MagLev Fan (<22 dBA) | $2.20 |
| **E3** | Main PCBA (Assembled) | 4-layer FR4, STM32L4, Buck DC-DC, PD, Passives | $5.40 |
| **E4** | Precision Sensor Tail | TMP117 + Flex PCB + Heatsink NTC | $1.30 |
| **E5** | OLED Display Module | 0.91" I2C Monochrome 128x32 OLED | $1.10 |
| **P1** | Packaging & Accessories | Rigid travel box, braided USB-C cable, manual | $2.50 |
| **L1** | Direct Labor & EOL Calibration | Assembly, helium vacuum test, burn-in | $2.70 |
| **TOTAL**| **Estimated Cost of Goods Sold (COGS)** | | **$33.00** |

*(Target Retail MSRP: $119.00 – $149.00 USD for active edition; $59.00 – $79.00 USD for passive PCM edition).*

---

## 6. Comprehensive Verification & Test Matrix

| Test ID | Test Category | Specification / Standard | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| **TR-01** | Hot Soak Hold (Passive PCM) | 40.0°C ambient, static chamber, 48 hours | Internal core stays between 2.0°C and 8.0°C for $\ge 36\,\text{h}$; $< 10.0^\circ\text{C}$ at 48h |
| **TR-02** | Active Cooling Pull-down | 40.0°C ambient, 5V/2A USB-C input | Pull down from 25°C to $\le 8.0^\circ\text{C}$ in $< 25\,\text{min}$ |
| **TR-03** | Anti-Freeze Safety Cutoff | Injected step disturbance or low ambient (5°C) | Core must NEVER dip below $2.0^\circ\text{C}$. Hardware interlock trips at $1.8^\circ\text{C}$ |
| **EM-01** | Acoustic Noise Signature | ISO 3744 anechoic chamber, 1m distance | Sound pressure level $\le 22.0\,\text{dBA}$ at nominal steady state |
| **ME-01** | Drop Impact Resistance | MIL-STD-810H Method 516.6 (1.2m drop to concrete) | No cracking, no vacuum degradation (verified via 12h thermal delta check) |
| **ME-02** | Cartridge Anti-Rattle | IEC 60068-2-6 sinusoidal vibration 10-500 Hz | Zero acoustic rattle; pen and glass cartridge intact without surface scuffing |
| **EE-01** | USB-C Interoperability | Tested on 20+ chargers, PD 3.0, QuickCharge, and 5V banks | Clean boot, no over-voltage latchup, auto-resume on power restore |
| **EE-02** | ESD Immunity | IEC 61000-4-2 ($\pm 8\,\text{kV}$ contact, $\pm 15\,\text{kV}$ air) | No resets, no display glitches, normal operation continues |

---

## 7. Next Steps & Immediate Engineering Milestones
1. **Week 1-2:** Procure benchtop components (standard $\varnothing 65\,\text{mm}$ vacuum bottle, $25 \times 25\,\text{mm}$ 5V TEC, STM32 Nucleo/RP2040, TMP117 breakout, 30mm maglev blower).
2. **Week 3-4:** 3D print Phase 1 active bench cap; lathe aluminum core insert; run 40°C thermal bench characterization.
3. **Week 5-6:** Layout Rev 1.0 circular PCBA with filtered buck converter and USB-C PD circuit.
4. **Week 7-8:** Assemble Alpha Prototype Rev 1; tune PID loop and validate cold-chain hold times under environmental chamber testing.

