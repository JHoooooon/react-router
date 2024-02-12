# Router Component

`Router` 컴포넌트의 모음이다

## `<BrowserRouter>`

> [BrowserRouter](https://reactrouter.com/en/main/router-components/browser-router) 의 내용이다

`<BrowserRouter>` 는 브라우저 주소바에 현재 `location` 을 `URL` 로 저장하며, 페이지 이동은 브라우저의 내장된 `history` 스택을 사용한다

> `Docs` 에서 `clean URL` 이라고 하는데, 이는 `relative Links` 에 할당한
> 주소값이 아닌(`path="/"` 같이), 이를 기반으로 사용가능한 `URL` 로 변형하여
> 저장되는것을 말하는 듯하다

다음은 `BrowserRouter` 의 타입정의이다

`Type declaration`

```tsx
declare function BrowserRouter(props: BrowserRouterProps): React.ReactElement;

interface BrowserRouterProps {
  basename?: string;
  children?: React.ReactNode;
  future?: FutureConfig;
  window?: Window;
}
```

위의 타입정의를 보면 `BrowerRouterProps` 를 `props` 옵션으로 사용한다

- `basename`

`URL` 에 `basename` 을 지정하도록 설정한다

```tsx
function App() {
  return (
    <BrowserRouter basename="/app">
      <Routes>
        <Route path="/" /> {/* 👈 Renders at /app/ */}
      </Routes>
    </BrowserRouter>
  );
}
```

- `future`

선택적 옵션인 `Future Flag` 을 활성화한다
`React Router` 측에서는 더 쉽게 `v7` 으로 마이그레이션 할수 있도록 한다
새롭게 릴리즈되는 기능을 더빠르게 선택하는 경우 추천한다고 말한다

```tsx
function App() {
  return (
    <BrowserRouter future={{ v7_startTransition: true }}>
      <Routes>{/*...*/}</Routes>
    </BrowserRouter>
  );
}
```

- `window`

`BrowserRouter` 는 기본적으로 현재 [`document.defaultView`](https://developer.mozilla.org/en-US/docs/Web/API/Document/defaultView) 를 사용한다
그러나, 다른 `window` 의 `URL` 에대한 변경을 추적하는데 사용하는것 또한 가능하다

> 예를 들어 `<iframe>` 같은.

## `<HashRouter>`

> [HashRouter](https://reactrouter.com/en/main/router-components/hash-router) 에 대한 내용을 정리한다

`<HashRouter>` 는 어떠한 이유로 인해 `URL` 이 `server` 에 전송되서는 안되거나, 전송될수 없는 경우 브라우저 에서 사용하기 위한것이다.

`server` 를 완전히 제어할수 없는 일부 공유 호스팅 시나리오에서 발생할 수 있다

이 상황에서, `<HashRouter>` 는 현재 `URL` 의 `hash` 에 현재 위치를 저장하는
것을 가능하게 만든다 그래서 이는 절대 `server` 로 보내지지 않는다

```tsx
import * as React from "react";
import * as ReactDOM from "react-dom";
import { HashRouter } from "react-router-dom";

ReactDOM.render(
  <HashRouter>{/* The rest of your app goes here */}</HashRouter>,
  root
);
```

> ⚠️ **중요**
> 꼭 필요한 경우가 아니라면 `HashRouter` 를 사용하지 않는것을 추천한다

`Props` 는 `BrowserRouter` 와 동일하다

## `<MemoryRouter>`

`MemoryRouter` 는 해당위치를 배열의 내부적 위치에 저장한다
이는 `BrowserRouter` 그리고 `HashRouter` 와 달리 브라우저의 `history stack` 과
같은 외부 소스에 연결되지 않는다

`MemoryRouter` 는 `testing` 같이 `history stack` 을 완벽하게 제어할 필요가 있는
시나리오에 이상적이다.

- `<MemoryRouter initialEntries>` 는 `default` 가 `["/"]` 이다

> `"/" (Root URL)` 단 하나의 항목으로 기본 설정되어 있다

- `<MemoryRouter initialIndex>` 는 `initialEntries` 의 마지막 `index` 가
  기본값이다.

> ❗**Tip**
>
> 대부분의 `React Router` 테스트는 `<MemoryRouter>` 를 경로정보를
> 가져와 쓰기위해 사용한다
>
> 그래서, 테스트를 보는것만으로도 이를 사용하는 몇가지 훌륭한
> 예를 볼 수 있다
>
> > `source of truth` 뜻이 `정보의 출처`를 말하는데,
> > 여기서는 `경로정보` 로 해석할수 있을듯 하다.

```tsx
import * as React from "react";
import { create } from "react-test-renderer";
import { MemoryRouter, Routes, Route } from "react-router-dom";

describe("My app", () => {
  it("renders correctly", () => {
    let renderer = create(
      <MemoryRouter initialEntries={["/users/mjackson"]}>
        <Routes>
          <Route path="users" element={<Users />}>
            <Route path=":id" element={<UserProfile />} />
          </Route>
        </Routes>
      </MemoryRouter>
    );

    expect(renderer.toJSON()).toMatchSnapshot();
  });
});
```

선언된 타입은 다음과 같다

```ts
declare function MemoryRouter(props: MemoryRouterProps): React.ReactElement;

interface MemoryRouterProps {
  basename?: string;
  children?: React.ReactNode;
  initialEntries?: InitialEntry[];
  initialIndex?: number;
  future?: FutureConfig;
}
```

- **initialEntiries**
  시작시 로드할 `URL` 경로 목록을 설정
  다음은 `"/"`, `"/about"` 경로를 로드한다

```tsx
<MemoryRouter initialEntries={["/", "/about"]}>...</MemoryRouter>
```

- **initialIndex**
  `initialIndex` 목록에서 처음 로드할 `URL` 경로의 인덱스 설정
  다음은 `/about` 경로를 로드한다

```tsx
<MemoryRouter initialEntries={["/", "/about", "/contact"]} initialIndex={1}>
  ...
</MemoryRouter>
```

`initialIndex` 에 `index` 를 지정하지 않을시, 기본값으로는 주어진
`initialEntries` 배열의 마지막 `index` 이다.

> 나머지는 `BrowserRouter` 의 `props` 와 같다

## `<NativeRouter>`

선언된 타입은 다음과 같다

```ts
declare function NativeRouter(props: NativeRouterProps): React.ReactElement;

interface NativeRouterProps extends MemoryRouterProps {}
```

`<NativeRouter>` 는 `React Native` 앱에서 `React Router` 를
실행하기 위해 추천하는 인터페이스 이다.

> `props` 는 `MemoryRouterProps` 이므로, `MemoryRouter` 와 같다

## `<Router>`

타입은 다음과 같다

```ts
declare function Router(props: RouterProps): React.ReactElement | null;

interface RouterProps {
  basename?: string;
  children?: React.ReactNode;
  location: Partial<Location> | string;
  navigationType?: NavigationType;
  navigator: Navigator;
  static?: boolean;
}
```

`<Router>` 는 저수준의 인터페이스이다.
이는 모든 `router` 들 (`<BrowserRouter>`, `<StaticRouter>` 같은) 에서
공유하는 컴포넌트이다.

`React` 측면에서 보면, `<Router>` 는 `Context Provider` 이며, 앱의 나머지
`routing` 정보를 제공한다

아마도 `<Router>` 를 수동으로 렌더링할 필요는 없을 것이다.
대신에, 환경에 따라 고수준의 `routes` 중 하나를 사용해야 한다
특정 앱에서는 라우터가 하나만 필요하기도 한다

`<Router basename>` `prop` 은 모든 `routes` 에서 사용할것이고,
모든 `links` 를 공유하는 `URL` 의 `pathname` 의 `"base"` 부분을 설정한다

> 앱 내 링크는 이 기본 경로(`base pathname`)을 기준으로 상대적으로
> 해석해서 만든다

여러 진입점을 가진 앱 일때 또는 `React Router` 와 함께 대규모 앱의 일부만
렌더링 할때 유용하다

> `Basename` 은 대소문자 구분을 하지 않는다.

## `<StaticRouter>`

타입은 다음과 같다

```ts
declare function StaticRouter(props: StaticRouterProps): React.ReactElement;

interface StaticRouterProps {
  basename?: string;
  children?: React.ReactNode;
  location?: Path | LocationPieces;
}
```

`<StaticRouter>` 는 `node.js` 에 `React Router` 웹 앱을 렌더링하는데
사용된다

`location` `prop` 을 통해 현재 `location` 을 제공한다

- `<StaticRouter location>` 은 `default` 로 `/` 이다

```ts
import * as React from "react";
import * as ReactDOMServer from "react-dom/server";
import { StaticRouter } from "react-router-dom/server";
import http from "http";

function requestHandler(req, res) {
  let html = ReactDOMServer.renderToString(
    <StaticRouter location={req.url}>
      {/* The rest of your app goes here */}
    </StaticRouter>
  );

  res.write(html);
  res.end();
}

http.createServer(requestHandler).listen(3000);
```
