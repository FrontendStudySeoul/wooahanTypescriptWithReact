# 들어가며

## 웹 개발의 역사

- 초기의 자바스크립트는 브라우저 생태계를 고려해서 작성된 것이 아님.
- 브라우저와 자바스크립트 간의 호환성 차이가 나서 이런 고민들을 해결해준 것이 제이쿼리.
- 이 과정에서 표준화된 자바스크립트의 필요성이 대두되며 ECMAScript가 탄생.
- 초기의 웹은 정보 제공만을 위한 정적인 웹사이트.
- 웹의 형태가 발전하면서 사용자와 쌍방향 소통을 하는 웹 애플리케이션이 등장.
- 계속된 웹의 발전으로 컴포넌트 베이스 개발과 같은 개발 방법론이 등장
- 자연스럽게 자바스크립트 개발자가 늘어나고 과연 자바스크립트는 유지보수에 좋은 언어인가에 대한 고민이 나옴.

# 타입

## 정적 타입과 동적 타입

- 타입을 결정하는 시점에 따라 분류.
- 정적 타입 시스템 : 모든 변수의 타입이 컴파일 타임에 결정, 컴파일 타임에 타입 에러를 발견할 수 있어 프로그램의 안정성을 보장.
- 동적 타입 시스템 : 변수 타입이 런타임에서 결정되며, 개발자가 직접 타입을 정의해 줄 필요가 없다. 하지만 프로그램을 실행할 때 에러가 발견된다.

## 강타입과 약타입

- 암묵적 타입 변환: 타입을 명시하지 않았는데 런타임에 타입이 자동으로 변경되는 것.

- 강타입 : 서로 다른 타입을 갖는 값끼리 연산을 시도하면 에러가 발생.
- 약타입 : 서로 다른 타입을 갖는 값끼리 연산할 때 내부적으로 판단해서 특정 값의 타입을 변환하여 값을 도출.

```js
console.log('10' + 1); // '101'
console.log(10 * '10'); // 100
```

## 구조적 서브타이핑

- 객체가 가진 속성을 바탕으로 타입 구분
- 유연한 타입 시스템을 구현

### 명목적 타이핑 (타입의 이름만 가지고 구별) vs 구조적 타이핑 (타입의 구조를 가지고 구별)

- 명목적 타이핑이 더 안전.
- 명목적 타입은 유니크한 타입이 있을 수 있지만, 구조적 타입은 유니크한 타입이 존재하지 않을 수 있다.
- 구조적 타이핑을 채택한 이유는 덕 타이핑을 기반으로 하는 자바스크립트를 모델링한 언어이기 때문.

## 값 vs 타입

- 값 공간과 타입 공간은 서로 충돌하지 않기 대문에 타입과 변수를 같은 이름으로 정의 가능
- 값과 타입 공간에 동시에 존재하는 심볼도 있다.
  - 클래스, enum

| 키워드          | 값  | 타입 |
| --------------- | --- | ---- |
| class           | Y   | Y    |
| const, let, var | Y   | N    |
| enum            | Y   | Y    |
| function        | Y   | N    |
| interface       | N   | Y    |
| type            | N   | Y    |
| namespace       | Y   | N    |

### enum vs as const

```ts
export enum Status {
  TODO = 'todo',
  DOING = 'doing',
  DONE = 'done',
}
```

```js
export var Status;
(function (Status) {
  Status['TODO'] = 'todo';
  Status['DOING'] = 'doing';
  Status['DONE'] = 'done';
})(Status || (Status = {}));
```

- rollup 기반 번들러에서 트리쉐이킹이 되지 않는다. (vite에서는 트리쉐이킹 가능)

## 타입을 확인하는 방법

### typeof

- JavaScript에 이미 존재

```ts
function print(value: number | string): string {
  if (typeof value === 'number') {
    return String(value);
  }

  if (typeof value === 'string') {
    return value;
  }

  return value;
}
```

### instanceof

- JavaScript에 이미 존재
- prototype 속성이 객체의 프로토타입 체인 어딘가 존재하는지 판별

```ts
function getDate(date: Date | string): Date {
  if (date instanceof Date) {
    return date;
  }

  return new Date(date);
}
```

### in

- JavaScript에 이미 존재
- javascript에서 객체가 특정 속성을 가지고 있는지 검사를 불리언으로 반환

```ts
interface Dog {
  name: string;
  bark(): '멍멍';
}

interface Cat {
  name: string;
  meow(): '냐옹';
}

function sayAnimal(animal: Dog | Cat) {
  if ('bark' in animal) {
    animal.bark();
    animal.name;
  }

  if ('meow' in animal) {
    animal.meow();
  }
}
```

### 타입 단언

- 개발자가 해당 값을 타입을 더 잘 알때
- 타입스크립트는 일부 불건전한 코드에 대해 허락해준다. (as 활용)
- 어쩔수 없는 경우 제외하고 as를 떡칠하는 경우 any를 쓰는 것과 같다.
- 어쩔 수 없는 경우? -> 이벤트 핸들러만들때, ref만들때?

# 정리

### 타입스크립트와 타입

- 타입스크립트는 타입 구문이 있는 JavaScript
- 객체와 함수가 어떻게 생겼는지를 정의
- 데이터를 코드로 설명할 수 있는 데이터 기술서
