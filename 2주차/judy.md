# 4.1 타입 확장하기


> 💡 새로운 요구사항이 생겨 기존의 인터페이스에 속성을 추가해서 개발을 하느냐, 혹은 기존의 인터페이스를 상속 받는 새로운 타입을 선언해서 구현하느냐 할 때 무분별하게 속성을 추가하여 사용하는 것보다 타입을 확장해서 사용하는 것이 좋다. 적절한 네이밍을 사용해서 타입의 의도를 명확히 표현할 수도 있고, 코드 작성 단계에서 예기치 못한 버그도 예방할 수 있기 때문이다.

## 타입 확장 장점

> 기존 작성한 타입을 바탕으로 타입 확장을 함으로써 불필요한 코드 중복을 줄일 수 있다.
> 
- interface

```tsx
/**
* 메뉴 요소 타입
* 메뉴 이름, 이미지, 할인율, 재고 정보를 담고 있다.
**/
interface BaseMenuItem {
	itemName: string | null;
	itemImageUrl: string | null;
	itemDiscountAmout: number;
	stock: number | null'
}

/**
* 장바구니 요소 타입
* 메뉴 타입에 수량 정보가 추가되었다.
**/
interface BaseCartItem extends BaseMenuItem {
	quantity: number;
}
```

- type

```tsx
type BaseMenuItem = {
	itemName: string | null;
	itemImageUrl: string | null;
	itemDiscountAmout: number;
	stock: number | null'
}

type BaseCartItem = {
	quantity: number;
} & BaseMenuItem;
```

> 확장성 :  요구사항이 늘어날 때마다 새로운 타입을 확장하여 정의할 수 있다.
> 

```tsx
interface EditableCartItem extends BaseCartItem {
	isSoldOut: boolean;
	optionGroups: SelectableOptionGroup[];
}

interface EventCartItem extends BaseCartItem {
	orderable: boolean;
}
```

기존 장바구니 요소에 대한 요구 사항이 변경되어도 BaseCartItem타입만 수정하고 EditableCartItem이나 EventCartItem은 수정하지 않아도 되기 때문에 효율적이다.

## 유니온 타입

합집합개념(or), 유니온 타입으로 선언된 값은 **유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근**할 수 있다.

```tsx
interface CookingStep {
  orderId: string;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

function getDeliveryDistance(step: CookingStep | DeliveryStep) {
  return step.distance;
	// Property 'distance' does not exist on type 'CookingStep | DeliveryStep'.BugFinder: 
  // Property 'distance' does not exist on type 'CookingStep'.
}
```

- 해결 방법
    - **타입 가드(type guard)**
    
    step이 DeliveryStap인스턴스인지 확인하기 위해 in연산자를 사용할 수 있다. 
    
    ```tsx
    function getDeliveryDistance(step: CookingStep | DeliveryStep) {
      if ('distance' in step) {
        // 이 경우 step은 DeliveryStep입니다.
        return step.distance;
      }
    
      return null; // 또는 적절한 오류 처리
    }
    ```
    

## 교차 타입

교집합개념(&), 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 만드는 것으로 이해

```tsx
interface CookingStep {
	orderId: string;
	time: number;
	price: number;
}

interface DeliveryStep {
	orderId: string;
	time: number;
	distance: string;
}

type BaedalProgress = CookingStep & DeliveryStep;

const a:BaedalProgress = {
  orderId: "abc123",
  time: 1,
  price:1000,
  distance:"2190m"
}
```

교차 타입이 두 타입의 모든 가능한 값들을 결합한다는 것이 아니라, 두 타입 모두에 적용 가능한 값을 찾는다는 것이다. 따라서 아래는 두 타입에 모두 적용될 수있는 number타입으로 결정된다. 

```tsx
type IdType = string | number;
type Numeric = number | boolean;

type Universal = IdType & Numeric; // number타입이 된다.
```


>💡 **다시 정리:  타입을 속성의 집합이 아니라 값의 집합으로 생각해야 한다.**
>
>- 타입을 속성의 집합이라 생각하면
>    - 합집합 : 모든 속성을 포함하는 새로운 집합을 상상할 수있다.
>    - 교집합 : 공통적인 속성을 가지고 있는 새로운 집합을 상상할 수있다.
>- **타입을 값의 집합**으로 생각하면
>    - 합집합 : 이 타입 또는 저 타입의 값들을 가질 수 있는, 즉 각각의 속성을 모두 가지는 것이 아니라 두 타입 중 하나의 인스턴스를 가리킬 수있다는 것을 의미. 따라서 유니온 타입의 변수에 접근할 때는 해당 변수가 어떤 타입의 인스턴스인지를 확인하는 추가적인 검사(타입 가드)가 필요하다.
>    - 교집합 : 이 타입과 저 타입의 값들을 둘 다 모두 가져야 하는 인스턴스이다.


## extends와 교차 타입

> 유니온 타입과 교차 타입을 사용한 새로운 타입은 오직 type 키워드로만 선언할 수 있다.
> 

```tsx
interface BaseMenuItem {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

interface BaseCartItem extends BaseMenuItem {
  quantity: number;
}
```

```tsx
type BaseMenuItem = {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

type BaseCartItem = {
  quantity: number;
} & BaseMenuItem;

const baseCartItem: BaseCartItem = {
  itemName: "엽기떡볶이",
  itemImageUrl: "https://www.엽떡사진",
  itemDiscountAmount: 2000,
  stock: 1,
  quantity: 2,
}
```

> 주의할 점: extends 키워드를 사용한 타입이 교차 타입과 100% 상응하지 않는다
> 

```tsx
interface DeliveryTip {
	tip: number;
}

interface Filter extends DeliveryTip {
	tip: string;
	// Interface 'Filter' incorrectly extends interface 'DeliveryTip'.
  // Types of property 'tip' are incompatible.
  // Type 'string' is not assignable to type 'number'.
}
```

DeliveryTip을 extends로 확장한 Filter타입에 string 타입의 속성 tip을 선언하면 tip의 타입이 호환되지 않는다는 에러가 발생한다.

같은 예시를 교차타입으로 작성할 시 에러가 발생하지 않는다. 

```tsx
type DeliveryTip = {
	tip: number;
}

type Filter = DeliveryTip & {
	tip: string;
}

const b: Filter = {
  tip: "d" // 👈 이거 가능?
}
```

<details>
<summary>해설</summary>
    
    
    const b: Filter = {
      tip: "d"
      // Type 'string' is not assignable to type 'never'.
    }
    
    
type 키워드는 교차 타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없기 때문에 선언 시 에러가 발생하지 않는다. 하지만 tip이라는 같은 속성에 대해 서로 호환되지 않는 타입이 선언되어 결국 never타입이 된 것이다. 
실제로 어떠한 값도 할당할 수 없다. 이는 타입스크립트의 타입 시스템이 타입의 안정성과 일관성을 유지하기 위한 메커니즘의 일부이다. 
</details>    


# 4.2 타입 좁히기 - 타입 가드

>💡 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정을 말한다.
>
>타입 좁히기를 통해 더 정확하고 명시적인 타입 추론을 할 수 있게 되고, 복잡한 타입을 작은 범위로 축소하여 타입 안정성을 높일 수 있다.



## 타입 가드에 따라 분기 처리하기

타입스크립트에서의 분기 처리는 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 상황에 따라 다른 동작을 수행하는 것을 말한다. 

여러 타입을 할당할 수 있다는 것은 변수가 유니온 타입 또는 any 타입 등 여러 가지 타입을 받을 수 있다는 것을 말하는데 조건으로 검사하려는 타입보다 넓은 범위를 갖고 있다.

타입 가드는 크게 **자바스크립트 연산자를 사용한 타입 가드**와 **사용자 정의 타입 가드**로 구분할 수 있다.

- **자바스크립트 연산자를 활용한 타입 가드**

: **typeof, instanceof, in과 같은 연산자를 사용해서 제어문으로 특정 타입 값을 가질 수 밖에 없는 상황을 유도**하여 자연스럽게 타입을 좁히는 방식이다.

자바스크립트 연산자를 사용하는 이유는 런타임에 유효한 타입 가드를 만들기 위해서다. 런타임에 유효하다는 말은 타입스크립트 뿐만 아니라 자바스크립트에서도 사용할 수 있는 문법이어야 한다. 

- **사용자 정의 타입 가드**

: **사용자가 직접 어떤 타입으로 값을 좁힐지를 직접 지정하는 방식**이다. 


## 원시 타입을 추론할 때: typeof 연산자 활용하기

자바스크립트 타입 시스템만 대응할 수 있다. 주로 원시 타입 좁히는 용도로만 사용할 것을 권장.

- string, number, boolean, undefined, object, function, bigint, symbol

```tsx
const replaceHyphen: (date: string | Date) => string | Date = (date) => {
  if (typeof date === "string") {
    // 이 분기에서는 date의 타입이 string으로 추론된다.
    return date.replace(/-/g, "/");
  }
  return date;
}
```

## 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기

인스턴스화된 객체 타입을 판별하는 타입 가드로 사용할 수 있다. `A instance of B` 형태로 사용하며 A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어간다. 

`instanceof`는 A의 프로토타입 체인에 생성자 B가 존재하는지를 검사해서 존재한다면 true, 그렇지 않다면 false를 반환한다. → A의 프로토타입 속성 변화에 따라 instanceof 연산자의 결과가 달라질 수 있다는 점은 유의해야 한다. 

아래 예시는 HTMLInputElement에 존재하는 blur 메서드를 사용하기 위해서, event.target이 HTMLInputElement의 인스턴스인지를 검사한 후 분기 처리하는 로직이 나타나 있다.

```tsx
const onKeyDown = (e: React.KeyboardEvent) => {
	if (**e.target instanceof HTMLInputElement** && e.key === "Enter") {
		// 이 분기에서는 e.target의 타입이 HTMLInputElement이며
		// event.key가 'Enter'이다.
		e.target.blur();
		onCTAButtonClick(e);
	}
}
```

## 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

`A in B`의 형태로 사용하는데 이름 그대로 A라는 속성이 B 객체에 존재하는지를 검사한다. 프로토타입 체인으로 접근 가능한 속성이면 전부 true를 반환한다. 

```tsx
interface BasicNoticeDialogProps {
  noticeTitle: string;
  noticeBody: string;
}

interface NoticeDialogWithCookieProps extends BasicNoticeDialogProps {
  cookieKey: string;
  noForADay?: boolean;
  neverAgain?: boolean;
}

export type NoticeDialogProps = 
	BasicNoticeDialogProps 
	| NoticeDialogWithCookieProps;
```

NoticeDialog 컴포넌트가 props로 받는 객체 타입이 BasicNoticeDialogProps인지 NoticeDialogWithCookieProps 인지에 따라 렌더링하는 컴포넌트가 달라지도록 하고 싶을때 props 타입에 따라 렌더링하는 컴포넌트를 분기 처리하면 된다. 

```tsx
const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
	if ("cookieKey" in props) return <NoticeDialogWithCookie {...props} />;
	return <NoticeDialogBase {...props} />;
};
```

if문 스코프에서 타입스크립트는 props 객체를 cookieKey 속성을 갖는 객체 타입인 NoticeDialogWithCookieProps로 해석한다.

## is 연산자로 사용자 정의 타입 가드 만들어 활용하기: 타입 명제

타입 명제는 `A is B` 형식으로 작성하면 되는데 여기서 A는 매개변수 이름이고 B는 타입이다. 

참/거짓의 진릿값을 반환하면서 **반환 타입을 타입 명제로 지정하게 되면 반환 값이 참일 때 A 매개변수의 타입을 B타입으로 취급**하게 된다. 

```tsx
const isDestinationCode = (x: string): x is DestinationCode => 
	destinationCodeList.includes(x);
```

isDestinationCode는 string 타입의 매개변수가 destinationCodeList 배열의 원소 중 하나인지를 검사하여 boolean을 반환하는 함수이다. 

함수의 반환 값을 boolean이 아닌 `x is DestinationCode`로 타이핑하여 타입스크립트에게 이 함수가 사용되는 곳의 타입을 추론할 때 해당 조건을 타입 가드로 사용하도록 알려준다. 

```tsx
const getAvailableDestinationNameList = async (): Promise<DestinationName[]> => {
  const data = await AxiosRequest<string[]>('get', '.../destinations');
  const destinationNames: DestinationName[] = [];
  data?.forEach(str => {
    if (isDestinationCode(str)) {
      destinationNames.push(DestinationNameSet[str]);
    }
    /**
     * isDestinationCode의 반환 값에 is를 사용하지 않고 boolean이라 하면 다음 에러가 발생
     * - Element implicitly has an 'any' type because expression of type 'string'
		 * can't be used to index type 'Record<"MESSAGE_PLATFORM" | "COUPON_PLATFORM" | "BRAZE",
		 * "통합메시지플랫폼" | "쿠폰대장간" | "braze">'
     */
  });
  return destinationNames;
};
```

if문 내 isDestinationCode 함수로 data의 str이 destinationCodeList의 문자열 원소인지 체크하고, 맞는다면 destinationNames배열에 push 한다. 

만약 isDestinationCode의 반환값 타이핑을 x is DestinationCode가 아닌 boolean으로 했다면 타입스크립트는 어떻게 추론할까? 개발자는 if문 내부에서 str타입이 DestinationCode라는 것을 알 수있다. Array.includes를 해석할 수있기 때문이다.

하지만 타입스크립트는 isDestinationCode 함수 내부에 있는 includes 함수를 해석해 타입 추론을 할 수 없다. 타입스크립트는 if문 스코프의 str타입을 좁히지 못하고 string으로만 추론한다. destinationNames의 타입은 DestinationName[]이기 때문에 string타입의 str을 push할 수없다는 에러가 발생한다. 

이처럼 타입스크립트에게 반환값에 대한 타입 정보를 알려주고 싶을 때 is를 사용할 수 있다. 

다른 예시)

```tsx
interface Bird {
  fly: () => void;
  layEggs: () => void;
}

interface Fish {
  swim: () => void;
  layEggs: () => void;
}

// isFish 함수는 pet이 Fish 타입인지를 판별
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function getPetAction(pet: Fish | Bird) {
  if (isFish(pet)) {
    // 이 블록에서 pet은 Fish 타입으로 간주
    pet.swim();
  } 
	return;
}
```

만약 isFish의 반환값 타이핑을 pet is Fish가 아닌 boolean으로 했다면 타입스크립트는 어떻게 추론할까? 

<details>
<summary>해설</summary>
    
isFish 함수의 반환 타입이 boolean이라면, getPetAction 함수 내에서 isFish(pet)가 true를 반환해도 pet의 타입이 Fish인지 확신할 수 없다. <br/>
따라서, pet.swim()을 호출하는 것은 타입 오류를 발생시킬 수 있다. 타입스크립트는 pet이 여전히 Fish | Bird 타입으로 간주되기 때문이다.
    

    function isFish(pet: Fish | Bird): boolean {
      return (pet as Fish).swim !== undefined;
    }
    
    function getPetAction(pet: Fish | Bird) {
      if (isFish(pet)) {
        // 여기서 타입스크립트는 pet이 Fish 타입인지 확신할 수 없습니다.
        // 따라서, pet.swim() 호출은 타입 오류를 발생시킬 수 있습니다.
        pet.swim();
      } else if (isBird(pet)) {
        // 마찬가지로, 여기서도 pet이 Bird 타입인지 확신할 수 없습니다.
        pet.fly();
      }
    }

</details>    

# 4.3 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

>💡 **식별할 수 있는 유니온이란 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아주어 포함 관계를 제거하는 것**

```tsx
type TextError = {
  errorCode: string;
  errorMessage: string;
};

type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; // 토스트를 띄워줄 시간
};

type AlertError = {
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
};

type ErrorFeedbackType = TextError | ToastError | AlertError;
const errorArr: ErrorFeedbackType[] = [
  { errorCode: '100', errorMessage: '텍스트 에러' },
  { errorCode: '200', errorMessage: '토스트 에러', toastShowDuration: 3000 },
  { errorCode: '300', errorMessage: '얼럿 에러', onConfirm: () => {} },
  { // 이거 잘못된건데 덕 차이핑 언어이기 때문에 별도 타입 에러를 뱉지 않음
    errorCode: '999',
    errorMessage: '잘못된 에러',
    toastShowDuration: 3000,
    onConfirm: () => {},
  },
];
```

맨 아래 잘못된 에러에서 타입 에러를 뱉지 않고 있음 → 추후 개발 과정에서 의미를 알 수 없는 무수한 에러가 발생할 거임 → 식별할 수 있는 속성을 넣자! 

```tsx
type TextError = {
	errorType: 'TEXT';  👈
  errorCode: string;
  errorMessage: string;
};

type ToastError = {
	errorType: 'TOAST'; 👈
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; // 토스트를 띄워줄 시간
};

type AlertError = {
	errorType: 'ALERT'; 👈
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
};

type ErrorFeedbackType = TextError | ToastError | AlertError;
const errorArr: ErrorFeedbackType[] = [
  {
    errorType: 'TEXT',
    errorCode: '999',
    errorMessage: '잘못된 에러',
    toastShowDuration: 3000,
		// 개체 리터럴은 알려진 속성만 지정할 수 있으며 
		// 'TextError' 형식에 'toastShowDuration'이(가) 없습니다.ts(2353)
    onConfirm: () => {},
  },
];
```

정확하지 않은 객체에 대해 타입 에러가 발생하는 것을 확인 가능

- 식별할 수 있는 유니온의 판별자 선정
    - 리터럴 타입이어야 한다.
    - 판별자로 선정한 값에 적어도 하나 이상의 **유닛 타입**이 포함되어야 하며, 인스턴스화할 수 있는 타입은 포함되지 않아야 한다.
- **유닛 타입:** 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가지는 타입을 말한다. null, undefined, 리터럴 타입을 비롯해 true, 1등 정확한 값을 나타내는 타입
- **인스턴스화할 수 있는 타입:** 클래스, 인터페이스나 타입 별칭으로 정의된 객체 타입, Error 객체

# 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

> 💡 모든 케이스에 대한 분기 처리를 해야 유지보수 측면에서 안전하다 생각되는 상황 > 모든 케이스에 대한 타입 검사 강제해보자 > Exhaustiveness Checking pattern

```tsx
type ProductPrice = "1000" | "2000" | "3000"

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === "1000") return "토스포인트 1천 원";
  if (productPrice === "2000") return "토스포인트 2천 원";
  // if (productPrice === "3000") return "토스포인트 3천 원";
  else {
    exhaustiveCheck(productPrice); // 'string' 형식의 인수는 'never' 형식의 매개 변수에 할당될 수 없습니다.ts(2345)
    return "토스포인트";
  }
}

const exhaustiveCheck = (param: never) => {
  throw new Error('type Error!');
}
```

exhaustiveCheck함수는 매개변수를 never타입으로 선언하고 있다.

매개변수로 그 어떤 값도 받을 수 없으며 만일 값이 들어온다면 에러를 내뱉는다. 이 함수를 타입 처리 조건문의 마지막 else문에 사용하면 앞의 조건문에서 모든 타입에 대한 분기 처리를 강제할 수있다.
