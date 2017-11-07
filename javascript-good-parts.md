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


第一个语句
