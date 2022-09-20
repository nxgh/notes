# react 中使用 function composition

> 这篇文章介绍了如何把 Function Composition 用在 React 项目中，换一种代码组织方式，让代码更简洁、优雅和可扩展。

> 原文地址: [Why Every React Developer Should Learn Function Composition](https://medium.com/javascript-scene/why-every-react-developer-should-learn-function-composition-23f41d4db3b1)

想象一下，你正在构建一个 React 应用程序。您希望在应用程序的几乎每个页面视图上执行许多操作。



目标场景：
您希望在应用程序的几乎每个页面视图上执行

1. 检查并更新用户验证状态
2. 检查当前活动功能以决定要呈现哪些功能（持续交付需要）
3. 记录每个页面组件挂载
4. 渲染标准布局（导航、侧边栏等）

像这样的事情通常被称为 cross-cutting concerns(横切关注点)。 起初，你不会这样想他们。你只是厌倦了将一堆样板代码复制粘贴到每个组件中。就像这样

> [横切关注点-wiki](https://zh.wikipedia.org/wiki/%E6%A8%AA%E5%88%87%E5%85%B3%E6%B3%A8%E7%82%B9)
> 
> 横切关注点指的是一些具有横越多个模块的行为，使用传统的软件开发方法不能够达到有效的模块化的一类特殊关注点。

```tsx
const MyPage = ({ user = {}, signIn, features = [], log }) => {
  // 判断登录状态
  useEffect(() => { 
    if (!user.isSignedIn) signIn(); 
  }, [user]); 
  // 记录每个页面组件装载
  useEffect(() => {
    log({ type: 'page', name: 'MyPage', user: user.id, });
  }, []);
  return <>
    {
      user.isSignedIn ?
        <NavHeader>
          <NavBar />
          { features.includes('new-nav-feature') && <NewNavFeature /> }
        </NavHeader>
          <div className="content">
          </div>
        <Footer /> :
        <SignInComponent />
    }
  </>;
};
```

我们可以通过将所有这些东西抽象到单独的 Provider 组件中来

```tsx
const MyPage = ({ user = {}, signIn, features = [], log }) => {
  return (
      <AuthStatusProvider>
        <FeatureProvider>
          <LogProvider>
            <StandardLayout>
              <div className="content">{/* our actual page content... */}</div>
            </StandardLayout>
          </LogProvider>
        </FeatureProvider>
      </AuthStatusProvider>
  );
};
```

不过，我们仍然有一些问题
如果我们的标准横切关注点(cross-cutting concerns)发生变化，我们需要在每个页面上更改它们，并保持所有页面同步, 我们还必须记住将 providers 添加到每个页面。

## Higher Order Components

更好的解决方案是使用高阶组件（HOC） 来包装我们的页面组件。HOC 是一个接受组件并返回新组件的函数。新组件将渲染原始组件，且具有一些附加功能。我们可以使用它来包装我们的页面组件与我们需要的所有提供程序。

```tsx 
const MyPage = ({ user = {}, signIn, features = [], log }) => {
  return <>{/* our actual page content... */}</>;
};
const MyPageWithProviders = withProviders(MyPage);
```

让我们来看看我们的记录器作为 HOC 的样子：

```tsx
const withLogger = (WrappedComponent) => {
  return function LoggingProvider (props) {
    useEffect(() => {
      log({
        type: 'page',
        name: 'MyPage',
        user: user.id,
      });
    }, []);
    return <WrappedComponent {...props} />;
  };
};
```

## 函数组合 Function Composition

为了让我们所有的提供者一起工作，我们可以使用函数组合将它们组合成一个 HOC。函数组合是组合两个或多个函数以产生新功能的过程。

这是一个非常强大的概念，可用于构建复杂的应用程序。函数组合是将一个函数应用于另一个函数的返回值。在代数中，它由函数组合运算符表示：`∘`

```tsx
(f ∘ g)(x) = f(g(x))
```

在JavaScript中，我们可以创建一个名为 `compose` 的函数，并使用它来编写高阶组件：

```tsx
const compose = (...fns) => (x) => fns.reduceRight((y, f) => f(y), x); 

const withProviders = compose(
     withUser,
     withFeatures,
     withLogger, 
     withLayout
)
export default withProviders; 
```

现在，您可以在任何需要的地方使用 `withProviders` 。不过，这还不够。大多数应用程序都有许多不同的页面，不同的页面有时会有不同的需求。例如，我们有时不想显示页脚（在具有无限内容流的页面上）。

## 函数柯里化

curried 函数是一个一次接受多个参数的函数，它通过返回一系列函数，每个函数都接受下一个参数。

```js
const add = (a) => (b) => a + b;

// 现在我们可以专门化该函数以将1加到任何数字中
const increment = add(1);
```

这是一个微不足道的例子，但柯里化有助于函数组合，因为一个函数只能返回一个值。如果我们想自定义布局函数以采用额外的参数，最好的解决方案是将其柯里化

```tsx
const withLayout = ({ showFooter = true }) =>
  (WrappedComponent) => {
    return function LayoutProvider ({ features, ...props}) {
      return (
        <>
          <NavHeader>
            <NavBar />
            { features.includes('new-nav-feature') && <NewNavFeature /> }
         </NavHeader>
        <div className="content">
          <WrappedComponent features={features} {...props} />
        </div>
        { showFooter && <Footer /> }
      </>
    );
  };
};
```
我们不能只是 curry 布局功能, 我们还需要使用 `withProviders`

```tsx
const withProviders = (options) =>
  compose( withUser, withFeatures, withLogger, withLayout(options));
```

现在，我们可以使用withProviders将任何页面组件与我们需要的所有提供者包裹起来，并为每个页面定制布局

```tsx
const MyPage = ({ user = {}, signIn, features = [], log }) => {
  return <>{/* our actual page content... */}</>;
};
const MyPageWithProviders = withProviders({
  showFooter: false
})(MyPage);
```

## API 路由中的函数组合

函数组合不仅在客户端有用。它还可用于处理 API 路由中的横切问题。一些常见的问题包括：

- 认证 (Authentication) 
- 授权 (Authorization )
- 验证 (Validation) 
- 日志 (Logging)
- 错误处理 (Error handling)

与上面的 HOC 示例一样，我们可以使用函数组合将 API 路由处理程序与所需的所有提供程序打包在一起

Next.js 使用轻量级云函数作为 API 路由，不再使用 Express。Express 主要用于其中间件堆栈和 `app.use()` 函数，它允许我们轻松地将中间件添加到我们的 API 路由中。

`app.use()` 函数只是 API 中间件的异步函数组合
```ts
app.use((request, response, next) => {
  // do something
  next();
});
```

但是我们可以用异步Pipe做同样的事情, 你可以用它来编写返回Promise 的函数。

```javascript
const asyncPipe = (...fns) => (x) =>
  fns.reduce(async (y, f) => f(await y), x);
```
现在我们可以像这样编写中间件和 API 路由：

```js
const withAuth = async ({request, response}) => {
  // do something
};
```

在我们构建的应用程序中，我们有为我们创建服务器路由的功能。它基本上是一个围绕异步Pipe 的包装器，内置了一些错误处理：

```js
const createRoute = (...middleware) =>
  async (request, response) => {
    try {
      await asyncPipe(...middleware)({
        request,
        response,
      });
    } catch (e) {
      const requestId = response.locals.requestId;
      const { url, method, headers } = request;
      console.log({
        time: new Date().toISOString(),
        body: JSON.stringify(request.body),
        query: JSON.stringify(request.query),
        method,
        headers: JSON.stringify(headers),
        error: true,
        url,
        message: e.message,
        stack: e.stack,
        requestId,
      });
      response.status(500);
      response.json({
        error: 'Internal Server Error',
        requestId,
      });
    }
  };
```

在您的 API 路由中，您可以像这样导入和使用它：

```javascript
import createRoute from 'lib/createRoute';
// A pre-composed pipeline of default middleware
import defaultMiddleware from 'lib/defaultMiddleware';

const helloWorld = async ({ request, response }) => {
  request.status(200);
  request.json({ message: 'Hello World' });
};

export default createRoute(
  defaultMiddleware,
  helloWorld
);
```

有了这些模式，函数组合就形成了将应用程序中的所有横切问题汇集在一起的主干。

每当你发现自己在想，**对于每个 component/page/route，我需要做X，Y和Z 时，你应该考虑使用函数组合来解决问题** 。

