---
layout: post
title: "ROS 2 학습 기록 07 — 6-DOF Virtual Encoder와 JointState Publisher"
date: 2026-11-10 23:12:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, Humble, Python, Node, Timer, JointState, Encoder, Sensor, Jitter]
excerpt: "6개 관절의 가상 엔코더 데이터를 100Hz와 50Hz로 갱신하고 JointState 메시지를 100Hz로 발행한 뒤 실제 주기와 메시지 구조를 검증한다."
---

## 1. 학습 목표

이번 실습에서는 6자유도(6-DOF) 관절의 가상 엔코더 데이터를 생성하고 ROS 2 표준 메시지인 `sensor_msgs/msg/JointState`로 발행한다.

핵심 목표는 다음과 같다.

- 6개 관절의 이름, 위치, 속도, 힘/토크 데이터를 구성한다.
- 관절 그룹별 업데이트 주기를 100Hz와 50Hz로 분리한다.
- 전체 관절 상태 메시지는 100Hz로 발행한다.
- 고정된 시간 간격을 가정하지 않고 실제 경과 시간(`dt`)으로 위치를 계산한다.
- `ros2 topic hz`와 `ros2 topic echo`로 발행 주기와 메시지 구조를 검증한다.

## 2. 시스템 구조

이번 구현은 하나의 Node 안에 세 개의 Timer를 둔다.

| Timer | 주기 | 대상 |
|---|---:|---|
| Fast update | 0.01초 / 100Hz | Joint 1, 2, 4, 6 |
| Slow update | 0.02초 / 50Hz | Joint 3, 5 |
| Publisher | 0.01초 / 100Hz | 6개 관절 전체 상태 |

관절 데이터 업데이트 주기와 메시지 발행 주기는 서로 다른 개념이다. Publisher는 100Hz로 메시지를 발행하지만, 각 관절의 position은 해당 관절 그룹의 업데이트 Timer가 마지막으로 계산한 최신 값을 포함한다.

```text
Virtual Encoder Node
 ├─ 100Hz Timer → Joint 1, 2, 4, 6 업데이트
 ├─  50Hz Timer → Joint 3, 5 업데이트
 └─ 100Hz Timer → JointState 발행
                         ↓
                    /joint_states
```

## 3. JointState 메시지

`JointState`는 같은 시점에 관측된 여러 관절의 상태를 하나의 메시지로 표현하는 표준 ROS 메시지다.

| 필드 | 의미 | 일반적인 단위 |
|---|---|---|
| `header.stamp` | 메시지에 담긴 관절 상태의 기준 시각 | ROS Time |
| `name` | 관절 이름 배열 | 문자열 |
| `position` | 관절 위치 또는 회전각 | rad 또는 m |
| `velocity` | 관절 속도 또는 각속도 | rad/s 또는 m/s |
| `effort` | 관절에 작용하는 힘 또는 토크 | N 또는 N·m |

회전 관절(Revolute Joint)의 effort는 일반적으로 토크(N·m), 직선 관절(Prismatic Joint)의 effort는 힘(N)이다. 실제 단위는 로봇 모델과 인터페이스 정의에 맞춰야 한다.

이번 실습에서 effort는 실제 센서 측정값이 아니라 `0.0`으로 채운 가상 데이터다. 또한 관절 위치의 단위를 별도로 모델링하지 않았으므로, 현재 position 값은 시간에 따른 누적 수치를 검증하기 위한 예제 데이터다.

## 4. 실제 경과 시간을 이용한 위치 계산

관절 위치는 다음 식으로 갱신한다.

`position_new = position_old + velocity × dt`

여기서 `dt`는 Timer에 설정한 주기값을 그대로 사용하지 않고, 이전 콜백과 현재 콜백 사이에 실제로 경과한 시간이다.

```python
current_time = time.monotonic()
dt = current_time - self.last_fast_time
self.last_fast_time = current_time

self.positions[i] += self.velocities[i] * dt
```

Timer를 0.01초로 설정해도 운영체제 스케줄링과 다른 콜백의 실행 시간에 따라 실제 콜백 간격은 정확히 0.01초가 아닐 수 있다. 실제 경과 시간으로 적분하면 이런 주기 변동을 위치 계산에 반영할 수 있다.

`time.monotonic()`은 경과 시간 측정에 적합한 단조 증가 시계다. 시스템 날짜/시각 변경의 영향을 받는 일반 wall-clock 시간과 구분된다. 이번 구현에서는 로컬 경과 시간 계산에 사용했고, 메시지 timestamp에는 ROS Clock을 사용했다.

## 5. Virtual Encoder Node 구현

파일: `joint_controller/virtual_encoder_node.py`

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import JointState
import time


class VirtualEncoderNode(Node):

    def __init__(self):
        super().__init__('virtual_encoder_node')

        # 각 관절 그룹의 마지막 업데이트 시간
        self.last_fast_time = time.monotonic()
        self.last_slow_time = time.monotonic()

        self.publisher_ = self.create_publisher(
            JointState,
            '/joint_states',
            10
        )

        self.timer_100hz = self.create_timer(
            0.01,
            self.update_fast_joints
        )

        self.timer_50hz = self.create_timer(
            0.02,
            self.update_slow_joints
        )

        self.publish_timer = self.create_timer(
            0.01,
            self.publish_joint_states
        )

        self.joint_names = [
            'joint_1',
            'joint_2',
            'joint_3',
            'joint_4',
            'joint_5',
            'joint_6'
        ]

        self.positions = [0.0] * 6

        self.velocities = [
            1.0, 0.5, 0.2, 0.0, -0.3, 0.1
        ]

        self.get_logger().info(
            '6-DOF Virtual Encoder Node has started!'
        )

    def update_fast_joints(self):
        current_time = time.monotonic()
        dt = current_time - self.last_fast_time
        self.last_fast_time = current_time

        for i in [0, 1, 3, 5]:
            self.positions[i] += self.velocities[i] * dt

    def update_slow_joints(self):
        current_time = time.monotonic()
        dt = current_time - self.last_slow_time
        self.last_slow_time = current_time

        for i in [2, 4]:
            self.positions[i] += self.velocities[i] * dt

    def publish_joint_states(self):
        msg = JointState()

        msg.header.stamp = self.get_clock().now().to_msg()
        msg.name = self.joint_names
        msg.position = self.positions.copy()
        msg.velocity = self.velocities.copy()
        msg.effort = [0.0] * 6

        self.publisher_.publish(msg)


def main(args=None):
    rclpy.init(args=args)
    node = VirtualEncoderNode()

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

### 구현 시 주의한 점

1. `super().__init__()`을 먼저 호출해 부모 클래스인 ROS 2 `Node`를 초기화한다.
2. `self.last_fast_time`, `self.last_slow_time`은 콜백 사이에서 유지되어야 하므로 인스턴스 변수로 선언한다.
3. `self.positions[i]`에 실제 경과 시간 기반 증가량을 누적한다.
4. `msg.position = self.positions.copy()`로 현재 위치 배열의 복사본을 메시지에 넣는다.
5. 100Hz마다 로그를 출력하면 측정과 실행에 불필요한 부하가 생길 수 있으므로 반복 로그는 비활성화했다.

## 6. 패키지 실행 파일 등록 및 빌드

`setup.py`의 `console_scripts`에 실행 진입점을 등록한다.

```python
'virtual_encoder_node = joint_controller.virtual_encoder_node:main',
```

패키지 의존성에는 `sensor_msgs`가 포함되어야 한다.

빌드 및 실행:

```bash
cd ~/ros2_ws
colcon build --packages-select joint_controller
source install/setup.bash
ros2 run joint_controller virtual_encoder_node
```

별도 터미널에서 확인:

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash

ros2 topic hz /joint_states
ros2 topic echo /joint_states
```

## 7. 실험 결과

### 7.1 발행 주기 측정

사용자가 실행한 `ros2 topic hz /joint_states` 결과:

| 항목 | 측정값 |
|---|---:|
| Average rate | 약 100.001Hz |
| Min interval | 0.004초 |
| Max interval | 0.014초 |
| Std dev | 약 0.00039초 |
| Window | 604 |

평균 발행 주기는 목표인 100Hz에 근접했다. 다만 최소/최대 간격은 각각 4ms와 14ms로 달랐다. 이는 Timer 콜백이 일반 OS 스케줄링의 영향을 받기 때문이다.

이 결과는 해당 개발 환경에서의 관측 결과다. 평균 100Hz가 측정되었다고 해서 매 메시지가 정확히 10ms 간격으로 발행되거나 하드 실시간 성능이 보장되는 것은 아니다.

### 7.2 메시지 구조 확인

사용자가 `ros2 topic echo /joint_states`로 확인한 메시지:

- `name`: joint_1부터 joint_6까지 6개
- `position`: 관절별 위치 값 6개
- `velocity`: `[1.0, 0.5, 0.2, 0.0, -0.3, 0.1]`
- `effort`: 6개 모두 0.0
- `header.stamp`: sec와 nanosec를 포함한 ROS timestamp

실제 출력된 position은 다음과 같았다.

| 관절 | 속도 | 관측 위치 |
|---|---:|---:|
| Joint 1 | 1.0 | 94.010 |
| Joint 2 | 0.5 | 47.005 |
| Joint 3 | 0.2 | 18.802 |
| Joint 4 | 0.0 | 0.000 |
| Joint 5 | -0.3 | -28.203 |
| Joint 6 | 0.1 | 9.401 |

위치 값은 실행 후 경과 시간에 비례해 누적되었다. Joint 4는 속도가 0이므로 위치가 계속 0으로 유지되었고, Joint 5는 음수 속도에 따라 음수 방향으로 이동했다.

## 8. Timer 주기와 데이터 발행 주기의 차이

50Hz로 업데이트되는 관절도 100Hz 메시지에 포함된다. 따라서 Publisher가 발행하는 모든 메시지에 해당 관절의 최신 position 값이 담기지만, 그 값이 매번 새로 계산되었다는 뜻은 아니다.

또한 서로 독립적으로 생성된 50Hz Timer와 100Hz Publisher는 시작 위상과 실행 지터가 다를 수 있다. 그러므로 50Hz 관절의 값이 출력 메시지마다 반드시 한 번씩 교대로 바뀌거나, 항상 두 번째 메시지에서만 바뀐다고 단정할 수 없다. 업데이트 횟수와 timestamp를 별도로 관측해 검증해야 한다.

## 9. 학습 결과와 현재 한계

이번 실습에서 확인한 내용:

- 6개 관절의 가상 위치·속도 데이터를 생성했다.
- 관절 그룹별로 100Hz와 50Hz Timer를 분리했다.
- 실제 경과 시간 기반으로 위치를 누적 계산했다.
- `JointState`를 100Hz로 발행했다.
- `ros2 topic hz`에서 평균 약 100Hz를 확인했다.
- `ros2 topic echo`에서 6개 관절의 이름과 position, velocity, effort, timestamp를 확인했다.

현재 구현은 가상 엔코더다. 실제 센서, 모터 드라이버, 관절 한계, 엔코더 해상도, 데이터 신선도, 하드웨어 timestamp는 구현하지 않았다. 따라서 실제 로봇 제어에 바로 사용할 수 있는 드라이버는 아니다.

## 10. 다음 단계

다음 학습에서는 URDF(Unified Robot Description Format)로 로봇의 링크와 관절 구조를 정의하고, `robot_state_publisher`와 `joint_state_publisher` 계열 도구를 연결해 관절 상태가 로봇 모델과 TF에 어떻게 반영되는지 확인한다.

이를 통해 지금까지 만든 `/joint_states` 데이터가 로봇 모델의 관절 움직임과 어떻게 연결되는지 이해한다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
