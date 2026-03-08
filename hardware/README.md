# Hardware Directory
Stores BOM, wiring diagrams, STL files, and datasheets.

## Component Selection

### Microcontroller: ESP32 Standard
**Objective:** Standardize on a single cheap, powerful WiFi-enabled microcontroller for all modules.

**Decision:** We will **only support the standard ESP32** (e.g., ESP32-WROOM-32 dev boards). 
- **Why:** It provides the best balance of cost, vast community support, ample GPIO, dual-core performance, and built-in WiFi/Bluetooth for network discovery.
- **Exclusions:** We are explicitly not supporting simple Arduinos (no WiFi), or newer variants like ESP32-C3/S3 unless strictly required in the future, to keep the baseline simple and universally compatible.

---

### Conveyor Belt
**Objective:** Identify affordable, off-the-shelf mini conveyor belt kits suitable for ESP32 control.

#### Option 1: AliExpress "OpenCV PU Belt Mini Conveyor"
- **Link:** [Example AliExpress Listing](https://www.aliexpress.com/w/wholesale-mini-conveyor-belt.html)
- **Price:** ~$30 - $50
- **Dimensions:** Typically 500mm x 60mm
- **Motors Included:** Small geared DC motor (often 12V or 24V).
- **Pros:** Extremely common, durable PU belt, pre-tensioned aluminum frame. Easy to interface with motor drivers like L298N.
- **Cons:** Long shipping times, motor quality varies.

#### Option 2: 3D Printable Modular Conveyor (ESP32-S3 / Servo)
- **Link:** [MakerWorld 3D Printable Conveyor](https://makerworld.com/en/models/144917)
- **Price:** Cost of filament + $5 continuous rotation servo + fastners (~$15 total).
- **Dimensions:** Modular, variable length.
- **Motors Included:** User provides a continuous rotation servo (e.g., FS90R or MG996R).
- **Pros:** Maximizes the "Factorio LEGO" feel. Fully open-source and customizable. Direct control via PWM from ESP32.
- **Cons:** Requires a 3D printer. Belt is usually printed flex-links or fabric, not as robust as commercial PU.

#### Option 3: CRCibernetica Educational Mini Conveyor Belt Kit
- **Link:** [CRCibernetica Kit](https://www.crcibernetica.com/mini-conveyor-belt-kit/)
- **Price:** ~$25
- **Dimensions:** 250mm length
- **Motors Included:** Standard TT Gear Motor (3-6V).
- **Pros:** Wood/acrylic DIY kit, great for learning, very cheap TT motor is easy to drive with low-voltage drivers like DRV8833 or L298N.
- **Cons:** Very short, fragile materials, TT motors lack torque for heavy items.

### 🏆 Recommendation for MVP: Option 1 (AliExpress OpenCV PU Belt)
For the first MVP, the **AliExpress aluminum frame conveyor** is the best choice. It provides a reliable mechanical baseline so we don't have to debug 3D printed belt tensions while writing the core networking and control software. It comes with a beefy enough DC motor that we can experiment with heavier objects, requiring a standard motor driver like an L298N or TB6612FNG.
