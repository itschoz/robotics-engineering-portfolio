---
layout: home
title: Home
---

# Robotics Engineering Portfolio

ROS 2, simulation, middleware, and embedded-system integration을 학습하고 직접 구현한 내용을 기록합니다.

이 블로그는 단순한 학습 일지가 아니라, 문제 정의부터 실험·검증·개선까지의 엔지니어링 과정을 보여주는 포트폴리오를 목표로 합니다.

## Learning Journal

강의별 개념과 실습 결과를 정리합니다. 강의 본문과 학습 중 발생한 질문·답변은 별도 글로 관리합니다.

| 강의 | 주제 | 진행 상태 |
|---|---|---|
| 01 | [ROS 1과 ROS 2의 통신 구조, DDS와 QoS](ros2/fundamentals/2026/11/06/ros2-communication-dds-qos/) | 게시 완료 |
| 02 | [ROS 2 Topic, Service, Action](ros2/fundamentals/2026/11/07/ros2-topic-service-action/) | 게시 완료 · 실습 완료 |
| 04 | [Workspace, Package, Node와 colcon 빌드](ros2/fundamentals/2026/11/07/ros2-workspace-package-node/) | 게시 완료 |
| 05 | [ROS 2 첫 Node와 Timer/Publisher 구현](ros2/fundamentals/2026/11/07/ros2-first-node-timer-publisher/) | 게시 완료 · 실습 및 2Hz 발행 검증 완료 |
| 06 | [Publisher/Subscriber 통신과 QoS 호환성 검증](ros2/fundamentals/2026/11/09/ros2-publisher-subscriber-qos/) | 게시 완료 · 두 Node 통신 및 Reliability 비호환 검증 완료 |

## Learning Q&A

강의 중 사용자가 직접 질문한 내용을 주제별 제목으로 정리합니다. 강의 본문과 분리해 개념을 다시 찾아보기 쉽게 구성합니다.

- [Fibonacci Action 실행 파일은 어느 패키지에 있을까? — ROS 2 질문 노트 02](ros2/learning-notes/2026/11/07/ros2-topic-service-action-qa/)
- [install/setup.bash, 빌드와 터미널 환경의 관계 — ROS 2 질문 노트 04](ros2/learning-notes/2026/11/07/ros2-workspace-build-qa/)
- [Node 실행 오류와 Float64 소수점 출력 — ROS 2 질문 노트 05](ros2/learning-notes/2026/11/07/ros2-first-node-timer-publisher-qa/)
- [QoSProfile 설정 위치와 Reliability 호환성 — ROS 2 질문 노트 06](ros2/learning-notes/2026/11/09/ros2-publisher-subscriber-qos-qa/)

## Technical References

- [Robotics Dictionary — 100 terms](glossary/)
- [ROS 2 Communication Reference](reference/ros2-communication/)
- [ROS 2 Package & CLI Catalog](reference/ros2-packages-cli/)

## 현재 학습 주제

- ROS 2 architecture and communication
- DDS discovery and QoS
- Robot description, TF2, and simulation
- ROS 2 and STM32 integration

## Main Project

**ROS 2 & Embedded Based Robotic Joint Control System**

ROS 2 기반 PC 애플리케이션과 MuJoCo 시뮬레이션 및 STM32 하드웨어를 연결하는 관절 제어 시스템을 구현하는 것을 목표로 합니다.
