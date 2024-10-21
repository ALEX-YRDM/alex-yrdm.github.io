@[toc]
[以下内容来自官方文档 https://reactrouter.com/en/main](https://reactrouter.com/en/main)
## 基本格式
```js
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

---

<a name="YAX0k"></a>
### [createBrowserRouter](https://reactrouter.com/en/main/routers/create-browser-router)
```js
import * as React from "react";
import * as ReactDOM from "react-dom";
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";

import Root, { rootLoader } from "./routes/root";
import Team, { teamLoader } from "./routes/team";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    loader: rootLoader,
    children: [
      {
        path: "team",
        element: <Team />,
        loader: teamLoader,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```
<a name="a5h7R"></a>
#### Type Declaration
```js
function createBrowserRouter(
  routes: RouteObject[],
  opts?: {
    basename?: string;
    future?: FutureConfig;
    hydrationData?: HydrationState;
    window?: Window;
  }
): RemixRouter;
```
<a name="cjix0"></a>
##### routes
An array of [Route](https://reactrouter.com/en/main/route/route) objects with nested routes on the children property.

```js
createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    loader: rootLoader,
    children: [
      {
        path: "events/:id",
        element: <Event />,
        loader: eventLoader,
      },
    ],
  },
]);
```
Route定义为:
```js
const router = createBrowserRouter([
  {
    // it renders this element
    element: <Team />,

    // when the URL matches this segment
    path: "teams/:teamId",

    // with this data loaded before rendering
    loader: async ({ request, params }) => {
      return fetch(
        `/fake/api/teams/${params.teamId}.json`,
        { signal: request.signal }
      );
    },

    // performing this mutation when data is submitted to it
    action: async ({ request }) => {
      return updateFakeTeam(await request.formData());
    },

    // and renders this element in case something went wrong
    errorElement: <ErrorBoundary />,
  },
]);
```
RouteObject对象的声明为:
```js
interface RouteObject {
  path?: string;
  index?: boolean;
  children?: React.ReactNode;
  caseSensitive?: boolean;
  id?: string;
  loader?: LoaderFunction;
  action?: ActionFunction;
  element?: React.ReactNode | null;
  Component?: React.ComponentType | null;
  errorElement?: React.ReactNode | null;
  ErrorBoundary?: React.ComponentType | null;
  handle?: RouteObject["handle"];
  shouldRevalidate?: ShouldRevalidateFunction;
  lazy?: LazyRouteFunction<RouteObject>;
}
```
对于Path:<br />动态路由参数, 可以在loader, action中通过params直接点出 或者使用useParams()钩子来获取该参数
```js
<Route
  // this path will match URLs like
  // - /teams/hotspur
  // - /teams/real
  path="/teams/:teamId"
  // the matching param will be available to the loader
  loader={({ params }) => {
    console.log(params.teamId); // "hotspur"
  }}
  // and the action
  action={({ params }) => {}}
  element={<Team />}
/>;

// and the element through `useParams`
function Team() {
  let params = useParams();
  console.log(params.teamId); // "hotspur"
}
```
你可以定义多个动态参数 都可以通过 . 运算符获取到
```js
<Route path="/c/:categoryId/p/:productId" />;
// both will be available
params.categoryId;
params.productId;
```
可选参数 通过在后面加上 ? 
```js
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
当然可以同时用动态参数和可选参数

模糊匹配所有的 * 参数
```js
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
布局路由
```js
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
<Outlet/> 组件留出子路由要渲染的位置,仅此而已<br />` index `索引路由  url为父路径时显示的组件
```jsx
<Route path="/teams" element={<Teams />}>
  <Route index element={<TeamsIndex />} />
  <Route path=":teamId" element={<Team />} />
</Route>
```

- 外层的`<Route>`有一个路径`path`为"/teams"，并渲染了一个组件`<Teams />`。
- 在这个路由内部，有另一个带有`index`属性的嵌套的`<Route>`。这意味着当路径匹配"/teams"时，组件`<TeamsIndex />`将被呈现到其父级路由的`Outlet`中（在这种情况下是`<Teams />`）。
- 此外，还有另一个带有`path=":teamId"`的嵌套路由，表示当路径匹配"/teams/:teamId"时，将呈现组件`<Team />`。

children: 嵌套路由<br />caseSensitive: 表示对路径中大小写敏感<br />Instructs the route to match case or not:

```js
<Route caseSensitive path="/wEll-aCtuA11y" />
```

- Will match "wEll-aCtuA11y"
- Will not match "well-actua11y"

loader<br />每一个路由都能定义一个loader函数来为路由组件渲染之前提供数据
```js
createBrowserRouter([
  {
    element: <Teams />,
    path: "teams",
    loader: async () => {
      return fakeDb.from("teams").select("*");
    },
    children: [
      {
        element: <Team />,
        path: ":teamId",
        loader: async ({ params }) => {
          return fetch(`/api/teams/${params.teamId}.json`);
        },
      },
    ],
  },
]);
```
通过 调用loader函数获得的数据可以通过 useLoaderData钩子拿到<br />`useLoaderData`: This hook provides the value returned from your route loader.
```js
import {
  createBrowserRouter,
  RouterProvider,
  useLoaderData,
} from "react-router-dom";

function loader() {
  return fetchFakeAlbums();
}

export function Albums() {
  const albums = useLoaderData();
  // ...
}

const router = createBrowserRouter([
  {
    path: "/",
    loader: loader,
    element: <Albums />,
  },
]);

ReactDOM.createRoot(el).render(
  <RouterProvider router={router} />
);
```
在调用路由操作之后，数据将自动重新验证，并从加载器返回最新的结果。<br />请注意，`useLoaderData` 不会启动抓取。它只是读取 React Router 在内部管理的抓取结果，因此您不必担心在重新呈现的原因之外的情况下重新获取数据。<br />这也意味着在重新呈现之间返回的数据是稳定的，因此您可以安全地将其传递给 React hooks（如 useEffect）中的依赖数组。它只在再次调用加载器后（在执行操作或特定导航后）才会更改。在这些情况下，标识将更改（即使值没有更改）。<br />您可以在任何组件或任何自定义 hook 中使用此钩子，而不仅仅是在 Route 元素中。它将返回上下文中最近路由的数据。<br />`params`: 由url中解析的动态参数会放在params中并传递给loader函数
```js
createBrowserRouter([
  {
    path: "/teams/:teamId",
    loader: ({ params }) => {
      return fakeGetTeam(params.teamId);
    },
  },
]);
```
`request`: 
```js
<a
  href={props.to}
  onClick={(event) => {
    event.preventDefault();
    navigate(props.to);
  }}
/>
```
React router阻止浏览器将请求request发送给server, 而是转给了loader函数
```js
function loader({ request }) {
  const url = new URL(request.url);
  const searchTerm = url.searchParams.get("q");
  return searchProducts(searchTerm);
}
```
返回 response<br />可以返回fetch()发送异步请求(ajax, axios)的结果,也可以自定义返回结果
```js
// an HTTP/REST API
function loader({ request }) {
  return fetch("/api/teams.json", {
    signal: request.signal,
  });
}

// or even a graphql endpoint
function loader({ request, params }) {
  return fetch("/_gql", {
    signal: request.signal,
    method: "post",
    body: JSON.stringify({
      query: gql`...`,
      params: params,
    }),
  });
}

function loader({ request, params }) {
  const data = { some: "thing" };
  return new Response(JSON.stringify(data), {
    status: 200,
    headers: {
      "Content-Type": "application/json; utf-8",
    },
  });
}
```
react router会自动调用 response.json()
```js
function SomeRoute() {
  const data = useLoaderData();
  // { some: "thing" }
}
```
在loader中抛出异常
```js
function loader({ request, params }) {
  const res = await fetch(`/api/properties/${params.id}`);
  if (res.status === 404) {
    throw new Response("Not Found", { status: 404 });
  }
  return res.json();
}
```
会渲染路由中定义的 errorElement组件<br />`action`: 进行写操作的函数
```js
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
 在应用发起一个非get请求时被调用  比如按钮点击
```js
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
element / Component: 要渲染的组件 <br />errorElement / ErrorBoundary: 异常时渲染的组件<br />当在loader, action函数执行或者组件渲染时抛出异常, 就不会渲染element组件, 而是用error path的组件 errorElement渲染, 该异常可以用 useRouteError获取<br />lazy : 
```js
let routes = createRoutesFromElements(
  <Route path="/" element={<Layout />}>
    <Route path="a" lazy={() => import("./a")} />
    <Route path="b" lazy={() => import("./b")} />
  </Route>
);
```
<a name="O1aOz"></a>
##### basename
The basename of the app for situations where you can't deploy to the root of the domain, but a sub directory.

```js
createBrowserRouter(routes, {
  basename: "/app",
});
```
<a name="dJL6E"></a>
##### future and window ... 省略
<a name="I6e2W"></a>
### <RouterProvider>
All [data router](https://reactrouter.com/en/main/routers/picking-a-router) objects are passed to this component to render your app and enable the rest of the data APIs
```js
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        path: "dashboard",
        element: <Dashboard />,
      },
      {
        path: "about",
        element: <About />,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider
    router={router}
    fallbackElement={<BigSpinner />}
  />
);
```
<a name="LCCQ6"></a>
#### fall上述文本提到 `createBrowserRouter` 函数的一个特性：在非服务器端渲染应用程序时，当它挂载时会初始化所有匹配的路由加载器。在此期间，你可以提供 `fallbackElement`，以向用户显示应用程序正在加载的指示。

这个特性的目的是在加载路由数据时，提供一个加载过程中的占位元素（fallback element），使用户在等待加载完成时能够看到一些界面反馈，而不是一片空白。<br />举例来说，你可以在 `createBrowserRouter` 中添加 `fallbackElement` 选项，像这样：
```javascript
createBrowserRouter(routes, {
  fallbackElement: <div>Loading...</div>,
});
```
这样，当路由加载的过程中，页面将显示 "Loading..."，以告诉用户应用程序正在处理数据，避免用户在加载过程中感到不确定或无反馈。
<a name="fRq1H"></a>
#### future ...
<a name="yPdAM"></a>
### Components
<a name="pAT0O"></a>
#### <Await>
```js
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
          errorElement={
            <div>Could not load reviews 😬</div>
          }
          children={(resolvedReviews) => (
            <Reviews items={resolvedReviews} />
          )}
        />
      </React.Suspense>
    </div>
  );
}
```
这段代码是一个使用 React Router 的 `useLoaderData` 和 `Await` 组件的 React 组件。让我解释一下它的主要部分：

1.  `useLoaderData` 是 React Router 提供的一个 hook，用于获取路由加载器（loader）返回的数据。在这里，通过 `useLoaderData` 获取了 `book` 和 `reviews`。 
2.  在 `return` 中，首先渲染了 `book` 的标题和描述。 
3.  使用了 React 的 `Suspense` 组件，它接受 `fallback` 属性，指定在子组件加载过程中显示的占位元素。在这里，如果 `reviews` 尚未加载完成，将显示一个 `ReviewsSkeleton` 组件，用作加载占位符。 
4.  使用 `Await` 组件包装了 `reviews` 的加载。`Await` 组件接受 `resolve` 属性，指定需要等待的 Promise。如果 `resolve` Promise 处于 pending 状态，它会渲染 `fallback` 中的占位元素；如果 Promise 处于 resolved 状态，它会调用 `children` 函数，并将 resolved 的数据传递给它。 
5.  在 `children` 函数中，渲染了一个 `Reviews` 组件，传递了从 `reviews` 获取的 resolved 数据。 

总体来说，这段代码的目的是展示书籍的标题和描述，同时以异步方式加载书籍的评论。在加载评论的过程中，会显示一个加载中的占位符，确保用户在等待数据加载完成时能够看到一些界面反馈。如果加载评论失败，将显示一个错误提示。