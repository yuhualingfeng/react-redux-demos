## 基本概念

### 使用Saga辅助函数
Saga辅助函数来自包`redux-saga/effects`,包括`takeEvery``takeLatest`
```
import {takeEvery,takeLatest} from 'redux-saga/effects'
```
+ takeEvery 可以同时进行多个异步请求
+ takeLatest 只能进行最后一次异步请求，之前有在执行中的请求会被中断


### 申明式Effects
在redux-saga中,我们把Generator函数中的yield返回的纯Javascript对象称作Effect,我们可以把Effect看做是发送给middleware的指令以执行某些操作。

我们同样可以通过`redux-saga/effects`包提供的函数来创建Effect：

```
import {call,apply,cps} from 'redux-saga/effects'
```
+ call 非常适合返回Promise结果的函数
+ apply 非常适合返回Promise结果的函数
+ cps 用来处理 Node 风格的函数 （例如，fn(...args, callback) 中的 callback 是 (error, result) => () 这样的形式，cps 表示的是延续传递风格（Continuation Passing Style））

### 发起action
在middleware(redux-saga)中，可以使用`put`来发起`action`.它和通过redux中通过dispatch发起action的作用是一致的
```
	import { call, put } from 'redux-saga/effects'

	function* fetchProducts() {
	  const products = yield call(Api.fetch, '/products')
	  // 创建并 yield 一个 dispatch Effect
	  yield put({ type: 'PRODUCTS_RECEIVED', products })
	}
```
### 错误处理
错误处理可以通过让你的API服务返回一个正常的含有错误标志的值.
```
import Api from './path/to/api'
import {call,put} from 'redux-saga/effects'

function fetchProductsApi(){
	return Api.fetch('/products')
	.then(response=>{response})
	.catch(error=>{error})
}

function fetchProducts(){
	let {response,error} = yield call(fetchProductsAPi);

	if(response){
		yield put({type:"PRODUCTS_RECEIVED",products:response});

	}esle{
		yield put({type:"PRODUCTS_REQUEST-FAILED",error:error});
	}
}
```