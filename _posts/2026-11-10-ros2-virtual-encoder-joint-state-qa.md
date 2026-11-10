---
layout: post
title: "ROS 2 질문 노트 07 — self, super(), Timer 주기와 JointState"
date: 2026-11-10 23:13:00 +0900
categories: [ros2, learning-notes]
tags: [ROS2, Humble, Python, OOP, self, super, Timer, JointState, Jitter]
excerpt: "Virtual Encoder 구현 중 나온 self와 super(), 인스턴스 변수, 50Hz/100Hz Timer의 위상 차이, JointState 필드와 effort 단위를 정리한다."
---

Lesson 7의 6-DOF Virtual Encoder를 구현하고 검증하는 과정에서 나온 질문과 답변을 정리한다.

## Q1. `self`와 `super()`는 무엇일까?

`self`는 현재 클래스의 인스턴스(객체)를 가리킨다. `self.positions`처럼 선언한 값은 객체에 저장되므로 다른 메서드에서도 접근할 수 있다.

반면 메서드 안에서 `positions = [...]`처럼 `self.` 없이 선언한 변수는 일반적으로 해당 메서드 안에서만 사용하는 지역 변수다.

`super()`는 부모 클래스의 메서드에 접근할 때 사용한다. 이번 클래스는 다음과 같이 ROS 2의 `Node`를 상속한다.

```python
class VirtualEncoderNode(Node):
    def __init__(self):
        super().__init__('virtual_encoder_node')
```

여기서 `super().__init__()`은 부모 클래스인 `Node`의 초기화 메서드를 호출하고 노드 이름을 설정한다. 이후 Publisher와 Timer 등 Node 기능을 생성한다.

## Q2. 왜 `last_slow_time`이 아니라 `self.last_slow_time`을 사용해야 할까?

다음 코드는 잘못된 상태 관리다.

```python
def update_slow_joints(self):
    current_time = time.monotonic()
    dt = current_time - self.last_slow_time
    last_slow_time = current_time
```

마지막 줄의 `last_slow_time`은 지역 변수에 값을 대입한다. 객체에 저장된 `self.last_slow_time`은 갱신되지 않으므로 다음 콜백에서도 이전 timestamp가 남는다.

올바른 코드는 다음과 같다.

```python
self.last_slow_time = current_time
```

콜백 사이에 유지해야 하는 상태라면 인스턴스 변수인 `self.`를 사용해야 한다.

## Q3. `time.monotonic()`을 사용해 실제 경과 시간 `dt`를 계산하는 이유는?

Timer를 0.01초로 설정했다고 해서 실제 콜백이 매번 정확히 0.01초 간격으로 실행되는 것은 아니다. 운영체제 스케줄링, 다른 콜백 실행 시간, 시스템 부하에 따라 실제 간격이 달라질 수 있다.

따라서 다음처럼 실제 경과 시간을 측정한다.

```python
current_time = time.monotonic()
dt = current_time - self.last_fast_time
self.last_fast_time = current_time
```

그 다음 위치를 다음 식으로 갱신한다.

`position_new = position_old + velocity × dt`

이렇게 하면 콜백 실행 간격의 변동을 위치 적분에 반영할 수 있다. `time.monotonic()`은 경과 시간 측정용 단조 증가 시계다.

## Q4. Python 리스트에서 `[2, 4]`는 어떤 관절을 의미할까?

Python 리스트 인덱스는 0부터 시작한다.

| 인덱스 | 관절 | 업데이트 주기 |
|---:|---|---:|
| 0 | joint_1 | 100Hz |
| 1 | joint_2 | 100Hz |
| 2 | joint_3 | 50Hz |
| 3 | joint_4 | 100Hz |
| 4 | joint_5 | 50Hz |
| 5 | joint_6 | 100Hz |

따라서 `[2, 4]`는 세 번째와 다섯 번째 관절인 Joint 3, Joint 5를 의미한다.

## Q5. 50Hz 관절인데 연속된 JointState 메시지에서 position이 모두 바뀌는 이유는?

`/joint_states` Publisher는 100Hz이고, Joint 3과 Joint 5의 업데이트 Timer는 50Hz다.

두 Timer는 별도로 생성되므로 시작 위상과 스케줄링 지터가 다를 수 있다. 50Hz Timer의 콜백이 두 번의 연속된 Publisher 실행 사이에 실행될 수도 있다. 따라서 연속된 두 메시지에서 값이 모두 달라지는 경우가 발생할 수 있다.

즉, Publisher 주기가 100Hz이고 관절 업데이트 주기가 50Hz라는 사실만으로 메시지마다 반드시 값이 한 번씩 반복된다고 보장할 수는 없다. 업데이트 콜백 횟수나 별도 timestamp를 기록해 확인하는 편이 정확하다.

## Q6. `position`, `velocity`, `effort`는 각각 무엇일까?

- `position`: 관절의 위치 또는 회전각
- `velocity`: 관절이 움직이는 속도 또는 각속도
- `effort`: 관절에 작용하는 힘 또는 토크

일반적인 단위는 다음과 같다.

| 필드 | 회전 관절 | 직선 관절 |
|---|---|---|
| position | rad | m |
| velocity | rad/s | m/s |
| effort | N·m (토크) | N (힘) |

따라서 effort가 항상 토크인 것은 아니다. 관절의 종류에 따라 힘 또는 토크를 나타낸다. 이번 실습의 `effort = [0.0] * 6`은 실제 측정값이 아니라 가상으로 0을 넣은 것이다.

## Q7. JointState의 timestamp는 관절마다 따로 존재할까?

아니다. `JointState` 메시지에는 `header.stamp`가 하나 있으며, 메시지에 포함된 관절 상태를 대표하는 시각이다. 현재 구현에서는 Publisher가 메시지를 만들 때 ROS Clock의 현재 시각을 기록한다.

따라서 이 timestamp가 각 관절의 실제 센서 샘플링 시각을 개별적으로 나타내는 것은 아니다. 관절마다 서로 다른 샘플링 시각이 중요하다면 별도의 데이터 구조나 메시지 설계가 필요하다.

## Q8. `position`이 계속 커지는 것이 정상일까?

이번 구현에서는 각 관절의 속도를 계속 적분하므로 위치가 누적되는 것이 정상이다.

`position(t) = position(0) + velocity × elapsed_time`

예를 들어 속도 1.0인 Joint 1은 약 94초 실행 후 약 94의 위치를 보였다. 속도 0인 Joint 4는 0을 유지하고, 음수 속도인 Joint 5는 음수 방향으로 이동했다.

다만 실제 로봇 관절에는 물리적 이동 범위와 안전 제한이 있다. 실제 시스템에 적용할 때는 URDF의 joint limit, 드라이버의 제한, 제어기 안전 로직 등을 함께 고려해야 한다.

## 이번 Q&A의 핵심

- `self.`는 객체 상태를 메서드 사이에서 유지한다.
- `super().__init__()`은 부모 ROS 2 Node 초기화를 수행한다.
- 경과 시간 기반 계산에는 `time.monotonic()`을 활용할 수 있다.
- Python 리스트 인덱스는 0부터 시작한다.
- 서로 다른 Timer는 실행 위상이 다를 수 있어 50Hz 데이터가 매번 교대로 반복된다고 보장할 수 없다.
- JointState의 `header.stamp`는 메시지 전체에 하나이며 관절별 센서 timestamp는 아니다.
- effort는 회전 관절에서 토크, 직선 관절에서 힘을 나타낸다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
