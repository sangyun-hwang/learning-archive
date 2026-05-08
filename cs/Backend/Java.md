# Java

Java 기초 문법과 객체지향 개념을 학습하면서, 백엔드 학습에 필요한 핵심 개념을 정리합니다.

## 배열과 ArrayList의 차이

날짜: 2026-05-08
분류: Java
상태: 이해 중

### 질문

`int[] numbers` 같은 배열은 배웠는데, `ArrayList`는 무엇인가?

### 짧은 답

배열은 길이가 고정된 자료구조이고, `ArrayList`는 값을 추가할 때 크기가 자동으로 늘어나는 목록 객체입니다.

### 내가 이해한 내용

배열은 처음 만들 때 칸 수를 정합니다.

```java
int[] numbers = new int[3];

numbers[0] = 10;
numbers[1] = 20;
numbers[2] = 30;
```

위 배열은 기본적으로 3개의 값을 담기 위한 구조입니다. 나중에 값을 계속 추가하는 용도로 쓰기에는 불편합니다.

`ArrayList`는 값을 하나씩 추가하면서 목록을 키울 수 있습니다.

```java
List<String> names = new ArrayList<>();

names.add("Java");
names.add("Spring");
names.add("Database");
```

`StudyLog` 예제에서는 공부 기록 하나가 `StudyLog` 객체이고, 여러 공부 기록은 `List<StudyLog>`로 표현할 수 있습니다.

```java
List<StudyLog> logs = new ArrayList<>();

logs.add(new StudyLog("Java class practice", "JAVA", 60, "field and constructor"));
logs.add(new StudyLog("Java array practice", "JAVA", 40, "array basics"));
```

`List<StudyLog>`는 `StudyLog` 타입의 객체만 담을 수 있는 목록이라는 뜻입니다.

### 다시 볼 포인트

- 배열은 크기가 고정되어 있고, `ArrayList`는 값을 추가하면서 크기를 늘릴 수 있다.
- `List<StudyLog>`의 꺾쇠 안 타입은 이 목록에 들어갈 항목의 타입을 의미한다.
- `logs.add(...)`는 목록에 새 항목을 추가한다.
- `for (StudyLog log : logs)`는 목록 안의 값을 하나씩 꺼내 순회한다.

### 현재 학습 코드와 연결

`spring-backend-study`의 Stage 01에서는 `StudyLog` 객체 여러 개를 `List<StudyLog>`에 담고, 반복문으로 출력하는 연습을 진행했습니다.

```java
for (StudyLog log : logs) {
    log.printInfo();
}
```
