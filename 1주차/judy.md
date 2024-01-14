# 1.1 웹 개발의 역사

- 새로운 버전의 브라우저가 출시되어 자바스크립트의 새로운 기능을 지원하더라도 사용자가 예전 버전의 브라우저를 사용한다면 사용하지 못한다. 이런 문제를 해결하기 위한 기능은 무엇인가?
    - 폴리필(polyfill)
        
        브라우저가 지원하지 않는 코드를 브라우저에서 사용할 수 있도록 변환한 코드 조각 ex) core.js, polyfill.io
        
    - 트랜스파일(transpile)
        
        최신 버전의 코드를 예전 버전의 코드로 변환하는 과정 ex) 바벨(babel)
        

- 웹사이트와 웹 애플리케이션의 차이는 무엇일까?
    - 웹사이트
        
        수집된 데이터 및 정보를 특정 페이지에 표시하기 위한 정적인 웹
        
        단방향으로 정보를 제공, 사용자와 상호작용X
        
        HTML에 링크가 연결된 웹 페이지 모음
        
    - 웹 애플리케이션
        
        사용자와 상호작용하는 쌍방향 소통의 웹 
        
    
- 컴포넌트 베이스 개발(CBD, Component Based Development)
    - 재사용할 수 있는 컴포넌트를 개발 또는 조합해서 하나의 애플리케이션을 만드는 개발 방법론
    - 서비스에서 다루는 데이터를 구분하고 그에 맞는 UI를 표현할 수 있게 컴포넌트 단위로 개발

- 컴포넌트 vs 모듈
    - 컴포넌트
        - UI의 독립적인 단위, 버튼, 헤더, 폼과 같은 UI요소, 재사용 가능한 UI의 조각
        - 모듈과는 달리 런타임 환경에서 독립적으로 배포, 실행될 수 있는 단위
        - 작은 기능을 가진 컴포넌트일수록 다른 컴포넌트에 의존하지 않고 독립적으로 존재할 수 있지만, 조합 과정에서 필연적으로 의존성이 생긴다.
    - 모듈
        - 코드의 묶음, 관련된 기능을 하나의 단위로 그룹화, 함수, 변수 클래스 등을 포함
        - 일반적으로 하나의 파일로 존재
    - 차이점
        - 범위와 목적 : 컴포넌트는 주로 UI와 관련된 범위, 모듈은 프로그래밍 코드의 재사용과 조직을 위한 광범위한 개념
    

# 1.2 자바스크립트의 한계

1. 동적 타입 언어
    - 변수에 타입을 명시적으로 지정하지 않고 코드가 실행되는 런타임에 변수값이 할당될 때 해당 값의 타입에 따라 변수 타입이 결정됨
    - 런타임에 변수값이 할당될 때 의미 → 실제 코드가 동작할 때 변수에 값이 할당되는 순간 결정됨
    
    ```jsx
    let a = "string"
    let b = 1
    ```
    
2. 동적 타이핑 시스템의 한계
    
    ```jsx
    const sum = (a, b) => {
    	return a + b;
    }
    
    sum(100) // NaN
    sum("a", "b") // ab
    ```
    
    (1) : b가 undefined이기에 + 연산자의 피연산자가 될 수 없지만 오류를 발생시키지 않고, b를 적절한 타입인 NaN으로 형변환한 다음 실행을 이어나감 ⇒ 따라서 NaN
    
    (2) : 동적 타입 언어라는 특성 때문에 sum 함수를 호출할 때 사용되는 인수 값에 따라 a와 b의 타입이 결정된다.
    
- 한계 극복을 위한 해결 방안
    - JSDoc : 모듈, 네임스페이스, 클래스, 메서드, 매개변수 등에 대한 API 문서 생성 도구 
    
    ```jsx
    // @ts-check
    
    /**
     * 두 수를 더하는 함수입니다.
     *
     * @param {number} a 첫 번째 숫자
     * @param {number} b 두 번째 숫자
     * @return {number} 두 숫자의 합
     */
    const sum = (a, b) => {
        return a + b;
    }
    
    sum(100); // 에러: 두 번째 인자가 누락됨
    sum("a", "b"); // 에러: 문자열 대신 숫자가 필요함
    ```
    
    - propTypes : 리액트에서 컴포넌트 props의 타입을 검사하기 위해 사용하는 속성, 유효한 값이 전달되었는지 확인할 수 있지만, 전체 애플리케이션의 타입 검사를 하는데는 사용할 수 없다. 
    
    - 자바스크립트
        - 자바스크립트의 슈퍼셋 언어
        
        > 슈퍼셋 언어란?
        > 
        > - 기존 언어에 새로운 기능과 문법을 추가해서 보완하거나 향상하는 것
        > - 기존 언어와 호환되며 일반적으로 컴파일러 등으로 기존 언어 코드로 변환되어 실행됨
        - 개발 생산성 향상, 협업에 유리, 자바스크립트의 점진적 도입 가능
     

# 2.1 타입이란

## 자료형으로서의 타입

컴퓨터 메모리 공간 한정적 → 메모리 효율적으로 저장하기 위해선 해당 메모리 공간을 차지할 값의 크기를 알아야 함 → 값의 크기를 명시하면 컴퓨터가 값을 참조할 때 한 번에 읽을 메모리 크기를 알 수 있어 값을 훼손하지 않고 가져올 수 있음

ex ) 메모리에 숫자 타입 값이 할당되어 있다면 자바스크립트 엔진은 8바이트 단위로 메모리 공간에 저장된 값을 읽어올 것

- 데이터 타입을 사용하면 메모리를 효율적으로 사용할 수 있다?
    - 데이터 타입은 메모리의 관점에서 프로그래밍 언어에서 일반적으로 부르는 타입 개념과 같다.
    - 위의 설명처럼 데이터 타입을 지정하면 메모리 입장에서는 공간의 크기를 알 수 있기에 효율적이다.

## 집합으로서의 타입

```tsx
function double(n: number) {
	return n * 2;
}

double(2);   // 4
double("z"); // Error: Argument of type 'string' is not assignable to parameter fo type 'number'
```

타입은 수학의 집합과 유사하다. 값이 가질 수 있는 유효한 범위의 집합을 말한다. 

타입을 제한하면 타입스크립트 컴파일러는 함수를 호출할 때 호환되는 인자로 호출했는지를 판단한다.

인자를 숫자로 받을거라 강제한 것이 집합의 경계처럼 func() 함수의 인자로 들어갈 수 있는 값을 number 타입의 집합으로 제한하는 것이라고 말한다.

## 정적 타입과 동적 타입

타입을 결정하는 시점에 따라 타입을 정적 타입과 동적 타입으로 분류할 수 있다.

- 정적 타입
    - 변수의 타입이 컴파일 타임에 결정된다.
    - C, 자바, 타입스크립트
    
- 동적 타입
    - 변수의 타입이 런타임에서 결정된다.
    - 파이썬, 자바스크립트
    
- 컴파일 타임
    - 기계가 소스코드를 이해할 수 있도록 기계어로 변환되는 시점
    
- 런타임
    - 컴파일 타임 이후 변환된 파일이 메모리에 적재되어 실행되는 시점

## 강타입과 약타입

- 암묵적 타입 변환
    - 개발자가 의도적으로 타입을 명시하거나 바꾸지 않았는데도 컴파일러 또는 엔진 등에 의해 런타임에 타입이 자동으로 변경되는 것
    - 작성자의 의도와 다르게 동작할 수 있기 때문에 예기치 못한 오류가 발생할 수 있다.
    

암묵적 타입 변환 여부에 따라 타입시스템을 분류

- 강타입 → 암묵적 타입 변환X
    - 서로 다른 타입을 갖는 값끼리 연산을 시도하면 컴파일러 또는 인터프리터에서 에러가 발생
    - 파이썬, 루비, 타입스크립트
    
- 약타입 → 암묵적 타입 변환O
    - 서로 다른 타입을 갖는 값끼리 연산할 때는 컴파일러 또는 인터프리터가 내부적으로 판단해서 특정 값의 타입을 변환하여 연산을 수행한 후 값을 도출한다.
    - C++, 자바, 자바스크립트

- 타입스크립트는 어떤 타입시스템을 사용하는가?
    
    두 가지 타입시스템의 영향을 받고 있다.
    
    1. 어떤 타입을 사용하는지를 컴파일러에 명시적으로 알려줘야 하는 타입 시스템
    2. 자동으로 타입을 추론하는 타입 시스템
    
    ⇒ 개발자는 직접 타입을 명시하거나, 타입스크립트가 타입을 추론하도록 하는 방식 중에 선택할 수 있다.
    

## 컴파일 방식

- 일반적인 컴파일러 : 서로 다른 수준(고수준-저수준) 간의 코드 변환
    
    ex) 자바, C# 등의 고수준 언어로 소스코드를 작성하면, 컴파일러는 컴퓨터가 해석할 수 있는 바이너리 코드로 변환
    
- 타입스크립트의 컴파일 결과물: 인간이 이해할 수 있는 자바스크립트 파일 → 자바스크립트의 컴파일 타입에 런타임 에러를 사전에 잡아내기 위한 목적으로 탄생했기에 컴파일하면 타입이 모두 제거된 자바스크립트 소스코드만 남는다.

# 2.2 타입스크립트의 타입 시스템

## 타입 애너테이션(annotation) 방식

: 변수나 상수 혹은 함수의 인자와 반환 값에 타입을 명시적으로 선언 → 그냥 일반적인 타입스크립트 사용법

## 구조적 타이핑(Structural type System) vs 명목적으로 구체화한 타입시스템(Nominal Reified Type Systems)

- 명목적으로 구체화한 타입시스템
    
    : 타입을 사용하는 프로그래밍 언어에서 값이나 객체는 하나의 구체적인 타입을 가지고 있고 **타입은 이름으로 구분**되며 컴파일 이후에도 남아있다. ex) Java, C
    
    ```tsx
    class Cat {
    	**String** name;
    	public void hit() {}
    }
    ```
    

## 구조적 서브 타이핑(Structural Subtyping)

- **타입스크립트의 타입은 값의 집합으로 생각할 수 있다.** → 타입은 단지 집합에 포함되는 값이고 특정 값은 많은 집합에 포함될 수 있다. → 따라서 특정 값이 string또는 number 타입을 동시에 가질 수 있다.
    - 넓은 타입에 좁은 타입을 할당할 수 있지만, 좁은 타입에 넓은 타입 할당은 불가능하다.
    - 좁은 타입이란 더 구체적인 타입을 의미
- 이처럼 집합으로 나타낼 수 있는 타입스크립트의 타입 시스템을 지탱하고 있는 개념이 구조적 서브 타이핑이라는 것이다.

- **구조적 서브 타이핑 :** 객체가 가지고 있는 속성을 바탕으로 타입을 구분하는 것, 이름이 다른 객체라도 가진 속성이 동일하다면 타입스크립트는 서로 호환이 가능한 동일한 타입으로 여긴다.
    - 서로 다른 두 타입 간 호환성은 오로지 타입 내부 구조에 의해 결정된다.
    - “타입 A가 타입 B의 서브타입이라면 A 타입의 인스턴스는 B 타입이 필요한 곳에 위치할 수 있다.” 에서 서브타입이라는 말이 더 좁은 타입, 구체적인 타입이라는 의미이다.
    
    ```tsx
    interface Pet {
      name: string
    }
    
    interface Cat {  // -> 더 구체적인 타입
      name: string;
      age: number
    }
    
    let pet: Pet
    let cat: Cat
    
    pet = cat
    // Cat은 Pet과 다른 타입으로 선언되어있지만 Pet이 갖고 있는 name이라는 속성을 가지고 있다.
    // 따라서 Cat 타입으로 선언한 cat을 Pet타입으로 선언한 pet에 할당할 수 있다.
    ```
    

## 자바스크립트, 타입스크립트, 덕 타이핑

- 명목적 타이핑
    
    ```java
    class Cat {
    	String name;
    	public void hit() {}
    }
    
    class Arrow {
    	String name;
    	public void hit() {}
    }
    
    Public class Main {
    	public static void main(String[] args) {
    		// error: incompatible types: Cat cannot be converted to Arrow
    		Arrow cat = new Cat();
    		// error: Incompatible types: Arrow cannot be converted to Cat
    		Cat arrow = new Arrow();
    	}
    }
    ```
    
    Cat과 Arrow는 구조가 동일하지만 서로 호환되지 않음 
    
    ⇒ 명목적 타이핑을 채택한 언어에서는 **이름으로 타입을 구분**하기 때문에 구조가 같더라도 이름이 다르면 다른 타입으로 취급된다. 
    
    ⇒ 구조적 타이핑에 비해 좀 더 안전하다. 개발자가 의도한 타입이 아니면 변수에 타입을 명시하는 과정에서 에러를 내뱉기 때문이다. 
    

- 덕 타이핑
    
    : 어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식
    
    - 자바스크립트는 본질적으로 덕 타이핑을 기반으로 한다.
    - 타입스크립트는 이런 동작을 그대로 모델링 → 조금 더 유연함
    - 덕 타이핑은 런타임에 타입 검사, 구조적 타이핑은 컴파일 타임에 타입 체커가 타입을 검사

## 타입스크립트의 점진적 타입 확인

: 타입스크립트에서는 필요에 따라 타입 생략할 수도 있고 타입을 점진적으로 추가할 수도 있다.

타입스크립트는 컴파일 타입에 프로그램의 모든 타입을 알고 있을 때 최상의 결과를 보여준다. 

- any 타입 : 어떤 값이든 할당할 수 있다.
    - 타입스크립트 컴파일 옵션 `noImplicitAny: true` 일 때는 에러가 발생, 변수가 any 타입으로 추론되는 것을 허락하지 않는다.

## 값 vs 타입

값 공간과 타입 공간의 이름은 서로 충돌 X > 타입과 변수를 같은 이름으로 정의 가능 > 타입스크립트 문법인 type으로 선언한 내용은 자바스크립트 런타입에서 제거되기 때문에 값 공간과 타입 공간은 서로 충돌하지 않는다.

```tsx
type Developer = { isWorking: true };
const Developer = { isWorking: true };

type Cat = { name: string; age: number };
const Cat = { jump: true };  
```

타입은 타입 선언(:), 단언(as), 값은 할당 연산자인 =으로 작성한다.

## class

자바스크립트에서 등장한 클래스는 객체 인스턴스를 더욱 쉽게 생성하기 위한 문법 기능으로 실제 동작은 함수와 같다. 동시에 클래스는 타입으로도 사용된다. 즉, 타입스크립트 코드에서 클래스는 값과 타입 공간 모두에 포함될 수 있다.

```tsx
class Developer {
  name: string;
  domain: string;

  constructor(name: string, domain: string) {
    this.name = name;
    this.domain = domain;
  }
}

const me: **Developer** = new **Developer**('zig', 'frontend');
```

변수명 me 뒤에 등장하는 :Developer에서 Developer는 타입에 해당하지만 new 키워드 뒤의 Developer 는 클래스의 생성자 함수인 값으로 동작한다.

타입스크립트에서 클래스는 타입 애너테이션으로 사용할 수 있지만 런타임에서 객체로 변환되어 자바스크립트의 값으로 사용되는 특징을 가지고 있다.

## enum

런타임에 객체로 변환되는 값이다. enum은 런타임에 실제 객체로 존재하며, 함수로 표현할 수도 있다.

- 타입으로 사용될 때

```tsx
enum WeekDays {
	MON = "Mon",
	TUES = "Tues",
	WEDNES = "Wednes",
	THURS = "Thurs",
	FRI = "Fri"
}

type WeekDaysKey = keyof typeof WeekDays;
function printDay(key: WeekDaysKey, message: string) {
	const day = WeekDays[key];
	if (day <= WeekDays.WEDNES) {
		console.log(`It's still ${day}day, ${message}`);
	}
}
printDay("TUES", "wanna go home");
```

WeekDays enum에 keyof typeof 연산자를 사용해서 type WeekDaysKey를 만들어, printDay()함수의 key 인자에 넘겨줄 수 있는 값의 타입을 제한한다.

- 값 공간에서 사용된 경우

```tsx
enum MyColors {
	BLUE = "#0000FF",
	YELLOW = "#FFFF00",
	MINT = "#2AC1BC"
}

function whatMintColor(palette: { MINT: string }) {
	return palette.MINT;
}
```

이 예시에서는 enum은 일반적인 객체처럼 동작한다.

타입스크립트에서 어떠한 심볼이 값으로 사용된다는 것은 컴파일러를 사용해서 타입스크립트 파일을 자바스크립트 파일로 변환해도 여전히 자바스크립트 파일에 해당 정보가 남아있음을 의미한다. 반면 타입으로만 사용되는 요소는 컴파일 이후에 자바스크립트 파일에서 해당 정보가 사라진다.

| 키워드 | 값 | 타입 |
| --- | --- | --- |
| class | Y | Y |
| const, let, var | Y | N |
| enum | Y | Y |
| function | Y | N |
| interface | N | Y |
| type | N | Y |
| namespace | Y | N |

## enum vs 유니온

- enum을 선호하는 입장
    - 유니온 타입은 타입이라 순회가 안되지만 enum은 값이기에 이터러블하다.
    - enum은 정의부를 바꾸면 알아서 사용하는 쪽에서도 변경되니까 편하다. 그렇기에 넓은 범위에 확장해서 써야 한다면 enum을 쓴다. DX 측면에서 선호할 수 있다.
    - 트리쉐이킹이 되지 않기에 번들 사이즈에 영향을 줄 수 있지만 const enum을 사용하면 해결할 수 있다.
- enum을 선호하지 않는 입장
    - enum은 타입을 위한 문법이라기 보다 개발을 위한 문법이라 생각
    - 성능에 영향을 줄 수 있다.
- const enum
    - 폴더에 따로 정의한 enum을 외부에서 전역적으로 참조할 때는 const enum을 사용한다. const enum은 빌드 과정에서 참조 값만 남기기 떄문에 트리 쉐이킹이 된다는 장점이 있다.
    - const enum은 enum과 다르게 직접적인 값으로 치환되기에 전체 네임스페이스에 접근하지 못하고 순회할 수도 없다는 단점이 있다.

> **트리쉐이킹(tree-shaking)**
: 자바스크립트, 타입스크립트에서 사용하지 않는 코드를 삭제하는 방식이다. ES6 이후에는 파일 내 특정 모듈만 임포트하면 해당 모듈을 사용하지 않는 파일 코드는 삭제되어 더 작은 크기의 번들링 파일을 생성할 수 있게 되었다.
> 

## 타입을 확인하는 방법

타입스크립트에서 typeof, instanceof 그리고 타입 단언을 사용해서 타입을 확인할 수 있다. 

- typeof
    - 자바스크립트의 7가지 기본 데이터 타입(Boolean, null, undefined, Number, BigInt, String, Symbol)과 Function(함수), 호스트객체 그리고 object 객체가 될 수 있다.        
    - 타입에서 사용된 typeof는 값을 읽고 타입스크립트 타입을 반환한다.
    - 값에서 사용된 typeof는 자바스트립트 런타임의 typeof 연산자가 된다.
    <img width="491" alt="image" src="https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/70371342/3bca8b4a-6a96-4367-a347-956b5999d006">
    <img width="1036" alt="image" src="https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/70371342/5dea15b1-4588-4df3-b24f-7bd79141e88b">


- instanceof
    - 자바스크립트에서 instanceof 연산자를 사용하면 프로토타입 체이닝 어딘가에 생성자의 프로토타입 속성이 존재하는지 판단할 수 있다.
    - typeof 연산자처럼 instanceof 연산자의 필터링으로 타입이 보장된 상태에서 안전하게 값의 타입을 정제하여 사용할 수 있다.
        
        ```tsx
        let error = unknown;
        
        if (error instanceof Error) {
        	showAlertModal(error.message);
        } else {
        	throw Error(error);
        }
        ```
      
- as
    - 타입스크립트에서는 타입 단언이라 부르는 문법을 사용해 타입을 강제한다 → as
    - 개발자가 해당 값의 타입을 더 잘 파악할 수 있을 때 사용되며 강제 형 변환과 유사한 기능을 제공한다.
    - 컴파일 단계에서는 타입 단언이 형 변환을 강제할 수 있지만 런 타임에서는 효력을 발휘하지 못한다.
 
# 2.3 원시 타입

```tsx
// boolean
const isEmpty: boolean = true

// undefined
let value: string;
console.log(value) // undefined

type Person = {
	name: string;
	job?: string;
}

// null
let value: null | undefined;
console.log(value); // undefined

// undefined는 있을 수도, 없을 수도 있단거고 null은 그냥 대놓고 없단거

// number
const a: number = 10;
const b: number = 1.1;
const c: number = +Infinity
const d: number = NaN

// bigInt
const a: bigInt = BigInt(999999999999);
const b: bigInt = 999999999999n;
// 자바스크립트에서는 가장 큰 수인 Number.MAX_SAFE_INTEGER(2^53-1)를 넘어가는 값을 처리할 수 없었는데
// bigInt를 사용하면 이보다 큰 수를 처리할 수 있다. 
// number 타입과 상호 작용 불가능

// string
const a: string = "hi"

// symbol : 어떤 값과도 중복되지 않는 유일한 값을 생성
const ABC = symbol('title')
const DEF = symbol('title')

console.log(ABC === DEF)   // false

```

- ts-config : `strictNullChecks` 옵션
    - 사용자가 명시적으로 해당 타입에 null이나 undefined를 포함해야만 null과 undefined를 사용할 수 있다.
    - 그렇지 않으면 null과 undefined가 될 수 있는 경우에 타입스크립트 에러가 발생하는데 보통 타입 가드로 null과 undefined가 되는 경우를 걸러낸다.
        
        ```tsx
        if(a != null) // 이런식으로 얕은 비교를 통해 걸러냄
        ```
        
    

# 2.4 객체 타입

- object
    - 원시 타입을 제외한 객체, 배열, 정규 표현식, 함수, 클래스 등 모두 object 타입과 호환된다.
    - object 타입은 any타입과 유사하게 객체에 해당하는 모든 타입 값을 유동적으로 할당할 수 있어 정적 타이핑의 의미가 크게 퇴색되기에 가급적 사용하지 말도록 권장된다.
    
    ```tsx
    function isObject(value: object) {
      return (
        Object.prototype.toString.call(value).replace(/\[|\]|\s|object/g, "") === "Object"
      );
    }
    
    isObject({})
    isObject({name: "KG"})
    isObject([0, 1, 2])
    isObject(new RegExp("object"))
    isObject(function() {console.log(`1`)})
    isObject(class Class {})
    ```
    
- {}
    - 자바스크립트에서 객체 리터럴 방식으로 객체 생성할 때 사용한다.
    - 중관호 안에 객체의 속성 타입을 지정해주는 식으로 사용한다. 타이핑되는 객체가 중괄호 안에서 선언된 구조와 일치해야 한다는 것을 말한다.
    - 빈 객체 타입을 지정할 때는 {}보다 Record<string, never>처럼 사용하는 것이 바람직하다.
    
    ```tsx
    const notice: {title: string; desc: string} = {
    	title: "지원 종료",
    	desc: "왜냐면 귀찮으니까",
    	startAt: "2024.01.13"  // 지정한 타입에 존재하지 않으므로 오류
    }
    ```
    
- array
    - 타입스크립트에서 배열 타입을 선언하는 방식은 Array키워드로 선언하거나 대괄호를 사용해서 선언한는 방법이 있다.
    
    ```tsx
    const getCartList = async (cartId: number[]) => {
    	const res = await CartApi.GET_CART_LIST(cartId);
    	return res.getData();
    }
    
    getCartList([]) // 빈배열도 ㄱㄴ
    getCartList([1001]) 
    getCartList([1001, "1002"]) // string 불가능
    ```
    
- type과 interface
    - object 타입은 잘 사용하지 않는다.
    - 객체 타이핑은 type과 interface로 한다.
    
    > **type이나 interface를 써야 하는 상황**
    > 
    > - 객체 지향적으로 코드를 짤 때, 상속하는 경우에 interface를 사용, extends나 implements
    > - 유니온 타입이나 교차 타입 등 type 정의에서만 쓸 수 있는 기능을 활용할 때 type을 사용
    > - interface는 사이즈가 다른 컴포넌트끼리 같은 속성을 공유하는 기준 인터페이스를 정의하고 확장할 때 사용
    > - props에 Record 형식을 extends할 때 interface로 선언된 변수를 넣으면 에러가 발생해서 type으로 바꿔 넣은 경험 → interface처럼 인텍스 키가 따로 설정되지 않으면 오류 발생
    > - computed value를 써야 했을 때 type 정의를 사용
- function
    - 함수 자체의 타입 작성은 아래처럼 호출 시그니처를 정의하는 방식을 사용하면 된다.
    
    ```tsx
    type add = (a: number, b: number) => number;
    ```
    
    > **호출 시그니처:** 
    타입스크립트에서 함수 타입을 정의할 때 사용하는 문법이다. 함수 타입은 해당 함수가 받는 매개 변수와 반환하는 값의 타입으로 결정된다. 호출 시그니처는 이러한 함수의 매개변수와 반환 값의 타입을 명시하는 역할을 한다.
    >
