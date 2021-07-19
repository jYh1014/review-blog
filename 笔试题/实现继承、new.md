### new 做了四件事
```js
let obj = new Object()
```
- 创建了一个新对象
- 将新对象继承父构造函数
- 调用父构造函数，并将this指向新对象
- 返回新对象

```js
function Parent(options){
    this.type = "animal"
    this.name = options
}
function myNew(ParentProto, ...args){
    let obj = {}
    obj.__proto__ = ParentProto.prototype;
    ParentProto.apply(obj, args)
    return obj
}
let p = myNew(Parent, 11)
// let p = new Parent()
console.log(p)
```
### 实现继承(寄生组合式)
```js
function Parent(){
    this.type = "animal"
}
Parent.prototype.say = function(){
    console.log(this.type)
}

function Child(){
    Parent.call(this)
    this.name = "tiger"
}
Child.prototype = Object.create(Parent.prototype)
Child.prototype.constructor = Child
let c = new Child()
console.log(Child.prototype.constructor)
console.log(c.say())
```
- 利用call来实现拥有父构造函数本身的属性和方法
- 利用Object.create来实现原型链的继承，以拥有父构造函数原型上的属性和方法
- 
```js
function create(ParentProto, Child){
    function fn(){

    }
    fn.prototype = ParentProto
    fn.prototype.constructor = Child
    return new fn()
}
Child.prototype = create(Parent.prototype, Child)
// Child.prototype.constructor = Child
let c = new Child()
console.log(Child.prototype.constructor)
console.log(c.say())
```
