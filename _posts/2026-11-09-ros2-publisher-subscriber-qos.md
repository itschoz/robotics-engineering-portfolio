---
layout: post
title: "ROS 2 학습 기록 06 — Publisher/Subscriber 통신과 QoS 호환성 검증"
date: 2026-11-09 22:50:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, Humble, Publisher, Subscriber, Topic, QoS, DDS, Python]
excerpt: "joint_controller 패키지에 Subscriber Node를 추가하고 두 Node 간 Topic 통신을 검증한 뒤, QoSProfile을 명시적으로 설정하고 Reliability 호환성 실험을 수행한다."
---

## 1. 학습 목표

Lesson 5에서 구현한 controller_node Publisher에 별도의 Subscriber Node를 연결한다. 두 Node가 서로 다른 프로세스로 실행되는 상황에서 Topic 통신이 이루어지는지 확인하고, QoSProfile을 코드에서 직접 설정한 뒤 Reliability 정책의 호환성과 비호환성을 실험한다.

## 2. 패키지 구성과 실행 진입점

기존 joint_controller Python 패키지에 다음 파일을 추가했다.

```text
joint_controller/
├── __init__.py
├── controller_node.py
└── state_monitor_node.py
```

새 Node를 ros2 run으로 실행할 수 있도록 패키지 루트의 setup.py에 console script를 등록한다.

```python
entry_points={
    'console_scripts': [
        'controller_node = joint_controller.controller_node:main',
        'state_monitor_node = joint_controller.state_monitor_node:main',
    ],
},
```

이 설정은 각 모듈의 최상위 수준에 main() 함수가 정의되어 있어야 한다는 뜻이다. Python 파일을 추가하는 것과 ROS 2 실행 명령으로 등록하는 것은 별개의 작업이다.

## 3. Subscriber Node 구현

state_monitor_node.py는 /joint_command Topic을 구독하고 수신한 Float64 값을 로그로 출력한다.

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64


class StateMonitorNode(Node):

    def __init__(self):
        super().__init__('state_monitor_node')

        self.subscription = self.create_subscription(
            Float64,
            '/joint_command',
            self.command_callback,
            10
        )

        self.get_logger().info(
            'State Monitor Node has started!'
        )

    def command_callback(self, msg):
        self.get_logger().info(
            f'Received joint command: {msg.data:.2f}'
        )


def main(args=None):
    rclpy.init(args=args)
    node = StateMonitorNode()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

create_subscription()의 인자는 메시지 타입, Topic 이름, 수신 시 호출할 callback, QoS 설정 순서다. callback은 메시지가 도착할 때 ROS 2 executor에 의해 호출된다.

## 4. 빌드와 두 Node 동시 실행

패키지에 Node를 추가하거나 코드를 변경한 뒤 빌드한다.

```bash
cd ~/ros2_ws
colcon build --packages-select joint_controller
source install/setup.bash
ros2 pkg executables joint_controller
```

실행 파일 목록에서 다음 두 항목을 확인했다.

```text
joint_controller controller_node
joint_controller state_monitor_node
```

서로 다른 터미널에서 각각 실행한다.

터미널 1:

```bash
source ~/ros2_ws/install/setup.bash
ros2 run joint_controller controller_node
```

터미널 2:

```bash
source ~/ros2_ws/install/setup.bash
ros2 run joint_controller state_monitor_node
```

Subscriber 로그에서 아래처럼 Publisher가 발행한 값이 약 0.5초 간격으로 수신되는 것을 확인했다.

```text
Received joint command: 0.80
Received joint command: 0.90
Received joint command: 1.00
Received joint command: 1.10
```

이로써 별도 Node 간 /joint_command Topic 통신이 동작함을 확인했다.

> 주의: 현재 state_monitor_node는 실제 엔코더 측정값을 받는 노드가 아니다. 실습에서 Publisher가 보낸 명령값을 구독해 출력하는 모니터링 예제다.

## 5. Node Graph와 Endpoint 확인

다음 CLI 명령으로 ROS 2 그래프와 Topic 연결을 검사했다.

```bash
ros2 node list
ros2 node info /controller_node
ros2 node info /state_monitor_node
ros2 topic info /joint_command --verbose
```

확인한 Endpoint는 다음과 같다.

- Publisher Node: controller_node
- Subscriber Node: state_monitor_node
- Topic: /joint_command
- Type: std_msgs/msg/Float64
- Publisher count: 1
- Subscription count: 1

Node와 Endpoint가 발견되었다는 사실과 QoS가 호환되어 실제 메시지가 전달된다는 사실은 구분해야 한다. Endpoint count만으로 통신 성공을 판단할 수 없다.

## 6. QoSProfile 명시적 설정

처음에는 create_publisher()와 create_subscription()의 마지막 인자에 숫자 10을 사용했다. 이는 depth를 지정하는 간편 QoS 설정이다.

이후 Publisher와 Subscriber 양쪽에서 QoS 정책을 직접 정의하도록 변경했다.

```python
from rclpy.qos import QoSProfile
from rclpy.qos import ReliabilityPolicy
from rclpy.qos import DurabilityPolicy
from rclpy.qos import HistoryPolicy

qos_profile = QoSProfile(
    history=HistoryPolicy.KEEP_LAST,
    depth=10,
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.VOLATILE
)
```

Publisher에는 다음과 같이 전달한다.

```python
self.publisher_ = self.create_publisher(
    Float64,
    '/joint_command',
    qos_profile
)
```

Subscriber에도 같은 프로파일을 전달한다.

```python
self.subscription = self.create_subscription(
    Float64,
    '/joint_command',
    self.command_callback,
    qos_profile
)
```

| 정책 | 설정 | 의미 |
|---|---|---|
| History | KEEP_LAST | 최근 N개 메시지를 유지 |
| Depth | 10 | 보관할 메시지 수 |
| Reliability | RELIABLE | 신뢰성 있는 전달을 요구 |
| Durability | VOLATILE | 늦게 참여한 Subscriber에게 과거 메시지를 제공하지 않음 |

ros2 topic info --verbose 출력에서 Reliability와 Durability는 양쪽 Endpoint 모두 기대한 값으로 표시됐다. 다만 History는 CLI에서 UNKNOWN으로 표시되었다. 코드에는 KEEP_LAST와 depth=10을 명시했으며, CLI의 UNKNOWN 표시만으로 코드 설정이 적용되지 않았다고 결론 내리면 안 된다.

## 7. Reliability QoS 호환성 실험

ROS 2 QoS의 요청/제공(requested/offered) 호환성 규칙을 확인하기 위해 Publisher와 Subscriber의 Reliability를 다르게 설정했다.

| Publisher가 제공하는 정책 | Subscriber가 요청하는 정책 | 호환성 |
|---|---|---|
| RELIABLE | RELIABLE | 호환 |
| RELIABLE | BEST_EFFORT | 호환 |
| BEST_EFFORT | BEST_EFFORT | 호환 |
| BEST_EFFORT | RELIABLE | 비호환 |

### 실험 A — RELIABLE Publisher / BEST_EFFORT Subscriber

Publisher를 RELIABLE로 두고 Subscriber를 BEST_EFFORT로 변경했다.

ros2 topic info /joint_command --verbose에서 다음 설정을 확인했다.

- Publisher Reliability: RELIABLE
- Subscriber Reliability: BEST_EFFORT
- Durability: 양쪽 VOLATILE
- Subscription count: 1

이 조합은 DDS QoS 호환성 규칙상 호환된다. RELIABLE Publisher는 BEST_EFFORT Subscriber가 요구하는 전달 수준을 충족할 수 있다.

### 실험 B — BEST_EFFORT Publisher / RELIABLE Subscriber

반대로 Publisher를 BEST_EFFORT로, Subscriber를 RELIABLE로 설정했다. Subscriber 실행 시 실제로 다음 경고가 발생했다.

```text
New publisher discovered on topic '/joint_command',
offering incompatible QoS. No messages will be received from it.
Last incompatible policy: RELIABILITY
```

이는 Subscriber가 요구하는 RELIABLE 수준을 Publisher가 제공하지 못해 통신이 성립하지 않았다는 의미다. 마지막 비호환 정책으로 RELIABILITY가 명시되었다.

BEST_EFFORT Publisher와 BEST_EFFORT Subscriber 조합은 호환성 표에 포함했지만, 이번 실습에서 별도로 실행해 확인한 조합은 아니다.

## 8. 이번 실습에서 배운 핵심

1. Python Node 파일을 추가하는 것만으로는 ros2 run 실행 명령이 생기지 않는다. setup.py의 console script 등록이 필요하다.
2. Publisher와 Subscriber는 서로 다른 Node로 실행할 수 있고, 동일한 Topic과 메시지 타입을 사용해 통신한다.
3. QoSProfile을 코드에서 명시적으로 설정하면 History, Depth, Reliability, Durability 정책을 의도적으로 관리할 수 있다.
4. Endpoint가 발견되어도 QoS가 호환되지 않으면 메시지를 주고받지 못할 수 있다.
5. BEST_EFFORT Publisher → RELIABLE Subscriber 조합은 Reliability 정책 불일치로 비호환이다.
6. ros2 topic info --verbose의 Endpoint 정보와 Subscriber 수신 로그를 함께 확인해야 통신 상태를 더 정확하게 판단할 수 있다.

**검증 상태: Subscriber 수신 로그, Publisher/Subscriber Endpoint, QoS 정책 표시, Reliability 비호환 경고를 확인해 Lesson 6 실습 완료.**

## 9. 다음 단계

다음 Lesson에서는 로봇 제어 시스템의 명령(Command)과 측정 상태(State)를 분리한다. 현재 예제의 /joint_command는 명령값이며, 향후 엔코더에서 읽은 실제 관절 상태를 별도 Topic으로 표현하는 구조로 확장한다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
