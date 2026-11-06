---
layout: page
title: STM32 Firmware
permalink: /projects/stm32/
---

# STM32 Firmware

<span class="status planned">Planned</span>

## Objective

STM32 기반 MCU에서 PC로부터 UART 명령을 수신하고, 패킷 검증 후 제어 출력으로 변환하며, 센서·엔코더 상태를 PC로 반환한다.

## Target platform

- Candidate board: STM32 Nucleo-F446RE
- Toolchain: STM32CubeIDE / HAL
- Interfaces: UART, GPIO, Timer, PWM, Encoder
- Later topics: FreeRTOS, micro-ROS

## Planned milestones

1. STM32CubeIDE 프로젝트 생성 및 UART loopback
2. Header / Payload / Checksum을 포함한 명령 패킷 정의
3. Timeout 및 오류 처리
4. PWM과 Encoder 테스트
5. ROS 2 Serial Bridge 연동

실제 보드와 회로가 준비되기 전까지는 하드웨어 동작을 완료로 표시하지 않는다.
