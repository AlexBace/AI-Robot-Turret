# AI-Robot-Turret

**Project Overview**
This is a beginner‑friendly, open‑source two‑axis turret driven by two stationary NEMA 23 closed‑loop steppers through a differential bevel‑gear mechanism. There is no dedicated “pan motor” or “tilt motor.” Instead, the central bevel gear (end effector) is controlled in both axes by the combined motion of both side motors.
A 3:1 reduction exists from each motor to the driven shaft (i.e., motors spin 3 rev for 1 rev at the output).

This repo provides wiring, setup, and a Python control script that:
Moves both motors to commanded motor‑shaft angles (for testing and calibration)
Supports independent motor moves or a symmetric move helper
Includes tunable acceleration (steeper ramps) and speed with a shaped trapezoid profile
Is structured to add an inverse‑kinematics layer later (mapping desired end‑effector yaw/pitch to motor angles)

**Hardware**
Raspberry Pi (40‑pin, e.g., Pi 4)
2× NEMA 23 closed‑loop steppers with drivers (step/dir interface)
24 V DC power supply (e.g., S‑350‑24, sized for both motors)
Wires, common ground, mounting, etc.

**Power (high level)**
PSU V+ → each driver V+ / DC+
PSU V‑ → each driver V‑ / GND
Pi GND must be common with driver logic ground

Never connect 24 V to the Pi. Power the Pi via its normal 5 V USB‑C adapter.

Control Signal Wiring (common‑anode with opto‑isolated drivers)
Tie each driver’s PUL+ / DIR+ / ENA+ to Pi 3.3 V.
Connect the “minus” inputs to GPIOs:

**Motor 1**
DIR‑ → GPIO20 (pin 38)
PUL‑ → GPIO21 (pin 40)
ENA‑ → GPIO22 (pin 15)
(PUL+/DIR+/ENA+ → Pi 3.3 V; GND shared)

**Motor 2**
DIR‑ → GPIO19 (pin 35)
PUL‑ → GPIO26 (pin 37)
ENA‑ → GPIO23 (pin 16)
(PUL+/DIR+/ENA+ → Pi 3.3 V; GND shared)

Many drivers accept 3.3 V logic on their opto inputs. If yours requires 5 V, use proper level‑shifting (don’t feed 5 V into Pi GPIO).

**Software Setup:**

sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-rpi.gpio

**Clone your repo and run from the project root:**

sudo python3 code/dual_turret_control.py --help

**How the Motion Model Works (for now):**
- Angles you command are motor‑shaft angles (side gears), not the end‑effector’s yaw/pitch yet.
- Because this is a differential: the end effector’s orientation is determined by combinations of the two motor angles. We’ll add the mapping (inverse kinematics) later.
- Right now: use this to verify wiring, tune ramps, and characterize behavior under load.

**Running the Code (CLI)** (look for more info on the commands file)
From the repo root:

cd ~/Documents
sudo python3 turret_repl.py --gears 15
sudo python3 turret_repl.py --m1 30 --m2 -10 --speed 240 --accel 1500 --shape 0.5

**Run (REPL – recommended for testing)** (look for more info on the commands file)
cd ~/Documents
sudo python3 turret_repl.py --repl

Symmetric test: both motors to +15° (Motor 2 auto-inverted so side gears “spin the same way”)
sudo python3 code/dual_turret_control.py --gears 15

Independent targets: M1 = +30°, M2 = -10°
sudo python3 code/dual_turret_control.py --m1 30 --m2 -10

Make it faster and snappier (steeper ramp)
sudo python3 code/dual_turret_control.py --m1 30 --m2 -10 --speed 240 --accel 1500 --shape 0.5

**CLI Options**

--gears <deg> — Move both motors to the same absolute motor angle (helpful symmetric test)
--m1 <deg> / --m2 <deg> — Independent absolute motor angles (deg)
--speed <deg/s> — Max motor‑shaft speed
--accel <deg/s^2> — Motor‑shaft acceleration
--shape <float> — Accel curve: <1.0 = snappier, 1.0 = linear, >1.0 = softer
--spr <int> — Steps per rev (e.g., 200)
--micro <int> — Microstepping (e.g., 16)
--invert2 / --no-invert2 — Flip Motor 2 logical direction (default inverted for opposing mounting)
--disable-on-exit — Drop ENA on exit to de‑energize motors (default holds position)

**NOTE** The script keeps track of absolute motor angles during a session (starting from 0° at launch). For persistent absolute positioning across boots, add homing or read encoders externally.

**Folder Structure**
├── README.md
├── LICENSE
├── code/
│   └── dual_turret_control.py
├── docs/
└── hardware/

