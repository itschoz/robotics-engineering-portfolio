---
layout: page
title: ROS 2 Package & CLI Catalog
permalink: /reference/ros2-packages-cli/
---

# ROS 2 Package & CLI Catalog

강의와 실습에서 실제로 사용한 ROS 2 패키지, 실행 파일, 명령어를 기록한다. 설치 여부와 실행 검증 여부를 구분한다.

## 1. 개발 환경

| 항목 | 현재 환경 |
|---|---|
| Host OS | Windows 11 |
| Linux 환경 | WSL2 / Ubuntu 22.04 |
| ROS 2 Distribution | Humble |
| ROS 설치 경로 | `/opt/ros/humble` |
| Workspace | `~/ros2_ws` |

환경 확인:
```bash
echo $ROS_DISTRO
ros2 --version
ros2 doctor --report
```

현재 셸에서 ROS 2 환경 불러오기:
```bash
source /opt/ros/humble/setup.bash
```

## 2. 사용 패키지 목록

| Package | 역할 | 사용 실행 파일 / 인터페이스 | 확인 상태 |
|---|---|---|---|
| `demo_nodes_cpp` | ROS 2 C++ 데모 노드 | `talker`, `add_two_ints_server` 등 | 설치 및 실행 파일 목록 확인 |
| `demo_nodes_py` | ROS 2 Python 데모 노드 | `listener` | 실습 명령 안내 |
| `action_tutorials_cpp` | C++ Action 튜토리얼 서버·클라이언트 | `fibonacci_action_server`, `fibonacci_action_client` | 설치 후 Action 실행 성공 |
| `action_tutorials_interfaces` | Action 튜토리얼 인터페이스 정의 | `action_tutorials_interfaces/action/Fibonacci` | Action 실행에서 사용 |
| `example_interfaces` | ROS 2 기본 예제 인터페이스 | `example_interfaces/srv/AddTwoInts` | Service 예제에서 사용 예정 |

## 3. 패키지 탐색 명령어

```bash
# 패키지 설치 위치
ros2 pkg prefix demo_nodes_cpp

# 패키지 내 실행 파일 목록
ros2 pkg executables demo_nodes_cpp

# 패키지 목록
ros2 pkg list

# 특정 패키지 필터링
ros2 pkg list | grep action_tutorials
```

중요한 차이:
- `ros2 pkg prefix`는 패키지가 설치된 Prefix를 보여준다.
- `ros2 pkg executables`는 해당 패키지에서 ROS 2 CLI로 실행 가능한 실행 파일 목록을 보여준다.
- 패키지가 설치되어 있어도 특정 실행 파일이 반드시 포함된 것은 아니다.

## 4. 실습 명령어 레퍼런스

### Topic
```bash
ros2 run demo_nodes_cpp talker
ros2 run demo_nodes_py listener
ros2 topic list
ros2 topic info /chatter
ros2 topic info /chatter --verbose
ros2 topic echo /chatter
ros2 topic hz /chatter
```

### Service
```bash
ros2 run demo_nodes_cpp add_two_ints_server
ros2 service list
ros2 service type /add_two_ints
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 10, b: 20}"
```

### Action
```bash
# 설치
sudo apt update
sudo apt install ros-humble-action-tutorials-cpp

# 실행 파일 확인
ros2 pkg executables action_tutorials_cpp

# Server 실행
ros2 run action_tutorials_cpp fibonacci_action_server

# Action 탐색 및 Goal 전송
ros2 action list
ros2 action info /fibonacci
ros2 action send_goal /fibonacci action_tutorials_interfaces/action/Fibonacci "{order: 10}" --feedback
```

## 5. 설치 오류 / 실행 파일 오류 트러블슈팅

### `No executable found`
예시:
```text
No executable found
```

확인 순서:
```bash
echo $ROS_DISTRO
ros2 pkg prefix demo_nodes_cpp
ros2 pkg executables demo_nodes_cpp
dpkg -l | grep ros-humble-demo-nodes-cpp
```

해석:
- `ros2 pkg prefix`가 경로를 반환하면 패키지를 찾은 것이다.
- 실행 파일 목록에 원하는 이름이 없다면, 해당 패키지에 그 실행 파일이 설치되지 않았거나 실행 파일 이름이 다른 것이다.
- 패키지 전체를 재설치하기 전에 올바른 튜토리얼 패키지와 실행 파일 이름을 확인한다.

이번 실습의 사례:
- `demo_nodes_cpp`는 정상 설치되어 있었음.
- `fibonacci_action_server`는 해당 패키지의 실행 파일 목록에 없었음.
- Action 튜토리얼용 `action_tutorials_cpp` 패키지를 설치해 해결했음.

## 6. 기록 규칙

각 새 패키지는 다음 정보를 함께 기록한다.
1. 패키지 이름과 역할
2. 설치 명령
3. 실행 파일 / 인터페이스 이름
4. 재현 가능한 실행 명령
5. 실행 결과와 검증 여부
6. 메인 프로젝트에서의 활용 후보

명령어를 안내받았다는 이유만으로 설치 또는 실행 완료로 표시하지 않는다.
