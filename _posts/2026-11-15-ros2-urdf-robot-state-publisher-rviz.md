---
layout: post
title: "ROS 2 학습 기록 08 — URDF, Robot State Publisher와 RViz2"
date: 2026-11-15 23:00:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, Humble, URDF, robot_state_publisher, TF2, RViz2, JointState, RobotDescription]
excerpt: "6-DOF 로봇의 링크와 관절을 URDF로 정의하고 robot_state_publisher, JointState, TF2, RViz2를 연결해 모델을 시각화한다."
---

## 1. 학습 목표

이번 실습에서는 6자유도(6-DOF) 로봇의 구조를 URDF로 정의하고, ROS 2에서 로봇 모델과 관절 상태가 어떻게 연결되는지 확인한다.

핵심 목표는 다음과 같다.

- URDF의 link, joint, visual, collision 요소 이해
- 6개 revolute joint와 tool0 fixed joint로 로봇 체인 구성
- `robot_state_publisher`가 URDF와 JointState를 이용해 TF를 발행하는 과정 이해
- RViz2의 RobotModel로 로봇 형상 시각화
- `/joint_states`, `/tf`, `/tf_static`의 역할 구분
- URDF의 collision 및 joint limit 정의와 실제 물리 시뮬레이션/제어의 차이 이해

## 2. URDF는 로봇의 구조를 정의한다

URDF(Unified Robot Description Format)는 로봇을 구성하는 링크와 관절, 각 링크의 형상, 관절 연결 관계 등을 기술하는 XML 기반 포맷이다.

| 요소 | 역할 |
|---|---|
| `robot` | 로봇 모델의 최상위 요소 |
| `link` | 로봇을 구성하는 강체 링크 |
| `joint` | 부모 링크와 자식 링크 사이의 연결 관계 |
| `parent` / `child` | 관절로 연결되는 링크 지정 |
| `origin` | 부모 링크 기준 관절 프레임의 위치와 방향 |
| `axis` | 관절 프레임 기준 움직임 축 |
| `limit` | 관절 위치, 속도, effort 한계 정보 |
| `visual` | 시각화에 사용하는 형상 |
| `collision` | 충돌 검사에 사용할 형상 |
| `geometry` | box, cylinder, sphere, mesh 등의 기하 형상 |

URDF는 로봇의 구조와 기구학적 관계를 설명한다. 하지만 그 자체가 모터를 구동하거나 충돌 물리를 계산하지는 않는다.

## 3. 6-DOF 로봇의 링크/관절 체인

이번 모델은 다음 순서로 연결했다.

```text
base_link
  └─ joint_1 (revolute, Z)
      └─ link_1
          └─ joint_2 (revolute, Y)
              └─ link_2
                  └─ joint_3 (revolute, Y)
                      └─ link_3
                          └─ joint_4 (revolute, X)
                              └─ link_4
                                  └─ joint_5 (revolute, Y)
                                      └─ link_5
                                          └─ joint_6 (revolute, X)
                                              └─ link_6
                                                  └─ tool_fixed_joint
                                                      └─ tool0
```

각 관절의 `origin xyz/rpy`는 부모 링크 기준으로 정의한다. `axis xyz`는 joint frame 기준으로 정의한다. 부모 링크가 회전하면 자식 관절의 base/world 기준 위치와 축 방향도 부모 프레임 변환의 영향을 받는다.

이번 모델은 URDF와 TF 학습을 위한 초기 기구학 모델이다. 실제 로봇의 모터, 감속기, 베어링, 샤프트, 케이블 배치, 간섭, 강성 및 하중 조건까지 검증한 기계 설계는 아니다.

## 4. Visual과 Collision의 차이

각 링크에는 시각화 형상인 `visual`과 충돌 계산용 형상인 `collision`을 정의할 수 있다.

- `visual`: RViz 등에서 로봇의 외관을 표시한다.
- `collision`: 물리 엔진이나 충돌 검사기가 충돌 계산에 사용할 수 있는 단순화 형상이다.

중요한 점은 collision geometry를 URDF에 작성했다고 충돌이 자동으로 감지되거나 움직임이 차단되는 것은 아니라는 사실이다. 충돌 검사 또는 물리 시뮬레이션을 수행하는 별도 시스템이 collision geometry를 읽고 계산해야 한다.

## 5. Joint limit은 정의와 강제 적용을 구분해야 한다

Revolute joint 예시:

```xml
<limit lower="-3.14" upper="3.14" effort="10" velocity="1.0"/>
```

| 속성 | 의미 | 일반 단위 |
|---|---|---|
| lower / upper | 최소·최대 관절 위치 | rad |
| effort | 허용 effort 한계 | N·m |
| velocity | 최대 관절 속도 | rad/s |

이 값은 관절의 한계 정보를 기술한다. 그러나 `robot_state_publisher`는 입력된 JointState를 받아 TF를 계산하는 역할이지, 명령 속도를 제한하거나 입력 위치를 자동으로 clamp하는 제어기가 아니다.

따라서 실제 시스템에서는 드라이버, controller, trajectory validation, safety layer 등에서 한계를 검증하고 강제해야 한다.

## 6. Robot State Publisher와 JointState

이번 실습에서 연결한 데이터 흐름:

```text
URDF ───────────────────────┐
                            v
/joint_states ──> robot_state_publisher ──> /tf
                                           /tf_static
                                                |
                                                v
                                             RViz2
```

- URDF: 링크와 관절의 구조, 고정된 기하 관계를 정의한다.
- `/joint_states`: 관절 이름과 현재 position, velocity, effort를 전달한다.
- `robot_state_publisher`: URDF와 관절 position을 이용해 각 링크의 TF 변환을 계산하고 발행한다.
- RViz2: Robot Description과 TF를 사용해 로봇 모델을 화면에 표시한다.

움직이는 관절의 TF 계산에 직접 필요한 핵심 값은 position이다. velocity와 effort는 JointState에 포함되는 상태 정보지만, RViz가 관절 자세를 계산하는 데 직접 사용되는 값은 아니다.

## 7. 실습 명령과 확인

### 7.1 URDF 파싱 및 링크 확인

ROS 2 Humble 환경에서 URDF 파일 경로를 인자로 전달해 실행했다.

```bash
ros2 run robot_state_publisher robot_state_publisher \
  ~/ros2_ws/src/robot_description/urdf/six_dof_robot.urdf
```

실행 로그에서 다음 8개 segment가 인식되는 것을 확인했다.

- base_link
- link_1
- link_2
- link_3
- link_4
- link_5
- link_6
- tool0

실행 시 `No robot_description parameter` 경고가 출력되었다. 이는 해당 실행 방식에서 파일 경로를 명령행 인자로 전달하는 구형 호환 경로를 사용했다는 경고이며, URDF 파싱 자체는 성공했다.

### 7.2 Virtual Encoder와 JointState 연결

Lesson 7에서 만든 Virtual Encoder가 실행 중이어야 `/joint_states`에 관절 상태가 발행된다.

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run joint_controller virtual_encoder_node
```

다른 터미널에서 확인:

```bash
ros2 topic echo /joint_states --once
```

메시지의 `name` 배열에 URDF의 관절 이름과 정확히 일치하는 `joint_1`부터 `joint_6`까지 포함되어야 한다.

### 7.3 RViz2 RobotModel 설정

RViz2에서 다음 항목을 확인했다.

1. Global Options의 Fixed Frame을 `base_link`로 설정한다.
2. Displays에서 RobotModel을 추가하거나 활성화한다.
3. RobotModel의 Description Source를 Topic으로 설정한다.
4. Description Topic을 `/robot_description`으로 지정한다.
5. Visual Enabled가 체크되어 있는지 확인한다.

실습 중 Description Topic을 올바르게 지정한 뒤 모델이 표시되는 것을 확인했다.

## 8. TF와 Fixed Joint

이번 모델의 `tool0`는 `link_6`에 fixed joint로 연결된다. Fixed joint는 관절 상태 메시지의 각도 값을 기다리지 않고 부모와 자식 프레임 사이의 고정 변환을 제공할 수 있다.

반면 revolute joint는 현재 position이 있어야 해당 관절의 동적 변환을 계산할 수 있다. Virtual Encoder를 실행하지 않았을 때 `/joint_states`와 `/tf` 확인 명령에서 데이터가 나오지 않았고, TF frame graph에는 `link_6 → tool0` 고정 관계만 확인되었다. Virtual Encoder를 실행한 뒤 RViz에서 base_link를 포함한 로봇 모델이 표시되었다.

이 실험을 통해 URDF 파싱 성공과 동작 중인 관절 상태 공급은 서로 다른 조건임을 확인했다.

## 9. 실습에서 확인한 핵심 구분

| 구분 | 정의/계산 | 자동으로 보장하지 않는 것 |
|---|---|---|
| URDF visual | 화면 표시 형상 | 물리적 움직임 |
| URDF collision | 충돌 계산용 형상 | 자동 충돌 감지/차단 |
| URDF joint limit | 관절 한계 메타데이터 | 입력 상태/명령의 자동 제한 |
| JointState | 관절 상태 데이터 | 실제 센서에서 측정된 값이라는 보장 |
| robot_state_publisher | 관절 상태를 TF로 변환 | 모터 제어 및 물리 시뮬레이션 |
| RViz2 | 모델과 TF 시각화 | 접촉력, 중력, 관성 계산 |

## 10. 다음 단계

다음 Lesson 9에서는 Forward Kinematics를 학습한다. 각 관절의 변환 행렬을 순서대로 곱해 base frame에서 tool frame까지의 위치와 방향을 계산하고, RViz에서 보이는 자세 변화와 수학적 계산을 연결한다.

그 이후 Joint Limit 및 State Validation을 구현하고, MuJoCo 물리 시뮬레이션과 ROS 2 연동으로 확장한다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
