# 10. 상태관리

> 1. 상태란 무엇인지 설명하고, 지역상태, 전역 상태, 서버 상태로 분류해 상태를 잘 관리하기 위해서는 어떻게 해야하는지에 대한 내용
> 2. 외부 상태 관리 라이브러리의 동작 방식에 대한 내용

## 상태

**상태** : 렌더링에 영향을 줄 수 있는 동적인 데이터

리액트에서의 상태는 시간이 지나며 변할 수 있는 동적 데이터이자, 렌더링 결과물에 영향을 줄 수 있는 존재다.

리액트 앱 내에서 상태는 크게 세가지로 지역상태, 전역상태, 서버상태로 나뉜다.

리액트에서 제공하는 내부 API만으로 상태를 관리할 수 있지만, 성능 문제와 복잡성으로 `Redux`, `MobX`, `Recoil` 같은 외부 상태 관리 라이브러리를 주로 활용한다.

- **지역 상태**: 컴포넌트 내부에서 사용되는 상태, useState 훅을 많이 사용하며 경우에 따라서 `useReducer` 같은 훅을 사용하기도 함. ex) 체크박스의 체크여부, 폼의 입력 값

- **전역 상태**: 앱 전체에서 공유하는 상태, 상태가 변경되면 전역상태를 공유하는 컴포넌트들도 업데이트 된다. 또한, `prop drilling` 문제를 피하기 위해 지역 상태를 해당 컴포넌트들 사이의 전역 상태로 공유할 수 있다.

  > **prop drilling** : props로 데이터를 전달하는 과정에서 중간 컴포넌트는 해당 데이터가 필요없음에도 불구하고 자식 컴포넌트에 전달하기 위해 props를 전달하는 과정을 뜻한다. 컴포넌트의 수가 많아지면 `prop drilling`의 문제로 인해 코드가 복잡해질 수 있다.

- **서버 상태**: 사용자 정보, 글 목록 등 외부 서버에 저장해야하는 상태, 지역, 전역 변수와 동일한 방법으로 관리되며 최근 `react-query`, `SWR`같은 외부 라이브러리를 사용하여 관리하기도 한다.

### 2) 상태를 잘 관리하기 위한 가이드

상태는 애플리케이션의 복잡성을 증가시키고 동작을 예측하기 어렵게 만든다.
어떠한 값을 상태로 정의할 때는 다음 2가지 사항을 고려해야 한다.

- **시간이 지나도 변하지 않는다면 상태가 아니다.**

  시간이 지나도 변하지 않는 값이라면 객체 참조 동일성을 유지하는 방법을 고려할 수 있다.

  컴포넌트가 마운트될 때만 스토어 객체 인스턴스를 생성하고, 컴포넌트가 언마운트될 때까지 해당 참조가 변하지 않는다고 가정해보자.

  단순히 상수 변수에 저장하여 사용할 수도 있지만, 이러한 방식은 렌더링될 때마다 새로운 객체 인스턴스가 생성되므로 불필요한 리렌더링이 자주 발생할 수 있다.

  따라서 리액트의 다른 기능을 활용하여 컴포넌트 라이프사이클 내에서 마운트될 때 인스턴스가 생성되고, 렌더링될 때마다 동일한 객체 참조가 유지되도록 구현해야 한다.

  ```tsx
  import React from 'react';

  const Component: React.VFC = () => {
  const store = new Store();
  return (
      <StoreProvider store={store}>
          <Children>
      </StoreProvider>
     )  ;
  };
  ```

  객체 참조 동일성을 유지하기 위해 널리 사용되는 방법 중 하나는 메모이제이션이다. `useMemo`를 사용하여 컴포넌트가 마운트될 때만 객체 인스턴스를 생성하고 이후 렌더링에서는 이전 인스턴스를 재활용할 수 있도록 구현할 수 있다.

  ```tsx
  const store = useMemo(() => new Store(), []);
  ```

  하지만 `useMemo`는 오로지 성능 향상을 위한 용도로만 사용하라고 공식문서에 언급되어있으며 리액트는 메모리 확보를 위해 이전 메모이제이션 데이터를 삭제할 수 있다. 따라서 `useMemo`가 없어도 올바르게 작동되도록 코드를 작성한 뒤 성능개선을 목표로 `useMemo`를 추가하는 것이 적절한 접근 방식이다.

  원하는 방식으로 동작하게 하는 방법은 아래와 같이 2가지가 있다.

  - useState의 초깃값만 지정하는 방법

    `useState(new Store())` 의 방식은 객체 인스턴스가 사용되지 않더라도 렌더링마다 생성되어 초깃값 설정에 큰 비용이 소모될 수 있다. 따라서 `useState(()=> new Store())`와 같이 초깃값을 계산하는 콜백을 지정하는 방식(지연 초기화 방식)을 사용한다.

    다만 useState를 사용하는 것은 의미론적으로는 좋은 방법이 아니다. 처음에는 상태를 시간이 지나면서 변화되어 렌더링에 영향을 주는 데이터로 정의했지만 현재의 목적은 모든 렌더링 과정에서 객체의 참조를 동일하게 유지하고자 하는 것이기 때문이다.

  - useRef를 사용하는 방법

    리액트 공식 문서에 따르면` useRef`가 동일한 객체 참조를 유지하려는 목적으로 사용하기에 가장 적합한 훅이다. `useRef()`의 인자로 `new Store()`를 바로 사용하면 `useState`와 마찬가지로 렌더링마다 불필요한 인스턴스가 생성되므로 다음과 같이 사용해야한다.

    ```tsx
    const store = useRef<Store>(null);

    if (!store.current) {
      store.current = new Store();
    }
    ```

    가독성 등의 이유로 팀 내에서 합의된 컨벤션으로 저장된 것이 아니라면 동일한 객체 참조를 할 때는 `useState`보다는 `useRef`를 사용할 것을 권장한다.

- **파생된 값은 상태가 아니다.**

  내려받은 props나 기존 상태에서 계산될 수 있는 값은 상태가 아니다.
  다른 값에서 파생된 값을 상태로 관리하게 되면 기존 출처와는 다른 새로운 출처에서 관리하게 되는것이므로 정확성과 일관성을 보장하기 어렵다.

  ```tsx
  import React, { useState } from 'react';

  type UserEmailProps = {
    initialEmail: string;
  };

  const UserEmail: React.VFC<UserEmailProps> = ({ initialEmail }) => {
    const [email, setEmail] = useState(initialEmail);
    const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
      setEmail(event.target.value);
    };

    return (
      <div>
        <input type="text" value={email} onChange={onChangeEmail} />
      </div>
    );
  };
  ```

  위 컴포넌트에서 전달받은 `initialEmail` props의 값이 변경되어도 input 태그의 value는 변경되지 않는다. useState의 초깃값으로 설정한 값은 컴포넌트가 마운트될 때 한 번만 `email` 상태의 값으로 설정되며 이후에는 독자적으로 관리된다.

  `useEffect`를 사용한 해결방법을 생각할 수 있는데 이는 좋은 방법이 아니다.
  사용자가 값을 변경한 다음 `initialEmail` prop이 변경된다면 input태그의 value는 사용자의 입력을 무시하고 부모 컴포넌트로부터 전달된` initialEmail` prop의 값을 value로 설정할 것이다.

  `useEffect`를 사용한 동기화는 리액트 외부 데이터와 동기화할때만 사용해야하며 내부 데이터를 상태와 동기화하는데 사용하면 안된다. 왜냐하면 이는 개발자가 추적하기 어려운 오류를 발생시킬 수 있기 때문이다.

  ```tsx
  import { useState, useEffect } from 'react';

  const [email, setEmail] = useState(initialEmail);

  useEffect(() => {
    setEmail(initialEmail);
  }, [initialEmail]);
  ```

  `useEffect`를 사용한 동기화보다는 상위 컴포넌트에서 상태를 관리하도록 도와주는 상태끌어올리기(Lifting State Up) 기법을 사용하여 단일한 출처에서 데이터를 사용하도록 변경해줘야한다.

  이를 이용해 `UserEmail`에서 관리하던 상태를 부모 컴포넌트로 옮겨서 `email `데이터의 출처를 props 하나로 통일할 수 있다.

  ```tsx
  import React, { useState } from 'react';

  type UserEmailProps = {
    email: string;
    setEmail: React.Dispatch<React.SetStateAction<string>>;
  };

  const UserEmail: React.VFC<UserEmailProps> = ({ email, setEmail }) => {
    const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
      setEmail(event.target.value);
    };
    return (
      <div>
        <input type="text" value={email} onChange={onChangeEmail} />
      </div>
    );
  };
  ```

  위와 같이 두 컴포넌트에서 동일한 데이터를 가진 경우에는 동기화가 아닌, 상태 끌어올리기를 사용하여 SSOT를 지킬 수 있도록 해야한다.

  다음 예시는 아이템 목록이 변경될 때마다 선택된 아이템 목록을 가져오도록 `useEffect`로 동기화 작업을 하고 있다.

  ```tsx
  import {useState, useEffect} from 'react';

  const [items, setItems] = useState<Item[]>([]);
  const [selectedItems, setSelectedItems] = useState<Item[]>([]);

  useEffect(() => {
   setSelectedItems(items.filter((item) = > item.isSelected));
  }, [items]);
  ```

  여기서의 가장 큰 문제는 `items`와 `selectedItems`가 동기화되지 않을 수 있다는 것이다. 여기서는 새로운 상태로 정의함으로써 단일 출처가 아닌 여러 출처를 가지게 되었다. 이에 따라 동기화 문제가 발생할 수 있다.

  이러한 문제를 해결하는 간단한 방법으로는 상태를 정의하지 않고 **계산된 값을 자바스크립트 변수로 담는 것**이다.

  그러면 `items`가 변경될 때마다 컴포넌트가 새로 렌더링되어 `selectedItems`를 다시 계산하게 된다. 이런 식으로 단일 출처를 가지면서 원하는 동작을 수행할 수 있다.

  ```tsx
  import {useState} from 'react';

  const [items, setItems] = useState<Item[]>([]);
  const selectedItems = items.filter((item) = > item.isSelected);
  ```

  성능 측면에서는` items`와 `selectedItems `2가지 상태를 유지하면서 `useEffect`로 동기화하는 과정을 거치면 `selectedItems` 값을 얻기 위해 두번의 렌더링이 발생한다

  자바스크립트 변수에 계산 결과를 담는 방법은 리렌더링 횟수를 줄일 수 있다. 다만 이 경우에는 매번 렌더링될 때마다 계산을 수행하므로 계산 비용이 크다면 성능 문제가 발생할 수 있다. 이럴때에는 `useMemo`를 사용하여 `items`가 변경할 때만 계산을 수행하고 결과를 메모이제이션하여 성능을 개선할 수 있다.

  ```tsx
  import { useState, useMemo } from 'react';

  const [items, setItems] = useState<Item[]>([]);
  const selectedItems = useMemo(() => veryExpensiveCalculation(items), [items]);
  ```

- **useState와 useReducer, 어떤 것을 사용해야 할까**
  `useState` 대신 `useReducer` 사용을 권장하는 경우는 크게 두가지가 있다.

      - 다수의 하위필드를 포함하고 있는 복잡한 상태 로직을 다룰 때

      - 다음 상태가 이전 상태에 의존적일 때

예를 들어, 배달의민족 리뷰 리스트를 필터링하여 보여주기 위한 쿼리를 상태로 저장해야 한다면 검색 날짜, 리뷰점수, 키워드 등 많은 하위 필드를 가지게 된다.

```tsx
type DateRangePreset = 'TODAY' | 'LAST_WEEK' | 'LAST_MONTH';
type ReviewRatingString = '1' | '2' | '3' | '4' | '5';

interface ReviewFilter {
  startDate: Date;
  endDate: Date;
  dateRangePreset: Nullable<DateRangePreset>;
  keywords: string[];
  ratings: ReviewRatingString[];
}

interface State {
  filter: ReviewFilter;
  page: string;
  size: number;
}
```

이렇게 많은 하위필드를 가지는 데이터를 `useState`로 관리하면 상태를 업데이트할 때마다 오류 가능성이 증가한다. 또한 특정한 업데이트 규칙이 있다면 `useState`로는 한계가 있다.

`useReducer`는 '무엇을','어떻게' 변경할지 분리하여 `dispatch`를 통해 어떤 작업을 할지 액션으로 넘기고 `reducer` 함수 내부에서 상태를 업데이트 하는 방식을 정의한다.

```tsx
import React, { useReducer } from 'react';

type Action =
  | { payload: ReviewFilter; type: 'filter' }
  | { payload: number; type: 'navigate' }
  | { payload: number; type: 'resize' };

const reducer: React.Reducer<State, Action> = (state, action) => {
  switch (action.type) {
    case 'filter':
      return {
        filter: action.payload,
        page: 0,
        size: state.size,
      };
    case 'navigate':
      return {
        filter: state.filter,
        page: action.payload,
        size: state.size,
      };
    case 'resize':
      return {
        filter: state.filter,
        page: 0,
        size: action.payload,
      };
    default:
      return state;
  }
};

// useReducer 사용
const [state, dispatch] = useReducer(reducer, getDefaultState());
// dispatch 예시
dispatch({ payload: filter, type: 'filter' });
dispatch({ payload: page, type: 'navigate' });
dispatch({ payload: size, type: 'resize' });
```

위는 리뷰 쿼리 상태에 대한 `reducer`를 정의하여 `useReducer`와 dispatch를 사용한 코드다.

이외에도 boolean 상태를 토글하는 액션만 사용하는 경우에는 `useState` 대신 `useReducer`를 사용하곤 한다.

```tsx
import { useReducer } from 'react';

//Before
const [fold, setFold] = useState(true);

const toggleFold = () => {
  setFold((prev) => !prev);
};

// After
const [fold, toggleFold] = useReducer((v) => !v, true);
```

### 3) 전역 상태 관리와 상태 관리 라이브러리

상태를 전역 상태로 정의할 때 크게 리액트 컨텍스트 API를 사용하는 방법과 외부 상태 라이브러리를 사용하는 방법이 있다.

- 컨텍스트 API(Context API)

  컨텍스트 API는 다른 컴포넌트들과 데이터를 쉽게 공유하기 위한 목적으로 제공되는 API로 prop drilling 같은 문제를 해결하기 위한 도구로 활용된다.

  컨텍스트 API를 사용하면 데이터를 컨텍스트로 제공하고 해당 컨텍스트를 구독한 컴포넌트에서만 데이터를 읽을 수 있다.

  아래와 같이 TabGroup 컴포넌트와 Tab 컴포넌트에 type이라는 prop을 전달한 경우, TabGroup에만 type을 전달하고 Tab 컴포넌트의 구현 내에서도 사용하려면 Context API를 사용하면 된다.

  ```tsx
  // 현재 구현된 것 - TabGroup 컴포넌트뿐 아니라 모든 Tab 컴포넌트에도 type prop을 전달

  <TabGroup type='sub'>
    <Tab name='텝 레이블 1' type='sub'>
     <div>123</div>
    </Tab>
    <Tab name='텝 레이블 2' type='sub'>
     <div>123</div>
    </Tab>
  </TabGroup>

  // 원하는 것 - TabGroup 컴포넌트에만 전달
  <TabGroup type='sub'>
    <Tab name='텝 레이블 1'>
     <div>123</div>
    </Tab>
    <Tab name='텝 레이블 2'>
     <div>123</div>
    </Tab>
  </TabGroup>
  ```

  다음과 같이 상위 컴포넌트 구현 부에 컨텍스트 프로바이더를 넣어주고, 하위 컴포넌트에서 해당 컨텍스트를 구독하여 데이터를 읽어오는 방식을 사용할 수 있다.

  ```tsx
  import { FC } from 'react';

  const TabGroup: FC<TabGroupProps> = (props) => {
    const { type = 'tab', ...otherProps } = useTabGroupState(props);
    /* ... 로직 생략 */
    return (
      <TabGroupContext.Provider value={{ ...otherProps, type }}>
        {/* ... */}
      </TabGroupContext.Provider>
    );
  };

  const Tab: FC<TabProps> = ({ children, name }) => {
    const { type, ...otherProps } = useTabGroupContext();
    return <>{/* ... */}</>;
  };
  ```

  하지만 컨텍스트 API를 사용한 전역 상태 관리는 대규모 어플리케이션이나 성능이 중요한 애플리케이션에서는 권장되지 않는 방법이다.

  컨텍스트 프로바이더의 props로 주입된 값이나 참조가 변경될 경우 해당 컴포넌트를 구독하는 모든 컴포넌트가 리렌더링되기 때문이다.

  애플리케이션이 커지고 전역 상태가 많아질수록 불필요한 리렌더링 상태의 복잡도가 증가한다.

## 10.2 상태관리 라이브러리

범용적으로 사용하는 상태관리 라이브러리는 대표적으로 MobX, Redux, Recoil, Zustand 가 있다.

### 1) MobX

- 객체 지향 프로그래밍과 반응형 프로그래밍 패러다임의 영향을 받은 라이브러리-

```tsx
import { observer } from 'mobx-react-lite';
import { makeAutoObservable } from 'mobx';

class Cart {
  itemAmount = 0;

  constructor() {
    makeAutoObservable(this);
  }

  increase() {
    this.itemAmount += 1;
  }

  reset() {
    this.itemAmount = 0;
  }
}

const myCart = new Cart();
const CartView = observer(({ cart }) => (
  <button onClick={() => cart.reset()}>
    amount of cart items: {cart.itemAmount}
  </button>
));

ReactDOM.render(<CartView cart={myCart} />, document.body);
```

### 2) Redux

- 함수형 프로그래밍의 영향을 받은 라이브러리
- 보일러 플레이트가 있고 러닝커브가 높다

```tsx
import { createStore } from 'redux';

function counter(state = 0, action) {
  switch (action.type) {
    case 'PLUS':
      return state + 1;
    case 'MINUS':
      return state - 1;
    default:
      return state;
  }
}

let store = createStore(counter);

store.subscribe(() => console.log(store.getState()));

store.dispatch({ type: 'PLUS' });
// 1
store.dispatch({ type: 'PLUS' });
// 2
store.dispatch({ type: 'MINUS' });
// 1
```

### 3) Recoil

- `atom`과 `selector`를 통해 상태를 관리하는 라이브러리
- 보일러 플레이트가 적고 난이도가 낮아 배우기 쉽다.

```tsx
import React from 'react';
import { RecoilRoot } from 'recoil';
import { TextInput } from './';

function App() {
  return (
    <RecoilRoot>
      <TextInput />
    </RecoilRoot>
  );
}
```

`Atom`은 상태의 일부를 나타내며 어떤 컴포넌트에서든 읽고 쓸 수 있도록 제공된다.

```tsx
import { atom } from 'recoil';

// Atom 생성
export const textState = atom({
  key: 'textState',
  default: '',
});

import { useRecoilState } from 'recoil';
import { textState } from './';

export function TextInput() {
  const [text, setText] = useRecoilState(textState);
  const onChange = (event) => {
    setText(event.target.value);
  };
  return (
    <div>
      <input type="text" value={text} onChange={onChange} />
      <br />
      Echo: {text}
    </div>
  );
}

setInterval(() => {
  myCart.increase();
}, 1000);
```

### 4) Zustand

- Flux 패턴을 사용

```tsx
import { create } from 'zustand';

const useBearStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));

function BearCounter() {
  const bears = useBearStore((state) => state.bears);

  return <h1>{bears} around here ...</h1>;
}

function Controls() {
  const increasePopulation = useBearStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>Plus</button>;
}
```
