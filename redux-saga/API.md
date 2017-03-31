## API手册
### 中间件API(Middleware API)
#### createSagaMiddleware(options)======================================未完待续
创建Redux中间件并连接到Redux Store.

+ `options`:`Object` 传输给中间件的选项列表.目前支持以下可选项：
    + `sagaMonitor`:如果此选项提供，中间件会把监视事件传送到监视器。

#### middleware.run(saga,...args)
动态运行saga,只能在applyMiddleware之后运行。

+ `saga`:`Function` Generator函数
+ `args`:`Array` 传递给`saga`的参数

**注意：**`saga`作为一个Generator函数，其返回值必须是一个Generator对象。
```javascript
/**saga参数示例**/
// example1
function* root(){
	yield [startup(),nextRedditChange(),invalidateReddit()] 
}

// example2
function* root() {
   yield fork(startup)
   yield fork(nextRedditChange)
   yield fork(invalidateReddit)
}

//example3
function* root(){
   yield [fork(startup),fork(nextRedditChange),fork(invalidateReddit)]
}
```

### Saga辅助函数(Saga Helpers)

#### takeEvery(pattern,saga,...args)
`takeEvery`是一个高阶函数,当redux中的action触发并与`pattern`匹配，就会执行`saga`函数。它的具体实现如下：
```javascript
	function* takeEvery(pattern,saga,...args){
		const task = yield fork(function* (){
			while(true){
				const action = yield take(pattern)
				yield fork(saga,...args.concat(action))
			}
		})
		return task
	}
```
参数说明：

+ `pattern`:`String Array Function` :匹配redux中dispatch的action
+ `saga`:`Function` :一个Generator函数
+ `...args`:`Array<any>`:作为`saga`的参数

#### takeLatest(pattern,saga,...args)
`takeLatest`和`takeEvery`一样也是高阶函数,实现的功能也基本类似，唯一不同在于：`takeLatest`会取消之前执行的所有`saga`任务.例如当用户极速点击按钮触发action,`takeEvery`可能在执行多个saga任务,`takeLatest`则只会执行最后一个saga任务.`takeLatest`的实现代码如下:
```javascript
function* takeLatest(pattern, saga, ...args) {
  const task = yield fork(function* () {
    let lastTask
    while (true) {
      const action = yield take(pattern)
      if (lastTask)
        yield cancel(lastTask) // cancel is no-op if the task has already terminated

      lastTask = yield fork(saga, ...args.concat(action))
    }
  })
  return task
}
```
#### throttle(ms,pattern, saga, ...args)
当`pattern`与action匹配时,第一次会立即执行,后面再次匹配时必须要等待上一次执行完ms秒后才执行.实现代码如下：
```javascript
function* throttle(ms, pattern, task, ...args) {
  const throttleChannel = yield actionChannel(pattern, buffers.sliding(1))

  while (true) {
    const action = yield take(throttleChannel)
    yield fork(task, ...args, action)
    yield call(delay, ms)
  }
}
```
### Effect创造器(Effect creators)

#### take([pattern])
`take`作用为暂停当前Generator直到dispatch一个与`pattern`匹配的`action`.

`pattern`参数可以省略,省略可以匹配所有的action,其类型可以是:
 
 + `String`:当字符串为'*'或者action与字符串相等即继续执行Generator.
 + `Array`:action与数组中的任意元素匹配即可继续执行Generator.
 + `Function`:action将被传入function, function的返回值为true即可继续执行Generator.
#### take.maybe(pattern)


#### take(chanel)
#### take.mybe(chanel)

#### put(action)
#### put.resolve(action)
#### put(chanel,action)
#### call(fn,...args)
#### call([context,fn],...args)
#### call(context,fn,[args])
#### cps(fn,...args)
#### cps([context,fn],...args)
#### fork(fn,...args)
#### fork([context,fn],...args)
#### spawn(fn,...args)
#### spawn([context,fn],args)
#### join(task)
创建一个Effect,指示中间件等待fork任务的结果
#### join(...tasks)
#### cancel(task)
#### select(selector,...args)
#### actionChanel(pattern,[buffer])
#### flush(channel)
#### cancelled()

### Effect组合器(Effect combinators)
#### race(effects)
#### [...effects](parallel effects)

### Interfaces
#### Task
#### Channel
#### Buffer
#### SagaMonitor

### External API
#### runSaga(iterator,options)

### Utils
#### channel([buffer])
#### eventChannel(subsrcibe,[buffer],[matcher])
#### buffers
#### delay(ms,[val])
####
