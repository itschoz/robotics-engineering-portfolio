---
layout: page
title: Robotic Joint Control System
permalink: /projects/joint-control/
---

# Robotic Joint Control System

<span class="status progress">In Progress · Architecture & Learning</span>

## Project objective

ROS 2 기반 PC 애플리케이션에서 관절 명령을 생성하고, 공통 명령·상태 인터페이스를 통해 MuJoCo 시뮬레이션과 STM32 기반 하드웨어를 각각 연결하는 시스템을 구현한다.

## Target architecture

![System architecture]({{ '/assets/diagrams/robot-system-architecture.svg' | relative_url }})

## Work packages

| Work package | Scope | Current status |
|---|---|---|
| ROS 2 middleware | Node, Topic, Service, Action, DDS, QoS | Learning / verified foundations |
| Robot model | URDF, Xacro, TF2, MJCF | Planned |
| Simulation backend | MuJoCo model and joint control | Planned |
| Control abstraction | ros2_control and Hardware Interface | Planned |
| Embedded backend | STM32 UART, PWM, Encoder | Planned |
| End-to-end integration | ROS 2 ↔ MCU ↔ actuator feedback | Planned |

## Engineering constraints

- Simulation and physical hardware must use clearly defined command and state contracts.
- Communication timeout and invalid commands must be handled explicitly.
- E-stop and power isolation require an independent hardware safety path.
- A planned feature is not labeled complete until it has been executed and validated.

## Evidence log

| Date | Evidence | Status |
|---|---|---|
| 2026-11-06 | ROS 2 Humble environment, DDS / QoS learning and Fibonacci Action demo | Verified by user confirmation |
| TBD | URDF / MuJoCo joint model | Planned |
| TBD | STM32 serial command and feedback | Planned |
