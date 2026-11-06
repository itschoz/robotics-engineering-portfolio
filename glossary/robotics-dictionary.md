---
layout: page
title: Robotics Glossary
permalink: /glossary/
---

# Robotics Dictionary — 핵심 용어 90개

출퇴근길 반복 학습용으로 만들었던 90개 용어 사전을 웹에서 검색할 수 있도록 Markdown으로 옮겼다. 정의는 학습용 요약이므로 실제 API나 보드 동작은 해당 버전의 공식 문서를 확인한다.

## A. 로봇과 컴퓨터 기본 개념

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 1 | Robotics | 기계·전자·제어·소프트웨어가 결합된 로봇 기술 분야 |
| 2 | Robot | 센서·제어기·구동기가 결합되어 동작하는 기계 |
| 3 | Actuator | 전기·공압 등 에너지를 운동으로 바꾸는 구동 장치 |
| 4 | Sensor | 위치·속도·힘·거리 등 물리량 측정 |
| 5 | Controller | 목표와 상태를 바탕으로 제어 출력을 계산 |
| 6 | MCU | CPU·메모리·주변장치를 통합한 마이크로컨트롤러 |
| 7 | Embedded System | 특정 장치 안에서 동작하는 하드웨어·소프트웨어 시스템 |
| 8 | Firmware | MCU 주변장치와 장치 동작을 직접 제어하는 소프트웨어 |
| 9 | CPU | 명령 실행과 연산을 담당하는 처리 장치 |
| 10 | Real-time System | 정해진 시간 제약 안에 응답해야 하는 시스템 |

## B. ROS 2와 통신

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 11 | ROS | 로봇 개발을 위한 라이브러리·도구·통신 프레임워크 생태계 |
| 12 | ROS 2 | 분산 로봇 소프트웨어와 DDS 기반 통신을 지원하는 ROS 세대 |
| 13 | Node | ROS 2에서 독립적으로 실행되는 프로그램 단위 |
| 14 | Topic | Publisher와 Subscriber가 비동기로 데이터를 교환하는 채널 |
| 15 | Publisher | Topic에 메시지를 발행하는 주체 |
| 16 | Subscriber | Topic 메시지를 구독하는 주체 |
| 17 | Message | ROS 2 노드 사이에 전달하는 타입이 정의된 데이터 |
| 18 | Service | 요청(Request)과 응답(Response)의 단발성 통신 |
| 19 | Action | Goal·Feedback·Result로 장시간 작업을 추적하는 통신 |
| 20 | QoS | 신뢰성·지속성·이력 등 데이터 전달 정책 |
| 21 | DDS | 분산 환경의 데이터 중심 발행·구독 미들웨어 표준 |
| 22 | Launch File | 여러 노드와 설정을 묶어 실행하는 파일 |
| 23 | Package | 코드·설정·의존성·빌드 메타데이터 단위 |
| 24 | colcon | ROS 2 작업 공간의 패키지를 빌드하는 도구 |
| 25 | rclpy / rclcpp | Python / C++에서 ROS 2 기능을 사용하는 Client Library |

## C. MuJoCo와 로봇 모델

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 26 | Simulation | 물리·수학 모델을 통해 실제 장치 없이 동작을 실험 |
| 27 | Physics Engine | 힘·중력·충돌·관절 운동을 계산하는 엔진 |
| 28 | MuJoCo | 로봇과 물리 시스템의 동역학 시뮬레이터 |
| 29 | MJCF | MuJoCo 모델과 물리 속성을 정의하는 XML 형식 |
| 30 | URDF | ROS 로봇의 Link·Joint·형상을 기술하는 XML 형식 |
| 31 | Xacro | 매크로와 파라미터로 URDF 작성을 간결하게 하는 도구 |
| 32 | Link | 로봇 모델을 이루는 강체 단위 |
| 33 | Joint | Link 사이의 상대 운동을 정의하는 연결 요소 |
| 34 | Revolute Joint | 한 축을 중심으로 회전하는 1자유도 관절 |
| 35 | Fixed Joint | 상대 운동이 없는 고정 연결 |
| 36 | DOF | 독립적으로 움직일 수 있는 자유도의 개수 |
| 37 | TF2 | 좌표 프레임 사이의 변환을 관리하는 ROS 라이브러리 |
| 38 | Coordinate Frame | 위치와 방향을 정의하는 기준 좌표계 |
| 39 | RViz2 | ROS 2 데이터와 로봇 상태를 시각화하는 도구 |
| 40 | Robot State Publisher | URDF와 Joint State를 바탕으로 TF를 발행 |

## D. 로봇 제어

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 41 | ros2_control | Controller와 하드웨어 인터페이스를 연결하는 ROS 2 프레임워크 |
| 42 | Controller Manager | Controller와 하드웨어 인터페이스의 lifecycle 관리 |
| 43 | Hardware Interface | 명령·상태 인터페이스를 추상화해 제어기와 하드웨어를 연결 |
| 44 | Joint State Broadcaster | 관절 상태 인터페이스를 Joint State로 발행 |
| 45 | Joint Trajectory Controller | 시간 기반 관절 궤적 명령을 실행 |
| 46 | PID Control | 비례·적분·미분 항으로 오차를 줄이는 제어 방식 |
| 47 | Feedback | 측정한 출력을 제어 입력에 반영하는 과정 |
| 48 | Position / Velocity / Effort | 위치·속도·힘 또는 토크 관련 명령·상태 인터페이스 |
| 49 | Trajectory | 시간에 따른 위치·속도·가속도 목표의 집합 |

## E. STM32와 펌웨어

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 50 | STM32 | STMicroelectronics의 마이크로컨트롤러 제품군 |
| 51 | STM32 Nucleo-F446RE | 펌웨어 실습에 사용할 수 있는 Nucleo 개발 보드 후보 |
| 52 | STM32CubeIDE | STM32 코드 작성·빌드·디버깅 통합 개발 환경 |
| 53 | HAL | MCU 주변장치를 추상화해 사용하는 라이브러리 계층 |
| 54 | GPIO | 디지털 입력과 출력을 제어하는 핀 |
| 55 | UART | 비동기 직렬 통신 인터페이스 |
| 56 | Serial Communication | 비트를 순서대로 전송하는 직렬 통신 방식 |
| 57 | TX / RX | 송신 / 수신 신호선 |
| 58 | Baud Rate | 초당 신호 심볼 수를 나타내는 전송 속도 단위 |
| 59 | Timer | 시간 측정과 주기적 이벤트 발생을 담당하는 주변장치 |
| 60 | Interrupt | 이벤트 발생 시 현재 실행 흐름을 잠시 중단해 처리하는 메커니즘 |
| 61 | Polling | 주변장치 상태를 반복적으로 확인하는 방식 |
| 62 | PWM | 펄스 폭을 조절해 출력 전력을 제어하는 데 쓰이는 신호 방식 |
| 63 | Duty Cycle | 한 주기 중 High 상태가 차지하는 비율 |
| 64 | ADC | 아날로그 입력을 디지털 값으로 변환 |
| 65 | Encoder | 회전 또는 선형 위치·속도를 측정 |
| 66 | Motor Driver | MCU 신호를 모터 구동에 필요한 전력으로 변환·제어 |
| 67 | C / C++ | 펌웨어와 ROS 2 노드 구현에 사용하는 프로그래밍 언어 |
| 68 | Pointer | 메모리 주소를 저장하고 참조하는 변수 |
| 69 | Struct | 여러 필드를 하나의 사용자 정의 데이터 구조로 묶는 기능 |

## F. RTOS와 임베디드

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 70 | RTOS | 실시간 제약에 맞춰 Task와 자원을 관리하는 운영체제 |
| 71 | Task | RTOS 스케줄러가 관리하는 실행 단위 |
| 72 | Scheduler | CPU 시간을 Task에 배분하는 구성 요소 |
| 73 | Queue | Task 사이에서 데이터를 순서대로 전달하는 자료구조 |
| 74 | Semaphore | Task 간 이벤트 통지 또는 자원 동기화 수단 |
| 75 | Mutex | 공유 자원의 동시 접근을 보호하는 상호 배제 수단 |
| 76 | FreeRTOS | 임베디드 시스템에서 널리 쓰이는 RTOS |
| 77 | micro-ROS | 제한된 임베디드 장치와 ROS 2 시스템을 연동하는 프레임워크 |

## G. 통신·테스트·포트폴리오

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 78 | Protocol | 데이터 형식·순서·오류 처리를 정한 통신 규칙 |
| 79 | Packet | 전송을 위해 구성한 데이터 단위 |
| 80 | Header | 패킷 종류·길이·식별 정보를 담는 부분 |
| 81 | Payload | 패킷에 포함되는 실제 명령 또는 센서 데이터 |
| 82 | Checksum | 전송 데이터 오류를 검출하기 위한 계산값 |
| 83 | Timeout | 정해진 시간 내 응답이 없을 때 처리하는 정책 |
| 84 | Latency | 데이터 전달 또는 처리에 걸리는 지연 |
| 85 | rosbag2 | ROS 2 토픽 데이터를 기록하고 재생하는 도구 |
| 86 | CI/CD | 코드 검사·빌드·배포 과정을 자동화하는 방식 |
| 87 | Git / GitHub | 버전 관리와 원격 저장소 협업 도구 |
| 88 | Debugging | 오류의 원인을 추적하고 수정하는 활동 |
| 89 | Validation | 요구사항을 만족하는지 증거로 확인하는 과정 |
| 90 | Integration | 개별 구성 요소를 결합해 전체 시스템으로 동작시키는 과정 |

## 헷갈리기 쉬운 개념

- **ROS 2 vs MuJoCo:** ROS 2는 로봇 소프트웨어 프레임워크, MuJoCo는 물리 시뮬레이터다.
- **URDF vs MJCF:** URDF는 로봇 구조 표현에, MJCF는 MuJoCo 모델과 물리 속성 표현에 사용한다.
- **Topic vs Service:** Topic은 비동기 발행·구독, Service는 요청·응답이다.
- **Service vs Action:** Action은 진행 피드백과 취소 등 장시간 작업 관리에 적합하다.
- **Sensor vs Actuator:** Sensor는 측정하고 Actuator는 움직임을 만든다.
- **Controller vs Hardware Interface:** Controller는 제어 명령을 계산하고 Hardware Interface는 하드웨어와 연결한다.
- **UART vs ROS 2:** UART는 보드 사이 직렬 링크, ROS 2는 로봇 소프트웨어 통신 생태계다.
- **PWM vs ADC:** PWM은 출력 파형 제어, ADC는 아날로그 입력 측정에 사용한다.

## 복습 우선순위

1. **1단계:** ROS 2, Node, Topic, Service, Action, DDS, QoS, URDF, Joint, MuJoCo, STM32, UART, PWM, RTOS
2. **2단계:** Xacro, TF2, RViz2, ros2_control, Controller Manager, Hardware Interface, GPIO, Timer, Interrupt, ADC, Encoder, FreeRTOS
3. **3단계:** micro-ROS, Protocol, Packet, Checksum, Timeout, Latency, rosbag2, Validation, Integration, Debugging


## H. ROS 2 통신 확장 용어

| No. | 용어 | 의미와 프로젝트에서의 역할 |
|---:|---|---|
| 91 | DDS Participant | DDS 통신 도메인에 참여하는 소프트웨어 주체 |
| 92 | Endpoint | DDS Publisher 또는 Subscriber처럼 데이터 송수신을 수행하는 통신 종단점 |
| 93 | Goal | Action Client가 Action Server에 요청하는 작업 목표 |
| 94 | Feedback | Action 실행 중 Server가 Client에 전달하는 진행 정보 |
| 95 | Result | Action 종료 후 반환되는 최종 결과 데이터 |
| 96 | Request | Service Client가 Server에 전달하는 요청 데이터 |
| 97 | Response | Service Server가 Client에 반환하는 응답 데이터 |
| 98 | Action Server | Goal을 수신하고 작업·Feedback·Result·취소 처리를 수행하는 노드 측 구성 요소 |
| 99 | Action Client | Goal을 보내고 Feedback·Result를 수신하며 취소를 요청하는 구성 요소 |
| 100 | Interface Definition | Message·Service·Action의 데이터 구조와 필드를 정의하는 명세 |

## I. ROS 2 통신 개념 빠른 비교

- **Publisher / Subscriber:** Topic 데이터를 발행하는 쪽과 구독하는 쪽
- **Client / Server:** Service 요청을 보내는 쪽과 응답하는 쪽
- **Action Client / Action Server:** Goal을 요청하는 쪽과 장시간 작업을 수행하는 쪽
- **Goal / Feedback / Result:** Action의 목표·진행 정보·최종 결과
- **Request / Response:** Service의 요청 데이터·응답 데이터
- **Participant / Endpoint:** DDS 도메인 참여자와 실제 데이터 송수신 종단점

관련 상세 내용은 [ROS 2 Communication Reference]({{ '/reference/ros2-communication/' | relative_url }})와 [Package & CLI Catalog]({{ '/reference/ros2-packages-cli/' | relative_url }})를 참고한다.
