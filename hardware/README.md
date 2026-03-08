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

---

### Motor Driver
**Objective:** Drive the 12V DC motor on the selected conveyor belt kit with variable speed and direction controlled by the ESP32 (3.3V logic).

- **L298N:** Cheap, common but very inefficient (drops ~2V internally). Creates excessive heat.
- **TB6612FNG:** Highly efficient (~95%), compact, supports up to 1.2A continuous (3A peak). Matches 3.3V logic of the ESP32 natively. Uses 2 digital pins for direction, 1 PWM for speed.
- **DRV8833:** Efficient, supports lower voltages well, up to 1.5A continuous. Doesn't have a dedicated PWM pin (uses dual PWM for speed/direction).

### 🏆 Recommendation for MVP: TB6612FNG
The **TB6612FNG** is the best choice. It easily handles the 12V small DC gear motor found on AliExpress conveyors. It solves the massive inefficiency/heat problem of the older L298N, fits flawlessly with the 3.3V nature of the ESP32, and keeps the code simple (1 PWM pin per motor).

---

### Encoders & Speed Feedback
**Objective:** Track the speed and position of the conveyor belt to enable closed-loop "Factorio-like" logic from the central brain.

- **External Rotary Encoders:** Optical or magnetic. Requires messy 3D-printed brackets and shaft coupling to the conveyor drive roller. Very precise, but difficult to physically assemble robustly.
- **Integrated DC Gearmotor with Built-in Hall Encoder (e.g. JGB37-520):** Slightly more expensive than a raw motor, but integrates the quadrature encoder magnetically onto the rear motor shaft. Outputs standard A/B pulses directly.

### 🏆 Recommendation for MVP: Integrated Hall-Effect Motor Encoders
We should replace the generic DC motor included in cheap conveyor kits with an **Integrated DC Gearmotor with a Built-in Hall Encoder (e.g. JGB37-520)**. 
Using a motor with a pre-mounted encoder completely eliminates the need for users to align and couple delicate external optical encoders, preserving the "Plug & Play" nature of the ecosystem. The ESP32's built-in PCNT (Pulse Counter) hardware block can read the dual channels flawlessly.

---

### Object Detection Sensors
**Objective:** Detect when objects are placed on the conveyor or arrive at the end of the line, handling mixed materials (cardboard, plastic, metal).

- **Infrared (IR) Reflection:** Extremely cheap (~$1). Bounces light off the object. Fails terribly on matte black plastic (absorbs light) or shiny metal (scatters light unpredictably), making it unreliable for mixed recycling/manufacturing lines.
- **Standard Capacitive Proximity:** Works great for hidden object detection, but industrial 10cm range sensors are large, expensive, and require 12V-24V logic.
- **Ultrasonic (HC-SR04):** Very cheap (~$2). Bounces a sound wave off the object. Relies purely on physical density, so it perfectly detects clear plastic bottles, matte black cardboard boxes, and shiny metal identically. 

### 🏆 Recommendation for MVP: HC-SR04 Ultrasonic Sensors
To support arbitrary materials moving down the Factorio-style belt, we must use the **HC-SR04 Ultrasonic Sensor**. 
While slightly bulkier than a tiny IR LED, it is entirely immune to ambient factory/room lighting and will flawlessly detect transparent plastics and dark cardboards that completely blind cheap IR sensors. We will mount one at the load zone (start) and one at the unload zone (end) of each conveyor module.
