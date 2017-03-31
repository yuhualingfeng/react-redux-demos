[基本概念](/redux-saga/Basic.md)

[高级概念](/redux-saga/Advanced.md)

[API](/redux-saga/API.md)

## 疑问
### 解释下Generator和Promise

 + Generator 生成器函数
 + Promise是一个异步编程的类库

### redux-saga的优点是什么？

 + redux-saga 让你可以用同步的方式写异步代码
 + 通过简单地迭代 yield 过的对象进行简单的单元测试

### redux-saga基本思路是什么？
redux-saga作为redux的中间件,当sagaMiddleware.run(saga)时,saga函数会被立即执行，执行的内容可以包含初始化异步请求，action监控,action触发.例如：redux出发一个action,saga可以进行监控,然后进行action触发和异步请求.

### 如何在动态加载的子React组件中执行sagaMiddleware.run(saga)?
可以通过在组件加载成功(componentDidmount)后触发dispatch来获取组件初始数据.

### 当dispatch一个action时,reducer和redux-saga中都有监控,这时会先执行哪一块的代码?
先执行reducer中的代码