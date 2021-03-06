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

使用模块模式就可以摈弃全局变量的使用，它促进了信息隐藏和其他优秀的设计实践。对于应用程序的封装，或者构造其他单例对象，模块模式非常有效。

假定我们想要构造一个用来产生序列号的对象：
```js
/*
 * 返回一个用来产生唯一字符串的对象
 * 唯一的字符串由两部分组成：前缀 + 序列号
 * 该对象包含一个设置前缀的方法，一个设置序列号的方法和产生唯一字符串的方法
 */
var serial_maker = function(){
  var prefix = '', seq = 0
  return {
    setPrefix: function(p){
      prefix = String(p)
    },
    setSeq: function(s){
      seq = s
    },
    gensym: function(){
      var result = prefix + seq
      seq += 1
      return result
    }
  }
}
var seqer = serial_maker()
seqer.setPrefix('S')
seqer.setSeq('12345')
var unique = seqer.gensym()
// "S12345"
```
seqer 包含的方法都没有用到 this，因此没有办法能够改变 seqer，除非调用对应的方法才可以更改 prefix 和 seq 的值。seqer 对象是可变的，所以它的方法可能会被替换掉，但替换后的方法依然不能访问私有成员，seqer 就是一组函数的集合，而且那些函数被授予访问和修改私有状态的能力。

如果我们把 seqer.gensym 作为一个值传递给第三方函数，那个函数能产生唯一字符串，但却不能通过它来改变 prefix 和 seq 的值。

## 级联
有一些方法没有返回值。例如一些修改或设置对象的某个状态却不反悔任何值的方法就是典型的例子。如果我们让这些方法返回 this 而不是 undefined，就可以启用级联。在一个级联中，我们可以在单独一条语句中依次调用同一个对象的很多方法。一个启用级联的 Ajax类库可能允许我们以这样的形式去编码：
```js
getElement('myBoxDix')
  .move(350,150)
  .width(100)
  .height(100)
  .color('black')
  .border('1px solid #fcc')
  .padding('4px')
  .on('mousedown',function(m){
    this.startDrag(m,this.getNinth(m))
  })
  .on('mousemove','drag')
  .on('mouseup','stopdrag')
  .later(2000,function(){
    this.color('orange')
        .setHtml('you have stopped')
  })
```
在这个例子中，getElement 函数产生一个对应于 `id = "myBoxDix"` 的 DOM 元素并提供了其他功能的对象，该方法允许我们移动元素，修改尺寸和样式，并添加行为。这些方法都返回该对象，所以调用返回的结果可以被下一次调用所用。

## 套用
函数也是值，从而我们可以用有趣的方式去操作函数值。套用允许我们将函数传递给它的参数相结合去产生一个新的函数。
```js
var add1 = add.curry(1)
console.log(add1(6)) 
// 7
```
add1 是把1传递给 add 函数的 curry 方法后创建的一个函数。add1 函数把传递给它的参数值加1。
```js
Function.prototype.curry = function(){
  var args = arguments, that = this
  return function(){
    return that.apply(null,args.concat(arguments))
  }
}
```
curry 方法通过创建一个保存着原始函数和被套用的参数的闭包来工作。它返回另一个函数，该函数被调用时，会返回调用原始函数的结果，并传递调用 curry 时的参数加上当前被调用的参数的所有参数。它使用 concat 方法去连接两个参数数组。

糟糕的是，arguments 数组并非一个真正的数组，所以它没有 concat 方法。要避开这个问题，我们必须在两个 arguments 数组上都应用数组的 slice 方法。这样产生出拥有 concat 方法的常规数组。
```js

```

# 继承
## 伪类
JavaScript 不直接让对象从其他对象继承，而是插入一个多余的间接层：通过构造器函数产生对象。

当一个函数对象被创建时，Function 构造器产生的函数对象会运行类似这样的一些代码：
```js
this.prototype = {constructor: this}
```
新函数被赋予一个 prototype 属性，它的值是一个包含 constructor 属性且属性值为该新函数的对象。这个 prototype 对象是存放继承特征的地方，因为每个 JavaScript 语言没有提供一种方法来确定哪个函数是打算用来做构造器的，所以每个函数都会得到一个 prototype 对象。constructor 属性没什么用，重要的是 prototype 对象。

当采用构造器函数调用模式，即用 new 前缀去调用一个函数时，函数执行的方式会被修改，如果 new 运算符是一个方法而不是一个运算符，它有可能这样执行：
```js
Function.prototype.new = function(){
  // 创建一个新对象，它继承自构造器函数的原型对象
  var that = Object.create(this.prototype)
  // 调用构造器函数，绑定 this 到新对象上
  var other = this.apply(that,arguments)
  // 如果它返回值不是一个对象，就返回该新对象
  return (typeof other === 'object' && other ) || that
}
```
我们可以定义一个构造器并扩充它的原型：
```js
var Mammal = function(name){
  this.name = name
}
Mammal.prototype.getName = function(){
  return this.name
}
Mammal.prototype.sayHi = function(){
  return 'Hi I\'m ' + this.name + '!'
}
```
我们可以构造另一个伪类来继承 Mammal，这是通过定义它的 constructor 函数并替换它的 prototype 为一个 Mammal 的实例来实现的：
```js
var myMammal = new Mammal('jack')
var name = myMammal.getName()
```

## 函数化
迄今为止，我们所看到的继承模式的一个弱点就是没法保护隐私。对象的所有属性都是可见的，我们无法得到私有变量和私有属性。我们可以应用模块模式来保护隐私。

我们从构造一个生成对象的函数开始，以小写字母开头来命名它，因为它不需要使用 new 前缀。
1. 创建一个新对象。
2. 有选择地定义私有实例变量和方法。
3. 给新对象扩充方法。这些方法拥有特权去访问参数。
4. 返回那个新对象。

这是一个函数化构造器的伪代码模板：
```js
var constructor = function(spec, my){
    var that, 其他私有实例变量
    my = my || {}
    把共享的变量和函数添加到 my 中
    that = 一个新对象
    添加给 that 特权方法
    return that
}
```
spec 对象包含构造器需要构造一个新实例的所有信息。spec 的内容可能会被复制到私有变量中，或者被其他函数改变，或者方法可以在需要的时候访问 spec 的信息。

my 对象是一个为继承链中的构造器提供秘密共享的容器。my 对象可以选择性的使用。如果没有传入一个 my 对象，那么会创建一个 my 对象。

接下来声明该对象私有的实例变量和方法。通过简单地声明变量就可以做到。构造器的变量和内部函数变成了该实例化的私有成员。内部函数可以访问 spec、my、that，以及其他私有变量。

接下来，给 my 对象添加共享的秘密成员。这通过赋值语句来实现：
```
my.member = value
```
现在，我们构造了一个新对象，并把它赋值给 that。my 对象允许其他的构造器分享我们放到 my 中的资料，其他构造器可能也会把自己可分享的秘密成员放进 my 对象里，以便我们的构造器可以利用它。

接下来我们扩充 that，加入组成该对象接口的特权方法。我们可以分配一个新函数成为 that 的成员方法。或者，更安全的，我们可以先把函数定义成为私有方法，然后再把它们分配给 that：
```js
var methodical = function(){
    ...
}
that.methodical = methodical
```
分两步去定义的好处是，如果其他方法想要调用 methodical，它们可以直接调用 methodical，而不是 that.methodical。如果该实例被破坏或篡改，甚至 that.methodical 被替换掉了，调用 methodical 的方法同样会继续工作，因为它们私有的 methodical 不受该实例被篡改的影响。

最后我们返回 that。

我们把这个模式应用到 mammal 例子里，name 和 saying 属性现在完全是私有的，只有 getName 和 says 两个特权方法才可以访问到它们。
```js
var mammal = function(spec){
    var that = {}
    that.getName = function(){
        return spec.name
    }
    that.says = function(){
        return spec.saying || ''
    }
    return that
}
var myMammal = mammal({name: 'Herb', saying: 'Hello'})
myMammal
// {getName: ƒ, says: ƒ}
```
函数模式化有很大的灵活性，它相比伪类模式不仅带来的工作更少，还让我们得到更好的封装和信息隐藏，以及访问父类的方法。

如果对象的所有状态都是私有的，name 该对象就成为一个“防伪 tamper-proof”对象，该对象的属性可以被替换或删除，但该对象的完整性不会受到伤害。如果我们用函数化的样式去创建一个对象，并且该对象的所有方法都不使用 this 或 that，那么该对象就出持久性的。一个持久性对象就是一个简单功能函数的集合。

一个持久性的对象不会被入侵。访问一个持久性的对象时，除非有方法授权，否则攻击者不能访问对象的内部状态。

## 部件

我们可以从一套部件中把对象组装出来。例如我们可以构造一个给任何对象添加简单事件处理特性的函数。它会给对象添加一个 on 方法、fire 方法和一个私有的事件注册表对象：
```js
var eventuality = function(that){
    var registry = {}
    that.fire = function(event){
        var array,func,handle,i,type = typeof event === 'string' ? event : event.type
        if(registry.hasOwnProperty(type)){
            array = registry[type]
            for(var i = 0; i < array.length; i++){
                handler = array[i];
                func = handler.method;
                if(typeof func === 'string'){
                    func = this[func]
                }
                func.apply(this,handler.parameters || [event])
            }
        }
    return this
    }
    that.on = function(type,method,parameters){
        var handler = {
            method: method,
            parameters: parameters
        }
        if(registry.hasOwnProperty(type)){
            registry[type].push(handler)
        }else{
            registry[type] = handler
        }
        return this
    }
    return that
}
```
我们可以在任何单独的对象上调用 eventuality，授予它事件处理方法。我们也可以赶在 that 被返回前在一个构造器函数中调用它。
```
eventuality(that)
```
用这种方式，一个构造器函数可以从一套部件中把对象组装出来。JavaScript 的弱类型在此处是一个巨大的优势，因为我们无需花费精力去了解对象在类型系统中的继承关系。相反，我们只需要专注它们的个性特征。

如果我们想要 eventuality 访问该对象的私有状态，可以把私有成员集 my 传递给它。

# 数组
## 删除
由于 JavaScript 的数组其实就是对象，所以 delete 运算符可以用来从数组中移除元素。
```js
var numbers = ['zero','one','two','three','four','five','six','seven','eight','nine']
delete numbers[2]
// true
numbers
// ["zero", "one", empty, "three", "four", "five", "six", "seven", "eight", "nine"]
```
不幸的是，那样会在数组下留下一个空洞，这是因为被排在删除元素之后的元素保留着它们最初的属性。而我们通常是想递减后面每个元素的属性。

数组中有个 splice 方法，他、它可以删除元素，并把它们替换成其他的元素。第一个参数数组中的序号，第二个参数是要删除元素的个数，后面的参数是要新插入的元素。

## 枚举
因为 JavaScript 数组其实就是对象，所以可以通过 for in 语句来遍历一个数组的所有属性。遗憾的是 for in 无法保证属性的顺序，而大多数遍历数组的场合都期望按照阿拉伯数字顺序来产生元素。此外可能从原型链中得到意外属性的问题存在。

所以我们可以通过 for 语句、forEach、for of 来进行数组的遍历。

## 方法
我们可以通过 Array.prototype 扩充数组的方法。

```js
Array.prototype.reduce = function(f,value){
  for(var i = 0; i < this.length; i++){
    value = f(this[i],value)
  }
  return value
}
```
在这个例子中，我们定义了一个 reduce 方法，它接受一个函数和一个初始值作为参数。它遍历这个数组，以当前元素和该初始值为参数调用这个函数，并且计算出一个新值，当完成时，它返回这个值。如果我们传入一个把两个数字相加的函数，它会计算出相加之和。如果我们传入把两个数字相乘的函数，它会计算两者的乘积。
```js
Array.prototype.reducer = function(f,value){
  for(var i = 0; i < this.length; i++){
    value = f(this[i],value)
  }
  return value
}
var add = function(a,b){
  return a + b
}
var arr = [1,2,3,4,5];
arr.reducer(add,0)
```

