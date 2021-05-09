#### 数据类型分类

基本类型

- String: 任意字符串
- 1Number: 任意数字
- undefined
- null

对象类型:

- Object: 任意对象
- Function: 一种特别的对象(可以执行)
- Array: 数值下标属性,内部数据是有序的

#### 判断类型

- typeof
  - typeof返回的是数据类型的字符串表达
- instanceof
  - 判断对象的类型
- ===

#### null与undefined的区别

undefined是以声明但未初始化的.(此处存疑)

null是一个特殊的值,把null值赋给变量可以表明该变量是个对象类型.

#### 闭包

```
function fn1() {
    var a = 1;
    function fn2() {
        a++;
        console.log(a);
    }
    return fn2;
}
debugger

var f = fn1();
f();
f();

var f2 = fn1();
f2();
f2();
```

