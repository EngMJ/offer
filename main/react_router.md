# API 参考

React Router 是 [React components](https://reactjs.org/docs/components-and-props.html)、[hooks](https://reactjs.org/docs/hooks-intro.html) 和一些其他实用程序的集合，可搭配 [React](https://reactjs.org) 轻松构建多页面应用程序，此参考包含 React Router 中各种接口（interfaces）的函数签名和返回类型。

## 概述

### 依赖包

React Router 在 npm 发布三个不同的包：

- [`react-router`](https://npm.im/react-router) 包含 React Router 的大部分核心功能，包括路由匹配算法以及大部分核心组件和hooks
- [`react-router-dom`](https://npm.im/react-router-dom) 包括 `react-router` 的所有内容，并添加了一些特定于 DOM 的 API，包括 [`<BrowserRouter>`](#browserrouter)，[`<HashRouter>`](#hashrouter) 和 [`<Link>`](#link)
- [`react-router-native`](https://npm.im/react-router-native) 包括 `react-router` 的所有内容，并添加了一些特定于 React Native 的 API，包括 [`<NativeRouter>`](#nativerouter) 和 [`<Link>` 的原生版本](#link-react-native)

`react-router-dom` 和 `react-router-native` 在安装时都会自动包含 `react-router` 作为依赖，并且都从 `react-router` 重新 export 所有内容。当 import 时，总是 import from `react-router-dom` 或 `react-router-native` 而非直接 import from `react-router`，否则可能会意外在应用中 import 不匹配版本的库（library）。

如果[安装](./getting-started/installation.md) React Router 以在全局使用（使用 `<script>` 标签），可以在 `window.ReactRouterDOM` 对象上找到该库。如果从 npm 安装，则可以 import 需要的部分。本参考中的示例均使用 `import` 语法。

### 构建

为了让 React Router 在应用中工作，需要在 element tree 的根部或附近渲染（render）一个 router 元素。 依据应用运行的位置，以下提供几种不同的routers：

- 在 Web 浏览器中运行时使用 [`<BrowserRouter>`](#browserrouter) 或 [`<HashRouter>`](#hashrouter)（根据个人偏好或需要的 URL 样式选择）
- 在服务器端渲染（server-rendering）网站时使用 [`<StaticRouter>`](#staticrouter)
- 在 [React Native](https://reactnative.dev/) 应用中使用 [`<NativeRouter>`](#nativerouter)
- [`<MemoryRouter>`](#memoryrouter) 在测试场景中较实用，也可以作为其他routers的参考实现

这些 routers 提供了 React Router 在特定环境中运行所需的条件。每个 router 都在内部渲染[一个 `<Router>`](#router)，如需更细粒度的控制也可以这样做。但大概率只需使用上述内置 routers 中的一个。

### 路由

路由是决定哪些 React 元素将在应用程序给定页面上渲染和如何嵌套的过程，React Router 提供了两个接口来声明路由。

- [`<Routes>` 和 `<Route>`](#routes-and-route) 使用 JSX 时
- [`useRoutes`](#useroutes) 如果更喜欢基于 JavaScript 对象的路由配置

内部一些底层 API 也暴露为公共 API，用于根据需要构建自己的高级接口。

- [`matchPath`](#matchpath) - 匹配路径模式（path pattern）与 URL pathname
- [`matchRoutes`](#matchroutes) - 根据 [location](#location) 匹配一组路由
- [`createRoutesFromChildren`](#createroutesfromchildren) - 从一组 React 元素（即 [`<Route>`](#routes-and-route) 元素）创建路由配置

### 导航

React Router 的导航接口可通过修改当前 [location](#location) 来改变当前渲染的页面，有两个主要接口可按需在应用程序页面之间导航。

- [`<Link>`](#link) 和 [`<NavLink>`](#navlink) 渲染可访问的 `<a>` 标签或在 React Native 上渲染可访问的 `TouchableHighlight`，让用户可以通过点击页面上的元素来导航。
- [`useNavigate`](#usenavigate) 和 [`<Navigate>`](#navigate) 以编程方式导航，通常在event handler中或响应某些状态变化时使用

内部一些底层 API也可用于构建自己的导航接口。

- [`useResolvedPath`](#useresolvedpath) - 解析当前 [location](#location) 的相对路径
- [`useHref`](#usehref) - 解析适合用作 `<a href>` 的相对路径
- [`useLocation`](#uselocation) 和 [`useNavigationType`](#usenavigationtype) - 描述了当前 [location](#location) 以及如何导航到该 location
- [`useLinkClickHandler`](#uselinkclickhandler) - 在 `react-router-dom` 中构建自定义 `<Link>` 时返回（return）用于导航的 event handler
- [`useLinkPressHandler`](#uselinkpresshandler) - 在 `react-router-native` 中构建自定义 `<Link>` 时返回用于导航的 event handler
- [`resolvePath`](#resolvepath) - 根据给定的 URL pathname 解析相对路径

### 搜索参数

通过 [`useSearchParams`](#usesearchparams) hook 提供对 URL [search parameters](https://developer.mozilla.org/en-US/docs/Web/API/URL/searchParams) 的访问。

---

## 参考

### `<BrowserRouter>`

<details>
  <summary>类型声明</summary>

```tsx
declare function BrowserRouter(
  props: BrowserRouterProps
): React.ReactElement;

interface BrowserRouterProps {
  basename?: string;
  children?: React.ReactNode;
  window?: Window;
}
```

</details>

`<BrowserRouter>`  Web 浏览器中运行 React Router 的推荐接口，BrowserRouter 是用于管理基于浏览器历史记录的路由的一种常用路由器。它使用 HTML5 的 history API 来管理 URL，确保页面导航时不重新加载整个页面。。

```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
    return (
        <BrowserRouter>
            // <App/>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
                <Route path="/contact" element={<Contact />} />
            </Routes>
        </BrowserRouter>
    );
}

```

### `<HashRouter>`

<details>
  <summary>类型声明</summary>

```tsx
declare function HashRouter(
  props: HashRouterProps
): React.ReactElement;

interface HashRouterProps {
  basename?: string;
  children?: React.ReactNode;
  window?: Window;
}
```

</details>

`<HashRouter>` 使用 URL 中的哈希部分（例如，http://example.com/#/about）来保持 UI 与当前路径同步。浏览器不会将 # 后的部分发送给服务器，这使得它适合在不需要服务器端处理路由的应用中使用。

```tsx
import * as React from "react";
import * as ReactDOM from "react-dom";
import { HashRouter } from "react-router-dom";

ReactDOM.render(
  <HashRouter>
    {/* app的其余部分在这里 */}
  </HashRouter>,
  root
);
```

<docs-warning>强烈建议不要使用`HashRouter`，除非必须。</docs-warning>

### `<NativeRouter>`

<details>
  <summary>类型声明</summary>

```tsx
declare function NativeRouter(
  props: NativeRouterProps
): React.ReactElement;

interface NativeRouterProps extends MemoryRouterProps {}
```

</details>

`<NativeRouter>` 是在 [React Native](https://reactnative.dev) 应用中运行 React Router 的推荐接口。

- `<NativeRouter initialEntries>` 默认为 `["/"]`（根 `/` URL 中的单个入口）
- `<NativeRouter initialIndex>` 默认为 `initialEntries` 的最后一个索引（index）

```tsx
import * as React from "react";
import { NativeRouter } from "react-router-native";

function App() {
  return (
    <NativeRouter>
      {/* app的其余部分在这里 */}
    </NativeRouter>
  );
}
```

### `<MemoryRouter>`

<details>
  <summary>类型声明</summary>

```tsx
declare function MemoryRouter(
  props: MemoryRouterProps
): React.ReactElement;

interface MemoryRouterProps {
  basename?: string;
  children?: React.ReactNode;
  initialEntries?: InitialEntry[];
  initialIndex?: number;
}
```

</details>

`<MemoryRouter>` 会把所有的路由信息存储在内存中，而不是在浏览器的地址栏中更新。这意味着它的 URL 不会反映在地址栏中，这也使得它不能直接通过地址栏进行页面刷新或书签操作。通常在测试环境中或需要隔离路由状态时使用

- `<MemoryRouter initialEntries>` 默认为 `["/"]`（根 `/` URL 处的单个入口）
- `<MemoryRouter initialIndex>` 默认为 `initialEntries` 的最后一个索引

```tsx
import * as React from "react";
import { create } from "react-test-renderer";
import {
  MemoryRouter,
  Routes,
  Route
} from "react-router-dom";

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

### `<Link>`

> **注意：**
>
> 这是 `<Link>` 的 web 版，React Native 版[去这里](#link-react-native)。

<details>
  <summary>类型声明</summary>

```tsx
declare function Link(props: LinkProps): React.ReactElement;

interface LinkProps
  extends Omit<
    React.AnchorHTMLAttributes<HTMLAnchorElement>,
    "href"
  > {
  replace?: boolean;
  state?: any;
  to: To;
  reloadDocument?: boolean;
}

type To = Partial<Location> | string;
```

</details>

`<Link>` 是一个让用户点击它导航到另一个页面的元素，`<Link>` 在 `react-router-dom` 中渲染一个带有所链接资源 `href` 的可访问 `<a>` 标签。 可以使用 `<Link reloadDocument>` 来跳过客户端路由，让浏览器正常处理跳转（就好像一个 `<a href>`）。

```tsx
import { Link } from 'react-router-dom';

function Navigation() {
    return (
        <nav>
            <ul>
                <li><Link to="/contact">Contact</Link></li>
                <li><Link to="/about" className="nav-link" style={{ fontWeight: 'bold' }}>
                    About
                </Link></li>
                <li><Link to="/profile" replace>Profile</Link></li>
                <li>
                    <Link to="/about" state={{ fromDashboard: true }}>
                        // 目标页面组件可以通过 useLocation() Hook 访问到这个 state
                        About
                    </Link>
                </li>
                <li>
                    <Link to={{
                        pathname: "/about",
                        search: "?sort=asc",
                        hash: "#section1"
                    }}>
                        About Section 1
                    </Link>
                </li>
            </ul>
        </nav>
    );
}

```

由渲染 `<Link>` 的路由匹配的 URL 路径所构建、相对于父路由进行解析的 `<Link to>` 相对值（不以 `/` 开头）可能包含 `..` 用以链接到更上层的路由，`..` 的工作方式与命令行的 `cd` 函数完全一样，每个 `..` 删除父路径一段。

> **注意：**
>
> 当当前 URL 以 `/` 结尾，
> 带有 `..` 的 `<Link to>` 行为与正常 `<a href>` 不同。 `<Link to>` 忽略尾部斜杠并删除
> 每个 `..` 一个 URL 段，而`<a href>` 值处理 `..` 的方式会以当前 URL 是否以 `/` 结尾而不同。

### `<Link>` (React Native)

> **注意：**
>
> 这是 `<Link>` 的 React Native 版，web 版[去这里](#link)。

<details>
  <summary>类型声明</summary>

```tsx
declare function Link(props: LinkProps): React.ReactElement;

interface LinkProps extends TouchableHighlightProps {
  children?: React.ReactNode;
  onPress?(event: GestureResponderEvent): void;
  replace?: boolean;
  state?: State;
  to: To;
}
```

</details>

`<Link>` 是一个让用户轻敲（tap）它导航到另一个视图的元素，类似于 `<a>` 元素在 web 应用中的工作方式，`<Link>` 在 `react-router-native` 中会渲染一个 `TouchableHighlight`。 要覆盖默认样式和行为，请参阅 [`TouchableHighlight` 的 Props 参考](https://reactnative.dev/docs/touchablehighlight#props)。

```tsx
import * as React from "react";
import { View, Text } from "react-native";
import { Link } from "react-router-native";

function Home() {
  return (
    <View>
      <Text>Welcome!</Text>
      <Link to="/profile">Visit your profile</Link>
    </View>
  );
}
```

### `<NavLink>`

<details>
  <summary>类型声明</summary>

```tsx
declare function NavLink(
  props: NavLinkProps
): React.ReactElement;

interface NavLinkProps
  extends Omit<
    LinkProps,
    "className" | "style" | "children"
  > {
  caseSensitive?: boolean;
  children?:
    | React.ReactNode
    | ((props: { isActive: boolean }) => React.ReactNode);
  className?:
    | string
    | ((props: { isActive: boolean }) => string);
  end?: boolean;
  style?:
    | React.CSSProperties
    | ((props: {
        isActive: boolean;
      }) => React.CSSProperties);
}
```

</details>

**属性:**

+ end: 如果你在 /about 页面上，/ 路径的链接也会被标记为活跃。你可以通过 end 属性让 NavLink 只在路径完全匹配时才被激活.
+ className: 属性自定义当链接处于活跃状态时应用的类名。该属性可以接受一个回调函数，回调函数会收到一个 isActive 参数，用来判断链接是否处于激活状态.
+ style: 接受一个函数，根据链接的活跃状态动态应用样式.
+ caseSensitive: 路由匹配区分大小写.

`<NavLink>` 是一种能知道它是否激活（active）的特殊 [`<Link>`](#link)，能用在构建导航菜单（例如面包屑或一组选项卡（tabs））时在高亮显示当前路由选项卡.


```tsx
import * as React from "react";
import { NavLink } from "react-router-dom";

function NavList() {
  // 将当前所选路由的样式应用于 <NavLink>。
  let activeStyle = {
    textDecoration: "underline"
  };

  let activeClassName = "underline"

  return (
    <nav>
      <ul>
        <li>
          <NavLink
            to="messages"
            style={({ isActive }) =>
              isActive ? activeStyle : undefined
            }
          >
            Messages
          </NavLink>
        </li>
        <li>
          <NavLink
            to="tasks"
            className={({ isActive }) =>
              isActive ? activeClassName : undefined
            }
          >
            Tasks
          </NavLink>
        </li>
        <li>
          <NavLink
            to="tasks"
          >
            {({ isActive }) => (
              <span className={isActive ? activeClassName : undefined}>
                Tasks
              </span>
            )}
          </NavLink>
        </li>
      </ul>
    </nav>
  );
}
```

### `<Navigate>`

<details>
  <summary>类型声明</summary>

```tsx
declare function Navigate(props: NavigateProps): null;

interface NavigateProps {
  to: To;
  replace?: boolean;
  state?: State;
}
```

</details>

`<Navigate>` 元素在渲染时会自动导航到指定路径,是 [`useNavigate`](#usenavigate) 的封装组件并接受相同参数作为 props。常见的使用场景包括登录后的重定向、页面权限检查后自动导航等.


```tsx
import { useParams, Navigate } from 'react-router-dom';

function UserProfile() {
    const { userId } = useParams();

    if (!userId) {
        return <Navigate to="/error" />;
    } else if (userId === 1) {
        // 目标页面中可以使用 useLocation Hook 获取传递的 state 信息
        return <Navigate to="/home" state={{ id: userId }} />;
    }

    return <div>Profile of user {userId}</div>;
}

```

### `<Outlet>`

<details>
  <summary>类型声明</summary>

```tsx
interface OutletProps {
  context?: unknown;
}
declare function Outlet(
  props: OutletProps
): React.ReactElement | null;
```

</details>

***属性:***

+ Outlet 时可以通过 context 属性传递数据,可以是任意数据. 在被渲染的子组件中使用useOutletContext获取其数据.

父路由元素中通过使用 `<Outlet>` 渲染子路由元素来显示嵌套 UI。 如果父路由精确匹配将渲染子索引路由，没有索引路由不渲染任何内容。

```tsx
function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      {/* 当 URL 为 "/messages" 时 <Outlet> 会渲染 <DashboardMessages>，为 "/tasks" 时会渲染 <DashboardTasks>，为 "/" 时 渲染 null */}
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
}
```

### `useOutletContext`

<details>
  <summary>类型声明</summary>

```tsx
declare function useOutletContext<
  Context = unknown
>(): Context;
```

</details>

在<Outlet>渲染的子组件中,获取父路由组件的context值:

```tsx lines=[3]
function Parent() {
  const [count, setCount] = React.useState(0);
  return <Outlet context={[count, setCount]} />;
}
```

```tsx lines=[2]
function Child() {
  const [count, setCount] = useOutletContext();
  const increment = () => setCount(c => c + 1);
  return <button onClick={increment}>{count}</button>;
}
```

使用 TypeScript时，父组件最好提供一个自定义 hook 来访问 context 值以使消费者更容易获得好的类型、控制消费者并知道谁在消费 context，下面是一个更实际的例子：

```tsx filename=src/routes/dashboard.tsx lines=[12,17-19]
import * as React from "react";
import type { User } from "./types";

type ContextType = { user: User | null };

export default function Dashboard() {
  const [user, setUser] = React.useState<User | null>(null);

  return (
    <div>
      <h1>Dashboard</h1>
      <Outlet context={user} />
    </div>
  );
}

export function useUser() {
  return useOutletContext<ContextType>();
}
```

```tsx filename=src/routes/dashboard/messages.tsx lines=[1,4]
import { useUser } from "../dashboard";

export default function DashboardMessages() {
  const user = useUser();
  return (
    <div>
      <h2>Messages</h2>
      <p>Hello, {user.name}!</p>
    </div>
  );
}
```

### `<Router>`

<details>
  <summary>类型声明</summary>

```tsx
declare function Router(
  props: RouterProps
): React.ReactElement | null;

interface RouterProps {
  basename?: string;
  children?: React.ReactNode;
  location: Partial<Location> | string;
  navigationType?: NavigationType;
  navigator: Navigator;
  static?: boolean;
}
```

</details>

`<Router>` 是所有 router 组件（[`<BrowserRouter>`](#browserrouter), [`<HashRouter>`](#hashrouter), [`<StaticRouter>`](#staticrouter)、[`<NativeRouter>`](#nativerouter) 和 [`<MemoryRouter>`](#memoryrouter)）共享的底层接口，对于 React 是一个 [context provider](https://reactjs.org/docs/context.html#contextprovider)，为应用的其余部分提供路由信息。

可能永远不需要手动渲染一个 `<Router>`，而是根据环境使用更高级 routers 的一种。 在一个给定的应用中只需一个 router。

在应用中，`<Router basename>` 属性可能在创建所有路由和链接都会被用到，因为它是它们都共享的 URL pathname 的“基本”部分，常用于使用 React Router 渲染较大应用的一部分或应用具有多个入口点时。 基本名称（Basenames）不区分大小写。

### `<Routes>` 和 `<Route>`

<details>
  <summary>类型声明</summary>

```tsx
declare function Routes(
  props: RoutesProps
): React.ReactElement | null;

interface RoutesProps {
  children?: React.ReactNode;
  location?: Partial<Location> | string;
}

declare function Route(
  props: RouteProps
): React.ReactElement | null;

interface RouteProps {
  caseSensitive?: boolean;
  children?: React.ReactNode;
  element?: React.ReactElement | null;
  index?: boolean;
  path?: string;
}
```

</details>

Routes 是一个用于定义路由规则的组件
`<Routes>` 包裹 `<Route>` 用于定义路由规则的组件。Routes 只渲染第一个匹配的 Route。

`<Route>` 属性:

+ caseSensitive 匹配是否区分大小写（默认为 `false`）
+ index 定义父路由的默认子路由,当父路由匹配时，index 路由会自动被渲染
+ path 匹配的路径
+ element 要渲染的react组件

```tsx
<Routes>
  <Route path="/" element={<Dashboard />}>
    <Route index element={<Index />} />
    <Route
      path="messages"
      element={<DashboardMessages />}
    />
    <Route path="tasks" element={<DashboardTasks />} />
  </Route>
  <Route path="about" element={<AboutPage />} />
</Routes>
```

> **注意：**
>
> 如果要把路由定义为常规 JavaScript 对象而不是使用 JSX，
> [请尝试使用 `useRoutes` 代替](#useroutes)。

`<Route element>` 默认是 [`<Outlet>`](#outlet)，即使没有 `element` 属性，默认渲染子元素.

例如:

父路由默认渲染子路由,子路由路径是 `/users/:id` 

```tsx
<Route path="users">
  <Route path=":id" element={<UserProfile />} />
</Route>
```

### `<StaticRouter>`

<details>
  <summary>类型声明</summary>

```tsx
declare function StaticRouter(
  props: StaticRouterProps
): React.ReactElement;

interface StaticRouterProps {
  basename?: string;
  children?: React.ReactNode;
  location?: Path | LocationPieces;
}
```

</details>

`<StaticRouter>` 用于在 [node](https://nodejs.org) 中服务端渲染 React Router Web 应用程序，通过 location 属性接收当前的 URL，模拟客户端路由行为.

- `<StaticRouter location>` 默认为 `"/"`

```tsx
import { StaticRouter } from 'react-router-dom/server';
import { Routes, Route } from 'react-router-dom';
import ReactDOMServer from 'react-dom/server';
import Home from './Home';
import About from './About';

function App({ location }) {
    return (
        <StaticRouter location={location}>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
            </Routes>
        </StaticRouter>
    );
}

// 服务器端渲染示例
const serverRender = (req) => {
    const html = ReactDOMServer.renderToString(<App location={req.url} />);
    return html;
};

export default serverRender;

```

### `createRoutesFromChildren`

<details>
  <summary>类型声明</summary>

```tsx
declare function createRoutesFromChildren(
  children: React.ReactNode
): RouteObject[];

interface RouteObject {
  caseSensitive?: boolean;
  children?: RouteObject[];
  element?: React.ReactNode;
  index?: boolean;
  path?: string;
}
```

</details>

`createRoutesFromChildren` 创建动态路由函数。再通过useRoutes进行路由渲染.

**示例代码:**

```jsx
import { BrowserRouter, Routes, Route, useRoutes, createRoutesFromChildren } from 'react-router-dom';

function App() {
  const routes = createRoutesFromChildren(
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/dashboard" element={<Dashboard />} />
    </Routes>
  );

  return useRoutes(routes);
}

function Home() {
  return <h1>Home Page</h1>;
}

function About() {
  return <h1>About Page</h1>;
}

function Dashboard() {
  return <h1>Dashboard Page</h1>;
}

export default function Root() {
  return (
    <BrowserRouter>
      <App />
    </BrowserRouter>
  );
}

```


### `generatePath`

<details>
  <summary>类型声明</summary>

```tsx
declare function generatePath(
  path: string,
  params?: Params
): string;
```

</details>

`generatePath` 生成路由路径。

**示例代码:**

```jsx
import { generatePath } from 'react-router-dom';

const path = generatePath("/users/:id/:tab?", { id: 42 });
console.log(path);  // 输出: "/users/42"

const pathWithTab = generatePath("/users/:id/:tab?", { id: 42, tab: "profile" });
console.log(pathWithTab);  // 输出: "/users/42/profile"
```

### `Location`

React Router 中的术语 “location” 是指来自 [history](https://github.com/remix-run/history) 库的 `Location` 接口。

**示例代码:**

```jsx

import { useLocation } from 'react-router-dom';

function MyComponent() {
  const location = useLocation();

  console.log(location);
  /**
   * Location 对象具有以下属性：
   pathname: 当前 URL 的路径部分（不包含查询字符串或哈希）。
   search: 当前 URL 中的查询字符串（以 ? 开头）。
   hash: 当前 URL 的哈希部分（以 # 开头）。
   state: 通过 navigate 或 Link 导航传递的状态（即非 URL 中显示的额外数据）。
   key: 当前导航的唯一标识符（通常用于浏览历史）。
   * */

  return (
    <div>
      <p>Current Pathname: {location.pathname}</p>
      <p>Current Search: {location.search}</p>
      <p>Current Hash: {location.hash}</p>
    </div>
  );
}

```

### `matchRoutes`

<details>
  <summary>类型声明</summary>

```tsx
declare function matchRoutes(
  routes: RouteObject[],
  location: Partial<Location> | string,
  basename?: string
): RouteMatch[] | null;

interface RouteMatch<ParamKey extends string = string> {
  params: Params<ParamKey>;
  pathname: string;
  route: RouteObject;
}
```

</details>

`matchRoutes` 对给定路径进行路由匹配，返回匹配的路由对象数组。

**示例代码:**

```jsx
import { matchRoutes } from 'react-router-dom';

const routes = [
    {
        path: "/",
        element: <Layout />,
        children: [
            { path: "about", element: <About /> },
            { path: "users/:id", element: <User /> },
        ],
    },
];

const matchedRoutes = matchRoutes(routes, "/users/123");

console.log(matchedRoutes);
// 输出类似以下结构
/*
[
  {
    route: { path: "/", element: <Layout /> },
    pathname: "/"
  },
  {
    route: { path: "users/:id", element: <User /> },
    pathname: "/users/123",
    params: { id: "123" }
  }
]
*/

```

### `renderMatches`

<details>
  <summary>类型声明</summary>

```tsx
declare function renderMatches(
  matches: RouteMatch[] | null
): React.ReactElement | null;
```

</details>

`renderMatches` 将 `matchRoutes()` 的结果渲染到 React 元素中。

**示例代码:**

```jsx
import { useLocation, matchRoutes, renderMatches } from 'react-router-dom';

function App() {
  const location = useLocation();
  
  // 匹配当前路径
  const matches = matchRoutes(routes, location.pathname);

  // 渲染匹配的路由
  return (
    <div>
      {renderMatches(matches) || <p>No matches found</p>}
    </div>
  );
}

```

### `matchPath`

<details>
  <summary>类型声明</summary>

```tsx
declare function matchPath<
  ParamKey extends string = string
>(
  pattern: PathPattern | string,
  pathname: string
): PathMatch<ParamKey> | null;

interface PathMatch<ParamKey extends string = string> {
  params: Params<ParamKey>;
  pathname: string;
  pathnameBase: string;
  pattern: PathPattern;
}

interface PathPattern {
  path: string;
  caseSensitive?: boolean;
  end?: boolean;
}
```

</details>

+ pattern: 用于匹配的路径模式，支持动态参数（如 :id）和通配符（如 *），也可以是一个对象。

+ pathname: 实际要匹配的路径。

**pattern 的结构:**

1. 路径字符串: 例如 /users/:id 或 /about

2. 对象: 
+ path: 匹配的路径模式，支持动态参数和通配符。
+ caseSensitive: 匹配是否区分大小写（默认是 false）。
+ end: 是否要求完全匹配（默认是 true，表示必须完全匹配路径）。

**返回值:**
1. 如果路径匹配成功，matchPath 返回一个对象，包含以下属性：

+ params: 包含匹配的动态参数（如 /users/:id 中的 id）。
+ pathname: 当前匹配的路径部分。
+ pathnameBase: 匹配的基础路径（忽略通配符部分）。
+ pattern: 使用的匹配模式对象。

2. 如果匹配失败，返回 null。

`matchPath` 匹配路由，匹配返回路由对象PathMatch，不匹配返回 `null`。

[`useMatch` hook](#usematch) 在内部使用此函数来匹配当前 location 的路由路径。

**示例代码:**

```jsx

import { matchPath } from 'react-router-dom';

const match = matchPath("/users/:id", "/users/123");
console.log(match);
// 输出:
// {
//   params: { id: "123" },
//   pathname: "/users/123",
//   pathnameBase: "/users/123",
//   pattern: { path: "/users/:id", caseSensitive: false, end: true }
// }

const match = matchPath("/docs/*", "/docs/react/getting-started");
console.log(match);
// 输出:
// {
//   params: { "*": "react/getting-started" },
//   pathname: "/docs/react/getting-started",
//   pathnameBase: "/docs",
//   pattern: { path: "/docs/*", caseSensitive: false, end: true }
// }

const match = matchPath({ path: "/users", end: false }, "/users/123");
console.log(match);
// 输出:
// {
//   params: {},
//   pathname: "/users",
//   pathnameBase: "/users",
//   pattern: { path: "/users", caseSensitive: false, end: false }
// }


const match = matchPath({ path: "/About", caseSensitive: true }, "/about");
console.log(match);
// 输出: null，因为大小写不匹配


```

### `resolvePath`

<details>
  <summary>类型声明</summary>

```tsx
declare function resolvePath(
  to: To,
  fromPathname?: string
): Path;

type To = Partial<Location> | string;

interface Path {
  pathname: string;
  search: string;
  hash: string;
}
```

</details>

+ to: 你想要解析的目标路径。可以是一个相对路径或绝对路径。
+ from: （可选）作为基准的路径。默认为当前的 location.pathname，但你可以传入一个特定的路径作为基准。

**返回值:**

resolvePath 返回一个对象，包含以下属性：

+ pathname: 解析后的路径。
+ search: URL 的查询字符串（例如 ?name=value 部分）。
+ hash: URL 的哈希片段（例如 #section 部分）。

`resolvePath` 用于将相对路径解析为绝对路径,可提供基准路径 from，不提供，它会使用当前的 location.pathname 作为基准.返回路径、查询字符串和哈希片段的对象.

[`useResolvedPath` hook](#useResolvedpath) 类似resolvePath, 是React Hook，可以在组件中直接使用，基于当前的路由上下文来解析相对路径。

**示例代码:**

```jsx

import { resolvePath } from 'react-router-dom';

const resolvedPath = resolvePath('about', '/home');
console.log(resolvedPath);
// 输出:
// {
//   pathname: '/home/about',
//   search: '',
//   hash: ''
// }

// 在这个例子中，'/about' 是绝对路径，所以基准路径 '/home' 被忽略，解析后的路径直接是 /about。
const resolvedPath = resolvePath('/about', '/home');
console.log(resolvedPath);
// 输出:
// {
//   pathname: '/about',
//   search: '',
//   hash: ''
// }

const resolvedPath = resolvePath('profile?name=john#bio', '/users');
console.log(resolvedPath);
// 输出:
// {
//   pathname: '/users/profile',
//   search: '?name=john',
//   hash: '#bio'
// }

// 如果 from 参数没有提供，则默认使用当前路径
const resolvedPath = resolvePath('about');
console.log(resolvedPath);
// 假设当前路径是 '/home'，输出:
// {
//   pathname: '/home/about',
//   search: '',
//   hash: ''
// }


```

### `useHref`

<details>
  <summary>类型声明</summary>

```tsx
declare function useHref(to: To): string;
```

</details>

`useHref` hook 返回一个可链接到给定 `to` 的 location 的 URL，该 location 可以在 React Router 之外。

**示例代码:**

```jsx

import { Routes, Route, Link, useHref } from 'react-router-dom';

function Contact() {
  const href = useHref('/about'); // 返回 "/about", 即当前 basename 下的完整路径
  return (
    <div>
      <h2>Contact</h2>
      <a href={href}>Go to About Page</a>
    </div>
  );
}

```

### `useLinkClickHandler`

<details>
  <summary>类型声明</summary>

```tsx
declare function useLinkClickHandler<
  E extends Element = HTMLAnchorElement,
  S extends State = State
>(
  to: To,
  options?: {
    target?: React.HTMLAttributeAnchorTarget;
    replace?: boolean;
    state?: S;
  }
): (event: React.MouseEvent<E, MouseEvent>) => void;
```

</details>

在 `react-router-dom` 中构建自定义 `<Link>` 时，`useLinkClickHandler` hook 会返回一个点击事件 handler 来进行导航。

**示例代码:**

```jsx

```

```tsx
import {
  useHref,
  useLinkClickHandler
} from "react-router-dom";

const StyledLink = styled("a", { color: "fuchsia" });

const Link = React.forwardRef(
  (
    {
      onClick,
      replace = false,
      state,
      target,
      to,
      ...rest
    },
    ref
  ) => {
    let href = useHref(to);
    let handleClick = useLinkClickHandler(to, {
      replace,
      state,
      target
    });

    return (
      <StyledLink
        {...rest}
        href={href}
        onClick={event => {
          onClick?.(event);
          if (!event.defaultPrevented) {
            handleClick(event);
          }
        }}
        ref={ref}
        target={target}
      />
    );
  }
);
```

### `useLinkPressHandler`

<details>
  <summary>类型声明</summary>

```tsx
declare function useLinkPressHandler<
  S extends State = State
>(
  to: To,
  options?: {
    replace?: boolean;
    state?: S;
  }
): (event: GestureResponderEvent) => void;
```

</details>

作为 `useLinkClickHandler` 在 `react-router-native` 中的对应项，`useLinkPressHandler` 返回一个用于自定义 `<Link>` 导航的 press 事件 handler。

**示例代码:**

```jsx

```

```tsx
import { TouchableHighlight } from "react-native";
import { useLinkPressHandler } from "react-router-native";

function Link({
  onPress,
  replace = false,
  state,
  to,
  ...rest
}) {
  let handlePress = useLinkPressHandler(to, {
    replace,
    state
  });

  return (
    <TouchableHighlight
      {...rest}
      onPress={event => {
        onPress?.(event);
        if (!event.defaultPrevented) {
          handlePress(event);
        }
      }}
    />
  );
}
```

### `useInRouterContext`

<details>
  <summary>类型声明</summary>

```tsx
declare function useInRouterContext(): boolean;
```

</details>

根据组件是否在包含 `<Router>` 的上下文中渲染， `useInRouterContext` hook 返回 `true` 或 `false`，常用于第三方扩展检验是否在包含 React Router 应用程序的上下文中。

**示例代码:**

```jsx

```

### `useLocation`

<details>
  <summary>类型声明</summary>

```tsx
declare function useLocation(): Location;

interface Location<S extends State = object | null>
  extends Path {
  state: S;
  key: Key;
}
```

</details>

此 hook 返回当前 [`location`](#location) 对象，可用于在当前 location 改变时执行一些副作用。

```tsx
import * as React from 'react';
import { useLocation } from 'react-router-dom';

function App() {
  let location = useLocation();

  React.useEffect(() => {
    ga('send', 'pageview');
  }, [location]);

  return (
    // ...
  );
}
```

### `useNavigationType`

<details>
  <summary>类型声明</summary>

```tsx
declare function useNavigationType(): NavigationType;

type NavigationType = "POP" | "PUSH" | "REPLACE";
```

</details>

此 hook 通过 history 堆栈上 pop、push 或 replace 操作，返回当前导航类型或者用户进入当前页的方式。

**示例代码:**

```jsx

```

### `useMatch`

<details>
  <summary>类型声明</summary>

```tsx
declare function useMatch<ParamKey extends string = string>(
  pattern: PathPattern | string
): PathMatch<ParamKey> | null;
```

</details>

返回给定路径相对当前 location 的路由匹配数据。

有关详细信息，请参阅 [`matchPath`](#matchpath)。

**示例代码:**

```jsx

```

### `useNavigate`

<details>
  <summary>类型声明</summary>

```tsx
declare function useNavigate(): NavigateFunction;

interface NavigateFunction {
  (
    to: To,
    options?: { replace?: boolean; state?: any }
  ): void;
  (delta: number): void;
}
```

</details>

`useNavigate` hook 返回一个用编程方式导航的函数，可用于例如提交表单。

**示例代码:**

```jsx

```

```tsx
import { useNavigate } from "react-router-dom";

function SignupForm() {
  let navigate = useNavigate();

  async function handleSubmit(event) {
    event.preventDefault();
    await submitForm(event.target);
    navigate("../success", { replace: true });
  }

  return <form onSubmit={handleSubmit}>{/* ... */}</form>;
}
```

`navigate` 函数有两个签名：

- 传一个带有可选第二个 `{ replace, state }` 变量（arg）的 `To` 值（与 `<Link to>` 类型相同） 或
- 传想要入 history 堆栈的增量，例如 `navigate(-1)` 相当于点击后退按钮。

### `useOutlet`

<details>
  <summary>类型声明</summary>

```tsx
declare function useOutlet(): React.ReactElement | null;
```

</details>

返回位于该子路由层级的子路由元素，[`<Outlet>`](#outlet) 在内部使用此 hook 来渲染子路由。

**示例代码:**

```jsx

```

### `useParams`

<details>
  <summary>类型声明</summary>

```tsx
declare function useParams<
  K extends string = string
>(): Readonly<Params<K>>;
```

</details>

`useParams` hook 返回当前 URL 与 `<Route path>` 匹配的动态参数的键值对（key/value pairs）对象，子路由继承父路由的所有参数。

**示例代码:**

```jsx

```

```tsx
import * as React from 'react';
import { Routes, Route, useParams } from 'react-router-dom';

function ProfilePage() {
  // 从 URL 中获取 userId
  let { userId } = useParams();
  // ...
}

function App() {
  return (
    <Routes>
      <Route path="users">
        <Route path=":userId" element={<ProfilePage />} />
        <Route path="me" element={...} />
      </Route>
    </Routes>
  );
}
```

### `useResolvedPath`

<details>
  <summary>类型声明</summary>

```tsx
declare function useResolvedPath(to: To): Path;
```

</details>

+ to: 你想要解析的目标路径，通常可以是相对路径或者绝对路径

**返回值:**

useResolvedPath 返回一个对象，包含以下属性：

+ pathname: 解析后的路径。
+ search: URL 的查询字符串（例如 ?name=value 部分）。
+ hash: URL 的哈希片段（例如 #section 部分）。

useResolvedPath 可以在组件中直接使用，基于当前的路由上下文来解析相对路径。

有关详细信息，请参阅 [`resolvePath`](#resolvepath)。

**示例代码:**

```jsx

import { Link, useResolvedPath } from 'react-router-dom';

function MyComponent() {
  const resolvedPath = useResolvedPath('about');
  
  console.log(resolvedPath);
  // useResolvedPath('about') 会根据当前的路由上下文（假设当前路径是 /home）解析相对路径，生成一个完整的路径，比如 /home/about。
  return (
    <div>
        // 结合Link组件实现绝对路径跳转
      <Link to={resolvedPath}>Go to About</Link>
    </div>
  );
}

// 生成动态路径
function UserProfile({ userId }) {
    const resolvedPath = useResolvedPath(`/users/${userId}/profile`);

    return (
        <div>
            <p>Resolved Path: {resolvedPath.pathname}</p>
        </div>
    );
}


```

### `useRoutes`

<details>
  <summary>类型声明</summary>

```tsx
declare function useRoutes(
  routes: RouteObject[],
  location?: Partial<Location> | string;
): React.ReactElement | null;
```

</details>

`useRoutes` hook 在功能上等同于 [`<Routes>`](#routes) 但使用 JavaScript 对象而不是 `<Route>` 元素来定义路由，所用对象与普通 [`<Route>` 元素](#routes-and-route) 具有相同属性但不需要用 JSX。

`useRoutes` 返回值是一个可用来渲染路由树的有效 React 元素，如果没有匹配项则返回 `null`。

**示例代码:**

```jsx

```

```tsx
import * as React from "react";
import { useRoutes } from "react-router-dom";

function App() {
  let element = useRoutes([
    {
      path: "/",
      element: <Dashboard />,
      children: [
        {
          path: "messages",
          element: <DashboardMessages />
        },
        { path: "tasks", element: <DashboardTasks /> }
      ]
    },
    { path: "team", element: <AboutPage /> }
  ]);

  return element;
}
```

### `useSearchParams`

> **注意：**
>
> 这是 `useSearchParams` 的 web 版，React Native 版[去这里](#usesearchparams-react-native)。

<details>
  <summary>类型声明</summary>

```tsx
declare function useSearchParams(
  defaultInit?: URLSearchParamsInit
): [URLSearchParams, URLSearchParamsSetter];

type ParamKeyValuePair = [string, string];

type URLSearchParamsInit =
  | string
  | ParamKeyValuePair[]
  | Record<string, string | string[]>
  | URLSearchParams;

interface URLSearchParamsSetter {
  (
    nextInit: URLSearchParamsInit,
    navigateOptions?: { replace?: boolean; state?: State }
  ): void;
}
```

</details>

`useSearchParams` hook 用于读取和修改 URL 中当前 location 的查询字符串（query string），和 React 的 [`useState` hook](https://reactjs.org/docs/hooks-reference.html#usestate) 一样返回一个长度为2的数组：当前 location 的 [search params](https://developer.mozilla.org/en-US/docs/Web/API/URL/searchParams) 和一个可用来更新前者的函数。

**示例代码:**

```jsx

```

```tsx
import * as React from "react";
import { useSearchParams } from "react-router-dom";

function App() {
  let [searchParams, setSearchParams] = useSearchParams();

  function handleSubmit(event) {
    event.preventDefault();
    // 这里序列化函数将用于从构成查询的表单字段创建一个键值对对象。
    let params = serializeFormQuery(event.target);
    setSearchParams(params);
  }

  return (
    <div>
      <form onSubmit={handleSubmit}>{/* ... */}</form>
    </div>
  );
}
```

> **注意：**
>
> `setSearchParams` 函数工作方式类似于 [`navigate`](#usenavigate)，但
> 仅适用于 URL 的 [search 部分](https://developer.mozilla.org/en-US/docs/Web/API/Location/search)。
> 另请注意，`setSearchParams` 的第二个参数与 `navigate` 的第二个参数类型相同。

### `useSearchParams` (React Native)

> **注意：**
>
> 这是 `useSearchParams` 的 React Native 版，web 版[去这里](#usesearchparams)。

<details>
  <summary>类型声明</summary>

```tsx
declare function useSearchParams(
  defaultInit?: URLSearchParamsInit
): [URLSearchParams, URLSearchParamsSetter];

type ParamKeyValuePair = [string, string];

type URLSearchParamsInit =
  | string
  | ParamKeyValuePair[]
  | Record<string, string | string[]>
  | URLSearchParams;

interface URLSearchParamsSetter {
  (
    nextInit: URLSearchParamsInit,
    navigateOptions?: { replace?: boolean; state?: State }
  ): void;
}
```

</details>

用法与 web 版相同。

```tsx
import * as React from "react";
import { View, SearchForm, TextInput } from "react-native";
import { useSearchParams } from "react-router-native";

function App() {
  let [searchParams, setSearchParams] = useSearchParams();
  let [query, setQuery] = React.useState(
    searchParams.get("query")
  );

  function handleSubmit() {
    setSearchParams({ query });
  }

  return (
    <View>
      <SearchForm onSubmit={handleSubmit}>
        <TextInput value={query} onChangeText={setQuery} />
      </SearchForm>
    </View>
  );
}
```

### `createSearchParams`

<details>
  <summary>类型声明</summary>

```tsx
declare function createSearchParams(
  init?: URLSearchParamsInit
): URLSearchParams;
```

</details>

`createSearchParams` 是 [`new URLSearchParams(init)`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/URLSearchParams) 增加了对带有数组值对象支持的一层简单封装，也是 `useSearchParams` 在内部用 `URLSearchParamsInit` 值创建 `URLSearchParams` 对象的函数。

**示例代码:**

```jsx

```