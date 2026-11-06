# Robotics Engineering Portfolio

ROS 2, robotics simulation, middleware, and embedded-system integration을 학습하고 구현하는 과정을 기록하는 기술 포트폴리오입니다.

## Start here

- [6주 학습 로드맵](curriculum/6-week-roadmap.md)
- [통합 로봇 시스템 아키텍처](architecture/system-architecture.md)
- [로봇 개발 직무 지도](architecture/robotics-role-map.md)
- [개발 환경 구축 기록](environment/development-environment.md)
- [Robotics Dictionary — 100 terms](glossary/robotics-dictionary.md)
- [ROS 2 Communication Reference](reference/ros2-communication.md)
- [ROS 2 Package & CLI Catalog](reference/ros2-packages-cli.md)
- [초기 환경 구축 메모](environment/initial-setup-notes.md)
- [블로그 글 작성 및 게시 가이드](guides/blog-publishing-guide.md)
- [학습 블로그 글](https://itschoz.github.io/robotics-engineering-portfolio/ros2/fundamentals/2026/11/06/ros2-communication-dds-qos/)

## Main project

**ROS 2 & Embedded Based Robotic Joint Control System**

ROS 2 PC application → control/middleware → interchangeable simulation or hardware backend → MuJoCo / STM32 → actuator and feedback.

See [System Architecture](architecture/system-architecture.md) for the design, boundaries, and safety notes.

## Repository structure

```text
_posts/                  # Learning notes
curriculum/              # Six-week learning roadmap
architecture/            # System architecture and role map
environment/             # Development environment notes
glossary/                # Robotics dictionary
assets/diagrams/         # SVG technical diagrams
projects/                # Implementation projects (added as built)
.github/workflows/       # Jekyll / GitHub Pages deployment
```

## Engineering principles

- Record what was actually implemented and tested.
- Distinguish completed work, ongoing experiments, and future plans.
- Include reproducible commands, logs, and validation evidence.
- Treat simulation and hardware safety separately.
- Do not publish confidential information from previous employers.

## Blog

The site is generated with Jekyll and deployed through GitHub Pages. A push to `main` triggers the GitHub Actions Pages workflow. The repository is the source; the public website is the rendered output.
