# Learning Archive

React와 Next.js를 중심으로 웹 서비스를 개발하며 필요한 프론트엔드, CS, 백엔드 기초, 클라우드와 데이터 흐름 관련 학습 내용을 주제별로 정리한 아카이브입니다.

프론트엔드 경험을 기반으로 웹 서비스 전반의 구조를 이해하기 위해 학습한 내용을 모아 두는 것이 목적입니다. 매일 작성한 TIL 목록을 보여주기보다, 포트폴리오와 이력서에서 학습 범위와 방향성을 빠르게 확인할 수 있도록 핵심 개념과 실습 기록을 정리합니다.

## Contents

- [Frontend](#frontend)
- [CS](#cs)
- [Backend Basics](#backend-basics)
- [Cloud & Data Basics](#cloud--data-basics)
- [Certificates](#certificates)
- [Practice Projects](#practice-projects)
- [Related Repositories](#related-repositories)

## Frontend

React, Next.js, TypeScript 기반의 프론트엔드 학습과 실습이 가장 중심이 되는 영역입니다.

| Topic | Status | Notes |
| --- | --- | --- |
| JavaScript | Study Notes | [데이터 타입, 함수, 클래스, 클로저, 이벤트 루프, 런타임 구조](cs/FE/Javascript.md) |
| TypeScript | Study Notes | [기본 타입, 제네릭, React를 위한 타입스크립트](cs/FE/Typescript.md) |
| React | Study Notes | [JSX, Virtual DOM, Fiber, 렌더링, 메모이제이션, Effect](cs/FE/React.md) |
| Next.js / SSR | Practice | [Next.js 정리](cs/FE/Next.js/next.js.md), [App Router](cs/FE/Next.js/app-router.md), [SSR Practice](https://github.com/sangyun-hwang/react-ssr-practice-without-nextjs) |
| Testing | Learning | [Cypress 설치, 테스트 구조, 커맨드, Assertion, intercept](cs/FE/Cypress.md) |
| State Management | Practice | [React Query, Redux, RTK 정리](cs/FE/StateManagement.md) |
| HTML / Web | Study Notes | [HTML 시맨틱 태그](cs/FE/HTML.md), [SEO](cs/SEO/SEO.md) |

## CS

웹 프론트엔드와 서비스 구조를 이해하는 데 필요한 브라우저, 네트워크, 보안, 아키텍처 기초를 정리합니다.

| Topic | Status | Notes |
| --- | --- | --- |
| Browser / Runtime | Study Notes | [JavaScript 런타임 구조와 이벤트 루프](cs/FE/Javascript.md#런타임-구조) |
| Network | Study Notes | [라우터, DNS, 네트워크 기초](cs/네트워크/네트워크.md) |
| Security | Study Notes | [웹 보안과 암호화 기본 개념](cs/보안/보안.md) |
| Architecture | Learning | [DDD, EDA, RPC, gRPC](cs/패턴/Pattern.md) |
| Information Processing | Study Notes | [정보처리기사 이론](cs/정보처리기사/정보처리기사_이론.md), [프로그래밍](cs/정보처리기사/정보처리기사_프로그래밍.md) |

더 자세한 목차는 [CS 정리](cs/README.md)에서 관리합니다.

## Backend Basics

백엔드 영역은 “백엔드 개발 가능”을 내세우기보다, 프론트엔드와 연결되는 API 구조, 인증, 데이터 저장, 서버 흐름을 이해하기 위한 기초 학습으로 정리합니다.

| Topic | Status | Notes |
| --- | --- | --- |
| Supabase | Basic Study | [Firebase와 Supabase 비교, Database/Auth/Storage/Realtime 개요](cs/Backend/Supabase.md) |
| WebSocket / Chat | Practice | [react_chat_app](https://github.com/sangyun-hwang/react_chat_app) |
| Java | Learning | 기초 문법과 객체지향 개념을 학습 중 |
| Spring | Planned | Java 기초 학습 이후 확장 예정 |

## Cloud & Data Basics

클라우드와 데이터 분석 도구는 실무 역량으로 내세우기보다, 서비스 구조와 데이터 흐름을 이해하기 위한 기초 학습으로 정리합니다.

| Topic | Status | Notes |
| --- | --- | --- |
| AWS | Basic Study | [Serverless, VPC, EC2, Data Lake, Security Hub 개요](cs/Cloud/AWS.md) |
| Google Cloud | Basic Study | [Cloud Computing, BigQuery, Looker, 데이터 분석 입문](cs/Cloud/GoogleCloud.md) |

## Certificates

| Certificate | Status | Notes |
| --- | --- | --- |
| SQLD | Acquired | SQL과 데이터 모델링 기본 역량 |
| 정보처리기사 | Acquired | [CS 이론](cs/정보처리기사/정보처리기사_이론.md), [프로그래밍 정리](cs/정보처리기사/정보처리기사_프로그래밍.md) |
| 리눅스마스터 2급 | Acquired | [리눅스 마스터 2급 정리](certificate/리눅스%20마스터%202급/README.md) |

## Practice Projects

학습한 개념을 코드로 실험하거나 과제 형태로 구현한 저장소입니다. 실무 성과로 과장하기보다, 개념을 적용해 본 연습 기록으로 정리합니다.

| Project | Focus | Link |
| --- | --- | --- |
| Wanted Pre-onboarding FE Challenge | 인증, Todo CRUD, API 연동 과제 | [Repository](https://github.com/sangyun-hwang/wanted-pre-onboarding-challenge-fe-27) |
| SimpleXClone | Next.js, React Query 기반 SNS 클론 실습 | [Repository](https://github.com/sangyun-hwang/SimpleXClone) |
| React Chat App | WebSocket 기반 채팅 실습 | [Repository](https://github.com/sangyun-hwang/react_chat_app) |
| React SSR Practice | Next.js 없이 SSR 흐름 이해 | [Repository](https://github.com/sangyun-hwang/react-ssr-practice-without-nextjs) |
| Next WebView Practice | Next.js 기반 WebView 실험 | [Repository](https://github.com/sangyun-hwang/next-webview-practice) |
| Monorepo Practice | Turborepo 구조와 패키지 분리 실습 | [Repository](https://github.com/sangyun-hwang/monorepo-practice) |

## Related Repositories

원본 저장소는 그대로 두고, 이 저장소에서는 필요한 학습 내용을 주제별로 병합해서 관리합니다.

- [study-CS](https://github.com/sangyun-hwang/study-CS) - CS 학습 원본
- [study-java](https://github.com/sangyun-hwang/study-java) - Java 기초 학습 원본
- [learn-react](https://github.com/sangyun-hwang/learn-react) - React 학습 원본
- [learn-typescript](https://github.com/sangyun-hwang/learn-typescript) - TypeScript 학습 원본
- [TIL](https://github.com/sangyun-hwang/TIL) - 과거 학습 기록 원본

## Note

이 저장소는 학습 범위와 방향성을 보여주기 위한 아카이브입니다. 임시 메모나 출처별 복사본은 줄이고, 웹 서비스 개발에 필요한 개념을 주제 중심 문서로 계속 정리합니다.
