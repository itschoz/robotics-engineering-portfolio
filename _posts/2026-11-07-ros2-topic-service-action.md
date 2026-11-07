---
layout: post
title: "ROS 2 학습 기록 02 — Topic, Service, Action으로 구성하는 노드 간 통신"
date: 2026-11-07 09:45:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, Humble, Topic, Service, Action, Publisher, Subscriber]
excerpt: "ROS 2의 세 가지 통신 패턴을 비교하고, Talker/Listener와 Service 및 Fibonacci Action 예제로 각각의 사용 목적을 확인한다."
---

## 1. 학습 목표

ROS 2 노드들은 서로 데이터를 주고받으며 하나의 로봇 시스템을 구성한다. 이번 학습에서는 ROS 2의 대표적인 통신 패턴인 Topic, Service, Action의 차이를 이해하고 실제 명령어로 동작을 확인했다.

## 2. 세 가지 통신 패턴

| 구분 | Topic | Service | Action |
|---|---|---|---|
| 통신 형태 | 발행/구독(Publish/Subscribe) | 요청/응답(Request/Response) | 목표/피드백/결과(Goal/Feedback/Result) |
| 기본 특성 | 비동기 데이터 스트림 | 짧은 작업의 요청과 응답 | 오래 걸리는 작업의 진행 상태 관리 |
| 주요 용도 | 센서 데이터, 상태, 연속 명령 | 모드 변경, 간단한 설정 요청 | 이동, 관절 동작 등 실행 시간이 있는 작업 |
| 실행 중 피드백 | 별도 토픽 등을 사용 | 기본 패턴에는 없음 | Action Feedback 제공 |
| 취소 | 별도 설계 | 기본 패턴에는 없음 | Goal 취소 지원 |

### 2.1 Topic

Publisher가 메시지를 발행하고 Subscriber가 이를 구독한다. 송신 측과 수신 측은 매 메시지마다 요청과 응답을 주고받지 않는다.

로봇 예시:
- `/joint_states`: 관절 위치·속도 등의 상태
- `/motor_status`: 모터 상태
- 카메라 영상이나 LiDAR 데이터 스트림

### 2.2 Service

Client가 Server에 요청을 보내고 Server가 응답을 반환한다. 짧고 한 번의 요청으로 완료되는 작업에 적합하다.

로봇 예시:
- 모터 Enable/Disable 요청
- 제어 모드 변경
- 파라미터 또는 간단한 설정 변경

### 2.3 Action

Action Client가 Goal을 보내면 Action Server가 작업을 수행한다. 수행 중 Feedback을 전달하고 완료 시 Result를 반환한다. Client는 실행 중 Goal 취소를 요청할 수도 있다.

로봇 예시:
- 특정 관절 각도로 이동
- 로봇팔의 지정된 자세까지 이동
- 일정 시간이 걸리는 주행 작업

Action 취소는 소프트웨어 작업의 취소 요청이다. 비상정지(E-stop)를 대신하지 않으며, 안전 정지는 별도의 하드웨어 및 안전 설계가 필요하다.

## 3. Topic 실습 — Talker와 Listener

터미널을 나누어 Publisher와 Subscriber를 실행한다.

Publisher:

```bash
ros2 run demo_nodes_cpp talker
```

Subscriber:

```bash
ros2 run demo_nodes_py listener
```

토픽과 노드 상태 확인:

```bash
ros2 node list
ros2 topic list
ros2 topic info /chatter
ros2 topic info /chatter --verbose
ros2 topic echo /chatter
ros2 topic hz /chatter
```

`/chatter`는 Talker가 발행하고 Listener가 구독하는 예제 토픽이다. `topic info`는 연결된 Publisher/Subscriber 정보를 확인하고, `echo`는 메시지 내용을, `hz`는 관측된 발행 빈도를 확인하는 데 사용한다.

## 4. Service 실습 — AddTwoInts

Service Server 실행:

```bash
ros2 run demo_nodes_cpp add_two_ints_server
```

별도 터미널에서 Service 확인 및 요청:

```bash
ros2 service list
ros2 service type /add_two_ints
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 10, b: 20}"
```

이 예제는 두 정수를 요청으로 전달하고 합계를 응답으로 받는 Service다. 요청과 응답이 한 번의 호출로 완료되는 형태를 확인할 수 있다.

## 5. Action 실습 — Fibonacci

처음에는 `demo_nodes_cpp`에서 Fibonacci Action Server 실행 파일을 찾으려 했지만, 해당 설치 환경의 실행 파일 목록에 존재하지 않았다. 패키지와 실행 파일을 확인한 뒤 Action 튜토리얼 패키지를 설치해 실습을 이어갔다.

필요한 패키지 설치:

```bash
sudo apt update
sudo apt install ros-humble-action-tutorials-cpp
```

Action Server 실행:

```bash
ros2 run action_tutorials_cpp fibonacci_action_server
```

별도 터미널에서 Action 확인:

```bash
ros2 pkg executables action_tutorials_cpp
ros2 action list
ros2 action info /fibonacci
```

Goal 전송 및 Feedback 확인:

```bash
ros2 action send_goal /fibonacci action_tutorials_interfaces/action/Fibonacci "{order: 10}" --feedback
```

이 실습을 통해 Action은 Goal을 전달하는 것에 그치지 않고 실행 중 Feedback과 최종 Result를 제공한다는 점을 확인했다.

## 6. 로봇 관절 제어 프로젝트에 적용

향후 관절 제어 프로젝트에서는 통신 특성에 맞춰 인터페이스를 분리한다.

- Topic: 관절 상태와 주기적으로 갱신되는 측정값 전달
- Service: 모터 Enable, 제어 모드 변경처럼 짧은 요청 처리
- Action: 목표 관절 위치까지 이동하는 등 시간이 걸리는 동작 수행

실제 인터페이스 이름과 메시지 타입은 프로젝트 구현 단계에서 정의한다. 위 항목은 설계 방향이며, 현재 구현이 완료된 기능을 의미하지 않는다.

## 7. 학습 결과

- Topic의 Publisher/Subscriber 구조를 실행하고 토픽 정보를 확인했다.
- Service Server에 요청을 보내고 응답을 확인하는 실습을 진행했다.
- Fibonacci Action Server에 Goal을 전송하고 Feedback을 확인했다.
- Action 튜토리얼 실행 파일이 어느 패키지에 포함되는지 확인하고 올바른 패키지로 실습을 완료했다.

**검증 상태:** Topic, Service, Action의 기본 실습 완료(사용자 확인).

## 8. 핵심 정리

통신 방식을 고를 때는 데이터의 양만이 아니라 작업의 시간적 특성과 상호작용 방식이 중요하다. 반복적으로 흘러가는 데이터에는 Topic, 짧은 요청과 응답에는 Service, 진행 상태와 취소가 필요한 장시간 작업에는 Action을 우선 검토한다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
