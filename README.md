# Fetch Source

基于 Fetch API 封装的 SSE 请求

# Install

```sh
npm install fetch-request-source
```

可以传入所有的 Fetch API [参数](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/fetch):

```ts
const ctrl = new AbortController()
fetchSource('/api/sse', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    foo: 'bar',
  }),
  signal: ctrl.signal,
})
```

完整示例:

```ts
class FatalError extends Error {}
const statusMap = {
  400: '请求参数错误（400）',
  401: '用户未认证（401）',
  403: '权限不足（403）',
  404: '请求资源不存在（404）',
  500: '服务器内部错误（500）',
  503: '服务不可用（503）',
  //...
}
import { fetchSource, StreamContentType } from 'fetch-request-source'
const ctrl = new AbortController()
fetchSource('/api/sse', {
  // method: 'POST',
  // headers: {
  //   'Content-Type': 'application/json',
  //   'Authorization': token,
  //   //...
  // },
  //body: JSON.stringify(data),
  openWhenHidden: true,
  reconnect: false,
  signal: ctrl.signal,
  onopen(response) {
    if (
      response.ok &&
      response.headers.get('content-type') === StreamContentType
    ) {
      return
    } else if (
      response.status >= 400 &&
      response.status < 500 &&
      response.status !== 429
    ) {
      const errorMessage =
        statusMap[response.status] || `服务器错误 (${response.status})`
      throw new FatalError(errorMessage)
    }
  },
  onmessage(res) {
    //res数据根据接口返回格式解析
    // 示例: {type: 'ERROR', message: '错误信息',[...other其它字段]}
    const { type, message } = JSON.parse(res.data)
    if (type === 'ERROR' || res.event === 'FatalError') {
      // 处理错误信息，阻断请求
      throw new FatalError(message)
    }
    // 其它业务处理
  },
  onclose(res) {
    console.log(res)
    // 连接关闭时的处理
  },
  onerror(err) {
    // 错误处理
    throw err // 阻止继续请求
  },
})
```
