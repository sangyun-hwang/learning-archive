# Supabase

## Supabase란?

Supabase는 PostgreSQL을 기반으로 하는 오픈소스 Backend as a Service입니다. Firebase의 대안으로 자주 언급되며, 데이터베이스, 인증, 파일 스토리지, 실시간 기능을 빠르게 붙일 수 있습니다.

## Firebase와 비교

Firebase는 Google이 운영하는 BaaS로, 빠른 앱 개발과 풍부한 SDK, 문서, 커뮤니티가 장점입니다. 반면 NoSQL 중심 구조라 복잡한 관계형 쿼리에는 불편할 수 있고, 플랫폼 종속성과 비용 증가를 고려해야 합니다.

Supabase는 PostgreSQL을 직접 사용한다는 점이 가장 큰 차이입니다.

- PostgreSQL 기반 관계형 데이터 모델을 사용할 수 있습니다.
- SQL, REST API, SDK, GraphQL, 직접 DB 연결 등 여러 접근 방식을 제공합니다.
- 오픈소스라 self-hosting 선택지가 있습니다.
- Firebase보다 생태계와 한국어 자료는 상대적으로 적을 수 있습니다.

## 주요 기능

### Database

PostgreSQL 데이터베이스를 제공하고, 테이블 정의를 기반으로 API를 자동 생성합니다. RLS(Row Level Security)를 통해 사용자별 접근 권한을 제어할 수 있습니다.

### Authentication

이메일, OAuth provider 등을 활용한 인증 기능을 제공합니다. 프론트엔드에서는 로그인 상태와 세션을 SDK로 다룰 수 있습니다.

### Storage

이미지, 문서 같은 파일을 저장하고 버킷 단위로 접근 권한을 설정할 수 있습니다.

### Realtime

데이터베이스 변경 사항을 구독해 실시간 UI를 구성할 수 있습니다. 채팅, 알림, 실시간 대시보드 같은 기능에 활용할 수 있습니다.

## 학습 메모

Supabase는 프론트엔드 프로젝트에서 백엔드 구현 부담을 줄이는 선택지입니다. 다만 인증, RLS, API key 노출 범위를 이해하지 않고 사용하면 보안 문제가 생길 수 있으므로 권한 설계를 먼저 확인해야 합니다.

실습 메모에 있던 프로젝트 URL, anon key, DB password 같은 민감 정보는 아카이브 문서로 가져오지 않았습니다.
