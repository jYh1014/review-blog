### 组件的渲染流程

#### 注册全局组件Vue.component("aa")
- 其实调用的是Vue.extend,然后把返回结果赋值给Vue.options.components上
- Vue.extend方法内部创建了一个子类Sub来继承父类Vue
```js
Vue.extend = function(extendOptions){
  const Super = this
  const Sub = function VueComponent(options) {
      this._init(options)
  }
  Sub.cid = cid++;
  Sub.prototype = Object.create(Super.prototype) //Sub继承Vue
  Sub.prototype.constructor = Sub
  Sub.options = mergeOptions(Super.options, extendOptions)
  Sub.components = Super.components
  return Sub
}
```
- 当父组件使用全局组件时，先从父组件开始创建vnode，在创建过程中会遇到子组件，从Vue.options.components中根据名字获得子组件Ctor，然后生成相应的vnode
- 在创建真实dom时，先渲染的是父组件，在渲染子组件的时候会调用new Ctor来创建组件实例，实例化会调用this._init(options)方法（和new Vue的初始化过程一样），让组件开始初始化接着调用$mount方法进行挂载。

- 在mount方法里会给每个子组件创建一个watcher对象，当data数据变化了，只会通知当前组件的watcher执行update。

#### 局部组件

- 涉及到组件components的合并策略，采用的是就近原则，先从局部组件上获取，若没有再从全局组件上获取。
- 具体实现上其实是把全局组件放到了组件对象的原型链上了，也就是先从自身组件对象上获取，若没有再从原型上获取
```js
//组件合并策略
strats.components = function (parentValue, childValue) {
    let res = Object.create(parentValue)
    if (childValue) {
        for (let key in childValue) {
            res[key] = childValue[key]
        }
    }
    return res
}
```

   
