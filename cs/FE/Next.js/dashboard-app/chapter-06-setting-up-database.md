# Chapter 6. Setting Up Your Database

## 학습 목표

Vercel Project에 PostgreSQL을 연결하고 환경 변수로 접속 정보를 관리한 뒤, 초기 데이터를 Seed하고 SQL Query로 연결 상태를 확인한다.

## 배포와 Database 연결

GitHub Repository를 Vercel Project에 연결하면 Production 배포와 Preview 배포를 자동화할 수 있다. Database는 Vercel Marketplace의 PostgreSQL Provider를 Project에 연결해 사용한다.

```text
GitHub Repository
-> Vercel Project 배포
-> PostgreSQL Integration 연결
-> 접속 정보를 환경 변수로 주입
```

Vercel Function과 Database 사이에서는 Page를 Rendering할 때 여러 Query가 오갈 수 있다. 사용자 위치만 기준으로 Database Region을 정하지 않고, Function과 Database를 같거나 가까운 Region에 배치해 Network 왕복 시간을 줄인다.

한국 사용자를 주 대상으로 한다면 가능한 경우 다음처럼 구성할 수 있다.

```text
Vercel Function
-> Seoul (icn1)

PostgreSQL
-> Seoul 또는 가까운 Region
```

## 환경 변수

Database 접속 문자열과 Password는 Source Code에 직접 작성하지 않고 환경 변수로 관리한다.

```text
POSTGRES_URL
POSTGRES_PRISMA_URL
POSTGRES_URL_NON_POOLING
POSTGRES_USER
POSTGRES_HOST
POSTGRES_PASSWORD
POSTGRES_DATABASE
```

실제 값이 들어 있는 `.env`는 `.gitignore`에 포함해 Git에 올리지 않는다. 공개 Repository에는 값 없이 필요한 Key만 보여주는 `.env.example`을 둘 수 있다.

Marketplace Integration은 접속 정보를 Project 환경 변수에 자동으로 추가한다. 같은 이름의 변수가 이미 있으면 충돌할 수 있으므로 기존 Database 연결과 환경 변수를 먼저 확인한다. 여러 Database를 연결할 때는 Custom Prefix를 사용할 수 있지만, Application Code에서 읽는 환경 변수 이름도 함께 맞춰야 한다.

## Seed

Seed는 개발과 학습에 필요한 초기 Table과 Data를 Database에 넣는 작업이다.

```text
users
customers
invoices
revenue
```

실습에서는 `/seed` Route Handler가 SQL로 Table을 만들고 `placeholder-data.ts`의 데이터를 저장했다. 사용자 Password는 평문으로 저장하지 않고 `bcrypt`로 Hashing한다.

Seed Route는 GET 요청만으로 Database를 변경하는 임시 학습용 Endpoint다. 외부에 계속 공개하면 누구나 실행할 수 있고, Seed 구현에 따라 중복 데이터가 생성될 수 있다. 초기 데이터 생성이 끝나면 Route를 제거하거나 실제 운영 환경에서는 별도의 Migration과 권한이 제한된 작업으로 처리한다.

## Query 확인

Database 연결과 Seed 결과를 확인하기 위해 임시 `/query` Route에서 Customer와 Invoice를 JOIN하는 SQL을 실행했다.

```sql
SELECT invoices.amount, customers.name
FROM invoices
JOIN customers ON invoices.customer_id = customers.id
WHERE invoices.amount = 666;
```

JOIN 결과가 반환되면 Application이 환경 변수로 PostgreSQL에 연결되고 Seed Data를 조회할 수 있다는 것을 확인할 수 있다. 확인이 끝난 `/query`도 공개 API로 사용할 목적이 없다면 제거한다.

## 구현에서 확인한 내용

- GitHub Repository를 Vercel Project에 연결했다.
- PostgreSQL Database와 Vercel Function Region 사이의 거리를 확인했다.
- Vercel의 Database 환경 변수를 로컬 `.env`에 적용했다.
- `.env`가 Git에 포함되지 않는 것을 확인했다.
- Seed Route로 초기 Table과 Data를 생성했다.
- Query Route에서 JOIN 결과를 확인했다.
- `ch06: connect and seed postgres database` Commit으로 작업을 구분했다.
- 실습 완료 후 임시 Seed와 Query Route를 제거했다.

## 핵심 정리

> Vercel Marketplace의 PostgreSQL Integration은 Database 접속 정보를 환경 변수로 Project에 주입한다. 실제 Secret이 있는 `.env`는 Git에 올리지 않으며, Function과 Database를 가까운 Region에 배치해 Query의 Network 왕복 시간을 줄인다. Seed와 확인용 Query Route는 초기 설정에는 편리하지만 외부에서 실행 가능한 임시 Endpoint이므로 작업이 끝나면 제거해야 한다.

## 참고 자료

- [Next.js Learn: Setting Up Your Database](https://nextjs.org/learn/dashboard-app/setting-up-your-database)
- [Vercel: Storage on Marketplace](https://vercel.com/docs/marketplace-storage)
- [Vercel: Environment Variables](https://vercel.com/docs/environment-variables)
- [Vercel: Regions](https://vercel.com/docs/regions)
