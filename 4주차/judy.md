## 8.2 타입스크립트로 리액트 컴포넌트 만들기

타입스크립트는 리액트 프로젝트에서 공통 컴포넌트에 어떤 타입의 속성(프로퍼티)이 제공되어야 하는지 알려주고, 필수로 전달되어야 하는 속성이 전달되지 않았을 때는 에러를 표시해줌

```tsx
// 각 Option의 키-값 쌍을 객체로 받고 있으며, 선택된 값이 변경될 때 호출되는 onChange 이벤트 핸들러를 받도록 구현
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value,
    )?.[0];
    onChange?.(selected);
  };

  return (
    <select
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  );
};
```

-> 각 속성에 어떤 타입의 값을 전달해야 할지 알기 어렵기 때문에 컴포넌트를 사용하는 개발자가 각 속성에 어떤 타입의 값을 전달해야 할지 명확하게 알 수 있도록 추가적인 설명이 필요

### 1. JSDocs로 일부 타입 지정하기

- JSDocs를 활용해 컴포넌트와 각 속성 역할 설명 -> 타입스크립트의 타입 체크 대체 불가
- JSDocs를 활용하면 각 속성의 대략적인 타입과 어떤 역할을 하는지 파악할 수 있지만 options가 어떤 형식의 객체를 나타내는지 onChange의 매개변수 및 반환 값에 대한 구체적인 정보를 알기는 쉽지 않다.

```jsx
/**
* Select 컴포넌트
* @param {Object} props - Select 컴포넌트로 넘겨주는 속성
* @param {Object} props.options - { [key: string]: string } 형식으로 이루어진 option 객체
* @param {string | undefined} props.selectedOption - 현재 선택된 option의 key값 (optional)
* @param {function} props.onChange - select 값이 변경되었을 때 불리는 callBack 함수 (optional)
* @returns {JSX.Element}
*/
const Select = //...
```

### 2. props 인터페이스 적용하기

```tsx
type Option = Record<string, string>; // {[key: string]: string}
// 키와 값 모두 문자열

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void;
  //string 혹은 undefined를 매개변수로 받음. 어떤 값도 반환하지 않음.
  // 이 핸들러는 옵셔널 프로퍼티이므로, 부모 컴포넌트에서 제공하지 않아도 됨.
}

const Select = ({
  options,
  selectedOption,
  onChange,
}: SelectProps): JSX.Element => {
  //...
};
```

❕ [key: string]은 사실상 모든 키값을 가질 수 있음을 나타낸다. 하지만 넓은 범위의 타입은 해당 타입을 사용하는 함수에 잘못된 타입이 전달되기 때문에, 가능한 한 타입을 좁게 제한하여 사용하는 것을 권장한다. ( 모든 문자열 키값을 허용하는 것이 아니라, 구체적인 키를 명시하거나 키의 값을 제한하는 것을 권장)

```tsx
interface Fruit {
  count: number;
}

interface Param {
  [key: string]: Fruit; // type Param = Record<string, Fruit>과 동일
}

// Param 타입의 객체를 매개변수로 받음.
// 객체분해할당을 사용하여 Param 객체 내의 apple 속성에 접근함.
const func: (fruits: Param) => void = ({ apple }: Param) => {
  console.log(apple.count);
};

// OK.
func({ apple: { count: 0 } });

// Runtime Error (Cannot read properties of undefined (reading 'count'))
func({ mango: { count: 0 } });
// func 함수가 mango에 접근을 시도하지만 전달된 객체에 mango은 없음.
// undefined의 count 속성에 접근하려고 하여 오류가 발생함.
```

### 3. 리액트 이벤트

리액트 이벤트는 브라우저의 고유한 이벤트와 완전히 동일하게 동작하지 않는다.

- **리액트 이벤트 처리의 특징**

1. 카멜 케이스 표기 : 이벤트 핸들러 이름을 카멜 케이스로 작성한다.
   1. `onclick`, `onchange`같이 DOM 엘리먼트에 등록되어 처리하는 이벤트 리스너와 달리, 리액트 컴포넌트(노드)에 등록되는 이벤트 리스너는 `onClick`, `onChange`처럼 카멜 케이스로 표기
2. 이벤트 버블링 : 기본적으로 이벤트 버블링 단계(이벤트가 발생한 요소에서 시작하여 DOM 트리를 따라 상위 요소로 전파)에서 이벤트 핸들러가 호출된다. 캡처 단계(이벤트가 최상위 노드에서 시작하여 이벤트가 발생한 노드까지 내려가는 과정)에서 이벤트 핸들러를 호출하려면, `Capture`를 추가한다. (ex. `onClickCapture`, `onChangeCapture`)
3. 합성 이벤트(`SyntheticEvent`) : 리액트는 브라우저의 **기본 이벤트를 감싸는 합성 이벤트를 제공**한다. 이는 브라우저 간 일관된 이벤트 객체를 제공하기 위함이다.

   1. 리액트가 제공하는 이벤트 처리 시스템, 브라우저의 기본 이벤트를 감싸는 래퍼 역할을 함
   2. 모든 브라우저에서 일관된 방식으로 작동하도록 설계

      ```tsx
      // Event System
      // ----------------------------------------------------------------------
      // TODO: change any to unknown when moving to TS v3
      interface BaseSyntheticEvent<E = object, C = any, T = any> {
        nativeEvent: E;
        currentTarget: C;
        target: T;
        bubbles: boolean;
        cancelable: boolean;
        defaultPrevented: boolean;
        eventPhase: number;
        isTrusted: boolean;
        preventDefault(): void;
        isDefaultPrevented(): boolean;
        stopPropagation(): void;
        isPropagationStopped(): boolean;
        persist(): void;
        timeStamp: number;
        type: string;
      }

      /**
       * currentTarget - a reference to the element on which the event listener is registered.
       *
       * target - a reference to the element from which the event was originally dispatched.
       * This might be a child element to the element on which the event listener is registered.
       * If you thought this should be `EventTarget & T`, see https://github.com/DefinitelyTyped/DefinitelyTyped/issues/11508#issuecomment-256045682
       */
      interface SyntheticEvent<T = Element, E = Event>
        extends BaseSyntheticEvent<E, EventTarget & T, EventTarget> {}
      ```

      https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts

- **이벤트 핸들러 타입**

```tsx
type EventHandler<Event extends React.SyntheticEvent> = (
  e: Event,
) => void | null;

type ChangeEventHandler = EventHandler<ChangeEvent<HTMLSelectElement>>;
```

이 코드에서 `EventHandler`는 리액트의 합성 이벤트를 매개변수로 받는 함수 타입이다. `ChangeEventHandler`는 `HTMLSelectElement`의 `ChangeEvent`에 대한 이벤트 핸들러 타입을 정의한다.

```tsx
/* 예시)
 * 이벤트의 추가적인 전파 중단, 상위 요소로 전파되는 것을 방지
 */
export function withoutPropagation<E extends React.SyntheticEvent<any>>(
  handler: React.EventHandler<E>,
) {
  return (event: E) => {
    // E는 React.SyntheticEvent의 확장 > 다양한 종류의 이벤트 처리
    event.stopPropagation();
    handler(event);
  };
}
```

- **일반 이벤트 vs 리액트 합성이벤트**

```tsx
// 일반적인 Javascript 이벤트 핸들러
const eventHandler1: GlobalEventHandlers['onchange'] = (e) => {
  e.target; // 일반 Event는 target이 없음 -> e.target은 정의되지 않을 수도 있음
};

// 리액트의 합성 이벤트 핸들러
const eventHandler2: ChangeEventHandler = (e) => {
  e.target; // 리액트 이벤트(합성 이벤트)는 target이 있음 -> e.target을 안전하게 사용
};
```

- **리액트 Select 컴포넌트 예시**

```tsx
const Select = ({ onChange, options, selectedOption }: SelectProps) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value,
    )?.[0];
    onChange?.(selected);
  };

  return <select onChange={handleChange}>{/* ... */}</select>;
};
```

> 위 예시에서 `React.ChangeEventHandler<HTMLSelectElement>`타입은 `React.EventHandler<ChangeEvent<HTMLSelectElement>>`와 동일한 타입이다.

이 Select 컴포넌트는 options 객체에서 사용자가 선택한 옵션을 찾아 onChange 핸들러에 전달한다. handleChange는 `HTMLSelectElement`에 대한 리액트 합성 이벤트 핸들러로, `ChangeEvent<HTMLSelectElement>`타입의 이벤트를 매개변수로 받아 `e.target.value`를 사용하여 선택된 값을 찾는다.

### 4. 훅에 타입 추가하기

- undefined 문제

```tsx
const fruits = { apple: '사과', banana: '바나나', blueberry: '블루베리' };

const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<string | undefined>();
  // 제네릭 타입을 명시하지 않으면 타입 스크립트 컴파일러는 초깃값(default value)의 타입을 기반으로 state 타입을 추론
  // 타입 매개변수가 없다면 fruit의 타입이 undefined로 추론되면서 onChange의 타입과 일치하지 않아 오류가 발생

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
  );
};
```

```tsx
// fruit: undefined;
// changeFruit: (v: React.SetStateAction<undefined>) => void;
const [fruit, changeFruit] = useState();
// 초기값이 제공되지 않았으므로, fruit의 타입은 undefined로 추론됨.

return (
  <Select
    {/* Error - SetStateAction<undefined>와 맞지 않음 (changeFruit에는 undefined만 매개변수로 넘길 수 있음)*/}
    onChange={changeFruit} // string 값을 받는 함수를 원함.
    options={fruits}
    selectedOption={fruit}
  />
);
```

- 타입 추론에 맞지 않는 값 문제

```tsx
// fruit가 반드시 apple, banana, blueberry 중 하나라고 기대
const fruits = { apple: '사과', banana: '바나나', blueberry: '블루베리' };
const [fruit, changeFruit] = useState('apple');

// error가 아님
const func = () => {
  changeFruit('orange');
};
```

```tsx
// fruit가 반드시 apple, banana, blueberry 중 하나라고 기대
const fruits = { apple: '사과', banana: '바나나', blueberry: '블루베리' };
type Fruit = keyof typeof fruits; // 'apple' | 'banana' | 'blueberry';
const [fruit, changeFruit] = useState<Fruit | undefined>('apple');

// 에러 발생
const func = () => {
  changeFruit('orange');
};
```

❕ string, number, boolean 같은 원시 타입은 자동으로 추론되므로 생략할 수 있다.

### 5. 제네릭 컴포넌트 만들기

```tsx
type Option = Record<string, string>; // {[key: string]: string}
// 키와 값 모두 문자열

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void;
  //string 혹은 undefined를 매개변수로 받음. 어떤 값도 반환하지 않음.
  // 이 핸들러는 옵셔널 프로퍼티이므로, 부모 컴포넌트에서 제공하지 않아도 됨.
}

/* ... */

const FruitSelect = () => {
  const [fruit, setFruit] = useState<Fruit | undefined>();

  return (
    <Select onChange={setFruit} options={fruits} selectedOption='orange' />
  );
};
```

- `setFruit`의 매개변수 `Fruit`는 prop으로 전달돼야 하는 onChange의 매개변수 타입인 string보다 좁기 때문에 타입 에러가 발생 (`Fruit | undefined -> string`)

- Select를 사용하는 입장에서 제한된 키와 값만을 가지도록 하려면 어떻게 해야할까? 함수 컴포넌트 역시 함수이므로 제네릭을 사용한 컴포넌트를 만들 수 있다.

```tsx
// 객체 형식의 타입을 받아 해당 객체의 키값을 selectedOption, onChange의 매개변수에만 적용
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  options,
  selectedOption,
  onChange,
}: SelectProps<OptionType>) => {
  // Select 컴포넌트에 전달되는 props의 타입 기반으로 타입이 추론되어 <Select<추론된\_타입>> 형태의 컴포넌트가 생성
  // Select component implementation
};
```

```tsx
const fruits = { apple: '사과', banana: '바나나', blueberry: '블루베리' };

const FruitSelect: VFC = () => {
  // ...
  // <Select<Fruit> ... />으로 작성해도 되지만, 넘겨주는 props의 타입으로 타입 추론을 해줍니다
  // Type Error - Type "orange" is not assignable to type "apple" | "banana" | "blueberry" | undefined

  return (
    <Select options={fruits} onChange={changeFruit} selectedOption='orange' />
    // selectedOption의 prop으로 orange를 전달하지만, fruits 객체의 키중 하나가 아니므로
    // 타입 에러가 발생함.
    // Select 컴포넌트가 OptionType에 기반하여 selectedOption을 추론하기 때문임.
  );
};
```

### 6. HTMLAttributes, ReactProps 적용하기

className, id와 같은 리액트 컴포넌트의 기본 props를 추가하려면 `SelectProps`에 직접 `{className?: string; id?: string;}`을 넣어도 되지만 리액트에서 제공하는 타입을 사용하면 더 정확한 타입을 설정할 수 있다.

```tsx
type ReactSelectProps = React.ComponentPropsWithoutRef<'select'>;

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps['id'];
  className?: ReactSelectProps['className'];
  // ...
}
```

- **ComponentPropsWithoutRef는 리액트 컴포넌트의 prop 타입을 반환해주는 타입**이다.

- Type[’key’]를 활용하면 객체 형식의 타입 내부 속성값을 가져올 수 있다.

- ReactProps에서 여러 개의 타입을 가져와야 한다면 Pick 키워드를 활용하여 아래처럼 사용할 수있다.

```tsx
interface SelectProps<OptionType extends Record<string, string>>
  extends Pick<ReactSelectProps, "id" | "key" | /* ... */> {
  // ...
}
// Pick<Type, ‘key1’ | ‘key2 …>는 객체 형식의 타입에서 key1, key2…의 속성만 추출하여 새로운 객체 형식의 타입을 반환한다.
```

#### 💡 @types/react에서 속성 반환하는 타입들

- **`[HTMLAttributes<T>]`**
  https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/v17/index.d.ts#L1810-L1876

  - `HTMLAttributes<T>` 는 Element 타입을 받을 수 있는 generic 인자를 받으며, 해당 Element 가 가질 수 있는 속성들을 반환한다. 아래와 같은 타입들의 조합으로 만들어진다.
    - Element 들이 공통적으로 가지는 속성들
    - 접근성을 위한 ARIA 속성들
    - generic 인자로 받은 Element 자신의 속성들

  ```tsx
  // div요소에서 height를 제외한 새로운 타입 생성
  export type Props = OwnProps &
    Omit<React.HTMLAttributes<HTMLDivElement>, keyof OwnProps>;

  type OwnProps = {
    height: number | string;
  };
  ```

- **`[~~~Element]`**
  https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/v17/global.d.ts

  - `types/react` 의 global.d.ts 엔 다양한 [Element](https://developer.mozilla.org/en-US/docs/Web/API/Element) 에 대한 타입이 있다.
  - 예를 들어, `<button>` 태그의 타입을 원한다면 `HTMLButtonElement` 를 참조하면 브라우저에서 지원하는 [HTMLButtonElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement) 의 속성에 접근할 수 있다.

- **`HTMLAttributes<~~~Element>`**
  - button 엘리먼트의 change 이벤트를 구독하기 위해선 리액트에서는 `onChange` prop 을 이용하지만, `HTMLButtonElement` 는 브라우저에서 지원하는 `onchange` 속성을 담고 있다. 이를 해소하기 위해선 HTMLAttributes 로 HTMLButtonElement 를 Wrapping 해줘야 한다.
  - 버튼에 대한 특화된 속성을 사용하려면 HTMLAttributes말고, ButtonHTMLAttributes를 사용할 수 있다. 예시) React.ButtonHTMLAttributes사용해 확장 가능한 NativeProps를 가지는 컴포넌트 Props type

  ```tsx
    /* React.ButtonAttribute<HTMLButtonElement> 에서 type 속성을 제외한 모든 속성 포함
    <button> 요소에 대한 표준 속성들과 React에서 추가적으로 제공하는 속성들을 포함하되, type속성은 포함X
    -> type 속성 커스터마이즈를 위한 Button컴포넌트를 만들 때
    */

    type NativeProps = Omit<
      React.ButtonHTMLAttributes<HTMLButtonElement>,
      'type'
    >;
  ```

- **`[ComponentProps<T>]`**
  https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/v17/index.d.ts#L828-L833
  - `ComponentProps<T>` 는 컴포넌트의 타입을 Generic 인자로 받으며, 해당 컴포넌트가 갖는 props 를 타입으로 갖는다.
  - 주석에서는 ComponentProps 보단 ComponentPropsWithRef, ComponentPropsWithoutRef 를 사용하길 권장하고 있다. -> 실제로 레포 내에 사용된 코드가 없음
- **`[ComponentPropsWithRef<T>]`**
  https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/v17/index.d.ts#L834-L83
  - `ComponentPropsWithRef<T>` 는 forwardRef 등으로 Wrapping 된 컴포넌트일 경우 해당 컴포넌트의 Props 를 타입으로 가질 수 있도록 해줌
- **`[ComponentPropsWithoutRef<T>]`**

  - `ComponentPropsWithoutRef<T>` 는 컴포넌트가 Ref 를 갖는다면 이를 제외한 Props 를 타입으로 가지며, 만약 Ref 를 갖지 않는다면 해당 컴포넌트가 갖는 Props 를 타입으로 갖습니다.
  - 예시) `React.ComponentPropsWithoutRef` 사용하여 button HTML요소에 대한 원하는 속성을 가진 컴포넌트 Props type 정의

    ```tsx
    type Props = CombineProps<
      OwnProps,
      React.ComponentPropsWithoutRef<typeof AddonButton>
    >;

    type OwnProps = {
      container: ModalMetadata;
      onClick?: MouseEventHandler;
    };

    export default function CustomButton({
      container,
      children,
      ...props
    }: Props) {
      return <Button container={container} {...props} />;
    }
    ```

  > 출처: https://www.pumpkiinbell.com/blog/react/reusable-components

### 7. styled-components를 활용한 스타일 정의

> 리액트 컴포넌트를 만들 때 CSS 파일 대신 자바스크립트 안에 직접 스타일을 정의하는 CSS-in-JS 기법을 사용할 수 있다. 그 중 대표적인 styled-components를 활용하여 리액트 컴포넌트에 스타일 관련 타입을 추가해보자.

```tsx
// 컴포넌트에 스타일을 적용하는 데 사용되는 값을 정의
// Select 컴포넌트에 글꼴 크기(fontsize)와 현재 선택된 option의 글꼴 색상(font color)을 설정
const theme = {
  fontSize: {
    default: '16px',
    small: '14px',
    large: '18px',
  },
  color: {
    white: '#FFFFFF',
    black: '#000000',
  },
};

type Theme = typeof theme;
type FontSize = keyof Theme['fontSize'];
type Color = keyof Theme['color'];
```

```tsx
// 다음 스타일과 관련된 props를 작성하고, color와 font-size의 스타일 정의를 담은 StyledSelect를 작성
interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;

// Select 컴포넌트의 props에 SelectStyleProps 타입을 상속한다.
// Partial<Type>을 사용하면 객체 형식의 타입 내 모든 속성이 옵셔널로 설정된다.
interface SelectProps extends Partial<SelectStyleProps> {
  // ...
}

const Select = <OptionType extends Record<string, string>>({
  fontSize = 'default',
  color = 'black',
}: // ...
SelectProps<OptionType>) => {
  // ...

  return (
    <StyledSelect
      // ...
      fontSize={fontSize}
      color={color}
      // ...
    />
  );
};
```

### 8. 공변성과 반공변성

- **함수에서 공변성과 반공변성 주의!**
  - 공변성 반공병성 → 함수 간에 서로 대입할 수 있냐 없냐 그걸 따지는 것. (타입 간의 관계)
  - 이걸 알아둬야 왜 이 함수는 여기에 대입이 안되고, 여기엔 대입이 되는 지 구별할 수 있음
- **공변성 (함수의 리턴값)**

  - A(좁은 타입)가 B(넓은 타입)의 서브타입이면, `T<A>`는 `T<B>`의 서브타입이다.
  - `A ⊂ B → T<A> ⊂ T<B>` 대부분 일반적인 경우 공변성을 따른다.

  ```tsx
  /* 책 예시 */
  interface User {
    id: string;
  }

  interface Member extends User {
    nickName: string;
  }

  let users: Array<User> = [];
  let members: Array<Member> = [];

  users = members; // OK
  members = users; // Error

  /* 함수 리턴값 예시 */

  /* 가능 예시 */
  function a(x: string): number {
    // string을 받아서 number를 리턴하는 함수
    return +x;
  }
  a('1'); // 1

  type B = (x: string) => number | string; // string을 받아서 number 또는 string을 리턴하는 함수
  const b: B = a; // 서로 타입이 다른데 대입이 된다??????
  //리턴 값은 더 넓은 타입으로 대입할 수 있기 때문

  /* 불가능 예시 */

  function a2(x: string): number | string {
    return +x;
  }
  // (x: string) => number 또는 (x: string) => string

  type B2 = (x: string) => number;
  let b2: B2 = a2; // Error : 더 좁은 타입으로 대입 불가능
  ```

  → 리턴값은 더 넓은 타입으로 대입할 수 있음. 리턴값이 더 넓은 타입에서 좁은 타입으로는 대입 불가능

- **반공변성 (매개변수, 제네릭 타입을 지닌 함수)**

  - A(좁은 타입)가 B(넓은 타입)의 서브타입이면, `T<B>`는 `T<A>`의 서브타입이다.
  - `A ⊂ B → T<B> ⊂ T<A>`위의 공변성 규칙이 함수의 매개변수로 전달된 경우, 반공병성을 따른다.

  ```tsx
  /* 책 예시 */
  type PrintUserInfo<U extends User> = (user: U) => void;

  let printUser: PrintUserInfo<User> = (user) =>
    console.log(user.id, user.nickName);

  printMember = printUser; // Ok

  printUser = printMember; // Error - Property 'nickName' is missing in type 'User'
  // but required in type 'Member'

  /* 함수 매개변수 예시 */

  /* 가능 예시 */

  function a3(x: string | number): number {
    return 0;
  }
  // (x: string) => number 또는 (x: string) => string
  // => 매개변수에서는 이렇게 생각하면 안됨. 매개변수는 저 둘을 하나로 봄

  type B3 = (x: string) => number;
  let b3: B3 = a3;

  /* 불가능 예시 */

  function a4(x: string): number {
    return 0;
  }

  type B4 = (x: string | number) => number;
  let b4: B4 = a4; // Error : 매개변수는 더 넓은 타입으로 대입 불가능
  ```

  → 매개변수는 좁은 타입으로 대입이 되고, 넓은 타입으로는 대입이 안됨.

- **이변성**
  - --strict모드에서 `onChangeA`같이 함수 타입을 화살표 표기법으로 작성한다면 반공변성을 띠게 된다.
  - onChangeB같이 함수 타입을 지정하면 공변성과 반공변성을 모두 가지는 이변성을 띠게 된다.
  - 안전한 타입 가드를 위해서는 특수한 경우를 제외하고는 일반적으로 반공변적인 함수 타입을 설정하는 것이 권장된다.

```tsx
interface Props<T extends string> {
  onChangeA?: (selected: T) => void; // 화살표 표기법
  onChangeB?(selected: T): void; // 메서드 시그니처
}
```

## 8.3 정리

리액트 프로젝트에서 타입스크립트는 컴포넌트를 안전하게 조합하고 사용할 수 있도록 도와준다. 또한 다양한 훅을 활용하여 컴포넌트 내부 동작을 구현할 때도 타입을 명확하게 지정함으로써 많은 실수를 미리 방지할 수 있게 해준다. 이처럼 타입스크립트를 잘 활용하면 리액트 프로젝트를 더 안정적으로 운영할 수 있다.
