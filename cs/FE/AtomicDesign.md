# Atomic Design

Atomic Design은 Brad Frost가 제안한 UI 디자인 시스템 구성 방법론이다. 인터페이스를 작은 구성 요소로 나누는 동시에 이들이 결합된 완성 화면까지 함께 바라보는 사고 모델이다.

React 전용 아키텍처나 반드시 따라야 하는 폴더 규칙은 아니다. CSS와 JavaScript 구조를 직접 정하는 방법도 아니며, 사용하는 기술과 관계없이 UI의 구성 요소와 계층을 이해하기 위한 방법론이다.

Atomic Design은 다음 다섯 단계를 사용한다.

```text
Atoms -> Molecules -> Organisms -> Templates -> Pages
```

이 단계는 atom부터 page까지 순서대로 한 번만 만드는 선형 개발 절차가 아니다. 완성된 page에서 필요한 구성 요소를 찾아 내려갈 수도 있고, 작은 구성 요소를 조합해 화면을 만들어 올라갈 수도 있다.

## Atoms

Atom은 더 분해하면 독립적인 UI 기능이나 의미를 잃는 가장 작은 구성 요소이다.

```tsx
<Button />
<Input />
<Label />
<Icon />
```

```tsx
type ButtonProps = {
  children: React.ReactNode;
  onClick?: () => void;
};

function Button({ children, onClick }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}
```

Atom은 특정 상품이나 주문 같은 비즈니스 의미보다 색상, 크기, 상태, 상호작용처럼 재사용 가능한 UI 규칙을 담당하는 경우가 많다.

HTML 태그 하나가 항상 atom인 것은 아니다. 별도의 재사용 가치나 변경 이유가 없는 모든 `span`과 `div`를 컴포넌트로 추출하면 파일과 props 전달만 늘어날 수 있다.

## Molecules

Molecule은 여러 atom을 조합해 하나의 비교적 단순한 기능을 수행하는 구성 요소이다.

```tsx
function SearchField() {
  return (
    <form>
      <Label htmlFor="keyword">검색어</Label>
      <Input id="keyword" />
      <Button>검색</Button>
    </form>
  );
}
```

`Label`, `Input`, `Button`이 합쳐져 검색이라는 한 가지 기능을 만든다. Atom과 molecule의 차이는 단순히 포함한 태그 수가 아니라 독립적인 기본 요소인지, 여러 요소가 결합해 작은 기능을 수행하는지에 있다.

## Organisms

Organism은 atom과 molecule을 조합한 화면의 독립적인 영역이다.

```tsx
function Header() {
  return (
    <header>
      <Logo />
      <Navigation />
      <SearchField />
      <UserMenu />
    </header>
  );
}
```

다음과 같은 영역을 organism으로 볼 수 있다.

- Header
- ProductList
- ProductDetail
- CheckoutSummary
- CommentSection

Molecule과 organism을 구분하는 절대적인 파일 크기나 코드 줄 수는 없다. 작은 단일 기능인지, 화면에서 독립적인 의미와 책임을 갖는 영역인지가 더 중요한 기준이다.

## Templates

Template은 page를 구성하는 organism의 배치와 콘텐츠 구조를 정의한다. 실제 상품명이나 사용자 정보보다 어떤 영역이 어디에 놓이는지에 초점을 둔다.

```tsx
type ProductTemplateProps = {
  header: React.ReactNode;
  productDetail: React.ReactNode;
  recommendations: React.ReactNode;
};

function ProductTemplate({
  header,
  productDetail,
  recommendations,
}: ProductTemplateProps) {
  return (
    <>
      {header}
      <main>
        {productDetail}
        {recommendations}
      </main>
    </>
  );
}
```

Template은 레이아웃과 콘텐츠 영역을 보여주는 page 수준의 추상 구조이다.

## Pages

Page는 template에 실제 데이터와 구체적인 상태를 적용한 최종 화면이다.

```tsx
function ProductPage() {
  const product = useProduct();

  return (
    <ProductTemplate
      header={<Header />}
      productDetail={<ProductDetail product={product} />}
      recommendations={<RecommendationList />}
    />
  );
}
```

Page에서는 실제 콘텐츠를 사용해 디자인 시스템이 현실적인 상태를 견딜 수 있는지 확인한다.

- 긴 상품명과 큰 이미지
- 품절 상품
- 비어 있는 결과
- 로딩과 오류
- 사용자 권한에 따른 화면 차이

Template이 페이지의 구조를 정의한다면 page는 그 구조에 실제 데이터와 상태를 적용한 구체적인 인스턴스이다.

## 쇼핑몰 예시

```text
Atoms
Button, Input, Label, Icon

Molecules
SearchField, PriceInput, QuantitySelector

Organisms
Header, ProductGrid, FilterPanel

Template
ProductSearchTemplate

Page
실제 검색어와 상품 결과가 들어간 ProductSearchPage
```

## 의존 방향

일반적으로 큰 구성 요소가 작은 구성 요소를 조합한다.

```text
Page
  -> Template
    -> Organism
      -> Molecule
        -> Atom
```

Atom이 특정 page나 organism을 직접 import하면 재사용 가능한 기초 구성 요소가 상위 화면에 의존하게 된다. 이 경우 작은 UI 수정이 특정 페이지 구조에 묶이고 의존 방향도 뒤집힌다.

다만 organism이 반드시 molecule만 사용해야 하는 것은 아니다. Organism이 atom을 직접 조합할 수도 있다. Atomic Design의 단계는 엄격한 컴파일 규칙이 아니라 UI를 이해하기 위한 계층이기 때문이다.

## 장점

- UI를 조합 가능한 단위로 나누고 재사용할 수 있다.
- 디자인과 개발에서 구성 요소를 설명하는 공통 언어를 제공한다.
- 작은 컴포넌트를 독립적으로 테스트하기 쉽다.
- Storybook과 함께 컴포넌트의 상태와 변형을 관리하기 좋다.
- 여러 페이지에서 디자인 규칙과 상호작용을 일관되게 유지할 수 있다.
- 추상적인 구성 요소와 실제 콘텐츠가 들어간 화면을 오가며 검증할 수 있다.

## 한계와 과도한 분리

Atomic Design의 가장 큰 어려움은 분류 기준이 상황에 따라 달라질 수 있다는 점이다.

```text
ProductCard는 molecule인가 organism인가?
Icon을 포함한 Button은 atom인가 molecule인가?
비즈니스 로직이 추가된 SearchField는 어디에 두는가?
```

프로젝트 전체를 다섯 폴더로만 구분하면 다음 문제가 생길 수 있다.

- 서로 관련 없는 컴포넌트가 크기가 비슷하다는 이유로 같은 폴더에 모인다.
- 컴포넌트가 성장할 때마다 molecule과 organism 사이를 이동할 수 있다.
- 이동할 때 import 경로가 자주 변경된다.
- 데이터 요청, 상태 관리, 비즈니스 규칙의 위치를 설명하지 못한다.
- 지나치게 작은 atom 분리로 파일 수와 props 전달이 늘어난다.
- 분류 자체에 시간이 들고 팀마다 판단이 달라질 수 있다.

모든 HTML 요소를 컴포넌트로 만드는 것이 목표가 아니다. 독립적인 의미, 재사용 가능성, 테스트 필요성, 별도로 변경될 이유가 있을 때 분리한다.

또한 `atoms`, `molecules` 같은 이름을 반드시 폴더명으로 사용해야 하는 것은 아니다. 팀이 이해하기 쉬운 `primitives`, `components`, `patterns`, `layouts`, `pages` 같은 용어로 조정할 수 있다.

## Atomic Design과 FSD

Atomic Design과 Feature-Sliced Design은 분류 기준과 해결하려는 문제가 다르다.

| 구분 | Atomic Design | FSD |
| --- | --- | --- |
| 중심 기준 | UI의 조합 단계 | 도메인과 비즈니스 기능 |
| 주요 관심사 | 디자인 시스템과 컴포넌트 재사용 | 애플리케이션 모듈 경계와 의존 방향 |
| 잘 설명하는 것 | Button에서 완성 화면까지의 UI 계층 | 상품, 검색, 인증 같은 기능과 도메인 |
| 부족한 부분 | 데이터 요청과 비즈니스 로직의 위치 | 디자인 시스템 내부의 세밀한 UI 조합 단계 |

Atomic Design만으로는 결제, 인증, 상품 같은 비즈니스 경계를 표현하기 어렵다. FSD는 애플리케이션을 도메인과 기능으로 구분하지만 디자인 시스템 내부의 UI 계층을 반드시 세분화하지는 않는다.

두 방법을 함께 사용한다면 FSD가 애플리케이션의 큰 경계를 담당하고 Atomic Design의 원칙을 공통 UI와 각 기능의 컴포넌트 구성에 적용할 수 있다.

```text
src/
  shared/
    ui/
      Button/
      Input/
      SearchField/
  entities/
    product/
  features/
    search-product/
  widgets/
    product-grid/
  pages/
    product-search/
```

이 구조에서 `shared/ui`는 재사용 가능한 atom과 molecule 성격의 요소를 제공할 수 있다. `entities`, `features`, `widgets`는 도메인과 기능의 경계를 유지하면서 내부 UI를 필요한 크기로 조합한다.

프로젝트 전체를 다시 `atoms/molecules/organisms`로 나누기보다 각 방법론이 잘 설명하는 범위에 적용하는 편이 응집도를 유지하기 쉽다.

## Atomic CSS와의 차이

Atomic Design과 Atomic CSS는 이름은 비슷하지만 다른 개념이다.

- Atomic Design: 작은 UI 구성 요소를 조합해 디자인 시스템과 화면을 만드는 방법
- Atomic CSS: 하나의 작은 스타일 역할을 수행하는 utility class 중심의 접근

Tailwind CSS를 사용한다고 Atomic Design을 적용한 것은 아니며, 두 방법은 함께 사용하거나 별도로 사용할 수 있다.

## 정리

> Atomic Design은 UI를 atoms, molecules, organisms, templates, pages로 바라보는 디자인 시스템의 사고 모델이다. 작은 컴포넌트의 재사용과 완성 화면의 일관성을 함께 다룰 수 있지만, 단계 사이의 분류에는 절대적인 기준이 없고 비즈니스 도메인 구조까지 설명하지는 않는다. 실무에서는 모든 요소를 억지로 분류하기보다 재사용성과 책임을 기준으로 적용하고, FSD 같은 도메인 중심 구조와 역할을 나눌 수 있다.

## 참고

- [Brad Frost: Atomic Design Methodology](https://atomicdesign.bradfrost.com/chapter-2/)
- [Brad Frost: Tools of the Trade](https://atomicdesign.bradfrost.com/chapter-3/)

