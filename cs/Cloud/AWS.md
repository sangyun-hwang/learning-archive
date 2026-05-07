# AWS

## Serverless

서버리스는 서버를 직접 운영하는 부담을 줄이고, 인프라 관리의 상당 부분을 클라우드 제공자에게 맡기는 방식입니다. 개발자는 서버 관리보다 비즈니스 로직과 사용자 기능에 집중할 수 있습니다.

장점은 다음과 같습니다.

- 서버 프로비저닝과 운영 부담 감소
- 사용량 기반 과금
- 트래픽 변화에 따른 확장성
- 작은 기능 단위 배포와 운영

대표 서비스는 AWS Lambda, API Gateway, DynamoDB, S3입니다.

## AWS 핵심 서비스

### Region과 Availability Zone

Region은 AWS 데이터센터가 위치한 지리적 영역입니다. 하나의 Region에는 여러 Availability Zone이 있고, AZ는 물리적으로 분리되어 장애 격리를 돕습니다.

고가용성을 위해 여러 AZ에 리소스를 분산할 수 있습니다.

### VPC

VPC는 AWS 안에서 격리된 가상 네트워크입니다.

- CIDR: 네트워크 주소 범위를 표현합니다.
- Subnet: VPC 안의 네트워크 구획입니다.
- Internet Gateway: VPC와 인터넷을 연결합니다.
- Route Table: 네트워크 트래픽 경로를 결정합니다.
- Public Subnet: 인터넷에서 접근 가능한 리소스를 배치합니다.
- Private Subnet: 외부에서 직접 접근하지 않는 리소스를 배치합니다.
- Security Group: 인스턴스 단위의 가상 방화벽입니다.
- NACL: 서브넷 단위의 네트워크 접근 제어입니다.

### EC2, ELB, Auto Scaling

EC2는 가상 서버 인스턴스입니다. ELB는 트래픽을 여러 인스턴스로 분산하고, Auto Scaling은 부하에 맞춰 인스턴스 수를 조절합니다.

Launch Template은 인스턴스 생성에 필요한 설정을 템플릿화합니다.

## Data Lake

데이터 레이크는 정형, 반정형, 비정형 데이터를 한곳에 저장하고 분석할 수 있도록 구성하는 아키텍처입니다. AWS에서는 S3를 중심 저장소로 두고 Glue, Athena, Redshift, Lake Formation 같은 서비스를 함께 사용할 수 있습니다.

## Security Hub

AWS Security Hub는 여러 보안 서비스와 계정의 보안 상태를 한곳에서 확인하고, 보안 표준 준수 여부와 위협 탐지 결과를 관리하는 서비스입니다.

## 학습 상태

AWS는 현재 서비스별 개념과 웹 애플리케이션 구성 흐름을 학습하는 단계입니다. 실무 운영 경험으로 과장하지 않고, 서버리스와 네트워크 기본 개념을 중심으로 정리합니다.
