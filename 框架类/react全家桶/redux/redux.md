### redux 相关面试题

#### redux解决了什么问题？为神马要用redux？

- 主要解决了状态层层转递的问题。
- 当我们从view层触发一个行为的时候会调用store.dispatch()这个方法，store是全局唯一的一个容器，也就是只有这一个store对象，这个对象是由createStore方法创建出来的，这个store对象里面有dispatch方法，getState方法以及subscribe等方法。
- 当调用dispatch方法时，会执行reducer方法来计算出最新的状态，如果组件想要拿到最新的状态就要绑定监听事件也就是调用subscribe方法，传入一个callback，将回调函数都缓存到一个数组里，等执行完reducer后依次执行这些callback。这样每个绑定subscribe事件的组件都能在回调里获取到最新的状态。其实就是利用的发布订阅模式。
```js
function createStore(reducer, initialState ){
    let newState = initialState
    let callbacks = []
    function dispatch(action){
        //只有派发action之后才会触发监听回调函数的执行
        newState = reducer(newState, action)
        callbacks.forEach(cb => { //发布
            cb()
        })
    }
    function subscribe(cb){ //订阅
        callbacks.push(cb)
        return function unsubscribe(){
            let index = callbacks.indexOf(cb)
            callbacks.splice(index, 1) //会修改原数组
        }
    }
    function getState(){
        return newState
    }
    const store = {
        dispatch,
        subscribe,
        getState
    }
    return store
}
```
#### 中间件

- 中间件是用来干嘛的？其实最根本的还是为了处理业务需求，比如当我们需要在真正的dispatch前后打印日志，之前的dispatch就不能满足我们的需求，我们需要增强原来的dispatch方法；还有一种情况，当我们封装好了一个公共方法fetchdata用于获取数据，调用的时候我们想直接通过store.dispatch调用，并且在真正拿到数据之后再调用dispatch（也就是处理异步操作）这样一系列的需求导致我们不得不对原有的dispatch方法进行重写。本质上中间件的实现就是重写了原来的dispatch方法。
- 中间件我觉得有点像拦截器，它位于action和reducer之间，当用户发出action之后会被拦截器拦截住进行一个一个中间件的处理，最后调用原生的dispatch，然后才到reducer。
- 中间件的实现原理
- 首先，我们要知道我们在view层调用的store.dispatch方法是重写后的dispatch方法，并不是原生的；当我们调用的时候，会被最外层中间件拦截进行处理，当处理完后执行next方法交换执行权给下一个中间件依次类推，最后一个中间件拿到的next就是原生的dispatch方法。
```js
//看一下中间件的写法redux-thunk
function thunk(middlewareAPI) {
    return function (next) {
        return function (action) {
            let { getState, dispatch } = middlewareAPI
            if(typeof action === "function"){
                return action(getState, dispatch) //此时的dispatch是重写后的
            }
            next(action)
        }
    }
}
```
- 中间件是作为applyMiddleware方法的参数传进去的,这个方法最终返回的是一个全局唯一的store对象，对象里面的dispatch方法是增强后的。首先执行createstore拿到原生的store对象取出dispatch（这个是原生的），然后遍历中间价将middlewareAPI={getState,dispatch(增强后的)}作为参数传进去执行，得到一个chain数组，本来中间件是有三层，执行过一次剩下两层。然后将chain作为参数传给compose方法，关键就是这个compose方法，它会将各个中间件像洋葱模型一样串联起来。
```js
applyMiddleware(thunk, logger)(createstore)(reducer)
```
- compose方法接收多个参数a,b，返回的是一个新的函数fn，执行这个fn，会先执行b函数，将其返回值作为a函数的参数并执行a，依次类推，最终将最外层函数（a）的返回值作为compose函数的返回值。如果我们传入的是多个中间件thunk, logger，那么最终compose函数执行结束后拿到的结果就是thunk函数的返回值function （action）{.....}，这个返回值其实就是增强后的dispatch方法。请记住每个中间件的next参数指的是上个中间件的返回值。我们调用dispatch时，会执行thunk函数返回的那个函数，里面遇到next方法调用就交换执行权给下一个中间件，下一个中间件再遇到next就继续交换执行权，直到最后一个中间件遇到next，这个时候的next已经是原生的dispatch方法，执行完最后一个中间件后会回溯到上层的中间件继续执行剩下的代码。
```js
function compose(...fns){
    return fns.reduce((a,b) => {
        return function(...args){
             return a(b(...args))
        }
    })
}
//等同于下面代码
function compose(...fns){
    return function(...args){
      a(b(c(...args)))
  }
}
```
```js
//中间件demo
function compose(...fns){
    return fns.reduce((a, b) => {
        return function(...args){
            return a(b(...args))
        }
    })
    // return function(...args){
    //     a(b(c(...args)))
    // }
}
function logger(next){
    return function(action){
        console.log("===logger开始===")
        next()
        console.log("===logger结束===")
    }
}
function thunk(next){
    return function(action){
        console.log("===thunk开始===")
        next()
        console.log("===thunk结束===")
    }
}
function promise(next){
    console.log(11)
    return function(action){
        console.log("===promise开始===")
        next()
        console.log("===promise结束===")
    }
}
function dispatch(){
    console.log("====dispatch====")
}
let result = compose(logger, thunk, promise)
let f = result(dispatch)
f({type: 'ADD'})
```
```js
//applymiddleware方法
function applyMiddleware(...middlewares){
    return function(createStore){
        return function(reducer){
            let store = createStore(reducer)
            let dispatch
            let middlewareAPI = { getState: store.getState, dispatch: action => dispatch(action)}
            let chain = middlewares.map(middleware => middleware(middlewareAPI))
            dispatch = compose(...chain)(store.dispatch) //重写后的dispatch
            return {
                ...store,
                dispatch
            }
        }
    }
}
```
#### react-redux

- redux是一个独立的库，能和vue、react等等结合使用，那怎么和react结合使用更方便，react-redux做的就是这件事。
```js
const mapDispatchToProps = dispatch => ({
    add: () => dispatch({ type: "ADD" }),
    minus: () => dispatch({ type: "MINUS" })
})
const mapStateToProps = state => ({value: state.number})
connect(mapStateToProps, mapDispatchToProps)(Counter)
```
- 在没有使用 react-redux之前，组件想要获取最新的状态都需要添加一个监听事件，当dispatch后调用监听的回调函数，我们才能获取状态，那每个组件都要这样做，react-redux就帮我们抽出公共的逻辑，使使用更加简单。也就是说组件只做业务相关的开发。
- 首先，我们是通过connect把组件和store的状态联系在一起的，connect方法返回的是一个高阶组件，高阶组件内部他获取到store对象，添加了监听事件，获取最新的状态然后通过mapStateToProps生成对象，包括mapDispatchToProps，然后将这些state和dispatch以属性的方式传递给旧的组件。
- 高阶组件内部是怎么拿到store对象的，这里利用了context的思想，在App.js组件外部又包了一层Provider容器，将store作为属性传递，这个Provider组件就是一个<Context.Provider>{this.props.children}</Context.Provider>,然后在高阶组件里面就能拿到传递的store对象了
```js
//Provider组件
render() {
        return (
            <ReactReduxContext.Provider value = {{store: this.props.value}}>
                {this.props.children}
            </ReactReduxContext.Provider>
        )
    }
```
```js
//connect方法，如果是类组件
function connect(mapStateToProps, mapDispatchToProps){
    return function(OldComponent){
        return class extends React.Component{
            static contextType = ReactReduxContext //利用context拿到store对象
            constructor(props){
                super(props)
                this.state = {}
                // this.state = mapStateToProps(this.context.store.getState())
            }
            componentDidMount(){
                this.unsubscribe = this.context.store.subscribe(()=>{
                    this.stateProps = mapStateToProps(this.context.store.getState())
                    this.setState(this.stateProps)
                })
            }
            componentWillUnmount(){
                this.unsubscribe()
            }
            render(){
                 //将state喝dispatch作为属性传递给组件
                let bindActions = bindActionCreators(mapDispatchToProps, this.context.store.dispatch)
                return (
                    <OldComponent {...this.props} {...bindActions} {...this.stateProps}/>
                )
            }
        }
    }
}
```
