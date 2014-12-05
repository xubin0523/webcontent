Title: JavaScript Core
Category: JavaScript
Tags: JavaScript , Basics

##what is javaScript?
JavaScript 是一种动态，弱类型，解释型语言，它还有一个原名叫ECMAScript。我还把它看成函数式编程语言，考虑到业界对函数式语言没有一个明确的定义，但是单从它可以把函数当成参数来看，它就是函数式的。 
 
* 动态：程序在运行时可以改变对象的结构。  
* 弱类型：语法中没有对类型的强制检测，类型之间可以相互转换。
- 解释型：相对于编译型语言存在的，源代码不是直接翻译成机器语言，而是先翻译成中间代码，再由解释器对中间代码进行解释运行。
- 函数式：我理解的是y = f(x) , x could be a function
	
当然，最重要的它还是面向对象的编程语言。
>  ECMAScript is an object-oriented programming language supporting delegating inheritance based on prototypes.

## Introduction
通过最近几天的学习，对于只接触过C，C++，Java 的我来说，javaScript真的是一门高级语言，awesome！以后我还要学习go，lisp，应该也是别有一番风味。
我试着去理解三个问题：  
1. 对象，函数，闭包是什么，他们之间有何联系。  
2. 它是怎么面向对象的，我们怎么使用它进行面向对象编程。  
3. 函数式编程的优缺点，对比OOP。  

***下面的讨论都是基于ECMAScript 5***
## Object
下面是对对象的非常准确的定义。
> Object is an unordered collection of key-value pairs.

这边就有一点疑惑了，函数是不是可以看成对象，如果它是，那它是键值对？如果不是，那它与对象的关系是什么？——hold  
Example:  

```
var x = { // object "x" with three properties: a, b, c
  a: 10, // primitive value
  b: {z: 100}, // object "b" with property z
  c: function () { // function (method)
    alert('method x.c');
  }
};
 
alert(x.a); // 10
alert(x.b); // [object Object]
alert(x.b.z); // 100
x.c(); // 'method x.c'
```

可以看到，key代表了对象的属性，value可以是任何类型。
###Property attributes
对象中的熟悉有下面几个性质：  
1. writable: 属性值可否更改  
2. enumerable: 属性值能否在循环中遍历  
3. configurable: 属性值能否被delete  
4. Internal：属性是否是内置的，一般编程用不到，这个在javaScript语言的实现层。    
Example:  

```
var foo = {};
 
Object.defineProperty(foo, "x", {
  value: 10,
  writable: true, 
  enumerable: false, 
  configurable: true 
});

Object.defineProperty(foo, "y", {
  value: 20,
  writable: false, 
  enumerable: true, 
  configurable: false 
});
console.log(foo.x); // 10
foo.x = 100;
console.log(foo.x); // 100
foo.y = 200;
console.log(foo.y); // 20

(function enumV() {
  for (property in foo){
    console.log(property);
  }
})();// y

delete foo.x;
delete foo.y;
console.log(foo);//Object {y: 20}
```

### Constructor
对象的创建是由构造函数来完成的。当O = new F( ); 执行时，在JS内部对象的创建遵循以下算法：  

```
F.[[Construct]](initialParameters):
 
O = new NativeObject();
 
// property [[Class]] is set to "Object", i.e. simple object
O.[[Class]] = "Object"
 
// get the object on which
// at the moment references F.prototype
var __objectPrototype = F.prototype;
 
// if __objectPrototype is an object, then:
O.[[Prototype]] = __objectPrototype
// else:
O.[[Prototype]] = Object.prototype;
// where O.[[Prototype]] is the prototype of the object
 
// initialization of the newly created object
// applying the F.[[Call]]; pass:
// as this value – newly created object - O,
// arguments are the same as initialParameters for F
R = F.[[Call]](initialParameters); this === O;
// where R is the returned value of the [[Call]]
// in JS view it looks like:
// R = F.apply(O, initialParameters);
 
// if R is an object
return R
// else
return O
```  

函数的创建遵循以下算法：  

```
F = new NativeObject();
  
// property [[Class]] is "Function"
F.[[Class]] = "Function"
  
// a prototype of a function object
F.[[Prototype]] = Function.prototype
  
// reference to function itself
// [[Call]] is activated by call expression F()
// and creates a new execution context
F.[[Call]] = <reference to function>
  
// built in general constructor of objects
// [[Construct]] is activated via "new" keyword
// and it is the one who allocates memory for new
// objects; then it calls F.[[Call]]
// to initialize created objects passing as
// "this" value newly created object 
F.[[Construct]] = internalConstructor
  
// scope chain of the current context
// i.e. context which creates function F
F.[[Scope]] = activeContext.Scope
// if this functions is created 
// via new Function(...), then
F.[[Scope]] = globalContext.Scope
  
// number of formal parameters
F.length = countParameters
  
// a prototype of created by F objects
__objectPrototype = new Object();
__objectPrototype.constructor = F // {DontEnum}, is not enumerable in loops
F.prototype = __objectPrototype
  
return F
```

从内置的创建算法来看，对象的prototype是对应的构造函数在创建时产生的prototype。函数与对象的第一个关系我们找到了，那就是（1）函数可以作为对象的构造函数，对象在创建时会去调用构造函数（2）并且对象的prototype在初始化时用的是其函数的prototype，修改函数的prototype，不会对已经通过函数创建的对象的prototype产生影响。(3) 通过new 关键字创造的对象首先是构造函数返回的对象，其次才是原生创建的对象。要正确理解第2，3点，看下面这个例子：
  
```
function A() {return [1,2]}
A.prototype.x = 10;
 
var a = new A();
alert(a.x); // 10 – by delegation, from the prototype
 
// set .prototype property of the
// function to new object; why explicitly
// to define the .constructor property,
// will be described below
A.prototype = {
  constructor: A,
  y: 100
};
 
var b = new A();
// object "b" has new prototype
alert(b.x); // undefined
alert(b.y); // 100 – by delegation, from the prototype
 
// however, prototype of the "a" object
// is still old 
alert(a.x); // 10 - by delegation, from the prototype
 
function B() {
  this.x = 10;
  return new Array();
}
 
// if "B" constructor had not return
// (or was return this), then this-object
// would be used, but in this case – an array
var b = new B();
alert(b.x); // undefined
alert(Object.prototype.toString.call(b)); // [object Array]
```  

###Reading and writing properties
读写属性值，使用的是js内置函数[[get]]、[[put]]，可以看到它们在js中的实现,一切都了然了：  
Get  

```
O.[[Get]](P):
 
// if there is own
// property, return it
if (O.hasOwnProperty(P)) {
  return O.P;
}
 
// else, analyzing prototype
var __proto = O.[[Prototype]];
 
// if there is no prototype (it is,
// possible e.g. in the last link of the
// chain - Object.prototype.[[Prototype]],
// which is equal to null),
// then return undefined;
if (__proto === null) {
  return undefined;
}
 
// else, call [[Get]] method recursively -
// now for prototype; i.e. go through prototype
// chain: try to find property in the
// prototype, after that – in a prototype of
// the prototype and so on, until
// [[Prorotype]] will be equal to null
return __proto.[[Get]](P)
```
可以看到Get的实现其实是一个递归，它一直会向上递归去寻找prototype对象中是否含有这个属性，直到找到或者到最顶端结束。这是一个prototype chain 。  
Put  

```
O.[[Put]](P, V):
 
// if we can't write to
// this property then exit
if (!O.[[CanPut]](P)) {
  return;
}
 
// if object doesn't have such own,
// property, then create it; all attributes
// are empty (set to false)
if (!O.hasOwnProperty(P)) {
  createNewProperty(O, P, attributes: {
    ReadOnly: false,
    DontEnum: false,
    DontDelete: false,
    Internal: false
  });
}
 
// set the value;
// if property existed, its
// attributes are not changed
O.P = V
 
return;
```

可以自己定义get/set 访问器：  

```
var chenhao = {};
Object.defineProperty( chenhao,
            'age', {
                      get: function() {return age+1;},
                      set: function(value) {age = value;},
                      enumerable : true,
                      configurable : true
                    }
);
chenhao.age = 100; //调用set
alert(chenhao['age']); //调用get 输出101（get中+1了）;
```  

##Execution context
Execution context(EC)是一个抽象概念，可以把EC看成函数的执行环境，可以把它抽象成一个栈，最底层是global context，每执行一次函数都会往栈里push一个当前的context。这里的函数是包括所有的函数，构造函数，递归等。  
那执行环境应该包含哪些东西呢？很多框架也有Context那么一个概念，感觉很类似。JS里面的Execution Context 包含三部分：Variable object、Scope chain、thisValue。  
###Variable object
variable object(vo) 就是在当前context中定义的函数，变量。它是一个抽象概念，你可以把它看成执行过程中能引用的资源。  

```
function test(a, b) {
  var c = 10;
  function d() {}
  var e = function _e() {};
  (function x() {});
}
  
test(10); // call
```
上面的例子，通过调用test，进入一个function context，这时vo的值可以表示为:  

```
VO(test) = {
  a: 10,
  b: undefined,
  c: undefined,
  d: <reference to FunctionDeclaration "d">
  e: undefined
};
```
可以得出vo的原则：   
 

- 在进入function context时，vo产生
- 函数参数有值，没有传入的参数为undefined
- context中的变量初始值为undefined
- context中声明的函数，有对应的引用
- vo不包含函数表达式


再看下面这个例子，就可以更好的理解了。  

```
alert(x); // function
 
var x = 10;
alert(x); // 10
 
x = 20;
 
function x() {}
 
alert(x); // 20
```  

####没有全局变量的存在
对于变量的理解，很多js中都把var声明的作为局部变量，没有var声明的作为全局变量，这其实是错误的。请看以下代码：  

```
alert(a); // undefined
alert(b); // "b" is not defined
 
b = 10;
var a = 20;
```

说明在vo创建时，只生成了a，并没有b，b是在执行过程中生成的。  

```
alert(a);//a is not defined
function foo(){
	var a = 10;
	b = 20
}
c = 30;
var d = 40;
foo();
alert(window.a)//undefined
alert(window.b)//20
alert(window.c)//30
alert(window.d)//40
delete b;
delete c;
delete d;
alert(window.b)//undefined
alert(window.d)//40
```  

说明b、c是window的一个属性，而js中的变量是configurable属性默认为false的。  
###Scope chain
scope chain 也是一个抽象概念，但是非常容易理解。既然它是一个chain，那先明确scope的概念
>[[Scope]] is a hierarchical chain of all parent variable objects, which are above the current function context; the chain is saved to the function at its creation

首先，js中的变量是function scope的。  

```
var b = 10;
if(true){var b = 20;}
alert(b);//20;
```

这是vo的性质，Scope chain 是所有上层context的vo、ao的集合。
还有非常重要的一点：scope 在函数创建的时候就决定的，并不是动态决定。可以参考函数创建的算法。  

```  
var x = 10;
 
function foo() {
  alert(x);
  alert(y);
  alert(z);
}
 
(function () {
  var x = 20;
  var y = 30;
  var z = 50;
  foo(); // 10, y is not defined, undefined
})();

var z = 40;
```

总结下来就是：  
1. Scope chain 是所有vo、ao的集合。  
2. vo、ao在一开始函数定义时决定  
3. Scope chain 就是用来查找变量的。  
> Scope chain is related with an execution context a chain of variable objects which is used for variables lookup at identifier resolution.

####Closure
理解了scope chain，基本上就理解闭包了。
> A closure is a combination of a code block and data of a context in which this code block is created.

先看例子：  

```
var incr;
var decr;
function foo(){
var k=100;
incr = function(){var b=10;k++;alert(k);}
decr = function(){k--;alert(k);alert(b);}
}
foo();
incr();//101
decr();//100, b is not defined
```

对于传统的利用栈来存储变量的语言来说，一旦函数结束返回，变量就出栈，内存也释放了。但是，对于要把函数当作参数传递的函数式语言，这是不科学的。因此有个free变量的概念。  

>A free variable is a variable which is used by a function, but is neither a parameter, nor a local variable of the function.

这些free 变量将常驻内存，调用闭包都能访问到，这才是闭包的精髓。那么问题来了，这些变量将一直存在，会不会造成内存泄漏呢？
答案是会的。但这要求我们有良好的编码习惯了，把函数都写入函数表达式中，不要定义全局变量等等。  

```
var data = [];
 
for (var k = 0; k < 3; k++) {
  data[k] = function () {
    alert(k);
  };
}
 
data[0](); // 3, but not 0
data[1](); // 3, but not 1
data[2](); // 3, but not 2
```

通过传递k值  

```
var data = [];
 
for (var k = 0; k < 3; k++) {
  data[k] = (function _helper(k) {
	  k+=6;
    return function () {
      alert(k);
    };
  })(k); // pass "k" value
}
 
// now it is correct
data[0](); // 0
data[1](); // 1
data[2](); // 2
```

####this value
this 对象也是属于EC的一个属性，它就是函数调用者本身，在函数调用时产生，并且reference无法修改，当无函数调用者时，this指向global。
下面这个例子，可以看到new一个对象和之间调用函数的区别：new 一个对象时，对象会调用构造函数，这时的this是对象本身。直接函数的调用，this则指向global。  

```
function foo() {
this.x = 123;
var bf = function(){alert(this.x)}
bf();
}

var obj = new foo; // undefined
foo(); // 123
```

##OOP
面向对象有三大概念：封装、继承、多态。是对是错，我们暂且不管，之后会讨论。
现在先看看Js是怎么实现这些原理的，因为如果知道js是怎么实现的，那就可以把在java，c++中学习到的面向对象编程方法用到js上来，那么就能用js开发复杂业务啦。
###Inheritance & Override
在一开始js的定义中，已经说到js是通过对象中的内置属性prototype用代理的方式来实现继承的。也就是说，只要理解了prototype的原理以及[[get]]方法的原理，那继承的原理也就了然啦。
可以使用Object.create:  

```
var foo = {x: 10, z: function()(return 100;)};
var bar = Object.create(foo, {y: {value: 20}, f: {value:function(x){return x+1}}});
console.log(bar.x, bar.y, bar.f(bar.x),bar.z()); // 10, 20, 11, 100
bar.z=function(){return 200;}
bar.z();//200
```

它的内部实现原理是：  

```
Object.create ||
Object.create = function (parent, properties) {
  function F() {}
  F.prototype = parent;
  var child = new F;
  for (var k in properties) {
    child[k] = properties[k].value;
  }
  return child;
}
```
###Mixins
```
// helper for augmentation
Object.extend = function (destination, source) {
  for (property in source) 
  if (source.hasOwnProperty(property)) {
    destination[property] = source[property];
  }
  return destination;
};
 
var X = {a: 10, b: 20};
var Y = {c: 30, d: 40};
 
Object.extend(X, Y); // mix Y into X
alert([X.a, X.b, X.c, X.d]); 10, 20, 30, 40
```
###Composition

```
var _delegate = {
  foo: function () {
    alert('_delegate.foo');
  }
};
 
var agregate = {
 
  delegate: _delegate,
 
  foo: function () {
    return this.delegate.foo.call(this);
  }
 
};
 
agregate.foo(); // delegate.foo
 
agregate.delegate = {
  foo: function () {
    alert('foo from new delegate');
  }
};
 
agregate.foo(); // foo from new delegate
```
面向对象的本质到底是什么？OOP的先驱者Alan Kay所说的[出处](http://lists.squeakfoundation.org/pipermail/squeak-dev/1998-October/017019.html)，系统的设计真正要在乎的是如何交互，关注的是行为。关注消息通讯的结构，性能，灵活性是我以后写代码时要考虑的。模块之间可否独立？RESTful？

> Smalltalk is not only NOT its syntax or the class library, it is not even about classes. I'm sorry that I long ago coined the term "objects" for this topic because it gets many people to focus on the lesser idea. The big idea is "messaging".

我被下面的代码惊艳到了。  

```
//Javascript
//集合类Set的构造函数
function Set() { 
    var _elements = arguments;
    //In方法：判断元素e是否在集合中 
    this.In = function(e) { 
        for (var i = 0; i < _elements.length; ++i) { 
            if (_elements[i] == e) return true; 
        } 
        return false; 
    }; 
}

//Join方法：对两个集合求交集
Set.prototype.Join = function(s2) { 
    var s1 = this; 
    var s = new Set(); 
    s.In = function(e) { return s1.In(e) && s2.In(e); } 
    return s; 
};

//Union方法：对两个集合求并集
Set.prototype.Union = function(s2) { 
    var s1 = this; 
    var s = new Set(); 
    s.In = function(e) { return s1.In(e) || s2.In(e); } 
    return s; 
};

var s1 = new Set(1, 2, 3, 4, 5); 
var s2 = new Set(2, 3, 4, 5, 6); 
var s3 = new Set(3, 4, 5, 6, 7); 
assert(false == s1.Join(s2).Join(s3).In(2)); 
assert(true == s1.Join(s2).Uion(s3).In(7));
```
## Functional Programing
函数式编程，现在的工作使用的场景不多，当然可以把它的思想用在平时的java编程中，或者使用java8的新特性。  
首先，我觉得是把要实现的逻辑，拆分成一个个函数逻辑，其实这在OO里面也是尽量把逻辑原子化，单单从形式上看不出来。主要的区别在于它讲究无状态性，正如皓哥说的：  
1）函数之间没有共享的变量。  
2）函数间通过参数和返回值来传递数据。  
3）在函数里没有临时变量。  
这个了解不多，以后学python使用到的时候再好好总结下。

##Conclusion
因为一个小需求，需要迁移页面，所以顺便学习了一下JS，收获还是挺多的，只是暂时实践的机会不多，行文比较仓促，查找到的资料很好，希望以后用的到。

- JS core
	- 执行栈，栈中每个EX包括this，vo，scope chain。其中this和vo在函数调用时产生，scope chain是在函数创建时决定的vo引用变量链
	- 查找变量，先通过scope chain查找，然后在通过prototype查找
- OOP的本质——对象的消息模型
- 怎样学习一门语言
	- 找一个好的老师，起码一定要找好的资料
	- 从语言的底层实现，设计着手可以看到更本质的东西
	- 作为OOP语言，它是怎么表示对象的，它是怎么管理内存的 
	- 再学一下它的特性
	
##Basics
- DOM（Document Object Model）, 就是把整个HTML页面映射为一个多层节点结构。
- script 元素
	- defer 延迟加载，脚本会被延迟到整个页面都解析完毕后在运行，只适用于外部脚本文件。
	- async 异步加载，但一定会在页面的Load事件前执行，只适用于外部脚本文件。
- null undefined 
	- null 代表一个空对象
	- undefined 检查有没有声明过
- window  document
	-  window指窗体。document指页面。document是window的一个子对象
	- 用户不能改变document.location(因为这是当前显示文档的位置)。但是,可以改变window.location (用其它文档取代当前文档)window.location本身也是一个对象,而document.location不是对象
	- window.onload() 是标准的DOM事件，会在页面的所有内容加载后触发（包括图片）。
	- $(document).ready() 是jQuery事件，会在document加载完成后触发。
	- 类型转换是通过valueOf方法

### Reference
1. http://coolshell.cn/articles/6731.html
2. http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html
3. JavaScript Closures for Dummies
4. JavaScript 高级程序设计
5. Secrets of the JavaScript Ninja
6. http://dmitrysoshnikov.com/
7. http://stackoverflow.com/questions/111102/how-do-javascript-closures-work
8. JavaScript: The Good Parts
9. JavaScript设计模式
10. High Performance JavaScript (Build Faster Web Application Interfaces)
11. http://www.cnblogs.com/weidagang2046/archive/2011/08/14/object-messaging-model.html
12. http://coolshell.cn/articles/10822.html#comment-1111518





