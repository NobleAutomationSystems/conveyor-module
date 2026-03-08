# Hardware Directory
Stores BOM, wiring diagrams, STL files, and datasheets.

## Component Selection

### Microcontroller: ESP32 Ecosystem
**Objective:** Standardize on a cheap, powerful WiFi-enabled microcontroller ecosystem for all modules.

**Decision:** We will **support the ESP32 family** (including the classic ESP32, ESP32-S3, and ESP32-C3). 
- **Why:** The ESP32 provides the best balance of cost, vast community support, ample GPIO, and built-in WiFi/Bluetooth for "Factorio-style" auto-discovery. Newer chips like the ESP32-S3 offer native USB and vector instructions, which may be extremely useful for future computer-vision modules.
- **Exclusions:** We are explicitly not supporting simple Arduinos (no WiFi), as network discovery is a core pillar of the platform.

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
For the first MVP, the **AliExpress aluminum frame conveyor** is the best choice. It provides a reliable mechanical baseline so we don't have to debug 3D printed belt tensions while writing the core networking and control software. 

---

### Motor Control Deep Dive: Stepper vs. DC Motor with Encoder
**Objective:** Determine the best motor architecture for a smooth "Factorio-style" conveyor belt that needs speed and position awareness.

#### Option 1: Stepper Motors (e.g., NEMA 17)
- **Pros:** Extremely precise positioning *without* needing feedback sensors. You tell it to move exactly 1,000 steps, and it stops exactly there. High holding torque at a dead stop.
- **Cons:** Low top speed, very noisy (whining/vibrating), inefficient (draws max current even when standing still), and if a heavy object blocks the belt, it will "skip steps" and silently lose track of its position.
- **Best For:** Pick-and-place robotic arms, 3D printers, or precise sorter pushers.

#### Option 2: DC Gear Motor + Magnetic Encoder (e.g., JGB37-520)
- **Pros:** Fast, highly efficient, very smooth and quiet continuous rotation. If overloaded, it slows down but the encoder keeps perfect track of position. Easily reaches higher RPMs for fast conveyor transport.
- **Cons:** Cannot easily hold a dead stop against a heavy load without physical braking. Requires PID control loops in software to target specific positions.
- **Best For:** Conveyor belts, drive wheels, continuous material transport.

### 🏆 Recommendation for MVP: DC Motor with Integrated Hall-Effect Encoder
For a conveyor belt, **smooth continuous flow** is far more critical than millimeter-perfect stopping. By using a DC Gear Motor with a built-in Hall-effect encoder (like the JGB37-520), we get the quiet, efficient speed of a DC motor along with the precise speed/position tracking of an encoder. We will use the ESP32's hardware PCNT (Pulse Counter) to read the quadrature pulses effortlessly.

---

### Motor Driver
**Objective:** Drive the 12V DC motor on the selected conveyor belt kit with variable speed and direction controlled by the ESP32 (3.3V logic).

- **L298N:** Cheap, common but very inefficient (drops ~2V internally). Creates excessive heat.
- **TB6612FNG:** Highly efficient (~95%), compact, supports up to 1.2A continuous (3A peak). Matches 3.3V logic of the ESP32 natively. Uses 2 digital pins for direction, 1 PWM for speed.
- **DRV8833:** Efficient, supports lower voltages well, up to 1.5A continuous. Doesn't have a dedicated PWM pin (uses dual PWM for speed/direction).

### 🏆 Recommendation for MVP: TB6612FNG
The **TB6612FNG** easily handles the 12V small DC gear motor found on AliExpress conveyors. It solves the massive inefficiency/heat problem of the older L298N, fits flawlessly with the 3.3V nature of the ESP32, and keeps the code simple (1 PWM pin per motor).

---

### Object Detection Sensors
**Objective:** Detect when objects are placed on the conveyor or arrive at the end of the line, handling mixed materials (cardboard, plastic, metal).

### 🏆 Recommendation for MVP: HC-SR04 Ultrasonic Sensors
To support arbitrary materials moving down the Factorio-style belt, we must use the **HC-SR04 Ultrasonic Sensor**. 
While slightly bulkier than a tiny IR LED, it is entirely immune to ambient factory/room lighting and will flawlessly detect transparent plastics and dark cardboards that completely blind cheap IR sensors. We will mount one at the load zone (start) and one at the unload zone (end) of each conveyor module.

---

### Power Architecture
**Objective:** Decide how the modular platform receives and distributes power organically safely without electrical noise crashing the microcontrollers.

### 🏆 Recommendation for MVP: Shared 12V Bus with Local Isolated Buck Converters
To preserve the modular, plug-and-play user experience, we must use a **Shared 12V DC Power Bus** across the factory line. We will use robust, quick-disconnect passthrough terminals (like XT60 or Wago 221s) on the edges of every conveyor module. 

Crucially, to protect the delicate ESP32 from the noisy 12V motor line, every module must include a locally stepped-down **DC-DC Buck Converter (e.g., MP1584 or LM2596)** heavily decoupled with capacitors. The ESP32 will run safely isolated at 3.3V from the Buck Converter, while the TB6612FNG motor driver pulls the inherently noisy 12V raw voltage straight from the bus.