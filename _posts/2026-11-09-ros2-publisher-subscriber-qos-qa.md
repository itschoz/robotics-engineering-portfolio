---
layout: post
title: "ROS 2 질문 노트 06 — QoSProfile 설정 위치와 Reliability 호환성"
date: 2026-11-09 22:51:00 +0900
categories: [ros2, learning-notes]
tags: [ROS2, Humble, QoS, QoSProfile, Reliability, DDS]
excerpt: "QoS 설정이 코드 어디에 반영되는지, 정수 depth 인자와 QoSProfile의 차이, verbose 출력의 UNKNOWN, Reliability 비호환 경고를 정리한다."
---

Lesson 6에서 Publisher와 Subscriber를 구현하고 QoSProfile을 직접 설정하면서 나온 질문과 실험 결과를 정리한다.

## Q1. QoS 설정은 어느 코드에 반영되어 있었을까?

처음에는 Publisher와 Subscriber 생성 함수의 마지막 인자에 숫자 10을 사용했다.

Publisher:

```python
self.publisher_ = self.create_publisher(
    Float64,
    '/joint_command',
    10
)
```

Subscriber:

```python
self.subscription = self.create_subscription(
    Float64,
    '/joint_command',
    self.command_callback,
    10
)
```

이 숫자는 QoS의 History Depth를 지정하는 간편 설정이다. 이후 QoSProfile 객체를 직접 생성해 각 정책을 명시했다.

```python
qos_profile = QoSProfile(
    history=HistoryPolicy.KEEP_LAST,
    depth=10,
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.VOLATILE
)
```

그리고 Publisher 또는 Subscriber 생성 함수의 마지막 인자로 qos_profile을 전달했다. QoS는 별도의 설정 파일이 아니라 각 Publisher/Subscriber Endpoint를 생성하는 코드에 반영된다.

## Q2. QoSProfile을 직접 설정하는 이유는?

숫자 10을 전달하는 간편 방식은 빠르고 간단하지만, 정책을 명시적으로 바꾸거나 검토할 때는 QoSProfile 객체가 더 명확하다.

- History: 메시지 보관 방식
- Depth: 보관할 메시지 개수
- Reliability: 전달 신뢰성 정책
- Durability: 늦게 참여한 Subscriber에 과거 메시지를 제공할지 여부

여러 Publisher와 Subscriber가 연결되는 시스템에서는 각 Endpoint의 QoS를 명확하게 관리하는 것이 중요하다.

## Q3. ros2 topic info --verbose에서 History가 UNKNOWN으로 표시된 이유는?

실습 코드에서는 다음을 명시했다.

```python
history=HistoryPolicy.KEEP_LAST,
depth=10,
```

그런데 CLI 출력에서는 다음과 같이 표시되었다.

```text
History (Depth): UNKNOWN
```

이 출력만으로 QoSProfile이 적용되지 않았다고 판단하면 안 된다. 이번 실습에서 Reliability와 Durability는 설정값대로 표시되었고, 코드에는 History와 Depth도 명시되어 있었다. CLI가 History 정보를 UNKNOWN으로 표시한 경우에는 코드 설정과 다른 Endpoint 정보를 함께 확인해야 한다.

## Q4. RELIABLE Publisher → BEST_EFFORT Subscriber는 왜 호환될까?

QoS 호환성은 Publisher가 제공하는(offered) 품질과 Subscriber가 요청하는(requested) 품질을 비교한다.

RELIABLE Publisher는 BEST_EFFORT Subscriber가 요구하는 수준을 충족할 수 있다. 따라서 이 조합은 호환된다.

반대 방향은 다르다. BEST_EFFORT Publisher는 RELIABLE Subscriber가 요구하는 신뢰성을 제공하지 못하므로 비호환이다.

## Q5. BEST_EFFORT Publisher → RELIABLE Subscriber에서 어떤 오류가 발생했을까?

Subscriber를 실행했을 때 다음 경고가 실제로 출력되었다.

```text
New publisher discovered on topic '/joint_command',
offering incompatible QoS. No messages will be received from it.
Last incompatible policy: RELIABILITY
```

핵심은 마지막 줄이다.

Last incompatible policy: RELIABILITY는 두 Endpoint의 Reliability 요구/제공 정책이 맞지 않는다는 뜻이다. 노드가 실행되고 Endpoint가 발견되더라도 QoS가 호환되지 않으면 메시지는 전달되지 않는다.

## Q6. Subscription count가 1이면 통신 성공이라고 볼 수 있을까?

아니다. Subscription count는 해당 Topic에 Subscriber Endpoint가 하나 있다는 뜻이다. Publisher와 Subscriber가 서로 발견되었다는 사실만으로 QoS 호환성이나 실제 메시지 전달까지 증명되지는 않는다.

통신 상태를 확인할 때는 다음을 함께 본다.

1. ros2 node list — 노드가 실행 중인지
2. ros2 topic info /joint_command --verbose — Publisher/Subscriber Endpoint와 QoS 정보
3. Subscriber 로그 또는 ros2 topic echo — 실제 메시지가 도착하는지
4. QoS 경고 — 비호환 정책이 있는지

## 이번 Q&A의 핵심

- QoS는 Publisher/Subscriber Endpoint 생성 코드에 설정한다.
- 숫자 depth 인자 대신 QoSProfile 객체로 정책을 명시할 수 있다.
- Endpoint 발견과 메시지 전달 성공은 서로 다른 단계다.
- RELIABLE Publisher → BEST_EFFORT Subscriber는 호환된다.
- BEST_EFFORT Publisher → RELIABLE Subscriber는 Reliability 정책 때문에 비호환이다.
- CLI의 History UNKNOWN 표시만으로 코드의 QoS 설정 실패를 단정하지 않는다.
