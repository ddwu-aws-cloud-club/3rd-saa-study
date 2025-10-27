# VPC

## 개요

- 리전당 최대 5개의 VPC 생성 가능
- VPC당 할당할 수 있는 CIDR 블록은 총 5개
  - 기본 CIDR과 다른 대역의 보조 CIDR를 할당 가능
    - 기본 CIDR: 10.0.0.0/16 (VPC 생성 시 지정)
    - 보조 CIDR 1: 10.1.0.0/16 (IP 주소 부족 시 추가)
    - 보조 CIDR 2: 172.16.0.0/16 (다른 IP 대역 필요 시)
- CIDR는 최소 `/28` - 최대 `/16`
- 사설 IPv4 범위만 허용
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`
- 다른 VPC, 네트워크와 연결해야 한다면 해당 사설망과 IP 범위가 겹치지 않도록 주의

## 구성 요소

### 서브넷

- VPC 내부의 IPv4 부분 범위
- 범위 내에서 주소 5개는 예약, 예약된 주소 사용 불가
  - `X.X.0.0` 네트워크 주소
  - `X.X.0.1` VPC 라우터용
  - `X.X.0.2` DNS 매핑
  - `X.X.0.3` 예약
  - `X.X.0.255` 브로드캐스트

### 라우팅 테이블

- 특정 IP로 온 트래픽을 특정 대상(local, NAT, IGW 등)으로 전송하도록 설정

### 인터넷 게이트웨이(IGW)

- VPC 리소스의 인터넷 연결 허용
- VPC 내 하나의 IGW만 존재
- IGW 자체는 인터넷 액세스를 허용하지 않음
  - 리소스가 퍼블릭 IP를 가져야 함
  - 라우팅 테이블에 IGW로의 경로가 설정되어 있어야 함

### NAT(Network Address Translation)

- 사설 IP를 하나의 공인 IP로 변환하는 IPv4 전용 NAT
  - IPv6 전용 NAT로는 EIGW가 별도로 존재
- NAT Instance
  - NAT Gateway가 너무 비싸 대안으로 사용(NAT Gateway 사용 권장)
  - 조건
    - EC2 세팅에서 소스/목적지 확인을 비활성화
    - 인스턴스에 탄력적 IP 연결
    - 보안그룹 관리 필요
- NAT Gateway
  - AWS의 관리형 NAT 인스턴스
  - 높은 대역폭, 가용성
  - AZ 종속적
  - 탄력적 IP 자동 할당
  - 보안그룹 관리 불필요

|  | Gateway | Instance |
| --- | --- | --- |
| 가용성 | 자체 복원 가능 | 장애 조치 스크립트 |
| 대역폭 | 45GB 이상 | 사용하는 인스턴스 유형에 따라 다름 |
| 관리 | AWS | 사용자 |
| 보안그룹 | X | O |
| Bastion hosts로 사용 | X | O(NAT + Bastion hosts 결합) |

### Bastion Hosts

- 프라이빗 서브넷에 위치하는 인스턴스에 액세스하기 위해 사용
- Bastion hosts는 퍼블릭 서브넷에 위치, 인터넷 액세스 허용
  - 클라이언트가 Bastion hosts로 퍼블릭 접근
  - Bastion hosts 내 진입 후 프라이빗 인스턴스로 접근

## VPC 보안

### NACL

- 서브넷 수준 트래픽 제어 방화벽
- NACL 규칙 정의
  - 규칙 숫자가 낮을 수록 우선 순위가 높아 먼저 적용
  - 숫자는 1 ~ 32766까지 존재, 보통 100 단위로 적용
  - 가장 마지막 규칙은 `*` , 일치하는 규칙이 없는 모든 요청 거부
- 기본 NACL
  - 새 서브넷 생성 시 할당
  - 모든 인, 아웃바운드 트래픽 허용

### 보안그룹 vs NACL

- 보안그룹은 Stateful: 인바운드, 아웃바운드 중 하나가 허용되면 나머지도 자동 허용
- NACL은 Stateless: 인바운드, 아웃바운드가 각각의 규칙을 따로 확인

|  | 보안그룹 | NACL |
| --- | --- | --- |
| 수준 | 인스턴스 | 서브넷 |
| 규칙 | 허용 | 허용/거부 |
| 상태 | Stateful | Stateless |
| 규칙 평가 | 모든 규칙 평가 | 우선 순위 순서대로 평가 |
| 지정 | 단일 EC2 | 서브넷 내의 모든 EC2 |

### AWS Network Firewall

- VPC 수준 방화벽
- GWLB 기반 서비스
  - Network Firewall 생성 시 내부적으로 GWLB 생성 및 관리

## VPC 통신

### VPC 피어링

- 두 VPC 간 프라이빗 통신 연결
  - 다른 리전, 계정의 VPC와도 통신
  - VPC 간 트래픽이 인터넷을 거치지 않고 AWS 네트워크 내부에서 전송
- 조건
  - 양쪽 VPC에서 피어링 요청 생성 및 수락 필요
  - 두 VPC 간 CIDR이 중복되지 않아야 함(대역이 일부라도 겹쳐서는 안됨)
  - 각 VPC의 라우팅 테이블에 상대 VPC CIDR로 향하는 경로 추가 필요
- 특징
  - 전이되지 않음(VPC A-B, A-C 연결 시 B-C 직접 통신 불가)
  - 다른 계정의 VPC와 피어링 시 보안그룹에서 다른 계정을 참조 가능

### VPC 엔드포인트

- VPC 내 리소스가 인터넷을 거치지 않고 AWS 서비스에 프라이빗 액세스
- 연결구조 비교
  - 기존: EC2(프라이빗) → NAT → IGW → AWS 서비스
  - VPC 엔드포인트: EC2(프라이빗) → VPC Endpoint → AWS 서비스
- 장점
  - 네트워크 아키텍처 단순화
  - 트래픽이 AWS 네트워크 내부에 유지되어 보안 강화
  - NAT Gateway 비용 절감
- VPC 엔드포인트 유형

    |  | 인터페이스 | 게이트웨이 |
    | --- | --- | --- |
    | 프로비저닝 | ENI | 라우팅 테이블 |
    | 설정 | 보안그룹 | 라우팅 테이블 |
    | 대상 | 대부분의 AWS 서비스 | S3, DynamoDB |
    | 가격 | 유료 | 무료 |

- S3 액세스를 위한 엔드포인트 선택
  - 게이트웨이
    - 대부분의 시나리오에서 권장
    - 추가비용 없이 라우팅 테이블만 설정하면 돼서 간편
  - 인터페이스
    - 온프레미스에서 S3 액세스
    - 다른 VPC에 연결

### PrivateLink

- VPC와 특정 서비스 간 프라이빗 연결 제공
  - AWS 서비스로의 프라이빗 액세스
  - 서드파티 SaaS 서비스와의 안전한 연결
  - 다른 AWS 계정의 VPC 내 서비스 노출
- 인터페이스 엔드포인트와 함께 사용됨
  - 인터페이스 엔드포인트 + PrivateLink를 통해 연결
- VPC 피어링과의 차이점
  - PrivateLink: 특정 서비스에 대한 단방향 액세스
  - VPC 피어링: 전체 VPC 간 양방향 네트워크 연결

## VPC 로그

### VPC Flow Logs

- ENI에서 송수신되는 IP 트래픽 정보 캡처
- 로그 수집 레벨
  - VPC 레벨: 전체 VPC의 모든 ENI 트래픽
  - 서브넷 레벨: 특정 서브넷의 모든 ENI 트래픽
  - ENI 레벨: 개별 네트워크 인터페이스 트래픽
- 주요 활용
  - VPC 연결 문제 진단 및 트러블슈팅
  - 네트워크 트래픽 패턴 분석
  - 비정상 트래픽, 포트 스캔 탐지
- 로그 저장 대상: CloudWatch Logs, S3, Kinesis Data Firehose
- 보안그룹, NACL 이슈 파악
  - 요청 들어옴

      | 인바운드 | 아웃바운드 | 이슈 |
      | --- | --- | --- |
      | REJECT |  | NACL / 보안그룹 |
      | ACCEPT | REJECT | NACL |

  - 요청 보냄

      | 인바운드 | 아웃바운드 | 이슈 |
      | --- | --- | --- |
      |  | REJECT | NACL / 보안그룹 |
      | REJECT | ACCEPT | NACL |

### VPC Traffic Mirroring

- ENI의 네트워크 패킷을 실시간으로 복제
- 수집한 트래픽을 ENI 또는 NLB로 전송하여 심층 분석
- 컨텐츠 검사, 위협 탐지, 네트워크 성능 모니터링 등에 사용

## 온프레미스 연결

### Site-to-Site VPN

- VPC와 온프레미스 간 암호화된 IPsec VPN 연결
- 연결 구성 요소
  - Virtual Private Gateway (VGW): VPC 측 VPN 게이트웨이
  - Customer Gateway (CGW): 온프레미스 측 VPN 디바이스
  - 공용 인터넷을 통한 암호화 통신
- AWS VPN Cloudhub
  - 저비용 Hub and Spoke 모델
  - 중앙의 VGW가 hub, 여러 CGW가 spoke로 연결
  - 동적 라우팅 프로토콜(BGP) 사용

### Direct Connect(DX)

- 온프레미스와 AWS 간 전용 물리적 네트워크 연결

### TGW(Transit Gateway)

- 여러 VPC, VPN, Direct Connect를 중앙 허브로써 연결하는 리전별 라우터
- 전이적 피어링 연결 형성
- 특징
  - VPC와 다른 별도의 라우팅 테이블을 가짐
  - AWS에서 유일하게 IP 멀티캐스트 지원
- 설정
  - Attachment(부착): VPC, VPN, DX를 TGW에 연결
  - Associaction(연결): Attachment를 라우팅 테이블에 연결
  - Propagation(전파): 라우팅 정보를 자동으로 전파하여 트래픽 라우팅
