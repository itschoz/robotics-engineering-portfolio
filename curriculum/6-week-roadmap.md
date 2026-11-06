---
layout: page
title: 6-Week Roadmap
permalink: /roadmap/
---

# 6주 로보틱스 엔지니어 전환 커리큘럼

**목표:** ROS 2 기반 로봇 소프트웨어와 시뮬레이션, 임베디드 제어를 하나의 시스템으로 연결하고, 그 과정을 재현 가능한 GitHub 포트폴리오로 증명한다.

이 계획은 약 **200시간**의 집중 학습을 기준으로 한다. 주차별 시간은 업무와 실습 난이도에 따라 조정하며, 각 주의 산출물을 확인한 뒤 다음 단계로 넘어간다.

## 최종 프로젝트

**ROS 2 & Embedded Based Robotic Joint Control System**

PC의 ROS 2 애플리케이션에서 관절 명령을 생성하고, 동일한 명령·상태 인터페이스를 통해 MuJoCo 시뮬레이터와 STM32 기반 하드웨어를 각각 제어한다. 시뮬레이션으로 검증한 뒤 실제 구동부로 확장한다.

## 전체 학습 시간 배분

| 영역 | 목표 시간 | 핵심 결과 |
|---|---:|---|
| ROS 2 기초 | 5h | Node, Topic, Service, Action 이해 |
| DDS / QoS | 10h | Discovery와 QoS 호환성 실습 |
| MuJoCo / URDF / TF2 | 35h | 가상 관절 로봇 모델 및 시뮬레이션 |
| ros2_control / 시스템 통합 | 60h | Controller와 Hardware Interface 구성 |
| STM32 펌웨어 | 70h | UART, PWM, Encoder, Timer, RTOS 기초 |
| 포트폴리오 | 20h | 코드, 실험, 문서, 영상 정리 |
| **합계** | **200h** | 통합 프로젝트 |

## 주차별 진행 계획

### Week 1 — ROS 2 Middleware Foundations

**학습 주제**
- ROS 1과 ROS 2 아키텍처 차이
- Node, Topic, Publisher, Subscriber
- Service와 Action의 사용 상황
- DDS, Discovery, RMW
- QoS: Reliability, Durability, History, Deadline
- ROS 2 CLI, package, colcon, launch 기초

**실습**
- Talker / Listener 실행 및 노드·토픽 확인
- QoS 프로파일 확인
- Best Effort / Reliable 호환성 비교
- Topic, Service, Action 예제 실행

**완료 기준**
- ROS 2 통신 구조를 계층별로 설명할 수 있다.
- QoS 설정이 통신에 미치는 영향을 실험 결과로 설명할 수 있다.

### Week 2 — Robot Description & Simulation

**학습 주제**
- Link, Joint, DOF, Revolute Joint
- URDF와 Xacro
- TF2와 Coordinate Frame
- MuJoCo와 MJCF
- 시뮬레이션 물리 파라미터와 관절 제한

**실습**
- 2관절 로봇 모델 작성
- RViz2에서 로봇 구조와 TF 확인
- MuJoCo에서 관절 움직임 검증
- ROS 2 명령과 시뮬레이션 상태 연결의 기초 구현

**완료 기준**
- 로봇 모델의 Link / Joint 구조를 설명할 수 있다.
- 가상 관절에 목표 위치를 전달하고 결과를 확인할 수 있다.

### Week 3 — ros2_control & Hardware Abstraction

**학습 주제**
- ros2_control 아키텍처
- Controller Manager
- Hardware Interface
- Joint State Broadcaster
- Joint Trajectory Controller
- Position / Velocity / Effort interface
- Command와 State의 분리

**실습**
- 시뮬레이터용 Hardware Interface 구조 검토
- Controller 로드 및 활성화
- 목표 관절 궤적 전달
- 목표값과 상태 피드백 비교

**완료 기준**
- Controller와 실제 장치 사이의 Hardware Interface 역할을 설명할 수 있다.
- 시뮬레이터를 제어하는 최소 경로를 구성할 수 있다.

### Week 4 — STM32 Firmware & Serial Protocol

**학습 주제**
- STM32 Nucleo-F446RE와 STM32CubeIDE
- HAL, GPIO, UART
- Timer, Interrupt, PWM
- Encoder, ADC, Motor Driver
- 패킷 구조: Header, Payload, Checksum
- Timeout, Error Handling

**실습**
- UART 송수신
- PC와 MCU 사이 명령 패킷 교환
- PWM 출력과 타이머 주기 구성
- 엔코더 입력 또는 테스트 신호 처리

**완료 기준**
- MCU가 명령을 수신하고 검증한 뒤 제어 출력으로 변환할 수 있다.
- 잘못된 패킷과 통신 단절을 감지할 수 있다.

### Week 5 — Embedded Control & Integration

**학습 주제**
- C/C++ 펌웨어 구조화
- FreeRTOS Task, Queue, Semaphore, Mutex
- 통신 Task와 제어 Task 분리
- ROS 2 ↔ UART ↔ STM32 데이터 흐름
- Latency, 주기, Timeout, Fault Handling

**실습**
- PC 측 ROS 2 Serial Bridge 구현
- MCU 명령 처리 및 상태 응답
- 시뮬레이터 Backend와 MCU Backend의 인터페이스 비교
- 반복 명령 / 피드백 테스트

**완료 기준**
- ROS 2 노드에서 보낸 명령이 MCU까지 전달되고 상태 응답이 돌아오는 경로를 검증한다.
- 측정값과 실패 조건을 기록한다.

### Week 6 — Validation & Portfolio

**학습 주제**
- 시스템 테스트 시나리오
- rosbag2 및 로그 수집
- Latency와 주기 측정
- Debugging, Validation, Integration
- README, architecture diagram, demo recording
- GitHub Actions 기반 문서 배포

**실습**
- End-to-end 제어 시나리오 실행
- 시뮬레이션 및 하드웨어 결과 비교
- 오류 조건과 복구 동작 문서화
- 프로젝트 영상 및 기술 문서 정리

**완료 기준**
- 다른 개발자가 저장소의 설명을 보고 프로젝트 목적·구조·실행 방법을 이해할 수 있다.
- 구현 완료, 실험 완료, 향후 계획을 명확하게 구분한다.

## 주의: 학습 계획과 구현 사실은 구분한다

이 로드맵은 목표 계획이다. 아직 설치하거나 실행하지 않은 항목은 완료로 표시하지 않는다. 각 실습 페이지에 환경, 명령어, 결과, 오류 및 수정 사항을 기록한다.

## 학습 방식

1. 개념을 먼저 이해한다.
2. 최소 재현 실습을 실행한다.
3. 터미널 출력과 로그를 저장한다.
4. 동작 원인과 실패 원인을 설명한다.
5. Markdown으로 정리하고 GitHub에 커밋한다.

