## 5.1 조건부 타입

타입스크립트에서는 조건에 따라 다른 타입을 반환해야 하는 경우 조건부 타입을 사용해 출력 타입을 다르게 도출할 수 있다. 조건부 타입은 자바스크립트의 삼항 연산자와 동일한 형태를 가지고 있다. `Condition ? A : B` 

조건부 타입을 활용하면 **중복되는 타입을 제거**하고 상황에 따라 적절한 타입을 얻을 수 있어 더욱 **정확한 타입 추론**을 할 수 있다. 

### extends와 제네릭을 활용한 조건부 타입

extends를 활용한 조건부 타입의 형태는 다음과 같다.

```tsx
T extends U ? X : Y
// T를 U에 할당할 수 있으면 X 타입
// T를 U에 할당할 수 없으면 Y 타입
```

조건부 타입에 제네릭을 활용하면 제네릭 매개변수에 따라 타입을 추론할 수 있다.

```tsx
interface Bank {
  financialCode: string;
}
interface Card {
  financialCode: string;
  appCardType?: string;
}
type PayMethod<T> = T extends 'card' ? Card : Bank;
type CardPayMethodType = PayMethod<'card'>; // Card
type BankPayMethodType = PayMethod<'bank'>; // Bank

// -------------------------------------------------

type Flatten<T> = T extends unknown[] ? T[number] : T;

type Str = Flatten<string[]>; // string
type Num = Flatten<number>; // number
```

### 조건부 타입을 사용하지 않았을 때의 문제점

```tsx
interface Dog {
  type: "dog";
}
interface Cat {
  type: "cat";
}

type Animal = Dog | Cat;

const pickAnimal = (type: "dog" | "cat"): Array<Animal> => {
	const data = [{ type: "dog" }, { type: "cat" }];

  return data.filter(v => v.type === type);
};

const result = pickAnimal("dog");
// 값: [{type: 'dog'}] 
// 타입: Animal[]
```

여러 동물 데이터 중 특정 동물 데이터만 추출하는 함수가 있다. 인자로 `dog` 타입을 전달해 해당하는 동물 데이터를 반환 받았지만 타입은 `Dog[]`으로 추론되지 않고 `Animal[]` 타입으로 추론되어 타입상으론 `Cat[]` 타입이 포함되는 문제가 발생했다. 값은 조건에 맞는 값을 반환하도록 구현했지만 타입은 조건에 따른 처리를 해주지 않았기 때문이다. 이렇듯 조건부 타입을 사용하지 않은 경우 값과 타입이 서로 맞지 않는 경우가 생길 수 있다.

```tsx
interface Dog {
  type: "dog";
}
interface Cat {
  type: "cat";
}

type Animal<T extends 'dog' | 'cat'> = T extends 'dog' ? Dog : Cat;

const pickAnimal = <T extends 'dog' | 'cat'>(type: T): Array<Animal<T>> => {
	const data = [{ type: "dog" }, { type: "cat" }];

  return data.filter(v => v.type === type);
};

const result = pickAnimal("dog");
// 값: [{type: 'dog'}] 
// 타입: Dog[]
```

유니온 타입이였던 `Animal` 타입을 조건부 타입으로 변경해 조건에 맞는 타입을 반환하도록 수정해 문제를 해결했다.

### **분산적인 조건부 타입**

```tsx
type ToArray<Type> = Type extends unknown ? Type[] : never;
type StrArrOrNumArr = ToArray<string | number>;
// StrArrOrNumArr의 타입은? 
// 1. (strnig | number)[]
// 2. strnig[] | number[]
```

제네릭 타입 위에서 조건부 타입은 유니온 타입을 만나면 **분산적**으로 동작한다.

이러한 동작을 방지하려면 `extends`키워드의 양 옆을 대괄호로 감싸면 된다.

```tsx
type ToArrayNonDist<Type> = [Type] extends [unknown] ? Type[] : never;
type StrArrOrNumArr = ToArrayNonDist<string | number>;
```

### infer를 활용해서 타입 추론하기

infer는 타입을 추론하는 역할을 한다. extends로 조건을 서술하고 infer로 타입을 추론하는 방식을 취한다.

```tsx
type UnpackPromise<T> = T extends Array<Promise<infer K>> ? K : any;

const promises = [Promise.resolve('Mark'), Promise.resolve(38)];
type Expected = UnpackPromise<typeof promises>; // string | number

// ----------------------------------------------------------------

type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;

type Str = Flatten<string[]>; // string
type Num = Flatten<number>; // number
```

`UnpackPromise` 타입은 제네릭으로 T를 받고 T가 Promise로 매핑된 경우라면 infer에 의해 추론된 K를 반환하고 그렇지 않은 경우엔 any를 반환한다. 

```tsx
interface RouteItem {
  path: string;
  private: boolean;
  children?: ReadonlyArray<RouteItem>;
}

const route = [
  {
    path: "/login",
    private: false,
  },
  {
    path: "/community",
    private: false,
    children: [
      {
        path: "/community/:id",
        private: true,
        children: [
          {
            path: "/community/:id/setting",
            private: true,
          },
        ],
      },
      {
        path: "/community/create",
        private: true,
      },
    ],
  },
  {
    path: "/profile",
    private: true,
  },
] as const;

type CheckPermission<Permission extends "private" | "pulic"> = Permission extends "private" ? true : false;
type UnpackPermissionRoutePath<
  Route extends ReadonlyArray<RouteItem>,
  Permission extends "private" | "pulic",
> = Route extends ReadonlyArray<infer Item>
  ? Item extends RouteItem
    ? Item["children"] extends infer Children
      ? Children extends ReadonlyArray<RouteItem>
        ?
            | UnpackPermissionRoutePath<Children, Permission>
            | (Item["private"] extends CheckPermission<Permission> ? Item["path"] : never)
        : Item["private"] extends CheckPermission<Permission>
          ? Item["path"]
          : never
      : never
    : never
  : never;

type PrivateRoutePath = UnpackPermissionRoutePath<typeof route, "private">;
// "/community/:id" | "/community/:id/setting" | "/community/create" | "/profile"

type PublicRoutePath = UnpackPermissionRoutePath<typeof route, "public">;
// "/login" | "/community"

```

## 5.2 템플릿 리터럴 타입 활용하기

타입스크립트에서는 타입을 문자열로 지정할 수 있는데 이런 방식을 템플릿 리터럴 타입이라고 부른다.

```tsx
type HeadingNumber = 1 | 2 | 3 | 4 | 5;
type HeaderTag = `h${HeadingNumber}`;
```

앞서 정리한 조건부 타입과 infer 그리고 템플릿 리터럴 타입을 활용하면 아래와 같은 타입도 만들 수 있다.

```tsx
type RouteParameters<Route> = Route extends `${string}/:${infer Param}/${infer Rest}`
  ? { [Entry in Param | keyof RouteParameters<`/${Rest}`>]: string }
  : Route extends `${string}/:${infer Param}`
    ? { [Entry in Param]: string }
    : never;

type Params = RouteParameters<"/path/:id/:type/path/:index">;
// { id: string, type: string, index: string }
```

## 5.3 커스텀 유틸리티 타입 활용하기

### 문제 상황

타입스크립트에는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 진행되지 않는 이슈가 있다. 

```tsx
type Card = {
  card: string
};

type Account = {
  account: string
};

function withdraw(type: Card | Account) {
  ...
}

withdraw({ card: "hyundai", account: "hana" });
```

`Card`, `Account` 중 하나의 객체만 받고 싶은 상황에서 유니온 타입을 작성하면 의도한 대로 타입 검사가 이뤄지지 않는다. 타입을 집합의 관점으로 봤을 때  `{ card: "hyundai", account: "hana" }`는 합집합의 범주에 들어가기 때문에 타입 에러가 발생하지 않는다.

### 식별할 수 있는 유니온

위 같은 문제를 해결하기 위해 식별할 수 있는 유니온 기법을 자주 활용한다. 각 타입에 type이라는 공통된 속성을 추가하여 구분짓는 방법이다.

```tsx
type Card = {
  type: "card";
  card: string;
};

type Account = {
  type: "account";
  account: string;
};

function withdraw(type: Card | Account) {
  ...
}

withdraw({ type: "card", card: "hyundai" });
withdraw({ type: "account", card: "hana" }); // type error
withdraw({ type: "account", account: "hana" });
```

각 타입에 식별자를 추가해 타입을 추론할 수 있도록 수정했다. 하지만 일일이 식별자를 넣어줘야하는 불편함과 이미 구현된 상태에서 식별자를 적용하려면 사용하는 모든 곳을 수정해줘야 한다. 이런 상황을 방지하기 위해 커스텀 유틸리티 타입을 구현하고 적용해보자.

### PickOne 커스텀 유틸리티 타입

```tsx
type PickOne<T> = {
  [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];
```

PickOne 타입은 속성 하나와 나머지는 옵셔널한 undefined인 타입으로 만들어준다. 

```tsx
function withdraw(type: PickOne<Card & Account>) {
  ...
}

withdraw({ card: "hyundai", account: "hana" }); // type error
```

PickOne을 활용해 앞의 코드를 수정하면 에러가 발생한다.

### NonNullable 타입 검사 함수

```tsx
type NonNullable<T> = T extends null | undefined ? never : T;

function NonNullable<T><value: T): value is NonNullable<T> {
	return value !== null && value !== undefined
}
```

타입 가드를 통해 null과 undefined을 처리해줄 수 있다.

## 5.4 불변 객체 타입으로 활용하기

상숫값을 담은 불변 객체에 `keyof`, `typeof` 를 활용해 타입으로 활용할 수 있다.

```tsx
const colors = { ... };

const theme = { ... };

type ColorType = typeof keyof theme.colors;
type BackgroundColorType = typeof keyof theme.backgroundColor;
type FontSizeType = typeof keyof theme.fontSize;

interface Props {
  color?: ColorType;
  backgroundColor?: BackgroundColorType;
  fontSize?: FontSizeType;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
}
```

## 5.5 Record 원시 타입 키 개선하기

객체 선언 시 키가 어떤 값인지 명확하지 않다면 Record의 키를 string이나 number 같은 원시 타입으로 명시하곤 한다. 이 때 타입스크립트는 키가 유효하지 않더라도 타입상으로는 문제가 없기 때문에 오류를 표시하지 않는다.

```tsx
type Category = string;
interface Food {
  name: string;
  // ...
}
const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

foodByCategory["양식"]; // Food[]로 추론
foodByCategory["양식"].map((food) => console.log(food.name)); // 타입 오류가 발생하지 않는다
```

### 유닛 타입으로 변경

```tsx
type Category = "한식" | "일식";
interface Food {
name: string;
// ...
}
const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

// Property ‘양식’ does not exist on type ‘Record<Category, Food[]>’.
foodByCategory["양식"];
```

키가 유한한 집합이라면 유닛 타입을 사용할 수 있다. 객체의 key를 `Category`으로 지정해 한식, 일식 외의 키를 사용하면 에러가 발생한다. 하지만 키가 무한해야하는 상황에서는 적합하지 않은 방식이다.

### Partial을 활용하여 정확한 타입 표현

```tsx
type PartialRecord<K extends string, T> = Partial<Record<K,T>>;
type Category = string;

interface Food {
  name: string;
  // ...
}

const foodByCategory: PartialRecord<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};
  
foodByCategory["양식"]; // Food[] 또는 undefined 타입으로 추론
foodByCategory["양식"].map((food) => console.log(food.name)); // Object is possibly 'undefined'
foodByCategory["양식"]?.map((food) => console.log(food.name)); // OK
```

키가 무한한 상황에서는 Parital을 사용하여 해당 값이 undefined일 수 있는 상태임을 표현할 수 있다.
