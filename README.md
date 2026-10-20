# Edge-AI Condition-Based Monitoring (CBM) Node

## Problem Statement
One of the most well-known disadvantages of traditional industrial strategies is reactive maintenance. This traditional method requires scheduled downtime and human intervention, which reduces productivity and poses high safety risks for factory workers.

## General Concept
It is a project inspired by Industry 4.0 core strategies and technologies, enabling real-time decision-making, high precision, and modular flexibility. 

* **Core Strategies:** Condition-Based Monitoring (CBM) & Predictive Maintenance (PdM), Edge Computing (Edge-AI), Industrial Internet of Things (IIoT), Cyber-Physical Systems (CPS).
* **Core Tools:** Edge Hardware (ESP32-S3 WROOM-1 N16R8), Sensor Fusion (MPU6500 6-DOF IMU, ACS712 5A Current Sensor), Firmware Development Tools (C++ / Arduino Framework, PlatformIO), Machine Learning (Scikit-Learn, m2cgen), Industrial IoT (MQTT & JSON).

## System Architecture
To realize this system, we used 3 main core subsystems designed for data acquisition, local edge processing, and IIoT telemetry:

1. **Mechanical & Electrical Sensing:**
   This system features an ESP32 running C++ firmware alongside two multi-domain sensors to measure structural vibrations (mechanical) captured via the MPU6500 sensor and current consumption using the ACS712 5A sensor (electrical).

2. **Localized Edge-AI Inferencing (No Cloud):**
   Raw sensor data is processed locally on the ESP32-S3 chip using a machine learning classification model (Scikit-Learn). Without relying on a network connection, the system autonomously categorizes machine health into clear operational states (*Optimal, Warning, Critical Shutdown*) in milliseconds.

3. **Industrial IoT (IIoT):**
   It streams live diagnostic data packets in JSON format over Wi-Fi using the MQTT protocol, feeding real-time monitoring dashboards for remote supervision.

---

## Hardware Architecture
1. **Components List:**
   * **Microcontroller:** ESP32-S3 WROOM-1 N16R8 (16MB Flash, 8MB PSRAM)
   * **Vibration Sensor:** MPU6500 (6-DOF Accelerometer/Gyroscope, I2C Interface)
   * **Current Sensor:** ACS712 5A Module (Analog Hall-effect current sensor)

2. **Pinout Connections (ESP32-S3):**
   * **MPU6500 (I2C Communication):**
     * `VCC` $\rightarrow$ `3.3V` (ESP32)
     * `GND` $\rightarrow$ `GND` (ESP32)
     * `SDA` $\rightarrow$ `GPIO 8` (default I2C SDA pin)
     * `SCL` $\rightarrow$ `GPIO 9` (default I2C SCL pin)
   * **ACS712 5A (Analog Output):**
     * `VCC` $\rightarrow$ `5V` (External supply / USB 5V line)
     * `GND` $\rightarrow$ `GND` (Common Ground)
     * `OUT` $\rightarrow$ **Voltage Divider** $\rightarrow$ `GPIO 4` (ESP32 ADC Pin)

---

## Edge-AI Workflow & Logic
1. **Data Collection Pipeline:**
   * Firmware runs a loop every 100ms reading MPU6500 and ACS712 5A.
   * Serial monitor streams data formatted as CSV strings.
   * Python script (`pyserial`) logs incoming data into `sensor_data.csv` under distinct states:
     * `0`: Optimal
     * `1`: Warning 
     * `2`: Critical 

2. **Machine Learning Training (Scikit-Learn):**
   * Dataset processed in Python using a **Decision Tree Classifier**.
   * Input features: `[Current, Accel_X, Accel_Y, Accel_Z]`.
   * Output labels: `[0, 1, 2]`.

3. **Edge Deployment (m2cgen):**
   * Trained Python model converted into pure C++ code via `m2cgen` into a header file (`model.h`).
   * Embedded directly into ESP32-S3 `main.cpp` firmware to perform real-time low-latency local inferencing without external cloud dependency.
