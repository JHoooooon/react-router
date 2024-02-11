# 🔥 React Router 정리

> `v6 Docs` 를 보면서 내용을 정리한다
> `NextJS` 를 사용하면 자체적으로 제공하는 `Router` 를 사용하여 처리해도 된다
> 하지만 공부 목적으로는 모든것들을 제공해주는 `NextJS Framework` 는 적합하지 않다
>
> 일반 리액트를 사용하며 사용하기에, 가장 범용적으로 쓰이는 `client side routing` 은
> 역시나 `React Router` 가 있기에 충분히 공부할 가치가 있다
>
> 그렇기에 해당 부분을 정리해볼까 한다
>
> > 💢 물론, 사용하면서 약간의 이상한 동작들을 하기에 뭐가 문제인지 내용을 알필요가 느껴졌다...

## :door: Client Side Routing

React Router 는 `client side routing` 이다.

전통적인 웹사이트는 웹 서버로 부터 `document` 를 요청하고,
`CSS`, `Javascript` 를 다운로드 및 평가하고, 서버로 부터 받은 `HTML` 을
렌더링 한다

`user` 가 어떠한 링크를 클릭하면, 새로운 페이지는 앞에 진행된 모든것을
다시 시작하고 적용한다

클라이언트쪽 라우팅은 링크 클릭을 하면 서버로부터 다른 `document` 에 대한 요청을
만드는것 없이, `URL` 만 업데이트 하게 해준다.

이러한 동작을 기반으로 서버로 부터 `document` 요청하는 대신에,
`page` 에 대한 새로운 정보를 `fetch` 요청으로 `data` 로 만들고,
즉각적으로 새로운 `UI` 를 렌더링할수 있다.

이러한 동작방식은 `다음 page` 에 대한 `assets` 에 대한 재평가 또는
완전히 새로운 `document` 를 요청할 필요가 없기 때문에, 빠른 사용자 경험을
제공한다

> 이것은 또한 `animation` 같은 좀더 다이네믹한 사용자 경험을 제공할수도 있다

클라이언트 사이드 라우팅은 `Router` 및 페이지에 `linking/submitting` 위한
`Link` 그리고 `<Form>` 생성으로 활성화 시킨다

```ts
import * as React from "react";
import { createRoot } from "react-dom/client";
import {
  createBrowserRouter,
  RouterProvider,
  Route,
  Link,
} from "react-router-dom";

const router = createBrowserRouter([
  {
    path: "/",
    element: (
      <div>
        <h1>Hello World</h1>
        <Link to="about">About Us</Link>
      </div>
    ),
  },
  {
    path: "about",
    element: <div>About</div>,
  },
]);

createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```

## 🚗 Nested Routes

`Nested Routes` 는 컴포넌트 구조와 데이터로 `URL` 의 세그먼트를 결합하는
일반적인 아이디어이다

> `React Router` 는 `Ember.js` 라우팅 시스템에서 영감을 받았다고 한다

`URL` 의 세그먼트는 거의 모든경우에 다음을 결정한다는것을 깨달았다고 한다

- 페이지에 렌더링할 레이아웃
  <br/>

- 해당 레이아웃 데이터 종속성

`React Router` 는 `URL` 세그먼트, `data` 가 결합된 중첩된 레이아웃을 생성위해
다음의 `API` 로 규칙을 정한다

```ts
// Configure nested routes with JSX
createBrowserRouter(
  createRoutesFromElements(
    <Route path="/" element={<Root />}>
      <Route path="contact" element={<Contact />} />
      <Route
        path="dashboard"
        element={<Dashboard />}
        loader={({ request }) =>
          fetch("/api/dashboard.json", {
            signal: request.signal,
          })
        }
      />
      <Route element={<AuthLayout />}>
        <Route path="login" element={<Login />} loader={redirectIfUser} />
        <Route path="logout" action={logoutUser} />
      </Route>
    </Route>
  )
);

// Or use plain objects
createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        path: "contact",
        element: <Contact />,
      },
      {
        path: "dashboard",
        element: <Dashboard />,
        loader: ({ request }) =>
          fetch("/api/dashboard.json", {
            signal: request.signal,
          }),
      },
      {
        element: <AuthLayout />,
        children: [
          {
            path: "login",
            element: <Login />,
            loader: redirectIfUser,
          },
          {
            path: "logout",
            action: logoutUser,
          },
        ],
      },
    ],
  },
]);
```

## 🧱 Dynamic Segments

`URL` 의 세그먼트는 동적인 `placeholder` 일수 있다. 이 동적인 `placeholder` 는
파싱되고, 다양한 `apis` 를 제공한다

```ts
<Route path="projects/:projectId/tasks/:taskId" />
```

위의 `:` 와 연결된 두개의 `segments` 는 동적이며, 다음의 `APIs` 를 제공한다

```ts
// If the current location is /projects/abc/tasks/3
// 만약 현재 location 이 /projects/abc/tasks/3 이라고 한다면
<Route
  // sent to loaders
  // loader 를 보낸다
  //
  // loader 는 주어진 element 가 렌더링 되기 전에
  // 데이터를 제공하는 함수이다
  loader={({ params }) => {
    // params 는 projectId 와 taskId 가 포함된 객체이다
    params.projectId; // abc
    params.taskId; // 3
  }}
  // and actions
  // 그리고 앤션은 다음과 같다
  //
  // `actions` 는 클라이언트측에서 폼 제출을 처리하는
  // 역할을 하는 함수이다
  // 리액트 라우터의 `action` 함수는 클라이언트 측에서 `URL` 경로
  // 변경과 데이터 처리를 동시에 수행하여 페이지 새로고침 없이 부드러운
  // 사용자 경험을 제공한다
  //
  action={({ params }) => {
    // params 는 projectId 와 taskId 가 포함된 객체이다
    params.projectId; // abc
    params.taskId; // 3
  }}
  // 렌더링한 Task 컴포넌트
  element={<Task />}
/>;

function Task() {
  // returned from `useParams`
  // useParams 를 사용하여 리턴한다
  const params = useParams();
  params.projectId; // abc
  params.taskId; // 3
}

function Random() {
  // useMatch 는 현재 location 을 기준으로 `path` 에
  // 매치된 data 를 반환한다
  const match = useMatch("/projects/:projectId/tasks/:taskId");
  match.params.projectId; // abc
  match.params.taskId; // 3
}
```

### 🥇 Ranked Route Matching

라우트에서 `URLs` 가 매칭될때, `React Router` 에서 `segments` 의 수,
`static segments`, `dynamic secments`, `slats`, 등등 에 따라
`routes` 순위를 매길것이다

다음의 두 라우트를 보도록 하자

```ts
<Route path="/teams/:teamId" />
<Route path="/teams/new" />
```

그리고, 다음의 `URL` 이 있다고하자

```sh
http://example.com/teams/new
```

이를 통해, `URL` 에 `Matching` 된 두개의 `routes` 에서 두번째 경로(`/teams/new`) 가
선택되기 원하는 것을 직관적으로 알 수 있다
