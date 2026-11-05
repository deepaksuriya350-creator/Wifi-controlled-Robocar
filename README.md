# SERBOT – ESP32 Wi-Fi Controlled Robot Car

A browser-controlled robotic car built using an **ESP32**, an **L298N motor driver**, and **four DC motor connections (two motor channels)**. The ESP32 creates its own Wi-Fi access point and hosts a web dashboard that can be opened from a phone or laptop. The dashboard provides movement controls, live communication statistics, trial calculations, and CSV export.

This project was developed for **21MEO106T – Introduction to Robotics** at **SRM Institute of Science and Technology, Kattankulathur**.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Software Requirements](#software-requirements)
- [System Architecture](#system-architecture)
- [Circuit Connections](#circuit-connections)
- [Installation and Setup](#installation-and-setup)
- [How to Operate](#how-to-operate)
- [Dashboard and Data Logging](#dashboard-and-data-logging)
- [Performance Evaluation](#performance-evaluation)
- [Literature Review](#literature-review)
- [Safety Notes](#safety-notes)
- [Limitations and Future Improvements](#limitations-and-future-improvements)
- [Repository Contents](#repository-contents)
- [References](#references)
- [Project Team](#project-team)

---

## Project Overview

SERBOT is a Wi-Fi-controlled robotic car that can be operated through a web browser without requiring a separate mobile application, an internet connection, or a cloud service.

The ESP32 operates as a Wi-Fi access point and runs a local web server. A user connects a phone or laptop to the robot's Wi-Fi network and opens the control dashboard. Commands from the dashboard are sent to the ESP32, which controls the motors through the L298N motor driver.

The project also includes a data-logging dashboard for collecting communication measurements such as round-trip time (RTT), Wi-Fi signal strength (RSSI), packet loss, and ESP32 command-processing time. Users can export collected records as a CSV file for further analysis.

## Objectives

1. Develop a low-cost Wi-Fi-controlled robotic car.
2. Control the robot through a browser-based interface.
3. Use an ESP32 and L298N motor driver to control the DC motors.
4. Measure communication performance using RTT, RSSI, packet loss, and latency statistics.
5. Compare theoretical and measured vehicle speed and turning time.
6. Export experimental records for analysis.
7. Identify limitations in existing Wi-Fi robot-car implementations and propose further improvements.

## Features

### Robot control

- Forward movement
- Backward movement
- Left and right steering while moving forward
- Stop command
- Touch/pointer-based browser controls
- Keyboard controls using W, A, S, and D
- Responsive dashboard for phone and laptop browsers

### Communication monitoring

- Command round-trip time (RTT)
- Mean and standard deviation of RTT
- Minimum and maximum RTT
- 95th-percentile RTT
- Jitter
- Packet-loss percentage
- IQR-filtered mean RTT
- ESP32 command-processing time
- Wi-Fi RSSI
- ESP32 uptime, command count, connected-client count, and free heap

### Experimental calculations

- Theoretical wheel speed
- Measured vehicle speed from distance and time
- Theoretical 90-degree turning time
- Absolute error and percentage error
- Mean absolute error (MAE)
- Root mean square error (RMSE)
- CSV export of collected communication records

---

## Hardware Requirements

| Component | Purpose |
|---|---|
| ESP32 development board | Main controller and Wi-Fi access point |
| L298N motor driver | Drives the DC motors |
| DC geared motors | Provide movement |
| Robot car chassis | Mechanical platform |
| Wheels | Vehicle movement |
| Battery pack | Motor power supply |
| USB cable / regulated supply | ESP32 programming and power |
| Jumper wires | Electrical connections |

The supplied implementation uses two motor-driver channels: one for the left side and one for the right side.

## Software Requirements

- Arduino IDE
- ESP32 board package for Arduino IDE
- ESP32 Arduino core compatible with the installed libraries
- A web browser on a phone or laptop

The Arduino sketch uses these libraries:

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <esp_wifi.h>
```

These are provided by the ESP32 Arduino environment.

---

## System Architecture

```text
Phone / Laptop
      |
      | Wi-Fi connection
      v
ESP32 Access Point
      |
      | Browser dashboard commands
      v
ESP32 Web Server
      |
      | GPIO motor-control signals
      v
L298N Motor Driver
      |
      v
DC Motors
      |
      v
Robot Car Movement
```

The ESP32 creates a local wireless network. The browser communicates directly with the ESP32 web server using HTTP requests. The internet is not required.

## Circuit Connections

The following table shows the pin definitions in the uploaded `SerBot_DataLogger.ino` sketch.

| L298N Pin | ESP32 Pin | Function |
|---|---:|---|
| IN1 | GPIO 26 | Left motor direction |
| IN2 | GPIO 27 | Left motor direction |
| IN3 | GPIO 14 | Right motor direction |
| IN4 | GPIO 12 | Right motor direction |
| ENA | GPIO 25 | Left motor enable |
| ENB | GPIO 15 | Right motor enable |
| GND | GND | Common ground |

**Important ENB wiring check:** The supplied PDF wiring guide specifies ENB on **GPIO 13** and asks for `#define ENB 13`, whereas the uploaded Arduino sketch currently defines `#define ENB 15`. These must match. Either wire ENB to GPIO 15 to match the unmodified sketch, or change the sketch to GPIO 13 if that is the actual connection on your car. Do not assume both configurations are interchangeable without checking the physical wiring.

**GPIO 12 note:** GPIO 12 is an ESP32 strapping pin. If the ESP32 fails to boot when the motor driver is connected, consider moving IN4 to an available suitable pin (the project guide suggests GPIO 33) and update the sketch accordingly.

---

## Installation and Setup

1. Install the Arduino IDE.
2. Install the ESP32 board package and select **ESP32 Dev Module**.
3. Connect the ESP32 board to the computer using USB.
4. Open `SerBot_DataLogger.ino`.
5. Check that the ENB pin definition matches the actual wiring.
6. Select the correct board and COM port.
7. Upload the sketch to the ESP32.
8. Open the Serial Monitor at **115200 baud**.
9. Confirm that the ESP32 starts its access point and prints its IP address.

The default Wi-Fi settings in the sketch are:

| Setting | Value |
|---|---|
| Wi-Fi network (SSID) | `RoboCar` |
| Password | `robot123` |
| Dashboard address | `http://192.168.4.1` |

Change the default password before using the project in an environment where other people could connect to the robot.

## How to Operate

1. Power the ESP32 and motor driver.
2. On a phone or laptop, connect to the Wi-Fi network **RoboCar**.
3. If the device warns that the network has no internet, remain connected.
4. Open a browser and visit **http://192.168.4.1**.
5. Use the dashboard buttons or keyboard controls.
6. Test the robot with its wheels raised before placing it on the floor.

### Controls

| Control | Action |
|---|---|
| FWD / W | Both motor channels move forward |
| BACK / S (dashboard button) | Both motor channels move backward |
| LEFT / A | Left side stops while right side drives forward |
| RIGHT / D | Right side stops while left side drives forward |
| STOP | Stops both motor channels |

For keyboard operation, W moves forward, S moves backward, A steers left, and D steers right. Releasing a movement key sends a stop command, except that releasing A or D while W remains held resumes forward movement.

The left and right controls are designed to steer while the robot is moving forward; they do not provide an independent reverse-turn mode.

---

## Dashboard and Data Logging

The dashboard is served directly by the ESP32. It contains three main areas.

### 1. Live link metrics

The dashboard records browser-side command RTT and periodically reads status information from the ESP32.

Displayed metrics include:

- Commands sent
- Mean RTT
- RTT standard deviation
- Minimum and maximum RTT
- P95 RTT
- Jitter (mean absolute difference between consecutive successful RTT samples)
- Packet loss
- IQR-filtered mean RTT and number of outliers removed
- ESP32 command-processing time
- Current client RSSI
- Pearson correlation between RSSI and RTT, when enough paired samples are available

The **Run latency test** button sends 50 STOP requests with a short interval between requests. This is intended as a non-driving communication test.

The distance field allows the operator to label samples with an approximate distance from the robot. The operator should update this value while moving away from the car if collecting range-related data.

### 2. Speed trials

Enter the motor RPM specification, wheel diameter, track width, and trial distance. Enter measured trial times as comma-separated values.

The dashboard calculates theoretical wheel speed and compares it with speed derived from the measured travel time.

### 3. 90-degree turn trials

Enter measured turning times as comma-separated values. The dashboard estimates a theoretical 90-degree turn time using the configured track width and theoretical wheel speed, then calculates error statistics.

### CSV export

Select **Download CSV** to export the collected command and RSSI records. The exported file contains fields such as timestamp, command, RTT, ESP32 processing time, RSSI, operator-entered distance, success flag, uptime, command count, and free heap.

The CSV file can be opened using spreadsheet software or processed using Python.

---

## Performance Evaluation

The project uses experimental records rather than a machine-learning training dataset. Each test trial can record the command, timestamps, distance, battery voltage, and expected versus observed outcome.

The following formulas are used in the project review and dashboard.

### Theoretical wheel speed

\[
v = \frac{\pi D N}{60}
\]

Where:

- \(v\) = theoretical wheel speed in m/s
- \(D\) = wheel diameter in metres
- \(N\) = motor speed in RPM

The dashboard uses the entered no-load motor RPM as the theoretical input. Actual loaded vehicle speed may differ.

### Measured vehicle speed

\[
v_{\text{measured}} = \frac{\text{distance travelled}}{\text{time taken}}
\]

### Mean RTT

\[
\overline{RTT} = \frac{\sum RTT_i}{n}
\]

### Packet loss

\[
\text{Packet loss (\%)} =
\frac{\text{Failed requests}}{\text{Total requests}}\times 100
\]

### Absolute error

\[
e = |x_{\text{measured}}-x_{\text{expected}}|
\]

### Percentage error

\[
e(\%) = \frac{e}{|x_{\text{expected}}|}\times 100
\]

### Mean absolute error

\[
MAE = \frac{1}{n}\sum_{i=1}^{n}|e_i|
\]

### Root mean square error

\[
RMSE = \sqrt{\frac{1}{n}\sum_{i=1}^{n}e_i^2}
\]

### Standard deviation

\[
s = \sqrt{\frac{\sum_{i=1}^{n}(x_i-\overline{x})^2}{n-1}}
\]

These metrics should be calculated from actual recorded trials. Illustrative values must not be presented as measured project results.

### Current evaluation status

The literature-review presentation records the following software evidence:

- 9 browser checks passed
- C++ motor state-machine test passed
- ESP32 firmware compiled successfully (reported build size: 962,923 bytes, ESP32 Arduino core 3.3.0)

Physical range, speed, latency, and other hardware trial measurements are marked as pending in the presentation. Update this section when actual test data is collected.

---

## Literature Review

The project review surveys three Wi-Fi-controlled robot-car implementations.

| Study | Main approach | Relevance to SERBOT |
|---|---|---|
| Karim et al. | Arduino Uno and ESP8266-based Wi-Fi spy robot car with camera streaming | Provides a baseline phone-to-Wi-Fi-to-motor control architecture |
| Badiger et al. | ESP32 browser-controlled surveillance car with camera and robotic arm | Demonstrates browser-based control and an expanded robot feature set |
| Pal et al. | ESP32 Wi-Fi robot car with ultrasonic obstacle detection and automatic stop | Provides reported range, distance accuracy, and obstacle-stop benchmarks |

### Identified research gaps

The literature review identifies three areas for improvement:

1. **Wi-Fi link loss:** The surveyed papers do not clearly describe motor behaviour when the wireless connection is lost during a movement command.
2. **Delayed or stale commands:** The review highlights the risk of delayed or duplicated requests causing unintended movement.
3. **Experimental evaluation:** Some studies provide qualitative results or measurements without clearly stating trial counts and measurement methods.

The project review proposes heartbeat-based stopping, command validation, and systematic experimental evaluation as responses to these gaps.

**Implementation status clarification:** The uploaded `SerBot_DataLogger.ino` currently implements HTTP command validation, browser-side RTT logging, and a STOP request when a movement control is released. However, this sketch does **not** currently contain the 600 ms heartbeat lease, session-token validation, or sequence-number rejection described in the literature-review presentation. Those items should be treated as proposed/reviewed safety mechanisms until they are implemented and tested in the firmware. In the current sketch, a lost Wi-Fi connection during movement may leave the last motor output active. Do not describe the 600 ms fail-safe as implemented in this version.

### Benchmarks reported in the reviewed literature

The presentation attributes the following results to Pal et al.:

- Indoor Wi-Fi range: 20–30 m
- Outdoor Wi-Fi range: 45–50 m
- Ultrasonic accuracy: ±1.5 cm up to 200 cm
- Obstacle-stop reaction time: 200–300 ms
- Torque reduction reported below 7 V

These are figures reported by the cited paper, not measurements from SERBOT.

---

## Safety Notes

- Test motor direction with the wheels raised before floor operation.
- Use a suitable battery pack for the L298N motor supply.
- Power the ESP32 through USB or an appropriately regulated supply.
- Never connect raw motor-battery voltage to the ESP32 3.3 V pin.
- Connect ESP32 and L298N grounds together.
- Switch off power before changing wiring.
- Secure loose wires and keep them away from wheels and moving parts.
- Check the ESP32 boot behaviour if GPIO 12 is connected to the motor driver.
- Keep the robot in a clear test area and be ready to disconnect motor power.
- The current uploaded firmware does not implement the 600 ms link-loss watchdog described in the presentation. Do not leave the robot unattended or rely on Wi-Fi disconnection to stop it.

---

## Limitations and Future Improvements

The following improvements are identified in the project review or follow from the current implementation:

- Implement and test a firmware-side communication timeout that disables the motors after link loss.
- Add session tokens and sequence numbers to reject stale or duplicated movement commands.
- Add an HC-SR04 ultrasonic sensor for obstacle detection and automatic stopping.
- Add wheel encoders for measured RPM and more accurate speed estimation.
- Collect repeated physical trials for each direction and operating condition.
- Measure control range, response time, and battery-voltage effects under documented test conditions.
- Improve the evaluation dataset by recording trial conditions and retaining failed trials.
- Validate the motor-driver wiring and pin definitions against the final assembled robot.

---

## Repository Contents

A suggested GitHub repository structure is:

```text
SERBOT/
├── README.md
├── SerBot_DataLogger.ino
├── docs/
│   ├── SerBot_WiFi_Robot_Car_Guide.pdf
│   ├── WiFi_Robot_Car_Literature_Review.pptx
│   └── WiFiCar_Review2.pdf
└── data/
    └── (Add exported CSV test records here)
```

The `docs/` and `data/` folders are suggested locations. Create them and add the corresponding files when uploading the project to GitHub. Do not upload private credentials or personal information in test records.

---

## References

1. N. Karim, M. A. Kamal, M. Maknojia, and S. Rafique, “Arduino Controlled Spy Robo Car using Wireless Camera With live Streaming,” *Journal of Information & Communication Technology (JICT)*, vol. 13, no. 1, pp. 6–9. arXiv:2108.03402.
2. P. Badiger, R. S. Borges, V. Holikatti, N. A. Kasalkar, and R. Kallur, “WiFi-Controlled Surveillance Car With Web Interface And Robotic Arm Integration,” *International Journal of Creative Research Thoughts (IJCRT)*, vol. 13, no. 5, May 2025, pp. l216–l224.
3. L. Pal, P. Kumar, and P. Kumar, “Design and Development of a Wi-Fi Controlled Robotic Vehicle with Ultrasonic Obstacle Detection,” *Spectrum of Emerging Sciences*, vol. 5, no. 2, 2025, pp. 31–36. DOI: 10.55878/SES2025-5-2-8.
4. Espressif Systems, Arduino-ESP32 Wi-Fi API and ESP32 GPIO documentation.
5. STMicroelectronics, L298 dual full-bridge driver datasheet.

---

## Project Team

**B.Tech – Computer Science and Engineering**  
**SRM Institute of Science and Technology, Kattankulathur**

- Saransh Sudan – RA2411003011302
- Yuvraj Singh Rana – RA2411003011300
- Deepak M – RA2411003011309
- Mahanth V – RA2411003011621

---

## License

No license has been specified for this repository. Add a license file if you intend to define permissions for reuse, modification, and distribution.
