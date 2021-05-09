### DOM

把html中的每一个标签,都用一个对象(节点)描述.而模型包含了所有的节点,包括各个节点直接的关系.

浏览器已经提供了文档节点对象(document),它可以再页面中直接使用

基本使用

```js
    var dom1 = document.getElementById('aa');
    dom1.innerText = '你好DOM';
```

### 事件

添加事件:

```js
    var btn = document.getElementById('btn');
// btn['onclick'] = function (ev) { // 这样也行
    btn.onclick = function (ev) {
        console.log('ev: ', ev);
    }
```

### 查询元素节点

#### 通过document对象调用

```js
    document.getElementById();
    document.getElementsByTagName();
    document.getElementsByName();
	document.body; //获取body元素
	document.documentElement; // 获取html根标签
	document.all; // 获取页面中所有元素,已过时
	document.getElementsByTagName("*"); // 也是获取所有元素

    // 获取类为.dd标签下的第一个span元素(css选择器)
    var res2 = document.querySelector('.dd span');
    console.log('res2: ',res2);
    // 获取类为.dd标签下的所有span元素
    var res3 = document.querySelectorAll('.dd span');
    console.log('res3: ',res3);
```

#### 通过具体的元素节点调用(HTML中的文本也是节点)

获取节点子元素,父元素

```js
	node.getElementsByClassName();
    node.getElementsByTagName();
    node.getElementsByTagNameNS();
    node.childNodes // 当前节点的所有子节点,包括文本节点
	node.children // 所有html标签的子节点
	node.parentNode // 获取父元素
    node.firstChild
    node.lastChild

```

获取标签中的内容

html代码:

```html
<div id="d1">
    <span id="s1">a</span>
    <span id="s2">b</span>
	<span id="s3">c</span>
</div>
```

innerHTML会返回html文本文件中标签中的所有内容,而innerText会返回标签中的文本内容

```js
		var ch = document.getElementById('s2');
        var parent = ch.parentNode;
		\\ <span id="s1">a</span> <span id="s2">b</span>...
        console.log('parent.innerHTML: ',parent.innerHTML);
        console.log('parent.innerText: ',parent.innerText);\\ a b
```

获取上一个和下一个节点(都是文本)

```js
        var d1 = document.getElementById('d1');
        var allNodes = d1.childNodes;
        var defNodes = d1.children;
        console.log('d1.childNodes: ', allNodes); // text
        console.log('d1.children: ', defNodes);// text
```

获取上一个和下一个元素

```js
var res3 = ch.previousElementSibling;
var res4 = ch.nextElementSibling;
console.log('res3: ',res3);
console.log('res4: ',res4);
```

获取元素中的值

```js
var ch = document.getElementById('inp');
console.log('value: ', ch.value);
```

### 对象节点的增删改

html

```html
<body>
<div id="d1" class="dd" style="border: 1px solid red">
    <span id="s1">a</span>
    <span id="s2">b</span>
    <span id="s3">c</span>
</div>
</body>
```

js,新建一个div并添加到body中

```js
// 创建一个div元素节点
var divEle = document.createElement('div');
// 添加属性
divEle.setAttribute('style', 'border: 1px solid pink');
// 创建文本节点
var textEle = document.createTextNode('于谦的父亲们');
// 将文本节点放进div元素中
divEle.appendChild(textEle);
// 将div添加到body中
document.body.appendChild(divEle);
```

新建一个文本节点并添加到`d1`前

```js
// 创建文本节点
var textEle = document.createTextNode('天上人间');
var d1 = document.getElementById('d1');
// 将节点放到d1前
document.body.insertBefore(textEle,d1);
```

其他操作

```js
// 替换子节点
parentEle.replaceChild(newCh,oldChild)
// 父元素删除子节点
parentEle.removeChild(Node);
// 自己找到父节点删除自己
subEle.parentNode.removeChild(subEle);
```

使用innerHTML也能对dom增删改(但是会修改所有的子元素)

```js
node.innerHTML += '<div>hello</div>';
```

### 操作元素的css属性

```js
var d1 = document.getElementById('d1');
// 通过js设置的css都是内联样式(如果样式中使用了!important,则js则修改无效)
// 通过js读取的也都是内联样式
d1.style.width = '100px';
d1.style.backgroundColor = 'pink';
// 获取元素当前的显示样式
var d1 = document.getElementById('d1');
// 获取元素当前的样式,window的方法
// 参数: 1. 要获取的样式的元素 2. 可以传递一个伪元素,一般为null
// 该方法会返回一个包含当前元素显示的css属性的对象(不是内联啥的,就是显示的)
var res = getComputedStyle(d1,null);
console.log('res: ',res.width);
```

### 元素的事件

方式一. 直接使用元素.事件设置事件(只能设置一个)

鼠标移动事件

html

```html
<div id="d1" class="dd" style="width:300px;height:300px; border: solid 1px red"></div>
<div id="d2" style="width: 200px;height: 100px;border: solid 1px red"></div>
```

js

```js
        var d1 = document.getElementById('d1');
        var d2 = document.getElementById('d2');
        d1.onmousemove = function (ev) {
            var cX = ev.clientX;
            var cY = ev.clientY;
            d2.innerText = 'X: '+cX+' Y: '+cY;
        };
```

方式二: 使用addEventListener()方法(可以添加多个响应函数)

```js
// 创建一个div标签
var dom = document.getElementById('d1');
/*
 * 参数:
 * 1. 事件的字符串
 * 2. 回调函数
 * 3. 是否再捕获节点触发事件,需要一个布尔值,一般传false
 */
dom.addEventListener('click', function () {
    alert('running');
},false);
```

#### 推拽div练习

```html
<div id="d1" class="dd" style="height: 100px;width: 100px;border: 1px solid red"></div>
<script>
    drag();
    function drag() {
        const dom = document.getElementById('d1');
        // 设置为相对定位
        var positionType = dom.style.position;
        if (positionType !== 'relative') {
            dom.style.position = 'relative';
        }
        // 鼠标按下事件
        dom.onmousedown = function (ev1) {
            // dom当前的坐标
            var computedStyle = getComputedStyle(dom, null);
            var left_ = computedStyle.left;
            var top_ = computedStyle.top;

            var left = left_.substr(0,left_.length-2)-0;
            var top = top_.substr(0,top_.length-2)-0;
            // 鼠标当前的坐标
            var cy = ev1.clientY;
            var cx = ev1.clientX;

            var xDif = cx - left;
            var yDif = cy - top;
            dom.onmousemove = function (ev) {
                // 鼠标的坐标
                var clientX = ev.clientX;
                var clientY = ev.clientY;
                dom.style.left = clientX - xDif + 'px';
                dom.style.top = clientY - yDif + 'px';
            }
        };

        document.addEventListener('mouseup', function () {
            dom.onmousemove = null;
            document.mouseup = null;
        }, false);
    }
</script>
```

键盘事件

```js
var d1 = document.getElementById('d1');
var sub = d1.children[0];
/*
 * 键盘主要绑定再可以获取到焦点的事件
 * onkeydown
 * - 按键按下被触发
 * - 如果按着不放事件会连续触发
 * - 第一次触发和第二次触发间隔事件较长
 * - 事件对象中还提供了altKey,ctrlKey,shiftKey如果按下返回true
 * onkeyup
 *
 */
window.onkeydown = function (ev) {
    sub.innerText = 'code: ' + ev.code;
}
```

键盘方向键移动div练习

```js
function go() {
    var d1 = document.getElementById('d1');
    // 确定为相对定位
    var positionType = d1.style.position;
    if (positionType !== 'relative') {
        d1.style.position = 'relative';
    }
    // 键盘按下监听事件
    window.onkeydown = function (ev) {
        var key = ev.key;
        switch (key) {
            case 'ArrowUp': {
                d1.style.top = d1.offsetTop - 10 + 'px';
                break;
            }
            case 'ArrowDown': {
                d1.style.top = d1.offsetTop + 10 + 'px';
                break;
            }
            case 'ArrowLeft': {
                d1.style.left = d1.offsetLeft - 10 + 'px';
                break;
            }
            case 'ArrowRight': {
                d1.style.left = d1.offsetLeft + 10 + 'px';
                break;
            }
            default: {
                return;
            }
        }
    }
}
```

