### JS对象

对象分类:

1. 内建对象
2. 宿主对象
3. 自定义对象

#### 自定义对象

两种方式:

```js
// 使用子面量(大括号)
var a  = {};
// 使用new 关键字
var b = new Object();
```

使用:

```js
// 添加属性
var a = new Object();
// 使用对象点
a.name = 'riden';
// 使用中括号
a['age'] = 21;

// 删除属性
delete a['name'];
delete a.name;

// 判断对象是否有某一个属性
var isContain = 'name' in a;
```

#### 基本属性类型和引用类型对象

基本类型

值都放在栈中

```js
var a = 123;
var b = a;
a++;
a; // 124
b; // 123
```

引用类型

例如`var a = new Object{}`

那么a引用保存到栈中,key是a,value是内存地址.而对象则保存在堆中

#### 枚举对象中的属性

```js
for(var key in obj){
    console.log('key: ',key);
    console.log('value: ',obj[key]);
}
```

### 函数

函数也是一个对象

创建方式

```js
// 使用构造函数的方式
var func = new Function();
var func2 = new Function("console.log('一起来玩呀')");
// 使用函数声明创建
function func3(){}
var func4 = new function(){};
```



传参问题:

如果参数列表缺少时,则默认值为undefined,如果方法没有返回值,则返回值为undefined



立即执行函数(只执行一次):

```js
(function(){
    alert('执行了匿名函数')
})();

(function(a,b){return a+b})(1,2);
```

#### 函数的方法

`call()`和`apply()`

- 这两个方法都是函数对象的方法,需要通过函数对象来调用.
- 当对函数调用call()和apply()都会调用函数执行
- 在调用call和apply可以将一个对象指定为第一个参数,此时这个参数就会成为函数执行时的this
- call()方法可以将实参在对象后一次传递`call(obj,'riden',123);`
- apply()方法需要将实参封装到一个数组中统一传递例如:`fun.apply(obj,[2,3])`

#### 关于:arguments对象

再调用函数对象时,浏览器都会传递两个隐含的参数:

1. 函数的上下文对象this
2. 封装实参的对象arguments
   - arguments是一个类数组对象(数组对象字类),它也可以通过索引操作数据,也可以获取长度.
   - 及时不定义形参,也可以通过arguments来使用实参
   - arguments里有一个属性`callee`,这个属性对应当前函数对象



### 作用域

js中有两种作用域,全局作用域和函数作用域.

#### 一. 全局作用域

- 直接编写在script标签中的JS代码,都在全局作用域
- 全局作用域在页面打开时创建,页面关闭时销毁
- 在全局作用域中有一个全局对象window
  - 它代表的时一个浏览器窗口,它由浏览器创建我们可以直接使用
  - 在全局作用域中创建的变量都会作为window对象的属性保存
  - 创建的函数都会作为window的方法保存
- 全局作用域中的变量都是全局变量,页面的任意部分都能访问到.

```js
// 声明变量的特殊方式
a = 123; // 相当于window.a = 123
alert(a);
alert(window.a);
```

#### 二. 函数作用域

- 调用函数式创建函数作用域,函数执行完后,作用域销毁
- 没调用一次就创建一个新的函数作用域
- 函数作用域中可以访问全局作用域,反之则不行
- 函数作用域中查找变量时: 会先查找本地的函数作用域找,没有,就会去上一级作用域找.
- 在函数中要指定访问全局的变量,可以使用window.a.
- 函数作用域中也有(变量,函数)声明提前特性.

#### 变量的声明提前:

 使用var关键字声明的变量,会在所有代码执行之前被声明(但是不会赋值)

```js
代码1
代码2
var a = 1111;
// 上面代码实际上会变成下面这样
var a;
代码1
代码2
a = 1111;
```

#### 函数的声明提前

使用函数声明形式声明的函数,它会在所有的代码执行之前被创建;

所以可以在函数声明前调用

```js
sayHi();
function sayHi(){
    alert('hello');
} 
```

但如果使用var形式创建(函数表达式)的函数就不行

```js
// 下面代码报错
sayHi();
var sayHi = function(){
    alert('hello');
}
```

### this关键字

解析器在调用函数每次都会向函数内部传递一个隐含的参数:this

this指向的是一个函数执行的上下文对象.

更具函数调用方式的不同,this会指向不同的对象

- 以函数的形式调用,this永远是window
- 以方法的形式调用,this就是调用方法的对象

### 使用工厂方法创建对象

```js
    function personFactory(name,age){
        var person = {};
        person['name'] = name;
        person['age'] = age;
        return person;
    }

    var p1 = personFactory('riden',12);
    var p2 = personFactory('张三',11);
    console.log(p1===p2); // false
```

上面使用工厂方法创建的对象,使用的构造函数都是Object,所以创建的都是Object这个类型,这使得我们无法区分出不同类型的对象.

### 构造函数

创建一个构造函数,构造函数就是一个普通的函数,创建函数和普通函数没有区别,不过习惯上首字母大写.

构造函数和普通函数的区别就是调用方式的不同

普通函数是直接调用,而构造函数则需要使用new关键字来调用.

```js
    function Person(name,age){
        this.name = name;
        this.age = age;
    }
    
    function Dog(){}
    var p1 = new Person('riden',21);
    var dog = new Dog();
    console.log(p1 instanceof Person);//true
    console.log(p1 instanceof Object);//true
    console.log(dog instanceof Person);//false
    console.log(dog instanceof Object);//true
```

构造函数的执行流程

1. 立即创建一个新的对象
2. 将新建的对象设置为函数中的this
3. 逐行执行函数中的代码
4. 将新建的对象作为返回值返回

### 原型对象

首先看看构造函数中的方法

如下的创建方式,每个实例都有一个独立的方法.

```js
    function Person(name,age){
        this.name = name;
        this.age = age;
        // 向对象中添加一个方法
        this.sayName = function(){
            alert('my name is '+name);
        }
    }
```

那么如何才能让所有对象公用一个sayName函数呢,可以使用原型对象.

我们所创建的每一个函数,解析器都会向函数中添加一个prototype

```js
function Dog(){}
console.log(Dog.prototype)
```

如果函数作为普通函数调用prototype,则没有任何用处

当函数以构造函数形式调用时,它所创建的每一个对象都会有一个隐含属性,指向构造函数的原型对象,我们可以通过`__proto__`来访问

原型对象就相当于一个公共区域,所有同一个类的实例都可以访问到这个原型对象,我们就可以将对象中共有的内容,统一设置到原型对象中.

我们访问对象的一个属性或方法时,它会现在对象自身中寻找,如果有则使用,否则就去原型对象中寻找,再没找到就去上一级作用域.

所以Person中的公共方法可以这样添加:

```js
Person.prototype.sayName = function(){
    alert(this.name);
}
```

检查对象中的属性.

使用in检查对象中是否含有某个属性时,如果对象中没有但是原型中有,也会返回true.

如果只检查对象中的属性,可以使用hasOwnProperty方法判断

```js
    console.log('res1: ','sayName' in p1);
    console.log('res2: ',p1.hasOwnProperty('sayName'));
```

这里多了一个hasOwnProperty方法,这个方法既不在对象中定义,也不再对象原型中,而是在该对象原型对象中的原型对象中.

```js
    var p1 = new Person('riden', 11);
    console.log('res1: ',p1.hasOwnProperty('hasOwnProperty')); // false
    console.log('res2: ',p1.__proto__.hasOwnProperty('hasOwnProperty')); // false
    console.log('res3: ',p1.__proto__.__proto__.hasOwnProperty('hasOwnProperty')); //true
```

其实每个对象都有它的原型对象,而原型对象本身也是对象,所以原型对象也有原型对象.那这样岂不是有无数个对象.当然不是.

当我们使用一个对象的属性和方法时.查找顺序如下:

1. 从当前对象中查找
2. 从当前对象的原型对象中查找
3. 从当前对象的原型对象的原型对象中查找,直到找到Object类的原型对象.在没有就返回null

#### 重写原型对象中的方法

直接在对象的原型中重写该方法.

### 数组

创建:

```js
var arr = new Array();
var arr = new Array(10);
var arr2 = [];
```

数组中的方法:

```js
    var arr = [1, 2, 3, 4, 5];
    // 添加,返回新长度
    var newLength = arr.push(1, 2, 3, 'aa');
    var lestElement = arr.pop(); // 删除数组最后一个元素,返回删除的元素
    newLength = arr.unshift('1', 2, 4); // 向数组开头添加一个或多个元素
    var firstElement = arr.shift(); // 删除数组的第一个元素
```



