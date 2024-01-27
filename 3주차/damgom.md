# 3장 고급 타입
## 타입스크립트만의 독자적인 타입 시스템
### any 타입
#### any 타입을 사용하는 경우

1. 개발 단계에서 임시로 값을 지정하는 경우
    - 변경될 가능성이 있거나 관련 타입이 정해지지 않은 경우
    - 타입에 대한 세부 스펙이 나온 이후에는 변경하는 것이 좋다

2. 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
    - api 요청 및 응답 처리
    - 콜백 함수 전달
    - 타입이 잘 정제되지 않아 파악이 힘든 외부 라이브러리 사용

``` ts
// 피드백을 나타내기 위해 모달 창을 그릴 때 사용되는 인자를 나타내는 타입

type FeedbackModalParams = {
  show: boolean;
  content: string;
  cancelButtonText?: string;
  confirmButtonText?: string;
  beforeOnClose?: () => void;
  action?: any;
}
```

  - `action`: 모달 창을 그릴 때 실행될 함수
  - 모달창을 화면에 그릴 때 다양한 범주의 액션에 따라 인자의 개수나 타입을 일일이 명시하기 힘든 경우, `any` 타입을 사용해서 다양한 액션 함수를 전달할 수 있다

3. 값을 예측할 수 없을 때 암묵적으로 사용
    - 외부 라이브러리나 웹 API 요청에 따라 다양한 값을 반환하는 API가 존재할 수 있다
    - fetch API의 일부 메서드는 요청 이후의 응답을 특정 포맷으로 파싱하는데 이때 반환 타입이 any로 매핑되어있다

<img width="438" alt="image" src="https://github.com/minkyung00/woowahan-typescript/assets/80238096/2398860b-0802-470a-a363-a2673283b328">

<br />

---

<br />

### unknown 타입
- unknown 타입은 any 타입과 유사하게 모든 타입의 값이 할당될 수 있다
- unknown 타입은 any 타입 외의 다른 타입으로 할당 불가능하다

<img width="734" alt="스크린샷 2024-01-26 오후 10 58 18" src="https://github.com/minkyung00/woowahan-typescript/assets/80238096/36da75bd-3ef0-4a02-a595-a5e7299f5420">

any 타입과 유사한데 unknown 타입이 추가된 이유
- unknown 타입 변수에 할당할 때는 컴파일러 아무런 경고를 주지 않지만, 이를 실행하면 다음과 같은 에러가 발생한다.
- 함수 뿐만 아니라 객체 내부에 접근하는 모든 시도에서 에러가 발생한다
- unknwon 타입은 어떤 타입이 할당될 수 없음을 나타내기 때문에 unknwon 타입으로 할당된 변수는 어떤 값이든 올 수 있음을 의미하는 동시에 개발자에게 엄격한 타입 검사를 강제하는 의도를 담고 있다.
- 따라서, 데이터 구조를 파악하기 힘들 때는 any 타입 대신에 unknown 타입을 대체해서 사용하는 방법을 권장한다.



#### unknown을 사용하는 경우

1. 강제 타입 캐스팅
``` ts
as unknown as Type
```
  - 사실상 any랑 동일

2. 뭔지 모르지만 하나씩 테스트하면서 알아내보자
    - any로 선언된 변수는 length를 참조하면 에러가 안나지만, unknown으로 선언한 것은 에러가 발생한다x

3. 예상할 수 없는 데이터
    - 타입스크립트 4.4부터 try-catch 에러 타입이 any에서 unknown으로 변경되어서 에러 핸들링할 때도 unknown을 사용

![image](https://github.com/minkyung00/woowahan-typescript/assets/80238096/1175b87d-10b0-4b8b-a8d1-ee843791d1bf)
- catch절의 error 변수 타입이 any -> unknown으로 변경되었다.

#### AS-IS
``` ts
try {
  // 어쩌구
} catch (x) {
  // x가 any라서
  e.message // obejct라고 가정할수도 있고
  e.toUpperCase() // string이라 가정할수도 있었음.
  e++
  e.yadda.yadda.yadda() // 맘대로 다 불러도 타입에러가 나지 않았음.
}
```

#### TO-BE
``` ts
try {
  // 어쩌구
} catch (e) {
  e.message // type error: Object is of type 'unknown'
  if (typeof e === "string") {
    e.toUpperCase() // 해결!
  }
}
```

<br />

---

<br />


### void 타입
- 함수 내부에 별도 반환문이 없다면 타입스크리브 컴파일러가 알아서 잘 추론하기 때문에 이 타입을 명시적으로 쓸 일이 적다

### never 타입
- never 타입은 값을 반환할 수 없는 타입을 말한다

자바스크립트에서 값을 반환할 수 없는 경우
1. 에러를 던지는 경우
    - 특정 함수가 실행 중에 에러를 던지는 작업을 수행한다면 해당 함수의 반환 타입은 `never`이다

2. 무한히 함수가 실행되는 경우
    - 무한 루프는 함수가 종료되지 않기 때문에 값을 반환하지 못한다

    - any 타입도 never 타입에 할당될 수 없다
    - 타입스크립트 조건부 타입을 결정할 때 특정 조건을 만족하지 않는 경우에 엄격한 타입 검사 목적으로 never 타입을 명시적으로 사용하기도 한다. 

<br />

---

<br />

### Array 타입

``` ts
// 숫자만 할당 가능
const array: number[] 
const array: Array<number>

// 여러 타입을 모두 관리해야하는 배열
const array1: Array<number | string> 
```

#### 튜플
- 대괄호를 사용해 선언할 수 있고 원소의 타입과 개수를 제한할 수 있다

``` ts
let tuple: [number] = [1]; // ✅
tuple = [1, 2] // ❌

// 원소마다 타입을 지정해줄 수 있음
let tuple2: [number, string, boolean] = [1, 'string', true]; // ✅

// 원소를 옵셔널로 설정할 수 있음
let tuple3: [number, number, number?] = [1, 2]; // ✅
```

useState
<img width="637" alt="image" src="https://github.com/minkyung00/woowahan-typescript/assets/80238096/cd59e0d0-b292-4863-b430-28bce500116a">
- useState는 튜플 타입을 반환한다.
- useState는 배열의 원소의 자리마다 명확한 의미를 부여하기 때문에 튜플 타입을 사용


### enum 타입

---
## 타입 조합
### Index Signature
- 다른 속성을 추가로 명시해줄 수 있는데, 추가로 명시되는 속성은 인덱스 시그니처에 포함되는 속성이어야한다.
``` ts
interface IndexSignature {
  [key: string]: number | boolean;
  length: number;
  isValid: boolean;
  name: string; // ❌ 에러 발생
}
```

#### Mapped Type
- 다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법
- 인덱스 시그니처 문법을 사용해서 반복적인 타입을 효과적으로 작성할 수 있다

``` ts
type Example = {
  a: number;
  b: string;
}

// 속성을 모두 optional로 변경
type Subset<T> = {
  [K in keyof T]?: T[K];
}

const a: Subset<Example> = { a: 3 };
```

- 기존 타입에 존재하던 `readonly` 또는  `?` 수식어 앞에 `-`를 붙이면 해당 수식어를 제거할 수 있다

``` ts
type ReadOnly = {
  readonly a: number;
  readonly b: string;
}

type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
}
```

``` ts
type Optional = {
  a?: number;
  b: string;
}

type Concrete<T> = {
  [P in keyof T]-?: T[P];
}
```

배달의 민족 바텀시트 예시

``` ts
const BottomSheetMap = {
  RECENT_CONTACTS: RecentContactsBottomSheet,
  CARD_SELECT: CardSelectBottomSheet,
  SORT_FILTER: SortFilterBottomSheet,
  PRODUCT_SELECT: ProductSelectBottomSheet,
  REPLY_CARD_SELECT: ReplyCardSelectBottomSheet,
  RESEND: ResendBottomSheet,
  STICKER: StickerBottomSheet,
  BASE: null,
};

export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;

// 불필요한 반복이 발생한다
type BottomSheetStore = {
  RECENT_CONTACTS: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  CARD_SELECT: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  SORT_FILTER: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  // ...
};

// Mapped Types를 통해 효율적으로 타입을 선언할 수 있다
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

- as 키워드를 사용해서 키를 재지정할 수 있다

``` ts
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};

```

### 템플릿 리터럴 타입
[토스 - Template Literal Types로 타입 안전하게 코딩하기](https://toss.tech/article/template-literal-types)

---

- next.js 13 파일 기반 라우팅 시스템

``` ts
type SearchOrHash = `?${string}` | `#${string}`

type SafeSlug<S extends string> = S extends `${string}/${string}`
  ? never
  : S extends `${string}${SearchOrHash}`
  ? never
  : S extends ''
  ? never
  : S
```

- /로 구분된 두 개의 문자열을 포함하는 경우를 허용하지 않음
- 또는 SearchOrHash 타입과 함께 사용되는 경우를 허용하지 않음

``` ts
  type StaticRoutes = 
    | `/`
    | `/my`
    | `/onboarding`
  type DynamicRoutes<T extends string = string> = 
    | `/home/${SafeSlug<T>}`
    | `/api/${OptionalCatchAllSlug<T>}`

  type RouteImpl<T> = 
    | StaticRoutes
    | SearchOrHash
    | WithProtocol
    | `${StaticRoutes}${SearchOrHash}`
    | (T extends `${DynamicRoutes<infer _>}${Suffix}` ? T : never)
```

---

1. [JSON 파서 만들기](https://www.typescriptlang.org/play?ts=4.1.0-pr-40336-88&ssl=64&ssc=73&pln=64&pc=1#code/C4TwDgpgBACghgJwM4QQUQQg9ggPAFSggA9gIA7AEySiWAQEtyBzAPigF4oBvIzHAFxR6AV2gBfKADIo+AFChIUNHGAB1ABYMySMHADGEXAGVgq6CTJUadRi3Yc5UWvSbMipCtSinzT51AA-LCIKOj8eABEKupaOnqGUMxYwEkUqAz6LnbuihCRrP7OQr5kHlbeAAZQACTcTABmqD5mZOLVAD5QlQA65HWNzaUQ7UUBwTGa2hC6BkbDhQEBJa0QCuDQAIKUlADSECAAanAANmK4ALIQALZY5V40AEoQ+jiUuLZuADRQcOQgrB++xA92s2W+UGOZwsnjBfwBnCg-iut2kPCgAG1dlAmFBgQBdIRQsRQcTrJTwZAQABSSCw5AA8gAjABWL2AJlWoO8nxYPxRd0sDygz1eCHevOYP3hDh44gc-kl3JowzGwUpYQw2CiGppdMZrPZSRSaXIGSySryBTGQkmcRmCXmq3YQrBlXEA3ITQQLXMoyWzmCGIFP2G+LGxWUqim8TmnPMLthVUinu9eIO4hT9S9Q1W-oDgajsWms0M8bIiYqNEqAlTub9lQjAfVoT19OJToTysxgx9HZ+lTrPuG7XDBYLE2j9tLnYr3cqXyHvrajfH45bVNp9OZbP0HOGP22ewOHcuNywQIOPw7rEWa4DtqnJcd5YgleF7qXI9X94nmKPwKniG6YgNepxiICy4QGOv4PiEVLhNquCVLqW4GruqQIC8EAMAAbhAlBQCIZrEJAe4EcIWAANYUEIdTfnev5CLqiE4MhqH6h2UBYcAIgIGahHESQZFkIRuHgdADSCLU3AMU2SzMa2rF4ChrZoTuRpYYYeEUUJpHshRwDUbRMlyWuikIVqbGqZu+oaXu3HYTpgkkSJhnGeQdGyXmlSFHk8EoGhmyYHAICvt2kr8ue3bwhi+KInFCrOEqrreKqSwbpqES4JEHH0sFCChcaqTMOkjAWq4LDCBs1oKUWMYOnGCzzviX4+U2QbAWGTYWYFnESa+75ghivaQhJA5tQ2MF-naz5Nc686LtmabfvJ4wBW25AFaFuCzbGZYLD8GIAHSncBHb4ox46PsW+2zm+LWTSua0ZZiJ1neeYHQvioarNN969agVkqXlW0hSCWk4fhLnCQZhFGTRnmmT5V11SxwPsWp-XQo5vH8bprlw1A4k41JCBeXJ-mg6ewwRZVbCcIq9Pdul63o9luVY+2EnFaa5rglVVpXTdDUzoN87kCIJwnE9Iw-q9GKS9Lv3mP9kZ7Y1B0Lal1ZZqNHaZrL+Z-hi-ZQWryz1dOL7NTr3QYkb8vrhtQXg4NL1QCL1vzV2duVNwjse4WoP2fuzpB57G3KZjtnczjkPOURhPkfDHkUyj5LQKDBB0zkSUu9j5z4EN3ijc8SBS8AaoijMlfdiNOZ9uNAvMBbwQdj1NcVycqR2+zSEymtwTl5Xa2A9HnOx+QXE8XxAlJ7DKdd5XtWW-3bGT31cckrP+Mub2hVMic0Ajz3BQANxyFf1+Z1AaGItnADkvCRDRIAAIyREIGKRCTYifz8JWJwfpQFfgcAATF-UBf8ICQNJI-Vgl8b7XyAA)
``` ts
// type Json = { key1: ['value1', null]; key2: 'value2' };
type Json = ParseJson<'{ "key1": ["value1", null], "key2": "value2" }'>;
```

2. [document.querySelector를 타입 안전하게 사용하기](https://www.typescriptlang.org/play?ts=4.1.0-dev.20200920#code/C4TwDgpgBAymA2BLYAeGUIA9gQHYBMBnKQ4AJ0VwHMAaKAEQ2zyJPMqoD4oBeWJnAWIADACQBvSgDMIZKABUAvhPrLJuGXICqi4VAD8UANry6AOgtwkqLXXqcAulABcxmA4DcAKFCQFAQwBrCAAZf1IUADVuPkiBFmIjJ0NcCAA3WRcoOKxBViNSCmpk7KMABidXHOYhY0KOcwtpTIAlEvkg0PDUFu5XVIyyb19oeQoAWxCIKVRqvOJ66hjs+Nq9CWa5Ft0DBQmpmZRerMjh8FGJlsQqAAtZ1dZFrl4V3ISoMXVNKG2P3bHEOMrrcen1smc-ADxlEHgt2EsXlCDqgocC7lFOJwIdAYOwwABZAD2+EQUkQshhb1qTzo+NhbCKz1i9M+mygITU+LUbIAgjtDCETtjYHiiSSybJCJSao94UyRRQCcTSeSyGhRcqJWrcYqxSqKTrEErxaqonQAORmc2cC0AYmtFqMDqg5uc1uFHWCYVIPJmsnkhOCuGl8wZDQU9Key0Nxv1ZClnq6ESsyBQUIxdHkmKxPnOUAA4hBgN7gABReAQcZ4YAAOX8VZD7yjiM6Jd9ODIAaDadb3Xb-sDeDNLpdNtH7tzfkLZf8AGMbuXK9W6w3InRBVTZYyki8kssvFBD68ZYknAejxeBVBzxfD1V6QU5Wfb0fDEYLGYQnRpyXF1XcLW9YQFE5QOI414vnex6ho+jKNGYbJtBBkGGNOpZzguFb-oBDYtHQ74WF+BZFr+WHLkBIEVI43A3rerhJMKaFkQBK4QFKcxNnKyxoRhf7kQ2KazBaNDWjmIxQHxAEAEIgKxjbUlxLy0RxtTBCAhJSFAAAS8j4iEknAB0VCsfi-hgMhL6GDpekGUZJlmUYkTPpB96bsQakabAkT5rZ-jGUBpnmbRl5eT5zGGX59lgI5zkvq4BnCqZwDzuhKXhcOG4nhJ4VQAAPlAuAAK7wPALxFSV+4Xip+SxbeV7BUerlZbBHC1SFgr5QZMlyZEoE0ZBJwPjSUAfohbWvlASUpbx6V4eyeXZUu0myRRvVUf1Lnst4k7QAAioVsggC0bHFSikaKXwU0LjNS2oExt2sQmmLbQA9AAVG9B5vRJmD1ggEBfS9Xj4BAs7wP4ZDQFIhW4LOwCIISuBQAAjgdZAgDAEAVnDhJqvI52MpwAAUqOHa48gAJSuPth3HYQp1pjmXizojpBQP4Lyk+jmPY8AuNE+aJJpGYABG-i4KkcjcP4Zizv4JUALR8wrc7w4j5oU1AL0vQr3DWSEPKwzcuMGczrPAFAIuc2jGNY6DfNkALlBgIVwB0ELGtazreu6SEACSuAu2WOX5fr9CIGkpss7gbOztbh08-b-PmrOiBkGDEBGLOmA8AARAAjAArGUucOJ72u66FADCacZ1H5tQPg8fc3bOOO+aIuu3zuC2p3IAK7ghIAO7l972m+1JXeI-XMcW9AfBc7bvPJ4Q9sI0jYDOGS8bAArGlK+cGseF7lf6wAChDflkGZmG3V4QA)

``` ts
declare function querySelector<T extends string>(query: T): QueryResult<T>;

const a = querySelector('div.banner > a.call-to-action') //-> HTMLAnchorElement
const b = querySelector('input, div') //-> HTMLInputElement | HTMLDivElement
const c = querySelector('circle[cx="150"]') //-> SVGCircleElement
const d = querySelector('button#buy-now') //-> HTMLButtonElement
const e = querySelector('section p:first-of-type'); //-> HTMLParagraphElement

```

``` ts
type TrimLeft<V extends string> = V extends ` ${infer R}` ? TrimLeft<R> : V;
```

![image](https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/80238096/11593c51-afb9-4f42-91d5-cb2b38b8c370)

![image](https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/80238096/73ab3d8e-305a-47e5-87e1-c7b0010b7667)


---
## 제네릭 사용법

``` ts
type Prettify<T> = {
  [K in keyof T]: T[K];
} & {}
```

| AS-IS | TO-BE |
|--------|--------|
| ![image](https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/80238096/1e62bf83-ba2c-477d-b02d-f4adf046fc3e) | ![image](https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/80238096/ac0c75d8-9b76-462f-aa89-4f8d12a390ec) |






