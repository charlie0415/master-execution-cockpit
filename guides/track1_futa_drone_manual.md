# Track 1: FUTA Drone FYP — Complete Hardware & Thesis Manual

**Project Title:** Design, Fabrication, and Flight Characterization of an Autonomous Quadcopter UAV for Aerial Inspection  
**Candidate:** Charles Oluwatuase | Mechatronics Engineering, FUTA  
**Strategy:** Spend minimum capital (~$147 – $185) and strictly Saturday time to achieve an unassailable A-grade defense without draining cognitive capacity for Track 2 & 3.

---

## 1. Subsystem Architecture & Pinout Breakdown

### Quad-X Motor Geometry & Rotation Directions
```
      Front of Drone (Forward ▲)
          [M3 CW]      [M1 CCW]
             \            /
              \   [FC]   /
               \        /
              /          \
             /            \
          [M2 CCW]     [M4 CW]
```

* **Motor 1 (Front Right):** Counter-Clockwise (CCW) — Black prop nut / Normal prop
* **Motor 2 (Rear Left):** Counter-Clockwise (CCW) — Black prop nut / Normal prop
* **Motor 3 (Front Left):** Clockwise (CW) — Silver prop nut / Pusher prop (1045R)
* **Motor 4 (Rear Right):** Clockwise (CW) — Silver prop nut / Pusher prop (1045R)

### Electrical & Power Distribution (PDB)
1. **XT60 Pigtail:** Solder 12AWG silicone wire directly to the F450 lower plate battery pads. **Check polarity with multimeter (Red = +, Black = -) before plugging in battery.**
2. **ESCs (30A SimonK):**
   * Solder power leads directly to the 4 corner pads on the F450 bottom board.
   * Strip signal wires: Pixhawk `MAIN OUT` 1 to 4 receive the 3-pin servo leads (Signal wire points DOWN toward bottom plate on Pixhawk 2.4.8).
3. **Power Module:** Plugs inline between 3S LiPo and bottom plate XT60. 6-pin cable feeds clean 5.3V to Pixhawk `POWER` port and provides voltage/current telemetry.

---

## 2. Firmware Flashing & Calibration (Mission Planner)

1. **Firmware:** Install **Mission Planner** on your laptop. Connect Pixhawk via USB. Under *Initial Setup > Install Firmware*, flash **ArduCopter V4.x (Quad-X)**.
2. **Frame Type:** Select **Frame Type: X**.
3. **Accelerometer Calibration:**
   * Place drone flat on table -> Press Calibrate.
   * Follow prompts: Left side, Right side, Nose down, Nose up, Back side. Keep completely still on each step.
4. **Compass Calibration:**
   * Mount GPS/Compass puck on the tall mast (minimum 10cm above battery/PDB to avoid magnetic interference).
   * In Mission Planner, click *Start Onboard Mag Calibration*. Rotate drone 360° on all axes until green bar completes.
5. **Radio Calibration (FlySky FS-i6 + FS-iA6B):**
   * Connect receiver via PPM or SBUS to Pixhawk `RC IN`.
   * Bind receiver (hold bind button on receiver while powering on, hold bind key on transmitter).
   * In Mission Planner, verify all 4 stick channels: Channel 1 (Roll), Channel 2 (Pitch), Channel 3 (Throttle), Channel 4 (Yaw).
   * Configure Flight Mode Switch on Channel 5:
     * Mode 1: **STABILIZE** (Manual leveling)
     * Mode 2: **ALT_HOLD** (Barometer holds height, manual roll/pitch)
     * Mode 3: **LOITER** (GPS + Barometer holds exact 3D position)
     * Mode 4: **RTL (Return-to-Launch)** (Failsafe return home)
6. **ESC Throttle Calibration:**
   * Remove ALL propellers.
   * Transmitter throttle to MAX -> Power on drone -> Pixhawk beeps -> Unplug battery -> Power on again -> Pixhawk tones play -> Press safety switch -> Lower throttle to MIN. Motors play confirmation tone.

---

## 3. Recommended F450 PID Tuning Parameters (3S LiPo + 1045 Props)

Safe default parameters in Mission Planner (*Config/Tuning > Basic Tuning*):

| Parameter | Recommended Value | Description |
| :--- | :--- | :--- |
| `ATC_RAT_RLL_P` | `0.135` | Roll angular rate P gain |
| `ATC_RAT_RLL_I` | `0.090` | Roll rate I gain (integrator) |
| `ATC_RAT_RLL_D` | `0.0036` | Roll rate D gain (damping) |
| `ATC_RAT_PIT_P` | `0.135` | Pitch angular rate P gain |
| `ATC_RAT_PIT_I` | `0.090` | Pitch rate I gain |
| `ATC_RAT_PIT_D` | `0.0036` | Pitch rate D gain |
| `ATC_RAT_YAW_P` | `0.180` | Yaw rate P gain |
| `ATC_RAT_YAW_I` | `0.018` | Yaw rate I gain |
| `MOT_SPIN_ARM` | `0.10` | Motor idle spin speed when armed |
| `FS_THR_ENABLE` | `1 (Enabled)` | RTL trigger on radio loss |
| `BATT_LOW_VOLT` | `10.5 V` | 3.5V/cell warning threshold |
| `BATT_CRT_VOLT` | `10.0 V` | Auto-Land critical threshold |

---

## 4. FUTA Mechatronics Thesis Structure (5-Chapter Template)

* **Chapter 1: Introduction & Problem Statement**
  * Background of unmanned aerial vehicles in civil and industrial inspection.
  * Limitations of manual inspection in complex environments.
  * Project objectives: Design, structural sizing, power budgeting, autonomous control, and flight verification.
  * Scope and project delimitation.
* **Chapter 2: Literature Review & Theoretical Modeling**
  * Review of multirotor aerial kinematics and Newton-Euler dynamic equations.
  * Quadcopter aerodynamic thrust equations ($T = C_T \rho n^2 D^4$).
  * Comparison of flight control hardware architectures (STM32 Cortex-M4 vs Atmel AVR).
  * Sensor fusion fundamentals: Extended Kalman Filter (EKF) combining IMU, barometer, and GNSS.
* **Chapter 3: Methodology & Hardware Implementation**
  * Mechanical design: CAD modeling of F450 frame, center of gravity (CoG) calculation in Fusion 360.
  * Electrical design: Power budget, motor-propeller thrust matching, battery discharge C-rating calculation.
  * Control architecture: Pixhawk 2.4.8 pinout, sensor placement, vibration damping isolation.
  * Firmware configuration: ArduCopter parameter tuning, failsafe protocols.
* **Chapter 4: Experimental Testing & Telemetry Analysis**
  * Bench test results: Motor thrust bench testing, vibration spectrum analysis via Dataflash logs.
  * Outdoor flight testing: Step response in Stabilize vs Alt-Hold modes.
  * Autonomous waypoint mission: Deviation error analysis in Loiter and GPS mission replay.
  * Fail-safe validation: Radio link loss RTL trajectory plots.
* **Chapter 5: Conclusion & Recommendations**
  * Summary of project outcomes against initial objectives.
  * Budget breakdown and economic feasibility for domestic deployment.
  * Recommendations for subsequent students (e.g., companion computer integration, ROS2 MAVLink bridge).

---

## 5. Defense Day Protocol (Zero-Failure Checklist)

1. **Pre-Flight Battery Check:** Measure battery voltage with a digital LiPo checker in front of the examiners (must be 12.4V – 12.6V).
2. **Propeller Safety:** Verify leading edges are smooth; check that nylon locknuts are tight.
3. **Flight Area Setup:** 10-meter perimeter cleared. Drone powered on with transmitter already active.
4. **Live Sequence:**
   * Arm in Stabilize mode -> Gentle liftoff to 1.5 meters.
   * Switch to Alt-Hold -> Hands off throttle to demonstrate barometer stability.
   * Switch to Loiter -> Hands off all sticks to demonstrate GPS position lock (examiners love this).
   * Perform smooth pitch/roll pirouette.
   * Land gently and disarm.
5. **Presentation Slide Deck:** Exactly 12 slides max. Spend 4 minutes on slides, 3 minutes on flight demo, 5 minutes on Q&A.
