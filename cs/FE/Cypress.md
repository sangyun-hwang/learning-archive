# Cypress

## 설치 및 기본 환경 설정

```bash
npm install -D cypress
```

```json
{
  "scripts": {
    "cypress:open": "cypress open"
  }
}
```

Cypress는 실제 브라우저 환경에서 사용자의 흐름에 가까운 방식으로 E2E 테스트를 작성할 수 있는 도구입니다.

## ESLint 설정

Cypress 전용 전역 객체와 권장 규칙을 사용하려면 `eslint-plugin-cypress`를 설정합니다.

```bash
npm install -D eslint-plugin-cypress
```

```json
{
  "extends": ["plugin:cypress/recommended"],
  "env": {
    "cypress/globals": true
  }
}
```

## 환경 변수 설정

`cypress.config.js` 또는 `cypress.config.ts`에서 `baseUrl`을 지정하면 테스트마다 전체 URL을 반복하지 않아도 됩니다.

```js
const { defineConfig } = require("cypress");

module.exports = defineConfig({
  e2e: {
    baseUrl: "http://localhost:3000",
  },
});
```

## Cypress 테스트의 기본 구조

테스트는 보통 준비, 행동, 검증 순서로 작성합니다.

- Preparation: 테스트할 화면에 진입하고 필요한 상태를 준비합니다.
- Action: 사용자가 하는 행동을 실행합니다.
- Assertion: 화면 또는 데이터가 기대한 상태인지 검증합니다.

```js
describe("counter", () => {
  it("increments count", () => {
    cy.visit("/");
    cy.get("[data-cy=count]").should("contain", "0");
    cy.get("[data-cy=increase]").click();
    cy.get("[data-cy=count]").should("contain", "1");
  });
});
```

## 공통 훅

- `before`: 전체 테스트 시작 전 한 번 실행합니다.
- `beforeEach`: 각 테스트 전에 실행합니다.
- `afterEach`: 각 테스트 후에 실행합니다.
- `after`: 전체 테스트 종료 후 한 번 실행합니다.

## 커스텀 커맨드

반복되는 선택자나 행동은 custom command로 분리할 수 있습니다.

```js
Cypress.Commands.add("getByCy", (value) => {
  return cy.get(`[data-cy=${value}]`);
});
```

```js
cy.getByCy("submit").click();
```

## Assertion

`should`와 `and`를 사용해 기대 결과를 검증합니다.

```js
cy.getByCy("submit").should("be.visible").and("not.be.disabled");
cy.url().should("include", "/dashboard");
cy.get("input").should("have.value", "hello");
```

## intercept

`cy.intercept`는 네트워크 요청을 감시하거나 응답을 대체할 때 사용합니다. API 호출보다 먼저 등록해야 안정적으로 동작합니다.

```js
cy.intercept("GET", "/products", {
  fixture: "products.json",
}).as("getProducts");

cy.visit("/");
cy.wait("@getProducts");
cy.getByCy("product-item").should("have.length.greaterThan", 0);
```
