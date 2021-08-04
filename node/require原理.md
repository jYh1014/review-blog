### require原理

#### 文件的查找规范

- node中的模块分为 内置模块（比如http、path这些）、第三方模块和文件模块（都是通过相对路径来访问的）这三种。
- 首先，如果引入的是内置模块，则不用做处理。
- 如果引入的文件模块X，也就是带有"./" "/" "../"开头的路径，则会先默认去查找同名文件X，如果没有，则再去查找后缀名是.js .json的同名文件，如果没有，就会把这个X当成是一个文件夹，会去找同级下的这个文件夹，先查找package.json文件看看main的配置项，如果没有配置文件则去查找文件夹下的index.js文件，index.json文件。
- 第三方模块，会去当前父级目录下的node_modules文件夹下找同名的文件夹，如果没找到则一直继续向上级寻找node_modules。如果找到根目录下都没找到，则not found。

#### require原理

```js
//
function Module(id){
    this.id = id
    this.exports = {}
}
let json = req("./a")
console.log(json)
```
- 先对引入的模块进行路径解析，不同的模块查找规范不同，我们主要来实现一下文件模块是如何require的。
- 解析出的文件路径是绝对路径，然后执行fs.existsSync查看此文件路径是否存在
- 如果存在则直接返回，如果不存在则会去查找后缀名是js和json的文件。
- 查看此文件是否已经加载过，就是缓存记录里是否存在，若存在就直接读取。
- 缓存不存在的话，就load此文件，先读取此文件内容，然后将内容拼接成一个完整的函数。
- 生成一个真正的函数，修改上下文变量将this指向module.exports，然后执行函数，将函数的返回值module.exports作为模块最终的输出结果。
```js
const wrapper = [
    '(function(module,exports,require,__filename,__dirname){' ,
    '})'
]
Module.extensions = {
    ".js"(module){
        //读出文件内容，格式是字符串，作为函数体。
        let script = fs.readFileSync(module.id, "utf8") 
        //拼接上函数名，成为一个完整的函数
        let fnstr = wrapper[0]+script+wrapper[1]
        //生成真正的函数，等同于new Function
        let fn = vm.runInThisContext(fnstr)
        //执行fn
        fn.call(module.exports, module, module.exports,req,module.id,path.dirname(module.id))
    },
    ".json"(module){
        let script = fs.readFileSync(module.id, "utf8")
        module.exports = JSON.parse(script)
    }
}
Module._cache = {} //保存缓存的结果
Module.prototype.load = function(){ //加载模块
    let ext = path.extname(this.id)
    Module.extensions[ext](this) //执行后缀名相对应的函数
}
Module.resolveFileName = function(filename) {
    //先确定filename的绝对路径
    let abspath = path.resolve(__dirname, filename)
    let flag = fs.existsSync(abspath) //判断文件是否存在
    if(!flag){
        //如果不存在的话,则去查找后缀名为.js .json的同名文件
        let keys = Object.keys(Module.extensions)
        for(let i = 0; i < keys.length; i++){
            current = abspath + keys[i]
            let flag = fs.existsSync(current) //判断文件是否存在
            if(flag){
                break;
            }else{
                current = null
            }
        }
        if(!current){
            throw new Error("文件不存在")
        }
    }
    return current
}
function req(filename){
    //文件名解析，查看文件是否存在
    let current = Module.resolveFileName(filename) //返回的是文件的绝对路径
    //查看此文件是否已经被加载过，如果是，则可以直接取缓存的结果。
    if(Module._cache[current]){
        return Module._cache[current].exports
    }
    let module = new Module(current)
    Module._cache[current] = module
    module.load()
    return module.exports
}
```