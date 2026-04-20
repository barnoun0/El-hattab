# Maze-Solving Robot (MPU6050 + PID Control)

An autonomous maze-solving robot built with Arduino, utilizing three ultrasonic sensors for spatial awareness and an MPU6050 IMU for precise directional control. The robot employs a wall-following algorithm combined with a dual-PID system to maintain stability and accuracy that used to compete in insat robolympix 4.0 in the autonomous challenge.

## 🛠 Navigation Logic: The Algorithm

The robot implements the **Right-Hand Rulex (Wall Follower). This is an "uninformed" maze-solving algorithm where the robot treats the maze walls as a single continuous surface, keeping the right wall at a constant relative distance.

### Decision Priority:
1. **Turn Right:** If the right sensor detects an opening (`dist > 24cm`), the robot moves forward slightly to clear the corner, then executes a $-79^\circ$ pivot.
2. **Drive Forward:** If a right turn isn't possible but the front path is clear, the robot continues forward using PID stabilization.
3. **Turn Left:** If both right and front paths are blocked but the left is open, the robot executes a $+79^\circ$ pivot.
4. **U-Turn:** If the robot hits a dead end (all three sides blocked), it performs a $\approx 165^\circ$ pivot to backtrack.

---

## 🧠 PID Control Implementation

The project uses two distinct PID controllers to handle different movement states.

### 1. Straight-Line "Walking" PID
When moving forward, the robot uses a full PID loop to ensure it travels in a perfectly straight line, correcting for motor inconsistencies or physical bumps.

* **Error calculation:** $Error = TargetAngle - CurrentYaw$.
* **Proportional (Kp):** Corrects the heading based on the current deviation.
* **Integral (Ki):** Eliminates steady-state error (e.g., if one motor is inherently weaker than the other).
* **Derivative (Kd):** Dampens the correction to prevent the robot from oscillating (wobbling) back and forth.
* **Output:** The correction is added to the left motor speed and subtracted from the right, nudging the robot back to the $0^\circ$ reference.

### 2. Rotational "Pivot" PID
Precise $90^\circ$ turns are difficult with simple timers due to battery voltage fluctuations. The `pivotToAngle()` function uses a **PD (Proportional-Derivative) controller** to turn accurately.

* **Dynamic Speed:** As the robot approaches the target angle, the error decreases, naturally slowing the motors down. This prevents overshooting the turn.
* **Inertia Compensation:** The `d_turn` (Derivative) component acts as a "brake," helping the robot stop exactly at the desired angle despite the momentum of the chassis.
* **Targeting:** The robot updates its global `tangle` variable after every turn, ensuring the "Walking PID" always has a fresh, accurate reference for the next straight path.

---

## 🔌 Hardware Mapping

| Component | Pin | Function |
| :--- | :--- | :--- |
| **MPU6050** | I2C (SDA/SCL) | Gyroscopic Heading |
| **Ultrasonic Front** | Trig 12 / Echo 13 | Obstacle detection |
| **Ultrasonic Left** | Trig 4 / Echo 10 | Wall detection |
| **Ultrasonic Right** | Trig 7 / Echo 8 | Wall detection |
| **Left Motor** | PWM 6 (Fwd), 3 (Bwd) | Locomotion |
| **Right Motor** | PWM 5 (Fwd), 9 (Bwd) | Locomotion |
| **Start Button** | 2 | Calibration & Launch |
| **LED** | 11 | Status Indicator |

---

## 🚀 Setup and Usage

1.  **Calibration:** On power-up, the robot waits for the `startButton`. During this time, the MPU6050 is calibrated. The robot must remain perfectly still.
2.  **PID Tuning:** * Adjust `Kp`, `Ki`, and `Kd` to stabilize forward walking.
    * Adjust `p_turn` and `d_turn` to sharpen the $90^\circ$ and $180^\circ$ maneuvers.
    * Modify `distw` (24cm) to change the sensitivity for wall detection.
3.  **Operation:** Once the button is pressed, the LED indicates the robot's state (turning vs. walking) as it solves the maze.
