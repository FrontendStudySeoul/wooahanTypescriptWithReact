<img width="548" alt="image" src="https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/103626175/4a306fce-d2a5-490a-9e2f-fb06e257d7ee"># 1. 웹 개발의 역사
## 웹의 성장
- 웹 사이트에서 웹 어플리케이션으로 성장 -> 사용자가 웹에서 할 수 있는 기능들이 많아지고 웹이 복잡해짐
## 자바스크립트의 한계
### 동적 타입 언어
자바스크립트는 런타임에서 타입을 검사하는 동적타입언어이다. 이에 반대되는 정적타입언어이고 이는 컴파일과정에서 타입을 체크하기 떄문에 초기에 타입에 대한 힌트를 받을 수 있습니다.
### 동적 타이핑 시스템의 한계
동적언어는 개발자의 한계가 발생한다.
```tsx
const sunNumber = (a,b)=>{
    return a+b
  }
```
위 코드애서 만약 a와b에 적절한 타입의 변수가 담기면 제대로 된 값을 리턴하겠지만 그렇지 않은 경우에는 원하는 리턴값을 받을 수 없다.
<br>
위 값에서 sunNumber(1,true),sunNumber(1,false),sunNumber(1,undefined),sunNumber(1,"모승")은 어떤 결과물이 나올까요 ?

### 한계 극복을 위한 해결 방안
1. JSDOC - 주석의 성격이 강해 강제화를 하기 힘들다.
2. propTypes - 리액트에서만 사용 가능한 라이브러리이다.
3. Dart - 새로운 언어임.

### 타입스크립트의 등장
마이크로소프트에서 자바스크립트의 슈퍼셋인 타입스크립트가 나와서 이런 문제들을 해결했다. 타입스크립트로 정적 타이핑 된 결과물을 자바스크립트로 컴파일링 하여 자바스크립트에서 타입에러를 줄일 수 있다.
<br>
그럼 타입스크립트는 컴파일언어일까요 트랜스파일링 언어일까요?


### 트랜스파일러와 컴파일러
1. 컴파일러 : 고수준의 언어를 저수준의 언어로 바꿔주는걸 일반적으로 컴파일러라고 하지만 꼭 고수준의 언어를 저수준의 언어로 바꾸는것만이 컴파일러는 아니고 언어를 바꿔주는걸 컴파일러라고 한다.
```
C → Assembly
Java → bytecode
```

2. 트랜스파일러 : 한 언어로 작성된 코드를 비슷한 수준의 다른 언어로 변환하는것.
```
C ++ → C
ES6 → ES5 (Babel)
typescript / coffeescript → Javascript 
sass / scss → css
```
<img width="563" alt="image" src="https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/103626175/071c1448-68da-400f-8a6a-0afb26b845f5">


# 2. 타입
## 타입이란
### 자료형으로서의 타입
모든 프로그래밍 언어는 변수를 선언하는 것부터 시작한다. <br>
기본적으로 자바스크립트는 7가지의 데이터 타입을 정의한다.
- undefined
- null
- boolean
- string
- symbol
- numeric(number,bigInt)
- Object

왜 배열은 자바스크립트에서 7가지 데이터타입에 포함돼지 않는걸까요 ?
<br>
```
JavaScript는 프로토타입 기반(prototype-based)의 객체 지향 프로그래밍 언어입니다. 이는 JavaScript에서 객체와 상속을 구현하는 방식 중 하나입니다.

JavaScript에서는 모든 객체가 프로토타입(prototype)이라고 불리는 다른 객체를 가지고 있습니다. 프로토타입은 해당 객체의 속성과 메서드를 상속하는 데 사용됩니다.
 배열(Array)은 Array.prototype을 프로토타입으로 가지고 있으며, 배열 인스턴스는 Array.prototype에
 정의된 메서드와 속성을 상속받아 사용할 수 있습니다.
```

## 타입스크립트의 타입 시스템
### 타입 애너테이션 방식
타입 애너테이션이란 변수나 상수 혹은 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 어떤 타입 값이 저장해주는지 컴파일러에 알려주는 방식.
```
const moseungname:string = "moseung";
```

<br>
ESLint의 explicit-function-return-type을 설정해주면 린트과정에서 타입 에너테이션을 강제할 수 있다.

### 구조적 타이핑
타입스크립트는 이름으로 타입을 구분하는 명목적인 타입 언어의 특징과 달리 구조로 타입을 구분한다. 이르 구조적 타이핑이라고 한다.

### 구조적 서브타이핑
타입스크립트의 타입 시스템은 집합으로 이해할 수 있다. 따라서 타입스크립트에서는 특정값이 두개의 타입을 동시에 가질 수 있다.
```tsx
const moseung:Human|Developer = "frontendDeveloper"
```

```tsx
interface Pet{
    name:string
  }

  let cat = {name:"dodhd",age:2};

  function greet(pet:Pet){
    console.log("hello", + pet.name)
  }
```
집합의 개념과 같기 때문에 cat은 Pet이라는 인터페이스의 name프로퍼티를 가지고 있기 때문에 위 코드는 문제없이 실행된다.

### 타입스크립트의 점진적 타입 확인
[타입스크립트 컴파일링 과정](https://github.com/FrontendStudySeoul/EffectiveTypeScript/blob/main/1%EC%A3%BC%EC%B0%A8/hardy.md)

### 값 vs 타입
타입스크립트에서는 enum과 class를 사용가능하며 이를 타입시스템으로 사용 가능하다. <br>
<img width="518" alt="image" src="https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/103626175/206b9c1c-2adc-43ab-b0cc-86f6f23a6838">

#### enum
enum은 타입스크립트에서 타입으로 사용될 수도 있지만, 값으로도 사용 가능하다.
```tsx

  enum Direction {
    up,
    down,
    left,
    right,
  }

  const fifaKey: Direction = 1;

  console.log(Direction.up, fifaKey);
```
위 코드는 as const로도 구현이 가능하다.
<img width="548" alt="image" src="https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/103626175/8e32d2ad-9813-4dc2-b769-8a186fcea509">

## 객체 타입
### 배열
[]형태로 나타내거나 Array타입으로 설정 가능하다.
```tsx
const studyMember:MoseungStudy[] = [...]
```
