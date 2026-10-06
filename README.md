## Project Overview: Autonomous Obstacle-Avoiding Car

This project is an autonomous robotics platform designed to navigate an environment without human intervention. By utilizing an ultrasonic sensor as its "eyes," the car continuously measures the distance to objects in front of it. When it detects an obstacle within a critical threshold (20 cm), the onboard microcontroller interrupts the forward movement, commands the motor driver to reverse slightly, and executes a turn to clear the path before resuming its forward trajectory.

The system uses pulse-width modulation (PWM) to regulate motor speed, ensuring smooth turns and controlled straight-line movement.

---

## Hardware Components

* **Arduino Uno R3:** The central processing unit of the car. It runs the logic loop, triggers the sensor, calculates distances based on sound wave return times, and outputs specific digital and PWM signals to the motor driver.
* **L298N Dual H-Bridge Motor Driver:** An intermediate power control module. Because the Arduino cannot provide enough current to drive DC motors directly, the L298N takes low-current logic signals from the Arduino and acts as a heavy-duty switch, delivering high current from the battery to the motors.
* **HC-SR04 Ultrasonic Sensor:** A sonar device that emits high-frequency sound waves (Trig) and listens for their reflection (Echo). The time it takes for the sound to bounce back is used to calculate the exact distance to an object.
* **DC Gear Motors & Chassis:** The physical drive system (usually a 2-wheel or 4-wheel drive kit) responsible for moving the vehicle.
* **Independent Power Supplies:**
* *Motor Power:* A higher-capacity battery (e.g., 9V to 12V pack) dedicated to the L298N for motor power.
* *Logic Power:* A separate power source (e.g., 9V battery or 5V USB power bank) dedicated to keeping the Arduino running stably without voltage drops caused by the motors.



---

## Pin Configuration and Wiring

Because the system uses independent power supplies instead of the L298N's onboard 5V regulator, establishing a common ground between the boards is the most critical part of the wiring.

### 1. Power and Ground Routing

| Component | Connection Point | Purpose |
| --- | --- | --- |
| **Motor Battery (+)** | L298N `12V` Terminal | Supplies raw power for the motors. |
| **Motor Battery (-)** | L298N `GND` Terminal | Closes the motor power circuit. |
| **Arduino Power** | Arduino `Barrel Jack` or `USB` | Supplies clean power to the logic board. |
| **Common Ground Wire** | L298N `GND` to Arduino `GND` | **Crucial:** Synchronizes the voltage reference so logic signals work. |
| **L298N 5V Terminal** | *Left Empty* | Regulator output is bypassed. |

### 2. Speed and Direction Control (Arduino to L298N)

The ENA and ENB jumpers on the L298N must be removed to allow the Arduino's digital PWM pins to dictate the speed.

| L298N Pin | Arduino Pin | Function |
| --- | --- | --- |
| **ENA** | Digital Pin `~10` | Speed control for Left Motor(s) via PWM. |
| **IN1** | Digital Pin `4` | Left Motor direction (Forward logic). |
| **IN2** | Digital Pin `5` | Left Motor direction (Reverse logic). |
| **IN3** | Digital Pin `6` | Right Motor direction (Forward logic). |
| **IN4** | Digital Pin `7` | Right Motor direction (Reverse logic). |
| **ENB** | Digital Pin `~11` | Speed control for Right Motor(s) via PWM. |

### 3. Obstacle Detection (Arduino to HC-SR04)

| Sensor Pin | Arduino Pin | Function |
| --- | --- | --- |
| **VCC** | `5V` | Powers the sensor. |
| **GND** | `GND` | Closes the sensor circuit. |
| **Trig** | Digital Pin `8` | Outputs the ultrasonic pulse. |
| **Echo** | Digital Pin `9` | Receives the reflected sound wave. |

---

## Operational Logic

1. **Scanning:** Every 50 milliseconds, the Arduino sends a 10-microsecond pulse to the Trig pin. The Echo pin times how long the sound takes to return.
2. **Cruising:** If the calculated distance is greater than 20 cm, the Arduino sends a PWM value (e.g., 150) to ENA and ENB, and sets IN1 and IN3 HIGH to drive the car forward.
3. **Evasion:** If the distance drops below 20 cm:
* *Halt:* ENA and ENB are set to 0.
* *Reverse:* IN2 and IN4 are set HIGH to back away from the object.
* *Turn:* IN1 (Left forward) and IN4 (Right reverse) are set HIGH to spin the chassis to the right, pointing the sensor toward a new, clear path.
