### react hooks

#### hooks是为了解决什么问题？

- hooks是结合函数组件使用的，由于函数组件内部无状态无生命周期，所以如果需要状态和生命周期等就需要用这些钩子。
- 注意事项：只能在函数组件最上面使用hook，不要在循环内或者条件判断内或者别的普通函数中使用。主要是因为每次调用hook的时候其实都是有固定对应的全局index索引值的，每次都会从对应的index里面去获取数据
```js
// 当前 hook 的全局索引
let currentIndex
// 第一次调用 currentIndex 为 0
if (Math.random() > 0.5) {
  useState('first')
}
// 第二次调用 currentIndex 为 1
useState('second') 
//第一次执行如果随机值是1的话会执行第一个hook，这个时候没问题，两个hook分别存放在hookState = [0: hook1的内容，1:hook2的内容]，但是第二次如果随机值是0，那第一个hook不会执行，只有第二个hook执行的时候会去从index为0的数组中获取state，但是它的状态本来是存放在index为1的地方，这样拿到的数据就不对了。
```

#### useState

- 给函数组件引入状态的钩子，传入初始状态，返回一个数组，数组第一项是状态，第二项是设置状态的函数
- 每次的useState都是一个独立的闭包，执行了setState后会重新render
- setState也是异步的，每次setState之后并不会立即去渲染页面，而是只会渲染一次。
- 如果setState的参数不是函数，那state将会合并，这个和类组件相同。
```js
//先点击一次alertNum，然后三秒内连续点击add
function Counter(){
     function alertNum() {
          setTimeout(() => {
            console.log(state) //会打印0，闭包的存在
          }, 3000)
    }
     function add() {
        setTimeout(() => {
          setState(state + 1)
          setState(state + 1) //state：1
        }, 3000)
      }
    return(){
        <div>
            <p>{state}</p>
            <button onClick={alertNum}> alertNum </button>
            <button onClick={add}> add </button>
        </div>
    }
}
```
```js
//简单实现一下useState原理
let lastState
function useState(initialState){
    let state = lastState || initialState
    function setState(newState){
        if (typeof newState === "function") {
            lastState = newState(lastState)
        }
        if (!Object.is(lastState, newState)) { //浅比较
             lastState = newState
             render() //重新执行函数组件
        }
    }
    return [state, setState]
}

```
```js
let hookStates = [] //存放组件所有的hooks的数据
let hookIndex = 0
function useState(initialState) {
    hookStates[hookIndex] = hookStates[hookIndex] || initialState
    let currentIndex = hookIndex
    function setState(newState) {
      hookStates[currentIndex] = newState
      render()
    }
    return [hookStates[hookIndex++], setState]
}
```
#### React.memo（性能优化点）

- 是为了解决父组件状态发生改变时，会造成子组件的重复渲染，即使没有给子组件传递属性或者传递的属性值没有发生变化。它的原理和pureComponent类似。
- 使用let MemoChild = React.memo(Child)

#### React.useCallback

- 父组件给子组件传递函数类型的属性时，由于父组件状态改变造成重新执行父函数，导致传递是一个新的函数，会造成子组件的重复渲染。参数第一项是函数，第二项是依赖值，只有依赖值变了才会重新生成一个新的函数。
```js
//当依赖值number不变时，将生成的是同一个函数。
 const handleClick = useCallback(() => setNumber(number + 1), [number])
<MemoChild handleClick={handleClick} />
```
#### React.useMemo

- 父组件给子组件传递对象类型的属性时，由于父组件状态改变造成重新执行父函数，导致传递的是一个新的对象，会造成子组件的重复渲染。
```js
let data = useMemo(()=>({number}),[number]) //缓存对象
<MemoChild handleClick={handleClick} data={data} />
```
#### React.useRef

- 和类组件的React.createRef()类似，不同点是每次createRef都会返回一个新的对象，而useRef会返回同一个对象,返回值也是{current: xxx}

#### React.useEffect

- 作用和类组件的生命周期相似，第一个参数是一个函数，会在组件渲染后执行，如果这个函数有返回值，那这个返回的函数将会在下次执行useEffect之前先执行，第二个参数是依赖项，依赖项改变了才会执行useEffect，依赖项是空数组则只会执行一次，依赖项不写则会一直执行。
- 场景：请求数据，修改dom以及设置定时器
- 也要注意闭包的问题。

#### React.useContext

- 作用和类组件的context相同
```js
let Context = createContext()
//父组件
 <Context.Provider value={{ a, b }}>
      <Counter />
 </Context.Provider>
//子组件
let { a, b } = useContext(Context)
``` 