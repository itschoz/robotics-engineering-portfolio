---
layout: post
title: "ROS 2 질문 노트 04 — install/setup.bash, 빌드와 터미널 환경의 관계"
date: 2026-11-07 00:16:00 +0900
categories: [ros2, learning-notes]
tags: [ROS2, Humble, colcon, setup-bash, Bash, Terminal, Workspace]
excerpt: "Workspace마다 install 공간이 있는지, setup.bash는 누가 만드는지, 기존 패키지 수정과 터미널별 환경 적용은 어떻게 동작하는지 정리한다."
---

이 글은 Workspace와 Package를 실습하면서 나온 질문과 답변을 강의 본문과 분리해 정리한 Q&A 기록이다.

## Q1. Workspace마다 install 디렉터리가 하나씩 있는 건가?

**질문 요지**

ROS 2 Workspace마다 하나의 `install/`이 있고, 그 안에서 여러 패키지를 관리하는 구조인지 확인했다.

**답변**

일반적인 colcon Workspace에서는 하나의 `install/` 디렉터리를 사용하고, 그 아래에 Workspace에 포함된 패키지의 설치 결과가 배치된다.

```text
ros2_ws/
├── src/
│   ├── joint_controller/
│   ├── joint_interfaces/
│   └── hardware_bridge/
├── build/
├── install/
│   ├── setup.bash
│   ├── local_setup.bash
│   └── ...
└── log/
```

패키지마다 별도의 Workspace 수준 `install/`을 만드는 것이 아니다. 다만 `--merge-install` 같은 빌드 옵션에 따라 설치 공간 내부의 세부 구조는 달라질 수 있다.

## Q2. setup.bash는 무엇이고, 우리가 직접 만든 파일인가?

**질문 요지**

`source install/setup.bash`를 실행할 때 어떤 기준으로 패키지 환경을 적용하는지, `setup.bash` 파일은 누가 만들었는지 궁금했다.

**답변**

`setup.bash`는 개발자가 직접 작성하는 파일이 아니다. `colcon build`를 통해 Workspace의 install 공간이 구성될 때 생성되는 환경 설정 스크립트다.

```bash
source ~/ros2_ws/install/setup.bash
```

이 명령은 현재 Bash 세션에 Workspace 환경을 적용한다. ROS 2 패키지 검색 경로, Python 모듈 검색 경로, 실행 파일 및 관련 리소스 경로 등이 환경에 반영된다.

ROS 2 기본 설치 환경은 별도로 존재한다.

```bash
source /opt/ros/humble/setup.bash
```

Workspace의 `install/setup.bash`는 현재 Workspace의 설치 결과를 기본 ROS 2 환경 위에 연결해 사용할 수 있도록 한다.

## Q3. 기존 패키지의 코드를 수정하면 설치 공간을 다시 생성해야 하나?

**질문 요지**

소스 코드를 수정할 때마다 설치 공간을 새로 만들어야 하는지, 이미 생성된 패키지라면 빌드를 생략할 수 있는지 물었다.

**답변**

패키지를 처음부터 다시 생성할 필요는 없다. 기존 패키지의 코드를 수정한 뒤 필요한 빌드 작업을 다시 수행하면 된다.

```bash
cd ~/ros2_ws
colcon build --packages-select joint_controller
```

`colcon build`는 패키지 빌드와 설치 결과 반영을 수행한다. 변경 사항과 의존성에 따라 필요한 작업을 수행하므로 매번 모든 패키지를 처음부터 다시 빌드하는 것은 아니다.

Python 패키지는 개발 방식에 따라 `--symlink-install`을 활용할 수도 있다. 이 옵션은 소스 파일과 설치 공간을 심볼릭 링크로 연결해 개발 중 반복 작업을 줄이는 데 유용하다. 다만 모든 변경 사항에서 빌드가 불필요해지는 것은 아니며, 패키지 설정이나 entry point 등의 변경은 다시 빌드해야 한다.

## Q4. 터미널은 무엇이고, 터미널마다 환경이 다른가?

**질문 요지**

"현재 작업을 지시 중인 터미널"이라는 표현에서 터미널이 정확히 무엇인지, 터미널이 여러 개일 때 환경 설정이 어떻게 적용되는지 질문했다.

**답변**

터미널은 명령어를 입력하고 결과를 확인하는 인터페이스 프로그램이다. 그 안에서 실제 명령어를 해석하는 프로그램은 Shell이며, 현재 환경에서는 Bash를 사용한다.

- Terminal: 명령어 입력·출력 인터페이스
- Shell (Bash): 명령어 해석 및 실행
- Process: 운영체제에서 실행되는 프로그램

VS Code에서 터미널을 여러 개 열면 각각 독립된 Bash 세션이 만들어진다. 각 세션은 환경 변수를 별도로 가진다.

예를 들어 터미널 1에서 다음 명령을 실행해도,

```bash
source ~/ros2_ws/install/setup.bash
```

터미널 2에 자동 적용되지는 않는다. 터미널 2에서도 Workspace 패키지를 사용하려면 동일하게 환경을 적용해야 한다.

## Q5. 소스 코드 수정부터 실제 로봇 배포까지의 흐름은?

**질문 요지**

로봇을 계속 개선하기 위해 소스 코드를 작성하고 배포하는 전체 개발 프로세스가 다음과 같은지 확인했다.

1. 소스 코드 작성 및 수정
2. 패키지 설치 공간 생성
3. 현재 터미널에 패키지 위치와 정보 적용
4. 패키지 실행

**답변**

큰 흐름은 맞다. 다만 2단계는 단순히 설치 공간만 만드는 것이 아니라, 빌드와 설치 결과 반영을 함께 수행하는 단계로 이해하는 것이 정확하다.

```text
소스 코드 작성 / 수정
        |
        v
colcon build
(필요한 빌드 작업 + install 공간에 결과 반영)
        |
        v
source install/setup.bash
(현재 터미널에 Workspace 환경 적용)
        |
        v
ros2 run / ros2 launch
(Node 또는 시스템 실행)
        |
        v
결과 확인 / 디버깅 / 개선
        |
        +----> 소스 코드 수정으로 반복
```

그리고 Build, Install, Run, Deploy는 구분해야 한다.

- Build: 소스 코드의 빌드 작업
- Install: 빌드 결과를 설치 공간에 반영
- Run: 프로그램 실행
- Deploy: 실제 운영 장비나 로봇에 배포

현재까지 수행한 것은 개발 PC의 Workspace에서 패키지를 빌드하고 설치한 단계다. 실제 로봇에 배포한 것은 아니다.

---

## 이번 Q&A에서 정리한 핵심

1. 하나의 Workspace에는 일반적으로 하나의 `install/` 디렉터리가 있다.
2. `setup.bash`는 빌드 시스템이 생성하는 환경 설정 스크립트다.
3. 기존 패키지를 수정할 때는 패키지를 재생성하지 않고 필요한 빌드를 다시 수행한다.
4. `source`는 현재 Bash 세션에 환경을 적용한다.
5. 터미널을 여러 개 열면 각 세션에 환경을 별도로 적용해야 한다.
6. Build와 실제 로봇 Deploy는 다른 단계다.
