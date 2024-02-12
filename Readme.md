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
`static segments`, `dynamic secments`, `slats`, 등등 에 따라 좀더 구체적인
`routes` 에 더 높은 순위를 매긴다

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
선택되기 원하는 것을 좀더 직관적으로 알 수 있다

> `/teams/:teamId` 보다는 `teams/new` 가 당연히 더 구체적인 `URL` 이다

랭킹화된 `routes` 로 인해, `route` 목록에 대한 걱정없이 사용할수 있다.

## 🤾‍♂️🔗 Active Links

대부분의 웹 앱은 `UI` 상단 , 사이드바, 또는 여러경로에 `natigation` 을 가지고 있다
활성화된 `navigation` 아이템들은 스타일링되어 `user` 는 어디인지(`isActive`), 어디로 가야할지(`isPending`) 쉽게 알수 있다

이러한 동작들을 `<NavLink>` 는 쉽게 구현할수 있도록 도와준다

```ts
<NavLink
  style={({ isActive, isPending }) => {
    return {
      color: isActive ? "red" : "inherit",
    };
  }}
  className={({ isActive, isPending }) => {
    return isActive ? "active" : isPending ? "pending" : "";
  }}
/>
```

또한, `links` 바깥에서 `active` 표시에 대해 `Match` 를 사용하여 확인할수도 있다

```ts
function SomeComp() {
  // match 하다면 PathMatch<ParamKey> : null 을 리턴한다
  const match = useMatch("/messages");
  // Boolean(match) 는 PathMatch<ParamKey> 타입인 객체를 반환하면 true
  // 아니면, false 이다
  return <li className={Boolean(match) ? "active" : ""} />;
}
```

## 🖇️ Relatvie Links

`<a href>`, `<Link to>` 그리고 `<NavLink to>` 같은 `HTML` 은
중첩된 라우트와 함께 향상된 동작을 위해 `relative path` 를 사용할 수 있다.

다음의 라우트 설정을 보자

`Route 계층 구조`

```tsx
<Route path="home" element={<Home />}>
  <Route path="project/:projectId" element={<Project />}>
    <Route path=":taskId" element={<Task />} />
  </Route>
</Route>
```

`https://example.com/home/project/123` 인 `url` 이며,
위의 `Route 계층 구조` 를 보면 다음의 `route` 컴포넌트 계층을 렌더링한다.

```tsx
<Home>
  <Project />
</Home>
```

`<Project>` 가 다음의 `links` 를 따른다면, `href` 는 다음과 같이 처리된다

| **In `<Project>` @ `/home/project/123`** | **Resolved `<a href>`** |
| :--------------------------------------- | :---------------------- |
| `<Link to="abc">`                        | `/home/project/123/abc` |
| `<Link to=".">`                          | `/home/project/123`     |
| `<Link to="..">`                         | `/home`                 |
| `<Link to=".." relative="path">`         | `/home/project`         |

`..` 은 `project/:project:id` 세그먼트를 둘다 지운다.
기본적으로, `..` `relative link` 는 `URL` 세그먼트가 아닌 `Route 계층 구조`를
탐색한다

추가된 `relative="path"` 는 기존의 `Route 계층 구조` 대신 `path` 세그먼트로
탐색하게 한다

## 📁 Data Loading

`URL` 세그먼트는 일반적으로 영구적 `data` 에 매핑하기 때문에,
`React Router` 는 `navigation` 중 초기 `data` 를 로딩하기 위한 컨밴션을 제공하기
위해 `hook` 을 사용한다

이를 중첩된 경로와 결합하면, 여러 `layouts` 에 지정된 `URL` 로부터 모든 데이터를
병렬로 `load` 할수 있게 된다.

```ts
<Route
  path="/"
  loader={async ({ request }) => {
    // loaders can be async functions
    const res = await fetch("/api/user.json", {
      signal: request.signal,
    });
    const user = await res.json();
    return user;
  }}
  element={<Root />}
>
  <Route
    path=":teamId"
    // loaders understand Fetch Responses and will automatically
    // unwrap the res.json(), so you can simply return a fetch
    loader={({ params }) => {
      return fetch(`/api/teams/${params.teamId}`);
    }}
    element={<Team />}
  >
    <Route
      path=":gameId"
      loader={({ params }) => {
        // of course you can use any data store
        return fakeSdk.getTeam(params.gameId);
      }}
      element={<Game />}
    />
  </Route>
</Route>
```

`Data` 는 컴포넌트내에서 `useLoaderData` 를 통해 사용하게 만들수 있다

```ts
function Root() {
  const user = useLoaderData();
  // data from <Route path="/">
}

function Team() {
  const team = useLoaderData();
  // data from <Route path=":teamId">
}

function Game() {
  const game = useLoaderData();
  // data from <Route path=":gameId">
}
```

## 💱 Redirects

`data` 를 변경 또는 로딩하는 동안, 유저를 다른 `route` 로 `redirect` 하는것은
일반적인 사용법이다.

```ts
<Route
  path="dashboard"
  loader={async () => {
    const user = await fake.getUser();
    if (!user) {
      // if you know you can't render the route, you can
      // throw a redirect to stop executing code here,
      // sending the user to a new route
      //
      // 만약에 route 를 렌더링할수 없다면,
      // 여기서 실행을 멈추고 user 를 에게 새로운 경로를 보내기 위해
      // redirect 한다
      throw redirect("/login");
    }

    // otherwise continue
    // 그렇지 않으면 지속 실행한다
    const stats = await fake.getDashboardStats();
    return { user, stats };
  }}
/>
```

```ts
<Route
  path="project/new"
  action={async ({ request }) => {
    const data = await request.formData();
    const newProject = await createProject(data);
    // it's common to redirect after actions complete,
    // sending the user to the new record
    //
    // 여기는 일반적으로 action 이 완료되면,
    // 새로운 data 를 사용자에게 보내기위해
    // 리다이랙트 한다.
    return redirect(`/projects/${newProject.id}`);
  }}
/>
```

## :construction: Pending Navigation UI

사용자가 앱을 탐색할 때, 다음 페이지에 대한 `data` 는 해당 페이지가 렌더링
되기 전에 로딩된다.

이것은 이 기간동안 사용자 피드백을 제공하는것은 중요하다
이러한 피드백을 통해 앱이 미응답하는것 처럼(`멈추는것 처럼`) 느끼지 않는다

```ts
function Root() {
  const navigation = useNavigation();
  return (
    <div>
      {navigation.state === "loading" && <GlobalSpinner />}
      <FakeSidebar />
      <Outlet />
      <FakeFooter />
    </div>
  );
}
```

## 💀 Skeleton UI with `<Suspense>`

다음페이지에 대한 데이터를 기다리는 대신, 데이터를 `defer`(`연기`) 할수 있다
그래서, `data` 를 로드하는 동안, `UI` 는 즉시 `placeholder`(`표시자`)를  
사용하여 다음 화면으로 전환한다

```ts
<Route
  path="issue/:issueId"
  element={<Issue />}
  loader={async ({ params }) => {
    // these are promises, but *not* awaited
    const comments = fake.getIssueComments(params.issueId);
    const history = fake.getIssueHistory(params.issueId);
    // the issue, however, *is* awaited
    const issue = await fake.getIssue(params.issueId);

    // defer enables suspense for the un-awaited promises
    return defer({ issue, comments, history });
  }}
/>;

function Issue() {
  const { issue, history, comments } = useLoaderData();
  return (
    <div>
      <IssueDescription issue={issue} />

      {/* Suspense provides the placeholder fallback */}
      <Suspense fallback={<IssueHistorySkeleton />}>
        {/* Await manages the deferred data (promise) */}
        <Await resolve={history}>
          {/* this calls back when the data is resolved */}
          {(resolvedHistory) => <IssueHistory history={resolvedHistory} />}
        </Await>
      </Suspense>

      <Suspense fallback={<IssueCommentsSkeleton />}>
        <Await resolve={comments}>
          {/* ... or you can use hooks to access the data */}
          <IssueComments />
        </Await>
      </Suspense>
    </div>
  );
}

function IssueComments() {
  const comments = useAsyncValue();
  return <div>{/* ... */}</div>;
}
```

## 📲 Data Mutations

`HTML form` 은 `link` 처럼, `navigation` 이벤트이다
`React Router` 는 클라이언트 쪽 라우팅을 위한 `HTML form` 에 대한 워크플로우를
제공한다

`form` 을 `submit` 할때, `submit` 의 `formData` 를 담은 `body` 와 함께
일반적인 브라우저 탐색 이벤트가 방지되고 `request` 는 생성된다

이 `request` 는 `<Route action>` 으로 보내지며, 이는 `<Form action>` 폼에
매치된다

`Form` 요소의 `name` 프로퍼티는 `action` 으로 `submit` 한다

```tsx
<Form action="/project/new">
  <label>
    Project title
    <br />
    <input type="text" name="title" />
  </label>

  <label>
    Target Finish Date
    <br />
    <input type="date" name="due" />
  </label>
</Form>
```

일반적인 `HTML` 문서 `request` 이 차단되며, 매치된 `route` 액션으로 보내진다

> `<Route path>` 는 `<form action>` 에 매치된다.

다음은 이 `route` 액션에 `request.formData` 에 포함되어있는것을 볼수 있다.

```tsx
<Route
  path="project/new"
  action={async ({ request }) => {
    const formData = await request.formData();
    const newProject = await createProject({
      title: formData.get("title"),
      due: formData.get("due"),
    });
    return redirect(`/projects/${newProject.id}`);
  }}
/>
```

## ✔️ Data Revalidation

`React router` 의 `HTML` 기반 데이터 변이 `API` 는 과거 웹 개발에서 많이
사용된 방식을 사용한다

> 과거 웹에서 `Form` 데이터는 `POST` 를 통해 웹서버로 요청을 보내고,
> 이렇게 보낸 `data` 를 통해 렌더링된 새로운 페이지를 보냈다.

`React router` 는 `action` 이 호출된 후에 페이지의 모든 데이터에 대한
`loader` 가 호출된다.

이 `loader` 를 통해 `UI` 가 데이터와 자동으로 동기화 되며,
캐시키를 만료시키거나 컨텍스트 프로바이더를 다시 로드할 필요없이 사용가능하다

## 👓 Busy Indicators

> `Busy Indicator` 는 `UI` 상 작업중이라는 것을 나타내는 시각적인 표시자 라고
> 보면 된다.

`form` 이 라우트 `action` 으로 `submit` 될때, `navigation.state` 에 접근하여
`fieldset` 을 `disable` 하는등의 `busy indicator` 를 표시한다

```tsx
function NewProjectForm() {
  const navigation = useNavigation();
  // 현재 작업중이라는 것을 알기위해
  // navigation.state 값이 `submitting` 인지 비교한다
  const busy = navigation.state === "submitting";
  return (
    <Form action="/project/new">
      <fieldset disabled={busy}>
        <label>
          Project title
          <br />
          <input type="text" name="title" />
        </label>

        <label>
          Target Finish Date
          <br />
          <input type="date" name="due" />
        </label>
      </fieldset>
      <button type="submit" disabled={busy}>
        {/* 
        busy 값이 true 라면, creating... 이고 아니면 create 를
        반환한다.
        */}
        {busy ? "Creating..." : "Create"}
      </button>
    </Form>
  );
}
```

## 😄 Optimistic UI

`action` 으로 전송되는 `formData` 를 아는것만으로도, `busy indicator` 를
건너뛸수 있는 경우가 많다 이는 설사 여전히 대기중인 비동기라 할지라도,
`UI` 에 즉각적으로 다음 `state` 를 렌더링할수 있다.

이러한 방식을 `Optimistic UI` 라 부른다

```tsx
function LikeButton({ tweet }) {
  const fetcher = useFetcher();

  // if there is `formData` then it is posting to the action
  // 만약 formData 가 있고, action 으로 posting 된다면
  const liked = fetcher.formData
    ? // check the formData to be optimistic
      // formData 를 낙관적으로 체크한다
      fetcher.formData.get("liked") === "yes"
    : // if its not posting to the action, use the record's value
      // 만약 action 으로 posting 되지 않았다면, 적정된 값을 사용한다
      tweet.liked;

  return (
    <fetcher.Form method="post" action="toggle-liked">
      <button
        type="submit"
        name="liked"
        {/* liked 값에 따라 value 값이 지정된다 */}
        value={liked ? "yes" : "no"}
      />
    </fetcher.Form>
  );
}
```

그리고 이에 따라 `value` 값을 지정한다
`fatcher` 를 사용하여 `optimistic` 을 사용하는것이 더 일반적이지만,
이는 `navigation.formData` 를 사용하는 일반적인 형식으로도 동일한 작업을
수행할수 있다.

## Data Fetchers

`HTML` `forms` 는 뮤테이션에 대한 모델이다
그러나 하나의 `주요 제한사항`을 가진다

> 제한사항으로 `form` `submit` 은 `navigation`(`페이지이동`) 이므로, 오직 한번만 할수 있다.

대부분의 웹 앱은 같은 시간에 발생하는 `완료표시`,
`각각 독립적으로 삭제할수 있는 저장된 목록`, 등 여러 `mutation`(`변형`)을
허용해야 한다

`Fetcher` 는 브라우저 `navigation`(`페이지 이동`) 을 유발하지 않고,
`route` 의 `action` 과 `loader` 와의 상호작용을 허용한다

그러나, `error` 핸들링, `revalidation`, `interruption` 핸들링, 그리고
`race condition` 핸들링 같은 기존의 것들 모두를 얻어 사용할수 있다.

```tsx
function Tasks() {
  const tasks = useLoaderData();
  return tasks.map((task) => (
    <div>
      <p>{task.name}</p>
      <ToggleCompleteButton task={task} />
    </div>
  ));
}
```

각 작업은 나머지 작업과 독립적으로 완료로 표시되며, `padding` 상태를 가지며,
`fetcher` 를 사용하여 `navigation`(`페이지 이동`)하지 않아도 된다

```tsx
function ToggleCompleteButton({ task }) {
  const fetcher = useFetcher();

  return (
    <fetcher.Form method="post" action="/toggle-complete">
      <fieldset disabled={fetcher.state !== "idle"}>
        <input type="hidden" name="id" value={task.id} />
        <input
          type="hidden"
          name="status"
          value={task.complete ? "incomplete" : "complete"}
        />
        <button type="submit">
          {task.status === "complete" ? "Mark Incomplete" : "Mark Complete"}
        </button>
      </fieldset>
    </fetcher.Form>
  );
}
```

## 🐎 Race Condition Handling

`React Router` 는 오래된 연산을 취소하고, 자동적으로 새로운 데이터만 커밋한다

비동기 `UI` 는 언제라도 `race condition` 의 리스크를 가진다

> 비동기 연산이 **이전 작업이후에 시작되지만, 먼저 완료된 경우** 그 결과
> 잘못된 상태값을 보여주게 된다

```sh
?q=ry    |---------------|
                         ^ commit wrong state
?q=ryan     |--------|
                     ^ lose correct state
```

위를 보면 `q?=ryan` 은 나중에 쿼리하지만, 먼저 완료된다
만약, 옳바르게 핸들링하지 않는다면, 이 결과는 `?q=ryan` 에 대한 옳바른 값이
적용되지만, `?q=ry` 에 대한 옳바르지 않은 값으로 변하게 된다

이는 `Throttling` 과 `debouncing` 만으로 충분하지 않으며, 이전의 쿼리를
취소해야 한다

`React Rounter` 의 데이터 규칙을 사용하는경우 자동적으로 그리고 완벽하게 이
문제를 방지할 수 있다

```sh
?q=ry    |-----------X
                     ^ cancel wrong state when
                       correct state completes earlier
?q=ryan     |--------|
                     ^ commit correct state
```

`React Router` 는 이와 같이 페이지 이동에 대한 `race condition` 을
처리할 뿐만 아니라, `fetcher` 와 함께 동시성 `mutation` 처리 또는
`loading` 에 따라 결과값을 자동완성하는 것 처럼 다른 많은 경우에도 이를 처리한다

## 🚥 Error Handling

대부분의 어플리케이션 에러는 `React Router` 에 위해 자동적으로 핸들링된다
이는 다음과 같은 경우 `error` 를 던지고 받는다

- 렌더링
  <br/>

- 데이터 로딩
  <br/>

- 데이터 업데이트
  <br/>

이는 실제로, 앱에서 `event handler`(`<button onClick>`) 또는 `useEffect` 에서
발생한 것을 제외하고, 거의 대부분의 에러 이다.

에러가 발생할때, `router` 는 `element` 를 렌더링하는 대신, `errorElement` 를
렌더링한다

```tsx
<Route
  path="/"
  loader={() => {
    something.that.throws.an.error();
  }}
  // this will not be rendered
  element={<HappyPath />}
  // but this will instead
  errorElement={<ErrorBoundary />}
/>
```

만약, `Route` 에 `errorElement` 가 없다면, 이 에러는 가까이 있는 부모 `Route` 로
타고 올라가, 부모 `Route` 의 `errorElement` 를 렌더링한다

```tsx
<Route path="/" element={<HappyPath />} errorElement={<ErrorBoundary />}>
  {/* Errors here bubble up to the parent route */}
  <Route path="login" element={<Login />} />
</Route>
```

## 📜 Scroll Restoration

`React Router` 는 페이지 이동에서 브라우저 `scroll Restoration`(`복원`)을
계산을 한다

이렇게 하면 스크롤 위치가 올바른 위치로 복원된다
또한, 어떠한 다른 `location` 에 기반한 `restoring`(`복원`)을 위한 동작으로
사용자정의할수 있고, 특정 링크에서 스크롤이 발생하지 않도록 방지한다

## 🌍 Web Standard APIs

`React Router` 는 `web` 표준 `APIs` 로 만들어졌다.
`loaders` 그리고, `actions` 는 `Web Fetch APIs` 의 `request` 객체 그리고
`response` 객체를 받는다

취소는 `Abort Signals` 통해 이루어지며, `serch params` 는 `URLSearchParams` 로
처리되고, 데이터 `mutation` 은 `HTML Forms` 로 처리한다

`React Router` 를 더 잘하게되면 웹 플랫폼도 더 잘하게 된다고 한다.
