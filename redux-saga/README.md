takeEvery 可以同时进行多个异步请求
takeLatest 只能进行最后一次异步请求，之前有在执行中的请求会被中断
call([obj, obj.method], arg1, arg2, ...)  非常适合返回Promise结果的函数
apply(obj, obj.method, [arg1, arg2, ...]) 非常适合返回Promise结果的函数
cps 用来处理 Node 风格的函数 （例如，fn(...args, callback) 中的 callback 是 (error, result) => () 这样的形式，cps 表示的是延续传递风格（Continuation Passing Style））
put 发送action
##
##疑问
###解释下Generator和Promise
 + Generator 生成器函数
 + Promise是一个异步编程的类库
###redux-saga的优点是什么？
优点：
 + redux-saga 让你可以用同步的方式写异步代码
 + 通过简单地迭代 yield 过的对象进行简单的单元测试

###redux-saga基本思路是什么？
redux-saga作为redux的中间件,当sagaMiddleware.run(saga)时,saga函数会被立即执行，执行的内容可以包含初始化异步请求，action监控,action触发.例如：redux出发一个action,saga可以进行监控,然后进行action触发和异步请求.

###如何在动态加载的子React组件中执行sagaMiddleware.run(saga)?
可以通过在组件加载成功(componentDidmount)后触发dispatch来获取组件初始数据.

###当dispatch一个action时,reducer和redux-saga中都有监控,这时会先执行哪一块的代码?
先执行reducer中的代码