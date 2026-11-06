---
layout: post
title: "ROS 2 통신 구조 이해하기: DDS, Discovery, QoS"
date: 2026-11-06 10:00:00 +0900
categories: [ROS2, Middleware]
tags: [ROS2, DDS, QoS, Robotics]
---

## 1. 왜 ROS 2 통신 구조를 다시 이해해야 할까?

ROS 1에서 ROS Master를 중심으로 노드들이 서로를 찾고 통신했다면, ROS 2는 기본 통신 구조에 DDS(Data Distribution Service)를 활용한다.

ROS 1 경험이 있는 개발자에게 중요한 변화는 단순히 명령어가 달라진 것이 아니다. 노드 검색과 데이터 전달을 담당하는 미들웨어 계층이 달라졌고, 통신 품질을 QoS(Quality of Service) 정책으로 명시적으로 설정할 수 있게 되었다.

이 글에서는 ROS 1과 ROS 2의 구조적 차이, DDS Discovery, QoS의 핵심 개념을 정리한다.

## 2. ROS 1과 ROS 2의 통신 구조

### ROS 1

ROS 1에서는 ROS Master가 노드와 토픽 정보를 관리한다. 노드들은 Master에 자신의 정보를 등록하고, 서로의 연결 정보를 확인한 뒤 데이터를 주고받는다.

일반적인 구조는 다음과 같다.

- ROS Master: 노드 및 토픽 등록·검색 지원
- Publisher / Subscriber: 토픽 데이터 송수신
- TCPROS 또는 UDPROS: 실제 데이터 전송

### ROS 2

ROS 2에서는 ROS Master가 기본 통신 구조의 필수 구성 요소가 아니다. ROS 2의 클라이언트 라이브러리는 RMW(ROS Middleware Interface)를 통해 DDS 구현체와 연결된다.

개념적인 계층은 다음과 같다.

```text
ROS 2 Application
       |
      rcl
       |
      RMW
       |
   DDS / RTPS
       |
     Network
```

- Application: 사용자가 작성하는 Node, Publisher, Subscriber
- rcl: ROS Client Library의 공통 기능 계층
- RMW: ROS와 미들웨어 구현체 사이의 추상화 인터페이스
- DDS: 분산 시스템에서 데이터 발행·구독과 검색을 지원하는 미들웨어

따라서 ROS 2에서는 DDS 구현체가 제공하는 Discovery와 QoS 기능을 ROS 2 환경에서 활용할 수 있다.

## 3. DDS Discovery

Discovery는 네트워크에 존재하는 Participant, Publisher, Subscriber 등의 정보를 서로 발견하는 과정이다.

예를 들어 서로 다른 컴퓨터에서 실행 중인 두 ROS 2 노드가 같은 도메인과 네트워크 조건에서 통신할 수 있다면, DDS Discovery가 상대 Participant와 Endpoint 정보를 발견하는 데 관여한다.

기억할 점:

- ROS 1처럼 반드시 하나의 중앙 ROS Master에 등록하는 구조가 아니다.
- ROS_DOMAIN_ID는 ROS 2 노드들이 참여하는 DDS 도메인을 구분하는 데 사용된다.
- 네트워크 설정, 방화벽, DDS 구현체 설정 등에 따라 Discovery가 실패할 수 있다.

## 4. QoS란?

QoS(Quality of Service)는 Publisher와 Subscriber 사이의 데이터 전달 특성을 정의하는 정책 모음이다.

토픽 이름과 메시지 타입이 같더라도 QoS 설정이 호환되지 않으면 통신이 연결되지 않을 수 있다.

### Reliability

메시지 전달 신뢰성 정책이다.

- Reliable: 미들웨어가 데이터 전달을 보장하기 위해 재전송 등을 활용한다.
- Best Effort: 전달을 시도하지만 손실 가능성을 허용한다.

센서 데이터처럼 최신 값이 중요하고 일부 손실을 허용할 수 있는 경우와, 명령·상태 데이터처럼 전달 신뢰성이 중요한 경우를 구분해 고려해야 한다.

### Durability

새로운 Subscriber가 참여했을 때 이전에 발행된 데이터를 받을 수 있는지와 관련된다.

- Volatile: 과거 데이터를 보관해 새 Subscriber에 제공하지 않는다.
- Transient Local: Publisher가 보관한 최근 데이터를 늦게 참여한 Subscriber에 제공할 수 있다.

예를 들어 로봇의 정적 설정값이나 초기 상태처럼 새로 참여한 노드도 직전 데이터를 받아야 하는 상황에서 고려할 수 있다.

### History와 Depth

- Keep Last(N): 최근 N개 샘플을 유지한다.
- Keep All: 미들웨어의 자원 한도 내에서 가능한 샘플을 유지한다.

Depth는 Keep Last 정책에서 보관할 샘플 수를 의미한다. 깊이를 크게 설정하면 버퍼 자원 사용량도 고려해야 한다.

### Deadline

Deadline은 발행 또는 수신 데이터가 기대되는 최대 간격을 정의한다. 설정한 주기를 지키지 못하는 상황을 감지하는 데 활용할 수 있지만, Deadline 자체가 발행 주기를 강제로 제어하는 것은 아니다.

## 5. QoS 호환성에서 주의할 점

Reliability 정책의 대표적인 호환 관계는 다음과 같다.

| Publisher | Subscriber | 호환 여부 |
|---|---|---|
| Reliable | Reliable | 호환 |
| Reliable | Best Effort | 호환 |
| Best Effort | Best Effort | 호환 |
| Best Effort | Reliable | 비호환 |

따라서 센서 Publisher가 Best Effort인데 Subscriber가 Reliable을 요구하면 연결이 성립하지 않을 수 있다.

실제 로봇 시스템에서는 토픽의 목적과 데이터 특성에 맞춰 Publisher와 Subscriber의 QoS를 함께 설계해야 한다.

## 6. 직접 확인할 실습

아래 실습은 ROS 2 Humble 환경에서 수행할 계획이다. 명령어와 기대 결과를 먼저 정리하고, 실제 실행 결과는 테스트 후 별도로 기록한다.

### 기본 통신 확인

터미널 1:

```bash
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_cpp talker
```

터미널 2:

```bash
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_py listener
```

### 토픽 QoS 확인

```bash
ros2 topic info /chatter --verbose
```

이 명령으로 Publisher와 Subscriber의 QoS 설정을 확인할 수 있다.

### 추가 검증 항목

- Publisher와 Subscriber의 Reliability를 다르게 설정했을 때 연결 결과
- Transient Local Publisher에 늦게 참여한 Subscriber가 데이터를 받는지
- ROS_DOMAIN_ID를 변경했을 때 Discovery 결과가 달라지는지

> **검증 상태:** 위의 추가 QoS 실험은 아직 실행 결과를 확인하지 않았다. 실제 실험을 수행한 뒤 명령어, 환경, 관찰 결과와 함께 업데이트할 예정이다.

## 7. 이번 학습의 핵심 정리

1. ROS 2는 ROS Master에 의존하지 않는 통신 구조를 제공한다.
2. RMW는 ROS 2와 DDS 구현체 사이의 인터페이스 역할을 한다.
3. DDS Discovery는 분산된 노드와 Endpoint를 발견하는 과정이다.
4. QoS는 데이터 전달의 신뢰성, 보관 방식, 주기 등의 특성을 정의한다.
5. QoS는 Publisher와 Subscriber 양쪽의 호환성을 함께 확인해야 한다.

다음 학습에서는 ROS 2의 통신 인터페이스인 Topic, Service, Action의 차이를 살펴보고, 각 방식이 로봇 시스템에서 어떤 상황에 적합한지 정리할 예정이다.
