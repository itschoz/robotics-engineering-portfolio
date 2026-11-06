---
layout: post
title: "ROS 2 학습 기록 04 — Workspace, Package, Node와 colcon 빌드"
date: 2026-11-07 00:15:00 +0900
categories: [ros2, fundamentals]
tags: [ROS2, Humble, Workspace, Package, Node, colcon, ament_python, WSL2]
excerpt: "ROS 2 Workspace와 Package의 구조를 이해하고 Python 패키지 joint_controller를 생성·빌드해 install 공간에 반영했다."
---

## 1. 학습 목표

이번 학습에서는 ROS 2 프로젝트의 기본 구성 단위인 Workspace, Package, Node의 관계를 이해하고, 직접 Python 패키지를 생성해 빌드한다.

이번 단계의 실습 목표는 다음과 같다.

- WSL2 Ubuntu 환경에서 ROS 2 Workspace 위치 확인
- `ament_python` 기반 패키지 `joint_controller` 생성
- `colcon build`로 패키지 빌드
- Workspace의 설치 환경을 현재 터미널에 적용
- ROS 2 CLI에서 패키지가 인식되는지 확인

## 2. Workspace, Package, Node

| 개념 | 의미 | 이번 프로젝트의 예 |
|---|---|---|
| Workspace | ROS 2 패키지들을 개발·빌드하는 작업 공간 | `ros2_ws` |
| Package | 노드, 인터페이스, 설정, 의존성 등을 묶는 배포 단위 | `joint_controller` |
| Node | 실행 중인 ROS 2 프로그램의 기능 단위 | 향후 `controller_node`, `state_monitor_node` |

로봇 관절 제어 프로젝트는 기능을 패키지 단위로 나누어 구성할 예정이다.

```text
ros2_ws/
└── src/
    ├── joint_controller/
    │   ├── controller_node.py
    │   └── state_monitor_node.py
    ├── joint_interfaces/
    └── hardware_bridge/
```

- `joint_controller`: 관절 명령 생성 및 상태 모니터링
- `joint_interfaces`: 노드 간 통신에 사용할 사용자 정의 인터페이스
- `hardware_bridge`: ROS 2와 STM32 사이의 통신 담당

위 구조는 향후 프로젝트를 위한 설계 방향이다. 이번 실습에서 실제 생성한 패키지는 `joint_controller` 하나다.

## 3. Workspace 디렉터리 구조 확인

현재 Workspace는 `/home/yongbum/ros2_ws`이며, 기본 구조는 다음과 같다.

```text
ros2_ws/
├── build/    # 빌드 중간 결과물
├── install/  # 빌드된 패키지의 설치 결과
├── log/      # 빌드 로그
└── src/      # 개발자가 작성하는 소스 코드
```

일반적으로 하나의 Workspace에는 하나의 `build/`, `install/`, `log/` 디렉터리가 있고, 그 안에서 여러 패키지를 관리한다. 실제 설치 디렉터리의 세부 구조는 빌드 옵션에 따라 달라질 수 있다.

## 4. Python 패키지 생성

ROS 2의 Python 패키지는 `ament_python` 빌드 타입으로 생성할 수 있다.

실습 명령:

```bash
cd ~/ros2_ws/src

ros2 pkg create --build-type ament_python --license Apache-2.0 joint_controller --dependencies rclpy std_msgs
```

여기서 패키지 이름을 `--dependencies` 앞에 배치했다. `--dependencies`는 뒤에 오는 의존성 이름들을 목록으로 받기 때문에 패키지 이름을 먼저 지정하면 인자 해석이 명확해진다.

생성된 기본 구조:

```text
joint_controller/
├── LICENSE
├── package.xml
├── setup.py
├── setup.cfg
├── resource/
├── joint_controller/
└── test/
```

주요 항목:

- `package.xml`: 패키지 메타데이터와 의존성 선언
- `setup.py`: Python 패키지 설치 및 실행 진입점 설정
- `setup.cfg`: 설치 관련 설정
- `resource/joint_controller`: ament 인덱스에서 패키지를 식별하기 위한 리소스 마커
- 내부 `joint_controller/`: Python 모듈과 노드 코드를 둘 디렉터리
- `test/`: 테스트 코드

## 5. colcon으로 빌드하고 설치 공간 구성하기

Workspace 루트로 이동한 뒤 패키지를 빌드한다.

```bash
cd ~/ros2_ws
colcon build --packages-select joint_controller
```

`colcon build`는 단순히 설치 디렉터리를 만드는 명령이 아니다. 패키지의 빌드 작업을 수행하고, 빌드 결과를 Workspace의 install 공간에 반영한다. 변경되지 않은 모든 패키지를 무조건 처음부터 다시 빌드하는 것은 아니며, 선택한 패키지와 필요한 의존성 및 변경 사항을 기준으로 작업한다.

기존 패키지의 Python 코드를 수정하는 경우에도 패키지를 다시 생성할 필요는 없다. 필요한 경우 다시 빌드하면 된다. 개발 중에는 `--symlink-install` 옵션을 활용해 소스 파일과 설치 공간을 심볼릭 링크로 연결하는 방식도 사용할 수 있다.

## 6. setup.bash와 source의 역할

빌드가 끝나면 Workspace의 `install/` 아래에 환경 설정 스크립트가 생성된다. 대표적으로 `setup.bash`와 `local_setup.bash`가 있다.

`setup.bash`는 직접 작성한 파일이 아니라 빌드 시스템이 생성한 환경 설정 스크립트다. 이 스크립트는 해당 Workspace의 설치 결과를 현재 셸 환경에서 사용할 수 있도록 필요한 환경 변수를 설정한다.

```bash
source ~/ros2_ws/install/setup.bash
```

여기서 `source`는 스크립트를 별도 프로세스로 실행하는 대신 현재 Bash 세션에 환경 설정을 적용한다. 따라서 해당 터미널에서 ROS 2 CLI가 Workspace에 설치된 패키지를 검색하고 실행할 수 있게 된다.

ROS 2 기본 환경과 Workspace 환경은 구분된다.

```bash
# ROS 2 Humble 기본 환경
source /opt/ros/humble/setup.bash

# 현재 Workspace 환경
source ~/ros2_ws/install/setup.bash
```

기본 ROS 2 환경은 `/opt/ros/humble`에 설치된 ROS 2 패키지와 도구를 제공하고, Workspace 환경은 우리가 빌드한 패키지를 추가로 사용할 수 있게 한다. Workspace의 `setup.bash`는 기반 환경을 함께 연결하는 용도로 사용된다.

## 7. Terminal과 Shell의 차이

개발 중 자주 사용하는 용어를 구분한다.

- **Terminal:** 명령어를 입력하고 출력 결과를 확인하는 인터페이스 프로그램
- **Shell (Bash):** 터미널에서 입력한 명령어를 해석하고 실행하는 프로그램
- **Process:** 실제 운영체제에서 실행 중인 프로그램

VS Code에서 터미널을 여러 개 열면 각각 별도의 Bash 세션이 만들어진다. 각 세션의 현재 디렉터리와 환경 변수는 독립적이다.

따라서 한 터미널에서 `source ~/ros2_ws/install/setup.bash`를 실행해도 다른 터미널에 자동으로 적용되지 않는다. 다른 터미널에서도 Workspace 패키지를 사용하려면 해당 세션에서 환경을 적용해야 한다.

## 8. 실습 결과

이번 실습에서 확인한 사항:

- Workspace 경로: `/home/yongbum/ros2_ws`
- `src/` 디렉터리가 비어 있는 상태에서 시작
- `ament_python` 타입의 `joint_controller` 패키지 생성
- 패키지 기본 파일(`package.xml`, `setup.py`, `setup.cfg` 등) 생성 확인
- `colcon build --packages-select joint_controller` 실행 완료
- `source install/setup.bash` 실행
- `ros2 pkg list | grep joint_controller`로 패키지 인식 확인

**검증 상태: 패키지 생성 및 빌드 성공.** 이번 단계에서는 아직 ROS 2 Node의 코드를 작성하거나 실행하지 않았다.

## 9. 개발 사이클 정리

```text
1. src/ 아래에서 소스 코드 작성 또는 수정
             |
             v
2. colcon build
   필요한 패키지 빌드 및 install 공간에 결과 반영
             |
             v
3. source install/setup.bash
   현재 터미널에 Workspace 환경 적용
             |
             v
4. ros2 run / ros2 launch
   Node 또는 시스템 실행
             |
             v
5. 결과 확인, 디버깅, 개선
             |
             +------> 다시 1번으로
```

Build, Install, Run, Deploy는 서로 다른 개념이다.

- **Build:** 소스 코드의 빌드 작업 수행
- **Install:** 빌드 결과를 사용할 수 있는 설치 공간에 반영
- **Run:** 프로그램 실행
- **Deploy:** 실제 운영 장비나 로봇 환경에 배포

이번 실습은 개발 Workspace에서 빌드하고 설치한 단계이며, 실제 로봇에 배포한 것은 아니다.

## 10. 다음 학습

다음 단계에서는 `joint_controller` 패키지 안에 `controller_node.py`를 작성하고, `setup.py`의 console entry point를 설정해 ROS 2 CLI로 직접 실행한다.

---

*실습 환경: Windows 11 + WSL2 Ubuntu 22.04 + ROS 2 Humble*
