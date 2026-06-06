# Spring Backend Study

Java/Spring 백엔드 학습 과정에서 단계별로 새로 이해한 개념을 정리한다.

## Stage 12 사용자별 데이터 소유권

날짜: 2026-06-07
분류: Backend / Spring Security / MyBatis / Database
상태: 이해 중

### 질문

로그인한 사용자마다 자신의 StudyLog만 조회, 생성, 수정, 삭제하게 만들려면 어떤 흐름이 필요할까?

### 지금의 답

사용자별 데이터 소유권은 개인 정보 보호와 데이터 격리를 위해 필요하다. `study_logs` 테이블에 `user_id` 컬럼을 두고, 이 값을 `users.id`와 외래키로 연결하면 각 학습 기록이 어느 사용자에게 속하는지 표현할 수 있다.

```sql
FOREIGN KEY (user_id) REFERENCES users(id)
```

이 설정은 `study_logs.user_id`에 들어가는 값이 반드시 `users.id`에 존재하는 값이어야 한다는 뜻이다.

### 생성 흐름

새 StudyLog를 만들 때 `user_id`는 form이나 JSON에서 받지 않는다. 브라우저나 Postman 요청 값은 사용자가 조작할 수 있기 때문이다.

따라서 서버에서 현재 로그인 사용자를 기준으로 userId를 정한다.

```text
Authentication
-> authentication.getName()
-> UserMapper.findByUsername(username)
-> currentUser.getId()
-> StudyLog.userId로 저장
```

### 목록 조회 흐름

목록 조회에서는 항상 현재 로그인 사용자의 데이터만 가져와야 한다.

```sql
WHERE user_id = #{userId}
```

이 조건이 없으면 다른 사용자의 학습 기록까지 화면이나 REST 응답으로 노출될 수 있다.

### 상세, 수정, 삭제 흐름

상세 조회, 수정, 삭제에서는 `id`만 확인하면 부족하다. URL이나 API 요청으로 남의 id를 직접 넣을 수 있기 때문이다.

```sql
WHERE id = #{id}
AND user_id = #{userId}
```

이렇게 두 조건을 같이 걸면 해당 id의 데이터가 존재하더라도 현재 로그인 사용자의 것이 아니면 조회, 수정, 삭제되지 않는다.

그래서 단순 `findById(id)`보다 `findByIdAndUserId(id, userId)`가 더 안전하다.

### 없는 데이터와 남의 데이터

`id`가 실제로 없는 경우와, 존재하지만 남의 데이터인 경우를 굳이 구분해서 알려주지 않는 것이 좋다.

남의 데이터가 존재한다는 사실 자체도 정보가 될 수 있기 때문이다. 따라서 둘 다 같은 예외 흐름으로 처리할 수 있다.

```text
없는 데이터
남의 데이터
-> 둘 다 StudyLogNotFoundException
```

### 화면 제어와 서버 제어

JSP에서 버튼을 숨기는 것은 사용자 경험을 위한 처리다. 하지만 브라우저 개발자 도구, URL 직접 입력, Postman 같은 다른 통로로 요청을 보낼 수 있으므로 실제 보안은 서버와 DB 조건에서 막아야 한다.

```text
버튼 숨김: UX 보조
Security 설정: URL 접근 제어
SQL user_id 조건: 최종 데이터 소유권 검증
```

### 실무적으로 아직 부족한 점

현재 구현은 학습용으로 충분하지만 실무 관점에서는 더 보강할 부분이 있다.

- 관리자 전용 조회 화면과 일반 사용자 화면 정책 분리
- 권한 없는 접근에 대한 전용 에러 페이지
- REST API의 404, 403 응답 정책 구분
- 주요 수정/삭제 작업에 대한 감사 로그
- Service 계층에서 소유권 검증 책임을 더 명확히 분리
- 통합 테스트로 남의 데이터 접근 차단 검증

### 다시 볼 포인트

- 사용자별 데이터 소유권은 `user_id` 컬럼에서 시작한다.
- 생성 시 userId는 클라이언트가 아니라 서버의 현재 로그인 사용자 기준으로 정한다.
- 조회에는 `WHERE user_id = #{userId}`가 필요하다.
- 수정/삭제에는 `WHERE id = #{id} AND user_id = #{userId}`가 필요하다.
- 화면에서 버튼을 숨기는 것보다 서버와 DB에서 막는 것이 더 중요하다.
