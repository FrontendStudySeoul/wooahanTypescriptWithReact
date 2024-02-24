# 12. 타입스크립트 프로젝트 관리

## 앰비언트 타입 활용하기
### 앰비언트 타입
`.d.ts` 확장자를 가진 파일에서만 타입 선언이 가능하고 "값"을 표현할 수는 없다.

declare 키워드는 "이미 존재하지만 ts가 알지 못하는 부분을 컴파일러에 "이런게 있다"라고 알려주는 역할.

- 값 선언이 불가능
- `declare`라는 키워드 사용하여 어딘가에 JS 값이 존재한다는 사실을 선언 가능하다.
- 단순히 여부만 알려주기 때문에 `컴파일러 대상이 아니다`.
- `.ts, .js`를 제외한 다른 파일은 인식하지 못함.
- `tsconfig.json`의 `{ declaration: true }`로 설정하면 컴파일러가 `.d.ts`파일을 자동으로 생성해줌.
- 전역타입 설정시 주의: 충돌 주의, 잘못 선언시 혼란야기


### 사용 예시
- 특정 형식의 모듈을 선언하여 컴파일러에 미리 정보를 제공
  ```ts
    declare module "*.png" {
      const src: string;
      export default src;
    }
  ```
- 이미 존재하는 전역 변수에 타입을 추가 해줄 때
  - 웹뷰 개발이나 외부 라이브러리 사용시 타입을 overwrite하거나 재선언 해주기도함
  ```ts
    // 웹뷰에서 디바이스 정보나 앱버전 타입 정보를 Window 객체 타입에 주입
    declare global {
      interface Window {
        deviceId: string | undefined;
        appVersion: string;
      }
    }
  ```
  ```ts
  // next-auth 라이브러리의 next-auth/jwt 타입에 기존 JWT 타입을 확장하여 커스텀하여 재선언함.
  declare module 'next-auth/jwt' {
    /** Returned by the `jwt` callback and `getToken`, when using JWT sessions */
    interface JWT extends DefaultJWT {
      refreshToken?: string;
      accessToken?: string;
      provider?: string;
      error?: string;
      expiresIn?: number;
      isLoginSuccess?: booelan;
      loginLocation?: 'snsJoin' | 'emailJoin';
    }
  }
  ```
  ```ts
  // cypress 라이브러리에서 Chainable 타입을 Overwrite해서 커스텀하게 사용가능하게 함
  declare namespace Cypress {
    interface Chainable {
      /**
      * 테스트 id를 가져옵니다.
      *
      * @example cy._('test-id')
      */
      _(id: string): Chainable<JQuery<HTMLElement>>;

      /**
      * 테스트 id를 가져옵니다.(storybook 테스트 ID 방식을 따름)
      *
      * @example cy.__('data-testid')
      */
      __(id: string): Chainable<JQuery<HTMLElement>>;
      /**
      * axios fetch 상태값을 체크 합니다.
      * @param response aixos response 결과 값
      */
      axiosStatus(response: any): boolean;
      /**
      * Custom command to visit google page
      * @example cy.google()
      */
      google(): Chainable<Window>;
      /**
      * Custom command to get element by data-cy value
      * @example cy.getByDataCy('selector')
      */
      getByDataCy(selector: string): Chainable<Window>;
    }
  }
  ```

### declare type(with. custom utility)
커스텀 유틸리티 타입을 declare type을 선언해서 전역에서 사용 가능
```ts
  declare type Nullable<T> = T | null;
  const name: Nullable<string> = "woowa";
```

### declare module(with. css-in-js)
```ts
  declare module 'styled-components' {
    export interface DefaultTheme extends BasicProperties {
    }
  }

  export interface BasicProperties {
    primitiveColor: typeof PrimitiveColor;
    coloTokenX: typeof ColoTokenX;
    fontToken?: typeof FontTokens;
    variable?: ColorCategoryMain;
    size?: PDSGlobalSizeType;
  }
```

### declare namespace
Node.js 환경일떄 활용( .env는 타입 추론이 안되기 때문에 유용하며, env 값 사용시 초기값 설정을 하지 않아도되어 편리함)
```ts
  declare namespace NodeJS {
    interface ProcessEnv {
      readonly NEXT_PUBLIC_LOCAL_SSL_URL_PREFIX: string;  
      readonly NEXT_PUBLIC_API_BASE_URL: string;
      readonly NEXT_PUBLIC_BASE_URL: string;
    }
  }
```

### declare global
```ts
  declare global {
    interface Window {
      newProperty: string;
    }
  }
```

## 모노레포
- [모던 프론트엔드 프로젝트 구성 기법 - 모노레포 개념 편](https://d2.naver.com/helloworld/0923884)
- [모던 프론트엔드 프로젝트 구성 기법 - 모노레포 도구 편](https://d2.naver.com/helloworld/7553804)
