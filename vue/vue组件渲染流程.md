#### 组件的渲染流程
- 先创建父组件的虚拟节点，在渲染父组件的过程中遇到子组件的标签，会调用Vue.extend方法创建一个Vue的子类Sub。
```js
Vue.extend = function (extendOptions) {
        const Super = this
        const Sub = function VueComponent(options) {
            this._init(options)
        }
        Sub.cid = cid++;
        Sub.prototype = Object.create(Super.prototype) //继承Vue
        Sub.prototype.constructor = Sub
        Sub.options = mergeOptions(Super.options, extendOptions)
        Sub.components = Super.components
        return Sub
    }
```
- 在将虚拟节点转化成真实节点时，会对子组件进行初始化,并且调用mount方法进行渲染，在mount方法里会给每个子组件创建一个watcher对象，当data数据变化了，只会通知当前组件的watcher执行update。
```js
let child = vnode.componentInstance = new Ctor({})
child.$mount()
```
