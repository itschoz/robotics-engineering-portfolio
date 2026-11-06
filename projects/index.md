---
layout: page
title: Projects
permalink: /projects/
---

<div class="hub-wrap">
  <header class="page-intro"><div class="eyebrow">Engineering Work</div><h1>Projects</h1><p>ROS 2, 시뮬레이션, 임베디드 하드웨어를 연결하는 메인 프로젝트와 각 기술 트랙의 구현 결과를 정리합니다. 계획과 실제 검증 결과를 구분해 공개합니다.</p></header>
  <div class="hub-grid">
    <a class="hub-card" href="{{ '/projects/joint-control/' | relative_url }}"><span class="eyebrow">Main Project</span><h2>Robotic Joint Control System</h2><p>ROS 2 PC 애플리케이션과 MuJoCo / STM32 Backend를 연결하는 통합 관절 제어 프로젝트.</p><span class="status progress">In Progress</span></a>
    <a class="hub-card" href="{{ '/architecture/' | relative_url }}"><span class="eyebrow">Architecture</span><h2>System Architecture</h2><p>제어 계층, Middleware, 교체 가능한 Simulation / Hardware Backend 구조.</p><span class="status verified">Design Document</span></a>
    <a class="hub-card" href="{{ '/projects/mujoco/' | relative_url }}"><span class="eyebrow">Simulation</span><h2>MuJoCo Simulation</h2><p>URDF / MJCF 모델링, 관절 동작 및 시뮬레이션 검증 트랙.</p><span class="status planned">Planned</span></a>
    <a class="hub-card" href="{{ '/projects/stm32/' | relative_url }}"><span class="eyebrow">Embedded</span><h2>STM32 Firmware</h2><p>UART 프로토콜, PWM, Timer, Encoder와 펌웨어 제어 트랙.</p><span class="status planned">Planned</span></a>
    <a class="hub-card" href="{{ '/projects/ros2-integration/' | relative_url }}"><span class="eyebrow">Integration</span><h2>ROS 2 Integration</h2><p>ROS 2 통신, Controller, Serial Bridge와 시스템 통합.</p><span class="status progress">In Progress</span></a>
  </div>
</div>
