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
`React Router` 측에서는 더 쉽게 `v7` 으로 마이그레이션 할수 있도록,
새롭게 릴리즈되는 기능을 더빠르게 선택하는것을 추천한다고 말한다

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

- **`basename`**

[BrowserRouter](/#BrowserRouter) 와 동일하다.

- **`future`**

[BrowserRouter](/#BrowserRouter) 와 동일하다.
