---
layout: post
title: "Ansible이란"
date: 2025-06-21 22:00:00 +0900
categories: ansible
background: '/img/posts/07.jpg'
---



  

#  1. Ansible이란

  

  

##  **1-1. Ansible 개념**

  

  

- Ansible은 **자동화 도구**로 인프라 환경 설정, 앱 배포, 서버 관리 등을 코드로 작성하고 자동 실행합니다.

  

- 시스템 구성, 소프트웨어 배포, 지속적 배포(CD), 다운타임없는 업데이트와 같은 고급 IT작업을 자동화하고 조정(오케스트레이션)할 수 있도록 설계되었다.

  

-  **자동화 범위**:

  

- 서버 설정 자동화 (ex. 사용자, 패키지, 방화벽 설정)

  

- 애플리케이션 배포 (ex. Git pull → 빌드 → 서비스 재시작)

  

- 보안 정책 적용 (ex. SSH 포트 제한, 암호 정책 설정)

  

- 클라우드 리소스 생성 (ex. AWS EC2 인스턴스 생성)

  

  
  

---

  

  

##  **1-2. Ansible의 주요 특징**

  

  

|  **특징**  |  **설명**  |
|  ---  |  ---  |
|  **에이전트리스 (Agentless)**  | 대상 서버에 별도의 에이전트 설치 없이 **SSH만으로 통신**함. 유지 관리가 간편하고 보안 부담이 적음. |
|  **간단한 문법 (YAML 기반 Playbook)**  | 사람이 읽기 쉬운 YAML 형식으로 작업 정의. 패키지 설치, 설정 배포, 서비스 재시작 등을 **순서대로 작성 가능**. |
|  **멱등성 (Idempotent)**  | 같은 작업을 여러 번 실행해도 **결과가 항상 동일**하게 유지됨. 예: 이미 설치된 nginx는 건너뜀. |
|  **모듈 기반 구조**  | 수천 개의 기능이 **모듈 형태로 제공**됨. 예: `yum`, `apt`, `copy`, `service` 등. 사용자 정의 모듈도 가능. |
|  **확장성**  |  **인벤토리 파일**을 통해 서버를 그룹 단위로 관리. AWS, Azure, GCP와 같은 **클라우드 환경과도 연동** 가능. |
|  |  |

  
  
  

---

  

  

##  **1-3. Ansible이 해결하는 문제**

  

  

|  **업무 영역**  |  **적용 내용**  |
|  ---  |  ---  |
|  **서버 초기 세팅**  | 사용자 계정 생성, 패키지 설치, Timezone, 방화벽 설정 등 |
|  **애플리케이션 배포**  | Git에서 코드 받아오기, 빌드, 배포, 서비스 재시작 등 전체 배포 프로세스 자동화 |
|  **보안 정책 적용**  | 패스워드 복잡성, 방화벽, 특정 포트 차단 등 정책을 모든 서버에 일관되게 적용 |
|  **로그 수집기 배포**  | Filebeat, Fluentd 등 로그 수집기 설치와 설정 자동화 |
|  **다수 서버 대상 작업**  | 수십~수천 대 서버에 동시에 설정 변경, 업데이트, 점검 |
|  **클라우드 자원 생성**  | AWS/GCP 인스턴스, VPC, 보안 그룹 생성과 설정 자동화 |
|  **Kubernetes 초기화**  | kubeadm으로 마스터/워커 클러스터 구성 자동화 |

  
  
  

---

  

  

##  **1-4. Ansible의 장점**

  

  

|  **장점**  |  **설명**  |
|  ---  |  ---  |
|  **에이전트리스 구조**  | 대상 서버에 별도 소프트웨어 설치 없이 SSH만 있으면 동작. 유지 관리가 단순하고 효율적. |
|  **간단한 문법 (YAML)**  | 사람이 읽고 쓰기 쉬운 YAML 문법 사용 → 코드 작성과 유지보수가 쉬움. |
|  **멱등성 보장**  | 같은 작업을 여러 번 실행해도 결과가 동일 → 실수로 중복 실행해도 시스템에 변화 없음. |
|  **빠른 도입 가능**  | 복잡한 설정 없이도 금방 실무에 적용 가능 → 작은 팀에서도 쉽게 시작 가능. |
|  **모듈 기반 구조**  | 다양한 작업을 위한 모듈이 내장되어 있어 확장성 뛰어남 (ex. docker, aws, service 등). |
|  **커뮤니티와 생태계**  | 문서화가 잘 되어 있고, 예제와 플러그인이 풍부하며 Red Hat의 공식 지원도 있음. |
|  **멀티 OS 지원**  | 리눅스뿐 아니라 윈도우 서버 관리도 가능 (WinRM 지원) |

---

  

  

##  **1-5. Ansible의 단점**

  

  

|  **단점**  |  **설명**  |
|  ---  |  ---  |
|  **속도 문제**  | SSH 방식은 순차적(Push 방식) 실행이기 때문에 **수백 대 이상의 서버 작업은 느릴 수 있음**. |
|  **복잡한 로직 구현 한계**  | 조건문, 반복문 등의 복잡한 흐름을 표현하기 어려움. (Jinja2 템플릿과 변수로 극복 가능하지만 한계 존재) |
|  **에러 로깅 및 디버깅 약함**  | 에러 발생 시 메시지가 단순하며, 복잡한 작업 디버깅에 어려움이 있음. |
|  **GUI 부족**  | 기본적으로 CLI 기반이며, GUI(AWX, Ansible Tower)는 별도 설치와 관리 필요. |
|  **대규모 분산 처리 비효율적**  | 수많은 노드를 동시에 처리하는 경우 Puppet/Chef 같은 Pull 방식 도구가 더 적합할 수 있음. |

---

  

  

##  **1-6. 다른 도구와 비교**

  

  

|  **항목**  |  **Ansible**  |  **Chef / Puppet**  |  **Terraform**  |
|  ---  |  ---  |  ---  | ---
| 설치 방식 | 에이전트 없음 | 에이전트 필요 | 에이전트 없음 |
| 실행 방식 | Push 방식 (SSH) | Pull 방식 | 선언적 실행 (IaC) |
| 구성 목적 | 설정 및 운영 자동화 | 설정 및 운영 자동화 | 인프라 프로비저닝 중심 |
| 문법 | YAML | Ruby DSL | HCL |
| 학습 곡선 | 쉬움 | 다소 어려움 | 보통 |
| 적합한 환경 | 중소규모, 빠른 시작 | 대규모 엔터프라이즈 | 멀티클라우드/클라우드 중심 |

  

---

  

  

##  1-7. Ansible의 개념 구성요소

  

  

| 요소 | 설명 |
|  ---  |  ---  |
|  **Inventory (인벤토리)**  | 관리 대상 노드의 목록 및 그룹을 정의 IP 또는 도메인 기반, 정적/동적 인벤토리 구성 가능 |
|  **Playbooks (플레이북)**  | 여러 Task를 순서대로 저장한 YAML 파일 반복 작업을 코드화하여 재사용 가능 |
|  **Modules (모듈)**  | Ansible의 기능 단위 코드 블록 예: `ping`, `yum`, `copy`, `service` 등 수많은 모듈 제공 JSON을 반환할 수 있는 언어로 컨스텀 모듈 작성 가능 |
|  **Control Node (제어 노드)**  | Ansible이 설치된 시스템 `/usr/bin/ansible`, `/usr/bin/ansible-playbook` 실행 가능 Python이 설치되어 있다면 대부분의 리눅스/유니스 계열에서 사용 가능 Windows는 제어 노드로 사용 불가 |
|  **Managed Nodes (관리 노드)**  | Ansible이 제어하는 대상 서버 또는 네트워크 장비 변수의 에이전트나 Ansible 설치 불필요 |
|  **Tasks (작업 단위)**  | 모듈을 호출하여 실행하는 단일 명령 Playbook 또는 ad-hoc 명령으로 실행 가능 |

  