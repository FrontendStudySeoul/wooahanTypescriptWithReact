## 리액트 훅

리액트 훅이 도입되기 전에는 주로 클래스 기반 컴포넌트를 사용했지만 클래스 기반 컴포넌트를 사용하면 몇가지 불편한 점을 마주하게 된다.

1. 컴포넌트 간 상태 로직을 재사용하기 어렵다.
2. 클래스 컴포넌트의 생명주기 메소드에 서로 관련 없는 로직들이 얽혀 코드의 복잡성을 증가시킨다.

리액트는 이런 문제를 16.8 버전부터 훅이라는 개념을 도입해 해결했다.

### useState

### 의존성 배열을 사용하는 훅

### useRef

```tsx
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
}

interface RefObject<T> {
  readonly current: T | null;
}

useRef<HTMLInputElement>(null); // RefObject<HTMLInputElement>
useRef<HTMLInputElement | null>(null); // MutableRefObject<HTMLInputElement>
```

useRef는 MutableRefObject또는 RefObject를 반환한다. 둘의 차이점은 MutableRefObject 타입은 current의 값을 변경할 수 있는 반면 RefObject는 current 값을 변경할 수 없다.

### forwardRef

```tsx
/**
  * 🚨 Warning: MyInput: `ref` is not a prop. Trying to access it will result in
  `undefined` being returned
  * If you need to access the same value within the child component, you should pass
  it as a different prop
  */
const MyInput = ({ ref }: Props) => {
  return <input ref={ref} />;
};
```

ref라는 속성의 이름은 리액트에서 “DOM 요소 접근”이라는 특수한 목적으로 사용되기 때문에 props를 넘겨주는 방식으로 전달할 수 없다. ref를 props로 전달하기 위해선 forwardRef를 사용해야 한다.

```tsx
const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
  return ...;
});

function forwardRef<T, P = {}>(
  render: ForwardRefRenderFunction<T, P>
): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;

interface ForwardRefRenderFunction<T, P = {}> {
  (props: P, ref: ForwardedRef<T>): ReactElement | null;
  displayName?: string | undefined;
  defaultProps?: never | undefined;
  propTypes?: never | undefined;
}

type ForwardedRef<T> =
| ((instance: T | null) => void)
| MutableRefObject<T | null>
| null;
```

forwardRef의 ref 타입은 `ForwardedRef`로 `((instance: T | null) => void)`, `MutableRefObject<T | null>`, `null` 타입을 가진 유니언 타입이다. 유니언 타입이기 때문에 컴포넌트 내부에서 ref의 current에 접근하려면 아래와 같은 타입 에러가 발생한다.

```tsx
const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
	ref.current // '((instance: HTMLCanvasElement | null) => void) | MutableRefObject<HTMLCanvasElement | null>' 형식에 'current' 속성이 없습니다.

	return ...;
});
```

forwardRef에서 current에 접근하기 위해선 `MutableRefObject` 타입이 추론되도록 타입을 좁혀야한다.

```tsx
const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
	// ((instance: HTMLCanvasElement | null) => void) 타입이 아닐 경우
	if(ref !== 'function') {
		ref.current
	}

	return ...;
});
```

### useImperativeHandle

useImplerativeHandle 훅은 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스터마이징된 메서드를 호출할 수 있게 된다.

```tsx
type CreateFormHandle = Pick<HTMLFormElement, "submit">;

type CreateFormProps = {
  defaultValues?: CreateFormValue;
};

const JobCreateForm: React.ForwardRefRenderFunction<
  CreateFormHandle,
  CreateFormProps
> = (props, ref) => {
  // useImperativeHandle Hook을 사용해서 submit 함수를 커스터마이징한다
  useImperativeHandle(ref, () => ({
    submit: () => {
      /* submit 작업을 진행 */
    },
  }));

  // ...
};

const CreatePage: React.FC = () => {
  // `CreateFormHandle` 형태를 가진 자식의 ref를 불러온다
  const refForm = useRef<CreateFormHandle>(null);

  const handleSubmitButtonClick = () => {
    // 불러온 ref의 타입에 따라 자식 컴포넌트에서 정의한 함수에 접근할 수 있다
    refForm.current?.submit();
  };

  // ...
};
```

## 커스텀 훅
