#### instanceof

```js
function Fn(){
    this.name = "jyy"
}
let fn = new Fn()
```
> 检测构造函数Fn的Prototype属性是否存在于实例对象fn的原型链上。

```js
function myInstanceof(instance, ParentProto){
    let proto = instance.__proto__
    while(true){
        if(proto == null) return false;
        if(proto === ParentProto.prototype) return true
        proto = proto.__proto__;
    }
}
console.log(myInstanceof(fn, Fn))
```
