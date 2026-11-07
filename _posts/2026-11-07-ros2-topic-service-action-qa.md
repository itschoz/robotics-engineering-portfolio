---
layout: post
title: "ROS 2 질문 노트 02 — Fibonacci Action 실행 파일은 어느 패키지에 있을까?"
date: 2026-11-07 09:46:00 +0900
categories: [ros2, learning-notes]
tags: [ROS2, Humble, Action, action_tutorials_cpp, demo_nodes_cpp]
excerpt: "demo_nodes_cpp에서 Fibonacci Action Server를 찾지 못한 문제를 패키지와 실행 파일 목록으로 진단하고 올바른 튜토리얼 패키지로 해결한 과정을 기록한다."
---

이 글은 Topic, Service, Action 학습 중 발생한 실행 파일 탐색 문제와 해결 과정을 강의 본문과 분리해 정리한 Q&A 기록이다.

## Q1. `ros2 run demo_nodes_cpp fibonacci_action_server`를 실행했는데 왜 실행 파일을 찾지 못할까?

**질문 요지**

Fibonacci Action Server를 실행하려고 했지만 `No executable found` 오류가 발생했다. ROS 2 설치가 잘못된 것인지, 실행 파일 이름이 틀린 것인지 확인할 필요가 있었다.

**답변**

먼저 패키지가 실제로 설치되어 있는지와 해당 패키지가 제공하는 실행 파일 목록을 구분해서 확인한다.

```bash
ros2 pkg prefix demo_nodes_cpp
ros2 pkg executables demo_nodes_cpp
```

패키지가 설치되어 있다는 사실과 특정 실행 파일이 그 패키지에 포함되어 있다는 사실은 다르다. 이번 환경에서는 `demo_nodes_cpp` 실행 파일 목록에 `fibonacci_action_server`가 없었다.

## Q2. Fibonacci Action Server는 어떤 패키지에서 실행해야 할까?

**답변**

ROS 2 Humble의 Action 튜토리얼 패키지를 설치한 뒤 해당 패키지에서 실행한다.

```bash
sudo apt update
sudo apt install ros-humble-action-tutorials-cpp
```

실행:

```bash
ros2 run action_tutorials_cpp fibonacci_action_server
```

실행 파일 목록과 Action 이름 확인:

```bash
ros2 pkg executables action_tutorials_cpp
ros2 action list
ros2 action info /fibonacci
```

Goal 전송:

```bash
ros2 action send_goal /fibonacci action_tutorials_interfaces/action/Fibonacci "{order: 10}" --feedback
```

여기서 구분할 패키지는 다음과 같다.

- `action_tutorials_cpp`: Fibonacci Action Server/Client 튜토리얼 실행 파일
- `action_tutorials_interfaces`: Fibonacci Action 인터페이스 정의
- `demo_nodes_cpp`: Talker, AddTwoInts Server 등 별도의 C++ 데모 실행 파일

## 이번 Q&A의 핵심

1. `ros2 run`의 첫 번째 인자는 패키지 이름, 두 번째 인자는 그 패키지가 제공하는 실행 파일 이름이다.
2. 패키지가 설치되어 있어도 원하는 실행 파일이 그 패키지에 포함되어 있지 않을 수 있다.
3. `ros2 pkg executables <package>`로 실행 가능한 항목을 확인하면 패키지 이름과 실행 파일 이름을 혼동하는 문제를 줄일 수 있다.
4. Action 인터페이스 패키지와 Action Server 실행 파일을 제공하는 패키지는 서로 다를 수 있다.

실습에서 확인한 실제 오류 원인은 실행 파일이 `demo_nodes_cpp`에 없었다는 점이다. 설치 손상으로 단정하지 않고, 실행 파일 목록을 확인해 올바른 튜토리얼 패키지로 해결했다.
