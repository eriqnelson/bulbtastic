# **Bulbtastic Project Documentation**

## **Overview**

A self-powered, battery-backed smart lightbulb form factor Meshtastic node, combining LoRa mesh communication with a low-power lighting element and Bluetooth setup support. The radio functions independently from the light, powered by a 24-hour battery. Ideal for emergency mesh nodes or passive infrastructure deployments in outdoor/utility settings.

## **Phase 1: Breadboard Prototype**

### **Objectives**

* Validate power management and battery operation for 24+ hours  
* Confirm Meshtastic firmware compatibility with Wio SX1262 \+ XIAO ESP32-S3  
* Test BLE pairing and LED status behavior using a single shared LED  
* Prototype light sensor \+ LED circuit with auto on/off functionality  
* Simulate single AC-DC power source behavior with battery isolation

### 

### 

### **BOM (Breadboard Phase)**

| Component | Quantity | Notes |
| ----- | ----- | ----- |
| Seeed Wio SX1262 (XIAO ESP32-S3) | 1 | Main MCU \+ LoRa radio |
| HLK-PM12 AC-DC Converter | 1 | Converts 120V AC to 12V DC (250mA max) |
| TP4056 charging module (USB-C, protected) | 1 | For LiPo charging |
| LiPo Battery 3.7V 1000mAh | 1 | Radio backup \+ isolation |
| AMS1117-3.3V or HT7333 regulator | 1 | Converts battery to 3.3V logic |
| N-channel MOSFET (IRLZ44N) | 1 | For switching COB LED |
| COB LED, 3W 12V | 1 | Lighting element |
| 10kΩ resistor | 1 | For LDR divider |
| LDR (photoresistor) | 1 | Light sensing |
| Single LED (Red or Green) | 1 | Status indicator (GPIO 10\) |
| Tactile pushbutton (membrane preferred) | 1 | BLE pairing/reset |
| 830-point breadboard | 1 | Full-size |
| Jumper wires | as needed | M/M and F/M for routing |

### 

### **Power Wiring**

* **AC Input** → HLK-PM12: 120V AC input → 12V DC output @ 250mA  
* **12V Output (HLK-PM12)** splits:  
  * → COB LED (+12V, up to 250mA)  
  * → TP4056 IN+ (reduced to 5V via resistor dropper or buck converter if needed)  
* **TP4056 OUT+** → Battery \+ (LiPo 3.7V)  
* **TP4056 OUT-** → GND  
* **Battery \+** → AMS1117 VIN  
* **AMS1117 VOUT** → ESP32S3 3.3V pin (max 800mA peak supported by AMS1117)  
* **ESP32 GPIO Pins**:  
  * GPIO 10 → Status LED cathode (with 220Ω resistor)  
  * GPIO 0 → Button  
  * GPIO 6 → LDR voltage divider output  
  * GPIO 2 → MOSFET Gate (optional software control)

---

## **Phase 2: First Article Integration (3D Printed Housing)**

### **Objectives**

* Build fully integrated bulb form factor  
* House all components in a thermal/RF-aware shell  
* Incorporate vertical neck antenna chamber  
* Add USB-C access port for charging/debugging  
* Integrate single-button, single-LED user interface with waterproof design

### **Mechanical Design**

* Edison E26 screw base specified for standard North American light socket compatibility  
* Conductive threading connects only to LED circuit and AC-DC converter input  
* Insulate internal radio/battery system from AC mains using isolated AC-DC module  
* 3D printed shell with three major zones:  
  1. **LED dome** — diffused plastic, housing COB LED  
  2. **Neck/stem** — vertical antenna chimney and light sensor port  
  3. **Base section** — contains battery, PCB, USB-C port, status LED, and membrane button

### **Component Layout for 3D Print**

* **Status LED**: Mounted on the exterior-facing side of the base, visible from below when the bulb is installed in a socket.  
* **Membrane Button**: Located adjacent to the status LED on the base, recessed slightly to prevent accidental presses.  
* **USB-C Port**: Exposed through the base, centered or side-mounted with a structural lip or dust cover for protection.  
* **LDR (Light Sensor)**: Mounted high on the neck/stem section, oriented outward and angled slightly downward. The position should allow it to detect ambient room or porch light without interference from the bulb’s own COB LED.  
* **Vertical Chimney**: Antenna runs up through the neck, thermally and electrically isolated from the LED core, spaced away from copper contacts and aluminum cores.

Internal component mounting plates and channels should be added for:

* Wio board alignment above battery plane  
* Secure anchoring of HLK and TP4056 modules  
* Threaded inserts or press-fit snap points for enclosing base shell  
* Cable routing from AC contacts to converter and from battery section to upper dome  
* Edison E26 screw base specified for standard North American light socket compatibility  
* Conductive threading connects only to LED circuit and AC-DC converter input  
* Insulate internal radio/battery system from AC mains using isolated AC-DC module  
* 3D printed shell with three major zones:  
  1. **LED dome** — diffused plastic, housing COB LED  
  2. **Neck/stem** — contains wire antenna, BLE button, and status LED  
  3. **Base section** — contains battery, PCB, and USB-C port  
* Single waterproof membrane pushbutton mounted in neck  
* Status LED visible through neck


### **Mechanical Notes**

* Use thermally isolated vertical chimney in neck for antenna placement  
* USB-C port exposed in base and recessed for access  
* Vent holes or RF-transparent mesh for airflow

### **Power System Design**

* Single 120V input feeds AC-DC converter  
* 12V rail feeds LED directly and charge controller module  
* Charge controller feeds LiPo battery  
* Battery powers ESP32/LoRa via LDO regulator  
* Lighting circuit fully isolated from radio via battery buffering

### **Final Firmware**

* Use Meshtastic config tuned for low power:  
  * `position_broadcast_secs = 900`  
  * `min_wake = 30`  
  * `max_wake = 300`  
* Enable BLE pairing via long-press  
* Blink LED for heartbeat, transmit, or battery status  
* Button logic:  
  * Short press (≤1 sec): Soft reset system  
  * Long press (3–5 sec): Enter Bluetooth pairing mode (flashing blue)  
  * Press and hold \>10 sec: Power off or enter deep sleep

## **LED Behavior Map**

| State | LED Behavior |
| ----- | ----- |
| Power On | Solid green |
| Message Waiting | Slow green blink |
| BLE Pairing Mode | Fast blue blink |
| Low Battery | Red solid or blink |
| Reset Triggered | LED off briefly |
| Power Off/Deep Sleep | LED off completely |
| Idle/Sleep | Very slow green blink |

## 

## **Button Press Logic**

| Press Duration | Action |
| ----- | ----- |
| \< 1 second | Reset device |
| 3–5 seconds | Enter Bluetooth pairing |
| \> 10 seconds | Power down / deep sleep |

## **🧪 Testing Procedure (Breadboard Validation)**

### **Test 1: Power Flow**

* Connect AC power to HLK-PM12  
* Use multimeter to confirm 12V output  
* Confirm TP4056 input receives 5V (step down if needed)  
* Confirm battery charging (\~4.2V on OUT+)  
* Confirm 3.3V output from LDO to ESP32

### **Test 2: ESP32 Operation**

* Flash Meshtastic firmware  
* Monitor serial output for boot messages  
* Confirm LED behavior matches state (on boot, pair, etc.)

### **Test 3: Button Function**

* Press button \<1s → watch for soft reset (reboot log)  
* Press 3–5s → LED should enter fast blink (BLE mode)  
* Press \>10s → LED off, system enters sleep

### **Test 4: Light Sensor**

* Shade LDR → MOSFET should activate, powering COB LED  
* Restore light → COB LED turns off

### **Test 5: LoRa & BLE**

* Pair with Meshtastic mobile app via Bluetooth  
* Confirm channel settings, transmission, mesh visibility

## **Assembly Validation Criteria**

* Battery charges and discharges correctly  
* LED responds to all system states  
* BLE pairing and reset work as described  
* Light sensing and LED control trigger reliably  
* Power to ESP32 remains stable regardless of lighting activity  
* No power leakage from AC to battery domain