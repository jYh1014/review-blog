每当使用一个组件时，都会对组件进行实例化，实例化过程中会对data进行初始化，若data是一个函数则执行这个函数将返回一个新对象（是一份独立的拷贝）挂载到组件实例上，这样当组件被复用多次时，可以保证数据互不影响。
```js
  class Vue {
      constructor(options) {
          this.data = options.data()
      }
  }
<!--   let data = () => ({ a: 10 }) -->
  let c = { //c组件
      data: { a: 10 }
  }
  // let data = () => ({ a: 10 })
  let v1 = new Vue(c) //a组件复用了c组件
  let v2 = new Vue(c) //b组件也复用里c组件
  v1.data.a = 100
  console.log(v2.data)
```
以上例子意思是a组件和b组件同时引用了相同的组件c，那a组件修改data会影响b组件的数据。
注意回答此题的时候会考虑到组件渲染的流程以及组件合并策略。
