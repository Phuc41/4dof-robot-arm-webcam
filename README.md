# 4-DOF Gesture-Controlled Robotic Arm using MediaPipe & ESP32 via WebSocket

[](https://www.espressif.com/)
[](https://www.python.org/)
[](https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker)
[](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
[]()
[License](https://www.google.com/search?q=LICENSE)

> **Academic Project:** Introduction to Engineering — Ho Chi Minh City University of Technology (HCMUT - VNU-HCM)  

---

## 📌 Project Overview

This project presents a **4-Degree-of-Freedom (4-DOF) Robotic Arm** capable of real-time human hand tracking and touchless teleoperation. By integrating a standard monocular webcam with Google's **MediaPipe Hands** framework, the system detects and extracts 21 three-dimensional hand landmarks, maps the normalized coordinates into joint angles, and streams actuation commands over a low-latency local **WebSocket** connection to an **ESP32** microcontroller driving 4 micro servo motors.

The architecture addresses the challenge of building an intuitive, non-contact human-machine interface (HMI) while ensuring sub-150ms real-time responsiveness and electrical power stability using cost-effective off-the-shelf components.

---

## 🎥 Live Demonstration

https://github.com/user-attachments/assets/f1f4b809-8819-4cd5-a519-aa21600221fd

*Figure 1: Full demonstration of real-time gesture tracking and 4-DOF robotic arm manipulation.*

---

## ⚡ Key Highlights & Engineering Features

* **Touchless Intuitive Teleoperation:** Eliminates the need for physical joysticks, wearable flex sensors, or glove-based rigs by relying entirely on vision-based hand tracking.
* **Low-Latency Wireless Streaming:** Implements a full-duplex WebSocket server on ESP32 running in SoftAP mode, achieving a deterministic end-to-end latency of **100 ms – 150 ms** over local Wi-Fi.
* **Dedicated High-Current Power Rail:** Resolves the notorious servo-induced brownout reset problem on microcontrollers by isolating the dynamic motor load with a 4-cell 18650 Li-ion battery pack, an **XL4015 5A DC-DC Buck Converter**, and a **1000 µF / 16V decoupling capacitor**.
* **Distributed Computing Architecture:** Offloads intensive computer vision inference to the host PC while reserving ESP32 firmware resources strictly for network parsing and hardware-timer-based PWM generation.

---

## 🏗️ System Architecture

```
[ Monocular Camera / Webcam ]
              │  (Raw Video Stream: RGB)
              ▼
[ Host PC / Laptop: Python Client ]
   ├── MediaPipe Hands Pipeline (21 3D Joint Landmarks)
   ├── Dynamic Gesture Trigger (Euclidean Distance: Thumb vs. Index Tip < 30px)
   └── Linear Coordinate Mapping (Y-Pixel Coordinate -> Joint Angle 0° - 180°)
              │
              │  (Serialized WebSocket Frame: "Key,Value" via Wi-Fi SoftAP)
              ▼
[ ESP32 WROOM-32D: Embedded WebSocket Server ]
   ├── Packet Deserializer / Lexical Parser
   └── LEDC Hardware PWM Generator (50 Hz, 500 µs – 2400 µs Pulse Width)
              │
              ▼
[ Actuation Subsystem: 4x TowerPro MG90S Servos ]
   ├── Joint 1: Base Rotation  ──────> GPIO 22
   ├── Joint 2: Shoulder Pitch ──────> GPIO 21
   ├── Joint 3: Elbow Pitch    ──────> GPIO 19
   └── End-Effector: Gripper   ──────> GPIO 18
```

---

## 🔌 Hardware Specifications & Pin Mapping

### 1. ESP32 Pin Allocation (DevKit V1 - 38 Pins)

| Actuator / Joint | ESP32 GPIO Pin | Signal Type | PWM Frequency | Pulse Width Range | Operating Range |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Base Joint (Yaw)** | `GPIO 22` | Hardware PWM | 50 Hz (20 ms) | 500 µs – 2400 µs | 0° – 180° |
| **Shoulder Joint (Pitch)** | `GPIO 21` | Hardware PWM | 50 Hz (20 ms) | 500 µs – 2400 µs | 0° – 180° |
| **Elbow Joint (Pitch)** | `GPIO 19` | Hardware PWM | 50 Hz (20 ms) | 500 µs – 2400 µs | 0° – 180° |
| **End-Effector (Gripper)** | `GPIO 18` | Hardware PWM | 50 Hz (20 ms) | 500 µs – 2400 µs | 0° – 180° |

### 2. Power Delivery Network (PDN) Design
* **Primary Battery Pack:** 4x 18650 Li-ion cells arranged in a hybrid series-parallel configuration (nominal voltage: 7.4V – 8.4V), delivering high continuous discharge capability (>5A).
* **Step-Down Regulation:** XLSEMI XL4015 DC-DC Buck converter stepping down 7.4V–8.4V to a regulated **5.0V DC (up to 5A peak)** dedicated to the servo rail.
* **Transient Decoupling:** A **1000 µF / 16V low-ESR electrolytic capacitor** is connected in parallel across the 5V servo power rail to buffer inrush currents during rapid motor acceleration and prevent MCU voltage drops.

---


## 📊 Experimental Results & Performance Benchmarks

| Evaluation Metric | Measured Benchmark | Technical Analysis |
| :--- | :---: | :--- |
| **End-to-End Latency** | **100 ms – 150 ms** | Satisfies real-time interactive requirements with smooth visual feedback and zero frame drops. |
| **Positioning Accuracy** | **±0.5 cm** | Primary error sources include hand tremor, landmark jitter, and slight gear backlash in micro servos. |
| **Payload Capacity** | **~ 100 grams** | Reliably clamps and manipulates small objects (foam blocks, plastic bottles, components). |
| **Power Stability** | **100% (Zero Resets)** | Verified zero microcontroller resets under concurrent multi-servo stall conditions. |

---

## 💡 Engineering Insights & Future Roadmap

### Technical Achievements
1. **Hardware Power Isolation:** Eliminated USB power reliance, isolating inductive noise and high-draw inrush currents from the digital logic core.
2. **Decoupled Architecture:** Clean client-server separation enables independent scaling, algorithm upgrades, or porting to ROS/ROS2 without altering firmware interfaces.

### Known Limitations & Roadmap
* **Inverse Kinematics (IK) Integration:** Transition from individual joint angle control to full 3D Cartesian coordinates $(X, Y, Z)$ using analytical Inverse Kinematics.
* **Power Management Architecture:** Integrate a dedicated 2S Battery Management System (BMS) with cell balancing and low-voltage cutoff.
* **High-Torque Actuation:** Upgrade shoulder and base servos to metal-geared MG996R units to increase dynamic payload capacity beyond 300g.
* **Teaching & Playback Mode:** Implement non-volatile Flash recording on ESP32 to store and autonomously replay learned trajectory routines.

---

## 🚀 Getting Started

### Prerequisites
* Python 3.9+ with `pip`
* Arduino IDE (configured with ESP32 board support package)
* Hardware setup matching the schematic in `docs/images/schematic_circuit.png`

### 1. Python Client Setup
```bash
# Clone the repository
git clone https://github.com/<your-username>/gesture-controlled-4dof-robot-arm.git
cd gesture-controlled-4dof-robot-arm/software_python

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate       # On Windows: venv\Scripts\activate

# Install required packages
pip install -r requirements.txt
```

### 2. Firmware Flashing (ESP32)
1. Open `firmware_esp32/firmware_esp32.ino` in Arduino IDE.
2. Install required Arduino libraries via Library Manager:
   * `ESPAsyncWebServer`
   * `AsyncTCP`
   * `ESP32Servo`
3. Select your ESP32 board (`ESP32 Dev Module`) and target COM port.
4. Compile and flash the firmware.

### 3. Execution
1. Power up the robotic arm using the battery switch.
2. Connect your PC to the Wi-Fi network broadcast by ESP32 (Default Gateway: `192.168.4.1`).
3. Launch the Python tracking script:
   ```bash
   python hand_tracking_client.py
   ```
4. Place your hand within webcam view and pinch your thumb and index finger to initiate joint control.

---






---

## 📄 License
This project is licensed under the **MIT License** — see the [LICENSE](https://www.google.com/search?q=LICENSE) file for details.
