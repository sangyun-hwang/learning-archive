# Amazon Bedrock

## 개념

Amazon Bedrock은 여러 제공사의 기반 모델(Foundation Model)을 AWS 환경에서 호출하고 생성형 AI 애플리케이션을 만들 수 있게 하는 관리형 서비스다. Amazon Nova, Anthropic Claude 같은 모델을 선택해 사용할 수 있다. Bedrock 자체가 하나의 LLM인 것은 아니다. [AWS 공식 개요](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html)

이 문서는 모델 호출과 웹 서비스 연결을 이해하기 위한 기초 학습이다.

## 줄여주는 운영 부담

모델을 직접 운영하면 GPU 등 연산 자원 구성, 모델 배포, 추론 서비스 운영과 용량 확장을 직접 관리해야 한다. Bedrock의 관리형 모델 호출을 사용하면 이런 인프라 운영 부담을 줄일 수 있다.

개발팀은 모델 선택, AWS 권한, 요청 구성, 비용과 서비스 로직을 관리한다. 기존 AWS 권한·운영 체계와 연결하기 편하다는 장점도 있다. 다만 관리형 서비스라고 호출 한도, 지연 시간이나 장애 처리가 사라지는 것은 아니다.

| 방식 | 주로 관리할 범위 |
| --- | --- |
| 모델 직접 운영 | 연산 자원, 모델 배포, 확장과 애플리케이션 로직 |
| 모델 제공사 API 직접 호출 | 제공사별 연동, 인증과 애플리케이션 로직 |
| Bedrock 이용 | AWS 권한·설정, 모델 선택과 애플리케이션 로직 |

모델마다 지원 기능과 응답 특성이 다르므로 모델 이름만 바꾸면 도구 사용이나 품질까지 동일하게 유지된다고 가정하지 않는다.

## 웹 서비스의 요청 흐름

```text
Browser
-> Next Server 또는 Spring Backend
-> Amazon Bedrock API
-> 선택한 모델의 추론 결과
-> Backend에서 결과 처리
-> Browser에 표시
```

예를 들어 상품 문의 챗봇에서는 브라우저가 질문을 백엔드에 보내고, 백엔드가 AWS SDK로 Bedrock을 호출한다. 백엔드는 다음 처리를 담당할 수 있다.

- 서비스 사용자의 인증과 인가
- 입력 길이·형식 검증과 호출량 제한
- AWS 자격 증명 보호와 필요한 모델 호출 권한 관리
- 서비스에 맞는 프롬프트와 자료 구성
- 응답 처리, 오류 대응과 사용량 관찰

AWS 장기 자격 증명을 프론트엔드 번들에 넣지 않는다. AWS에서 실행하는 서버라면 IAM Role 등 실행 환경에 맞는 인증 방식을 사용한다. Bedrock을 연결해도 애플리케이션의 업무 권한 검사는 서버 책임으로 남는다.

## Knowledge Bases와 RAG

Knowledge Bases는 사내 문서 등에서 질문과 관련된 자료를 검색해 답변 생성에 활용하는 RAG 구성을 지원한다. [AWS: Knowledge Bases](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html)

```text
사내 규정에 대한 질문
-> 관련 문서 검색
-> 검색 자료를 모델에 제공
-> 자료를 참고해 답변 생성
```

이 과정은 검색 자료를 요청의 문맥으로 제공하는 방식이다. 모델의 가중치를 변경하는 추가 학습과 구분한다. 문서를 연결했다는 이유만으로 모델이 그 내용을 영구적으로 학습한 것은 아니다.

검색 자료의 품질과 최신성, 사용자가 해당 자료를 볼 권한이 있는지도 확인해야 한다. RAG가 잘못된 답변이나 Prompt Injection을 자동으로 해결하지는 않는다.

## Guardrails

Bedrock Guardrails는 입력과 출력의 유해 콘텐츠, 금지 주제, 민감 정보 등을 정책에 따라 검사하는 기능을 제공한다. [AWS: Guardrails](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html)

민감 정보를 차단하거나 마스킹하는 기능과 '이 사용자가 해당 주문을 환불할 권한이 있는가'를 검사하는 것은 별개다. Guardrails를 설정해도 서버의 인증·인가와 업무 규칙 검증은 필요하다.

## 실습 방향

콘솔에서 모델의 짧은 응답을 확인한 뒤, 백엔드에서 질문을 한 번 호출하고 웹 화면에 연결하는 순서로 시작할 수 있다. 실행 전 선택한 모델과 리전의 사용 가능 여부, 접근 조건과 요금을 확인한다. 이 학습에서는 계정 설정이나 유료 호출을 실행하지 않았다.

## 면접용 설명

Amazon Bedrock은 AWS에서 여러 제공사의 기반 모델을 호출할 수 있는 관리형 서비스입니다. 모델 추론 인프라를 직접 운영하는 부담을 줄이면서 기존 AWS 권한 체계에 연결할 수 있습니다. 웹 서비스에서는 백엔드가 인증과 호출 제한 등을 처리하고 Bedrock을 호출하며, Knowledge Bases로 RAG를 구성하거나 Guardrails로 콘텐츠 정책을 적용할 수 있습니다. 서비스의 권한과 업무 규칙 검증은 별도로 유지해야 합니다.

## 관련 학습

- [AWS 기초](AWS.md)
- [LLM Guardrails](../AI/LLMGuardrails.md)
- [Prompt Injection](../AI/PromptInjection.md)
- [AX와 AI 통합](../AI/AgentExperience.md)

공식 문서 확인 기준: 2026-10-21. 실무 운영이나 직접 호출 경험이 아닌 개념 학습 기록이다.
