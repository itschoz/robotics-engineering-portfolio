---
layout: post
title: "ROS 2 학습 기록 01 — ROS 1과 달라진 통신 구조, DDS와 QoS"
date: 2026-11-06 22:00:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, ROS1, DDS, RMW, QoS, Humble, WSL2]
excerpt: "ROS 1의 Master 기반 구조와 ROS 2의 DDS 기반 통신 구조를 비교하고, RCL·RMW·DDS 계층과 Discovery 및 QoS의 핵심 개념을 정리한다."
---

## 1. 학습 목표

과거 Gazebo와 ROS 1으로 가상 바퀴형 로봇을 구성하고, Publisher/Subscriber를 이용해 명령을 전달했던 경험을 바탕으로 ROS 2의 통신 구조를 다시 정리했다.

이번 학습의 목표는 ROS 1과 ROS 2의 차이를 단순한 API 변경이 아니라 **통신 아키텍처의 변화**로 이해하는 것이다.

## 2. ROS 1의 Master와 ROS 2의 DDS

### ROS 1

ROS 1의 ROS Master는 노드 등록과 검색, 연결 정보를 중개한다. 다만 Master가 일반적인 토픽 메시지를 계속 중계하는 것은 아니다. Publisher와 Subscriber는 연결 정보를 얻은 뒤 보통 TCPROS 또는 UDPROS로 직접 통신한다.

### ROS 2

ROS 2의 기본 통신 구조는 DDS(Data Distribution Service) 기반이다. DDS는 분산 환경에서 참여자와 통신 Endpoint를 발견하고, 데이터 전달을 담당한다.

기본적인 ROS 2 DDS 통신에서는 ROS 1과 같은 단일 ROS Master가 필수는 아니다. 다만 DDS Discovery Server와 같은 중앙집중형 Discovery 구성도 선택할 수 있으므로, 모든 ROS 2 구성이 반드시 중앙 요소가 없다고 이해해서는 안 된다.

| 구분 | ROS 1 | ROS 2 기본 구조 |
|---|---|---|
| 노드 발견 | ROS Master | DDS Discovery |
| 필수 단일 Master | 필요 | 불필요 |
| 토픽 데이터 전달 | TCPROS/UDPROS 등 | DDS 기반 |
| 통신 품질 정책 | 상대적으로 제한적 | QoS 정책 지원 |

## 3. ROS 2 소프트웨어 계층: Application, RCL, RMW, DDS

- **Application:** 카메라 처리, 물체 인식, 경로 계획, 관절 제어 등 로봇 기능을 구현하는 프로그램과 노드
- **RCL:** ROS Client Library 계층
- **RMW:** ROS Middleware Interface. 상위 ROS 2 코드와 실제 미들웨어 구현 사이의 추상화 인터페이스
- **DDS 구현체:** Discovery와 데이터 통신을 수행하는 미들웨어. 예를 들어 Fast DDS, Cyclone DDS 등이 있다.

```text
ROS 2 Application (Nodes)
          |
         RCL
          |
         RMW
          |
   DDS Implementation
          |
   Network / Transport
```

## 4. DDS Discovery: 서로를 어떻게 찾을까?

예를 들어 다음 노드가 실행된다고 가정한다.

- Camera Node: `/camera/image` 토픽 발행
- Object Detection Node: `/camera/image` 토픽 구독
- Motion Planning Node: `/object/position` 토픽 구독

DDS Discovery는 네트워크에 참여한 DDS Participant와 각 Participant의 Publisher/Subscriber Endpoint 정보를 교환해 서로의 존재와 통신 가능성을 파악하도록 돕는다.

여기서 두 과정을 구분해야 한다.

1. **Discovery:** 어떤 Participant와 Endpoint가 존재하는지 발견하고 매칭 가능성을 확인한다.
2. **Data Communication:** 매칭된 Publisher와 Subscriber 사이에서 실제 데이터를 전달한다.

따라서 노드나 Endpoint가 발견되었다는 사실만으로 모든 데이터 통신이 성공한다고 단정할 수 없다. 토픽 이름, 데이터 타입, QoS 호환성 등도 중요하다.

## 5. QoS (Quality of Service)

QoS는 데이터 통신에 적용할 품질 정책이다. 카메라 영상처럼 최신성이 중요한 데이터와 제어·상태 데이터처럼 신뢰성이 중요한 데이터는 서로 다른 통신 요구사항을 가질 수 있다.

### 5.1 Reliability

- **Reliable:** 미들웨어가 전달을 위해 재전송 등을 수행한다. 전달 지연이나 자원 사용이 늘어날 수 있다.
- **Best Effort:** 전달 실패 시 재전송을 보장하지 않는다. 지연을 줄이고 최신 데이터를 우선하는 센서 스트림 등에 활용할 수 있다.

Best Effort가 언제나 더 빠르거나 Reliable이 모든 상황에서 안전하다는 뜻은 아니다. 데이터의 성격과 시스템 요구사항을 기준으로 선택해야 한다.

### 5.2 Durability

- **Volatile:** 과거 샘플을 새 Subscriber에게 제공하도록 보관하지 않는다.
- **Transient Local:** Publisher가 유지하는 샘플을 늦게 참여한 Subscriber에게 제공할 수 있다.

### 5.3 History

- **Keep Last(N):** 최근 N개 샘플을 유지한다.
- **Keep All:** 미들웨어의 자원 한도와 설정이 허용하는 범위에서 샘플을 유지한다.

### 5.4 Deadline

데이터가 발행되거나 수신될 것으로 기대하는 최대 간격을 정의한다. Deadline은 발행 주기를 강제로 보장하는 기능이 아니라, 설정한 기대 간격이 지켜지지 않는 상황을 감지하기 위한 정책이다.

## 6. QoS 호환성: Discovery가 되어도 데이터가 안 올 수 있다

| Publisher | Subscriber | 호환 |
|---|---|---|
| Reliable | Reliable | 가능 |
| Reliable | Best Effort | 가능 |
| Best Effort | Best Effort | 가능 |
| Best Effort | Reliable | 불가능 |

센서 토픽이 보이는데도 `ros2 topic echo`에서 데이터가 나오지 않는 상황에서는 Publisher와 Subscriber의 QoS를 함께 확인해야 한다.

## 7. ROS 2 Humble 환경 확인

### 개발 환경

- Windows 11 + WSL2
- Ubuntu 22.04 LTS
- ROS 2 Humble Desktop

### Talker 실행

```bash
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_cpp talker
```

Talker가 `Hello World` 메시지를 발행하는 것을 확인했다.

별도 터미널에서 다음 명령으로 활성 노드와 토픽을 확인했다.

```bash
source /opt/ros/humble/setup.bash
ros2 node list
ros2 topic list
```

Talker 실행 후에는 `/talker` 노드와 `/chatter` 토픽이 나타나는 것을 확인했다.

추가로 다음 명령을 통해 토픽의 연결 정보, 발행 주기, 메시지 타입을 살펴볼 수 있다.

```bash
ros2 topic info /chatter
ros2 topic hz /chatter
ros2 topic type /chatter
```

## 8. QoS 호환성 실습 완료

QoS 설정을 직접 확인하고 Publisher와 Subscriber 간 호환성 실습을 진행했다. 사용자가 안내된 QoS 실습을 모두 수행했음을 확인했다.

- Best Effort Publisher와 Reliable Subscriber 조합 확인
- Best Effort Publisher와 Best Effort Subscriber 조합 확인
- `ros2 topic info -v`를 이용한 Endpoint QoS 확인
- QoS 호환성에 따른 통신 동작 확인

**검증 상태:** 실습 완료(사용자 확인). 상세 터미널 로그와 조합별 출력은 별도 증거 자료로 저장하지 않았으므로, 구체적인 경고 문구나 측정값은 임의로 재구성하지 않는다.

## 9. 정리

1. ROS 1 Master는 노드 등록과 연결 정보 중개를 담당하며, 일반적인 토픽 데이터를 직접 중계하지 않는다.
2. ROS 2는 DDS 기반 Discovery와 데이터 통신을 사용한다.
3. RMW는 ROS 2와 미들웨어 구현 사이의 추상화 인터페이스다.
4. QoS는 Reliability, Durability, History, Deadline 등 데이터 통신 정책을 정의한다.
5. Discovery와 실제 데이터 통신은 구분해야 하며 QoS 호환성도 확인해야 한다.

QoS 호환성 실습을 마쳤으며, 다음 학습에서는 ROS 2의 Topic, Service, Action이 각각 어떤 상황에 적합한지 실제 예제로 확인한다.

---

*이 글은 개인 학습 기록이며, 실제 시스템 적용 시에는 ROS 2 Humble 및 사용 중인 RMW/DDS 구현체의 공식 문서와 설정을 추가로 확인한다.*
