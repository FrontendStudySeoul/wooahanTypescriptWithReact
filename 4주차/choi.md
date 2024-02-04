# 8장 JSX에서 TSX로

## 목차

- 타입스크립트로 리액트 코드를 작성 시 @types/react 패키지에 정의된 리액트 내장 타입을 사용
- 대표적인 리액트 컴포넌트 타입
- 함수 컴포넌트의 반환 타입
- 리액트에서 기본 HTML 요소 타입 활용하기

## 대표적인 리액트 컴포넌트 타입

### 1. 클래스 컴포넌트 타입

```tsx
interface Component<P = {}, S = {}, SS = any> extends ComponentLifecycle<P, S, SS> {}

class Component<P, S> {
	/* ... 생략 */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```
- P와 S는 props와 상태를 의미하며 각각 타입을 제네릭으로 받음
- 상태가 있는 컴포넌트일 때는 제네릭의 두번째 인자로 타입을 넘겨주면 상태에 대한 타입을 지정 가능
  
### 2. 함수 컴포넌트 타입

- React.FC & React.VFC : 리액트에서 함수 컴포넌트의 타입 지정을 위해 제공되는 타입
   - React.FC : children props 허용 O
   - React.VFC : children props 허용 X

	⇒ 리액트 v18 이후, React.VFC 가 deprecated 되고 React.FC에서는 children이 삭제됐다. 따라서 React.VFC 대신 React.FC를 사용하거나 props타입, 반환 타입을 직접 지정해 타이핑해야 한다.

## Children props 타입 지정

```tsx
type PropsWithChildren<P> = P & {children?: ReactNode | undefined };
```
- 가장 보편적인 children 타입 : ReactNode | undefined
- ReactNode
  - ReactElement 외에도 boolean, number 등 여러 타입을 포함하고 있는 타입
  - 더 구체적으로 타이핑 하는 용도에는 부적합
    - (ex) 특정 문자열만 허용하고 싶을 때는 children에 대해 추가로 타이핑

```tsx
// example 1
type WelcomeProps = {
	children: "천생연분" | "더 귀한 분" | "귀한 분" | "고마운 분"
};

// example 2
type WelcomeProps = {
	children: string;
};

// example 3
type WelcomeProps = {
	children: ReactElement;
};
```



## 함수 컴포넌트의 반환 타입

### 포함 관계

- 세 가지 모두 모두 리액트의 요소를 나타내는 타입
  
![image](https://github.com/FrontendStudySeoul/wooahanTypescriptWithReact/assets/25587196/2333ddda-0c0a-416d-8c3c-3aba28ba39a1)

- React.createElement의 반환 타입은 ReactElement와 JSX.Element
- 리액트는 실제 DOM이 아니라 가상 DOM을 기반으로 렌더링하는데 가상 DOM의 엘리먼트는 ReactElement 형태로 저장


### React.Node

- ReactNode는 ReactElement의 superset
- ReactNode는 리액트의 render 함수가 반환할 수 있는 모든 형태를 담음
- 즉, 리액트 컴포넌트가 가질 수 있는 모든 타입을 의미
- prop으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 하고싶을 때 유용
- 
```tsx
type ReactFragment = {} | Iterable<ReactNode>; // ReactNode의 배열 형태
type ReactNode = 
	| ReactChild
	| ReactFragment
	| ReactPortal
	| boolean
	| null
	| undefined;
```
 
### ReactElement

- 트랜스파일러는 JSX 문법을 createElement 메서드 호출문으로 변환하여 리액트 엘리먼트를 생성
- 리액트는 리액트 엘리먼트 객체를 읽어서 DOM을 구성
- ReactElement 타입은 JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입

### JSX.Element

```tsx
declare global {
	namespace JSX {
		interface Element extends React.ReactElement<any, any> {}
	}
}
```

- JSX.Element 타입은 리액트의 ReactElement를 제너릭으로 확장하고 있는 타입
- props와 타입 필드를 any로 가짐
- 글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의할 수 있는 유연성 제공
- 컴포넌트 타입을 재정의하거나 변경하는 것에 용이
- 리액트 엘리먼트를 prop으로 전달받아 render props 패턴으로 컴포넌트를 구현할 때 유용

### prop을 JSX.Element 타입으로 선언

- icon prop을 JSX.Element 타입으로 선언함으로써 해당 prop에는 JSX 문법만 삽입 가능
- icon.props에 접근하여 prop으로 넘겨받은 컴포넌트의 상세 데이터를 가져올 수 있다
  - JSX.Element 대신에 ReactElement를 사용하는 것이 추론 관점에서 더 유용

```tsx
interface Props {
	icon: JSX.Element;
}

const Item = ({ icon }: Props) => {
	// prop으로 받은 컴포넌트의 props에 접근할 수 있다
	const iconSize = icon.props.size;

	return (<li>{icon}</li>);
};

// icon prop에는 JSX.Element 타입을 가진 요소만 할당할 수 있다
const App = () => {
	return <Item icon={<Icon size={14} />} />
```



### prop을 ReactElement 타입으로 선언

- 이처럼 코드를 작성하면 icon.props에 접근할 때 어떤 props가 있는지 IDE에 표시되어 추론 가능
```tsx
interface IconProps {
	size: number;
}

interface Props {
	// ReactElement의 props 타입으로 IconProps 타입 지정
	icon: React.ReactElement<IconProps>;
}

const Item = ({ icon }: Props) => {
	// icon prop으로 받은 컴포넌트의 props에 접근하면, props의 목록이 추론된다
	const iconSize = icon.props.size;

	return <li>{icon}</li>;
};
```

### 리액트에서 기본 HTML 요소 타입 활용하기

- 리액트에서 HTML button 태그를 확장한 Button 컴포넌트를 생성해보기
- 새롭게 만든 컴포넌트는 HTML 태그의 속성 타입을 활용해 타입을 지정

### DetailedHTMLProps

- 새롭게 만든 Button 컴포넌트도 onClick 이벤트 핸들러를 지원해야만 일관성과 편의성을 지킬 수 있다

```tsx
// 기존 HTML button과 같은 역할을 하면서도 새로운 기능이나 UI가 추가된 형태
const SquareButton = () => <button>정사각형 버튼</button>;
```

```tsx
// ButtonProps의 onClick 타입은 실제 HTML button 태그의 onClick 이벤트 핸들러 타입과 동일하게 할당되는 것을 확인할 수 있다

type NativeButtonProps = React.DetaildHTMLProps<
	React.ButtonHTMLAttributes<HTMLButtonElement>,
	HTMLButtonElement
>;

type ButtonProps = {
	onClick?: NativeButtonProps["onClick"];
};
```

### ComponentWithoutRef

```tsx
// React.ComponentPropsWIthoutRef 타입은 아래와 같이 활용
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
	onClick?: NativaButtonType["onClick"];
};
```

### 언제 ComponentPropsWithoutRef를 사용하면 좋을까

- 이외에도 HTML 태그의 속성을 지원하기 위한 다양한 타입이 존재
- HTMLProps, ComponentPropsWithRef 등

```tsx
// HTML button 태그와 동일한 역할을 하지만 커스텀한 UI를 적용해 재사용성을 높이기 위한 Button 컴포넌트
const Button = () => {
	return <button>버튼</button>;
};
```

- HTML 태그를 대체하는 역할이므로 아래와 같이 기존 button 태그의 HTML 속성을 props로 받을 수 있게 지원

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
	React.ButtonHTMLAttributes<HTMLButtonElement>,
	HTMLButtonElement
>;

const Button = (props: NativeButtonProps) => {
	return <button {...props}>버튼</button>;
};
```

- 클래스 컴포넌트와 함수 컴포넌트에서 ref를 props로 받아 전달하는 방식에 차이가 있다
	- 클래스 컴포넌트에서 ref 객체는 마운트된 컴포넌트의 인스턴스를 current 속성값으로 가짐
	- 함수 컴포넌트에서는 생성된 인스턴스가 없기 때문에 ref에 기대한 값이 할당되지 않음

```tsx
// 클래스 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
class WrappedButton extends React.Component {
	constructor() {
		this.buttonRef = React.createRef();
	}

	render() {
		return (
			<div>
				<Button ref={this.buttonRef} />
				{/* 클래스 컴포넌트는 전달받은 ref가 Button 컴포넌트의 button 태그를 바라봄 */}
			</div>
		);
	}
}
```

```tsx
// 함수 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
const WrappedButton = () => {
	const buttonRef = useRef();

	return (
		<div>
			<Button ref={buttonRef} />
			{/* 함수 컴포넌트는 전달받은 ref가 Button 컴포넌트의 button 태그를 바라보지 않음 */}
		</div>
	);
};
```


### React.forwardRef

```tsx
// forwardRef를 사용해 ref를 전달받을 수 있도록 구현
const Button = forwardRef((props, ref) => {
	return <button ref={ref}{...props}>버튼</button>;
});

// buttonRef가 Button 컴포넌트의 button 태그를 바라볼 수 있다
const WrappedButton = () => {
	const buttonRef = useRef();

	return (
		<div>
			<Button ref={buttonRef} />
		</div>
	);
};
```

```tsx
// button 태그에 대한 HTML 속성을 모두 포함하지만, ref 속성은 제외
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

// forwardRef의 제네릭 인자를 통해 ref에 대한 타입으로 HTMLButtonElement를, props에 대한 타입으로 NativeButtonType을 정의했다
const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
	return (
		<button ref={ref}{...props}>
			버튼
		</button>
	);
});
```

- 함수 컴포넌트에서 props로 ref를 포함하는 타입을 사용하면 실제로는 동작하지 않는 ref를 받도록 타입이 지정된다
- HTML 속성을 확장하는 props를 설계할 때는 ComponentPropsWIthoutRef 타입을 사용
- ref가 실제로 forwardRef와 함께 사용될 때만 props로 전달되도록 타입을 정의하는 것이 안전하다.



