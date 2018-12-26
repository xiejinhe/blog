# JS原型、原型链深入理解

> 目录

* 原型介绍
* 初识原型
* 创建规则
* 初识Object
* 初识Function
* "prototype"和"\__proto__"混淆分析
* 原型链
* 趁热打铁
* 总结及相关推荐

原型是JavaScript中一个比较难理解的概念，原型相关的属性也比较多，对象有”[[prototype]]”属性，函数对象有”prototype”属性，原型对象有”constructor”属性。

`初识原型`

在JavaScript中，`原型也是一个对象，通过原型可以实现对象的属性继承`，
JavaScript的对象中都包含了一个”`[[Prototype]]`”内部属性，这个属性所对应的就是该对象的原型。

`“[[Prototype]]”`作为对象的内部属性，是不能被直接访问的。所以为了方便查看一个对象的原型，
Firefox和Chrome中提供了`__proto__`这个非标准（不是所有浏览器都支持）的访问器
（ECMA引入了标准对象原型访问器”Object.getPrototype(object)”）。`(CODE TEST)`

在JavaScript的原型对象中，还包含一个”constructor”属性，这个属性对应创建所有指向该原型的实例的构造函数

##__规则__
---
    在JavaScript中，每个函数 都有一个prototype属性，当一个函数被用作构造函数来创建实例时，

    这个函数的prototype属性值会被作为原型赋值给所有对象实例（也就是设置 实例的`__proto__`属性），

    也就是说，所有实例的原型引用的是函数的prototype属性。(****`只有函数对象才会有这个属性!`****)

`new 的过程分为三步 (CODE TEST)`

```javascript
    var p = new Person('张三',20);
```
1. var p={}; 初始化一个对象p。  

2. p.\_proto\_=Person.prototype;，将对象p的 \__proto__ 属性设置为 Person.prototype

3. Person.call(p,"张三",20);调用构造函数Person来初始化p
code
```javascript
function objectFactory() {

    var obj = new Object(),

    Constructor = [].shift.call(arguments);

    obj.__proto__ = Constructor.prototype;

    var ret = Constructor.apply(obj, arguments);

    return typeof ret === 'object' ? ret : obj;

};
var person = objectFactory(Otaku, 'Kevin', '18');
```

作者：冴羽
链接：https://juejin.im/post/590a99015c497d005852cf26
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
                       

##初识Object##
---
Object对象本身是一个函数对象。`(CODE TEST)` 既然是Object函数，就肯定会有prototype属性，

所以可以看到”Object.prototype”的值就是”Object {}”这个原型对象。反过来，

当访问”Object.prototype”对象的”constructor”这个属性的时候，就得到了Obejct函数。

另外，当通过”Object.prototype.\_proto_”获取Object原型的原型的时候，将会得到”null”，

也就是说”Object {}”原型对象就是原型链的终点了。

##初识Function##
---
如上面例子中的构造函数，JavaScript中函数也是对象，所以就可以通过`_proto_`查找到构造函数对象的原型

Function对象作为一个函数，就会有`prototype`属性，该属性将对应”function () {}”对象。

Function对象作为一个对象，就有`__proto__`属性，该属性对应”Function.prototype”，

也就是说，”Function.\__proto\__ === Function.prototype”



##在这里对”prototype”和”__proto__”进行简单的介绍：
---
对于所有的对象，都有`__proto__`属性，这个属性对应该对象的原型

对于函数对象，除了`__proto__`属性之外，还有`prototype`属性，当一个函数被用作构造函数来创建实例时，

该函数的prototype属性值将被作为原型赋值给所有对象实例（也就是设置实例的`__proto__`属性）


![](http://s3.51cto.com/wyfs02/M00/75/71/wKiom1Y5Z6-znxFpAAJXoj1qs6I106.jpg '原型链结构图')

##**原型链**##
---

因为每个对象和原型都有原型，对象的原型指向原型对象，
而父的原型又指向父的父，这种原型层层连接起来的就构成了原型链。

##**属性查找**##
---

当查找一个对象的属性时，JavaScript 会向上遍历原型链，直到找到给定名称的属性为止，

到查找到达原型链的顶部（也就是 `Object.prototype`），如果仍然没有找到指定的属性，就会返回 undefined。

```javascript
    function Person(name, age){ 
        this.name = name; 
        this.age = age; 
    } 
 
Person.prototype.MaxNumber = 9999; 
Person.__proto__.MinNumber = -9999; 
 
var will = new Person("Will", 28); 
 
console.log(will.MaxNumber); 
// 9999 
console.log(will.MinNumber); 
// undefined 
```

在这个例子中分别给”Person.prototype “和” Person.__proto__”这两个原型对象添加了”MaxNumber “和”MinNumber”属性，

这里就需要弄清”prototype”和”__proto__”的区别了。

“Person.prototype “对应的就是Person构造出来所有实例的原型，也就是说”Person.prototype “属于这些实例原型链的一部分，

所以当这些实例进行属性查找时候，就会引用到”Person.prototype “中的属性。


##**对象创建方式影响原型链**##
---
```javascript
    var July = { 
    name: "张三", 
    age: 28, 
    getInfo: function(){ 
        console.log(this.name + " is " + this.age + " years old"); 
    }, 
} 
    console.log(July.getInfo()); 
```
当使用这种方式创建一个对象的时候，原型链就变成下图了

July对象的原型是”Object.prototype”也就是说对象的构建方式会影响原型链的形式。

![](http://s1.51cto.com/wyfs02/M01/75/6F/wKioL1Y5Z_TQmz04AADKZF3NiH4003.jpg '{}对象原型链结构图')


##**综图所述**##
---
所有的对象都有`__proto__`属性，该属性对应该对象的原型(实例的__proto__属性（原型）等于其构造函数的prototype属性)

所有的函数对象都有`prototype`属性，该属性的值会被赋值给该函数创建的对象的`_proto_`属性

所有的原型对象都有`constructor`属性，该属性对应创建所有指向该原型的实例的构造函数

函数对象和原型对象通过`prototype`和`constructor`属性进行相互关联

---

## 2018年12月腾讯xxx一面面试题（原型链相关）
> 面试题
1. `有一个类如下`
```javascript
function Person(name) {
    this.name = name
}
let p = new Person('Tom');

```
* p.__proto__分别等于？ 答： Person.prototype
* Person.__proto__等于什么? 答： Function.prototype

解析：`实例的__proto__属性（原型）等于其构造函数的prototype属性。实例p的构造函数为Person，而Person的构造函数为Function`

---


2. `有一个类如下`
```javascript
var foo = {},
    F = function(){};
Object.prototype.a = 'value a';
Function.prototype.b = 'value b';

console.log(foo.a)   //'value a'
console.log(foo.b)   // undefined
console.log(F.a)     // value a
console.log(F.b)     // value b
```

解析：`Function.prototpye.__proto__ 指向Object.prototype`

---

3. `typeof和instanceof的区别`

    typeof ：在javascript中，判断一个变量的类型常常会用typeof运算符，在使用typeof运算符时采用引用类型存储会出现一个问题，无论引用的是什么类型的对象，都返回"object"

    instanceof ：运算符用来测试一个对象在其原型链中是否存在一个构造函数的prototype属性。（obejct instanseof constructor） instanceof表示的就是一种继承关系，或者原型链的结构。
    ```javascript
    function Person (name){
        this.name = name || 'haha';
        this.getName = function () {
            return this.name;
        }
    }
    function SubPerson(name) {
        
    }
    SubPerson.prototype = Person.prototype;
    SubPerson.prototype.constructor = SubPerson;
    var sub = new SubPerson('北京');
    // true
    sub instanceof SubPerson; 
    // true
    sub instanceof Person;
    // true sub.__proto__.constructor.__proto__ === Function.prototype
    SubPerson instanceof Function;
    // true Function.prototype.__proto__ === Object.prototype
    SubPerson instanceof Object; 
    ```
---

4. `new和instanceof的内部机制`

    > new ：
        >> 1. 创建一个新对象，同时继承对象类的原型，即Person.prototype;
        >> 2. 执行对象类的构造函数，同时该实例的属性和方法被this所引用，即this指向新构造的实例
        >> 3. 如果构造函数ruturn一个新"对象"，那么这个对象就会取代整个new出来的结果。
        如果构造函数没有return对象，那么就会返回步骤1所创建的对象，即隐式返回this。
    ```javascript
        let p = (function () {
            let obj = {};
            obj.__proto__ = Person.prototype;
            // 其他赋值语句
            return obj
        })
    ```

    > instanceof ： x instanceof y内部实现如下
    ```javascript
    var flag = false;
    while(x.__proto__ !== null) {
        if(x.__proto__ === y.prototype){
            flag = true;
            break;
        }
        x.__proto__ = x.__proto__.__proto__;
    }
    if(x.__proto__ == null){ return flag };
    ```
    x会一直沿着隐式原型链__proto__向上查找到x.__proto__.__proto__...... = y.prototype为止，
    如果找到则返回true，则x是y的一个实例，否则反之。

---

5. `说出以下代码输出结果`
    ```javascript
    function F() {}
    function O() {}

    O.prototype = new F();
    var o = new O();
    var f = new F();

    // true
    console.log(o instanceof O);
    // true
    console.log(o instanceof F);
    // true
    console.log(o.__proto__ === O.prototype);
    // true
    console.log(f.__proto__ === F.prototype);
    // true 
    console.log(o.__proto__.__proto__ === F.prototype);

    ```
    
---

6. `和上面一题有些变化，改变O的原型在后面`

    ```javascript
    function F() {}
    function O() {}

    var obj = new O();
    O.prototype = new F();

    console.log(obj instanceof O); // false
    console.log(obj instanceof F); // false
    console.log(obj.__proto__ === O.prototype); // false
    console.log(obj.__proto__.__proto__ === F.prototype); // false
    // Object.getPrototypeOf(O) === Function.prototye   //true
    Object.getPrototypeOf 获取对象的原型
    ```

##__相关推荐__

[JavaScript 原型概念深入理解](http://developer.51cto.com/art/201511/496178.htm)









