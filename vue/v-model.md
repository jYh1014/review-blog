#### v-model

- 默认情况下是@input和:value的语法糖。
```js
<input v-model="a" />
<input v-bind:value="a" v-on:input="a = $event.target.value" />
```
当用在组件上时
```js
//父组件
<currency-input v-model="price"></currentcy-input>
<currency-input :value="price" @input="price = arguments[0]"></currency-input>

//子组件
<div>{{value}}</div>

props:{
    value: String
},
methods: {
  test1(){
     this.$emit('input', '小红')
  },
}
```
- 如果某些情况下，比如checkbox等需要的不是value而是checked，事件不是input而是change，那我们也可以使用v-model，只不过代表的是checked和change的语法糖
- 2.2版本以后，我们可以自定义选项来配置prop/event
```js

<my-checkbox v-model="foo"></my-checkbox>
 
Vue.component('my-checkbox', {
  template: `<input 
        type="checkbox"
        @change="$emit('balabala', $event.target.checked)"
        :checked="checked"
       />`,
 props: ['checked'],
 model: {
  prop: 'mychecked',
  event: 'myaaa'
 }
```
