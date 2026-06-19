# MPU-6050 6-DOF IMU Data Acquisition & Teleoperation Pipeline

## Overview
This repository provides a robust, high-frequency data acquisition pipeline for the **MPU-6050 6-DOF IMU** (Inertial Measurement Unit). It is specifically designed to read, process, and transmit spatial orientation data (Quaternions and Euler angles) for robotic kinematics, digital twin simulations (e.g., Unity 3D), and autonomous navigation systems.

By leveraging the MPU-6050's onboard **Digital Motion Processor (DMP)**, this implementation offloads complex sensor fusion calculations from the main microcontroller, ensuring high-speed and drift-minimized orientation tracking.

## Key Features
* **Hardware-Level Sensor Fusion:** Utilizes the MPU-6050's DMP to calculate 6-axis data internally, significantly reducing the microcontroller's computational load.
* **Gimbal-Lock Free Orientation:** Outputs data in **Quaternions** (`w, x, y, z`), making it directly compatible with 3D engines like Unity and ROS 2 TF trees without the risk of Gimbal Lock.
* **Automated Calibration:** Includes a zero-drift calibration routine on startup to eliminate static offset errors in gyroscope and accelerometer readings.
* **High-Speed Data Pipeline:** Configured for fast I2C communication (400kHz) and optimized serial/network transmission (UDP/Serial) for real-time teleoperation.

## Hardware Architecture & Wiring

The system uses standard I2C communication. Ensure your microcontroller's logic level matches the MPU-6050 module (typically 3.3V, though many breakout boards include voltage regulators).

| MPU-6050 Pin | MCU Connection (e.g., ESP32) | Description |
| :---: | :--- | :--- |
| **VCC** | 3.3V (or 5V depending on board) | Power Supply |
| **GND** | GND | Common Ground |
| **SCL** | GPIO 22 (Default I2C SCL) | I2C Clock Line |
| **SDA** | GPIO 21 (Default I2C SDA) | I2C Data Line |
| **INT** | GPIO 2 (Interrupt Pin) | Required for DMP data ready interrupts |

> **Note on Interrupts:** The `INT` pin is crucial for DMP operation. It signals the microcontroller exactly when a new packet of processed orientation data is ready, preventing redundant polling and ensuring timing accuracy.

## Software Dependencies

This project relies on the widely adopted I2Cdev library ecosystem for robust I2C communication and DMP configuration.

* [I2Cdev Library](https://github.com/jrowberg/i2cdevlib)
* `MPU6050_6Axis_MotionApps20.h` (Included in the I2Cdev repo)
* Standard `Wire.h` (Built into the Arduino/ESP core)

## Data Pipeline & Integration (Unity / ROS 2)

This module is designed to act as the front-end data publisher for downstream kinematic controllers.

1.  **Read:** The MPU-6050 samples physical motion at high frequency.
2.  **Process:** The onboard DMP fuses Accel and Gyro data into Quaternions.
3.  **Interrupt:** The `INT` pin triggers the microcontroller.
4.  **Transmit:** The MCU parses the FIFO buffer and packages the data into a standardized payload (e.g., JSON or comma-separated string `w,x,y,z`).
5.  **Receive:** The host machine (Unity Engine via Serial/UDP, or ROS 2 node) receives the Quaternion and applies it to the virtual object's transform.

### Example Serial Output Format (Quaternion):
```text
q_data:0.7071,0.0000,0.7071,0.0000
q_data:0.7070,0.0010,0.7069,0.0015
