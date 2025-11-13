# ASG

Auto Scaling Group

### 1. Scaling 개념

**수직적 확장 (Scale Up / Down)**

- 인스턴스 1개의 성능을 키움
- vCPU, RAM 증가
- 예: t3.micro → t3.large

**수평적 확장 (Scale Out / In)**

- 인스턴스 **개수를 늘리거나 줄임**
- 예: t3.micro 1개 → 3개
- **ASG는 수평적 확장을 자동으로 수행하는 서비스**

---

### 2. Auto Scaling Group(ASG) 기본 정의

- **EC2 인스턴스 개수를 자동으로 조절하는 서비스**
- 최소/희망/최대 용량 설정
- 여러 AZ에 걸쳐 구성 가능 (리전 안에서만)
- ASG 자체는 무료 → EC2 비용만 청구
- **ELB와 함께 사용하면 고가용성 + 자동 확장 구현 가능**

---

### 3. ASG 주요 구성 요소

- **시작 템플릿(Launch Template)**
    
    → AMI, 인스턴스 타입, 보안그룹 등 인스턴스 기본 설정
    
- **용량 설정**
    - 최소(Min)
    - 희망(Desired)
    - 최대(Max)
- VPC, Subnet, AZ
- 헬스 체크(Health Check)

---

### 4. Scaling 정책(Policy)

**동적(Dynamic) 스케일링**

CloudWatch 지표 기반 자동 확장

1. **Target Tracking Scaling** (가장 추천)
- 특정 지표의 목표값을 유지하도록 자동 조정
- 예: CPU 50% 유지
1. **Step Scaling**
- CloudWatch 경보의 단계(조건)에 따라 단계별 확장
- 예: CPU > 70% → +2
    
    CPU > 90% → +4
    
1. **Simple Scaling**
- 경보 발생 시 일정 수량 증가/감소
- 쿨다운(cooldown) 필요

---

**예측(Predictive) 스케일링**

- AWS 머신러닝 기반
- 과거 패턴 학습 → 미래 트래픽을 미리 예측
- 트래픽 증가하기 전에 미리 Scale-out

---

**예약된 스케줄링(Scheduled Actions)**

- 특정 시간에 확장/축소
- 예: 매일 09시 Scale-out

---

### 5. Scaling 지표(Metrics)

- 평균 CPU 사용률
- ALB 요청 수(RequestCountPerTarget)
- 평균 네트워크 In/Out
- 사용자 정의(CW 커스텀 메트릭)

---

### 6. 인스턴스 유지 관리 정책(Instance Maintenance Policy)

ASG가 노후/비정상 인스턴스 교체 시 **어떤 순서로 시작/종료할지 제어**

**종료 전 시작 (Launch Before Terminate)**

- 새 인스턴스를 **먼저 시작**, 기존을 나중에 종료
- 최대 정상 백분율 = 100~200%
- **가용성 높음**, 비용 잠시 증가

**종료 및 시작 (Terminate and Launch)**

- 기존 인스턴스를 먼저 종료하고 새 인스턴스 시작
- 최소 정상 백분율 = 0~100%
- **비용 절감**, 다만 잠깐 가용성 저하 가능

**사용자 지정**

- 최대/최소 정상 백분율 커스텀 설정

---

### 7. 인스턴스 종료 정책(Termination Policy)

ASG가 **Scale In 시 어떤 인스턴스를 먼저 종료할지** 결정

**가장 우선순위는 AZ 균형 유지!**

**정책 종류**

- **기본(Default)**
    
    OldestLaunchConfiguration → ClosestToNextInstanceHour → Random
    
- **Allocation Strategy 기반 종료**
- **NewestInstance** (가장 최근 생성된 인스턴스 종료)
- **OldestInstance** (가장 오래된 인스턴스 종료)
- **OldestLaunchConfiguration**
- **OldestLaunchTemplate**
- **사용자 지정(Custom)**

---

### 8. 상태 확인(Health Check)

비정상 인스턴스 자동 교체

- **EC2 상태 확인**
- **EBS 상태 확인**
- **ELB 상태 확인**(ALB/NLB 연결 시)

→ 비정상 시 자동으로 terminate + 새로운 인스턴스 생성

### 9. 추가 기능

**스케일링 쿨다운(Cooldown)**

- 스케일링 직후 **추가 조정 잠시 중지**
- CloudWatch 지표 안정화 기다림
- Simple Scaling에서 사용

---

**인스턴스 워밍업(Instance Warm-up)**

- 새 인스턴스가 지표에 반영되기 전까지 대기하는 시간
- Target Tracking/Step Scaling에서 사용
- 잘못 설정하면 과도한 스케일 아웃 발생 가능

---

**수명 주기 후크(Lifecycle Hook)**

- 인스턴스 시작/종료 시 **사용자 정의 작업 수행**
- Lambda, SQS, SNS와 연동
- 예: 시작 전에 보안 패치 설치, 종료 전에 로그 수집

---

**인스턴스 보호(Instance Protection)**

- Scale In 시 특정 인스턴스를 **종료되지 않게 보호**

---

**대기 상태(Standby)**

- ASG 안에 있지만
    - 트래픽도 안 받음
    - 헬스 체크도 안함
- 유지보수용 기능

---

**웜 풀(Warm Pool)**

- 스케일링 대비해 인스턴스를 미리 준비시켜놓는 풀
- 상태:
    - Stopped
    - Hibernate
    - Running
- Scale-out 시 **빠르게 준비된 인스턴스 제공**
