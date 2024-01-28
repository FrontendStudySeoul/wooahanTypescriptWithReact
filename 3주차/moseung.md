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
각 서버가 담당하는 부분이 다르거나 만약 새로운 프로젝트의 일부로 홤되면 기존 Baset URL과는 다른 새로운 URL로 요청할 수 있다.
<br>
만약 그럴경우에는 두개의 instance를 만들어 사용한다. 여기서는 Base URL을 예시로 들었지만 header나 interceptor도 포함이다.

### Axios 인터셉터 사용하기
각각의 requester는 서로 다른 역할을 담당하게 다른 헤더를 설정해야할 수도 있다.

```tsx
// 요청을 보내기 전 실행
axios.interceptors.request.use();

// 응답을 받은 후 실행
axios.interceptors.response.use();
```

```tsx
// API 엔트리로 요청을 보내기 위한 Axios 인스턴스를 생성하고, 요청 시에 5초 동안 대기하는 timeout 설정
const apiRequester : AxiosInstance = axiost.create({
    baseURL:"https://api.baemin.com/",
    timeout:5000
});

// Axios 요청 구성(config)에 특정 헤더 값을 추가하는 함수
const setRequesterDefaultHeader = (requestConfig: AxiosRequestConfig) => {
    const config = requestConfig;

    config.headers = {
        ...config.headers,
        "Content-Tyep":"application/json;charset=utf8",
        user:getUserToken(),
        agent:getAgent()
    }

    return config;
}


// Axios 주문 요청 구성(config)에 특정 헤더 값을 추가하는 함수
const setOrderRequesterDefaultHeader = (requestConfig: AxiosRequestConfig) => {
    const config = requestConfig;

    config.headers = {
        ...config.headers,
        "Content-Tyep":"application/json;charset=utf8",
        "order-client": getOrderClienToken();
    }

    return config;
}

// apiRequester에 요청 전에 setRequesterDefaultHeader 함수를 호출하여
// 기본 헤더 값을 설정하는 Axios 요청 인터셉터(interceptors)
apiRequester.interceptors.request.use(setRequesterDefaultHeader)


// 주문 API를 위한 Axios 인스턴스를 생성하고,
// 해당 API의 기본 URL과 기본 설정인 defaultConfig를 사용
const orderApiRequester: AxiosInstance = axios.create({
    baseURL:orderApiBaseUrl,
    ...defaultConfig,
})

//orderApiRequester에 요청 전에 setOrderRequesterDefaultHeader 함수를 호출하여 주문 API 전용 헤더 값을 설정하는 Axios 요청 인터셉터가 등록
orderApiRequester.interceptors.reuquest.use(setOrderRequesterDefaultHeader)

// 응답을 처리하는데 있어서 httpErrorHandler를 사용하도록 설정
orderApiRequester.interceptors.response.use({
    (response: AxiosResponse) => response,
    httpErrorHandler
})

//주문 카트 API를 위한 Axios 인스턴스를 생성하고, 해당 API의 기본 URL과 기본 설정인 defaultConfig를 사용
const orderCartApiRequester: AxiosInstance = axios.create({
    baseURL:orderCartApiBaseUrl,
    ...defaultConfig,
})

//orderCartApiRequester에 요청 전에 setRequesterDefaultHeader 함수를 호출하여 기본 헤더 값을 설정하는 Axios 요청 인터셉터가 등록
orderCartApiRequester.interceptors.request.use(setRequesterDefaultHeader);

```

요청 옵션에 따라 다른 인셉터를 만들기 위해 [빌더 패턴](https://refactoring.guru/ko/design-patterns/builder)을 추가하여 APIBuilder같은 클래스 형태로 구성하기도 한다.

### API 응답 지정하기
```tsx
interface Response<T> {
  data: T;
  status: string;
  serverDateTime: string;
  errorCode?: string;
  errorMessage?: string;
}

// 카트 정보를 가져오기 위한 API 요청
// AxiosPromise를 반환하며, 해당 Promise의 제네릭 타입은 Response<FetchCartResponse>로 정의
// FetchCartResponse는 서버에서 받아온 카트 정보에 대한 타입
const fetchCart = (): AxiosPromise<Response<FetchCartResponse>> => {
  apiRequester.get < Response < FetchCartResponse >> 'cart';
};

// 카트에 데이터를 추가하거나 업데이트하기 위한 API 요청
// AxiosPromise를 반환하며, 해당 Promise의 제네릭 타입은 Response<PostCartResponse>로 정의
// PostCartResponse는 서버에서 받아온 카트에 대한 업데이트 결과에 대한 타입
const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<Response<PostCartResponse>> => {
  apiRequester.post<Response<PostCartResponse>>('cart', postCartRequest);
};
```
위 값에서 apiRequester에서 타입을 처리할 수도 있지만 그럴경우 응답값이 없는 요청에 대해서 처리하기 까다로워지므로 requester은 응답값이 어떤지 알 수 없게 설계한다.
<br>
만약 프론트에서 사용하진 않지만 다른 API서버로 넘겨줘야하는 값이 있거나 어떤 값이 내려올지 알 수 없는 것들은 unknown을 사용한다. 그리고 만약 프론트에서 써야할 값이 생기면 새로 타입을 지정해준다.
```tsx
interface response {
data:{
cart:Cart[],
forpass:unknown
}
}
```

### 뷰모델 사용하기
API응답은 변할 가능성이 크고 초기 프로젝트일수록 그럴 가능성이 높아진다. 그러므로 변화에 유리한 방법을 설계해야한다.
<br>
아래 코드에는 어떤 문제들이 있을까 ?
```tsx
import React, { useEffect, useState } from 'react';

const List = () => {
  const [totalItemLength,setTotalItemLength] = useState(0);
  const [items,setItems] = useState<ListItem[]>()

  useEffect(()=>{
    fetchList().then(({data})=>{
      setItems(data)
      setTotalItemLength(data.length
        
        )
    })
  },[])

  return (
    <div>
      {items.map}
      <p>{totalItemLength}</p>
    </div>
  );
};

export default List;
```

```tsx
interface JobListItemResponse {
  name: string;
}

interface JobListResponse {
  jobItems: JobListItemResponse[];
}

class JobList {
  readonly totalItemCount: number;
  readonly items: JobListItemResponse[];

  constructor({ jobItems }: JobListResponse) {
    this.totalItemCount = jobiItems.length;
    this.items = jouItems;
  }
}

const fetchJobList = async (
  filter?: ListFetchFilter
): Promise<JobListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get('/apis/get-list-summaries')
    .call<Response<JobListResponse>>();

  return new JobList(data);
};
```

