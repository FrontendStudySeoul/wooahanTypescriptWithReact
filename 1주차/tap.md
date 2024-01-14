# 1장 - 들어가며

## <span style="color:#fff000">Javascript</span> 생태계에서 <span style="color:#3300ff">Typescript</span>가 나올 수밖에 없었던 이유
- 모든 브라우저 생태계를 고려하기 위해 시스템 표준인 ECMAScript가 탄생하게 되었고, 또한 웹 애플리케이션의 가속화로 인해 [폴리필(Polyfill)과 트랜스파일(transpile)](https://dev.to/hashcode01/transpiler-vs-polyfills-2j8a)이 생기게 되었다.
- 단순히 논문이나 정보 전달의 목적인 정적 웹 페이지를 제공만 하는 것이 아닌, 웹 애플리케이션으로 진화가 되면서 프로덕트 결과물의 규모가 커지고, 그에 따른 개발자 간의 협업 필요성이 증가하게 되었다. 하지만 <span style="color:#fff000">**Javascript는 동적 타입 언어**</span>를 지원하기 때문에 코드 작성에 상당히 관대하다. 이를 해결하기 위해 *JSDoc, propTypes, Dart* 등이 나오게 되었으나, Javascript 스스로가 인터페이스를 기술할 수 있는 언어가 더 필요해졌고 <span style="color:#3300ff">**Typescript가**</span> 탄생하게 되었다.

## <span style="color:#3300ff">Typescript</span> 사용 이유
- Javascript의 슈퍼셋(superset)
- 안정성 보장
- 개발 생산성 향상
- 협업 용이

<br />

# 2장 - 타입
## What is "type"?
프로그래밍에서의 타입은 수학의 집합과 유사하다. 
> *타입은 값이 가질 수 있는 유효한 범위의 집합을 말한다.*

### 자료형 타입
  - `undefined`, `Boolean`, `String`, `Symbol`, `Numeric`(Number와 BigInt), `Object`
### 집합 타입
  - 타입은 자신에게 속하지 않은 타입을 구분한다.
    ```ts
      const num: number = 123;
      type func = (arg: number) => void; // func 함수는 인자값으로 "number" 타입 집합만 받는다.
      func("123"); // 🚨 Error: 'func' only refers to a type, but is being used as a value here
    ```
### 정적 타입(static type)과 동적 타입(dynamic type)
  - 누구나 한 번쯤 봤을 법한 Javascript 에러 메시지!
  ![Javascript Error Top 10](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvaEH1%2FbtqXU4L2wHT%2F6b6GujeuwjimZ6kQpqHY4K%2Fimg.png)
  
  - 정적 타입: 변수 타입이 <span style="color:#fff000">런타임 시점</span>에서 결정
  - 동적 타입: 변수 타입이 <span style="color:#3300ff">컴파일 시점</span>에서 결정

  > 컴파일 타임과 런타임 
  >
  > - 컴파일타임: 기계(컴퓨터, 엔진)가 코드를 이해할 수 있도록 기계어로 변환되는 시점
  > 
  > - 런타임: 컴파일 이후 변환된 파일이 메모리에 적재되어 실행되는 시점

### 강타입과 약타입
  - 암묵적 타입 변환의 여부에 따라 강타입(strongly type)과 약타입(weekly type)으로 분류
    - 강타입
        - 서로 다른 타입을 갖는 값끼리 연산을 시도하면 <u>컴파일러 or 인터프리터에서</u> 에러가 발생
      > ex) Python, Ruby, Typescript
    - 약타입
        - 서로 다른 타입을 갖는 값끼리 연산할 때는 <u>컴파일러 or 인터프리터가 내부적으로 판단해서 특정 값의 타입을 변환하여 연상을 수행 후 값 도출</u>
      > ex) C++, Java, Javascript
      > 암묵적 타입 변환(Implicit coercion/conversion)
  > 
  > - **개발자가 의도적으로 타입을 명시하거나 바꾸지 않았는데도** 컴파일러 or 엔진에 의해서 런타임 시점에 <u>타입이 자동으로 변경</u>되는 것
  ```js
    // Javascript
    console.log("2" - 1); // 1
    console.log(3 + []); // "3"
    console.log(null + 12); // 12
    console.log({}.foo); // undefined
    console.log("bar"/2); // NaN
    
    // Typescript
    /* 🚨 The left-hand side of an arithmetic operation must be of type 'any', 'number', 'bigint' or an enum type */
    console.log("2" - 1); 
  ```

### 컴파일 방식

  사람이 이해할 수 있는 방식(고수준) 언어를 컴퓨터가 이해할 수 있는 기계어(저수준)로 바꾸는 과정
  > ex) Java, C+와 같은 소스 코드 -> 바이너리(binary) 코드가 변환

## 타입 스크립트 시스템
### 타입 애너테이션 방식
  타입 스크립트에선 다른 언어의 애너테이션 방식과는 다르게 `{변수 이름}: {type}`으로 사용한다.
  ```ts
    const isBoolean:boolean = false;
    const num:number = 16;
    const list: number[] = [1,2,3];
    const tuple: [string, number] = ['abc',123];
  ```
  > 타입 애너테이션(type annotation)
  > 
  > 변수나 상수 혹은 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 어떤 타입 값이 저장될 것인지를 컴파일러에 직접 알려주는 문법


### <span style="color:#FF0000">**타이핑 시스템**</span>
  - 구조적 타이핑(Structural typing): 명목적 언어의 특징과 달리 이름으로 구분X, <u>구조로 타입을 구분한다.</u>
  - 구조적 서브타이핑(Structural Subtyping): 객체가 가지고 있는 속성(프로퍼티)를 바탕으로 타입을 구분한다.
  1. 구조적 타이핑
      ```ts
        interface Person { age: number; }
        interface Man { age: number; }

        let person: Person = { age: 30 };
        let man: Man = { age: 23 };

        person = man; // ✅
        man = person; // ✅
      ```
  2. 구조적 서브 타이핑: 서로 다른 객체에도 동일한 속성이 있다면?
      ```ts
        interface Person { age: number; }
        interface Man { name: string; age: number; }

        let person: Person;
        let man: Man = { name: "우아한", age: 23 };
        
        person = man; // ✅ 가능
        ```
  3. 구조적 서브 타이핑: 함수 매개변수의 타입 달라도 동일한 속성이 있다면?
      ```ts
        interface Person { name: string; }
        let man = { name: "우아한", age: 23 }
        const greeting = (person: Person) => { 
          console.log(`안녕하세요 ${person.name} 입니다.`)
        };
        greet(man); // ✅ 가능
      ```
  4. 구조적 서브 타이핑: 타입의 상속(타입의 계층구조로부터 자유로움)
      ```ts
        class Person {
          name: string;
          age: number;

          constructor(name: string, age: number) {
            this.name = name;
            this.age = age
          }
        }

        class Man {
          name: string;
          age: number;
          sleepTime: number;

          constructor(name: string, age: number, sleepTime: number) {
            this.name = name;
            this.age = age;
            this.sleepTime = sleepTime;
          }
        }

        const greeting = (p: Person) => {
            console.log(`안녕하세요 ${p.name} 입니다.`)
        }

        const man = new Man("우아한", 26, 10);
        greeting(man); // ✅ 안녕하세요 우아한 입니다.
      ```

### 구조적 타이핑의 결과
  - 명목적 타이핑
    - 오로지 타입 이름으로 호환한다는 개념이다.
    - 타입의 동일성을 확인하는 과정에서 구조적 타이핑에 비해 안전하며, 객체의 속성을 다른 객체의 속성과 호환되지 않도록하여 안정성을 추구한다.
    > ex) C++, Java
  - 구조적 타이핑
    - 컴파일 타입 체커
    > ex) Typescript
    
  - 덕 타이핑(duck typing)
    - 어떤 함수의 매개변숫값이 올바르게 주어진다면 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다는 개념이다.
    - 런타임 타입 체커
    > ex) Javascript

  > 덕 타이핑과 구조적 타이핑의 차이로 덕 타이핑은 런타임 타입체커, 구조적 타이핑은 컴파일 타입체커를 한다.

### 타입스크립트의 점진적 타입 확인
- 타입스크립트는 점진적 타입 확인 언어이고, 컴파일 타임에 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용하는 방식이다.(동적 검사 수행 가능)
```ts
  function add(x,y) { return x + y;}
  function add(x:any, y:any): any; // 암시적 타입 변환

  const names = ["a", "b"];
  console.log(names[2].toUpperCase()); // 🚨 TypeError: Cannot read property 'toUpperCase' of undefined
```

### 자바스크립트 슈퍼셋으로서의 타입스크립트
![js&ts-superset](https://media.licdn.com/dms/image/C4E12AQGjvH3eTBDdmg/article-cover_image-shrink_600_2000/0/1597593610760?e=2147483647&v=beta&t=rMt8_v4wXHDrrndOaO6L-n9XS4lNFCxFZFEinEYKX20)

```ts
let dev = "woo";
// If Javascript?
console.log(dev.toUppercase()); // 런타임 에러
// If Typescript?
// 🚨 Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
console.log(dev.toUppercase()); // 컴파일 에러 - toUpperCase 제안
```

### 값 vs 타입
- 타입스크립트로 선언한 문법 interface, type은 자바스크립트 런타임에서 제거되기 때문에 값 공간과 타입 공간은 서로 충돌하지 않는다.
- class와 enum

  값과 타입 공간에 동시에 존재하는 symbol
  - class
    ```ts
      class Person {
        age: number;
        constructor(age: number) {
          this.age = age;
        }
      }

      // 💡 Person 클래스를 인스턴스로 값으로 할당이 가능하고, person 값에 Person 클래스를 인터페이스 타입으로 사용이 가능하다.
      const person: Person = new Person({ age: 26 }); 
    ```
  - enum

    런타임에 객체로 변환되는 값 중의 하나인데, enum은 런타임에 실제 객체로 존재하며, 함수로 표현할 수도 있다.
    ```ts
      // typescript
      enum direction {
        UP,
        DOWN,
        LEFT,
        RIGHT
      }
    ```
    ```js
      // javascript
      "use strict";
      var direction;
      (function (direction) {
          direction[direction["UP"] = 0] = "UP";
          direction[direction["DOWN"] = 1] = "DOWN";
          direction[direction["LEFT"] = 2] = "LEFT";
          direction[direction["RIGHT"] = 3] = "RIGHT";
      })(direction || (direction = {}));
    ```
    > 타입스크립트에서 어떤 Symbol이 값으로 사용된다는 것은 **Compiler를 사용해서 타입스크립트 파일을 자바스크립트 파일로 변환해도 여전히 자바스크립트 파일에 해당 정보가 남아있음을 의미**한다.

### 타입 확인하는 방법
- Javascript의 데이터 타입과 Typescript의 타입 차이
  ```ts
    type javascriptType = String | Number | Bigint | Boolean | Object | Symbol | Function | null | undefined;
    type typescriptType = string | number | boolean | undefined | null | void | object | symbol | any | unknown | never;
  ```
- 타입스크립트에선 <span style="color:#fff000">값 공간</span>과 <span style="color:#3300ff">타입 공간</span>이 별도로 존재하는데 typeof 연산ㄷ자도 값과 타입에서 쓰일 때의 역할이 다르다.
  ```ts
    interface Person { lastName: string; }
    const person: Person = { lastName: 'Kim'}

    function email(options: { person: Person; subject: string; }) {}

    // 값에서 사용된 typeof는 자바스크립트 런타임의 typeof 연산자가 된다.
    const v1 = typeof person; // 값은 'object'
    v2 = typeof email; // 값은 'function'

    // 반면 타입에선 tpyeof는 값을 읽고 타입스크립트 타입을 변환한다.
    type T1 = typeof person; // 타입은 Person
    type T2 = typeof email; // 타입은 (options: { person: Person; subject: string; }): => void;
  ```
  `email` 함수는 <u>타입 공간에서 typeof 연산자로 값</u>을 읽을 때 <u>함수의 매개변수 타입과 리턴타입</u>을 포함한 <span style="color:#fff000">함수 시그니처 타입</span>을 반환한다.
  - `instanceof`
  
    자바스크립트에서 instanceof 연산자를 사용하면 <u>프로토타입 체이닝</u> 어딘가에 <u>생성자의 프로토타입 속성</u>이 존재하는지 판단할 수 있다. -> 컴파일 or 런타임 환경 관계없이 타입의 안전성을 가져간다.
    ```ts
      let err = unknown;
      if(err instanceof Error) {
        showError(err.msg);
      } else {
        throw Error(err)
      }
    ```


## 원시 타입
`boolean, undefined, null, number, bigInt, string`
### 원시 값과 원시 래퍼 객체
> 원시 래퍼 객체
> - 타입을 파스칼 표기법으로 표기하면 자바스크립트에서 원시 래퍼 객체라고 부른다.
> - null과 undefined를 제외한 모든 원시 값은 원시 값을 래핑한 객체.
> - 값이 아닌 "객체". 따라서 <span style="color:#3300ff">타입스크립트</span>에서는 <u>내장 원시 타입에 해당하는 타입을 파스칼 표기법으로 쓰지 않도록 주의!</u>

참고) [MDN 원시 값(Primitive values)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Data_structures#%EC%9B%90%EC%8B%9C_%EA%B0%92primitive_values)

### 객체 타입
`object, {}, array, type과 interface 키워드, function`
- `object` 타입은 가급적 권장하지 않는다. `any` 타입관 유사하게 객체에 해당하는 모든 타입 값을 유동적 할당이 가능하여, <u>정적 타이핑에 크게 퇴색</u>된다.
- `{}` 타입의 객체는 <u>비어있는 순수한 객체가 아니다</u>. <span style="color:#fff000">자바스크립트</span> 프로토타입 체이닝으로 `Object` 객체 래퍼에서 제공하는 속성에서는 정상적으로 접근이 가능하다. <span style="color:#3300ff">타입스크립트</span>에서 객체 래퍼를 타입으로 지정할 수 있는 데도 이런 이유 때문에 <u>소문자로 된 타입스크립트 타입 체계를 사용</u>하는 것이 일반적이다.
  ```ts
    let str: string | String = "";
    let obj: object | Object = {};
  ```

## 호출 시그니처
타입 스크립트에서 함수 타입을 정의할 때 사용한다. <u>함수 타입은 매개 변수와 반환 값읠 타입으로 결정</u>된다.
```ts
  type add = (a: number, b: number) => number;
```
> 💡 Info
>
> 자바스크립트에서 함수를 작성할때 function 키워드 or 화살표 함수를 사용하나 타입스크립트에선 화살표 함수로만 호출 시그니처를 정의한다.
