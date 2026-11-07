---
layout: post
title: "ROS 2 학습 기록 05 — 첫 번째 Node와 Timer/Publisher 구현"
date: 2026-11-07 23:55:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, Humble, Node, Timer, Publisher, Topic, Float64]
excerpt: "ament_python 패키지에 controller_node를 구현하고 Timer와 Publisher로 0.5초 주기의 관절 명령 예제 메시지를 발행한 뒤 echo와 hz로 검증한다."
---

## 1. 학습 목표

ROS 2 Python 패키지에 실행 가능한 Node를 만들고, Timer를 이용해 주기적으로 콜백을 실행하며, Publisher를 통해 Topic으로 메시지를 발행한다. 마지막으로 CLI 도구로 실제 메시지 수신과 발행 주기를 확인한다.

## 2. Node 구현

패키지 `joint_controller`에 `controller_node.py`를 만들고 `setup.py`의 console script를 통해 실행한다.

핵심 구조:

- `ControllerNode`는 `rclpy.node.Node`를 상속한다.
- `super().__init__('controller_node')`로 노드 이름을 지정한다.
- `main()`에서 `rclpy.init()`으로 ROS 2 Python 클라이언트를 초기화한다.
- `rclpy.spin(node)`으로 콜백을 처리하며 노드를 실행 상태로 유지한다.
- 종료 시 노드를 정리하고 `rclpy.shutdown()`을 호출한다.

실행 확인:

```bash
ros2 pkg executables joint_controller
ros2 run joint_controller controller_node
ros2 node list
ros2 node info /controller_node
```

초기 Node에는 사용자 정의 Publisher, Subscriber, Service, Action이 없으므로 `ros2 node info`에 기본 파라미터 서비스와 `/rosout`, `/parameter_events`만 보이는 것이 정상이다.

## 3. Timer와 Publisher 추가

`controller_node.py`의 핵심 구현:

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64


class ControllerNode(Node):

    def __init__(self):
        super().__init__('controller_node')

        self.publisher_ = self.create_publisher(
            Float64,
            '/joint_command',
            10
        )

        self.timer = self.create_timer(
            0.5,
            self.timer_callback
        )

        self.command_value = 0.0
        self.get_logger().info('Joint Controller Node has started!')

    def timer_callback(self):
        msg = Float64()
        self.command_value += 0.1
        msg.data = self.command_value

        self.publisher_.publish(msg)
        self.get_logger().info(
            f'Publishing joint command: {msg.data:.2f}'
        )


def main(args=None):
    rclpy.init(args=args)
    node = ControllerNode()

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

### 주요 API

| API | 역할 |
|---|---|
| `create_publisher(Float64, '/joint_command', 10)` | Float64 타입 Publisher 생성 |
| `create_timer(0.5, callback)` | 0.5초 간격으로 콜백 예약 |
| `publisher_.publish(msg)` | Topic으로 메시지 발행 |
| `rclpy.spin(node)` | Node의 콜백 처리 |

마지막 인자 `10`은 QoS 프로파일에서 Keep Last history의 depth로 사용되는 값이다.

## 4. 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --packages-select joint_controller
source install/setup.bash
ros2 run joint_controller controller_node
```

실행 로그에서 약 0.5초마다 0.1씩 증가하는 명령값을 확인했다.

## 5. Topic 수신 검증

별도 터미널에서 환경을 source한 뒤 Topic을 확인한다.

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash

ros2 topic list
ros2 topic info /joint_command
ros2 topic echo /joint_command
```

확인한 인터페이스:

- Topic: `/joint_command`
- Message type: `std_msgs/msg/Float64`
- Publisher count: 1
- Subscriber count: CLI의 `ros2 topic echo` 실행 전에는 0

`ros2 topic echo`에서 `data` 값이 계속 증가하는 것을 확인해 Publisher에서 발행한 메시지가 실제 ROS 2 그래프를 통해 전달됨을 검증했다.

## 6. 발행 주기 측정

```bash
ros2 topic hz /joint_command
```

측정 결과:

| 항목 | 관측 결과 |
|---|---:|
| Average rate | 약 2.000 Hz |
| Min interval | 0.497 s |
| Max interval | 0.501 s |
| Std dev | 약 0.00118 s |

Timer 설정 주기는 0.5초이므로 목표 발행 빈도는 다음과 같다.

`f = 1 / T = 1 / 0.5 = 2 Hz`

실제 관측값은 약 2.000Hz였으며, 측정 구간에서 메시지 간격도 0.5초에 근접했다. 이는 해당 개발 환경에서의 관측 결과이며, 일반 OS에서의 Timer 측정만으로 하드 실시간 성능을 보장하지는 않는다.

## 7. 부동소수점 출력 참고

`0.1`을 반복해서 더하면 `14.199999999999966`처럼 보일 수 있다. 이는 10진 소수 중 일부를 이진 부동소수점으로 정확히 표현할 수 없어 생기는 표현 오차이며 ROS 2 통신 오류가 아니다. 로그에서 `:.2f` 포맷을 사용하면 표시 자릿수를 제한할 수 있지만, 내부 값 자체가 정확한 십진수가 되는 것은 아니다.

## 8. 학습 결과와 현재 한계

- Python 기반 ROS 2 Node를 패키지 실행 파일로 등록하고 실행했다.
- Timer callback을 0.5초 간격으로 실행했다.
- `std_msgs/msg/Float64` Publisher를 생성해 `/joint_command`를 발행했다.
- `ros2 topic echo`로 실제 메시지 수신을 확인했다.
- `ros2 topic hz`에서 평균 약 2.000Hz를 측정했다.

**검증 상태: 사용자가 실행 로그, Topic 수신 데이터, 주기 측정 결과를 확인해 Lesson 5 실습 완료.**

현재 `/joint_command`는 통신 연습용 숫자 데이터다. 모터 드라이버나 실제 관절을 구동하지 않으며, 실제 제어에는 단위·범위·안전 제한·명령 인터페이스 정의가 추가로 필요하다.

## 9. 다음 단계

다음 실습에서는 별도의 Subscriber Node인 `state_monitor_node`를 구현해 `/joint_command`를 구독한다. 이를 통해 두 ROS 2 Node 사이의 Publisher/Subscriber 통신을 패키지 내부 코드로 직접 구성한다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
