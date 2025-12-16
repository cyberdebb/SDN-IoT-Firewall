# Intelligent IoT Firewall with SDN-Inspired Architecture

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![PlatformIO](https://img.shields.io/badge/PlatformIO-Build%20%26%20Flash-orange)
![Python](https://img.shields.io/badge/Python-3.9.13-blue.svg)
![Framework: Flask](https://img.shields.io/badge/Framework-Flask-green.svg)
![Language: C++](https://img.shields.io/badge/Language-C++-purple.svg)

**Authors:** July Chassot and Débora Castro  
**Course:** Wireless Networks – Computer Engineering (UFSC)

---

## About the Project

This project is a complete, low-cost solution for **Internet of Things (IoT) network security**. It implements a dynamic firewall using **Software-Defined Networking (SDN)** concepts to detect, visualize, and mitigate threats in real time.

The system is composed of ESP32 microcontrollers and a Raspberry Pi, working in an orchestrated manner to separate monitoring functions (Data Plane) from decision-making functions (Control Plane). The solution culminates in an interactive web dashboard for management and attack simulation.

---

## Concepts and Architecture

Inspired by SDN principles, this project decouples network intelligence from devices that merely forward data. While traditional SDN architectures use controllers such as **Ryu** and protocols like **OpenFlow**, this solution adopts a lighter and customized approach tailored for IoT environments, using **WebSockets** for communication between the control and data planes.

The architecture is divided into three main components:

1. **Central Orchestrator (Raspberry Pi)**  
   The brain of the system. It hosts:
   - A **Web Control Dashboard** (frontend) for user interaction.
   - A **Northbound API** (Flask backend in Python) that receives commands from the dashboard and the sensor/firewall.
   - A **WebSocket Server** to send real-time rules to the Switch.
   - Modules for **attack simulation** (Ping Flood, MAC Spoofing) to test system responsiveness.

2. **Firewall/Sensor (ESP32)**  
   The “Northbound Client” of the network.
   - Operates in **promiscuous mode**, capturing and analyzing all Wi-Fi traffic.
   - Implements **embedded anomaly detection logic**.
   - When a threat is detected, it reports to the Orchestrator via `HTTP POST` and can locally **deauthenticate** malicious clients. Periodically, it can also send rule updates based on detected behavior.

3. **SDN Switch (ESP32)**  
   The muscle of the network (Data Plane).
   - Acts as a **transparent filtering bridge** in the network.
   - Receives blocking rules (ACL – Access Control List) from the Orchestrator via WebSocket.
   - Its sole responsibility is to **forward allowed packets** and **drop blocked packets**, without making complex decisions.
   - Operates in **Fail-Secure mode**: if it loses connection with the controller, it blocks all traffic by default.

---

## Features

- **Web Control Dashboard:** Graphical interface to manage the firewall and launch security tests.
- **Real-Time Rule Management:** Add MAC address blocking rules that are instantly applied to the network.
- **Active and Reactive Firewall:** The system not only filters traffic based on rules but also actively detects anomalies and reacts to them.
- **Attack Simulator:**
  - **Ping Flood:** Generates heavy traffic to test detection and blocking capabilities.
  - **MAC Spoofing:** Changes the MAC address of a network interface to test data link layer security.
- **Secure Architecture:** Communication between sensor and controller is validated via API key, with *Fail-Secure* operation on the switch.
- **Low Cost:** Implemented using affordable hardware components and open-source software.

---

## Required Hardware and Software

### Hardware
- 1x Raspberry Pi 3 (or newer) with Raspberry Pi OS.
- 2x ESP32 microcontrollers (development boards, e.g., ESP32 DevKitC).
- Micro-USB cables for power and programming.
- A Wi-Fi router to create the test network.
- Development computer (tested on Windows 11).

### Software
- **IDE:** VS Code with the PlatformIO IDE extension.
- **USB Driver:** CP2102 USB to UART Bridge Controller Driver (required for most ESP32 boards).
- **Python:** Version 3.9.13 or newer.
- **Python Libraries:**
  ```sh
  pip install Flask Flask-Cors websockets scapy
  ```
> Note: Some installations may encounter a dependency issue with eventlet. If you see ImportError: cannot import name 'ALREADY_HANDLED', fix it with:
   ```sh
   pip install eventlet==0.30.2
   ```

- Command-line tools: git.

---

## Installation and Configuration

Follow the steps below to replicate the environment.

### Step 1: Prepare the Development Environment

1. Install VS Code, Python 3.9+, Git, and the CP2102 drivers on your computer.
2. Inside VS Code, install the **PlatformIO IDE** extension from the marketplace.

---

### Step 2: Configure the Raspberry Pi (Orchestrator)

1. **Prepare the OS**  
   Use Raspberry Pi Imager to install *Raspberry Pi OS (32-bit)* on a MicroSD card.

2. **Remote Access**  
   After booting, access the Raspberry Pi terminal (locally or via SSH).

3. **Clone the Repository**
   ```sh
   git clone https://github.com/cyberdebb/Firewall-SDN.git
   cd Firewall-SDN/1_Raspberry
    ```
   
4. Configure Environment Variables
Create a .env file in the root of the 1_Raspberry folder. This file must not be committed to Git.
   ```sh
   API_KEY="YOUR_SHARED_SECRET_KEY"
   ```

5. Set Up the Python Environment
   ```sh
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

6. Start the Server
   ```sh
   python3 dashboard_api.py
   ```

> The server will run on port 5000. Take note of the Raspberry Pi IP address.

---

### Step 3: Configure the ESP32 Devices (Switch and Firewall)

For both ESP32 devices, the setup process is similar.

1. Credential Management
In each firmware folder (2_ESP32_Controlador and 3_ESP32_Switch), create a configs.h file inside the src/ directory using configs.h.example as a template. This file must be ignored by Git.

   ```cpp
   #define WIFI_SSID "YOUR_WIFI_NAME"
   #define WIFI_PASS "YOUR_WIFI_PASSWORD"
   #define CONTROLLER_URL "http://RASPBERRY_PI_IP:5000"
   #define WEBSOCKET_HOST "RASPBERRY_PI_IP"
   #define API_KEY "YOUR_SHARED_SECRET_KEY"
   ```

2. Configure platformio.ini
   ```ini
   [env:esp32dev]
   platform = espressif32
   board = esp32dev
   framework = arduino
   monitor_speed = 115200
   lib_deps =
       bblanchon/ArduinoJson@^6.21.2
   ```

3. Compile and Upload

- Open the firmware folder in VS Code with PlatformIO.
- Fill in src/configs.h with your network and Raspberry Pi information.
- Connect the ESP32 to your computer.
- Compile and upload the firmware using the PlatformIO controls.
- Repeat the process for the second ESP32.

--- 

## How to Use

1. Start the Services

- On the Raspberry Pi:
   ```sh
   python3 dashboard_api.py
   ```

- Power on both ESP32 devices.

2. Monitor Activity

- Open the PlatformIO Serial Monitor for the ESP32 Firewall/Sensor.
- Set the baud rate to 115200 to view traffic and attack detection logs.

3. Access the Dashboard

- Connect your computer to the same Wi-Fi network as the Raspberry Pi.
- Open a browser and navigate to:
   ```cpp
   http://<RASPBERRY_PI_IP>:5000
   ```

4. Block a Device Manually

- Enter the target device MAC address in the TARGET MAC ADDRESS field.
- Click BLOCK MAC.
- The rule will be applied in real time and appear in the active rules list.

5. Simulate an Attack

- Enter a device IP address in the TARGET IP FOR FLOOD field.
- Click START PING FLOOD.
- Observe the ESP32 Firewall/Sensor logs as the attack is detected and mitigated.
- Click STOP ATTACK to end the simulation.

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.

---

## Autors

* **July Chassot** - [GitHub](https://github.com/LastChassot)
* **Débora Castro** - [GitHub](https://github.com/cyberdebb) 

*Feito com ❤️ e ☕ e ☕ e mais ☕ para a disciplina de Redes Sem Fio.*
