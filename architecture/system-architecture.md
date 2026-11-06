---
layout: page
title: System Architecture
permalink: /architecture/
---

# 통합 로봇 시스템 아키텍처

## 프로젝트 목표

PC에서 실행하는 ROS 2 애플리케이션이 로봇 관절 명령을 생성하고, 동일한 상위 제어 흐름을 사용해 **MuJoCo 시뮬레이션** 또는 **STM32 기반 실제 하드웨어**를 제어하도록 설계한다.

![로봇 시스템 아키텍처]({{ '/assets/diagrams/robot-system-architecture.svg' | relative_url }})

## 계층별 역할

| 계층 | 구성 요소 | 책임 |
|---|---|---|
| Application | ROS 2 Nodes, Topic, Action | 사용자 명령, 상태 표시, 작업 실행 |
| Middleware | RCL, RMW, DDS | 노드 통신, Discovery, QoS |
| Control | ros2_control, Controller Manager | 목표값 처리, 제어기와 하드웨어 추상화 연결 |
| Backend A | MuJoCo Hardware Interface | 가상 관절에 명령 전달, 상태 반환 |
| Backend B | Serial Bridge + STM32 | UART 명령 전달, 펌웨어 제어, 상태 응답 |
| Actuation | Motor Driver, Motor, Encoder | 실제 구동과 센서 피드백 |

## 설계 핵심: 공통 인터페이스, 교체 가능한 Backend

시뮬레이터와 실제 하드웨어는 물리적 특성과 저수준 통신 방식이 다르다. 하지만 상위 ROS 2 애플리케이션이 사용하는 관절 명령과 상태 인터페이스는 가능한 한 일관되게 유지한다.

- **Simulation Backend:** MuJoCo가 물리 모델을 계산하고 관절 상태를 반환한다.
- **Hardware Backend:** STM32 펌웨어가 명령을 처리하고 센서 또는 엔코더 상태를 반환한다.
- **Shared contract:** 관절 이름, 단위, 명령 범위, 상태 의미와 오류 처리 규칙을 문서화한다.

이렇게 하면 상위 애플리케이션을 크게 바꾸지 않고 Backend를 바꾸어 테스트할 수 있다. 다만 실제 ros2_control 플러그인 구조와 Serial Bridge 경계는 구현 과정에서 검증하고 확정한다.

## 명령과 피드백 흐름

1. 사용자가 ROS 2 애플리케이션에서 목표 관절값을 지정한다.
2. Controller가 목표값을 처리한다.
3. 선택된 Hardware Interface가 시뮬레이터 또는 실제 장치에 명령을 전달한다.
4. 시뮬레이터 또는 MCU가 관절 상태를 반환한다.
5. 상태 피드백을 통해 목표값과 실제값의 차이를 확인한다.

## 안전과 검증 원칙

- QoS 설정만으로 비상정지나 기능 안전이 보장되지는 않는다.
- 실제 모터는 적절한 모터 드라이버, 전원, 전류 제한 및 기구적 안전 대책을 갖춘 뒤 연결한다.
- 통신 Timeout, 패킷 오류, 명령 범위 초과, 센서 이상 시 안전한 상태로 전환하는 로직을 별도로 설계한다.
- 시뮬레이션 성공을 실제 하드웨어 안전성의 증거로 간주하지 않는다.

## 현재 상태

**완료 확인**
- Windows 11 + WSL2 Ubuntu 22.04 환경
- ROS 2 Humble Desktop 설치
- Talker 실행 및 기본 통신 확인
- GitHub SSH 인증과 저장소 연동

**진행 예정**
- MuJoCo 설치 및 모델 실행
- URDF / TF2 로봇 모델
- ros2_control Backend
- STM32 보드 및 펌웨어
- Serial Bridge 및 End-to-end 테스트

상태는 실제 실습 결과가 확인되는 대로 갱신한다.
