---
layout: page
title: Development Environment
permalink: /environment/
---

# 개발 환경 구축 기록

이 문서는 2026-11-05 기준으로 정리한 개발 환경의 초기 상태다. 설치 완료와 향후 계획을 구분한다.

## PC 및 개발 환경

| 항목 | 구성 | 상태 |
|---|---|---|
| Host OS | Windows 11 | 사용 중 |
| Linux | WSL2 Ubuntu 22.04 LTS | 설치 완료 |
| ROS | ROS 2 Humble Desktop | 설치 완료 |
| Build | colcon, rosdep | 설치 및 초기 설정 완료 |
| Workspace | `~/ros2_ws` | 디렉터리 구성 및 초기 빌드 확인 |
| Version Control | Git / GitHub SSH | 인증 확인 |
| GPU | NVIDIA GTX 1050 Ti | 호스트 PC 사양 |

## 확인한 내용

- ROS 2 Humble 설치 완료
- `ros2 run demo_nodes_cpp talker` 실행으로 기본 Publisher 동작 확인
- 별도 터미널에서 Node 및 Topic 목록 확인
- `python3-colcon-common-extensions`와 `rosdep` 설치
- `rosdep init`, `rosdep update` 수행
- `~/ros2_ws` 작업 공간 생성 및 `colcon build` 성공
- `~/.bashrc`에서 ROS 2와 작업 공간 환경을 로드하도록 설정
- GitHub SSH 인증 성공

## 환경 구조

```text
Windows 11 Host
└── WSL2
    └── Ubuntu 22.04 LTS
        ├── ROS 2 Humble Desktop
        ├── colcon / rosdep
        ├── ~/ros2_ws
        └── GitHub SSH
```

## 다음 환경 구성

1. Python MuJoCo 설치 및 기본 모델 실행
2. ROS 2와 MuJoCo 사이 최소 통신 루프 구성
3. URDF / Xacro / TF2 / RViz2 도구 체인 정리
4. STM32CubeIDE 및 Nucleo-F446RE 개발 환경 준비
5. WSL2에서 보드 USB 연결 및 Serial 통신 확인

## 재현성 기록 규칙

새 패키지를 추가할 때마다 설치 명령, 버전, 환경 변수, 실행 테스트를 기록한다. 설치를 시도한 상태와 정상 실행을 검증한 상태를 구별한다.
