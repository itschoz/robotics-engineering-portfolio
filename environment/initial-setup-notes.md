---
layout: page
title: Environment Notes
permalink: /environment-notes/
---

# WSL2 + ROS 2 Humble Environment Notes

이 페이지는 초기 환경 구축 요약을 정리한 것이다. 상세 내역은 [개발 환경 페이지]({{ '/environment/' | relative_url }})를 참고한다.

- Host: Windows 11 (GTX 1050 Ti)
- Linux: WSL2 + Ubuntu 22.04 LTS
- ROS: ROS 2 Humble Desktop
- Workspace: `~/ros2_ws`
- Build tools: `colcon`, `rosdep`
- Git: GitHub SSH 인증 완료

Talker 실행과 초기 `colcon build`까지 확인했다. MuJoCo와 STM32는 이 문서 작성 시점에 향후 설치·연동 대상으로 분류한다.
