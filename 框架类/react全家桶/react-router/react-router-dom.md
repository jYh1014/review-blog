### 1.比较hash模式和history模式

#### 相同点

- 两种模式下 url变化都不会引起页面的重新刷新，也就是不会去重新请求服务器。
- 都适合单页应用
- 都会保存 url的历史记录栈。

#### 不同点

- 主要区别在当你重新刷新页面的时候，hash模式的请求地址只包括#之前的地址，所以即使路由没有匹配到组件，也不会出现404的情况；但是history模式的请求地址和浏览器url一致，如果服务器在这个路径下部署的没有资源，那将会出现404的情况。
- 实现原理不同，hash模式是根据监听hashChange事件来对url做监听从而匹配组件；而history模式是利用的window.history.pushState的h5的api来实现的；

#### react-router-dom和react-router的区别

- react-router提供了核心原理；react-router-dom是在react-router的基础上添加了用于跳转的Link、NavLink等组件，还有history模式下的HistoryRouter以及hash模式下的HashRouter组件。
- 开发只需要引用react-router-dom即可，它内部包含了react-router库。

### 从源码层面介绍一下两种模式

```js
<BrowserRouter>
    <Route path="/login" component={login} />
    <Route path="/home" component={home} />
</BrowserRouter>
```
#### history模式

- 利用Link等组件跳转的时候，会调用history.push方法，将当前目标地址作为参数传递进去，这个方法内部会调用window.history.pushState方法来修改浏览器的url，调用setState方法来依次执行监听函数而且还会重新创建一个location对象作为监听函数的参数。我们是在Router组件内部做的监听，当触发时会进行setState来重新渲染。然后Route组件会去进行路由匹配，将匹配到的组件渲染到页面上。
- 但是浏览器的某些行为下比如前进后退，并不会调用pushState，这个时候我们需要添加监听事件popstate
```js
window.addEventListener('popstate',function(e){
    /* 监听改变 */
    //获取当前location,然后setState
})
```

```js
//BrowserRouter组件
class BrowserRouter extends React.Component {
    history = createHistory(this.props)
    render() {
        return (
            <Router history = { this.history } children = {this.props.children}/>
        )
    }
}
```
```js
//Router

constructor(props){
    super(props)
    this.unlisten = this.props.history.listen(({ location }) => {
            //监听事件，当调用history.push方法时，会触发回调函数的执行。
            this.setState({ location })
    })
}
render() {
        let value = { history: this.props.history, location: this.state.location }
        return (
            <RouterContext.Provider value={ value }>
                {this.props.children}
            </RouterContext.Provider>
        )
    }
```
```js
//history
function push(pathname){
    let location = { pathname, state }
    window.history.pushState(state, null, pathname) //利用h5的api
    setState({action, location})
}
function setState(nextState){
    Object.assign(history, nextState)
    //触发监听
    listeners.forEach(listener => listener(nextState))
}
```
```js
//Route
render(){
    return (
      <RouterContext.Consumer>
      {context => {
      //做匹配然后渲染组件
      //return React.createElement(component, props)
      }}
      </RouterContext.Consumer>
    )
}
```
#### hash模式

- 当你通过Link组件跳转的时候，也会调用history.push方法，里面底层是修改了window.location.hash值，同样的也需要hashChange事件来监听hash值的变化，当hash值变化了，会调用回调函数，执行setState方法，从而触发Router组件内部的监听事件。
