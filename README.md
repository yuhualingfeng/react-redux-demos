##介绍

###动机

管理不断变化的state非常困难，Redux试图让`state`的变化变得可预测

###核心概念

强制使用`action`来描述所有变化带来的好处是可以清晰地知道应用中到底发生了什么.为把`action`和`state`串起来,开发一些函数，这就是`reducer`.`reducer`只是一个接受`state`和`action`并返回新的`state`的函数。当state的数量逐渐变多时，我们很难进行管理，因此我们把每一个state拆分，每个reducer管理一个state,最后用一个reducer将这些小的reducer结合起来.
```
function visibilityFilter(state = 'SHOW_ALL', action) {
  if (action.type === 'SET_VISIBILITY_FILTER') {
    return action.filter;
  } else {
    return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    return state.concat([{ text: action.text, completed: false }]);
  case 'TOGGLE_TODO':
    return state.map((todo, index) =>
      action.index === index ?
        { text: todo.text, completed: !todo.completed } :
        todo
   )
  default:
    return state;
  }
}
function todoApp(state = {}, action) {
  return {
    todos: todos(state.todos, action),
    visibilityFilter: visibilityFilter(state.visibilityFilter, action)
  };
}

```
###三大原则

####单一数据源

整个应用的 `state` 被储存在一棵 `object tree`中，并且这个 object tree 只存在于唯一一个 `store` 中

####State 是只读的

唯一改变state的方法就是触发`action`,`action`是一个用于描述已发生事件的普通对象.

####使用纯函数来执行修改

为了描述 `action` 如何改变 `state tree` ，你需要编写 `reducers`。	

###先前技术
Redux 是一个混合产物。它和一些设计模式及技术相似，但也有不同之处。
+ Flux(Redux 的灵感来源于 Flux 的几个重要特性)
+ Elm（一种函数式编程语言）
+ Immutable
+ Baobab
+ Reactive Extensions

###生态系统
+ [Awesome Redux ](https://github.com/xgrommx/awesome-redux)是一个包含大量与 Redux 相关的库列表
+ [React-Redux Links](https://github.com/markerikson/react-redux-links)React、Redux、ES6 的高质量文章、教程、及相关内容列表
+ [Redux Ecosystem Links ](https://github.com/markerikson/redux-ecosystem-links) Redux 相关库、插件、工具集的分类资源
+ [更多](http://cn.redux.js.org/docs/introduction/Ecosystem.html)