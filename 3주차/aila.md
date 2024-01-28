# 7. 비동기 호출

## 7.3 API 에러 핸들링

- 비동기 API 에러를 구체적이고 명시적으로 핸들링하는 방법

### 1. 타입가드 활용

- `isAxiosError`
  - axios에서 제공해주는 타입 가드

```ts
interface ErrorResponse {
  status: string;
  serverDateTime: string;
  errorCode: string;
  errorMessage: string;
}
```

- ErrorResponse 인터페이스를 사용해 타입 가드를 명시적으로 작성할 수 있다.
- 주어진 에러가 Axios에서 발생한 서버 에러인지를 판별하여 그 결과를 타입 가드로 반환하는 함수

```ts
function isServerError(error: unknown): error is AxiosError<ErrorResponse> {
  return axios.isError(error);
}
```

```ts
const onClickDeleteHistoryButton = async (id: string) => {
  try {
    await axios.post('https://....', { id });

    alert('주문 내역이 삭제되었습니다.');
  } catch (e: unknown) {
    if (isServerError(e) && e.response && e.response.data.errorMessage) {
      // 서버 에러일 때의 처리임을 명시적으로 알 수 있다 setErrorMessage(e.response.data.errorMessage);
      return;
    }
    setErrorMessage('일시적인 에러가 발생했습니다. 잠시 후 다시 시도해주세요');
  }
};
```

### 2. 에러 서브클래싱 활용

- 단순 서버 에러 뿐만 아니라 인증, 네트워크, 타임아웃 등 다양한 에러가 발생할 수 있다. 이를 더욱 명시적으로 표시하기 위해 `서브클래싱`을 활용할 수 있다.

> 서브클래싱: 기존 클래스를 확장하여 새로운 하위 클래스를 만드는 과정.  
> 새로운 클래스는 상위 클래스의 모든 속성과 메서드를 상속받아 사용할 수 있고 추가적인 속성과 메서드를 정의할 수 있다.

```ts
const getOrderHistory = async (page: number) => Promise<History> {
  try {
    const data = await axios.get("https://...");
    const history = await JSON.parse.(data);
    } catch (e) {
      // 에러가 발생했을 때 alert을 띄워서 사용자에게 표시
      alert(e.message)
    }
  }
```

- alert을 통해 사용자는 어떤 에러가 발생한 것인지 판단할 수 있지만, 개발자 입장에서는 어떤 에러가 발생한 것인지 정확히 구분할 수 없다.
- 이때 `서브클래싱`을 활용하면 에러가 발생했을 때 어떤 에러인지 바로 확인할 수 있고, **에러 인스턴스에 따라 에러 처리 방식을 다르게 구현할 수 있다.**

```ts
// Error 클래스를 확장한 사용자 정의 에러 클래스 OrderHttpError
class OrderHttpError extends Error {


  // privateResponse라는 읽기 전용(private readonly) 속성
  private readonly privateResponse: AxiosResponse<ErrorResponse : undefined>


  // 생성자를 정의
  // 첫 번째 매개변수는 에러 메시지, 두 번째 매개변수는 AxiosResponse<ErrorResponse> 타입의 응답 객체
  constructor(message?: string, response?:AxiosResponse<ErrorResponse>){

    // 부모 클래스인 Error의 생성자를 호출해 에러 메시지 전달
    super(message);
    // 에러 객체의 이름 설정
    this.name = "OrderHttpError"
    // 응답 객체 할당
    this.privateResponse = response
  }

  // getter 정의
  // 외부에서 privateResponse에 접근 가능
  get response(): AxiosResponse<ErrorResponse> | undefined {
    return this.privateResponse
  }
}


class NetworkError extends Error{
    constructor(message: ""){
    super(message);
    this.name = "NetworkError"
  }
}

class UnauthorizedError extends Error{
    constructor(message?: string, response?:AxiosResponse<ErrorResponse>){
    super(message, response);
    this.name = "Unauthorized"
  }
}
```

```ts
// 주문 내역 가져오기
const fetchOrderHistory = async () => {
  try {
    const response = await apiRequester.get('/order-history');
  } catch (error: unknown) {
    if (error instanceof AxiosError) {
      if (error.response) {
        const { status, data } = error.response;
        const { errorCode, errorMessage } = data;

        if (status === 401) {
          // UnauthorizedError
          throw new UnauthorizedError(errorMessage, error.response);
        } else {
          // OrderHttpError
          throw new OrderHttpError(errorMessage, error.response);
        }
      } else if (error.request) {
        // 요청 전송 후 응답이 없는 경우
        throw new NetworkError('No response received');
      } else {
        // 요청 전송 전에 에러가 발생한 경우
        throw new NetworkError('Request error');
      }
    } else {
      // Axios 에러가 아닌 경우
      throw error;
    }
  }
};
```

### 3. 인터셉터 활용

```ts
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error,
): Promise<Error> => {
  (error) => {
    // 401 에러인 경우 로그인 페이지로 이동
    if (error.response && error.response.status === '401') {
      window.location.href = `/login`;
    }
    return Promise.reject(error);
  };
};

orderApiRequester.interceptors.response.use(
  // 응답 성공시
  (response: AxiosResponse) => response,
  // 응답 실패시 httpErrorHandler() 호출
  httpErrorHandler,
);
```

```ts
requester.interceptors.response.use(
  (response) => response,
  (error) => {
    const originalRequest = error.config;
    const refreshToken = Cookie.get('refreshToken') as string;

    if (
      error.response &&
      error.response.status === UNAUTHORIZED &&
      !DISABLE_UNAUTHORIZED_CHECK_PATH.includes(originalRequest.url) &&
      !originalRequest._retry &&
      !isRefreshing
    ) {
      isRefreshing = true;

      // 토큰을 갱신
      return (
        AuthApi.refresh({ refreshToken })
          .then((response) => {
            originalRequest._retry = true;

            // 갱신된 토큰 쿠키에 저장
            Cookie.set('refreshToken', response.data.result?.refreshToken, {
              expires: dayjs().add(4, 'hour').toDate(),
            });

            return axios(originalRequest);
          })

          // 실패한 경우 쿠키 삭제하고 세션 만료 페이지로 리다이렉트
          .catch((error) => {
            Cookie.remove('refreshToken');
            window.location.assign('/session_expired');

            return Promise.reject(error);
          })
          .finally(() => {
            isRefreshing = false;
          })
      );
    } else if (
      // 중복 로그인 처리
      error.response &&
      error.response.status === CONFLICT &&
      error.response.data.status.code === DUPLICATED_LOGIN_CODE
    ) {
      window.location.assign('/duplicated');
    }

    return Promise.reject(error);
  },
);
```

### 4. 에러 바운더리 활용

- `ErrorBoundary`
  - 리액트 컴포넌트트리에서 에러가 발생할 때 공통으로 에러를 처리하는 리액트의 컴포넌트
  - 해당 컴포넌트의 하위에 있는 컴포넌트에서 발생한 에러를 캐치하고 해당 에러를 가장 가까운 부모 에러 바운더리에서 처리

```tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

// ErrorBoundaryProps와 State를 interface로 정의
interface ErrorBoundaryProps {
  children: ReactNode;
}

interface ErrorBoundaryState {
  hasError: boolean;
}

// Component 클래스를 상속하는 ErrorBoundary 클래스 선언
// ErrorBoundaryProps와 ErrorBoundaryState를 제네릭으로 받음.
class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  // 생성자. hasError의 default값을 false로 설정.
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false };
  }

  //자식 컴포넌트에서 오류가 발생했을 때 호출
  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    this.setState({ hasError: true });
    console.error('에러가 발생했습니다:', error, errorInfo);
  }

  // hasError의 상태에 따라 컴포넌트 렌더링
  render() {
    const { children } = this.props;
    const { hasError } = this.state;

    if (hasError) {
      // 에러가 발생했을 때 에러 페이지를 보여줄 수 있는 컴포넌트 리턴
      return <ErrorPage />;
    }

    return children;
  }
}

function App() {
  return (
    <ErrorBoundary>
      <OrderHistoryPage />
    </ErrorBoundary>
  );
}

export default App;
```

```tsx
interface Props {
  fallback?: React.ReactNode;
  children: React.ReactNode;
}

interface State {
  hasError: boolean;
  error: any;
}

class ErrorBoundary extends React.Component<Props, State> {
  state = {
    hasError: false,
    error: null,
  };

  // 에러를 콘솔에 기록하고 알림을 띄우는 메소드 정의
  alertError(error: Error) {
    console.log(error);
    window.alert('오류가 발생했습니다.');
  }

  // 에러가 발생하면 상태를 업데이트하는 라이프사이클 메소드
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  // 에러가 발생했을 때 호출되는 메소드
  componentDidCatch(error: Error) {
    this.alertError(error);
  }

  // children 속성이 변경되면 에러 상태를 초기화
  componentDidUpdate(previousProps: Props) {
    if (previousProps.children === this.props.children) {
      return;
    }

    this.setState({ hasError: false, error: null });
  }

  render() {
    const { hasError, error } = this.state;
    const { children } = this.props;
    return hasError ? <Fallback error={error} /> : children;
  }
}

export default ErrorBoundary;
```

### 5. react-query 활용

- react-query나 swr과 같은 데이터 페칭 라이브러리를 사용하면 요청에 대한 상태를 반환해주기 때문에 요청 상태를 확인하기 쉽다.

```tsx
const JobComponent: React.FC = () => {
  // 요청에 대한 상태를 반환
  const { isError, error, isLoading, data } = useFetchJobList();

  // 에러 발생
  if (isError) {
    return <div>{`${error.message}`}가 발생했습니다. 다시 시도해주세요</div>;
  }

  // 로딩
  if (isLoading) {
    return <div>로딩 중입니다</div>;
  }

  // 정상적으로 실행시 화면에 데이터 출력
  return (
    <>
      {data.map((job) => (
        <JobItem key={job.id} job={job} />
      ))}
    </>
  );
};
```

```tsx
interface Props {
  error: AxiosError<any>;
  errorMessageTitle: string;
  errorMessage?: string;
}

interface CommonErrorHandlerProps {
  errorMessageTitle: string;
  errorMessage?: string;
}

// 에러 처리 커스텀훅
export const useHandleApiError = () => {
  const snackbar = useSnackbar();

  const handleApiError = useCallback(
    ({
      error,
      errorMessageTitle,
      errorMessage = API_500_ERROR_MASSAGE,
    }: Props) => {
      const statusCode = error?.response?.status;

      snackbar.error({
        title: errorMessageTitle,
        content:
          statusCode === 500
            ? errorMessage
            : error?.response?.data?.status.message,
      });
    },
    [snackbar],
  );

  return {
    handleApiError,
  };
};

const useGetCustomers = (
  params?: GetCustomersRequestQuery,
  isSearch?: boolean,
) => {
  const { handleApiError } = useHandleApiError();
  const queryKey = ['customers', params];

  return useQuery(queryKey, () => CustomerApi.get(params), {
    enabled: isSearch,
    onError: (error: AxiosError<any>) =>
      handleApiError({
        error,
        errorMessageTitle: '회원 통합조회 실패',
      }),
  });
};
```

### 6. 그 외

- 보통 200번대 에러는 API의 성공 응답을 나타낸다. 하지만 비즈니스 로직에서 유효성 검증에 의해 추가된 커스텀 에러는 200번대 응답과 함께 응답 바디에 별도의 상태 코드를 전달하기도 한다.

```tsx
// 보통은 API 성공 응답, 여기서는 커스텀 에러로 사용
httpStatus:200
{
  "status":"C2005",// 응답 바디에 별도의 상태 코드와 메세지 전달
  "message":"장바구니에 품절된 메뉴가 있습니다"
}
```

이러한 에러를 처리하기 위해서 요청 함수 내에서 조건문으로 status를 비교할 수 있다.

```tsx
const successHandler = (response: CreateOrderResponse) => {
  if (response.status === 'SUCCESS') {
    // 성공 시 진행할 로직
    return;
  }
  throw new CustomError(response.status, response.message);
};

const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post('https://...', data);

    successHandler(response);
  } catch (error) {
    errorHandler(error);
  }
};
```

하지만 이렇게 처리해야하는 API가 많을 경우에는 매번 조건문을 추가하여 에러를 처리해야한다.

이때 axios등의 라이브러리를 사용하면 특정 호스트에 대한 requester를 별도로 선언하고 상태 코드 비교 로직을 인터셉터에 추가할 수 있다.

```ts
export const apiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
});

export const httpSuccessHandler = (response: AxiosResponse) => {
  if (response.data.status !== 'SUCCESS') {
    throw new CustomError(response?.data.message, response);
  }

  return response;
};

// 응답 인터셉터
// 응답이 성공(httpSuccessHandler) 또는 실패(httpErrorHandler)하는 경우에 대한 처리 설정하고 있습니다.
apiRequester.interceptors.response.use(httpSuccessHandler, httpErrorHandler);

const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post('https://...', data);

    successHandler(response);
  } catch (error) {
    // status가 SUCCESS가 아닌 경우 에러로 전달된다
    errorHandler(error);
  }
};
```

인터셉터에서 커스텀 에러를 판단하고 에러를 던지기 때문에 외부에서 200번대로 온 응답이라고 해도 400,500번대와 같은 에러로 받게 된다.

## 7.4 API 모킹

- 가짜 모듈을 활용하는 것
- 서버에 문제가 생겼을 때, api가 늦어질 때 등 다양한 상황에서 유연하게 대처할 수 있다.

### 1. JSON 파일 활용

- 별도의 환경설정 없이 쉽게 구현

```ts
const SERVICES: Service[] = [
  {
    id: 0,
    name: '우아한',
  },
  {
    id: 1,
    name: '형제들',
  },
];

export default SERVICES;

// api.ts
const getServices = ApiRequester.get('/mock/service.ts');
```

### 2. NextApiHandler 활용

- 핸들러를 정의해 응답하고자 하는 값을 정의하고 핸들러 안에서 요청에 대한 응답을 정의할 수 있다.
- 핸들러 함수 내부에서 추가로직을 작성하여 응답 처리 로직을 추가할 수 있다.

```ts
import { NextApiHandler } from 'next';

const BRANDS: Brand[] = [
  {
    id: 1,
    label: '배민스토어',
  },
  {
    id: 2,
    label: '비마트',
  },
];

const handler: NextApiHandler = (req, res) => {
  // 추가 로직 작성

  // 요청에 대한 응답 정의
  res.json(BRANDS);
};

export default handler;
```

### 3. API 요청 핸들러 활용

- 모든 API요청 함수에 분기처리를 해야한다.

```ts
// 0.5초 후에 response 반환하는 함수
const mockFetchBrands = (): Promise<FetchBrandsResponse> =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        status: 'SUCCESS',
        message: null,
        data: [
          {
            id: 1,
            label: '배민스토어',
          },
          {
            id: 2,
            label: '비마트',
          },
        ],
      });
    }, 500);
  });

const fetchBrands = () => {
  // useMock이 true인 경우 mockFetchBrands를 호출함.
  // false인 경우 requester.get('/brands')를 호출하여 실제 데이터를 가져옴.
  if (useMock) {
    return mockFetchBrands();
  }

  return requester.get('/brands');
};
```

### 4. axios-mock-adapters 라이브러리 활용

- axios 요청을 가로채서 요청에 대한 응답 값을 대신 반환
- `onGet`을 사용하여 HTTP 메서드(GET, POST, PUT, 등) 및 엔드포인트에 대한 요청을 가로채고 `reply`를 통해 해당 요청에 대한 목업 응답을 설정하고 반환한다.

```ts
import axios from "axios";
import MockAdapter from "axios-mock-adapter";
import fetchOrderListSuccessResponse from "fetchOrderListSuccessResponse.json";

interface MockResult {
  status?: number;
  delay?: number;
  use?: boolean;
}


// mockAdapter 생성
// onNoMatch: "passthrough"는 가짜 응답이 없는 경우에는 요청을 실제 서버로 전달하여 실제 응답을 받아옴.
const mock = new MockAdapter(axios, { onNoMatch: "passthrough" });

export const fetchOrderListMock = () =>

// /order/list로 시작하는 GET 요청에 대해 200 status와 fetchOrderListSuccessResponse를 응답으로 설정한 response를 생성
  mock.onGet(/\/order\/list/).reply(200, fetchOrderListSuccessResponse);

// fetchOrderListSuccessResponse.json
{
    "data": [
        {
            "orderNo": "ORDER1234", "orderDate": "2022-02-02", "shop": {
            "shopNo": "SHOP1234",
            "name": "가게이름1234" },
            "deliveryStatus": "DELIVERY"
        },
    ]
}

```

### 5. 목업 사용여부 제어

- 로컬에서 목업을 사용하고 dev나 운영환경에서는 사용하지 않으려면 플래그를 사용하여 목업을 사용하는 상황을 구분할 수 있다.

```tsx
const useMock = process.env.REACT_APP_MOCK === 'true';

const mockFn = ({ status = 200, time = 100, use = true }: MockResult) =>
  use &&
  mock.onGet('/order/list').reply(() =>
    new Promise((resolve) =>
      setTimeout(() => {
        resolve([
          status,
          status === 200 ? fetchBrandSuccessResponse : undefined,
        ]);
      }, time)
    );
  );

  if (useMock){
    mockFn({status:200,time:100,use:true})
  }
```

또한 플래그에 따라 mockFn을 제어할 수 있는데 매개 변수를 넘겨 특정 mock 함수만 동작하게 하거나 동작하지 않게 할 수 있다.

만약 스크립트 실행 시 구분 짓고자 한다면 package.json에서 관련 스크립트를 추가할 수 있다.

```json
{
  ...,
  "scripts":{
    ...,
    "start:mock":"REACT_APP_MOCK=true npm run start",
    "start":"REACT_APP_MOCK=false npm run start",
  }}
```

이렇게 자바스크립트 코드의 실행 여부를 제어하지 않고 config 파일을 별도로 구성하거나 프록시를 사용할 수 있다.

## ETC. 우리회사에서 api 요청시에 typescript를 사용하는 방법

1. openapi-generator

- 백엔드에서 받는 response에 대한 타입을 하나하나 정의하지 않고 백엔드 스웨거의 타입을 가져와서 프론트에서 사용

```json
"scripts": {
    ...
    "pull-swagger": "npx openapi-typescript@4.3.0 [스웨거 docs url] -o ./src/@types/admin/backend.ts  -c ./.prettierrc",

  },
```

```ts
import { operations } from '@/@types/admin/backend';

export type GetUserResponse =
  operations['getUser']['responses']['200']['content']['*/*'];
```

![](https://velog.velcdn.com/images/nsunny0908/post/5e9fa8b9-49d5-4e64-b92f-fbf0063bb495/image.png)

2. tanstack-query

```ts
const CustomerApi = {
  get: (params?: GetCustomersRequestQuery) => {
    return requester.get<PaginationResponse<GetCustomersResponse>>(BASE_URL, {
      params,
    });
  },
};

const useGetCustomers = (
  params?: GetCustomersRequestQuery,
  isSearch?: boolean,
) => {
  const queryKey = ['customers', params];

  return useQuery(queryKey, () => CustomerApi.get(params), {
    enabled: isSearch,
  });
};
```

3. msw 목업

- HTTP 요청을 가로채서 목업(Mock) 데이터로 응답하도록 해줌.

- 핸들러 작성

```ts
import { rest } from 'msw';

export const customerHandlers = [
  rest.get('*/customer', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        status: {
          message: 'ok',
          code: 2000,
        },
        success: true,
        result: {
          items: [...Array(50)].map((_, i) => ({
            id: `HW${i}`,
            evaluationExecuteDate: '2022-12-10',
            evaluationStartDate: '2022-12-10 ~ 2023-03-09',
            name: '한*봉',
            nid: 'e82de34f-as55-9999-9966-123234234',
            recentTotalAsset: '610,000,000',
            evaluationAsset: '610,000,000',
            riskGrade: '고위험',
            recentDetectedDate: '2022-12-10',
            status: i % 2 === 0 ? 'TEMPORARY' : 'REJECTED',
          })),
        },
      }),
    );
  }),
];
```

- 워커 setup

```ts
import { setupWorker } from 'msw';

import { customerHandlers, adminHandlers } from './handlers';

export const worker = setupWorker(...customerHandlers, ...adminHandlers);
```

- `_app.tsx`에서 환경변수에 따라 워커 설정해줌.

```ts
if (process.env.NEXT_PUBLIC_MOCK_ENABLED === 'true') {
  if (isSSR()) {
    (() => {
      server.listen({ onUnhandledRequest: 'bypass' });
    })();
  } else {
    (async () => {
      const { worker } = await import('../../mocks/browser.js');
      // 처리되지 않은 요청에 대해 무시하도록 설정
      worker.start({ onUnhandledRequest: 'bypass' });
    })();
  }
}
```
