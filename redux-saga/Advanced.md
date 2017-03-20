##高级概念

###监听未来的action
前面已经介绍了`takeEvery`是高阶函数,其实现的低阶函数中包括`take`,`take` 会暂停当前Generator,直到匹配的action被触发才会继续往下执行.很明显`take`相对`takeEvery`会更加灵活,下面两段代码实现的功能相同:
```
	import {select,takeEvery} from 'redux-saga/effects'
	function* watchAndLog(){
		yield takeEvery("*",function* logger(action){
		const state = yield select()

		console.log('action',action)
		console.log('state after',state)
		})

	}
```

```
import {select,take} from 'redux-saga/effects'

function* watchAndLog(){
	while (true){
		const action = yield take("*")
		const state = yield select()

		console.log('action',action)
		console.log('state after',state)
	}
}
```

###非阻塞调用(Non-blocking calls)
前面提到的`take`,`call`会阻塞Generator的执行,`fork`可以实现和`call`一样的作用,但其执行是非阻塞的,在执行`fork`参数中的函数的同时,Generator会继续往下执行,如果需要取消`fork`任务,则可以使用`cancel`.下面通过代码演示:
```
import { take, call, put, cancelled } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
    yield call(Api.storeItem, {token})
    return token
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  } finally {
    if (yield cancelled()) {  // 假如在执行中此task被cancel,则执行下面的代码
      // ... put special cancellation handling code here
    }
  }
}

function* loginFlow() {
  while (true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    // fork return a Task object
    const task = yield fork(authorize, user, password)
    const action = yield take(['LOGOUT', 'LOGIN_ERROR'])
    if (action.type === 'LOGOUT')
      yield cancel(task)
    yield call(Api.clearItem, 'token')
  }
}
`loginFlow`实现了一个登陆，登出功能.


###并行执行任务(Running Tasks In Parallel)
通过yield数组(数组中为执行的任务),可以并行执行任务.
```
import { call } from 'redux-saga/effects'

// correct, effects will get executed in parallel
const [users, repos]  = yield [
  call(fetch, '/users'),
  call(fetch, '/repos')
]
```
###在多个Effects中竞赛(Start a race between multiple Effects)

使用`race`Effect可以让多个Effects进行比赛.
```
import {race,take,put} from 'redux-saga/effects'
import {delay} from 'redux-saga'

function* fetchPostsWithTimeout(){
  const {posts,timeout} = yield race({
    posts:call(fetchApi,'/posts'),
    timeout:call(delay,1000)
  })
  if(posts)
    put({type:'POST_RECEIVED',posts})
  else
    put({type:'TIMEOUT_ERROR'})
}
```
当然`race`还有一个用处是自动中断在比赛中失败的Effects.
```
import {race,take,put} from 'redux-saga/effects'

function* backgroundTask(){
  while(true){
    ...
  }
}

function* watchStartBackgroundTask(){
  while(true){
    yield take('START_BACKGROUND_TASK')
    yield race({
      task:call(backgroundTask),
      cancel:take('CANCEL_TASK')
    })

  }
}
```

###通过`yield*` 依次执行Sagas(Sequencing Sagas via yield*)
```
function* playLevelOne(){ ... }
function* playLevelTwo(){ ... }
function* playLevelThree(){ ... }

function* game(){
  const score1 = yield* playLevelOne()
  yield put(showScore(score1))

  const score2 = yield* playLevelTwo()
  yield put(showScore(score2))

  const score3 = yield* playLevelThree()
  yield put(showScore(score3))
}
```
###构成Sagas(Composing Sagas)
当使用`yield*`来构成sagas时,其存在某些限制:

+ 在同一时间只能执行一个Generator函数
+ 需要单独测试嵌套的Generator函数

你可以简单使用yield来同步开始一个和多个子任务.
```
  function* fechPosts(){
    yield put(actions.requestPosts())
    const products = yield call(fetchApi,'./products')
    yield put(actions.receivePosts(products))
  }

  function* watchFetch(){
    while(yield take(FETCH_POSTS)){
      yield call(fetchPosts)
    }
  }
```

通过`yield`数组可以同步执行所有子Generator并返回它们的执行结果.
```
  function* mainSaga(getState){
    const results = yield [call(task1),call(task2),...]
    yield put(showResults(results))
  }
```
###任务取消(Task cancellation)
```
import { take,put,call,fork,cancel,cancelled } from 'redux-saga/effects'
import { delay } from 'redux-saga'
import {someApi , actions} from 'somewhere'

function* main(){
  while(yield take(START_BACKGROUND_SYNC)) {

    const bgSyncTask = yield fork(bgSync)
    yield take(STOP_BACKGROUND_SYNC)
    yield cancel(bgSyncTask)

  }
}

function* bgSync(){
  try{
    while(true){
      yield put(actions.requestStart())
      const result = yield call(someApi)
      yield put(action.requestSuccess(result))
      yield call(delay,5000)
    }
  } finnally{
      if(yield cancelled())
        yield put(acitons.requestFailure('Sync cancelled!'))
  }
}

```
上面这段代码演示了如和取消执行的任务,`main` Generator函数作为Saga,一旦触发`START_BACKGROUND_SYNC`action就进行异步请求随后等待触发`STOP_BACKGROUND_SYNC`,然后取消异步任务`bgSync`.假如`bgSync`中还存在嵌套的子task,其也会被一起取消.

如果我们需要测试`main` Generator函数,需要用到`createMockTask`函数来模拟任务.
```
  describe('main', () => {
  const generator = main();

  it('waits for start action', () => {
    const expectedYield = take(START_BACKGROUND_SYNC);
    expect(generator.next().value).to.deep.equal(expectedYield);
  });

  it('forks the service', () => {
    const expectedYield = fork(bgSync);
    expect(generator.next().value).to.deep.equal(expectedYield);
  });

  it('waits for stop action and then cancels the service', () => {
    const mockTask = createMockTask();

    const expectedTakeYield = take(STOP_BACKGROUND_SYNC);
    expect(generator.next(mockTask).value).to.deep.equal(expectedTakeYield);

    const expectedCancelYield = cancel(mockTask);
    expect(generator.next().value).to.deep.equal(expectedCancelYield);
  });
  });
```

在某些情况下task可以自动取消:
 
+ 使用`race` effect
+ 平行的effect. 例如:yield [...]

###redux-saga的fork模式(redux-saga's fork model)
在redux-saga中你可以动态在后台fork task通过使用下面两个Effects

+ `fork`用来创建附加到父级的forks
+ `spawn`用来创建与父级分离的forks

####附加到父级的forks(使用`fork` Effect)
#####完成(Completion)
发生下面情况Saga被终止：

+ 所有附加的fork是执行完成
+ 终止自己的指令体

我们来看下下面这段代码:
```
import { delay } from 'redux-saga'
import { fork, call, put } from 'redux-saga/effects'
import api from './somewhere/api' // app specific
import { receiveData } from './somewhere/actions' // app specific

function* fetchAll() {
  const task1 = yield fork(fetchResource, 'users')
  const task2 = yield fork(fetchResource, 'comments')
  yield call(delay, 1000)
}

function* fetchResource(resource) {
  const {data} = yield call(api.fetch, resource)
  yield put(receiveData(data))
}

function* main() {
  yield call(fetchAll)
}
```
`call(fetchAll)`执行完成的前提是两个`fork` Effects和`call(delay, 1000)`都已完成.

细心的朋友可能注意到实现`call(fetchAll)`可以通过平行Effects来实现
```
function* fetchAll(){
  yield [
    call(fetchResource,'user')
    call(fetchResource,'comments')
    call(delay,1000)
  ]
}
```
确实,这两种实现方式达到了同样的效果
#####错误传播(Error propagation)
我们来解释下平行Effects如何处理错误
```
yield [
  call(fetchResource,'user'),
  call(fetchResource,'comments'),
  call(delay,1000)
]
```
假如三个子effect中任意一个失败那么这个主effect就会失败,此外其他的正在处理的子effect也会被取消.
同样对于附加forks,当执行主体抛出错误或者某个`fork`抛出错误时Saga就会终止.
可以在主effect中捕获异常,因为我们使用了阻塞调用
```
function* main() {
  try {
    yield call(fetchAll)
  } catch (e) {
    // handle fetchAll errors
  }
}
```
**注意：不能从`fork` effect中去捕获异常,附加的fork中的失败将导致其父进程中止**

####与父级分离的forks(使用`spawn` Effect)
取消主effect不会取消`spawn`Effect;`spawn`Effect发生异常也不会冒泡到主effect

###将sagas与外部输入/输出通信
上面我们提到的都是通过redux中间件来使用redux-saga,redux-saga其实还提供了在中间件之外来控制输入输出.
```
import {runSaga} from 'redux-saga'

function* saga(){ ... }

const myIO = {
  subscribe:..., // this will be used to resolve take Effects
  dispatch:...,  // this will be used to resolve put Effects
  getState:...   // this will be used to resolve select Effects
}

runSaga(saga(),myIO)

```
###Using Channels
目前我们知道通过`take`,`put`我们可以与Redux Store进行交互.Channels可以实现与外部事件源或者sagas之间进行交互。通过这个章节我们可了解到:

+ 如何通过`yield eventChannel`Effect 来缓冲来具体的actions
+ 如何通过`eventChannel`工厂函数来连接`take` Effects 和外部事件源.
+ 如何使用通用的`channel`工厂函数来创建channel.并将它用在`take/put`Effects中来实现saga之间的通信。


####使用`actionChannel` Effect
首先我们来看一段代码:
```
import{take,fork,...} from 'redux-saga/effects'
function* watchRequests(){
  while(true){
    const {payload} = yield take('REQUEST')
    yield fork(handleRequest,payload)
  }
}

function* handleRequest(payload){ ... }
```
这里当触发 `REQUEST` action就会执行handleRequest任务,当`REQUEST`触发太频繁时就可能存在同一时间在执行多个handleRequest.
假如我们希望当`REQUEST` action连续触发时,handleRequest能够依次排队执行应该怎么处理呢?这里我们提供了`actionChannel`可以解决此问题.
```
import {take,actionChannel,call,...} from 'redux-saga/effects'

function* watchRequests(){
  const requestChan = yield actionChannel('REQUEST')
  while(true){
    const {payload} = yield take(requestChan)

    yield call(handleRequest,payload)
  }
}

function* handleRequest(payload){
    ...
}

```
默认情况下,`actionChannel`会无限制的接受排队action,你可以通过buffers来做一些限制.比如下面代码只接受并处理最近的5条actions。
```
import { buffers } from 'redux-saga'
import {actionChannel} from 'redux-saga/effects'

```
function* watchRequests(){
  const requestChan = yield actionChannel('REQUEST',buffers.sliding(5))
  ...
}
####使用`eventChannel`工厂来连接外部事件
```
import {eventChannel,END} from 'redux-saga'

function countdown(secs){
  return eventChannel(emitter=>{
    const iv = setInverval(()=>{
      secs-=1
      if(secs>0){
        emitters(secs)
      }else{
        emitter(END)
        clearInterval(iv)
      }
    },1000);
    return ()=>{
      clearInterval(iv)
    }

  });
}
```
`eventChannel`的第一参数是订阅者(subscriber)函数,它的角色是初始化外部事件源(上面使用了`setInterval`).然后通过调用提供的emitter将所有的输入事件从源路由到channel,在上面例子中我们每秒调用一次`emitter`(即take能够捕获到action的触发),这里返回了一个取消订阅的函数，当事件源执行完成便会通过它取消订阅

注意:你需要对事件源进行校验，不能传入null或者undefined,虽然可以传入数字，但是我们还是建议传入像redux action一样的数据.

接下来我们看看如何在saga中使用这个channel.
```
import {take,put,call} from 'redux-saga/effects'
import {eventChannel,END } from 'redux-saga'

export function* saga(){
  const chan = yield call(countdown,value)
  try{
    while(true){
      let seconds = yield take(chan)
      console.log(`countdown:${seconds}`)
  }
  }finally{
    console.log('countdown terminated')
  }
}
```
假如我们想提前结束事件订阅，可以使用`chan.close()`.
```
import { take, put, call, cancelled } from 'redux-saga/effects'
import { eventChannel, END } from 'redux-saga'

// creates an event Channel from an interval of seconds
function countdown(seconds) { ... }

export function* saga() {
  const chan = yield call(countdown, value)
  try {    
    while (true) {
      let seconds = yield take(chan)
      console.log(`countdown: ${seconds}`)
    }
  } finally {
    if (yield cancelled()) {
      chan.close()
      console.log('countdown cancelled')
    }    
  }
}
```

####使用`channel`进行在sagas间交互
前面我们通过`actionChannel`实现了限制某个任务可同时执行的次数.下面我们实现限制某个时刻可同时执行n个任务.
```
import { channel } from 'redux-saga'
import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  // create a channel to queue incoming requests
  const chan = yield call(channel)

  // create 3 worker 'threads'
  for (var i = 0; i < 3; i++) {
    yield fork(handleRequest, chan)
  }d

  while (true) {
    const {payload} = yield take('REQUEST')
    yield put(chan, payload)
  }
}

function* handleRequest(chan) {
  while (true) {
    const payload = yield take(chan)
    // process the request
  }
}
```



