### 介绍



微信小程序网络请求工具，包含以下功能：

1. 请求拦截器
2. 响应拦截器
3. 并发请求
4. 本地资源上传等
5. 其他...



### 安装



```shell
npm install mina-request
```



> 📌 构建 npm：
>
>    在安装后，需要要在微信开发者工具，点击 `工具` ➡️ `构建 npm`



<br />

### 实例



#### 创建实例

您可以使用自定义配置新建一个实例

```js
import WxRequest from 'mina-request'

// 对 WxRequest 进行实例化
const instance = new WxRequest({
  baseURL: 'https://some-domain.com/api/', // 使用时请换成真实接口
  timeout: 1000, // 超时时长
  isLoading: false // 是否使用默认的 loading 效果
})

```



#### 实例方法

以下是可用的实例方法。指定的配置将与实例的配置合并。

> 📌config 对象属性值和 wx.request() 方法调用时传递的参数保持一致

```js
instance.request(config)
instance.get(url, data, config)
instance.delete(url, data, config)
instance.post(url, data, config)
instance.put(url, data, config)
instance.all(...promise) // 传入一个个 Promise 即可
instance.upload(url, filePath, name, config) // 将本地资源上传到服务器

```



#### 代码示例

```js
// request 实例方法
instance.request({
  url: '/path',
  method: 'POST'
})

// get 实例方法，
instance.get('/path') // 不需要请参数，不需要自定义请求配置
instance.get('/path', { id: 123 }) // 不需要自定义请求配置
instance.get('/path', { id: 123 }, { timeout: 15000 }) // 需要请求参数，也需要自定义请求配置
instance.get('/path', null, { timeout: 15000 }) // 不需要请求参数，但需要自定义请求配置

```





<br />

### 请求配置



这些是创建请求时可以用的配置选项。如果没有指定 `method`，请求将默认使用 `GET` 方法。



```js
{
  baseURL: '', // 请求基准路径，方便进行接口的统一管理
  url: '', // 是用于请求的服务器 URL
  method: 'GET', // method是创建请求时使用的方法，默认值是 GET
  headers: {}, // 请求头
  data: {}, // 请求参数
  timeout: 60000, // 默认的超时时长，小程序默认的超时时长是 1 分钟
  isLoading: true // 控制是否使用默认的 loading，默认值是 true 表示使用默认的 loading
    
  // ... 📌 其他属性值和 wx.request() 方法调用时传递的参数保持一致
}

```





<br />

### 默认配置

您可以指定默认配置，它将作用于每个请求。



#### 全局默认值

```js
instance.defaults.baseURL = 'https://some-domain.com/api/'
instance.defaults.header['token'] = token
instance.defaults.isLoading = fasle
```



#### 自定义实例默认值

```js
// 创建实例时配置默认值
const instance = new WxRequest({
  baseURL: 'https://some-domain.com/api/'
})

// 创建实例后修改默认值
instance.defaults.header['token'] = token

```





#### 配置的优先级



实例方法配置(2) > 创建实例时配置项(1) > 默认配置(0)



```js
// 默认配置，优先级 0
{
  timeout: 60000 // 默认的超时时长，小程序默认的超时时长是 1 分钟
}

// 创建实例时配置项，优先级 1
const instance = new WxRequest({
  timeout = 2500
})

// 调用实例方法，优先级 2
instance.get('/longRequest', {
  timeout: 5000
})

```





<br />

### 拦截器



在请求之前新增、修改参数

在响应以后进行逻辑判断、处理



```js
// 添加请求拦截器
instance.interceptors.request = (config) => {

  // 在发送请求之前做些什么
  return config
}

// 添加响应拦截器
instance.interceptors.response = (response) => {

  // response.isSuccess = true，代码执行了 wx.request 的 success 回调函数
  // response.isSuccess = false，代码执行了 wx.request 的 fail 回调函数
    
  // response.statusCode // http 响应状态码
    
  // response.config // 网络请求请求参数
    
  // response.data 服务器响应的真正数据
    
  // 对响应数据做点什么
  return response
}

```



<br />

### 本地资源上传



将本地资源上传到服务器.



```js
/**
 * @description upload 实例方法，用来对 wx.uploadFile 进行封装
 * @param {*} url 文件的上传地址、接口地址
 * @param {*} filePath 要上传的文件资源路径
 * @param {*} name 文件对应的 key
 * @param {*} config 其他配置项
 */

upload(url, filePath, name = 'file', config = {}) {
  return this.request(
    Object.assign({ url, filePath, name, method: 'UPLOAD' }, config)
  )
}

// 使用示例
await instance.upload('/path', filePath, name, { ... })
                                   
```



<br />

### 并发请求



```js
/**
 * @description 处理并发请求
 * @param  {...promise} promise 传入的每一项需要是 Promise
 * @returns Promise
 */
all(...promise) {
  // 那么展开运算符会将传入的参数转成数组
  return Promise.all(promise)
}

// 使用示例
await instance.all(promise1, promise2, promise3)

```



<br />

### 完整示例



导入 `mina-request` ，进行网络请求统一配置



```js
import WxRequest from 'mina-request'

// 对 WxRequest 进行实例化
const instance = new WxRequest({
  baseURL: 'https://some-domain.com/api/',
  timeout: 15000,
  isLoading: false // 不使用默认 loading
})

// 添加请求拦截器
instance.interceptors.request = (config) => {
  // 在请求发送之前做点什么……

  // 新增请求头
  const token = wx.getStorageSync('token')
  if (token) {
    config.header['token'] = token
  }

  return config
}

// 添加响应拦截器
instance.interceptors.response = (response) => {
  // 对响应数据做点什么

  // 从 response 中解构 isSuccess
  const { isSuccess, data } = response

  // 如果 isSuccess 为 false，说明执行了 fail 回调函数
  // 这时候就说明网络异常，需要给用户提示网络异常
  if (!isSuccess) {
    wx.showToast({
      title: '网络异常，请稍后重试 !'
    })

    return response
  }
    
  // 对业务状态码进行
  // ...

  // 将服务器响应的数据返回
  return data
}


export default instance

```



导入实例，使用实例提供的方法

```js
// 导入封装的 网络请求模块实例
import http from '../utils/http'

/**
 * @description 用来获取首页轮播图数据
 */
export const reqSwiperData = () => http.get('/index/findBanner')

```



