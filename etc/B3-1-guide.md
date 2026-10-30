# ☁️ [B3-1 미션 가이드] 내가 만든 웹사이트를 인터넷에 올려 누구나 쓰게 하기

> **대상 독자**: AWS(Amazon Web Services)를 처음 접하는 입문자  
> **목표**: 나만의 가상 네트워크(VPC)를 직접 설계하고, 가상 컴퓨터(EC2)에 웹 서버(Nginx)를 띄워 전 세계 누구나 접속할 수 있는 웹사이트를 구축한 뒤, 안전하게 리소스를 정리(과금 방지)하기

---

## 🧭 목차

1. [미션 전체 그림 한눈에 보기](#1-미션-전체-그림-한눈에-보기)
2. [실습 전 필수 핵심 개념 (AWS 입문자 눈높이 설명)](#2-실습-전-필수-핵심-개념-aws-입문자-눈높이-설명)
3. [단계별 미션 수행 절차 (Step-by-Step)](#3-단계별-미션-수행-절차-step-by-step)
   - [Step 0. 사전 준비: IAM 최소 권한 사용자 만들기](#step-0-사전-준비-iam-최소-권한-사용자-만들기)
   - [Step 1. 네트워크(VPC) 구축하기](#step-1-네트워크vpc-구축하기)
   - [Step 2. 보안 그룹(Security Group) 만들기](#step-2-보안-그룹security-group-만들기)
   - [Step 3. 가상 서버(EC2) 생성 및 SSH 원격 접속하기](#step-3-가상-서버ec2-생성-및-ssh-원격-접속하기)
   - [Step 4. 웹 서버(Nginx) 설치 및 실행하기](#step-4-웹-서버nginx-설치-및-실행하기)
   - [Step 5. 외부 접속 검증하기](#step-5-외부-접속-검증하기)
   - [Step 6. 트러블슈팅(오류 해결) 실습 및 보고서 작성](#step-6-트러블슈팅오류-해결-실습-및-보고서-작성)
   - [Step 7. 최종 제출물 문서 정리하기](#step-7-최종-제출물-문서-정리하기)
   - [Step 8. 리소스 정리 및 과금 방지 (필수!)](#step-8-리소스-정리-및-과금-방지-필수)
4. [과제 목표 셀프 체크 Q&A (5문 5답)](#4-과제-목표-셀프-체크-qa-5문-5답)
5. [번외: 보너스 과제 가이드](#5-번외-보너스-과제-가이드)
   - [번외 1. 무료 도메인 연결 및 HTTPS(Let's Encrypt) 적용하기](#번외-1-무료-도메인-연결-및-httpslets-encrypt-적용하기)
   - [번외 2. Docker 컨테이너로 웹 서비스 배포하기](#번외-2-docker-컨테이너로-웹-서비스-배포하기)

---

## 1. 미션 전체 그림 한눈에 보기

내 노트북에서 만든 웹사이트는 내 노트북(`localhost`)에서만 보입니다. 다른 사람도 접속하게 하려면 **24시간 켜져 있고, 공인된 인터넷 주소(Public IP)를 가진 컴퓨터(서버)**가 필요합니다.

이번 미션에서는 AWS라는 클라우드 공간에 **나만의 땅(VPC)을 분양받고 → 대문(Internet Gateway)과 도로(Route Table)를 깔고 → 경비원(Security Group)을 세운 뒤 → 컴퓨터(EC2)를 한 대 들여놓아 웹 서버(Nginx)를 실행**합니다.

### 제출해야 할 최종 결과물 4가지

실습을 진행하면서 아래 4가지 결과물을 챙겨야 합니다.

| 번호 | 제출물 | 파일 위치 / 형식 | 핵심 내용 |
| :---: | :--- | :--- | :--- |
| **1** | **아키텍처 다이어그램** | `docs/architecture.png` (또는 `.pdf`) | VPC, Subnet, IGW, EC2, Security Group 및 트래픽 화살표 표시 |
| **2** | **외부 접속 증빙** | `README.md` + `screenshots/` 이미지 | 브라우저(`http://<퍼블릭IP>`) 또는 `/health` 접속 성공 화면 캡처 |
| **3** | **트러블슈팅 보고서** | `docs/troubleshooting.md` (또는 `.pdf`) | 최소 1건 이상 오류에 대해 `증상 → 가설 → 검증 → 조치 → 결과 → 재발방지` 작성 |
| **4** | **리소스 정리 체크리스트** | `docs/cleanup-checklist.md` | 실습 후 과금 방지를 위해 EC2, EBS, VPC 등을 모두 삭제했는지 체크 |

---

## 2. 실습 전 필수 핵심 개념 (AWS 입문자 눈높이 설명)

AWS 콘솔에는 낯선 영어 약어가 많이 등장합니다. **'신도시 아파트 단지 건설'**에 비유하면 아주 쉽게 이해할 수 있습니다.

### 2.1 클라우드 & 리전(Region)
* **클라우드(AWS)**: 아마존이 전 세계에 지어놓은 초대형 컴퓨터 임대 센터입니다. 필요한 만큼 빌려 쓰고, 다 쓰면 반납해 요금을 아낄 수 있습니다.
* **리전(Region)**: 데이터센터가 위치한 **국가/도시**입니다. 한국 사용자를 위한 서버라면 물리적으로 가장 가까운 **서울 리전(`ap-northeast-2`)**에 만들어야 속도가 빠릅니다.
* **가용 영역(Availability Zone, AZ)**: 서울 리전 안에서도 정전이나 화재에 대비해 물리적으로 떨어뜨려 놓은 여러 개의 데이터센터 건물(`ap-northeast-2a`, `2b`, `2c`, `2d`)입니다.

### 2.2 네트워크 4총사 (VPC, Subnet, IGW, Route Table)

```text
                    🌐 [ 외부 사용자 (내 PC 브라우저) ]
                             │               ▲
       (1) 요청 (목적지: 퍼블릭IP)             │ (6) 인터넷으로 응답 반환
                             ▼               │
              ┌──────────────────────────────┴────────────┐
              │       🚪 Internet Gateway (my-igw)        │
              │ • 들어올 때: 퍼블릭 IP ➔ 사설 IP(10.0.1.x) │
              │ • 나갈 때  : 사설 IP   ➔ 퍼블릭 IP (NAT)   │
              └──────────────┬───────────────▲────────────┘
┌────────────────────────────┼───────────────┼──────────────────────────┐
│ 🏞️ VPC (10.0.0.0/16)       │ (2)           │ (5)                      │
│                            ▼               │                          │
│              ┌───────────────────────────────────────────┐            │
│              │        🪧 Route Table (도로 이정표)        │            │
│              │ • [규칙 1] 10.0.0.0/16 ➔ local (들어올 때) │            │
│              │ • [규칙 2] 0.0.0.0/0   ➔ IGW   (나갈 때)   │            │
│              └─────────────┬───────────────▲─────────────┘            │
│     (3) [규칙 1: local] 참조│               │ (4) [규칙 2: 0.0.0.0/0]  │
│         10.0.1.x 찾아 이동 │               │     참조하여 IGW로 이동  │
│                            ▼               │                          │
│   ┌────────────────────────────────────────┴──────────────────────┐   │
│   │ 🏘️ Public Subnet (10.0.1.0/24)                                │   │
│   │                                                               │   │
│   │   ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐   │   │
│   │     🛡️ Security Group (방화벽 경비원)                          │   │
│   │   │ • 인바운드 허용: 80포트(0.0.0.0/0), 22포트(내 IP만)     │   │   │
│   │   │  ┌─────────────────────────────────────────────────┐  │   │   │
│   │   │  │ 💻 EC2 인스턴스 (Nginx 웹 서버, 80번 포트)       │  │   │   │
│   │   │  │ • 실제 할당된 IP (사설 IP): 10.0.1.x             │  │   │   │
│   │   │  └─────────────────────────────────────────────────┘  │   │   │
│   │   └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘   │   │
│   └───────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
```

1. **VPC (Virtual Private Cloud) = '나만의 사유지(전용 단지)'**
   * AWS라는 거대한 땅 안에서 다른 사용자들과 완벽히 격리된 **나만의 가상 네트워크 공간**입니다.
   * 처음 VPC를 만들면 외부 인터넷과 완전히 단절된 안전한 빈 땅 상태입니다.
   * **`10.0.0.0/16` (CIDR 표기법)**: 이 단지 안에서 쓸 수 있는 내부 사설 IP 주소의 범위입니다. (`10.0.0.0`부터 `10.0.255.255`까지 약 65,536개의 IP를 가질 수 있는 큰 땅이라는 뜻입니다.)

2. **Subnet (서브넷) = '단지 안의 구역(블록)'**
   * VPC라는 큰 땅을 용도에 맞게 더 작은 구역으로 나눈 것입니다.
   * **Public Subnet (퍼블릭 서브넷)**: 외부 인터넷과 직접 통신할 수 있는 구역입니다. 웹 서버처럼 외부 손님이 찾아와야 하는 건물은 여기에 짓습니다. (`10.0.1.0/24` → `10.0.1.0` ~ `10.0.1.255`까지 256개의 IP를 가진 구역)
   * *(참고) Private Subnet (프라이빗 서브넷)*: 외부에서 직접 접근할 수 없는 내부 전용 구역입니다. 주로 데이터베이스(DB)를 숨겨둘 때 씁니다.

3. **Internet Gateway (IGW, 인터넷 게이트웨이) = '단지 정문(톨게이트)'**
   * 고립된 VPC 단지와 바깥 세상(외부 인터넷)을 연결해 주는 **유일한 출입문**입니다. VPC에 이 문을 달아주지 않으면 안에서 밖으로 나갈 수도, 밖에서 안으로 들어올 수도 없습니다.

4. **Route Table (라우팅 테이블) = '도로 이정표(내비게이션)'**
   * 패킷이 **들어올 때**와 **나갈 때** 모두 목적지 IP를 확인하여 어디로 보낼지 결정하는 이정표입니다. 다만 **들어올 때와 나갈 때 참조하는 규칙(줄)이 다릅니다.**
   * **[규칙 1] `10.0.0.0/16` → `local` (들어올 때 & 내부 통신할 때 참조)**:
     * VPC 생성 시 기본으로 들어있는 규칙입니다.
     * 외부에서 IGW로 들어온 패킷은 IGW에서 목적지가 사설 IP(`10.0.1.x`)로 변환(NAT)된 뒤, 이 **`10.0.0.0/16 → local` 규칙**을 참조하여 해당 서브넷의 EC2를 찾아갑니다.
   * **[규칙 2] `0.0.0.0/0` → `igw-xxxx` (밖으로 나갈 때 참조)**:
     * 우리가 직접 추가해야 하는 규칙입니다. `0.0.0.0/0`은 **"세상의 모든 IP 주소"**를 뜻합니다.
     * EC2가 외부 인터넷으로 요청을 보내거나 외부 사용자(공인 IP)에게 응답을 돌려보낼 때는 목적지가 사설 IP(`10.0.0.0/16`)가 아니므로, 이 **`0.0.0.0/0 → IGW` 규칙**을 참조하여 정문(Internet Gateway)으로 나갑니다. 이 규칙이 연결된 서브넷을 **Public Subnet**이라고 부릅니다.

### 2.3 컴퓨팅과 스토리지 (EC2, EBS, Key Pair)
* **EC2 (Elastic Compute Cloud) = '임대 컴퓨터 본체'**
  * 퍼블릭 서브넷 안에 설치하는 가상 컴퓨터(서버)입니다.
  * **프라이빗 IP (예: `10.0.1.x`)**: VPC 단지 내부에서만 통하는 내선 번호입니다.
  * **퍼블릭 IP (예: `43.201.x.x`)**: 전 세계 인터넷에서 이 컴퓨터를 찾아올 수 있는 공인 주소입니다.
* **EBS (Elastic Block Store) = '컴퓨터에 꽂는 외장 SSD(하드디스크)'**
  * EC2 컴퓨터의 운영체제(Ubuntu)와 파일들이 저장되는 저장장치입니다.
* **키 페어 (Key Pair, `.pem` 파일) = '현관문 마스터 열쇠'**
  * 비밀번호 대신 사용하는 고도의 암호화 열쇠 파일입니다. 이 파일을 잃어버리면 내가 만든 EC2 컴퓨터에 원격 접속(SSH)할 수 없으므로 잘 보관해야 합니다.

### 2.4 보안의 두 기둥: Security Group vs IAM

입문자가 가장 많이 헷갈리는 두 가지 보안 개념입니다. **역할이 완전히 다릅니다!**

| 구분 | Security Group (보안 그룹) | IAM (Identity and Access Management) |
| :--- | :--- | :--- |
| **한 줄 비유** | **EC2 컴퓨터 앞을 지키는 방화벽 문지기** | **AWS 관리 콘솔 출입 사원증(권한증)** |
| **지키는 대상** | **네트워크 트래픽** (누가 몇 번 포트로 서버에 접속하는가?) | **AWS 기능/API** (누가 AWS에서 서버를 만들고 삭제할 수 있는가?) |
| **질문 예시** | "이 IP에서 오는 80번 포트(웹) 요청을 통과시킬까?" | "이 사용자가 EC2를 생성하거나 S3에 접근할 자격이 있는가?" |
| **최소 권한 적용법** | 꼭 필요한 포트만 열기 (`80`번은 전체, `22`번 SSH는 **내 IP만** 허용) | 루트 계정 사용 금지, 실습에 필요한 서비스(EC2, VPC) 권한만 부여 |

> 💡 **중요: 포트(Port)란?**  
> IP 주소가 '건물 주소'라면, 포트 번호는 건물 안의 **'창구 번호'**입니다.  
> * **22번 포트 (SSH)**: 서버 관리자가 원격으로 명령어를 치기 위해 들어가는 **관리자 전용 뒷문**입니다. 해커들이 가장 먼저 공격하므로 반드시 **내 IP(`x.x.x.x/32`)에서만** 열어야 합니다.  
> * **80번 포트 (HTTP)**: 일반 사용자가 웹 브라우저로 홈페이지에 접속할 때 들어오는 **정문**입니다. 누구나 홈페이지를 봐야 하므로 **전 세계 모든 IP(`0.0.0.0/0`)에** 열어줍니다.

---

## 3. 단계별 미션 수행 절차 (Step-by-Step)

이제 AWS 콘솔에서 하나씩 따라 하며 미션을 수행해 봅시다.

---

### Step 0. 사전 준비: IAM 최소 권한 사용자 만들기

> **관련 요구사항**: 루트 계정은 사용하지 않으며, `AdministratorAccess`(전체 관리자 권한) 없이 EC2/VPC 구성에 필요한 최소 권한만 가진 IAM 사용자로 진행합니다.

AWS에 처음 가입할 때 만든 이메일 계정을 **루트 사용자(Root User)**라고 합니다. 루트 계정은 모든 권한을 가지고 있어서 해킹당하면 수천만 원의 요금 폭탄을 맞을 수 있습니다. 따라서 일상적인 작업용 부계정(**IAM 사용자**)을 만들어 사용합니다.

1. **AWS 관리 콘솔 로그인**: 처음 한 번만 루트 계정으로 로그인합니다.
2. **IAM 서비스 이동**: 상단 검색창에 `IAM`을 입력하고 클릭합니다.
3. **사용자 생성**:
   * 왼쪽 메뉴에서 **[사용자(Users)]** → 우측 상단 **[사용자 생성(Create user)]** 클릭
   * **사용자 이름**: `codyssey-student` (원하는 이름) 입력
   * **[AWS Management Console에 대한 사용자 액세스 권한 제공]** 체크박스 선택
   * **[IAM 사용자를 생성하고 싶음(I want to create an IAM user)]** 선택
   * 콘솔 암호: **[사용자 지정 암호]** 선택 후 기억하기 쉬운 비밀번호 입력 (아래 '다음 로그인 시 새 암호 생성 필요'는 체크 해제하면 편리합니다) → **[다음]** 클릭
4. **권한 설정 (진짜 최소 권한 원칙 적용 — 사용자 지정 JSON 정책 만들기)**:
   * AWS에서 기본 제공하는 `AmazonEC2FullAccess` 같은 관리형 정책은 로드밸런서(ELB), 오토스케일링, NAT 게이트웨이, VPN 등 이번 실습에 쓰지 않는 권한까지 통째로 허용합니다.
   * 미션 요구사항(*"생성·조회·태그·연결 등 실습에 필요한 작업 중심"*)을 엄격하게 만족하기 위해, **이번 실습에 꼭 필요한 리소스(VPC, Subnet, IGW, Route Table, Security Group, Key Pair, EC2, EBS)만 서울 리전(`ap-northeast-2`)에서 다룰 수 있는 맞춤형 최소 권한 정책**을 만들어 연결합니다.
   * 권한 설정 화면에서 **[직접 정책 연결(Attach policies directly)]** 선택 → 우측의 **[정책 생성(Create policy)]** 버튼 클릭 (새 탭이 열립니다)
   * 새 탭의 정책 편집기 우측 상단에서 **[JSON]** 버튼을 클릭하고, 기존 내용을 지운 뒤 아래 JSON을 그대로 복사해 붙여넣습니다:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowConsoleReadOnlyAndAmiLookup",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "ec2:GetConsoleOutput",
                "ec2:GetConsoleScreenshot",
                "ssm:GetParameter",
                "ssm:GetParameters",
                "ssm:DescribeParameters"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowMissionVpcNetworkInSeoulOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVpc",
                "ec2:ModifyVpcAttribute",
                "ec2:DeleteVpc",
                "ec2:CreateSubnet",
                "ec2:ModifySubnetAttribute",
                "ec2:DeleteSubnet",
                "ec2:CreateInternetGateway",
                "ec2:AttachInternetGateway",
                "ec2:DetachInternetGateway",
                "ec2:DeleteInternetGateway",
                "ec2:CreateRouteTable",
                "ec2:CreateRoute",
                "ec2:ReplaceRoute",
                "ec2:DeleteRoute",
                "ec2:AssociateRouteTable",
                "ec2:DisassociateRouteTable",
                "ec2:DeleteRouteTable"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-northeast-2"
                }
            }
        },
        {
            "Sid": "AllowMissionSecurityGroupInSeoulOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:ModifySecurityGroupRules",
                "ec2:DeleteSecurityGroup"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-northeast-2"
                }
            }
        },
        {
            "Sid": "AllowMissionEc2ComputeAndTaggingInSeoulOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags",
                "ec2:DeleteTags",
                "ec2:CreateKeyPair",
                "ec2:DeleteKeyPair",
                "ec2:RunInstances",
                "ec2:StartInstances",
                "ec2:StopInstances",
                "ec2:RebootInstances",
                "ec2:TerminateInstances",
                "ec2:CreateVolume",
                "ec2:AttachVolume",
                "ec2:DetachVolume",
                "ec2:ModifyVolume",
                "ec2:DeleteVolume",
                "ec2:CreateNetworkInterface",
                "ec2:DeleteNetworkInterface",
                "ec2:ModifyNetworkInterfaceAttribute",
                "ec2:AttachNetworkInterface",
                "ec2:DetachNetworkInterface",
                "ec2:AllocateAddress",
                "ec2:AssociateAddress",
                "ec2:DisassociateAddress",
                "ec2:ReleaseAddress"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "ap-northeast-2"
                }
            }
        },
        {
            "Sid": "AllowBillingDashboardReadOnly",
            "Effect": "Allow",
            "Action": [
                "aws-portal:View*",
                "billing:Get*",
                "ce:Get*",
                "ce:Describe*",
                "freetier:Get*"
            ],
            "Resource": "*"
        }
    ]
}
```

   * **[다음]** 클릭 → **정책 이름**에 `Codyssey-B3-LeastPrivilege` 입력 → 하단 **[정책 생성]** 클릭
   * 다시 원래의 사용자 생성 탭으로 돌아와서 정책 목록 옆의 **새로고침(↻) 버튼**을 누른 뒤, 검색창에 `Codyssey-B3-LeastPrivilege`를 검색해 체크합니다.
   * **[다음]** → **[사용자 생성]** 클릭

#### 📊 기존 `FullAccess` 대비 어떻게 권한을 줄였는지 요약

| 구분 (`Sid`) | 허용한 작업 (필수 권한만 선별) | 차단(제외)된 위험 요소 |
| :--- | :--- | :--- |
| **1. 조회 (`AllowConsoleReadOnlyAndAmiLookup`)** | `ec2:Describe*`, Ubuntu AMI 목록 조회용 `ssm:GetParameter*` | 수정/삭제 불가 (순수 콘솔 화면 표시용) |
| **2. 네트워크 (`AllowMissionVpcNetworkInSeoulOnly`)** | `Vpc`, `Subnet`, `InternetGateway`, `RouteTable`, `Route`의 생성·연결·삭제 | 요금 폭탄 주범인 **NAT Gateway, VPN, Transit Gateway 생성 원천 차단** + **서울 리전(`ap-northeast-2`) 외 생성 차단** |
| **3. 보안그룹 (`AllowMissionSecurityGroupInSeoulOnly`)** | `SecurityGroup` 생성·삭제 및 인/아웃바운드 규칙 편집 | 네트워크 ACL 임의 조작 등 제외 + **서울 리전으로 제한** |
| **4. 컴퓨트 (`AllowMissionEc2ComputeAndTaggingInSeoulOnly`)** | `KeyPair`, `Instances(Run/Stop/Terminate)`, `Volume(EBS)`, `NetworkInterface`, `Tag` | 로드밸런서(ELB), 오토스케일링, 스팟 플릿 등 제외 + **서울 리전으로 제한** |
| **5. 과금 확인 (`AllowBillingDashboardReadOnly`)** | Step 8에서 Billing 대시보드 `$0.00` 조회 권한 (`View*`, `Get*`) | 결제 수단 변경 등 수정 권한 일절 없음 |
5. **IAM 사용자로 다시 로그인하기**:
   * 완료 화면에 표시되는 **콘솔 로그인 URL**(`https://<12자리숫자>.signin.aws.amazon.com/console`)을 복사해 메모장에 적어둡니다.
   * 우측 상단 계정 이름을 눌러 **로그아웃**한 뒤, 방금 복사한 URL로 접속하여 `codyssey-student` 계정으로 로그인합니다.
6. **서울 리전 확인 (필수!)**:
   * 로그인 후 우측 상단 메뉴바에서 지역이 **아시아 태평양(서울) `ap-northeast-2`**로 되어 있는지 반드시 확인합니다. (다른 지역으로 되어 있다면 클릭해서 `서울`로 변경하세요.)

---

### Step 1. 네트워크(VPC) 구축하기

> **관련 요구사항**: VPC 1개, Public Subnet 1개, Internet Gateway 1개를 만들고 Route Table에 `0.0.0.0/0 → IGW` 경로를 설정합니다.

상단 검색창에 `VPC`를 검색하여 **VPC 대시보드**로 이동합니다.

#### 1-1. VPC 생성하기
1. 왼쪽 메뉴에서 **[내 VPC(Your VPCs)]** → **[VPC 생성(Create VPC)]** 클릭
2. 설정 입력:
   * **생성할 리소스**: **[VPC만(VPC only)]** 선택 *(직접 원리를 익히기 위해 'VPC만'을 선택합니다)*
   * **이름 태그**: `my-web-vpc`
   * **IPv4 CIDR 블록**: `10.0.0.0/16` 입력
   * 나머지 항목은 기본값 그대로 두고 하단 **[VPC 생성]** 클릭

#### 1-2. Public Subnet 생성하기
1. 왼쪽 메뉴에서 **[서브넷(Subnets)]** → **[서브넷 생성(Create subnet)]** 클릭
2. 설정 입력:
   * **VPC ID**: 방금 만든 `my-web-vpc` 선택
   * **서브넷 이름**: `my-public-subnet`
   * **가용 영역(Availability Zone)**: `아시아 태평양(서울) / ap-northeast-2a` 선택
   * **IPv4 VPC CIDR 블록**: `10.0.0.0/16` (자동 표시됨)
   * **IPv4 서브넷 CIDR 블록**: `10.0.1.0/24` 입력
   * 하단 **[서브넷 생성]** 클릭

#### 1-3. Internet Gateway(인터넷 게이트웨이) 생성 및 VPC 연결
1. 왼쪽 메뉴에서 **[인터넷 게이트웨이(Internet gateways)]** → **[인터넷 게이트웨이 생성]** 클릭
2. **이름 태그**: `my-igw` 입력 후 **[인터넷 게이트웨이 생성]** 클릭
3. **VPC에 연결하기 (초보자 필수 주의 구간!)**:
   * 생성 직후 상태가 `Detached(분리됨)`로 되어 있습니다. 대문을 사기만 하고 아직 단지에 달지 않은 상태입니다.
   * 우측 상단 노란 배너의 **[VPC에 연결(Attach to a VPC)]** 버튼(또는 **[작업] → [VPC에 연결]**) 클릭
   * **사용 가능한 VPC**에서 `my-web-vpc`를 선택하고 **[인터넷 게이트웨이 연결]** 클릭
   * 상태가 초록색 **`Attached(연결됨)`**로 바뀌었는지 확인합니다.

#### 1-4. Route Table(라우팅 테이블) 설정 및 서브넷 연결
1. 왼쪽 메뉴에서 **[라우팅 테이블(Route tables)]** → **[라우팅 테이블 생성]** 클릭
2. 설정 입력:
   * **이름**: `my-public-rt`
   * **VPC**: `my-web-vpc` 선택 → **[라우팅 테이블 생성]** 클릭
3. **외부 인터넷 경로(`0.0.0.0/0`) 추가하기**:
   * 생성된 `my-public-rt` 상세 화면 아래쪽 **[라우팅(Routes)]** 탭에서 **[라우팅 편집(Edit routes)]** 클릭
   * **[라우팅 추가(Add route)]** 버튼 클릭
   * **대상(Destination)**: `0.0.0.0/0` 입력
   * **대상(Target)**: 드롭다운에서 **[인터넷 게이트웨이(Internet Gateway)]** 선택 → 아래에 뜨는 `igw-xxxx (my-igw)` 선택
   * **[변경 사항 저장]** 클릭
4. **서브넷과 라우팅 테이블 연결하기 (빼먹기 쉬운 핵심 단계!)**:
   * 이정표를 만들었으니 이제 `my-public-subnet` 구역에 이 이정표를 세워야 합니다.
   * 아래쪽 탭 중 **[서브넷 연결(Subnet associations)]** 탭 클릭
   * **명시적 서브넷 연결** 영역의 **[서브넷 연결 편집(Edit subnet associations)]** 클릭
   * `my-public-subnet` 체크박스를 선택하고 **[연결 저장(Save associations)]** 클릭

---

### Step 2. 보안 그룹(Security Group) 만들기

> **관련 요구사항**: HTTP(80)는 `0.0.0.0/0` 허용, SSH(22)는 내 IP만 허용, 전체 포트(`0-65535`) 허용 금지

1. VPC 왼쪽 메뉴 하단(또는 EC2 메뉴)에서 **[보안 그룹(Security groups)]** → **[보안 그룹 생성]** 클릭
2. 기본 세부 정보 입력:
   * **보안 그룹 이름**: `my-web-sg`
   * **설명**: `Allow HTTP from anywhere and SSH from my IP`
   * **VPC**: 기본값으로 잡혀 있는 기본 VPC를 지우고(X 클릭), 반드시 방금 만든 **`my-web-vpc`**를 선택합니다! *(다른 VPC에 만들면 나중에 EC2에 연결할 수 없습니다)*
3. **인바운드 규칙(Inbound rules) 추가**:
   * **[규칙 추가]**를 눌러 아래 **2개의 규칙만** 정확히 만듭니다.

| 규칙 | 유형(Type) | 프로토콜 | 포트 범위 | 소스 유형(Source) | 소스 IP 값 | 설명 |
| :---: | :--- | :---: | :---: | :--- | :--- | :--- |
| **1** | **SSH** | TCP | `22` | **내 IP (My IP)** | 자동으로 내 공인 IP(`x.x.x.x/32`) 입력됨 | 관리자 원격 접속용 |
| **2** | **HTTP** | TCP | `80` | **Anywhere-IPv4** | `0.0.0.0/0` | 웹 서비스 외부 접속용 |

> ⚠️ **주의사항**
> * 카페, 도서관, 핫스팟 등 장소를 이동하면 내 IP가 바뀔 수 있습니다. 만약 자리를 옮겨서 SSH 접속이 안 된다면 보안 그룹의 SSH 규칙에서 다시 **[내 IP]**로 수정해 주면 됩니다.
> * 절대 `모든 트래픽(All traffic)`을 `0.0.0.0/0`으로 열지 마세요!

4. **아웃바운드 규칙(Outbound rules)**:
   * 기본값(`모든 트래픽 / 0.0.0.0/0`)을 그대로 둡니다. (서버 안에서 바깥 인터넷으로 패키지를 다운로드하거나 통신할 때 필요합니다.)
5. 하단 **[보안 그룹 생성]** 클릭

---

### Step 3. 가상 서버(EC2) 생성 및 SSH 원격 접속하기

> **관련 요구사항**: Public Subnet에 프리티어 micro 급 EC2 1대 생성, SSH 접속, 아웃바운드 통신(`curl https://example.com`) 확인

상단 검색창에 `EC2`를 검색하여 **EC2 대시보드**로 이동합니다.

#### 3-1. EC2 인스턴스 생성하기
1. **[인스턴스 시작(Launch instance)]** 주황색 버튼 클릭
2. **이름 및 태그**: `my-web-server` 입력
3. **애플리케이션 및 OS 이미지(AMI)**:
   * **[Ubuntu]** 선택
   * 아래 드롭다운에서 **`Ubuntu Server 24.04 LTS (HVM), SSD Volume Type`** (또는 `22.04 LTS`) 옆에 **`프리티어 사용 가능(Free tier eligible)`** 배지가 있는지 확인
4. **인스턴스 유형(Instance type)**:
   * **`t2.micro`** 또는 **`t3.micro`** 중 **`프리티어 사용 가능`**이라고 적힌 것을 선택합니다.
5. **키 페어(로그인)**:
   * **[새 키 페어 생성(Create new key pair)]** 클릭
   * **키 페어 이름**: `my-web-key`
   * **키 페어 유형**: `RSA` / **프라이빗 키 파일 형식**: `.pem` 선택
   * **[키 페어 생성]**을 누르면 내 컴퓨터 `다운로드(Downloads)` 폴더에 `my-web-key.pem` 파일이 저장됩니다. (절대 삭제하지 마세요!)
6. **네트워크 설정 (가장 중요! ⭐)**:
   * 오른쪽 위의 **[편집(Edit)]** 버튼을 반드시 클릭합니다.
   * **VPC**: **`my-web-vpc`** 선택
   * **서브넷**: **`my-public-subnet`** 선택
   * **퍼블릭 IP 자동 할당(Auto-assign public IP)**: 기본값이 '비활성화'로 되어 있을 수 있습니다. 반드시 **`활성화(Enable)`**로 변경합니다! *(이걸 안 켜면 퍼블릭 IP가 안 나와서 외부 접속이 불가능합니다)*
   * **방화벽(보안 그룹)**: **[기존 보안 그룹 선택(Select existing security group)]** 클릭 → 아래 드롭다운에서 아까 만든 **`my-web-sg`** 선택
7. **스토리지 구성(EBS)**:
   * 기본값인 `8 GiB` `gp3` 그대로 둡니다.
8. 우측 **[인스턴스 시작(Launch instance)]** 클릭!

#### 3-2. 생성된 EC2의 퍼블릭 IP 확인하기
1. **[인스턴스 보기]**를 눌러 목록으로 이동합니다.
2. `my-web-server`의 **인스턴스 상태**가 `Pending(대기 중)`에서 초록색 **`Running(실행 중)`**으로 바뀔 때까지 약 30초 기다립니다.
3. 인스턴스를 클릭하면 하단 세부 정보에 **퍼블릭 IPv4 주소**(예: `43.201.xxx.xxx`)가 표시됩니다. 이 주소를 복사해 둡니다.

#### 3-3. 내 컴퓨터에서 EC2로 SSH 원격 접속하기
내 맥(Mac)의 **터미널(Terminal)** 앱을 열고 아래 명령어를 순서대로 입력합니다.

```bash
# 1. 키 파일이 있는 폴더로 이동 (보통 다운로드 폴더)
cd ~/Downloads

# 2. 키 파일의 권한을 '나만 읽을 수 있게(400)' 변경 (필수! 안 하면 보안 에러 발생)
chmod 400 my-web-key.pem

# 3. SSH로 EC2 서버에 접속 (<퍼블릭IP> 자리에 본인의 EC2 퍼블릭 IP 입력)
ssh -i my-web-key.pem ubuntu@<퍼블릭IP>
```

* 처음 접속 시 `Are you sure you want to continue connecting (yes/no/[fingerprint])?`라고 물어보면 키보드로 **`yes`**를 입력하고 엔터를 칩니다.
* 터미널 프롬프트가 `ubuntu@ip-10-0-1-xxx:~$` 모양으로 바뀌었다면 **클라우드 서버 안으로 접속 성공**한 것입니다! 🎉

#### 3-4. 아웃바운드 인터넷 통신 검증하기 (기능 요구사항 1)
EC2 서버 안에서 바깥 인터넷으로 통신이 잘 나가는지 확인합니다.

```bash
curl -I https://example.com
```
* 결과 첫 줄에 `HTTP/2 200` (또는 `HTTP/1.1 200 OK`)이 출력되면 Public Subnet → Route Table → Internet Gateway를 통한 아웃바운드 통신이 완벽하게 동작하는 것입니다.

---

### Step 4. 웹 서버(Nginx) 설치 및 실행하기

> **관련 요구사항**: 웹 서버(Nginx)를 설치·실행하고, 인스턴스 내부에서 `curl http://localhost`가 200 응답을 반환해야 합니다.

여전히 EC2에 SSH로 접속된 상태(`ubuntu@ip-10-0-1-xxx:~$`)에서 진행합니다.

#### 4-1. Nginx 설치 및 상태 확인
```bash
# 1. 우분투 패키지 목록 최신화
sudo apt update

# 2. Nginx 웹 서버 설치 (-y는 중간에 물어보지 않고 바로 설치하라는 옵션)
sudo apt install -y nginx

# 3. Nginx가 잘 켜져 있는지 상태 확인
sudo systemctl status nginx
```
* 초록색 글씨로 **`active (running)`**이 보이면 정상 실행 중입니다. (확인 후 키보드 `q`를 누르면 빠져나옵니다.)

#### 4-2. (선택) 나만의 웹 페이지 및 `/health` 경로 설정하기
기본 Nginx 화면(`Welcome to nginx!`)을 그대로 써도 되지만, 나만의 문구를 넣거나 요구사항의 방식 (B)인 `/health` 헬스체크 경로까지 모두 동작하게 만들면 더욱 완성도가 높아집니다.

```bash
# 1. 기본 웹 페이지(index.html)를 나만의 인사말로 꾸미기 (선택)
echo "<h1>Hello Cloud! Welcome to My First AWS Web Server</h1>" | sudo tee /var/www/html/index.html

# 2. /health 호출 시 200 OK와 함께 "OK"를 반환하도록 헬스체크 파일 생성
echo "OK" | sudo tee /var/www/html/health

# 3. 확장자가 없는 /health 파일을 브라우저에서 열었을 때 다운로드되지 않고 텍스트(OK)로 바로 보이게 설정
sudo sed -i 's/default_type application\/octet-stream;/default_type text\/plain;/' /etc/nginx/nginx.conf
sudo systemctl reload nginx
```

#### 4-3. 인스턴스 내부 응답 확인 (필수 검증)
외부에서 접속하기 전에, 서버 자기 자신(`localhost`)에게 먼저 웹 요청을 보내 200 응답이 오는지 확인합니다.

```bash
curl -i http://localhost
```

**정상 출력 예시:**
```http
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
...
<h1>Hello Cloud! Welcome to My First AWS Web Server</h1>
```

---

### Step 5. 외부 접속 검증하기

> **관련 요구사항**: (A) 브라우저로 `http://<퍼블릭IP>` 접속 또는 (B) `GET http://<퍼블릭IP>/health` 호출 중 1개를 선택해 외부 접속을 증명하고 스크린샷을 남깁니다.

이제 EC2가 아닌 **내 PC(노트북)의 웹 브라우저(크롬, 사파리 등)** 또는 **내 PC의 새 터미널 창**에서 확인합니다.

* **방식 (A) 선택 시 - 웹 브라우저 접속**:
  * 브라우저 주소창에 **`http://<EC2_퍼블릭_IP>`**를 입력합니다.
  * ⚠️ **주의**: 반드시 `https://`가 아닌 **`http://`**로 입력해야 합니다! (아직 443 포트와 SSL 인증서가 없기 때문)
  * 화면에 `Welcome to nginx!` 또는 방금 적은 `Hello Cloud!...` 화면이 뜨면 **주소창의 IP와 화면이 함께 보이도록 스크린샷을 찍어** 프로젝트의 `screenshots/` 폴더에 저장합니다.

* **방식 (B) 선택 시 - `/health` 호출**:
  * 브라우저 주소창에 `http://<EC2_퍼블릭_IP>/health`를 입력하거나, 내 PC 터미널에서 아래 명령어를 실행합니다.
    ```bash
    curl -i http://<EC2_퍼블릭_IP>/health
    ```
  * `HTTP/1.1 200 OK`와 `OK` 응답이 나오는 화면을 캡처하여 `screenshots/` 폴더에 저장합니다.

---

### Step 6. 트러블슈팅(오류 해결) 실습 및 보고서 작성

> **관련 요구사항**: 최소 1건 이상의 문제 해결 사례를 `증상 → 가설 → 검증 → 조치 → 결과 → 재발방지` 흐름으로 `docs/troubleshooting.md`에 기록합니다.

실습 도중 겪은 실제 오류가 있다면 그것을 적는 것이 가장 좋고, 만약 한 번에 성공해서 겪은 오류가 없다면 **현업에서 가장 흔히 발생하는 아래 시나리오 중 하나를 직접 재현해보고 로그와 증상을 눈으로 확인한 뒤 기록**해 보세요!

#### 💡 추천 트러블슈팅 실습 시나리오 3선

##### 시나리오 A: 웹 서버 프로세스 중단으로 인한 `Connection Refused` (로그 기반 분석 연습)
1. **의도적 재현**: EC2에서 `sudo systemctl stop nginx` 명령어로 웹 서버를 꺼봅니다.
2. **증상 확인**: 내 PC에서 `curl -v http://<퍼블릭IP>` 또는 브라우저 접속 시 **`Connection refused`(연결 거부)** 에러가 즉시 발생합니다.
   * *핵심 포인트*: `Timed out`(무한 로딩)은 네트워크/보안그룹에서 막힌 것이고, `Connection refused`는 서버(EC2)까지는 도착했으나 80번 포트를 듣고 있는 프로그램(Nginx)이 꺼져 있을 때 발생합니다!
3. **검증**:
   * EC2 내부에서 `sudo systemctl status nginx` 실행 → `inactive (dead)` 확인
   * `sudo ss -tulpn | grep :80` 실행 → 80번 포트를 리스닝하는 프로세스가 없음을 확인
4. **조치 및 재발방지**:
   * `sudo systemctl start nginx`로 재시작 후 `sudo systemctl enable nginx`(서버 재부팅 시에도 자동 시작) 설정

##### 시나리오 B: 보안 그룹 80번 포트 누락으로 인한 `Connection Timed Out` (네트워크 보안 연습)
1. **증상**: EC2 내부에서 `curl http://localhost`는 `200 OK`가 잘 나오는데, 외부 브라우저에서 `http://<퍼블릭IP>`로 접속하면 화면이 하얗게 돌기만 하다가 **`ERR_CONNECTION_TIMED_OUT`** 발생.
2. **가설**:
   * 가설 1: 퍼블릭 서브넷의 라우팅 테이블에 IGW가 연결되지 않았다.
   * 가설 2: EC2 보안 그룹 인바운드에 HTTP(80) 포트가 열려 있지 않다.
3. **검증**: AWS 콘솔 → EC2 → 보안 그룹(`my-web-sg`) 인바운드 규칙 확인 결과 22번(SSH)만 있고 80번(HTTP) 규칙이 빠져 있음(또는 소스가 내 IP로 잘못 지정됨)을 확인.
4. **조치**: 인바운드 규칙에 `HTTP (TCP 80) / 0.0.0.0/0` 추가 후 즉시 접속 성공 확인.

##### 시나리오 C: SSH 키 파일 권한 오류 (`UNPROTECTED PRIVATE KEY FILE`)
1. **증상**: `ssh -i my-web-key.pem ubuntu@<퍼블릭IP>` 실행 시 `WARNING: UNPROTECTED PRIVATE KEY FILE! Permissions 0644 for 'my-web-key.pem' are too open.` 메시지와 함께 접속 거부(`Permission denied`).
2. **가설 및 검증**: SSH 클라이언트는 개인키 파일이 다른 사용자에게 읽힐 수 있는 권한(`0644`)이면 보안상 접속을 차단함. `ls -l my-web-key.pem`으로 권한이 `-rw-r--r--`임을 확인.
3. **조치**: `chmod 400 my-web-key.pem`으로 소유자 읽기 전용(`-r--------`)으로 변경 후 접속 성공.

---

### Step 7. 최종 제출물 문서 정리하기

미션 제출 규격에 맞춰 프로젝트 폴더 안에 아래 파일들을 작성합니다.

```text
Codyssey-B3-1/
├── README.md                    # 선택한 검증 방식(A/B), 접속 URL/IP, 스크린샷 포함
├── docs/
│   ├── architecture.png         # 아키텍처 다이어그램 이미지 (또는 .pdf)
│   ├── troubleshooting.md       # 트러블슈팅 보고서
│   └── cleanup-checklist.md     # 리소스 정리 체크리스트
└── screenshots/                 # 증빙 스크린샷 이미지 보관 폴더
```

#### 1) `docs/architecture.png` 그리는 팁
* 무료 웹 도구인 **[draw.io (app.diagrams.net)](https://app.diagrams.net/)**에 접속합니다.
* 좌측 도형 검색창에 `AWS`를 검색하거나 기본 사각형 박스를 이용해 다음 포함 관계를 그려 넣고 `PNG` 또는 `PDF`로 내보내기(`File -> Export as -> PNG`) 하여 `docs/architecture.png`로 저장합니다:
  * 가장 큰 박스: **VPC (`10.0.0.0/16`)**
  * VPC 경계에 걸친 문: **Internet Gateway**
  * VPC 내부 박스: **Public Subnet (`10.0.1.0/24`)** 및 연결된 **Route Table (`0.0.0.0/0 -> IGW`)**
  * Subnet 내부 점선 박스: **Security Group (Inbound: 80 `0.0.0.0/0`, 22 `My IP`)**
  * 가장 안쪽: **EC2 인스턴스 (Nginx, Public IP)**
  * 화살표: **외부 사용자(Internet) → Internet Gateway → Public Subnet → Security Group → EC2 (80포트)**

#### 2) `README.md` 작성 템플릿
```markdown
# 내가 만든 웹사이트를 인터넷에 올려 누구나 쓰게 하기 (Codyssey B3-1)

## 1. 아키텍처 다이어그램
![Architecture](./docs/architecture.png)

## 2. 웹 서비스 외부 접속 검증
- **선택한 검증 방식**: (A) 브라우저로 `http://<퍼블릭IP>` 접속
- **접속 주소(URL)**: `http://<본인의_EC2_퍼블릭_IP>`
- **검증 결과 스크린샷**:
  ![Web Access Result](./screenshots/web-access.png)

## 3. 관련 문서 링크
- [트러블슈팅 보고서](./docs/troubleshooting.md)
- [리소스 정리 체크리스트](./docs/cleanup-checklist.md)
```

---

### Step 8. 리소스 정리 및 과금 방지 (필수!)

> 🚨 **과금 방지 필수 안내**  
> **모든 스크린샷 촬영과 문서 작성이 끝났다면 반드시 아래 순서대로 리소스를 삭제하세요!**  
> 클라우드에서는 리소스를 켜두는 시간과 용량만큼 요금이 계산됩니다. 특히 리소스들은 서로 연결(의존성)되어 있어서 **안쪽(EC2)부터 바깥쪽(VPC) 순서로** 지워야 에러 없이 깔끔하게 삭제됩니다.

#### 안전한 리소스 삭제 순서

1. **EC2 인스턴스 종료 (Terminate)**
   * EC2 대시보드 → **[인스턴스]** → `my-web-server` 체크
   * 우측 상단 **[인스턴스 상태(Instance state)]** → **[인스턴스 종료(Terminate instance)]** 클릭!
   * ⚠️ **주의**: `인스턴스 중지(Stop)`가 아니라 반드시 **`인스턴스 종료(Terminate)`**를 눌러야 합니다. '중지'는 컴퓨터 전원만 끈 것이라 하드디스크(EBS) 요금이 계속 부과됩니다.
   * 상태가 `Shutting-down`을 거쳐 **`Terminated(종료됨)`**가 될 때까지 1분 정도 기다립니다.

2. **EBS 볼륨(Volumes) 남아있는지 확인**
   * EC2 왼쪽 메뉴 **[Elastic Block Store] → [볼륨(Volumes)]** 클릭
   * 인스턴스 종료 시 함께 자동 삭제되지만, 혹시 `Available(사용 가능)` 상태로 남아있는 볼륨이 있다면 선택 후 **[작업] → [볼륨 삭제]**를 해줍니다.

3. **Elastic IP (탄력적 IP) 확인**
   * EC2 왼쪽 메뉴 **[네트워크 및 보안] → [탄력적 IP(Elastic IPs)]** 클릭
   * 이번 기본 실습에서는 만들지 않았으므로 비어있는 것이 정상입니다. 만약 따로 할당받은 것이 있다면 선택 후 **[탄력적 IP 주소 릴리스(Release)]**를 해줍니다. *(연결 안 된 탄력적 IP는 시간당 요금이 부과됩니다)*

4. **VPC 및 네트워크 구성요소 삭제 (IGW, Subnet, Route Table, SG)**
   * 상단 검색창에 `VPC`를 쳐서 **VPC 대시보드**로 이동합니다.
   * 왼쪽 메뉴 **[내 VPC(Your VPCs)]** → 내가 만든 **`my-web-vpc`** 체크 → 우측 상단 **[작업(Actions)] → [VPC 삭제(Delete VPC)]** 클릭!
   * 💡 **최신 AWS 콘솔 꿀팁**: 예전에는 IGW 분리 → IGW 삭제 → 서브넷 삭제 → 라우팅 테이블 삭제 → 보안그룹 삭제를 일일이 손으로 해야 했지만, 현재 AWS 콘솔에서는 **`my-web-vpc` 삭제를 누르면 그 안에 속한 서브넷(`my-public-subnet`), 라우팅 테이블(`my-public-rt`), 인터넷 게이트웨이(`my-igw`), 보안 그룹(`my-web-sg`)까지 자동으로 함께 분리 및 삭제**해 줍니다!
   * 삭제 창에 함께 삭제될 리소스 목록이 뜨면 확인 후 하단 입력창에 `delete`를 입력하여 삭제를 완료합니다.
   * *(만약 수동으로 하나씩 지우고 싶다면: `EC2 종료 완료 대기` → `IGW에서 VPC 분리(Detach)` → `IGW 삭제` → `서브넷 삭제` → `라우팅 테이블 삭제` → `보안 그룹 삭제` → `VPC 삭제` 순서로 진행하면 됩니다.)*

5. **Billing 대시보드 확인 및 `docs/cleanup-checklist.md` 작성**
   * 상단 검색창에 `Billing and Cost Management`를 검색해 예상 청구 금액이 `$0.00`인지 확인하고, `docs/cleanup-checklist.md`의 모든 체크박스를 `[x]`로 채웁니다.

---

## 4. 과제 목표 셀프 체크 Q&A (5문 5답)

미션을 마친 뒤 스스로 설명할 수 있어야 하는 5가지 핵심 질문과 모범 답안입니다.

**Q1. VPC, Subnet, Route Table, Internet Gateway는 각각 어떤 역할을 하며 트래픽은 어떻게 흐르나요?**
> * **VPC**는 AWS 내의 격리된 가상 네트워크 공간이고, **Subnet**은 그 안을 나눈 구역입니다.
> * **Internet Gateway(IGW)**는 VPC와 외부 인터넷을 연결하는 출입문이며, **Route Table**은 트래픽이 어디로 갈지 정해주는 이정표입니다.
> * 외부 사용자가 퍼블릭 IP로 요청을 보내면 **Internet Gateway(IGW)**가 이를 EC2의 프라이빗 IP(`10.0.1.x`)로 변환(1:1 NAT)하여 VPC 내부(`local`)의 **Public Subnet** 내 **EC2**로 바로 전달합니다. 이후 EC2가 외부 사용자에게 응답을 돌려보내거나 외부 인터넷으로 나갈 때 서브넷의 **Route Table**에 설정된 `0.0.0.0/0 → IGW` 규칙을 보고 IGW를 통해 인터넷으로 나갑니다.

**Q2. Security Group과 IAM의 역할 차이는 무엇이며, 최소 권한 원칙은 왜/어떻게 적용하나요?**
> * **Security Group**은 인스턴스 레벨에서 허용된 IP와 포트만 통과시키는 **네트워크 방화벽**이고, **IAM**은 AWS 계정 내에서 누가 어떤 AWS 리소스를 조작할 수 있는지 통제하는 **권한 관리 서비스**입니다.
> * **최소 권한 원칙**은 해킹이나 실수로 인한 피해 범위를 최소화하기 위해 꼭 필요한 권한만 부여하는 원칙입니다. Security Group에서는 웹 접속용 80포트만 전체(`0.0.0.0/0`)에 열고 관리용 22포트(SSH)는 내 IP로만 제한했으며, IAM에서는 루트 계정이나 `AdministratorAccess` 대신 실습에 필요한 EC2/VPC 권한만 부여했습니다.

**Q3. 외부 요청이 EC2의 웹 서버까지 도달하기 위해 필요한 설정들을 순서대로 설명해 보세요.**
> 1. VPC에 **Internet Gateway**가 연결(Attached)되어 있어야 합니다.
> 2. EC2가 속한 서브넷의 **Route Table**에 `0.0.0.0/0 → IGW` 경로가 연결되어 있어야 합니다.
> 3. EC2 인스턴스에 공인 **퍼블릭 IP**가 할당되어 있어야 합니다.
> 4. EC2의 **Security Group** 인바운드 규칙에 HTTP(80번 포트)가 `0.0.0.0/0`으로 열려 있어야 합니다.
> 5. EC2 내부에서 웹 서버(**Nginx**)가 실행(`running`) 중이고 80번 포트를 리스닝하고 있어야 합니다.

**Q4. 오류가 발생했을 때 어떻게 원인을 찾고 해결하나요?**
> 막연하게 설정을 다 지우고 다시 만드는 것이 아니라, **증상(에러 메시지)**을 먼저 관찰합니다. 예를 들어 접속 시 `Timed out`이면 네트워크 경로(IGW, 라우팅 테이블, 보안그룹 80포트)를 의심하여 콘솔 설정을 검증하고, `Connection refused`이면 서버까지는 도달했으나 웹 서버 프로세스가 꺼져 있다는 가설을 세워 `systemctl status nginx`와 `/var/log/nginx/error.log` 로그를 확인해 조치합니다.

**Q5. 클라우드에서 과금이 발생하는 대표 요인과 안전한 삭제 순서는 무엇인가요?**
> * 대표 과금 요인은 실행 중인 **EC2 인스턴스**, 삭제되지 않은 **EBS 스토리지 볼륨**, 인스턴스에 연결되지 않고 방치된 **Elastic IP**, 그리고 시간당 요금이 비싼 **NAT Gateway / ALB / RDS** 등입니다.
> * 리소스 간 의존성이 있기 때문에 가장 안쪽에 있는 **EC2 종료(Terminate) → EBS 볼륨/Elastic IP 정리 → Internet Gateway 분리 및 삭제 → Subnet/Route Table/Security Group/VPC 삭제** 순서로 정리해야 에러 없이 삭제할 수 있습니다.

---

## 5. 번외: 보너스 과제 가이드

필수 과제를 모두 마쳤다면, 아래 보너스 과제에 도전해 보세요! *(주의: Step 8에서 리소스를 지우기 전에 진행하거나, 새로 실습하면서 진행하세요)*

---

### 번외 1. 무료 도메인 연결 및 HTTPS(Let's Encrypt) 적용하기

지금까지는 `http://43.201.x.x`처럼 숫자로 된 IP 주소로 접속했고, 브라우저 주소창 옆에 **'주의 요함(보안되지 않음)'** 경고가 떴을 것입니다.  
이 단계에서는 **무료 도메인 주소(예: `my-cloud-web.duckdns.org`)**를 연결하고, 무료 SSL 인증서 기관인 **Let's Encrypt**를 통해 암호화된 **`https://`**(자물쇠 아이콘 🔒)를 적용해 봅니다.

> 📌 **참고**: Let's Encrypt는 IP 주소(`43.201.x.x`)에는 인증서를 발급해주지 않고, 반드시 **도메인 이름**이 있어야 발급해 줍니다.

#### 1단계: DuckDNS에서 1분 만에 무료 서브도메인 만들기
1. [https://www.duckdns.org](https://www.duckdns.org) 에 접속하여 Google 또는 GitHub 계정으로 로그인합니다.
2. `sub domain` 입력창에 원하는 영문 이름(예: `codyssey-web-홍길동`)을 입력하고 **[add domain]**을 클릭합니다.
3. 아래 목록에 생성된 도메인의 **`current ip`** 칸에 **내 EC2의 퍼블릭 IP 주소**를 붙여넣고 **[update ip]**를 클릭합니다.
   * 이제 전 세계 어디서나 `http://<내도메인>.duckdns.org`를 치면 내 EC2 퍼블릭 IP로 연결됩니다!

#### 2단계: AWS 보안 그룹에 HTTPS(443번 포트) 열어주기 (필수!)
HTTPS는 80번 포트가 아니라 **443번 포트**를 사용합니다.
1. AWS 콘솔 → **보안 그룹(`my-web-sg`)** → **[인바운드 규칙 편집]** 클릭
2. **[규칙 추가]** 클릭:
   * **유형**: `HTTPS` (프로토콜 TCP, 포트 `443`)
   * **소스**: `Anywhere-IPv4` (`0.0.0.0/0`)
3. **[규칙 저장]** 클릭

#### 3단계: Nginx 설정에 내 도메인 등록 및 Certbot으로 인증서 발급하기
EC2에 SSH로 접속한 터미널에서 아래 명령어를 순서대로 실행합니다.

```bash
# 1. Certbot 및 Nginx 플러그인 설치
sudo apt update
sudo apt install -y certbot python3-certbot-nginx

# 2. Certbot을 실행하여 SSL 인증서 자동 발급 및 Nginx HTTPS 설정 적용
# (<내도메인> 자리에 본인이 만든 도메인 전체 주소를 넣으세요. 예: codyssey-web-hong.duckdns.org)
sudo certbot --nginx -d <내도메인>.duckdns.org
```

실행 도중 물어보는 질문에 아래와 같이 답합니다:
1. `Enter email address`: 본인 이메일 입력 후 엔터
2. `Please read the Terms of Service... (Y)es/(N)o`: **`Y`** 입력 후 엔터
3. `Would you be willing to share your email... (Y)es/(N)o`: **`N`** 입력 후 엔터
4. `Successfully deployed certificate...` 메시지가 뜨면 성공입니다!

#### 4단계: HTTPS 접속 확인
* 웹 브라우저 주소창에 **`https://<내도메인>.duckdns.org`**로 접속합니다.
* 주소창 옆에 보안 경고가 사라지고 안전하게 연결된 화면이 뜨면 캡처하여 `README.md`에 추가합니다!

---

### 번외 2. Docker 컨테이너로 웹 서비스 배포하기

Step 4에서는 EC2 운영체제(Ubuntu)에 직접 `apt install nginx`로 웹 서버를 설치했습니다. 하지만 실무에서는 서버 환경이 바뀌어도 똑같이 동작하도록 애플리케이션을 **Docker 컨테이너**라는 박스 안에 담아 실행합니다.

```text
                                ┌─────────────────────────────────────────────────┐
                                │ 💻 EC2 인스턴스 (Ubuntu)                         │
                                │                                                 │
🌐 [ 외부 사용자 ] ──(80포트)──▶ │ [호스트 80번 포트] ──(-p 80:80 연결)──▶ ┌──────┴──────────────────────┐ │
  http://<퍼블릭IP>             │                                        │ 🐳 Docker 컨테이너 (80포트) │ │
                                │                                        │    (이미지: nginx:alpine)   │ │
                                │                                        └─────────────────────────────┘ │
                                └─────────────────────────────────────────────────┘
```

#### 1단계: 기존에 켜져 있던 호스트 Nginx 중지하기 (충돌 방지!)
한 컴퓨터에서 80번 포트는 하나의 프로그램만 쓸 수 있습니다. 앞서 Step 4에서 설치한 Ubuntu의 Nginx가 80번 포트를 잡고 있으면 Docker 컨테이너가 80번 포트를 열 수 없으므로 먼저 꺼줍니다.

```bash
# 기존 호스트 Nginx 중지 및 자동 시작 해제
sudo systemctl stop nginx
sudo systemctl disable nginx
```

#### 2단계: EC2에 Docker 설치하기
EC2 SSH 터미널에서 아래 명령어로 Docker를 설치합니다.

```bash
# 1. Docker 설치
sudo apt update
sudo apt install -y docker.io

# 2. Docker 서비스 시작 및 활성화
sudo systemctl start docker
sudo systemctl enable docker

# 3. 매번 sudo를 치지 않아도 되도록 현재 사용자(ubuntu)에게 docker 권한 부여
sudo usermod -aG docker ubuntu

# 4. 변경된 그룹 권한 즉시 적용
newgrp docker

# 5. Docker 설치 확인
docker --version
```

#### 3단계: Docker 컨테이너로 웹 서비스 실행하기 (`/health` 응답 포함)
공식 `nginx:alpine` 이미지를 사용해 컨테이너를 백그라운드(`-d`)로 띄우고, EC2의 80번 포트와 컨테이너의 80번 포트를 연결(`-p 80:80`)합니다.

```bash
# 1. Nginx 컨테이너를 80:80 포트 매핑으로 실행
docker run -d --name my-web-container -p 80:80 nginx:alpine

# 2. (선택) 컨테이너 내부에 나만의 문구 및 /health 파일 넣어주기
docker exec my-web-container sh -c "echo '<h1>Hello from Docker Container on EC2!</h1>' > /usr/share/nginx/html/index.html"
docker exec my-web-container sh -c "echo 'OK' > /usr/share/nginx/html/health"
```

#### 4단계: 보너스 2 필수 검증 및 스크린샷 촬영
보너스 과제 2를 인정받으려면 아래 항목들을 확인하고 **스크린샷 2장 이상**과 함께 `README.md`에 기록해야 합니다.

1. **컨테이너 실행 상태 확인 (스크린샷 1 촬영 필수 📸)**
   ```bash
   docker ps
   ```
   * `STATUS` 열에 **`Up ...`** 상태와 `PORTS` 열에 **`0.0.0.0:80->80/tcp`**가 보이도록 터미널 화면을 캡처합니다.

2. **EC2 내부 응답 검증 (필수)**
   ```bash
   curl -i http://localhost
   curl -i http://localhost/health
   ```
   * `HTTP/1.1 200 OK` 응답이 오는지 확인합니다.

3. **외부 접속 검증 (스크린샷 2 촬영 필수 📸)**
   * 내 PC 브라우저에서 `http://<퍼블릭IP>` 접속 화면 또는 `http://<퍼블릭IP>/health` 호출 결과를 캡처합니다.

4. **`README.md`에 추가할 내용 예시**:
   ```markdown
   ## 보너스 과제: Docker 컨테이너 배포
   - **실행한 컨테이너 이미지명**: `nginx:alpine`
   - **실행 방식(명령어)**: `docker run -d --name my-web-container -p 80:80 nginx:alpine`
   - **포트 매핑 정보**: 호스트(EC2) `80` 포트 → 컨테이너 `80` 포트 (`0.0.0.0:80->80/tcp`)
   - **검증 스크린샷**:
     1. `docker ps` 실행 결과 (Up 상태 확인)
        ![docker ps](./screenshots/docker-ps.png)
     2. 외부 접속 결과
        ![docker web](./screenshots/docker-web.png)
   ```

