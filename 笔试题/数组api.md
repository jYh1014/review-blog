#### 1.将数组转化为树形结构

        let treeData = [
            {
                id: "1",
                parentId: 0,
                name: "节点1"
            },
            {
                id: "1-1",
                parentId: 1,
                name: "节点1-1"
            },
            {
                id: "1-2",
                parentId: 1,
                name: "节点1-2"
            },
            {
                id: "2",
                parentId: 0,
                name: "节点2"
            },
            {
                id: "2-1",
                parentId: "2",
                name: "节点2-1"
            },
            {
                id: "2-2",
                parentId: 2,
                name: "节点2-2"
            },
            {
                id: "1-1-1",
                parentId: "1-1",
                name: "节点1-1-1"
            },
            {
                id: "1-1-2",
                parentId: "1-1",
                name: "节点1-1-2"
            }
        ]
方法一：非递归，时间复杂度o(n)
```js
function toTree(data){
    let map = {}
    let res = {}
    data.forEach(item => {
        map[item.id] = item
    })
    for(let i = 0; i < data.length; i++){
        let ele = data[i]
        let parent = map[ele.parentId]
        if(parent){
             //有父节点
            (parent.children || (parent.children = [])).push(ele)
        }else{
            //没有父节点
            res[ele.id] = ele
        }
    }
    return res
}
toTree(data)
```

方法二：递归,时间复杂度是o(n*n)
```js
function toTree(data, pid){
    let res = {}
    for(let i = 0; i < data.length; i++){
        let ele = data[i]
        if(ele.parentId == pid){
            //递归遍历，以ele为父节点
            ele.children = toTree(data, ele.id)
            res[ele.id] = ele
        }
    }
    return res
}
toTree(data, 0)
```

#### 2.数组扁平化 flat

```js
let arr = [1, [2, [3,4],5], 6]
function flatten(array){
    let res = []
    flat(array, res)
    return res
}
function flat(array, res){
    array.forEach(item => {
        if(Array.isArray(item)){
            flat(item, res)
        }else{
            res.push(item)
        }
    })
}
let res = flatten(arr)
//或者
function flatten(arr){
    let newarr = []
    for(let i = 0; i < arr.length; i++){
        if(Array.isArray(arr[i])){
            newarr = newarr.concat(flatten(arr[i]))
        }else{
            newarr.push(arr[i])
        }
    }
    return newarr
}
//还可以使用reduce
const flatten = arr => {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? flatten(cur) : cur);
  }, [])
}

```
#### 3.数组forEach(没有返回值)
```js
let arr = [1,2,3,4]
Array.prototype.myforEach = function(callback){
    let thisArg = this
    for(let i = 0; i<thisArg.length; i++){
        let item = thisArg[i]
        callback(item, i)
    }
}
arr.myforEach(item => {
    console.log(item)
})
```
#### 4.数组map（有返回值）
```js
let arr = [1, 2, 3, 4]
Array.prototype.myMap = function (callback) {
    let newArr = [];
    let thisArg = this
    for (let i = 0; i < thisArg.length; i++) {
        newArr[i] = callback(thisArg[i], i)
    }
    return newArr
}
let newarr = arr.myMap((item, index) => {
    return item + 1
})
```
#### 5.数组reduce
```js
let arr = [1, 2, 3, 4]
Array.prototype.myReduce = function (callback, initData) {
    let thisArg = this
    let prev = initData
    let i = 0
    if(!initData){
        prev = thisArg[0]
        i = 1
    }
    for (i; i < thisArg.length; i++) {
        prev = callback(prev, thisArg[i], i)
    }
    return prev
}
let newarr = arr.myReduce((prev, cur, index) => {
    return prev + cur
}, 0)
```
#### 6.数组some（判断数组中的每一项，如果有一项满足条件，就返回true）
```js
let arr = [1, 2, 3, 4]
Array.prototype.mysome = function (callback) {
    let thisArg = this
    let res = false
    for (let i=0;i<thisArg.length;i++) {
        res = callback(thisArg[i], i)
        if(res) break;
    }
    return res;
}
let newarr = arr.mysome((item, index) => {
    return item > 2
}, 0)
```
#### 7.数组filter(判断数组每一项，返回一个所有满足条件项的数组)
```js
let arr = [1, 2, 3, 4]
Array.prototype.myfilter = function (callback) {
    let thisArg = this
    let res = []
    for (let i=0;i<thisArg.length;i++) {
        if(callback(thisArg[i], i)){
            res.push(thisArg[i])
        }
    }
    return res;
}
let newarr = arr.myfilter((item, index) => {
    return item > 2
}, 0)
```
#### 8.类数组对象转化为真正的数组

> 类数组对象的特点：1.具有length属性 2.对象的键值是number类型但是本质上还是对象

let arr = new Set([1,2,3])
- 方法1:Array.from(arr)
- 方法2:扩展运算符 [...arr]
- 方法3:Array.prototype.slice.call(arr)

#### 9.数组去重

1. 方法一: 利用includes或者indexof或者map等等遍历数组,原理类似

```js
let arr = [1,2,2,4]
function fn(arr){
    let newarr = []
    arr.forEach(item=>{
        if(newarr.indexOf(item) == -1){
            newarr.push(item)
        }
    })
    return newarr
}
```
2. new Set() + Array.from()
3. filter + indexof
```js
function unique(array) {
    var res = array.filter(function(item, index, array){
        return array.indexOf(item) === index;
    })
    return res;
}
```
#### 10.Function.prototype.call
```js
let foo = {
    a: 1
}
function bar() {
    console.log(this.a); //1
}
Function.prototype.mycall = function(context = window, ...args){
    //非严格模式下， null自动指向window
    let fn = Symbol("call")
    context.fn = this
    let res = context.fn(...args)
    delete context.fn
    return res
}
bar.mycall(foo, 1,2)
```
#### 11.Function.prototype.apply(和call差别就是第二个参数是传的参数数组)
```js
let foo = {
    a: 1
}
function bar() {
    console.log(this.a); //1
}
Function.prototype.myapply = function(context = window, args){
    let fn = Symbol("call")
    context.fn = this
    let res = context.fn(...args)
    delete context.fn
    return res
}
bar.myapply(foo, [1,2])
```
#### 12.Function.prototype.bind(返回一个新函数)
```js
let foo = {
    a: 1
}
function bar() {
    console.log(this.a); //1
}
Function.prototype.mybind = function(context = window){
    let self = this
    let args = Array.prototype.slice.call(arguments, 1)
    return function(){
        return self.apply(context, args)
    }
}
let fn = bar.mybind(foo, 1,2)
fn()
```
