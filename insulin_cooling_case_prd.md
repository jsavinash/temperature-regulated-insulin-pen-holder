# Production-Grade Insulin Cooling Case: Product Requirement Document (PRD)

## 1. Product Overview & Market Positioning
This product is a premium, travel-ready medical cooling case designed specifically for individuals managing diabetes who require a highly secure, temperature-regulated solution for prefilled insulin pens and 3 mL cartridges. 

### Target Audience
*   Active travelers, business professionals, and outdoor enthusiasts using Fiasp, NovoRapid, Humalog, or similar insulin regimens.
*   Users seeking a discreet, luxury aesthetic that replaces clinical-looking medical organizers.

---

## 2. Technical Specifications & Dimension Mapping
To ensure universal compatibility with standard U-100 delivery formats, the internal chambers must adhere to strict tolerances to prevent internal movement or rattling.

### Device Geometry
*   **Fiasp / Novo Nordisk FlexTouch Pen:** 150 mm (L) × 16 mm (D)
*   **Standard 3 mL Penfill Cartridge:** 65 mm (L) × 11.2 mm (D)

### Material Architecture
| Component | Material | Manufacturing Process | Purpose |
| :--- | :--- | :--- | :--- |
| **Outer Shell** | 304 or 316 Food-Grade Stainless Steel | Deep drawing & Double-wall laser welding | Rugged impact protection and structural integrity. |
| **Insulation Layer** | High-vacuum internal gap | Vacuum pumping chamber | Eliminates conductive and convective heat transfer. |
| **Internal Core** | Medical-grade Anodized Aluminum / Food-Grade Silicone | CNC machining / Injection molding | Secure fitment, anti-rattle friction clips, and high thermal conductivity. |

---

## 3. Thermal Performance Standards
The device must operate reliably in extreme ambient temperatures without risking the structural stability or biological efficacy of the insulin.
*   **Temperature Target Range:** Maintain internal temperatures between **2°C and 8°C** for active storage, or below **30°C** for open-pen maintenance.
*   **Cold-Chain Integrity:** No direct contact permitted between the glass cartridges and the cooling medium to avoid accidental freezing (0°C damage).
*   **Passive Model (PCM):** Must sustain the target range for at least **24–48 hours** in 40°C ambient external heat using a 4°C Phase Change Material biogel.
*   **Active Model (Peltier):** Must provide continuous cooling below 8°C indefinitely via 5V/2A USB-C power input.

---

## 4. Hardware & Electrical Subsystems (Active Variant)
*   **Cooling Engine:** Miniature Thermoelectric Peltier cooling module.
*   **Thermal Dissipation:** Low-profile aluminum heat sink paired with a brushless, silent micro-fan (noise signature under 25 dB).
*   **User Interface:** Cap-integrated flush OLED or LCD screen displaying real-time internal temperature probe telemetry.
*   **Power Management:** USB-C input interface supporting power banks, vehicle adaptors, and airline seat power supply ports.

---

## 5. Regulatory & Compliance Blueprint
*   **Classification Strategy:** Marketed as a "Premium Medical Supply Organizer and Travel Case" to streamline initial consumer retail launch.
*   **Certifications Required:**
    *   **TSA & Aviation Compliance:** Biogel packs must be clearly chemically labeled, non-toxic, and non-flammable to pass carry-on X-ray inspection points safely.
    *   **Electrical Compliance:** CE, FCC, and RoHS certifications required for the active USB electronics and circuitry.
    *   **Clinical Validation:** Independent laboratory chamber validation data must support all thermal retention claims on consumer packaging.
