# 1장 들어가며
### 자바스크립트 표준, ECMAScript의 탄생
- 브라우저 별 DOM 구조가 다른 크로스 브라우징 이슈 발생
  - 개발자가 브라우저 별 스크립트를 따로 개발하는 어려움
- 자바스크립트와 브라우저의 발전 속도 간의 차이 발생
    - 자바스크립트의 추가 기능을 런타임 환경인 브라우저에서도 지원해야 함
    - 폴리필, 트랜스파일 개념 등장
        - 폴리필 : 브라우저가 지원하지 않는 코드를 브라우저에서 사용할 수 있도록 변환한 코드 조각이나 플러그인
        - 트랜스파일 : 최신 버전의 코드를 예전 버전의 코드로 변환하는 과정 (ex. babel)
    - 이에 따라, 브라우저 호환성이 필요 없는 JQuery 같은 라이브러리가 유행

### 1장 요약
- 자바스크립트의 역사와 한계
- 타입스크립트가 등장하게 된 배경

# 2장 타입
### ECMA 표준 데이터 타입
- undefined
- null
- Boolean
- String
- Symbol
- Numeric(Number와 BigInt)
- Object

### 집합으로서의 타입

```jsx
const num: number = 123;
const str: string = “abc”;

function func(n: number) {
	// …
}

func(num);
func(str); // type error
```

### 타입 시스템의 기능
코드에서 사용되는 유효한 값의 범위를 제한
런타임에서 발생할 수 있는 유효하지 않은 값에 대한 에러를 방지

### 정적 타입과 동적 타입
타입을 결정하는 시점에 따라 분류

- 정적 타입
    - 모든 변수의 타입이 컴파일 타임에 결정
    - 코드 수준에서 개발자가 타입을 명시해줘야 하는 C, 자바, TS 등이 정적 타입 언어
    - 컴파일 타임에 타입 에러를 발견할 수  있어 프로그램 안정성 보장
- 동적 타입
    - 변수 타입이 런타임에 결정
    - 개발자가 직접 타입을 정의해 줄 필요가 없음
        - 파이썬, JS
    - 프로그램을 실행할 때 타입 에러가 발견
    - 개발 과정에서 에러 없이 코드를 작성할 수 있지만 언제 프로그램에 오류 예측 불가능
    

### 암묵적 타입 변환 (Implicit coercion / conversion)
개발자가 타입을 명시하거나 바꾸지 않았음에도 런타임에 타입이 자동으로 변경되는 것
암묵적 타입 변환 여부에 따라 강타입과 약타입으로 분류
- 강타입 언어 : 파이썬, 루비, 타입스크립트
- 약타입 언어 : C++, 자바, 자바스크립트

### 타입 애너테이션
변수, 상수, 함수의 인자와 반환 값에 타입을 명시적으로 선언
어떤 타입 값이 저장될 것인지를 컴파일러에 직접 알려주는 문법

### 구조적 타이핑
타입스크립트는 이름이 아닌 구조로 타입을 구분
객체가 가지고 있는 속성(프로퍼티)을 바탕으로 타입을 구분하는 것
이름이 다른 객체라도 속성이 동일하다면 타입스크립트는 서로 호환이 가능한 동일 타입으로 판단
별도의 클래스 상속 과정을 거치지 않아도 타입은 계층 구조로부터 자유로움

```jsx
// 집합으로 나타낼 수 있는 TS의 타입 시스템
type stringOrNumber = string | number;
```

```jsx
// 구조적 타이핑
interface Pet {
	name: string;
}
let cat = {name: "Zag", age: 2 };
function greet(pet: Pet) {
	console.log("Hello, " + pet.name);
}

greet(cat); // OK
```

### 덕 타이핑
함수의 매개변수값이 올바르게 주어진다면 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다
자바스크립트의 본질적 기반이자 타입스크립트가 구조적 타이핑을 채택한 이유
- 쉬운 사용성, 안전성 제공
이름으로 타입을 구분하는 명목적 타이핑과 달리 객체가 가진 속성을 기반으로 타입을 검사

### 점진적으로 타입을 확인하는 언어
컴파일 타임에 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용하는 방식
타입을 지정한 변수와 표현식은 정적으로 타입을 검사하지만 타입 선언 생략 시 동적으로 검사 수행
타입 선언을 생략하면 암시적 타입 변환 발생

### any 타입과 noImplicitAny 옵션
타입스크립트 내 모든 타입의 종류를 포함하는 최상위 타입
코드 작성 시 정확한 타이핑을 위해 tsconfig의 noImplicitAny 옵션을 true로 설정하는 것이 좋음
타입 애너테이션이 없을 때 변수가 any 타입으로 추론되는 것을 허용하지 않음

### 값 vs 타입
타입 작성하는 방식
- 타입 선언(:)
- 단언 문(as)
- type, interface 키워드로 커스텀 타입 정의

값과 타입은 동일한 이름으로 정의가 가능하다
type으로 선언한 내용은 자바스크립트 런타임에서 제거
값 공간과 타입 공간은 서로 충돌하지 않음

### 구조 분해 할당
```jsx
// 오류 : 구조 분해 할당 시 오류 발생
function email({ person:
	Person, // subject:
	string, // body:
	string, //
}) {
	// ...
}

function email({ person, subject, body }) {
	// ...
}
```

```jsx
// 해결
// 값-타입 공간을 혼동하는 문제를 해결하기 위해 값과 타입을 구분해서 작성
function email({
	person,
	subject,
	body,
}: {
	person: Person;
	subject: string;
	body: string;
}) {
	// ...
}
```

### Class와 enum
값과 타입 공간에 동시에 존재하는 심볼

| 키워드 | 값 | 타입 |
| --- | --- | --- |
| class | Y | Y |
| const, let, var | Y | N |
| enum | Y | Y |
| function | Y | N |
| interface | N | Y |
| type | N | Y |
| nnamespace | Y | N |

### 타입을 확인하는 방법
- typeof
- instanceof
- in
- 타입 단언

### 2장 요약
- 다른 언어에서의 타입 동작 방식
- 타입스크립트가 제공하는 타입의 종류
- 정적 타입과 동적 타입
- 값 공간과 타입 공간의 구분
