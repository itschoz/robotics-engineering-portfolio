---
layout: post
title: "ROS 2 질문 노트 05 — Node 실행 오류와 Float64 소수점 출력"
date: 2026-11-07 23:56:00 +0900
categories: [ros2, learning-notes]
tags: [ROS2, Humble, Python, Node, Float64, FloatingPoint]
excerpt: "controller_node의 main 속성 오류 관측과 Float64 값의 부동소수점 표현, Topic 주기 측정 결과를 정리한다."
---

이 글은 첫 번째 ROS 2 Python Node와 Publisher 실습 중 관찰한 문제 및 출력 결과를 강의 본문과 분리해 기록한다.

## Q1. `ros2 run joint_controller controller_node`에서 `module ... has no attribute 'main'` 오류가 발생했다가 재실행하니 정상 동작한 이유는?

**관찰한 현상**

첫 실행에서는 다음 오류가 발생했다.

```text
AttributeError: module 'joint_controller.controller_node' has no attribute 'main'
```

이후 다시 실행했을 때는 다음 로그가 출력되며 Node가 시작됐다.

```text
[INFO] [controller_node]: Joint Controller Node has started!
```

**답변 및 진단 원칙**

이 기록만으로 최초 오류의 정확한 원인을 확정할 수는 없다. 이런 경우에는 다음 항목을 순서대로 확인하는 것이 좋다.

1. `controller_node.py` 안에 `def main(args=None):` 함수가 모듈 최상위 수준에 정의되어 있는지 확인한다.
2. `setup.py`의 console script가 `controller_node = joint_controller.controller_node:main`으로 정확히 연결되어 있는지 확인한다.
3. 패키지 빌드와 현재 터미널의 workspace overlay source 상태를 확인한다.
4. 실행 중인 Python 모듈이 예상한 workspace의 패키지인지 확인한다.

재실행 성공만으로 특정 원인이 해결됐다고 단정하지 않는다. 재발하면 위 항목을 확인해 원인을 좁힌다.

## Q2. Topic에서 `14.199999999999966`처럼 소수점 아래 숫자가 길게 보이는 이유는?

**답변**

이는 `Float64` 메시지나 ROS 2의 통신 오류가 아니라 IEEE 754 부동소수점 표현 특성이다. 0.1과 같은 일부 10진 소수는 이진수로 정확히 표현되지 않기 때문에 반복 덧셈 결과가 근삿값으로 나타날 수 있다.

로그 표시 자릿수는 다음처럼 제한할 수 있다.

```python
self.get_logger().info(f'{msg.data:.2f}')
```

표시만 반올림하는 것과 내부 계산값을 정수 단위로 설계하는 것은 서로 다른 문제다. 정밀도가 중요한 제어 로직에서는 단위와 수치 표현 방식을 의도적으로 선택해야 한다.

## Q3. Timer가 0.5초인데 `ros2 topic hz`에서 2.000Hz가 나오는 것이 맞을까?

**답변**

맞다. 주파수는 주기의 역수다.

`f = 1 / T = 1 / 0.5 = 2 Hz`

실습에서 관측한 결과:

- 평균 발행 주파수: 2.000~2.001Hz
- 메시지 간격: 0.497~0.501초
- 표준편차: 약 0.0012~0.0015초

측정 결과는 설정한 0.5초 Timer와 잘 일치했다. 다만 `ros2 topic hz`는 수신 측에서 관측한 주기를 측정하는 도구이며, 일반 OS 환경에서 하드 실시간 보장을 입증하는 시험은 아니다.

## 이번 Q&A의 핵심

- Python console script의 진입점은 `setup.py`와 실제 모듈의 `main()` 정의가 일치해야 한다.
- 최초 오류의 원인을 모를 때는 추측하지 말고 파일, entry point, 빌드, source 환경을 확인한다.
- Float64의 긴 소수점 출력은 부동소수점 표현 오차일 수 있다.
- 0.5초 주기는 2Hz에 해당하며, `ros2 topic hz`로 실제 관측 주기를 확인할 수 있다.
