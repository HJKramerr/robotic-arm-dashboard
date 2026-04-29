# Robotic Arm Dashboard

A web-based control dashboard for a 3D-printed 6-DOF robotic arm, built as a learning project in robotics, embedded systems, and full-stack Python development.

## Status

Early development — foundation and architecture phase.

## Goals

This project is as much about learning as it is about the end result. Planned milestones, in rough order:

1. Reliable Pi ↔ Arduino serial communication
2. Manual jog controls with live feedback
3. Teach-and-playback for recording and replaying point sequences
4. Smooth interpolated motion between points
5. Forward kinematics with 3D visualization
6. Inverse kinematics ("move tool to XYZ")
7. Polish: soft limits, emergency stop, collision bounds

## Hardware

- **Compute**: Raspberry Pi (the brain — dashboard, program logic, kinematics)
- **Motor control**: Arduino Uno (real-time servo signals)
- **Arm**: 3D-printed, 6 degrees of freedom
  - Rotating base
  - Joint 1, Joint 2, Joint 3
  - Rotating wrist
  - Closeable gripper
- **Actuators**: Hobby servos (180° range)

## Architecture

Three-layer design with clean separation of concerns:

- **Arduino layer** — minimal firmware that listens for serial commands and drives servos. No higher-level logic.
- **Controller layer (Python)** — the "brain." Manages state, talks to the Arduino, holds taught points, runs programs, handles kinematics. Usable on its own without a UI.
- **Dashboard layer (Python)** — web interface that calls into the controller. A view, not a source of truth.

This separation means the controller can be driven from a script, REPL, or future alternate interfaces without rewriting robot logic.

## Project Structure
robotic-arm-dashboard/
├── arm_controller/     # Robot brain (Python package)
├── dashboard/          # Web interface (Python package)
├── arduino/            # Arduino firmware (.ino files)
├── docs/               # Protocol spec, daily logs
│   └── daily/          # Daily progress entries
├── tests/              # Controller tests
├── scripts/            # Utility scripts (calibration, etc.)
└── requirements.txt

## Setup

_Instructions to come once the project is runnable._

## Progress Log

Daily progress entries live in [`docs/daily/`](docs/daily/).
