# Function
## 调用
调用一个函数会暂停当前函数的执行，把传递控制权和参数给新函数。除了声明时定义的形参，每个函数还会接受两个附加参数：this 和 arguments。参数 this 的值取决于调用模式。在 JavaScript 中有四种调用模式：方法调用模式、函数调用模式、构造器调用模式和 apply 调用模式。这些模式在初始化 this 上存在差异。

### 方法调用模式
当函数被保存为对象的一个属性时，我们称之为一个方法。当一个方法被调用时，this 被绑定到该对象。
```js
function f1(){
  console.log(this)
}
var obj = {
	a : 1,
	fn: f1
}

obj.fn();
// {a: 1, fn: ƒ}
```
方法调用可以使用 this 访问自己所属的对象，所以他能从对象中取值或对值进行修改。this 对象的绑定发生在调用的时候，通过 this 可以取得它们所属对象的上下文的方法称为公共方法。

```js
function f2(){
	this.a += 1
	console.log(this.a)
}

obj.f2 = f2
obj.f2();
// {a: 2, fn: ƒ, f2: ƒ}
```
### 函数调用模式
当函数并非一个对象的属性时，它就是被当做一个函数来调用的。此时 this 被绑定到全局对象上。
```js
var a = 5
function add(a,b){
	return a + b
}
obj.add2 = function(){
    var helper = function(){
	this.a = add(this.a,this.a)
    }
    helper()
}
obj.add2()
console.log(a)
// 10
```
当函数作为一个函数来调用时，this 指向的是 window,此时改变的是 window.a 的值。而作为函数的方法调用时，this 指向的是调用该方法的对象。
```js
obj.add1 = function(){
	this.a = add(this.a,this.a)
}
obj
// {a: 2, fn: ƒ, add1: ƒ}
```
因此如果通过方法定义一个变量，并把它赋值为 this，那么内部函数就可以通过那个变量访问到 this。
```js
obj.add2 = function(){
    var that = this
    var helper = function(){
	that.a = add(that.a,that.a)
    }
    helper()
}
obj
// {a: 4, fn: ƒ, add1: ƒ, add2: ƒ}
```

### 构造器调用模式
如果在一个函数前面带上 new 来调用，那么背地里会创建一个连接到该函数的 prototype 成员的新对象，同时 this 会绑定到那个新对象上。

new 前缀也会改变 return 语句的行为。

### apply 调用模式
apply 方法让我们构建一个参数数组传递给调用函数，它允许我们选择 this 的值，apply 接收两个参数，第一个是要绑定给 this 的值，第二个是参数数组。

## 返回
当一个函数被调用时，它从第一个语句开始执行，直到关闭函数体的`}`时结束，然后把函数的控制权交还给调用该函数的程序。

return 语句可以使函数提前返回，如果没有指定返回值，则返回一个 undefined。

如果函数调用的时候在前面加上了 new 前缀，且返回值不是一个对象，则返回 this（该新对象）。

## 异常
异常是干扰程序的正常流程的的事故，当发现这样的事故时，程序中应该抛出一个异常：
```js
var add = function (a,b){
    if(typeof a !== 'number' || typeof b !== 'number'){
        throw {
            name: 'TypeError',
            message: 'input numbers'
        }
    }
    return a + b
}
```
throw 语句中断函数的执行，它应该抛出一个 exception 对象，该对象包含一个用来识别异常类型的 name 属性和一个描述性的 message 属性。你也可以添加其他属性。

该 exception 对象将会被传递到一个 try 语句的 catch 从句：
```js
var try_it = function(){
    try{
        add("seven")
    }catch(e){
        console.log(e.name + ':' + e.message)
    }
}
try_it()
```
如果在 try 代码块内抛出一个异常，控制权就会跳转到它的 catch 从句。

一个 try 语句只会有一个捕获所有异常的 catch 代码块。如果你的处理手段取决于异常的类型，name 异常处理器必须检查异常对象的 name 属性来确定异常的类型。

如果我直接执行`add('aaa')`这样的语句，那么控制台会直接扔出错误
```
Uncaught 
{name: "TypeError", message: "input numbers"}
```
当我执行 try_it 函数时，控制台会打印
```
TypeError:input numbers
```

## 扩充类型的功能
JavaScript 允许给语言的基本类型扩充功能。在对象中我们可以通过 Object.prototype 添加方法，可以让该方法对所有对象都适用。这样的方式对函数、数组、字符串、数字、正则表达式和布尔值同样适用。

举例来说，我们可以给 Function.prototype 增加方法来使得该方法对所有函数可用：
```js
Function.prototype.method = function(name, func){
    this.prototype[name] = func
    return this
} 
```
通过给 Function.prototype 增加 method 方法，下次我们给对象增加方法的时候就不必键入 prototype 这几个字符了。

JavaScript 没有专门的整数类型，但有时候需要提取数字的中的整数部分。我们可以通过给 Number.prototype 增加一个方法来改善它。它会根据数字的正负来判断是使用 Math.ceil 还是 Math.floor 。
```js
Number.prototype.integer = function(){
    return Math[this < 0 ? 'ceil' : 'floor'](this)
}
(-33.3).integer()
// -33
```
JavaScript 缺少一个去处头尾字符串空白的方法，可以这样弥补：
```js
String.prototype.trim = function(){
    return this.replace(/^\s+|\s+$/g,'')
}
'   sfsdafd dfsdf "  '.trim()
// "sfsdafd dfsdf ""
```
基本类型的原型是公用结构，所以在类库混用的时候务必小心。一个保险的做法是只在确定没有该方法的时候才添加它。
```js
// 符合条件时添加方法
Function.prototype.method = function(name, func){
    if(!this.prototype[name]){
        this.prototype[name] = func
    }
    return this
}
```

## 递归
递归函数时会直接或间接调用自身的一种函数。它把一个问题分解为一组相似的子问题，每一个都用一个寻常解去解决。一般来说，一个递归函数调用自身去解决它的子问题。

一些语言提供了尾递归优化，这意味着如果一个函数返回自身递归调用的结果，name 调用的过程会被替换为一个循环，它可以显著提高速度。遗憾的是 JavaScript 没有提供尾递归优化，深度递归的函数可能会因为堆栈溢出而运行失败。
```js
// 构建一个带尾递归的函数，因为它会返回自身调用的结果
var factorial = function factorial(i, a){
    a = a || 1;
    if( i < 2){
        return a
    }
    return factorial(i - 1, a * i)
}
factorial(4)
// 24
```
## 作用域
在编程语言中，作用域控制着变量与参数的可见性和生命周期。对程序员来说这是一项重要的服务。因为它减少了命名冲突，并且提供了内存自动管理。
```js
var foo = function(){
    var a = 3, b = 5
    var bar = function(){
        var b = 7, c = 11
        a += b + c 
    }
    bar()
    // bar() 运行后 a = 21 b = 5
}
```
JavaScript 确实有函数作用域，这意味着函数中的参数和变量在函数外部是不可见的，而在函数内部任何位置定义的变量，在函数内部任何地方都可见。

## 闭包
作用域的好处是内部函数可以访问定义在它们外部函数的参数和变量（除了 this 和 arguments ）。

更有趣的情形是，内部函数拥有比外部函数更长的生命周期。

之前我们构造了一个 myObject 对象，它拥有一个 value 属性和一个 increment 方法。假定我们希望保护该值不会被非法更改。和以字面量形式去初始化 myObject 不同，我们通过调用一个函数去初始化 myObject，该函数会返回一个对象字面量。函数里定义了一个 value 变量。该变量对 increment 和 getValue 方法总是可用的，但函数的作用域使得它对其他的程序来说是不可见的。
```js
var myObject = (function(){
    var value = 0;
    return{
        increment: function(inc){
            value += typeof inc === 'number' ? inc : 1
        },
        getValue: function(){
            return value
        }
    }
})()
myObject.getValue();
//0
myObject.getValue
// 这是打印函数
<!-- ƒ (){
            return value
        } -->
myObject.increment(3);
myObject.getValue();
// 3
```
我们没有把一个函数赋值给 myObject。我们把调用该函数后返回的结果赋值给它。该函数返回一个包含两个对象的方法，并且这些方法继续享有访问 value 变量的特权。但是我们无法直接更改 value 的值，只能通过 increment 方法增加输入的 number 值。

```js
var fade = function(node){
    var level = 1
    var step = function(){
        var hex = level.toString(16)
        node.style.backgroundColor = '#ffff' + hex + hex;
        if(level < 15){
            level += 1
            setTimeout(step,100)
        }
    }
    setTimeout(step,100)
}
fade(document.body)
```
我们调用 fade，把 document.body 作为参数传递给它。fade 函数设置 level 为1，它定义了一个 step 函数，接着调用 setTimeout，并传递 step 函数和一个时间（100ms）给它，然后它返回，fade 函数结束。

在大约100ms 之后，step 函数被调用，它把 fade 函数的 level 变量转化为16进制字符，接着它修改 fade 函数得到的节点的背景颜色，然后查看 fade 函数的 level 变量。如果背景色尚未变成白色，它会增大 level 的数值，接着用 setTimeout 预定让它自己再次运行。

当页面里有5个 a 标签，给每个 a 标签加上点击事件，当被点击时显示 a 标签的次序。
```js
  var btns = document.querySelectorAll('a')
  var add_handlers = function(nodes){
    for(var i = 0; i < nodes.length; i++){
      nodes[i].onclick = function(i){
          alert(i)
      }
    }
  }
  add_handlers(btns)
```
当采用这种写法时，并不能达到预期效果，它总是会显示节点的数量。因为事件处理函数绑定了变量 i 本身，而不是函数在构造时的变量 i 的值。所以我们通过闭包来改造这个函数：
```js
  var add_handlers = function(nodes){
    for(var i = 0; i < nodes.length; i++){
      nodes[i].onclick = (function(i){
        return function(){
          alert(i)
        }
      })(i)
    }
  }
```

## 回调
函数使得对不连续事件的处理变得更容易。假定有这样一个序列，由用户的交互行为触发，向服务器发送请求，最终显示服务器的响应，最自然的写法可能会是这样的：
```
request = prepare_the_request()
response = send_request_synchronously(request)
display(response)
```
这种方式的问题在于，网络上的同步请求会导致客户端进入假死状态，如果网络传输或者服务器很慢，响应会慢到让人无法接受。

更好的方式是发起异步请求，提供一个当服务器的响应到达是随即触发的回调函数，异步函数立即返回，这样客户端就不会被阻塞。
```
request = prepare_the_request()
send_request_synchronously(request,function(response){
    display(response)
})
```
我们传递一个函数给 `send_request_synchronously` 函数，一旦接收响应，它就会被调用。

## 模块
我们可以使用函数和闭包来构造模块。模块是一个提供接口却隐藏状态与实现的函数或对象。

模块模式的一般形式是：一个定义了私有变量和函数的函数；利用闭包创建可以访问私有变量和函数的特权函数；最后返回这个特权函数，或者把它们保存到一个可访问到的地方。