---
layout: page
title: Robotics Role Map
permalink: /role-map/
---

# 로봇 개발 직무 지도

이 페이지는 사용자가 제공한 '로봇 개발 직무 지도'의 개념을 웹에서 읽기 쉽도록 재구성한 것이다. 하나의 로봇 시스템을 완성하기 위해 어떤 역할이 연결되는지, 각 분야가 어떤 질문에 답하는지 정리한다.

![로봇 개발 직무 지도]({{ '/assets/diagrams/robot-development-role-map.svg' | relative_url }})

## 6개 핵심 직무 영역

| 영역 | 핵심 질문 | 대표 기술 / 도구 |
|---|---|---|
| 01 인지·상태 추정 | 로봇은 주변과 자신의 상태를 어떻게 파악하는가? | OpenCV, PCL, 센서 융합, TF2, EKF |
| 02 애플리케이션 | 로봇은 어떤 순서로 작업을 수행하는가? | C++, Python, ROS 2 API, Action, Behavior Tree |
| 03 동작·경로 계획 | 목적지까지 어떤 경로와 궤적을 선택하는가? | MoveIt 2, OMPL, Nav2, 경로·궤적 계획 |
| 04 로봇 운동 제어 | 목표 동작을 어떻게 안정적으로 실행하는가? | 제어 이론, PID, ros2_control, C++ |
| 05 임베디드·구동 | 명령을 실제 전기 신호와 구동으로 어떻게 바꾸는가? | STM32, C/C++, UART, PWM, Encoder, FreeRTOS |
| 06 기구·구동계 | 힘과 운동을 견디는 구조를 어떻게 설계하는가? | CAD, SolidWorks, Fusion, 해석·검증 |

## 여러 직무를 연결하는 공통 기반

**미들웨어·플랫폼**
- 노드 간 API, 메시지, 서비스, 액션 설계
- ROS 2, DDS, QoS, Linux, 통신 구조

**시뮬레이션·검증**
- 물리 모델과 실제 동작의 차이 분석
- MuJoCo, Gazebo, RViz2, rosbag2, 테스트 자동화

**개발 인프라·CI/CD**
- 코드 변경 검증, 빌드, 테스트, 협업
- Git/GitHub, CMake, colcon, Docker, CI

## 이번 포트폴리오의 집중 영역

이번 6주 프로젝트에서 모든 로봇 직무를 전문가 수준으로 다루려는 것은 아니다. 단기간에 구현 가능한 연결 지점에 집중한다.

1. **ROS 2 Middleware / System Integration:** 통신과 시스템 경계 이해
2. **Simulation & Validation:** MuJoCo 기반 가상 관절 제어 및 검증
3. **Embedded Integration:** STM32 펌웨어와 ROS 2 사이 명령·상태 교환

인지·상태 추정, 고급 경로 계획, 기구 설계는 전체 로봇 시스템을 이해하기 위한 지도에 포함하되 이번 핵심 구현 범위와는 구분한다.
