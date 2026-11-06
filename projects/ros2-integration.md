---
layout: page
title: ROS 2 Integration
permalink: /projects/ros2-integration/
---

# ROS 2 Integration

<span class="status progress">In Progress · Middleware Foundations</span>

## Current verified foundations

- ROS 2 Humble on WSL2 Ubuntu 22.04
- ROS 1과 ROS 2 통신 구조 비교
- DDS Discovery 및 QoS 정책 이해
- QoS 호환성 실습 완료
- Action tutorial의 Fibonacci Server / Client 실행 성공

## Planned integration path

1. Topic / Service / Action을 실제 명령·상태 인터페이스에 매핑
2. ROS 2 Node를 Python 또는 C++로 직접 구현
3. MuJoCo simulation backend 연결
4. UART Serial Bridge를 통해 STM32 backend 연결
5. 통신 주기, latency, timeout, error handling 검증

관련 문서: [ROS 2 Communication Reference]({{ '/reference/ros2-communication/' | relative_url }}) · [Package & CLI Catalog]({{ '/reference/ros2-packages-cli/' | relative_url }})
