# 13. 타입스크립트와 객체 지향
## 프론트엔드에서 객체지향
객체지향은 프론트엔드 개발과는 거리가 먼 개념처럼 느껴질 수 있다. <br>
하지만 자바스크립트는 프로토타입 기반의 객체 지향 언어로 분류된다. 하지만 자바스크립트만 사용할 경우 전통적인 객체 지향 프로그래밍에서 지원하는 기능을 지원하지 않지만<br>
타입스크립트를 사용하면 이런 제약을 해결가능하다.
<br>
예를들면 private같은 접근 제어자나 추상 클래스. 추상 메서드 같은 기능을 지원한다.<br>
그리고 타입스크립트는 점진적 타이핑, 구조ㅓ적 타이핑 그리고 덕 타이핑이 결합한 언어이다. <br>
이런점들이 객체지향의 폭을 넓혀준다.<br>
프론트엔드에서 공통적으로 사용되는 컴포넌트와 비즈니스로직에서 객체 지향 원칙을 적용하여 설계하면 좋은 구조를 개발할 수 있더.
## 우아한형제들의 활용 방식
우아한형제들의 한 팀에서는 다음과 같은 설계 방식을 활용한다고 한다.
- 온전히 레이아웃만 담당하는 컴포넌트
- 컴포넌트 영역 위에서 레이아웃과 비즈니스 로직을 연결하는 커스텀 훅
- 훅 영역 위에서 객체로서 상호 협력하는 모델
- 모델 위에서 API를 해석하여 모델로 전달하는 API 레이어

### 컴포넌트 영역
아래는 장바구니 관련 다이얼로그 컴포넌트이다.
```tsx
import React from 'react';

const CartCloseoutDialog: React.VFC = () => {
  const cartStore = useCartStore();

  return (
    <Dialog
      opened={cartStore.PresentationTracker.isDialogOpen('closeout')}
      title={'마감 세일이란?'}
      onRequestClose={cartStore.PresentationTracker.closeDialog}>
      ...마크업 로직
    </Dialog>
  );
};

export default agag;

```
## 캡슐화와 추상화
캡슐화는 객체 지향에서 끊임없이 나오는 핵심 개념 중 하나이다. 올바른 협력을 설꼐하기 위해서는 적절한 캡슐화가 필요하다.
<br>
프로젝트 설계의 궁극적인 목표는 객체들이 유기적으로 협력하여 적절하게 도메인 분리를 하는것이다.
<br>
캡슐화는 다른 객체 내부의 데이터를 꺼내와서 직접 다루지 않고 요청만 보내는것이다.

```tsx
class CartStore {
    data:Cart
    constructor(cart를 받을때 필요한 인자){
      this.getCartDate(인자)
    }

//
  ...get 메소드를 이용하여 computed된 데이터를 추가적으로 만든다.
    get 모든주문대이터 {
      return this.data로 컴퓨팅된 데이터
    }

  }
```

이제 이걸 프론트엔드에서 컴포넌트를 예로 들 수 있다. 컴포넌트의 상태와 prop을 잘 다루는 것도 캡슐화의 개념에 부합하는 것이다.
#### 모승의 생각
왜 컴포넌트의 상태와 prop을 잘 다루는것이 캡슐화일지를 생각해봤다.<br>
가령 우리가 어떤 데이터를 다루는 상태가 있고 이 상태에서 파생되는 여러 상태가 있을 수 있다. 다만 똑같이 상태를 다루더라도 사람마다 다루는 방법이 천차만별이다.
<br>
다음은 내가 생각하는 나쁜 코드 예시중의 하나이다. 어떤점이 문제일까 ?
```tsx
import React, { useEffect, useState } from 'react';

const Test = () => {
  const [stateA,setStateA] = useState()
  const [stateB,setStateB] = useState()
  useEffect(()=>{
    if(stateA가 바뀌면) setStateB(어떤값)
  },[stateA])

  return (
    <div>
      <p>{stateA}</p>
      <p>{stateB}</p>
    </div>
  );
};

export default Test;
```
내가 봤을때 위 코드에서 stateB는 굳이 필요없는 코드이다. 단순 계산을 하는 함수(캡슐화된)로 stateB를 만들어낼 수 있기 때문이다.
