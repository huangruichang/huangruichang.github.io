# React 中 render props 和 onRef 比较

## 前言

笔者在最近接触了 React 的 render props 和 onRef 写法，感觉两种写法功效各有异同。

## render props

render props 是指通过函数作为 props 的传入以达到 React 组件间代码共享的技术。例子如下：

```
....
<DataProvider render={data => (
	<h1>Hello {data.target}</h1>
)}/>
....

class DataProvider extends React.Component  {
    render() {
		return (
			<div>{this.props.render(this.state)}</div>
		)
	}
}
```

这种写法把组件原来的 render 方法交到外部决定，组件本身仅仅提供自己的内部 state，像样例代码一样，只是一个 DataProvider。通过 render props，我们可以直接在传入函数中进行通信处理。
在实现 Tab，Panel，Container，Dialog 等内容不定的组件时，可以考虑使用 render props。

## onRef

父组件需要主动调用子组件方法时，就需要使用 onRef 的写法。

```
/* Child.js */
import React from 'react'

class Child extends React.Component {
  componentDidMount() {
    this.props.onRef(this)
  }
  componentWillUnmount() {
    this.props.onRef(undefined)
  }
  method() {
    window.alert('do stuff')
  }
  render() {
    return <h1>Hello World!</h1>
  }
}

export default Child;
```

```
/* Parent.js */
import React from 'react'
import Child from './Child'

class Parent extends React.Component {
  onClick = () => {
    this.child.method() // do stuff
  }
  render() {
    return (
      <div>
        <Child onRef={ref => (this.child = ref)} />
        <button onClick={this.onClick}>Child.method()</button>
      </div>
    );
  }
}
```

在父组件可以获取到子组件的 this，所以父组件可以随意获取子组件所有数值和调用所有方法。当不想通过 props 里传入 onXXX 方法获取子组件数据的时候，就可以使用 onRef 了。

## 两者异同

相同点：
两者都是通过组件内**回调**外部传入函数，达到把逻辑开放给父组件的目的。

不同点：

1.  render props 是侧重点是渲染，onRef 侧重点是获取组件实例所有数值和方法
2.  render props 的回调在组件的 render 方法，onRef 的回调在 constructor、componentDidMount 等可以获取到 this 引用的地方

## 参考

https://reactjs.org/docs/render-props.html

https://github.com/kriasoft/react-starter-kit/issues/909
