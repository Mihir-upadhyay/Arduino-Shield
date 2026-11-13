# Arduino-Shield
# 🔌 Arduino Uno Custom Shield (V1)

A custom-designed **Arduino Uno shield** that adds an **I²C temperature sensor**, **MOSFET-driven DC motor speed control**, and **dual servo headers** in one plug-in board. It was designed in **EasyEDA**, fabricated as a PCB, assembled with SMD parts, and tested with the Arduino IDE.

![Platform](https://img.shields.io/badge/board-Arduino%20Uno%20R3-00979D)
![EDA](https://img.shields.io/badge/design-EasyEDA-blue)
![Interface](https://img.shields.io/badge/sensor-I%C2%B2C-green)
![Status](https://img.shields.io/badge/status-built%20%26%20tested-brightgreen)

<!-- Add the board photo here -->
<!-- ![Assembled shield](docs/images/shield_photo.jpg) -->

📄 **Full project report:** [`docs/FINAL_PROJECT_REPORT_fixed.pdf`](docs/FINAL_PROJECT_REPORT_fixed.pdf)

---

## ✨ Features

| Feature | Details |
|---|---|
| 🌡️ **Digital temperature sensor** | TI **TMP75AIDR** on the I²C bus (A4/SDA, A5/SCL), so no ADC is needed |
| ⚙️ **DC motor speed control** | MOSFET low-side driver with a flyback diode for PWM speed control from software |
| 🦾 **Dual servo control** | Two 3-pin servo headers, each with a decoupling capacitor |
| 🔋 **Selectable power** | Jumper (`EXT` / `5V`) chooses the Arduino 5 V rail or an external supply for servos and the motor |
| 💡 **Indicator LEDs** | One power LED and one user LED for your own code |
| 🧩 **Full pin breakout** | All Arduino headers broken out and labeled on the silkscreen |

Good for robotics, small automation tools, pan-tilt mounts, and walking robots.

---

## 🧱 Design Overview

### Schematic blocks

- **External power:** screw/JST-style connector with a 100 nF bulk cap
- **Power selection:** 3-pin jumper that picks `EXT` or `5V` for the servo and motor rail
- **Power LED:** resistor + SMD LED on 3.3 V
- **Temperature sensor:** TMP75AIDR with I²C pull-ups and a 100 nF decoupling cap
- **User LED:** SMD LED + resistor on a digital pin
- **MOSFET header:** gate resistor and pull-down, N-channel MOSFET, and flyback diode (SIA-13-F) for the DC motor
- **Servo 1 / Servo 2:** 3-pin headers with decoupling capacitors

> Exact pin assignments are in the schematic. Check them before writing code, since your pin numbers may differ if you modify the design.

### PCB

- 2-layer board that matches the **Arduino Uno R3** shield outline and mounting holes
- Large copper area around the power section, with labeled headers and a signal-annotated silkscreen
- Designed in EasyEDA and checked with **DRC** before fabrication

<!-- Add schematic + PCB images here -->
<!-- ![Schematic](docs/images/schematic.png) -->
<!-- ![PCB layout](docs/images/pcb_layout.png) -->
<!-- ![PCB 3D](docs/images/pcb_3d.png) -->

---

## 🧾 Bill of Materials

| # | Component | Part / Value |
|---|---|---|
| 1 | SMD multilayer ceramic capacitor | CL21B104KCFNNNE (100 nF) |
| 2 | Standard recovery diode | SIA-13-F |
| 3 | Resistor 10 kΩ | 0805WBF5600T5E |
| 4 | Resistor 0 Ω | 0805WBF0000T5E |
| 5 | Resistor (LED) | RC0805FR-07180RL |
| 6 | Female stackable header | 8-pin and 10-pin |
| 7 | Temperature sensor | TMP75AIDR |
| 8 | SMD LEDs (red, green) | 0805 |

> The MOSFET, jumper, power and servo headers are also visible on the board. Add them to this table with exact part numbers from your EasyEDA BOM.

### Cost

| Item | Qty | Cost (₹) |
|---|---|---|
| PCB (shield) | 1 | 200 |
| Resistors and capacitors | 7 | 20 |
| Female headers (8-pin, 10-pin) | 5 | 130 |
| Arduino board | 1 | 600 |
| **Total** | | **₹950** |

---

## 🛠️ Build & Test

### Build

1. Open the EasyEDA project, or order directly from the Gerber files in `hardware/gerber/`.
2. Solder the SMD parts first (TMP75, resistors, capacitors, diode, LEDs), then the MOSFET and the through-hole headers and jumper.
3. Visually inspect the board and test continuity. Check for shorts between 5 V and GND **before** first power-on.
4. Plug the shield onto the Arduino Uno.

### Test with the Arduino IDE

1. **LEDs:** blink the user LED, and confirm the power LED is on.
2. **Temperature sensor:** run an I²C scanner first (the TMP75 address depends on its A0–A2 pins), then read the temperature.
3. **Servos:** sweep with the `Servo` library on both headers.
4. **DC motor:** drive the MOSFET gate pin with `analogWrite()` to vary speed.

Minimal I²C temperature read (adjust the address to match your board):

```cpp
#include <Wire.h>

const uint8_t TMP75_ADDR = 0x48;   // default with A0–A2 tied to GND; verify with an I2C scanner

void setup() {
  Serial.begin(9600);
  Wire.begin();
}

void loop() {
  Wire.beginTransmission(TMP75_ADDR);
  Wire.write(0x00);                       // temperature register
  Wire.endTransmission();
  Wire.requestFrom(TMP75_ADDR, (uint8_t)2);
  if (Wire.available() == 2) {
    int16_t raw = (Wire.read() << 8) | Wire.read();
    float tempC = (raw >> 4) * 0.0625;    // 12-bit, 0.0625 °C/LSB
    Serial.print("Temperature: ");
    Serial.print(tempC);
    Serial.println(" °C");
  }
  delay(1000);
}
```

> ⚠️ **Power note:** Don't power motors or several servos from the Arduino's 5 V pin. Set the jumper to `EXT` and use a separate supply (common ground with the Arduino) for anything that draws significant current.

---

## 📁 Suggested Repository Structure

```
.
├── README.md
├── docs/
│   ├── FINAL_PROJECT_REPORT_fixed.pdf
│   └── images/            # schematic, PCB layout, 3D render, assembled photo
├── hardware/
│   ├── easyeda/           # exported EasyEDA project (.json)
│   ├── gerber/            # Gerber + drill files
│   └── bom.csv
├── firmware/
│   ├── led_test/
│   ├── temp_sensor_test/
│   ├── servo_test/
│   └── motor_pwm_test/
└── LICENSE
```

---

## 🚀 Future Improvements

- Add a motor-driver IC (H-bridge) for direction control
- Add more I²C headers (Qwiic/Grove style)
- Reverse-polarity and overcurrent protection on the external power input
- Optional wireless module footprint (ESP-01 / Bluetooth)

---

## 📚 References

- Texas Instruments, [TMP75 datasheet](https://www.ti.com/lit/gpn/tmp75)
- SparkFun, *Introduction to Arduino Shields*
- EasyEDA documentation (schematic → PCB → Gerber workflow)

---

## 📜 License

Add a license of your choice (for example MIT for firmware and CERN-OHL for hardware) in a `LICENSE` file.
