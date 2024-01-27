# 7장 비동기 호출
## 7-1 API 요청
### fetch로 API 요청하기
장바구니를 조회해서 볼 수 있는 기능을 만들기 위하여 AJAX 요청을 하게 됐다.
```tsx
const CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0);
  useEffcet(() => {
    fetch('카트정보 URL').then(({ cartItem }) => {
      setCartCount(cartItem.length);
    });
  }, []);
};
return <>{/*cartCount 상태를 이용하여 컴포넌트 렌더링*/}{</>
```
위처럼 해당 API를 요청하는 여러곳에서 API URL을 복붙하여 사용하게 됐다. 이런 코드들은 추후 변경 요구에 취약하다.

### 서비스 레이어로 분리하기
만약 API 요청 정책이 추가되어 코드가 변경될 수 있다는 것을 감안한다면, 비동기 호출코드는 컴포넌트(View)와 분리되어 다른 영역(서비스 레이어)에서 처리되어야 한다.
<br>
위 코드에서 예를들면 마크업부분을 제외한 코드가 서비스 레이어로 이동해야한다. 그리고 컴포넌트에서 서비스 레이어의 비동기 함수를 호출하고 그 결과값을 마크업에 넣어 렌더링해야한다.
<br>
하지만 단순히 코드를 분리한다고 위에서 이전 챕터에서 나온 요구사항을 해결할 수 있는건 아니다.

### Axios 활용하기
fetch는 내장 라이브러리이기 때문에 따로 임포트하거나 설치할 피료가 없다. 하지만 많은 기능(baseURL이나 header interceptor 등)을 사용하려면 직접 구현해야한다.
<br>
그래서 Axios를 이용하여 위 부분의 많은 기능을 위임하여 사용한다.
```tsx
//기본적인 API 엔트리를 사용하는 Axios 인스턴스
const apiRequester: AxiosInstance = axios.create(defaultConfig);

//다른 API 엔트리인 "https://api.baemin.or/" 주소로 요청을 보내기 위한 Axios 인스턴스
const orderApiRequester: AxiosInstance = axiost.create({
  baseURL: 'https://api.baemin.or/',
  ...defaultConfig,
});

//다른 API 엔트리인 "https://api.order/" 주소로 요청을 보내기 위한 Axios 인스턴스
const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: 'https://api.order/',
  ...defaultConfig,
});
```
