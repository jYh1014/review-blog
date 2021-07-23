#### watch
- 用来监听数据的变化，监听的是已经挂载到vue实例上的数据，一般适用于一个数据会影响多个数据的场景
- computed是计算属性，计算一个新的属性，并将该属性挂载到vue实例上，一般适用于一个数据受多个数据影响的场景
```js
  watch:{
      "a": {
          handler(newValue, oldValue){
              console.log(newValue, oldValue)
          },
          sync: true
      }
  }
```
- 当数据改变时，这里假如a并没有被模版使用，那就是说不能通过get方法来进行依赖收集，我们怎么才能监听到这个a什么时候改变了呢？只能通过取值的方式，然后才能进行依赖收集
- 这里如果配置了watch属性，将会调用Vue.prototype.$watch方法生成一个watcher对象
```js
function createWatcher(vm, exprOrFn, handler, options) { //exprOrFn就是key，也就是说是个字符串
    if (typeof handler == "string") {
        handler = vm[handler]
    }
    if (typeof handler == "object") {
        options = handler
        handler = handler.handler
    }
    return vm.$watch(exprOrFn, handler, options)
}
 Vue.prototype.$watch = function (exprOrFn, cb, options) {
      let watcher = new Watcher(this, exprOrFn, cb, { ...options, user: true })
      if (options && options.immediate) {
          cb()
      }
  }
```
- 当exprOrFn不是函数的时候，就不能用以前那套逻辑
```js
class Watcher {
   if (typeof exprOrFn === "function") {
            this.getter = exprOrFn
    } else {
        //对$watch的key，只有去取值的时候才会进行依赖收集。
        this.getter = function () {
            let path = exprOrFn.split(".")
            let obj = vm
            for (let i = 0; i < path.length; i++) {
                obj = obj[path[i]]
            }
            return obj
        }
    }
}
```
- 每当依赖的key发生改变时，就会执行这个getter方法返回最新的值，然后调用handler将新旧值传递进去。

#### computed
```js
<div>{{fullname}}</div>
data: {firstname: "张",lastname: "三"}
computed: {
    fullname: function () {
        console.log("computed会执行吗")
        return this.firstname + this.lastname
    }
}
console.log(vm.fullname)
console.log(vm.fullname) //这一次将不会执行computed，
```
- 只有当fullname依赖的值firstname和lastname有变化才会执行computed，并且是第一次读取fullname的值，多次读取将取之前的缓存不会去重新计算。
- 当配置了computed属性时，会给computed的每个新属性都生成一个新的watcher对象，而且也用defineProperty来对这个新属性做了监听，当读取新属性的值时，会判断当前watcher.dirty是否为true，
true的话就会进行重新计算调用watcher的evaluate方法，
接着执行computed，在执行过程中会获取其依赖值firstname和lastname，这样依赖值也会被监听，计算完值后将dirty赋值为false，下次再来读取将不会重新计算，会直接返回老的值。后面当你修改了firstname或者lastname的时候，
会执行watcher的update方法，
在执行过程中会获取其依赖值firstname和lastname，这样依赖值也会被监听，也就是说计算属性watcher也会订阅firstname和lastname属性，计算完值后将dirty赋值为false，下次再来读取将不会重新计算，会直接返回老的值。
以后当你修改了firstname或者lastname的时候，会执行watcher的update方法，
当前若为计算属性的watcher，则不会去渲染页面，而是将dirty赋值为true，当下次取值的时候，就会重新进行计算。
```js
function initComputed(vm) {
    let computed = vm.$options.computed
    let watchers = vm._computedWatchers = {}
    for (let key in computed) {
    //为所有的计算属性都添加监听，并且生成一个新的watcher
        const userDef = computed[key]
        const getter = typeof userDef === "function" ? userDef : userDef.get
        watchers[key] = new Watcher(vm, getter, () => { }, { lazy: true })
        defineComputed(vm, key, userDef)
    }
}
function defineComputed(target, key, userDef) {
    let sharedPropertyDefinition = {
        enumerable: true,
        configurable: true,
        get: () => { },
        set: () => { }
    }
    if (typeof userDef === "function") {
        sharedPropertyDefinition.get = createComputedGetter(key)
    } else {
        sharedPropertyDefinition.get = createComputedGetter(key)
        sharedPropertyDefinition.set = userDef.set
    }
    Object.defineProperty(target, key, sharedPropertyDefinition)
}
function createComputedGetter(key) {
    return function () {
      let watcher = this._computedWatchers[key]
      if(watcher){
          if(watcher.dirty){
          //为true才会重新计算
              watcher.evaluate()
          }
          if(Dep.target){ //如果计算结束后，Dep.target还有值，此时Dep.target就是渲染watcher，那就将渲染watcher也订阅计算属性依赖的dep
              watcher.depend()
          }
          //当dirty为false
          return watcher.value
      }
  }
}
```
  - 现在还有一个问题没有解决，当我修改了firstname和lastname的时候，只有计算属性的watcher进行了订阅，而负责渲染的watcher并没有订阅，那视图就不能做到响应式更新。我们可以把计算属性watcher订阅的dep让渲染watcher也订阅，
  这样firstname和lastname变化的时候，视图就能及时更新。注意，渲染watcher和计算属性watcher的存在关系，是先有的渲染，然后才有的计算属性，接着计算属性watcher先执行结束，最后是渲染watcher执行结束。
  
  -
