## useEffect 中的请求竞态条件

> [竞争条件（race condition）](https://zh.wikipedia.org/wiki/%E7%AB%B6%E7%88%AD%E5%8D%B1%E5%AE%B3)，它旨在描述一个系统或者进程的输出依赖于不受控制的事件出现顺序或者出现时机

### 场景

前端开发最常见的逻辑就是从后台服务器获取并处理数据然后渲染到浏览器页面上, 网络请求的过程是复杂的，且响应时间是不确定的，访问同一个目的地址，请求经过的网络链路不一定是一样的路径。

先发出的请求不一定先响应，如果前端以先发请求先响应的规则来开发的话，那么就可能会导致错误的数据使用，即出现了竞态条件问题。

在使用 useEffect 获取数据时，如果 id 更改得足够快，它可能具有竞态条件, 为了使竞态更加明显这里为每个请求添加了一个随机等待时间

```jsx
import { useEffect, useState } from 'react'

function DataDisplayer(props) {
  const [data, setData] = useState(null)

  useEffect(() => {
    const fetchData = async () => {
      setTimeout(async () => {
        const response = await fetch(`https://swapi.dev/api/people/${props.id}`)
        setData(await response.json())
      }, Math.round(Math.random() * 5000))
    }
    fetchData()
  }, [props.id])

  if (data) {
    return (
      <div>
        <p>
          Displaying Data for: fetchedId: {data.url.match(/[1-9]\d*/)[0]}, props.id: {props.id}
        </p>
        <p>{data.name}</p>
      </div>
    )
  } else {
    return null
  }
}

export default () => {
  const [id, setId] = useState(1)

  return (
    <>
      <button onClick={() => setId(Math.round(Math.random() * 80))}>Change ID</button>
      <DataDisplayer id={id} />
    </>
  )
}
```

这里如果快速点击 ChangeID 按钮，程序将发出多个请求，而请求完成的顺序是随机的， 最终展示哪个用户的数据，取决于哪个请求先返回。这就是请求的竞态问题

### 如何解决

1. 清理 useEffect

```diff jsx
useEffect(() => {
+   let flag = true;
    const fetchData = async () => {
      setTimeout(async () => {
        const response = await fetch(`https://swapi.dev/api/people/${props.id}`)
+       if (!flag) return
        setData(await response.json())
      }, Math.round(Math.random() * 5000))
    }
    fetchData()
+   return () => { flag = false}
  }, [props.id])
```

2. 中止请求

```diff
 useEffect(() => {
+   const abortController = new AbortController()
    const fetchData = async () => {
      setTimeout(async () => {
      try {
          const response = await fetch(`https://swapi.dev/api/people/${props.id}`, {
+           signal: abortController.signal,
          })
          setData(await response.json())
+       } catch (error) {}
      }, Math.round(Math.random() * 5000))
    }
    fetchData()
+    return () => { abortController.abort() }
  }, [props.id])
```

使用 AbortController 中止一个 promise

```js
function wait(time: number, signal?: AbortSignal) {
  return new Promise<void>((resolve, reject) => {
    const timeoutId = setTimeout(() => {
      resolve();
    }, time);
    signal?.addEventListener('abort', () => {
      clearTimeout(timeoutId);
      reject();
    });
  });
}
const abortController = new AbortController();

setTimeout(() => {
  abortController.abort();
}, 1000);

wait(5000, abortController.signal)
  .then(() => {
    console.log('5 seconds passed');
  })
  .catch(() => {
    console.log('Waiting was interrupted');
  });
```

3. 更好的解决方案

- 使用 react-query 或者 swr 🙂

## 出现的原因

[useEffect 完整指南-Dan](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)

> **组件内的每一个函数（包括事件处理函数，effects，定时器或者 API 调用等等）会捕获定义它们的那次渲染中的 props 和 state**

effect 的清除并不会读取“最新”的 props。它只能读取到定义它的那次渲染中的 props 值

思考以下代码：

```jsx
useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange)
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange)
  }
})
```

React 只会在浏览器绘制后运行 effects。这使得你的应用更流畅因为大多数 effects 并不会阻塞屏幕的更新。Effect 的清除同样被延迟了。上一次的 effect 会在重新渲染后被清除：

- React 渲染{id: 20}的 UI。
- 浏览器绘制。我们在屏幕上看到{id: 20}的 UI。
- React 清除{id: 10}的 effect。
- React 运行{id: 20}的 effect。


## 参考

- [useEffect 完整指南-Dan](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)
- [将 React 作为 UI 运行时-Dan](https://overreacted.io/zh-hans/react-as-a-ui-runtime/)
- [竞态条件-wiki](https://zh.wikipedia.org/wiki/%E7%AB%B6%E7%88%AD%E5%8D%B1%E5%AE%B3)
- [race-conditions-fetching-data-react-with-useeffect](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)
