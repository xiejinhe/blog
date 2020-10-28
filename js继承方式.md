## 六种继承方式：原型链继承、构造函数继承、组合继承、原型式继承、寄生式继承、寄生组合式继承

写在前面：class继承有他的局限性，不利于代码的分层和模块的拆分，基于原型链去拓展方法可以很好地拆分功能模块，极大的提高代码的复用性。

例：首先创造一个超类型的构造函数`Super`，它拥有自己的实例属性`name`，以及原型链方法`getSuper`。
在创造一个子类构造函数`Sub`，下面我们将使用6中方法去分析如何让`Sub`继承`Super`，并讨论优劣。

> 函数式继承

1. 原型链继承

```javascript{.line-numbers}
function Super() {
    this.name = ["super"];
}
Super.prototype.getSuper = function() {
    return this.name;
}
function Sub() {};
Sub.prototype = new Super();

var sub1 = new Sub();
sub1.name.push("sub1");

var sub2 = new Sub();
sub2.name.push("sub2");

console.log(sub2.getSuper()); //["super", "sub1", "sub2"]
```
解析：将`Sub`的原型对象`Sub.prototype`指向`Super`的实例，然后创建两个`Sub`的实例`sub1`、`sub2`。这样我可以在`Sub`中继承`Super`的属性`name`以及原型链方法`getSuper`，然而在`sub1`中修改`name`时，`sub2`的`name`也受到影响。

小结：将子类的原型对象指向超类实例的方式称作是原型链继承，这种继承缺点：

- 超类的属性会被所有实例共享
- 子类的实例不能向超类构造函数传参

2. 构造函数继承

```javascript{.line-numbers}
function Super() {
    this.name = name;
}
Super.prototype.getSuper = function() {
    return this.name;
}
function Sub(name) {
    Super.call(this, name);
}

var sub1 = new Sub('tom');
// console.log(sub1.getSuper())  Uncaught TypeError
console.log(sub1.name) // tom

var sub2 = new Sub();
console.log(sub2.name); //undefined

var sup = new Super();
console.log(sup.getSuper()); //undefined
```
解析：在`Sub`中使用`call`去调用`Super`时，继承了`Super`的所有实例属性。在实例`sub1`、`sub2`中，各自对`name`的修改也互不影响，做到了属性不共享，同时子类的实例也能向超类构造函数传参。这种方式的缺点显而易见：
- 不能继承原型链方法：`console.log(sub1.getSuper())  //Uncaught TypeError`

3. 组合继承
```javascript{.line-numbers}
function Super(name) {
    this.name = name;
}
Super.prototype.getSuper = function() {
    return this.name;
}
function Sub(name) {
    //为了Sub继承Super实例属性
    Super.call(this, name);  //第二次调用Super
}
//为了Sub继承Super原型链属性
Sub.prototype = new Super();  //第一次调用Super
Sub.prototype.constructor = Sub;

var sub1 = new Sub("tom");
console.log(sub1.getSuper()); // tom
console.log(sub1.name); //tom
console.log(sub1 instanceof Sub); //true
console.log(sub1 instanceof Super); //true

var sub2 = new Sub();
console.log(sub2.name) //undefined
```
解析：在子类`Sub`中，我们仍然使用`call`去继承超类的属性，同时也使用原型链的继承方式去继承原型链的方法和属性。这样我们弥补了以上两种继承方式的三种不足。唯一美中不足的是：

- 调用了两次超类的构造函数

第一次是在使用原型链继承`Sub.prototype = new Super()`时，调用了一次超类构造函数，第二次是在实例化Sub `new Sub()`，然后在`Sub`内使用`call`方法时，又调用了一次超类构造函数。并且在之后的每次实例化子类`sub1`、`sub2`...的过程中，都会调用超类的构造函数，这种方式显然不是我们要的。

以上三种继承（原型链继承、构造函数继承、组合继承）方式都属于函数式继承。

---------------------------------------分界线-----------------------------------------

> 原型式继承

4. 原型式继承

例：
```javascript{.line-numbers}
function object(o) {
    function F() { };
    F.prototype = o;
    return new F();
}
```
首先创造了一个临时的构造函数`F`，将`F`的原型指向传进来的对象，在返回`F`的实例。等等，是不是和原型链继承很类似？这样，我们就完成了一次对对象的浅拷贝。来看个栗子：

```js{.line-numbers}
function object(o) {
    function F() { };
    F.prototype = o;
    return new F();
}

var person = {
    name: "Nicholas",
    friends: ["Sherlly", "Van"]
}

// 在传入一个参数的情况下，Object.create()和object()相同
// var people1 = Object.create(person);

var people1 = object(person);
people1.name = "Greg";
people1.friends.push("Rob");

var people2 = object(person);
people2.name = "Linda";
people2.friends.push("Barbie");

console.log(person.name); // Nicholas
console.log(person.friends); // ["Sherlly", "Van", "Rob", "Barbie"]
```
解析：原型式继承和原型链继承类似，区别是一个是`对对象进行复制`，另一个是`对构造函数进行继承`。缺点也是一致的：

- 属性会被共享

5. 寄生式继承

基于4，还有一种对于对象复制的方式：寄生式继承。实质是基于4的一层封装

```js{.line-numbers}
function object(o) {
    function F() { }
    F.prototype = o;
    return new F();
}
function createAnother(o) {
    var clone = object(o)
    clone.sayHi = function () {
        console.log("hi")
    }
    return clone;
}
var person = {
    name: "Nicholas",
    friends: ["Sherlly", "Van"]
}
var anotherPerson = createAnother(person);
anotherPerson.sayHi();
```
解析：可以通过这种方式实现子类方法`sayHi`的复用。通过`createAnother`创造出来的对象，都拥有`sayHi`方法。这种封装方式和工厂模式类似。

---------------------------------------分界线-----------------------------------------

6. 寄生组合式继承

思考一下，在方式3-组合继承中，如果我们需要优化一次调用，那一定是第一次调用，对于原型链继承的优化，怎么优化呢？方式4-原型式继承恰巧满足我们的需要。

`Sub.prototype = new Super()`，实质上就是完成一次对超类原型对象的拷贝，看代码:

```javascript{.line-numbers}
function object(o) {
    function F() {};
    F.prototype = o;
    return new F();
}

function inheritPrototype(subType, superType) {
    var clone = object(superType.prototype); //复制超类的原型对象
    clone.constructor = subType; //将构造函数指向子类
    subType.prototype = clone;
}

function Super(name) {
    this.name = name;
}
Super.prototype.getSuper = function() {
    return this.name;
}
function Sub(name) {
    Super.call(this, name); //第二次调用
}
// 优化前：
// Sub.prototype = new Super(); //第一次调用
// Sub.prototype.constructor = Sub;
// 优化后：
inheritPrototype(Sub, Super);

var sub1 = new Sub("tom");
console.log(sub1.getSuper()); //tom
console.log(sub1.name); //tom
console.log(sub1 instanceof Sub); //true
console.log(sub1 instanceof Super); //true

var sub2 = new Sub();
console.log(sub2.name); //undefined;
```
我们封装了`inheritPrototype`这样一个函数，首先利用`object`（或`Object.create()`）复制出超类的原型对象，然后将原型对象的构造函数指向自身（`clone.constructor = subType`，`constructor`相当于一张身份证，身份证上的名字一定得是自己），最后将拷贝出来的对象塞给子类的原型对象。至此，完成了子类对超类的原型对象的继承。

<span style="color:red;font-size:20px">总结：</span>

结合方式一的原型链继承和方式二的构造函数继承，衍生出方式三的组合继承。为了优化组合继承，引入了方式四原型式继承，最终得到方式六寄生组合式继承。

--------------------------------------完结--------------------------------------



相关推荐:

![原型链结构图](http://s3.51cto.com/wyfs02/M00/75/71/wKiom1Y5Z6-znxFpAAJXoj1qs6I106.jpg)

[这次，彻底弄清js的继承方式](https://www.jianshu.com/p/4ad2b06d638d)