# Components

`Components` 들은 `Route` 컴포넌트와 상호작용하기 위해, 랜더링할 컴포넌트에서
사용할 `React Router` 에서 제공하는 컴포넌트들의 묶음이다.

---

## `<Await>`

타입은 다음과 같다

```ts
declare function Await(props: AwaitProps): React.ReactElement;

interface AwaitProps {
  children: React.ReactNode | AwaitResolveRenderFunction;
  errorElement?: React.ReactNode;
  resolve: TrackedPromise | any;
}

interface AwaitResolveRenderFunction {
  (data: Awaited<any>): React.ReactElement;
}
```

`React Router` 에서는 자동 `error` 핸들링과 `defferd value` 렌더링할때
사용한다고 한다

이를 위해서는, `loader` 에서 유틸리티함수인 `defer` 함수를 사용하여
리턴한 값이 필요하다
이는 `promise` 값을 `resolve` 한 값이 아닌, `promise` 값 자체를 넘겨주면
나중에 `resolve` 하도록 해주는 함수이다

`defer`

```tsx
async function loader() {
  // 첫번째 변수는 promise 의 resolve 한 값을 받는다
  let book = await getProduct();
  // 두번째 변수는 `promise` 값을 받는다
  let reviews = getProductReviews();
  // defer 로 넘겨주면 알아서 resolve 해준다
  return defer({ book, reviews });
}
```

이러한 방식으로, 사용한 `loader` 의 값을 받을때, 이 컴포넌트를 사용한다

이 `<Await>` 컴포넌트는 `fallback` `UI` 를 활성화하기 위해
`<React.Suspense>` 또는 `<React.SuspenseList>` 내부에서 렌더링해야 한다

`<Await>` 은 `promise` 를 받아 결과값이 `resolve` 될때까지 `fallback UI` 를
보여준다

```tsx
import { Await, useLoaderData } from "react-router-dom";

function Book() {
  const { book, reviews } = useLoaderData();
  return (
    <div>
      <h1>{book.title}</h1>
      <p>{book.description}</p>
      <React.Suspense fallback={<ReviewsSkeleton />}>
        <Await
          resolve={reviews}
          errorElement={<div>Could not load reviews 😬</div>}
          children={(resolvedReviews) => <Reviews items={resolvedReviews} />}
        />
      </React.Suspense>
    </div>
  );
}
```

위의 예시를 보면, `<React.Suspense fallback={...}>` 을 통해,
`<Await>` 컴포넌트가 `resolve` 될때까지, `fallback` 을 보여준다

`defer` 에 의해 `resolve` 가 되는것을 감지하면, 그때, `<Await>` 은
자신의 `children` 을 렌더링한다

만약, `reject` 을 감지하면, `errorElement` 를 렌더링한다

## `children`

`React element` 혹은 `function` 둘중 하나일수 있다

`function` 을 사용할때, 오직 하나의 `parameter` 만 제공되어야 한다

```tsx
<Await resolve={reviewsPromise}>
  {(resolvedReviews) => <Reviews items={resolvedReviews} />}
</Await>
```

`React Element` 를 사용할때, `useAsyncValue` 로 `data` 를 받을수 있다

```tsx
<Await resolve={reviewsPromise}>
  <Reviews />
</Await>;

function Reviews() {
  const resolvedReviews = useAsyncValue();
  return <div>{/* ... */}</div>;
}
```

## `errorElement`

`promise` 가 `reject` 되었을때, `children` 대신 `error` `element` 를
렌더링한다

`error` 에 접근하려면, `useAsyncError` 를 통해 접근할수 있다

만약, `reject` 되었다면, 조작하기 위해 선택가능한 `props` 인 `errorElement`
사용할수 있으며, `useAsyncError` `hook` 을 통해 `error` 를 사용할수 있다.

```tsx
<Await resolve={reviewsPromise} errorElement={<ReviewsError />}>
  <Reviews />
</Await>;

function ReviewsError() {
  const error = useAsyncError();
  return <div>{error.message}</div>;
}
```

만약 `errorElement` 를 제공하지 않았다면, 이 `error` 는 버블링되어 근처의
`useRouteError` `hook` 과 함께 `errorElement` 를 사용하는 상위컴포넌트가
대신 처리한다

## `resolve`

`defferd loader` 에서 값이 `resolve` 되어 리턴된 `promise` 를 가진다.
그리고 렌더링된다

```tsx
import { defer, Route, useLoaderData, Await } from "react-router-dom";

// given this route
<Route
  loader={async () => {
    let book = await getBook();
    let reviews = getReviews(); // not awaited
    return defer({
      book,
      reviews, // this is a promise
    });
  }}
  element={<Book />}
/>;

function Book() {
  const {
    book,
    reviews, // this is the same promise
  } = useLoaderData();
  return (
    <div>
      <h1>{book.title}</h1>
      <p>{book.description}</p>
      <React.Suspense fallback={<ReviewsSkeleton />}>
        <Await
          // and is the promise we pass to Await
          resolve={reviews}
        >
          <Reviews />
        </Await>
      </React.Suspense>
    </div>
  );
}
```

---

## `<Form>`

타입은 다음과 같다

```ts
declare function Form(props: FormProps): React.ReactElement;

interface FormProps extends React.FormHTMLAttributes<HTMLFormElement> {
  method?: "get" | "post" | "put" | "patch" | "delete";
  encType?:
    | "application/x-www-form-urlencoded"
    | "multipart/form-data"
    | "text/plain";
  action?: string;
  onSubmit?: React.FormEventHandler<HTMLFormElement>;
  fetcherKey?: string;
  navigate?: boolean;
  preventScrollReset?: boolean;
  relative?: "route" | "path";
  reloadDocument?: boolean;
  replace?: boolean;
  state?: any;
  unstable_viewTransition?: boolean;
}
```

`Form` 은 `HTML` `form` 을 래핑한 컴포넌트이다
이는 클라이언트 쪽 라우팅과 `data` 변환을 위해 `browser` 를 에뮬레이트한다

이는 `React ecosystem` 에서 익숙할수 있는 `validation` / `state`
관리 라이브러리가 아니다

```tsx
import { Form } from "react-router-dom";

function NewEvent() {
  return (
    <Form method="post" action="/events">
      <input type="text" name="title" />
      <input type="text" name="description" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

> `input` 에는 `name` 값이 있어야 한다, 없으면 `formData` 에서 값을
> 확인할수 없다
>
> 다음의 글이 이해가 없는 상태에서 보면 이해가 가지 않는다
>
> All of this will trigger state updates to any rendered useNavigation hooks so you can build pending indicators and optimistic UI while the async operations are in-flight.
>
> 비동기 작업중에 `optimistic UI` 나 로딩중임을 알려주는 `indicator` 를
> 보여준다는 말은 이해했는데, **이 모든것은 rendered useNavigation hooks 의 상태업데이트를 트리거할 것이다** 라는 말이 이상하다..
>
> 해당 내용이 조금 불친절하다
> 이 내용을 좀더 자세하게 알아본다

🔢 내가 이해한 내부 로직의 순서는 다음과 같다

1️⃣ `<Form>` 을 작성하고 `action` 을 넘겨준다

2️⃣ `<Form>` 은 내부적으로 `useFormAction` `hook` 을 호출함과 동시에
`action` 값을 넘겨준다
<br/>

3️⃣ `useFromAction` 은 내부적으로 `useNavigation` `hook` 을 호출함과
동시에 `<From>` 에서 전달한 `action` 값을 넘겨준다
이는 넘겨준 `action` 값을 사용하여 해당하는 `route` 로 이동한다
<br/>

**이 모든것은 rendered useNavigation hooks 의 상태업데이트를 트리거할 것이다**
라는 말은 위의 3️⃣ 의 동작을 말한다. 이 동작은 `action` 값을 받아서
해당 `action` 으로 상태업데이트하고 이 업데이트된 상태를 기반으로
페이지를 이동한다.

이건 기존의 `HTML Form` 의 동작은 아니다
기존 `HTML Form` 은 `action` 값을 넘겼다고 해서 `submit` 된 이후
`page` 이동을 하지 않는다

> If the form doesn't feel like navigation, you probably want useFetcher.
>
> **만약 폼이 네비게이션 처럼 느껴지지 않는다면, `useFetcher` 를 사용하고
> 싶을것이다**
>
> 이 내용도 조금 이상하다.
> 이 내용은 간단하게, 페이지 이동을 하지 않고 데이터만 전송하는 경우에는
> `useFetcher` 를 사용하라는 것이다
>
> 내용을 계속 훑어보는데, 설명이 굉장히 불친절한 느낌이 든다..

### `action`

`action` 은 `<Form>` 에서 `submit` 될때 이동할 `url` 을 말한다
이는 `HTML form action` 의 `action` 과 다르게 행동한다

`HTML form` 은 `full URL` 을 받지만, `<Form>` 는 `context` 에서 가장 가까운
`relative URL` 이 기본값이다.

```tsx
function ProjectsLayout() {
  return (
    <>
      <Form method="post" />
      <Outlet />
    </>
  );
}

function ProjectsPage() {
  return <Form method="post" />;
}

<DataBrowserRouter>
  <Route
    path="/projects"
    element={<ProjectsLayout />}
    action={ProjectsLayout.action}
  >
    <Route
      path=":projectId"
      element={<ProjectsPage />}
      action={ProjectsPage.action}
    />
  </Route>
</DataBrowserRouter>;
```

현재 `URL` 이 `/project/123` 이라면, `ProjectsPage` 안의 `<Form>`은
기본적으로 현재 `URL` 인 `/project/123` 으로 설정된다

이 경우에는 `React Router` 의 `<Form>` 컴포넌트와 `HTML form` 태그 모두
동일한 결과가 된다

하지만, `useFormAction` `hook` 은 데이터 검증, 에러 처리, 페이지 이동등
추가적인 기능을 제공한다

> `preventDefault` 를 사용한다면, 페이지 이동을 막을수는 있다

만약, 다른 `route` 로 이동되길 원한다면, 다음처럼 `URL` 을 작성한다

```tsx
<Form action="/projects/new" method="post" />
```

### `method`

어떠한 `HTTP verb` 사용할지 결정한다
이는 `HTML form` 과 동일한 `attribute` 이다

이는 `'get'`, `'post'` 외에 `'put'`, `'patch'` 그리고 `'delete'` 또한
제공한다. `default` 값으로는 `get` 이다.

#### Get Submissions

`method` 의 `default` 값은 `get` 이다.
`get` 은 어떠한 `action` 도 호출하지 않는다.

`get` `submit` 은 사용자가 `Form` 에서 `URL` 로 이동하는 검색 매개변수를
제공한다는 점을 제외하면, 일반적인 페이지 이동(`사용자가 링크를 클릭`)과 동일하다

```tsx
<Form method="get" action="/products">
  <input aria-label="search products" type="text" name="q" />
  <button type="submit">Search</button>
</Form>
```

위의 예를 보자 `"/products"` 액션이며, `method` 가 `get` 이면
이는 일반적인 페이지 이동되도록 `Router` 를 `emutatie` 하므로, `form` 안에
`name` 의 이름과 값은 `URLSearchParams` 로 `sereialize` 된다.

그리고 `user` 는 `"/products?q=running+shoes"` 로 페이지 이동하게된다
이는 `<Link to="/products?q=running+shoes" />` 와 동일하다

그러나, `<Form>` 에서 `get`을 사용한 위의 코드상에서는 정해진 값 대신에
동적으로 값을 변경할수 있다는 장점이 있다

해당 `action` 경로를 가진 `Route` 는 `loader` 함수를 실행시키고,
해당 사용할 `loader` 는 `URLSearchParams` 를 사용해야 하므로
다음처럼 작성하게 될것이다

```tsx
<Route
  path="/products"
  loader={async ({ request }) => {
    let url = new URL(request.url);
    let searchTerm = url.searchParams.get("q");
    return fakeSearchProducts(searchTerm);
  }}
/>
```

여기서 사용된 `new URL` 을 사용해서 `searchParams` 의 `q` 를 얻어서
사용한다

#### Mutation Submissions

`get` 이외의 `method` 는 `mutation submissions` 이다.
이는 `post`, `put`, `patch`, `delete` 로 `data` 를 변경하도록 의도한
메서드라는 말이다

> `HTML form` 은 오직 `POST` 와 `GET` 만 제공하며 이 두가지를 고수하는
> 경향도 있다고 한다

`mutation submissions` 의 `method` 를 가지며, `<Form>` 에서 `submit` 했을때
주어진 매치되는 `route` 의 `path` 에 해당하는 `action` 을 호출한다

호출된 `action` 은 `serialize` 된 `FormData` 를 가진다
그리고 `action` 이 완료될때, `page` 의 모든 `loader` 는 `data` 와 동기화된 `UI` 를 유지하기위해 자동적으로 `revalidate` 한다

이 `method` 는 `route.action` 이 호출될때, `request.method` 로 활용할수 있다
이러한 방법은 `submission` 의 의도에 따라, `data` 를 추상화하여 실행하도록
지시가능하다

```tsx
<Route
  path="/projects/:id"
  element={<Project />}
  loader={async ({ params }) => {
    return fakeLoadProject(params.id);
  }}
  action={async ({ request, params }) => {
    switch (request.method) {
      case "PUT": {
        let formData = await request.formData();
        let name = formData.get("projectName");
        return fakeUpdateProject(name);
      }
      case "DELETE": {
        return fakeDeleteProject(params.id);
      }
      default: {
        throw new Response("", { status: 405 });
      }
    }
  }}
/>;

function Project() {
  let project = useLoaderData();

  return (
    <>
      <Form method="put">
        <input type="text" name="projectName" defaultValue={project.name} />
        <button type="submit">Update Project</button>
      </Form>

      <Form method="delete">
        <button type="submit">Delete Project</button>
      </Form>
    </>
  );
}
```

위는 같은 `router` 에서 `form` `submit` 하는것을 볼 수 있다
그러나 `request.method` 를 이용해서 분기처리한다

`action` 이 완료된 이후에, `loader` 는 `UI` 를 새로운 데이터와 함께
동기화 시키고 `revalidation` 한다.

### `naviagate`

`<Form naviage={false}>` 를 사용하여, `naviagation` 을 건너띄고 싶다고 알려주고, `fetch` 를 사용하도록 지정가능하다 이는 본질적으로 `submission`
만 하는 `useFetcher() + fetcher.Form` 와 같다고 한다.

이를 알기 위해서는 `userFetchers()` 와 `useFetcher()` `hook` 에 대해서
알 필요가 있다

---

#### `useFetcher`

`HTML` / `HTTP` 에 `data` 변형과 `load` 는 `<a href>` 와 `<form action>` 같은
페이지이동 통해 모델링된다

둘다, `browser` 에서 페이지 이동을 한다
`React Router` 는 `<Link>` 와 `<Form>` 이 그와 같은 기능을한다

그러나 때때로, 현재 페이지의 바깥의 `loader` 및 `action` 를 호출하기
원할때가 있다
또는 `caching` 된 `URL` 이 없거나 같은 시간에 여러 `mutation` 이
`data` 를 가져오는 중이라면 `page` 를 `revalidation`(`재검증`) 한다.

> `revalidation` 은 `data` 를 가져와서 `UI` 와 동기화 하는 과정을
> 말한다. 이는 해당 페이지를 리렌더링한다.
>
> `useFetcher` 는 마치 `React Query` 같이 값이 `stale` 한지 아닌지를
> 확인한후 `stale` 하다면 데이터를 새로 가져와 적용하고, 적용된 데이터를
> 페이지에 리렌더링 한다

서버와의 상호작용은 페이지 이동 `events` 가 아니며, 이 `hook` 은
페이지 이동없이도 `UI` 를 `loader` 그리고 `action` 에 연결할수 있다

이는 다음과 같을때 유용하게 사용가능하다

- `UI` 라우트와 연결되지 않은 `fetch` `data` 를 가져온다
  <br/>

- `페이지 이동` 없이 `action` 에 대한 `data` 를 `submit` 한다
  <br/>

- 리스트 안에서 여러 동시성 `submission` 을 처리한다
  (일반적으로 `"todo app"` 리스트 에서 여러 `button` 들을 클릭하고, 같은
  시간에 `pending` 상태 이어야 한다)

- 무한 스크롤 컨테이너

등등..

대화형의 `앱과 같은` 사용자 인터페이스를 구축하는 경우 `Fetcher` 를 자주
사용하게 된다

```tsx
import { useFetcher } from "react-router-dom";

function SomeComponent() {
  const fetcher = useFetcher();

  // call submit or load in a useEffect
  React.useEffect(() => {
    fetcher.submit(data, options);
    fetcher.load(href);
  }, [fetcher]);

  // build your UI with these properties
  fetcher.state;
  fetcher.formData;
  fetcher.json;
  fetcher.text;
  fetcher.formMethod;
  fetcher.formAction;
  fetcher.data;

  // render a form that doesn't cause navigation
  return <fetcher.Form />;
}
```

`fetcher` 는 다음과 같은 여러 `built-in` 동작을 가진다

- **`fetch` 에서 중단시 자동적으로 처리한다**
  <br/>

- **`action` 이 먼저 호출될때, 다음 작업이 실행된다** <br/>
  ▶️ 작업이 완료된후 페이지의 데이터가 `revalidation` 되어 발생했을 수 있는
  모든 `mutation` 을 캡쳐하고, 자동으로 `UI` 와 `server` 상태의 동기화를
  유지한다
  <br/>

- **한번에 여러가지 `fetcher` 가 진행중일때**<br/>
  ▶️ 데이터가 도착할때 마다 사용가능한 최신 데이터를 커밋한다<br/>
  ▶️ 응답에 반환되는 순서에 관계없이 `stale` 데이터가 `fresh` 한 데이터를
  덮어씌우지 않게 보장한다
  <br/>

- 렌더링된 가장 가까이 있는 `errorElement` 를 렌더링하여 포착되지 않은
  `error` 를 처리할수 있다

> `<Link>` 또는 `<Form>` 으로 부터 일반적인 페이지 탐색을 했을 경우와 같다

- 만약 `action` / `loader` 가 호출되어 `redirect` 를 반환한다면,
  `app` 은 `redirect` 될것이다

> `<Link>` 또는 `<Form>` 으로 부터 일반적인 페이지 탐색을 했을 경우와 같다

##### options

**_`key`_**

기본적으로, `useFetcher` 는 컴포넌트로 범위가 지정된 고유한 `fetcher` 를  
생성한다

> 진행중인 `useFetcher()` 에서 조회가능하다

만약 `fetcher` 를 식별하기를 원한다면 `key` 옵션을 사용하여 생성하고,
이를 사용하여 `app` 의 다른곳에서 접근할수 있다

```tsx
function AddToBagButton() {
  const fetcher = useFetcher({ key: "add-to-bag" });
  return <fetcher.Form method="post">...</fetcher.Form>;
}

// Then, up in the header...
function CartCount({ count }) {
  const fetcher = useFetcher({ key: "add-to-bag" });
  const inFlightCount = Number(fetcher.formData?.get("quantity") || 0);
  const optimisticCount = count + inFlightCount;
  return (
    <>
      <BagIcon />
      <span>{optimisticCount}</span>
    </>
  );
}
```

##### `Components`

**_`fetcher.Form`_**

이는 단지 페이지 이동을 하지 않는다는것만 제외하면 `<Form>` 과 같다

```tsx
function SomeComponent() {
  const fetcher = useFetcher();
  return (
    <fetcher.Form method="post" action="/some/route">
      <input type="text" />
    </fetcher.Form>
  );
}
```

##### `Method`

**_`fetcher.load(href, options)`_**

`route loader` 에서 `data` 를 `load` 한다

```tsx
import { useFetcher } from "react-router-dom";

function SomeComponent() {
  const fetcher = useFetcher();

  useEffect(() => {
    if (fetcher.state === "idle" && !fetcher.data) {
      fetcher.load("/some/route");
    }
  }, [fetcher]);

  return <div>{fetcher.data || "Loading..."}</div>;
}
```

`URL` 이 중첩된 여러 `route` 와 일치될수 있지만, `fetcher.loade` 는
가장 마지막에 일치된 `route` 의 `loader` 를 호출할것이다.

**_`options.unstable_flushSync`_**

`useFetcher` 훅을 사용하여 데이터를 로딩할때, 일반적으로 데이터 로딩이
완료된 후에 `UI` 업데이트가 발생한다
`UI` 업데이트는 가능한 빠르게 `UI` 를 그리지만, 여러 과정을 거치며
순차적으로 이루어진다. 즉, 사용자에게는 순간적으로 없데이트된 내용이
보이지 않을수 있다

이러한 상황이 발생할때, `UI` 업데이트를 즉각적으로 반영하기 위한 과정을
`동기적` 으로 처리한다는 이야기다

동기적으로 모든 처리를 하는것은 `javascript` 특성상 좋은 방법은 아니다
꼭 필요할때 사용하는것이 좋다

**_`fetcher.submit`_**

`<fetcher.Form>` 의 명령적인 버전이며, 사용자의 인터렉션에서 `fetch 초기화` 를
하는 경우, `<fetcher.Form>` 을 사용해야 한다

반면, 프로그래머가 `fetch` 하는 경우에는 `fetcher.submit` 기능을 사용해야 한다

> 이는 사용자가 `button` 을 클릭해는것에 대한 응답이 아닌경우에 해당한다

```tsx
import { useFetcher } from "react-router-dom";
import { useFakeUserIsIdle } from "./fake/hooks";

export function useIdleLogout() {
  const fetcher = useFetcher();
  const userIsIdle = useFakeUserIsIdle();

  useEffect(() => {
    if (userIsIdle) {
      fetcher.submit({ idle: true }, { method: "post", action: "/logout" });
    }
  }, [userIsIdle]);
}
```

`fetcher.submit` 은 `useSubmit` 의 래퍼이며,
`useSubmit` 과 같은 옵션을 가진다

만약에, `index route` 를 `submit` 하기 원한다면, `?index` `param` 을 사용
한다

만약에, `handler` 를 클릭하는 로직이라면, 간단하게 `<fetcerh.Fomr>` 을
사용하길 권장한다

##### `properties`

**_`fetcher.state`_**

`fetcher.state` 와 함께 `fercher` 의 상태를 알 수 있다
이는 다음중 하나이다

- **idle**
  아직 `fetch` 되지 않았다
  <br/>

- **submiting**
  `fetcher` 에서 `POST`, `PUT`, `PATCH`, `DELETE` 로 `submit` 을 통해
  `route action` 에서 호출되고 있다
  <br/>

- **loading**
  현재 `fetcher` 는 `loader` 를 호출하며, 이는 `useValidator` 를 호출하거나
  별도 `submission` 된후 `revalidation` 된다

**_`fetcher.data`_**

이는 `loader` 또는 `action` 에 저장된 데이터를 리턴한다
한번 `data` 가 설정되면, 이는 `reload` 나 `resubmission` 을 통해 `data` 가
변경 유지된다

```tsx
function ProductDetails({ product }) {
  const fetcher = useFetcher();

  return (
    <details
      onToggle={(event) => {
        if (
          event.currentTarget.open &&
          fetcher.state === "idle" &&
          !fetcher.data
        ) {
          fetcher.load(`/product/${product.id}/details`);
        }
      }}
    >
      <summary>{product.name}</summary>
      {fetcher.data ? (
        <div>{fetcher.data}</div>
      ) : (
        <div>Loading product details...</div>
      )}
    </details>
  );
}
```

**_`fetcher.formData`_**

`<fetcher.Form>` 또는 `fetcer.submit()` 을 사용할때, `form data` 는
`optimistic UI` 을 구축하는데 활용가능하다

```tsx
function TaskCheckbox({ task }) {
  let fetcher = useFetcher();

  // while data is in flight, use that to immediately render
  // the state you expect the task to be in when the form
  // submission completes, instead of waiting for the
  // network to respond. When the network responds, the
  // formData will no longer be available and the UI will
  // use the value in `task.status` from the revalidation
  let status = fetcher.formData?.get("status") || task.status;

  let isComplete = status === "complete";

  return (
    <fetcher.Form method="post">
      <button
        type="submit"
        name="status"
        value={isComplete ? "complete" : "incomplete"}
      >
        {isComplete ? "Mark Complete" : "Mark Incomplete"}
      </button>
    </fetcher.Form>
  );
}
```

**_`fetcher.json`_**

`fetcher.submit(data, { formEncType: "application/json" })` 을 사용하는 경우,
`fetcher.json` 을 통해 `submit` 된 `JSON` 을 사용할수 있다

**_`fetcher.text`_**

`fetcher.submit(data, { formEncType: "text/plain" })` 을 사용하는 경우,
`submit` 된 `text` 는 `fetcher.text` 를 통해 `text` 를 사용할수 있다

**_`fetcher.formAction`_**

`submit` 되는 `form` 의 `action url` 을 알려준다

```tsx
<fetcher.Form action="/mark-as-read" />;

// when the form is submitting
fetcher.formAction; // "mark-as-read"
```

**_`fetcher.formMethod`_**

`submit` 되는 `from` 의 `formMethod` 를 알려준다

```tsx
<fetcher.Form method="post" />;

// when the form is submitting
fetcher.formMethod; // "post"
```

---

`useFetchers` 는 `fetcher` 의 `list` 이며, 사용할 `fetcher` 들을
`filtering` 하여 원하는 `fetcher` 를 일괄적으로 처리한다.

### `fetcherKey`

페이지 이동을 하지 않는 `Form` 을 사용할때, `<Form navigate={false} fetcherKey="my-key">` 를 통해 `fetcher key` 를 선택적으로 지정할수 있다

### `replace`

새로운 항목을 `history stack` 에 `push` 하는대신, 현재 항목을 바꾸도록
지시한다

```tsx
<Form replace />
```

`defult` 동작은 `form` 동작에에 따라 달라진다.

- `method=get` `form` 은 기본값으로 `false` 이다
  <br/>

- `submit` 메서드는 `action` 과 `formAction` 의 동작에 따라 달라진다 <br/>
  ▶️ 만약 `action` 에서 에러가 발생한다면, `default` 는 `false` 이다 <br/>
  ▶️ 만약 `action` 에서 현재 `location` 으로 `redirect` 된다면 `default` 는 `ture` 이다<br/>
  ▶️ 만약 `action` 에서 다른곳으로 `redirect` 된다면 `default` 는 `false` 이다<br/>
  ▶️ 만약 `formAction` 이 현재 `location` 이라면, `default` 는 `true` 이다<br/>
  ▶️ 다른 경우에, `default` 는 `false` 이다
  <br/>

### `relative`

`default` 에 의해, `path` 는 `route` 계층으로 부터 `relative` 하다.
그래서 `..` 는 `Route` 계층에서 위의 계층으로 올라간다

때때로, 중첩하기에 적합하지 않아서 `URL` 패턴에 매치되는 곳을 찾으려고하면
경로 라우팅을 사용하는것을 선호할수 있다

`<Form to="../some/where" relative="path">` 를 함께 사용하여 이 동작을
선택할수 있다

### `reloadDocument`

`React Router` 를 건너띄고 브라우저에 내장된 동작으로 `form` 을 `submit`
하도록 지시한다

```tsx
<Form reloadDocument />
```

이는 일반 `HTML form` 과 동일하지만 연관된 `action` 과 기본 동작의 이점을
얻기 위해 일반 `HTML <form>` 이 아닌 `reloadDocument` 를 추천한다

### `preventScrollReset`

`<ScrollRestoration>` 을 사용한다면, `action` 에 의해 새로운 `location` 으로
`redirect` 될때, `scroll` 위치가 `window` 의 상단으로 `reset` 되는것을
`prevent` 한다

```tsx
<Form method="post" preventScrollReset={true} />
```

### `unstable_viewTransition`

`unstable_viewTransition` `prop` 은 `document.startViewTransition()` 에서
최종 상태 없데이트를 래핑하여, 이 페이지 이동에 대한 `View Transition` 을
활성화 한다

만약, `view transition` 을 위한 특정 `styles` 을 적용해야 하는경우
`unstalbe_useViewTransition()` 도 같이 활용해야 한다

---

## `<Link>`

타입은 다음과 같다

```tsx
declare function Link(props: LinkProps): React.ReactElement;

interface LinkProps
  extends Omit<React.AnchorHTMLAttributes<HTMLAnchorElement>, "href"> {
  to: To;
  preventScrollReset?: boolean;
  relative?: "route" | "path";
  reloadDocument?: boolean;
  replace?: boolean;
  state?: any;
  unstable_viewTransition?: boolean;
}

type To = string | Partial<Path>;

interface Path {
  pathname: string;
  search: string;
  hash: string;
}
```

`<Link>` 는 사용자가 `tappint` 혹은 `click` 에 의해 다른페이지로 이동하는
`element` 이다

`react-router-dom` 에서, `<Link>` 는 `resource` 지점을 가리키는 실제
`href` 와 함께 엑세스 가능한 `<a>` 요소를 렌더링한다

이는 `<Link>` 에서 오른쪽 클릭같은것과 같은 작업이 예상대로 동작한다는 것을 의미한다

`<Link reloadDocument>` 를 사용하여 `client` 쪽 라우팅을 건너띌수 있고,
브라우저가 일반적인 전환을 처리한다 (마치 `<a href>` 처럼동작한다)

```tsx
import * as React from "react";
import { Link } from "react-router-dom";

function UsersIndexPage({ users }) {
  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            <Link to={user.id}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

`<Link to>` 의 `value` 는 `relative` 하며, 이는 `parent route` 기준으로
한다

이는 해당 `<Link` 가 랜더링한 경로와 일치하는 `URL` 경로를 기반으로
만들어진다는 의미이다

### `relative`

기본값으로, `relative="route"` 이며 이는 `route` 계층 구조를 기반으로
상대경로를 설정한다

이는 `route` 계층 구조 기반이 아닌, `relative="path"` 로 `URL` 기반으로
설정할 수 있다

```tsx
// Contact and EditContact do not share additional UI layout
<Route path="/" element={<Layout />}>
  <Route path="contacts/:id" element={<Contact />} />
  <Route path="contacts/:id/edit" element={<EditContact />} />
</Route>;

function EditContact() {
  // Since Contact is not a parent of EditContact we need to go up one level
  // in the current contextual route path, instead of one level in the Route
  // hierarchy
  //
  return (
    <Link to=".." relative="path">
      Cancel
    </Link>
  );
}
```

### `preventScrollReset`

`<ScrollRestoration>` 을 사용한다면, `link` 클릭시 `scroll` 위치가 `window`
의 상단으로 초기화 되는것을 방지할수 있다

```tsx
<Link to="?tab=one" preventScrollReset={true} />
```

`user` 가 `back` / `forward` 버튼을 사용하여 해당 위치로 돌아올때,
`scroll` 위치가 복원되는것을 방지하지 않으며, 사용자가 링크를 클릭할 때
초기화되는 것을 방지할 뿐이다

이 행동을 원하는 예로써, 이는 페이지 목록이 없는 `url searche params` 를
조작하는 `tabs` 목록이 있다

`tabs` 목록은 `scroll` 위치가 `window` 의 `top` 으로 올라가는것을 원치 않을 것이다. 왜냐하면, 토글된 컨텐츠를 `viewport` 바깥으로 스크롤할수 있기 때문이다

```sh
      ┌─────────────────────────┐
      │                         ├──┐
      │                         │  │
      │                         │  │ scrolled
      │                         │  │ out of view
      │                         │  │
      │                         │ ◄┘
    ┌─┴─────────────────────────┴─┐
    │                             ├─┐
    │                             │ │ viewport
    │   ┌─────────────────────┐   │ │
    │   │  tab   tab   tab    │   │ │
    │   ├─────────────────────┤   │ │
    │   │                     │   │ │
    │   │                     │   │ │
    │   │ content             │   │ │
    │   │                     │   │ │
    │   │                     │   │ │
    │   └─────────────────────┘   │ │
    │                             │◄┘
    └─────────────────────────────┘

```

### `replace`

`replace` 프로퍼티는 `default` 로 사용하는 `history.pushState` 가 아닌
`history.replaceState` 를 통해 `history.stack` 안의 현재 항목을
교체하는것 같이 동작한다

> `like to replace the current entry in the history stack` 이 뭔가 걸린다
> **`history stack` 에서 현재 항목을 교체하는것 같이** 로 해석되는데,
> 다른방식을 사용할수도 있나 싶다..

### `state`

이 `state` 프로퍼티를 사용하여 `history state` 에 저장되는 새 `location` 에
대한 상태 저장값을 설정할 수 있다

이 값은 나중에 `useLocation()` 을 통해 접근할수 있다

```tsx
<Link to="new-path" state={{ some: "value" }} />
```

이 상태 값은 `new-path` 라우터에서 접근한것이다

```tsx
let { state } = useLocation();
```

### `reloadDocument`

이 `reloadDocument` 프로퍼티을 사용하여 클라이언트 쪽 라우팅을 건너뛸수 있다
그리고, 브라우저가 전환을 정상적으로 처리하도록 한다 (마치 `a href` 인 것처럼)

### `unstable_viewTransition`

`unstable_viewTransition` `prop` 은 `document.startViewTransition()` 에서
최종 상태 업데이트를 래핑하여 페이지 전환시에 `View Transition` 을
활성화한다

```tsx
<Link to={to} unstable_viewTransition>
  Click me
</Link>
```

이 `view transition` 에 대한 특정 `style` 을 적용해야 하는 경우,
`unstable_useViewTransitionState()` `hook` 도 활용해야 한다

> 이 `API` 는 `unstable` 표시가 되어있으므로, 변경사항이 적용될수 있다한다

---

## `<NavLink>`

`<NavLink>` 는 `active`, `pending` 또는 `transitioning` 인지 여부를 아는
특별한 종류의 `<Link>` 이다.

이는 몇가지 시나리오에 유용하다

- 네비게이션 메뉴를 만들때,
  예를 들어 현재 선택된 항목을 표시하려는 탐색경로 또는 탭 세트
  <br/>

- `screen reader` 같은 보조 기술을 위한 유용한 컨텍스트를 제공한다
  <br/>

- `View Transition` 을 더 세밀하게 제어할 수 있도록 `transitioning` 값을
  제공한다

```tsx
import { NavLink } from "react-router-dom";

<NavLink
  to="/messages"
  className={({ isActive, isPending }) =>
    isPending ? "pending" : isActive ? "active" : ""
  }
>
  Messages
</NavLink>;
```

### Default `active` class

기본적으로, `active` 클래스는 `<NavLink>` 컴포넌트에가 `active` 되었을때
해당 컴포넌트에 추가되므로 사용할 `css` 스타일을 설정할 수 있다

```tsx
<nav id="sidebar">
  <NavLink to="/messages" />
</nav>
```

```css
#sidebar a.active {
  color: red;
}
```

### `className`

`className` `prop` 은 일반적인 `className` 같이 동작한다. 그러나
링크의 `active` 와 `pending` 상태에 따라 적용된 `className` 을 사용자 지정하는 함수에 전달할수도 있다

```tsx
<NavLink
  to="/messages"
  className={({ isActive, isPending, isTransitioning }) =>
    [
      isPending ? "pending" : "",
      isActive ? "active" : "",
      isTransitioning ? "transitioning" : "",
    ].join(" ")
  }
>
  Messages
</NavLink>
```

### `children`

`pending` 상태 와 `active` 상태를 기반으로, `<NavLink>` 의 콘텐츠를 사용자
정의하기 위해 `children` 을 전달하여 렌더링 가능하다 이는 내부 요소들의
스타일을 변경하는데 유용하다

```tsx
<NavLink to="/tasks">
  {({ isActive, isPending, isTransitioning }) => (
    <span className={isActive ? "active" : ""}>Tasks</span>
  )}
</NavLink>
```

### `end`

이 `end` `prop` 은 오직 `<NavLink to>` 의 `path` 에서 `end` 와 일치하도록
`active` 및 `pending` 상태에 대한 `match logic` 을 변경한다

만약, 이 `URL` 이 `to` 보다 길다면, 더이상 활성상태로 간주되지 않는다

| Link                           | CurrentURL   | isActive |
| :----------------------------- | :----------- | :------- |
| `<NavLink to="/tasks" />`      | `/tesks`     | true     |
| `<NavLink to="/tasks" />`      | `/tesks/123` | false    |
| `<NavLink to="/tasks" end />`  | `/tesks`     | true     |
| `<NavLink to="/tasks" end />`  | `/tesks/123` | false    |
| `<NavLink to="/tasks/" end />` | `/tesks`     | false    |
| `<NavLink to="/tasks/" end />` | `/tesks/`    | true     |

**_A note on links to the root route_**

`<NavLink to="/">` 는 예외적 케이스이다. 왜냐하면 모든 `URL` 과 매치되기
때문이다. 기본적으로 모든 단일 경로와 일치하는 것을 피하기 위해 `end`
`prop` 을 효과적으로 무시하고 루트 경로에 있을때만 일치하다

### `caseSensitive`

`caseSensitive` `prop` 를 추가하면, `matching logic` 을 대소문자 구분하게
만든다

| Link                                         | URL           | isActive |
| :------------------------------------------- | :------------ | :------- |
| `<NavLink to="/SpOnGe-bOB" />`               | `/sponge-bob` | true     |
| `<NavLink to="/SpOnGe-bOB" caseSensitive />` | `/sponge-bob` | false    |

### `aria-current`

`NavLink` 가 `active` 일때, 자동적으로 `<a aria-current="page" >` 가
적용된다.

### `reloadDocument`

`reloadDocument` 프로퍼티는 `client side routing` 을 건너띄길때 사용되며,
브라우저가 `transition` 을 정상적으로 처리하도록 한다 (마치 `a href` 인것 처럼)

### `unstable_viewTransition`

이는 `Link` 의 `unstalbe_viewTransition` 과 같다

---

## `<Navigate>`

타입은 다음과 같다

```ts
declare function Navigate(props: NavigateProps): null;

interface NavigateProps {
  to: To;
  replace?: boolean;
  state?: any;
  relative?: RelativeRoutingType;
}
```

`<Navigate>` 엘리먼트는 렌더링될때, 현재 `location` 을 변경한다
이는 `useNavigate` 를 래핑한 컴포넌트이며, `prop` 에서 같은 인자값을
받는다

이에 대해서 `useNavigate` 를 보도록 하자

---

#### `useNavigate`

일반적으로 `loader` 와 `action` 의 `redirect` 를 사용하는것이 더 좋다

타입은 다음과 같다

```ts
declare function useNavigate(): NavigateFunction;

interface NavigateFunction {
  (to: To, options?: NavigateOptions): void;
  (delta: number): void;
}

interface NavigateOptions {
  replace?: boolean;
  state?: any;
  preventScrollReset?: boolean;
  relative?: RelativeRoutingType;
  unstable_flushSync?: boolean;
  unstable_viewTransition?: boolean;
}

type RelativeRoutingType = "route" | "path";
```

`useNavigate` `hook` 은 함수를 리턴하며, 프로그래밍 방식으로 페이지 이동
할수 있다

```tsx
import { useNavigate } from "react-router-dom";

function useLogoutTimer() {
  const userIsInactive = useFakeInactiveUser();
  const navigate = useNavigate();

  useEffect(() => {
    if (userIsInactive) {
      fake.logout();
      navigate("/session-timed-out");
    }
  }, [userIsInactive]);
}
```

`navigate` 함수는 두개의 `signature` 가 있다

- 선택적 두번째 옵션 인수(`Link` 에 전달할 수 있는 `prop` 과 같은)를
  사용하여 `To` 값(`Link to` 와 동일한 유형)을 전달하거나
  <br/>

- `history stack` 에 들어가고 싶은 `delta` 를 전달한다
  예를 들어 `navigate(-1)` 은 `back` 버튼을 누른것과 같다

#### `options.replace`

`replace: true` 는 페이지 이동시 새로운 항목을 삽입하는 대신 `history stack`
현재항목으로 교체한다

#### `options.state`

`history state` 안에 저장하는 선택적 `state` 값을 포함할수 있다
이는 `route` 목적지에서 `useLocation` 을 통해 접근할수 있다

```tsx
navigate("/new-route", { state: { key: "value" } });
```

#### `options.preventScrollReset`

`<ScrollRestoration>` 컴포넌트를 사용할때, `option.preventScrollReset` 을
통해 스크롤이 페이지 상단으로 재설정되는것을 비활성화 할 수 있다

#### `options.relative`

기본값으로 `navigation` 은 `route` 계층 구조를 기준으로 한다

> (`relative: "route"`) 와 같다

그래서, '..' 는 `route` 계층에서 하나 위로 올라간다.
때때로, 중첩된것으로 처리되지 못하는 상황이 생겨, `URL` 패턴에 매칭되는것을
찾고 싶다면 `상대 경로` 를 기준으로 하길 원할수 있다

`relative: "path"` 를 사용하여 이러한 동작을 선택할수 있다

```tsx
// Contact and EditContact do not share additional UI layout
<Route path="/" element={<Layout />}>
  <Route path="contacts/:id" element={<Contact />} />
  <Route path="contacts/:id/edit" element={<EditContact />} />
</Route>;

function EditContact() {
  // Since Contact is not a parent of EditContact we need to go up one level
  // in the path, instead of one level in the Route hierarchy
  navigate("..", { relative: "path" });
}
```

#### `options.unstable_flushSync`

`unstalbe_flushSync` 의 `flush` 는 모든 렌더링 작업을 완료하고 업데이트된 `DOM` 을 브라우저에 반영하는것을 의미한다

이 옵션은 기본적으로 비동기로 실행되는 `react` 업데이트를 동기적으로
실행하도록 강제한다

이는 탐색후 발생하는 초기 상태 업데이트를 `ReactDOM.flushSync` 함수를
사용하여 동기적으로 처리한다

이를 통해 탐색후 `DOM` 요소에 접근하고 조작하는 등 동기적인 작업을
수행할 수 있다

기본 `React.startTransition` 대신 `ReactDOM.flushSync` 호출에서
페이지 이동에 대한 초기 상태 업데이트를 래핑하도록 `React Route DOM` 에
지시한다

---

## `<Outlet>`

타입은 다음과 같다

```tsx
interface OutletProps {
  context?: unknown;
}
declare function Outlet(props: OutletProps): React.ReactElement | null;
```

`<Outlet>` 은 `child route elements` 를 렌더링하기 위해
`parent route elements` 안에 사용된다

이는 `child routes` 를 렌더링될때 중첩된 `UI` 가 표시될수 있다
만약, `parent route` 가 정확하게 `child route` 의 경로와 매칭된다면,
`child index route` 가 렌더링 되고, `index route` 가 없다면 아무것도
렌더링되지 않는다

> `index route` 는 색인된 경로를 말하며, 이는 `child route elements` 를
> 뜻하는듯 하다.
>
> 이 예시에서, 정확히 일치되는 `child` 경로를 가진다면,
> 해당 `child` 경로를 렌더링하지만, 그렇지 않고 `parent` 경로를 렌더링한다면
> `null` 을 렌더링한다는 이야기다.

```tsx
function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* This element will render either <DashboardMessages> when the URL is
          "/messages", <DashboardTasks> at "/tasks", or null if it is "/"

          이 `element` 는 "/messages" 일때 <DashboardMessages> 를,
          "/tasks" 이면 <DashboardTasks> 또는 "/" 이면 null 중 하나를 렌더링한다
      */}
      <Outlet />
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Dashboard />}>
        <Route
          path="messages"
          element={<DashboardMessages />}
        />
        <Route path="tasks" element={<DashboardTasks />} />
      </Route>
    </Routes>
  );
```

---

## `<Routes>`

이는 컴포넌트를 중첩구조로 `route` 를 선언하여 코드를 더 읽기 쉽고
관리하기 쉽게 만들 수 있다.

`location` 이 변경될때마다, `<Routes>` 는 모든 `child routes` 를 살펴보고
가장 일치하는 `child routes` 를 렌더링한다.

```tsx
<Routes>
  <Route path="/" element={<Dashboard />}>
    <Route path="messages" element={<DashboardMessages />} />
    <Route path="tasks" element={<DashboardTasks />} />
  </Route>
  <Route path="about" element={<AboutPage />} />
</Routes>
```

---

## `<ScrollRestoration />`

이 `component` 는 `loader` 가 완료된 이후 위치 변경시, 스크롤을 복원을
에뮬레이트하여 도메인 전체에서 스크롤의 위치가 원하는 위치로 복원되도록
한다

```tsx
function RootRouteComponent() {
  return (
    <div>
      {/* ... */}
      <ScrollRestoration />
    </div>
  );
}
```

### `getKey`

이 선택적 `prop` 은 `scroll` 위치를 복원하기 위해 사용되는
`React Router` 의 `key` 를 정의한다

> 📎 **여기서 부가적인 설명이 들어가야한다.**
>
> `location.key` 는 `React Router` 가 **경로가 변할때 마다 새로 생성되는
> 고유한 식별자** 이다.
>
> 이 말인 즉슨, `history stack` 상에 같은 경로가 2번 `push` 되도라도
> 그 경로마다 다른 `location.key` 가 생성된다는 것이다
>
> 이 내용을 모르고 보면 뒤의 내용이 약간 애매해질수 있다..

```tsx
<ScrollRestoration
  getKey={(location, matches) => {
    // default behavior
    return location.key;
  }}
/>
```

`default` 로 `location.key` 를 사용하며, `client side routing` 없이
브라우저 기본 동작을 에뮬레이팅한다

사용자는 `stack` 에서 같은 `URL` 을 여러번 페이지이동할수 있으며,
`stack` 의 각 항목은 복원할 `scroll` 위치를 얻는다

일부 앱에서는 이 동작을 재정의하고 다른 것을 기반으로 위치를 복원할 수
있다. 4개의 기본 페이지가 있는 소셜 앱을 생각해보자

- "/home"
- "/message"
- "/notifications"
- "/search"

사용자가 `"/home"` 에서 `scroll` 을 약간 아래로 내리고, 메뉴상의
`"/messages"` 를 클릭한다. 그리고 메뉴상의 `"/home"` 버튼을 다시 누른다

> ⚠️ **`back` 버튼을 누른것이 아니다.**

이는 `history stack` 안에 3개의 항목이 생길것이다

`history stack`

```sh
1. /home
2. /messages
3. /home
```

기본적으로, `React Router` 는 같은 `URL` 일지라도 `1` 과 `3` 에서 두개의 다른
`scroll` 위치를 저장한다

이는 사용자가 `2` `->` `3` 에서 페이지이동하면 `1` 에서 스크롤위치를 복원하는 대신에 최상단으로 이동하게 된다

여기서 더 좋은 `UX` 를 제공하기 위해서는, 어디에서 어떻게 오든 상관없이
(`button` 클릭 혹은 `back` 버튼을 누르는것 같은..) `scroll` 위치는
유지되어야 한다

이를 위해서 `location.pathname` 같은 `key` 를 사용해야 한다

```tsx
<ScrollRestoration
  getKey={(location, matches) => {
    return location.pathname;
  }}
/>
```

또는 일부 `path` 에 대한 `pathname` 만을 사용하기 원하고, 다른 모든 경우
에는 일반적인 동작을 하기 원할수 있다.

```tsx
<ScrollRestoration
  getKey={(location, matches) => {
    const paths = ["/home", "/notifications"];
    return paths.includes(location.pathname)
      ? // home and notifications restore by pathname
        // `pathname` 을 `home` 과 `notification` 만 복원기능을 사용한다
        location.pathname
      : // everything else by location like the browser
        location.key;
  }}
/>
```

### `Preventing Scroll Reset`

새로운 페이지 이동 `scroll key` 를 생성할때, 이 `scroll` 위치는 페이지의
최상단으로 재설정된다.

이러한 동작을 다음으로 막을수 있다

```tsx
<Link preventScrollReset={true} />
<Form preventScrollReset={true} />
```

더 간단하게 말하면, 페이지 이동시 `scroll` 이 `reset` 되는것을 막으며,
해당 이동한 스크롤만큼 유지하게 해준다

### `Scroll Flashing`

`server side rendering` 을 하지 않는다면, `page` 를 처음 읽어올때
`scroll flashing` 현상이 발생할수 있다.

이는 `React Router` 가 `data` 를 읽어오고, `JS` 번들을 다운로드하고,
`page` 전체를 렌더링 하는동안 `scroll` 위치를 복원할수 없기 때문이다

`Server Rendering` 프레임워크는 초기 로드시 완전히 구성된 문서를
보내주기 때문에 이러한 `scroll flashing` 을 막을수 있다

그래서 `page` 첫 렌더링시 `scroll` 은 복원된다
