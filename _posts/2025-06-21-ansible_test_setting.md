---
layout: post
title: "테스트환경설정"
date: 2025-06-21 22:00:00 +0900
categories: ansible
background: '/img/posts/08.jpg'
---


| 역할 | 이름 | 예시 IP
|  ---  |  ---  |  ---  |
제어 노드 | `ansible-master` |192.168.0.101
대상 노드 |`node1`, `node2` | 192.168.0.102, 192.168.0.103

-   **네트워크 어댑터**: VirtualBox → 브리지 어댑터

----------

## 1단계: Ubuntu 설치 후 기본 설정

각 VM에서 아래 명령을 수행

### 1.1 사용자 이름 확인

```bash
whoami


```

→ 이후 Ansible의 `ansible_user`로 사용됨

### 1.2 네트워크 확인을 위한 net-tools 설치

```bash
sudo apt update
sudo apt install -y net-tools
ifconfig


```

→ IP 확인 (브리지 모드에서 자동 할당됨)

----------

## 2단계: SSH 서버 설정 (node1, node2)

```bash
sudo apt install -y openssh-server
sudo dpkg-reconfigure openssh-server   # 호스트 키 재생성
sudo systemctl enable ssh
sudo systemctl start ssh


```

SSH 포트 확인:

```bash
sudo ss -tnlp | grep :22


```

----------

## 3단계: Ansible 제어 노드에서 SSH 키 생성 및 배포

### 3.1 키 생성

```bash
ssh-keygen   # 계속 Enter

```

### 3.2 키 배포

```bash
ssh-copy-id 사용자명@192.168.0.102  # node1
ssh-copy-id 사용자명@192.168.0.103  # node2


```

→ 비밀번호 입력 후 키 복사 성공

----------

## 4단계: Ansible 설치 및 설정 (ansible-master)

### 4.1 Ansible 설치

```bash
sudo apt update
sudo apt install -y ansible
ansible --version

```

### 4.2 인벤토리 디렉토리 수동 생성

```bash
sudo mkdir -p /etc/ansible
sudo touch /etc/ansible/hosts
sudo chmod 644 /etc/ansible/hosts


```

----------

## 5단계: 인벤토리(hosts) 파일 작성

```bash
sudo vi /etc/ansible/hosts


```

내용 예시:

```
[targets]
node1 ansible_host=192.168.0.102 ansible_user=ubuntu
node2 ansible_host=192.168.0.103 ansible_user=ubuntu


```

> ※ ubuntu는 대상 서버의 사용자명 (실제 로그인 가능해야 함)

----------

## 6단계: 연결 테스트

```bash
ansible targets -m ping

```

또는 명시적 경로 지정:

```bash
ansible targets -i /etc/ansible/hosts -m ping

```

예상 출력:

```
node1 | SUCCESS => {"ping": "pong"}
node2 | SUCCESS => {"ping": "pong"}


```

----------

## 문제 해결 체크리스트

문제 | 해결 방법
| --- | --- |
`no inventory was parsed` | `/etc/ansible/hosts` 존재 여부 확인 또는 `-i` 옵션 사용
`E212: Can't open file for writing` | `sudo` 없이 파일 열었음 → `sudo vi /etc/ansible/hosts` 사용
`sshd: no hostkeys available` | `sudo dpkg-reconfigure openssh-server` 실행하여 호스트 키 생성
SSH 접속 안됨 | 대상 노드에 `openssh-server` 설치 및 `sshd` 실행 확인

----------

## 명령어 요약

----------

```bash
# 사용자명 확인
whoami

# 네트워크 확인
ifconfig

# SSH 시작
sudo systemctl start ssh

# SSH 포트 확인
sudo ss -tnlp | grep :22

# Ansible 설치
sudo apt install ansible

# 키 복사
ssh-copy-id 사용자명@192.168.0.102

# 연결 테스트
ansible targets -i /etc/ansible/hosts -m ping


```