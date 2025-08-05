# **Bulbtastic v1**

## **Power System**

### **1\. AC-DC Power Supply**

Component: HLK-PM12  
Input: 120V AC (L and N terminals)  
Output: \+12V DC, GND  
Max Output Current: 250mA

### **2\. LiPo Battery Charging**

Component: TP4056 Module (with USB-C and protection)  
\- IN+ → HLK-PM12 VOUT (12V stepped down to 5V via buck or resistor dropper)  
\- IN- → GND  
\- OUT+ → LiPo Battery \+  
\- OUT- → GND  
Battery: 3.7V LiPo 1000mAh

### **3\. 3.3V Regulation for Logic**

Component: AMS1117 or HT7333  
\- VIN → Battery \+  
\- GND → GND  
\- VOUT → ESP32 3.3V input  
Output Voltage: 3.3V  
Max Output Current: 800mA

## **Microcontroller and Communication**

### **4\. ESP32-S3 Microcontroller (Seeed Wio XIAO \+ SX1262 LoRa)**

Pin: 3.3V  → From regulator  
Pin: GND   → Common ground

GPIO Assignments:  
\- GPIO 0  → Button input  
\- GPIO 2  → MOSFET gate  
\- GPIO 6  → LDR analog input  
\- GPIO 10 → Status LED cathode

## **Lighting System**

### **5\. COB LED (12V)**

Anode → HLK-PM12 VOUT (12V)  
Cathode → MOSFET D (Drain)  
LED Power: 3W typical, draw \~250mA @ 12V

### **6\. MOSFET (IRLZ44N or similar logic-level N-channel)**

Gate   → GPIO 2 (through 10k pull-down to GND optional)  
Drain  → COB LED cathode  
Source → GND

## **User Interface**

### **7\. Status LED (single-color, e.g., green)**

Anode   → 3.3V  
Cathode → GPIO 10 via 220Ω resistor

### **8\. Membrane Pushbutton (Reset / Pairing / Sleep)**

One leg → GPIO 0  
Other leg → GND  
Use internal pull-up on GPIO

## **Light Sensor**

### **9\. LDR (Photoresistor) Voltage Divider**

LDR (light-dependent resistor)  
\- One leg → 3.3V  
\- Other leg → junction with 10kΩ resistor and GPIO 6

10kΩ resistor  
\- One leg → GND  
\- Other leg → GPIO 6 (shared with LDR)

Result: Analog voltage input on GPIO 6 based on ambient light level

## **Power Domains Summary**

| Voltage | Components |
| ----- | ----- |
| 120V AC | HLK-PM12 input |
| 12V DC | COB LED anode, TP4056 IN+ |
| 5V DC | Optional step-down from 12V for TP4056 |
| 3.7V DC | Battery (from TP4056 OUT+, to LDO) |
| 3.3V DC | ESP32, LDR, status LED, button pull-up |

---

## **Test Points**

* TP1: 12V out of HLK-PM12

* TP2: TP4056 OUT+

* TP3: Battery \+ (verify charge)

* TP4: Regulator VOUT (should be 3.3V)

* TP5: ESP32 RX/TX lines for USB debugging

* TP6: Gate of MOSFET

* TP7: GPIO 6 analog voltage from LDR

