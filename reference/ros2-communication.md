---
layout: page
title: ROS 2 Communication Reference
permalink: /reference/ros2-communication/
---

# ROS 2 Communication Reference

ROS 2 통신 방식을 개념, 인터페이스, CLI, 설계 사례 기준으로 누적 관리하는 기술 레퍼런스다. 강의에서 새롭게 확인한 내용은 이 문서에 반영한다.

## 1. 통신 방식 한눈에 비교

| 구분 | Topic | Service | Action |
|---|---|---|---|
| 통신 모델 | Publish / Subscribe | Request / Response | Goal / Feedback / Result |
| 기본 성격 | 비동기 스트림 | 짧은 요청과 응답 | 장시간 작업 관리 |
| 호출 측 대기 | 메시지 수신을 기다리지 않음 | 응답을 기다림 | Goal 수락 후 진행 상태를 추적 |
| 진행 상태 | 별도 메시지 설계 | 기본 제공하지 않음 | Feedback 제공 |
| 취소 | 해당 없음 | 기본 취소 모델 없음 | Goal 취소 지원 |
| 대표 용도 | 센서, 상태, 이벤트 | 설정 변경, 단발성 명령 | 이동, 궤적 실행, 복합 작업 |

선택 기준:
- 반복적으로 흘러가는 데이터인가? → Topic
- 짧게 요청하고 결과를 받으면 끝나는가? → Service
- 오래 걸리고 중간 진행률이나 취소가 필요한가? → Action

## 2. Topic

### 개념
Publisher가 특정 Topic 이름으로 타입이 정해진 Message를 발행하고, Subscriber가 이를 구독한다. 하나의 Topic에 여러 Publisher 또는 Subscriber가 존재할 수 있다. 실제 연결과 데이터 전달은 ROS 2 middleware 및 QoS 설정의 영향을 받는다.

### 예시
- `/chatter`: 데모 문자열 메시지
- `/joint_states`: 관절 상태
- `/motor_status`: 모터 상태
- `/camera/image_raw`: 카메라 이미지 스트림

### CLI
```bash
ros2 topic list
ros2 topic type /chatter
ros2 topic info /chatter
ros2 topic info /chatter --verbose
ros2 topic echo /chatter
ros2 topic hz /chatter
ros2 topic bw /chatter
```

### 실행 예제
```bash
# Terminal 1
ros2 run demo_nodes_cpp talker

# Terminal 2
ros2 run demo_nodes_py listener

# Terminal 3
ros2 topic info /chatter --verbose
ros2 topic echo /chatter
```

관찰 포인트:
1. Talker가 Publisher 역할을 수행한다.
2. Listener가 Subscriber 역할을 수행한다.
3. `ros2 topic echo`도 별도의 Subscriber로 동작한다.
4. `ros2 topic hz`는 수신 메시지 간격을 바탕으로 주기를 추정한다.

## 3. Service

### 개념
Client가 Request를 보내고 Server가 Response를 반환하는 요청-응답 방식이다. 짧은 작업이나 즉시 결과를 돌려줄 수 있는 작업에 적합하다. 오래 걸리는 작업을 Service 안에서 장시간 수행하도록 설계하는 것은 피하는 편이 좋다.

### 예시
- 모터 Enable 상태 변경 요청
- 파라미터 또는 설정 변경
- 짧은 계산 요청
- 장치 상태 초기화 요청

### CLI
```bash
ros2 service list
ros2 service type /add_two_ints
ros2 service info /add_two_ints
ros2 interface show example_interfaces/srv/AddTwoInts
```

### 실행 예제
```bash
# Terminal 1: Server
ros2 run demo_nodes_cpp add_two_ints_server

# Terminal 2: Client
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 10, b: 20}"
```

예상 응답의 합계는 30이다. 이 명령은 예제 패키지와 인터페이스가 설치되어 있을 때 실행할 수 있다.

## 4. Action

### 개념
Action은 장시간 실행되는 작업을 위해 Goal, Feedback, Result를 제공한다. Client는 Goal을 보내고, Server는 Goal을 수락하거나 거절할 수 있다. 실행 중 Feedback을 전달하고 최종 Result를 반환하며, Client는 작업 취소를 요청할 수 있다.

Action의 취소는 소프트웨어 수준의 작업 취소 요청이다. 모터 전원 차단이나 안전 정지 회로를 대체하지 않는다.

### 예시
- 로봇 팔을 목표 관절 각도로 이동
- 내비게이션 목표 지점까지 이동
- 정해진 궤적 실행
- 일정 시간 걸리는 조립 동작

### CLI
```bash
ros2 action list
ros2 action info /fibonacci
ros2 interface show action_tutorials_interfaces/action/Fibonacci
```

### 실행 예제: Fibonacci
```bash
# Terminal 1: Server
ros2 run action_tutorials_cpp fibonacci_action_server

# Terminal 2: Goal 전송 및 Feedback 확인
ros2 action send_goal /fibonacci action_tutorials_interfaces/action/Fibonacci "{order: 10}" --feedback
```

### Action 데이터 구조
- Goal: Client가 수행을 요청하는 목표 데이터
- Feedback: 작업 진행 중 Server가 보내는 중간 정보
- Result: 작업 종료 후 반환되는 최종 결과

## 5. 메인 프로젝트에 적용할 통신 설계 초안

| 인터페이스 | 방식 | 방향 | 목적 |
|---|---|---|---|
| `/joint_states` | Topic | Hardware/Simulation → ROS 2 | 관절 위치·속도 등 상태 전달 |
| `/motor_status` | Topic | MCU/Driver → ROS 2 | 상태·오류·진단 정보 |
| `/motor_enable` | Service 후보 | ROS 2 → MCU | 구동 허가 상태 변경 요청 |
| `/move_joint` | Action 후보 | ROS 2 → Controller | 관절 이동 Goal 및 진행 상태 |

이는 초기 설계안이며, 실제 구현 시에는 ros2_control의 표준 인터페이스, 명령 주기, 실패 처리, 안전 요구사항을 검토해 확정한다.

**안전 원칙:** 비상 정지(E-stop)는 일반 Topic, Service, Action 취소만으로 구현하지 않는다. 하드웨어 안전 회로와 구동 전원 차단 등 독립적인 안전 경로를 별도로 설계해야 한다.

## 6. QoS와 통신 연결 문제 점검

Endpoint가 발견되어도 QoS 호환성이나 네트워크 조건에 따라 메시지가 전달되지 않을 수 있다.

점검 순서:
1. Topic 이름과 Message type 확인
2. Publisher와 Subscriber가 실제 실행 중인지 확인
3. `ros2 topic info /topic_name --verbose`로 Endpoint와 QoS 확인
4. Reliability 등 QoS 호환성 확인
5. ROS_DOMAIN_ID, 네트워크 인터페이스, RMW 설정 확인

## 7. 학습 및 검증 이력

| 항목 | 기록 | 상태 |
|---|---|---|
| ROS 2 Humble 환경 | WSL2 Ubuntu 22.04, `ROS_DISTRO=humble` | 확인 |
| Talker 실행 | `demo_nodes_cpp talker` 실행 및 `/chatter` 확인 | 이전 실습에서 확인 |
| C++ 데모 패키지 | `demo_nodes_cpp` 패키지 경로 및 실행 파일 목록 확인 | 확인 |
| Fibonacci Action | `action_tutorials_cpp` 예제 실행 성공을 사용자 확인 | 완료 |
| Listener 수신 | 출력 로그를 별도로 보관 | 증거 추가 예정 |
| AddTwoInts Service | 요청·응답 결과를 별도로 보관 | 증거 추가 예정 |

실습 완료 표시는 실제 실행 결과나 사용자의 확인이 있을 때만 변경한다.
