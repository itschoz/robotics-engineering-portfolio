---
layout: post
title: "ROS 2 질문 노트 08 — URDF와 JointState, TF 및 RViz 표시 문제"
date: 2026-11-15 23:01:00 +0900
categories: [ros2, learning-notes]
tags: [ROS2, URDF, JointState, TF2, RViz2, robot_state_publisher, Collision, JointLimit]
excerpt: "URDF를 만들었는데 로봇이 보이지 않는 이유, JointState와 TF의 관계, collision geometry와 joint limit의 역할을 정리한다."
---

Lesson 8에서 6-DOF URDF 모델을 RViz2에 표시하는 과정에서 나온 질문과 답변을 정리한다.

## Q1. URDF를 만들었는데 왜 RViz에서 로봇이 보이지 않았을까?

URDF 파일을 만들었다고 해서 현재 관절 자세까지 자동으로 정해지는 것은 아니다.

URDF는 링크와 관절의 구조를 정의하고, `/joint_states`는 각 관절의 현재 상태를 전달한다. `robot_state_publisher`는 이 두 정보를 사용해 링크 사이의 TF 변환을 계산한다.

이번 실습에서는 Virtual Encoder를 실행하지 않아 `/joint_states` 데이터가 공급되지 않았다. 그 결과 revolute joint의 현재 각도를 알 수 없어 동적 TF를 계산할 수 없었다. Virtual Encoder를 실행한 뒤 로봇 모델이 RViz에 표시되었다.

## Q2. JointState의 position, velocity, effort가 모두 있어야 RViz가 움직임을 표시할까?

아니다. 움직이는 관절의 현재 자세를 계산하는 데 직접 필요한 핵심 값은 `position`이다.

- `position`: 관절 위치/각도. TF 계산에 사용된다.
- `velocity`: 관절 속도/각속도. 상태 정보로 전달된다.
- `effort`: 관절 힘/토크. 상태 정보로 전달된다.

따라서 position이 유효하게 공급되면 robot_state_publisher는 관절의 현재 자세를 계산할 수 있다. velocity와 effort가 없더라도 TF 자세 계산 자체는 가능하다.

## Q3. URDF에 collision을 정의했는데 왜 로봇 링크가 서로 관통할까?

URDF의 `collision`은 충돌 검사에 사용할 기하 형상을 정의하는 요소다. 하지만 형상을 정의하는 것과 충돌을 계산하는 것은 다르다.

RViz는 주로 모델의 시각화와 TF 표시를 담당한다. URDF에 collision geometry가 있어도 RViz가 물리 엔진처럼 접촉을 계산하거나 관절 움직임을 차단하지 않는다.

충돌을 검출하거나 접촉력으로 움직임을 제한하려면 별도의 충돌 검사기 또는 물리 시뮬레이터가 필요하다. 이번 학습에서는 이후 MuJoCo를 이용해 질량, 관성, 중력 및 충돌을 다룰 계획이다.

## Q4. URDF의 velocity와 effort limit은 왜 자동으로 적용되지 않을까?

URDF의 `<limit>`은 관절의 한계 정보를 모델에 기록한다.

```xml
<limit lower="-3.14" upper="3.14" effort="10" velocity="1.0"/>
```

그러나 `robot_state_publisher`는 제어기가 아니다. 들어오는 JointState를 검증해 속도를 제한하거나 위치를 자동으로 잘라내는 역할을 하지 않는다.

실제 시스템에서는 controller나 hardware driver, trajectory validation 및 safety layer에서 명령과 상태의 한계를 검사해야 한다.

## Q5. RobotModel Status가 OK인데 화면에 모델이 안 보이면 무엇을 확인해야 할까?

RViz의 RobotModel 표시 상태가 정상이어도 Description Source와 Description Topic이 올바르게 설정되지 않으면 모델 설명을 제대로 가져오지 못할 수 있다.

이번 실습에서는 RobotModel의 Description Source를 Topic으로 두고, Description Topic을 `/robot_description`으로 지정한 뒤 모델이 표시되는 것을 확인했다.

추가로 다음 항목을 확인한다.

- Global Options의 Fixed Frame이 실제 TF 트리에 존재하는가?
- RobotModel이 Enabled 상태인가?
- Visual Enabled가 체크되어 있는가?
- Description Topic이 실제로 발행되는 토픽과 일치하는가?
- RViz 카메라가 모델을 향하고 있는가?

## Q6. URDF의 origin과 axis는 어떤 좌표계를 기준으로 할까?

Joint의 `origin xyz/rpy`는 parent link frame을 기준으로 관절 프레임의 위치와 방향을 정의한다.

`axis xyz`는 joint frame에서 표현한 관절의 움직임 축이다. 따라서 부모 링크가 회전하면 자식 관절의 base/world 기준 축 방향도 부모 프레임의 회전 영향을 받는다. URDF에 적힌 local axis 값 자체가 바뀌는 것은 아니다.

## 핵심 정리

- URDF는 로봇의 구조를 정의하고 JointState는 현재 관절 상태를 전달한다.
- robot_state_publisher는 URDF와 관절 position을 사용해 TF를 계산한다.
- RViz는 물리 엔진이 아니므로 collision geometry만으로 충돌을 막지 않는다.
- URDF joint limit은 정의된 한계 정보이며, robot_state_publisher가 이를 자동으로 강제하지 않는다.
- RViz RobotModel은 Description Source/Topic, Fixed Frame, Enabled 상태를 함께 확인해야 한다.
- Joint origin은 parent link 기준이고 axis는 joint frame 기준이다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
