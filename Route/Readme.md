# 🔥️ Route

`Route` 에서 제공하는 각 기능을 살펴본다

---

## 🛣️ Route

`Routes` 는 어쩌면 `React Route` 앱에서 가장 중요한 부분일지도 모른다
`Routes` 는 `URL` 세그먼트를 `Component` 와 연결짓고, `data` 를 로딩하고,  
`data` 를 변환한다.

`Route` 중첩을 통해서, 복잡한 어플리케이션 레이아웃과 간단하게
독립적인 `data` 를 만들고 선언한다

`Routes` 는 생성한 `route` 에 여러 기능들을 객체를 전달한다

1️⃣ `번 예시`

```tsx
const router = createBrowserRouter([
  {
    // 해당하는 컴포넌트를 렌더링
    element: <Team />,

    // URL 이 이 세그먼트와 같을때,
    path: "teams/:teamId",

    // 렌더링 되기전에 이 데이터와 함께 로딩
    loader: async ({ request, params }) => {
      return fetch(`/fake/api/teams/${params.teamId}.json`, {
        signal: request.signal,
      });
    },

    // 데이터가 submit 될때, 이 mutation 을 실행
    action: async ({ request }) => {
      return updateFakeTeam(await request.formData());
    },

    // 그리고 뭔가 잘못되었을 경우에 이 엘리먼트를 렌더링 한다
    errorElement: <ErrorBoundary />,
  },
]);
```

또한 `JSX` 와 `createRouteFromElements` 를 사용하여 `routes` 를 선언할수 있다.
`element` 에 대한 `props` 는 `route` 객체의 프로퍼티(`상단의 router 프로퍼티`)와 동일하다.

2️⃣ `번 예시`

```tsx
const router = createBrowserRouter(
  createRoutesFromElements(
    <Route
      element={<Team />}
      path="teams/:teamId"
      loader={async ({ params }) => {
        return fetch(`/fake/api/teams/${params.teamId}.json`);
      }}
      action={async ({ request }) => {
        return updateFakeTeam(await request.formData());
      }}
      errorElement={<ErrorBoundary />}
    />
  )
);
```

> 📎
> 여기서 말하고자하는 바는 `createBrowserRouter` 를
> 사용하는 방법은 총 2가지라는 말이다
>
> 1. `Router` 객체를 가진 배열을 사용하여 선언 (`1번 예시`)
> 2. `createRouteFromElements` 를 사용하여 `JSX` 컴포넌트로 `router` 선언(`2번 예시`)
>    <br/>

이 방식들은 동일한 동작을 하며 어느쪽을 사용하든 상관없다
이 문서의 대부분에서는 `JSX` 방식을 사용할 것이다. 왜냐하면 대부분의
사람들이 `React Router` 의 컨텍스트에 익숙하기 때문이다

> ❗ **Note**
>
> `RouterPovider` 를 사용할때,
> 만약 `React element` (예: `element={<MyComponent>}`) 를 지정하기를 원치
> 않는다면, 대신 `Component` (예: `Component={MyComponent}`) 를 지정할수
> 있다
> 그리고 `React Router` 는 내부적으로 `createElement` 를 호출할 것이다
>
> `Routes` 내부에서 `Component` 를 사용하면 렌더링 전체에서
> 생성된 요소를 재사용하는 `React` 의 기능을 저하시킨다 (`최적화 하지 못한다.`)
>
> > 이는 해당 컴포넌트는 매 렌더링시에 새로 생성하는 결과를 갖는다
> > (`Component` `prop` 사용시 `createElement` 를 내부적으로 실행한다고 했다.)
> >
> > 그러므로, 리액트가 자랑하는 재사용 매커니즘을 방해하며 불필요한
> > 성능 저하를 초래한다
>
> 그렇기에, 성능최적화를 위해서 `RouterProvider` 를 사용하여 `Component` 를
> 사용하길 권장한다
> <br/>

`Route` 의 타입은 다음과 같다

```ts
interface RouteObject {
  path?: string;
  index?: boolean;
  children?: React.ReactNode;
  caseSensitive?: boolean;
  id?: string;
  loader?: LoaderFunction;
  action?: ActionFunction;
  element?: React.ReactNode | null;
  hydrateFallbackElement?: React.ReactNode | null;
  errorElement?: React.ReactNode | null;
  Component?: React.ComponentType | null;
  HydrateFallback?: React.ComponentType | null;
  ErrorBoundary?: React.ComponentType | null;
  handle?: RouteObject["handle"];
  shouldRevalidate?: ShouldRevalidateFunction;
  lazy?: LazyRouteFunction<RouteObject>;
}
```

### path

이 `route` 가 `URL`, `link` 의 `href`, `form action` 과 매치되는지
확인하기 위해 `URL` 과 일치시킬 경로 패턴이다.

### Dynamic Segments

`path` 의 세그먼트가 `:` 와 함께 시작된다면, 이는 `dynamic segment` 가 된다

`URL` 이 `route` 와 일치할때, 이 `dynamic segment` 는 `URL` 에서
구문분석하고, 다른 `router APIs` 에서 `params` 로 제공된다

```tsx
<Route
  // `path` 는 다음중 하나의 `URL` 과 매치된다
  // - /teams/hotspur
  // - /teams/real
  path="/teams/:teamId"
  // 매치된 params 는 loader 에 존재할될것이다
  loader={({ params }) => {
    console.log(params.teamId); // "hotspur"
  }}
  // 액션
  action={({ params }) => {}}
  element={<Team />}
/>;

// useParams 를 통한 element 에서의 접근
function Team() {
  let params = useParams();
  console.log(params.teamId); // "hotspur"
}
```

여러 하나의 경로 `path` 에 여러 `dynamic segments` 를 가질수 있다

```ts
<Route path="/c/:categoryId/p/:productId" />;
// 양쪽 모두 존재한다
params.categoryId;
params.productId;
```

다음과 같이 `partial dynamic segments` 일수 없다

- 🚫 `"/teams-:teamId"`
  <br/>

- ✅ `"/teams/:teamId"`
  <br/>

- 🚫 `"/:category--:productId"`
  <br/>

- ✅ `"/:productSlug"`

> `-` 같은 특수문자를 포함하는 경우 `partial dynamic segment` 라고 한다
> `React Router v6` 에서는 공식적으로 지원하지는 않는 기능이다

`partial dynamic segment` 를 사용하기 위해서는
약간의 구문분석을 수행해야 한다

```ts
function Product() {
  const { productSlug } = useParams();
  const [category, product] = productSlug.split("--");
  // ...
}
```

> 이는 공식적으로 지원하는 기능이 아니니, 이왕이면 보통의
> `dynamic segments` 를 사용하자

### Optional Segments

끝에 `"?"` 을 추가하면 세그먼트를 옵셔널하게 만들수 있다

```tsx
<Route
  // this path will match URLs like
  // - /categories
  // - /en/categories
  // - /fr/categories
  path="/:lang?/categories"
  // the matching param might be available to the loader
  loader={({ params }) => {
    console.log(params["lang"]); // "en"
  }}
  // and the action
  action={({ params }) => {}}
  element={<Categories />}
/>;

// and the element through `useParams`
function Categories() {
  let params = useParams();
  console.log(params.lang);
}
```

선택적 `static arguments` 역시 가질수 있다

```tsx
<Route path="/project/task?/:taskId" />
```

### Splats

`route` 의 `path` 끝에 `/*` 와 함께 사용한다면, 그때 이 `path` 는
`/` 에 따라오는 어떠한 문자라도 매치된다, 이는 뒤따라 오는 다른 `/`
문자도 포함된다

```tsx
<Route
  // this path will match URLs like
  // - /files
  // - /files/one
  // - /files/one/two
  // - /files/one/two/three
  path="/files/*"
  // the matching param will be available to the loader
  loader={({ params }) => {
    console.log(params["*"]); // "one/two"
  }}
  // and the action
  action={({ params }) => {}}
  element={<Team />}
/>;

// and the element through `useParams`
function Team() {
  let params = useParams();
  console.log(params["*"]); // "one/two"
}
```

또한 `*` 을 구조분해하여 새로운 변수 이름에 할당할 수 있다
보통 이러한 변수 이름을 `splat` 으로 짓는다

```tsx
let { org, "*": splat } = params;
```

### Layout Routes

`Layout route` 인 `route` 는 `path` 를 생략하여 만든다
이것은 `UI` 를 중첩하지만, `URL` 에 세그먼트를 추가하지 않는다

```tsx
<Route
  element={
    <div>
      <h1>Layout</h1>
      <Outlet />
    </div>
  }
>
  <Route path="/" element={<h2>Home</h2>} />
  <Route path="/about" element={<h2>About</h2>} />
</Route>
```

이 예제에서는 `<h1>Layout</h1>` 과 함께 `layout route` 인 `<Outlet />` 에 `child route` 이 렌더링된다

### index

`route` 가 `index route` 인지 알려준다.
`index route` 는 자신의 부모인 `<Outlet>` 에서 `index route` 로 렌더링된다

```tsx
<Route path="/teams" element={<Teams />}>
  <Route index element={<TeamsIndex />} />
  <Route path=":teamId" element={<Team />} />
</Route>
```

이 특별한 `route` 는 처음 이해하기에 혼란스러울수 있다. 그래서 이에 대한
가이드를 위해 [Index Route](https://reactrouter.com/en/main/start/concepts#index-routes) 을 보라고 한다

### caseSensitive

대소문자를 구분을 할지 안할지 지정하는 명령

```tsx
<Route caseSensitive path="/wEll-aCtuA11y" />
```

- `"wEll-aCtuA11y"` 와 매치된다
  <br/>

- `"well-actua11y"` 와는 매치되지 않는다

### loader

`route loader` 는 `route` 를 렌더하기 전에 호출되고,
`useLoaderData` 를 통해 `element` 에서 `data` 를 받을수 있다

```ts
<Route
  path="/teams/:teamId"
  loader={({ params }) => {
    return fetchTeam(params.teamId);
  }}
/>;

function Team() {
  let team = useLoaderData();
  // ...
}
```

> ⚠️ **IMPORTANT**
> 만약 `createBrowserRouter` 같은 `router` 데이터를 사용하지 않았다면,
> 아무런 동작도 하지 않는다.

이는 `loader` 문서에 더 자세한 설명이 있다.

> 📎 이는 나중에 추가적으로 작성할 예정이다

### action

`Form`, `fetcher` 또는 `submission` 에서 경로로 `submit` 이 전송되면,
`route action` 이 호출된다

```tsx
<Route
  path="/teams/:teamId"
  action={({ request }) => {
    const formData = await request.formData();
    return updateTeam(formData);
  }}
/>
```

> ⚠️ **IMPORTANT**
> 만약 `createBrowserRouter` 같은 `router` 데이터를 사용하지 않았다면,
> 아무런 동작도 하지 않는다.

이는 `action` 문서에 더 자세한 설명이 있다.

> 📎 이는 나중에 추가적으로 작성할 예정이다

### element / component

`URL` 이 `route` 와 매치될때, `React Element` / `Component` 로 렌더링된다

`React Element` 를 생성하기 원한다면, `element` 를 사용한다

```tsx
<Route path="/for-sale" element={<Properties />} />
```

그렇지 않으면 `Component` 를 사용한다 그러면, `React Router` 는
주어진 컴포넌트로 `React Element` 를 생성할 것이다

```tsx
<Route path="/for-sale" Component={Properties} />
```

### errorElement / ErrorBoundary

`route` 에서 렌더링중 `loader` 또는 `action` 에서 예외를 발생시킬때,
이 `React Element/Component` 는 일반적인 `element` / `Component` 를 대신해서
렌더링 된다

> 여기서 영어라서 그런지 헷갈리는 용어가 나온다
> 일반적인 `element` / `Component` 와 `this reactElement` / `Component` 가 그렇다..
>
> 여기서 사용된 일반적인 `element` / `Component` 는 렌더링될 컴포넌트 혹은
> `JSX Element` 를 말한다
>
> 반면, `this React Element`/ `Component` 는 앞에서 말한 예외를 발생시킬때
> 사용될 `JSX Element` 혹은 `Component` 를 말한다

만약 이미 소유한 `React Element` 를 생성하기를 원한다면, `errorElement` 를
사용한다

```tsx
<Route
  path="/for-sale"
  // if this throws an error while rendering
  element={<Properties />}
  // or this while loading properties
  loader={() => loadProperties()}
  // or this while creating a property
  action={async ({ request }) => createProperty(await request.formData())}
  // then this element will render
  errorElement={<ErrorBoundary />}
/>
```

반면, `Component` 형식으로 적용하고 싶다면, `ErrorBoundary` 를 사용한다
이는 `React Router` 가 내부적으로 `react element` 로 만들어줄것이다

```tsx
<Route
  path="/for-sale"
  Component={Properties}
  loader={() => loadProperties()}
  action={async ({ request }) => createProperty(await request.formData())}
  ErrorBoundary={ErrorBoundary}
/>
```

> ⚠️ **IMPORTANT**
> 만약 `createBrowserRouter` 같은 `router` 데이터를 사용하지 않았다면,
> 아무런 동작도 하지 않는다.

이에 대해선 더 잘알고 싶다면 `errorElement` 문서를 찾아보라고 한다

> 📎 이는 나중에 추가적으로 작성할 예정이다

### `hydrateFallbackElement` / `HydrateFallback`

만약, `Server-Side Rendering` 을 사용하고 부분적으로 `hydration` 을 활용
하기도 한다

이때 아직 `hydration` 되지 않은 `router` 에 대한 `ReactElement` / `Component` 를 지정할 수 있다

> ⚠️ **IMPORTANT**
> 만약 `createBrowserRouter` 같은 `router` 데이터를 사용하지 않았다면,
> 아무런 동작도 하지 않는다.

이에 대해선 더 잘알고 싶다면 `hydrateFallbackElement` 문서를 찾아보라고 한다

> 📎 이는 나중에 추가적으로 작성할 예정이다

### Handle

모든 애플리케이션별 데이터이다.
이에 대해 더 자세히 알고 싶다면 `useMatche` 문서를 살펴보라고 한다

> 📎 이는 나중에 추가적으로 작성할 예정이다

### lazy

어플리케이션에서 번들을 작게 유지하고, `route` 에서
코드 스플리팅을 지원하기 위해서 각 `route` 는 비동기 함수를 제공한다

이 비동기 함수가 `lazy` `prop` 이다.

`lazy` 지원하기 위해서 `route` 사용시, 사용되지 않은 여러 `props` 에 대한
정의를 하기도 한다

이는 예시를 보아야 이해가 쉬울듯 싶다 아래를 보도록하자.
각 `lazy` 함수는 일반적으로 동적 `import` 의 결과를 리턴한다

```tsx
let routes = createRoutesFromElements(
  <Route path="/" element={<Layout />}>
    <Route path="a" lazy={() => import("./a")} />
    <Route path="b" lazy={() => import("./b")} />
  </Route>
);
```

이때, `lazy` `route` 모듈은, `route` 에 정의할 원하는 프로퍼티를 내보낸다

```tsx
export async function loader({ request }) {
  let data = await fetchData(request);
  return json(data);
}

export function Component() {
  let data = useLoaderData();

  return (
    <>
      <h1>You made it!</h1>
      <p>{data}</p>
    </>
  );
}
```

> ⚠️ **IMPORTANT**
> 만약 `createBrowserRouter` 같은 `router` 데이터를 사용하지 않았다면,
> 아무런 동작도 하지 않는다.

이에 대해 더 자세히 알고 싶다면 `lazy` 문서를 살펴보라고 한다

> 📎 이는 나중에 추가적으로 작성할 예정이다

---

<!--  markdownlint-disable-next-line -->

## action

`route action` 은 쓰고, `route loader` 에서 읽는다

`React Router` 는 복잡한 비동기 `UI` 를 추상화하고, 재평가하는 동시에
간단한 `HTML` 그리고 `HTTP` 를 이용하여 `data` 의 `mutation` 을 실행하기 위한
방법을 제공한다

이는 현대적 `SPAs` 의 `UX` 동작 및 기능과 함께, `HTML` 과 `HTTP` 의 간단한 멘탈모델을 제공한다.

```tsx
<Route
  path="/song/:songId/edit"
  element={<EditSong />}
  action={async ({ params, request }) => {
    let formData = await request.formData();
    return fakeUpdateSong(params.songId, formData);
  }}
  loader={({ params }) => {
    return fakeGetSong(params.songId);
  }}
/>
```

`action` 은 `route` 에서 `GET` 이 아닌 `submit` 을 보낼때 마다 호출된다

> `POST`, `PUT`, `PATCH`, `DELETE` 에서 호출된다

```tsx
// forms
<Form method="post" action="/songs" />;
<fetcher.Form method="put" action="/songs/123/edit" />;

// imperative submissions
let submit = useSubmit();
submit(data, {
  method: "delete",
  action: "/songs/123",
});
fetcher.submit(data, {
  method: "patch",
  action: "/songs/123/edit",
});
```

### params

`Route` 의 `params` 는 `dynamic segments` 로 부터 구문분석되며,
`action` 에 전달된다

이는 어떤 리소스를 변경할지 알아내는데 유용하다

```tsx
<Route
  path="/projects/:projectId/delete"
  action={({ params }) => {
    return fakeDeleteProject(params.projectId);
  }}
/>
```

### request

이는 `route` 로 전송되는 `Fetch Request` 인스턴스이다.
대부분의 경우 `request` 로 부터 `FormData` 를 구문분석하는데 사용된다

```tsx
<Route
  action={async ({ request }) => {
    let formData = await request.formData();
    // ...
  }}
/>
```

처음에는 액션에서 `request` 를 받는것이 이상해보이겠지만, 이런 코드를
작성해 본적이 있다면 생각이 달라진다

```tsx
<form
  onSubmit={(event) => {
    event.preventDefault();
    // ...
  }}
/>
```

`Javascript` 가 없이, `HTML` 과 `HTTP` 웹 서버는 기본값으로 `event` 를
`prevent` 하며 실제로 꽤나 훌륭하다고 설명한다

> 위 설정값인 `event.preventDefault` 가 기본값이라는 이라고 말하는듯하다

브라우저는 `FormData` 안에서 `data` 를 직렬화할것이고, 서버로 새로운
요청의 `body` 를 보낸다

위의 코드처럼 `React Router` `<Form>` 은 해당 `request` 를 보내는
브라우저를 `prevent` 고, 대신 `route action` 으로 `request` 를 보낸다

이는 `HTML` 과 `HTTP` 모델을 심플하게 만들어 굉장히 동적인 `web app` 을
가능하게 한다

`formData` 안의 값은 `form` `submit` 으로 부터 자동적으로 직렬화되므로
`inputs` 에 `name` 이 꼭 필요하다는 것을 기억해야 한다

```tsx
<Form method="post">
  <input name="songTitle" />
  <textarea name="lyrics" />
  <button type="submit">Save</button>
</Form>;

// accessed by the same names
formData.get("songTitle");
formData.get("lyrics");
```

### Opt-in serialization types

`useSubmit` 을 사용할때, `encType: "application/json"` 또는
`encType: "text/plain"` 을 전달하여 `paylaod` 를
`request.json()` 또는 `request.text()` 로 직렬화할수도 있다.

### Throwing in Actions

`action` 내에서 현재 `call stack` 을 벗어나려면 `throw` 를
사용할 수 있다

> `call stack` 에서 벗어난다는 것은 현재 실행중인 코드를 중단한다는
> 의미이다

`React Router` 는 `error` 경로에서 다시 시작된다

> 이는 `route` 에 설정된 `errorElement` 및 `ErrorBoundary` 를
> 사용하여 처리된다는 뜻이다

```tsx
<Route
  action={async ({ params, request }) => {
    const res = await fetch(`/api/properties/${params.id}`, {
      method: "put",
      body: await request.formData(),
    });
    if (!res.ok) throw res;
    return { ok: true };
  }}
/>
```

### 🎬 Handling multiple actions per route

꽤 자주 등장하는 공통적인 질문은 다음과 같다

**_" 만약, `action` 에서 다른 동작을 하는 여러 `handler` 가 필요하면 어떻게 하나요?"_**

이는 여러 방법들로 해결할수 있지만, 일반적으로 가장 간단한 것은
`<button type="submit">` 에 `name`/`value` 에 추가하는 것이다

그리고 이는 코드 실행하면 해당하는 `action` 을 가리킨다

코드를 보도록 하자

```tsx
async function action({ request }) {
  let formData = await request.formData();
  let intent = formData.get("intent");

  if (intent === "edit") {
    await editSong(formData);
    return { ok: true };
  }

  if (intent === "add") {
    await addSong(formData);
    return { ok: true };
  }

  throw json({ message: "Invalid intent" }, { status: 400 });
}

function Component() {
  let song = useLoaderData();

  // When the song exists, show an edit form
  if (song) {
    return (
      <Form method="post">
        <p>Edit song lyrics:</p>
        {/* Edit song inputs */}
        <button type="submit" name="intent" value="edit">
          Edit
        </button>
      </Form>
    );
  }

  // Otherwise show a form to add a new song
  return (
    <Form method="post">
      <p>Add new lyrics:</p>
      {/* Add song inputs */}
      <button type="submit" name="intent" value="add">
        Add
      </button>
    </Form>
  );
}
```

만약 `button` 에서 `name` / `value` 을 사용한 경우, 위에 주석처리된
`input` 을 보낼것이다.

그리고 위의 경우처럼 `intend` 이름을 사용해도 되고, `<Form method>` `prop` 을
통해 다른 `HTTP` 메서드를 `submit` 해도 된다

> `<Form method>` 의 `props` 는 다음과 같다
> `POST` 는 `add`, `PUT` / `PATCH` 는 `edit`, `DELETE` 는 제거
>
> 이러한 `Form method` 를 식별해서 다른 처리가 가능하다고 하는것이다
