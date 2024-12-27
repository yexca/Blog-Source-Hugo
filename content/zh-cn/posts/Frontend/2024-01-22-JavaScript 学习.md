---
slug: 148
# layout: post
title: JavaScript 学习
author: yexca
date: 2024-01-22T19:51:24+08:00
# permalink: /archives/148
categories:
    - Front-end
    - JavaScript
---

JS 是一门跨平台、面向对象的脚本语言，是用来控制网页行为的，使网页可交互

## JS 引入方式

分为内部脚本和外部脚本两种

### 内部脚本

将 js 代码定义在 HTML 页面中

* JS 代码必须位于 `<script></script>` 标签之间
* 在 HTML 文档中，可以在任意地方，放置任意数量的 `<script>`
* 一般会把脚本置于 `<body>` 元素的底部，可以改善显示速度

```html
<script>
	alert("Hello JavaScript")
</script>
```

### 外部脚本

将 JS 代码定义在外部 JS 文件中，然后引入到 HTML 页面中

* 外部 JS 文件中，只包含 JS 代码，不包含 `<script>` 标签

```html
<html>
    <head>
        <script src="./js/1.js"></script>
    </head>
</html>
```

JS 文件内容

```javascript
alert("Hello JavaScript")
```

## JS 基础语法

区分大小写，每行结尾的分号可有可无。注释有两种

```javascript
// 单行注释

/*
	多行注释
*/
```

大括号表示代码块

```javascript
// 判断
if(count==3){
    alert(count);
}
```

### 输出语句

可以输出到警告框、HTML 或控制台

```javascript
// 浏览器弹出警告框
window.alert("Hello from alert")
// 写入HTML在浏览器展示
document.write("Hello from HTML")
// 写入浏览器控制台
console.log("Hello from console")
```

### 变量

JS 是一门弱类型语言，变量可以存放不同类型的值，变量名需要遵循如下规则：

* 组成字符可以是任意字母、数字、下划线或者美元符号
* 数字不能开头
* 建议使用驼峰命名

定义变量有三个关键字 vat、let 和 const

#### var

var 是 variable 的缩写，声明的变量为全局变量，可以重复定义

```javascript
// 重复声明
var a = 1;
var a = 'A';
alert(a);

// 全局变量
{
    var b = 'B';
}
alert(b);
```

#### let

在 ECMAScript6 新增，所声明的变量只在 let 所在的代码块内有效，且不允许重复声明

```javascript
let a = 'A';
alert(a);

// 局部变量
{
    let a='A';
}
alert(a); // 无输出，且在控制台报错
```

#### const

用来声明一个只读的常量，一旦声明，无法被改变

```javascript
const a = 'A';
a = 1;
alert(a); // 无输出，且在控制台报错
```

### 数据类型

JS 中分为原始类型和引用类型，即基本数据类型与对象

原始类型有五种：

* number：数字 (整数、小数、NaN (Not a Number))
* string：字符串，单双引皆可
* boolean：布尔。true 和 false
* null：对象为空
* undefined：当声明的对象未初始化时，该变量默认值是 undefined

使用 `typeof` 运算符可以获取数据类型

```javascript
// number
console.log("number 类型");
console.log(typeof 3);
console.log(typeof 3.14);

// string
console.log("\nstring 类型");
console.log(typeof 'A');
console.log(typeof "string");

// boolean
console.log("\nboolean 类型");
console.log(typeof true);
console.log(typeof false);

// null - object
console.log("\nnull-object 类型");
console.log(typeof null);

// undefined
var a;
console.log("\nundefined 类型");
console.log(typeof a);
```

> 您也许会问，为什么 typeof 运算符对于 null 值会返回 "Object"。这实际上是 JavaScript 最初实现中的一个错误，然后被 ECMAScript 沿用了。现在，null 被认为是对象的占位符，从而解释了这一矛盾，但从技术上来说，它仍然是原始值
>
> Reference: <https://www.w3school.com.cn/js/pro_js_primitivetypes.asp>

### 运算符

* 算数运算符：+、-、*、/、%、++、--
* 赋值运算符：=、+=、-=、*=、/=、%=
* 比较运算符：>、<、>=、<=、!=、==、===
* 逻辑运算符：$$、||、!
* 三元运算符：condition ? true : false

> == 与 ===
>
> == 会进行类型转换，而 === 不会进行类型转换，即类型与值都相等才为 true

```javascript
var a = 20;
var aStr = "20";
var aInt = 20;

console.log(a==aStr); // true
console.log(a===aStr);// false
console.log(a===aInt);// true
```

### 类型转换

字符串转换为数字使用 `parseInt()` 函数即可

转换是从第一个字符开始，直到遇到非数值。若开始非数值，则转为 `NaN`

```javascript
var a = "12";
var b = "12A34";
var c = "A34";

console.log(parseInt(a)); // 12
console.log(parseInt(b)); // 12
console.log(parseInt(c)); // NaN
```

其他类型转为 Boolean

* Number：0 和 NaN 为 false，其他均为 true
* String：空字符串为 false，其他均为 true
* Null 和 undefined：均为 false

```javascript
// number
if (0) {
    console.log("0");
}
if (NaN) {
    console.log("NaN");
}
if (-1) {
    console.log("-1");
}
// 运行结果：-1

// String
if ("") {
    console.log("空字符");
}
if (" ") {
    console.log("空格");
}
// 运行结果：空格

// Null 和 undefined
if (null) {
    console.log("null")
}
if (undefined) {
    console.log("undefined")
}
if (1) {
    console.log("null 和 undefined 都是 false")
}
// 运行结果：null 和 undefined 都是 false
```

### 流程控制

* if...else if...else
* switch
* for
* while
* do...while

Reference: <https://www.w3school.com.cn/jsref/jsref_statements.asp>

## 函数

函数是被设计为执行特定任务的代码块

函数的定义有两种形式，通常的语法为

```javascript
function functionName(var1,var2,...){
    // code
}
```

其中：

* 形式参数不需要类型
* 返回值也不需要定义类型，可以之间在函数内部 return 返回

```javascript
function add1(a, b){
    return a+b;
}

var result = add1(10, 20);
console.log(result); // 30
```

---

定义函数的方式二

```javascript
var functionName = function(var1, var2,...){
    // code
}
```

上例采用此方法

```javascript
var add2 = function(a, b){
    return a+b;
}

var result = add2(10, 20);
console.log(result); // 30
```

> JS 中，函数调用可以传递任意个数的参数，但只接收定义形参个数

## 对象

基础对象、浏览器对象模型 BOM、文档对象模型 DOM

### Array 数组

定义方法一

```javascript
var name = new Array(element1,element2,...);
// 例如
var arr = new Array(1,2,3,4);
```

定义方法二

```javascript
var name = [element];
// 例如
var arr = [1,2,3,4];
```

访问与赋值

```javascript
// 访问，下标从0开始
arr[2];
// 赋值
arr[4]=5;
```

> 数组长度可变，也可以存储任意类型的数据

```javascript
var arr = [1,2,3,4];
// console.log(arr);

// 长度可变
arr[9] = 8;
// console.log(arr);

// 类型可变
arr[8] = 'A';
console.log(arr);
```

#### 属性

length 属性可以返回数组中元素的数量，用此属性遍历数组

```javascript
var arr = [1,2,3,4];
for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}
```

#### 方法

| 方法      | 描述                                                 |
| --------- | ---------------------------------------------------- |
| forEach() | 遍历数组中的每个**有值**的元素，并调用一次传入的函数 |
| push()    | 将新元素添加到数组的末尾，并返回新的长度             |
| splice()  | 从数组中删除元素                                     |

forEach 遍历

```javascript
var arr = [1,2,3,4];
arr.forEach(function(e){
    console.log(e);
})
```

上述代码可以用箭头函数简化

```javascript
// 箭头函数：(...) => {...} 简化函数定义
arr.forEach(e => {
    console.log(e);
});
```

push 函数添加数值

```javascript
var arr = [1,2,3,4];
// 可以有多个值
arr.push(5,6,7,8);
console.log(arr);// [1,2,3,4,5,6,7,8]
```

splice 删除元素

```javascript
var arr = [1,2,3,4];// [1,2,3,4,5,6,7,8]
arr.push(5,6,7,8);
// 从第几个元素开始，删除几个元素
arr.splice(2,4); // 从第2个元素开始，删除4个元素
console.log(arr);// [1,2,7,8]
```

#### 两种遍历的区别

for 遍历会遍历所有元素，包括 undefined。而 forEach 仅遍历有值元素

```javascript
var arr = [1,2,3,4];
arr[9] = 10;

for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}// 1,2,3,4,undefined,,,,,10

console.log("==============================");

arr.forEach(e => {
    console.log(e);
})// 1,2,3,4,10
```

### String 字符串

创建方式有两种

```javascript
// 方式一
var name = new String("");
// 方式二
var name = ""; // 单双引皆可
```

#### 属性与方法

| 属性或方法  | 描述                                   |
| ----------- | -------------------------------------- |
| length      | 字符串的长度                           |
| charAt()    | 返回在指定位置的字符                   |
| indexOf()   | 检索字符串                             |
| trim()      | 去除字符串两边的空格                   |
| substring() | 提取字符串中两个指定的索引号之间的字符 |

```javascript
var str = "Hello String";

console.log(str.length); // 12

// 从0开始
console.log(str.charAt(4)); // 0

console.log(str.indexOf("lo")); // 3

var s = "    Hello String    ";
var s = s.trim();
console.log(s); // Hello String

// 开始，结束，含头不含尾
var s = s.substring(0,5);
console.log(s); // Hello
```

### JS 自定义对象

定义格式

```javascript
var 对象名 = {
    属性名: 属性值,
    函数名: function(形参){
        
    }
}
```

例如

```javascript
var person = {
    name: "tom",
    age: 18,
    gender: "male",
    eat: function(){
        console.log("恰饭nya");
    }
}

console.log(person.age);
person.eat();
```

其中方法有简写

```javascript
var person = {
    name: "tom",
    age: 18,
    gender: "male",
    // eat: function(){
    //     console.log("恰饭nya");
    // }
    eat(){
        console.log("恰饭nya");
    }
}
```

### JSON

JavaScript Object Notation，JavaScript 对象标记法，JSON 是通过 JavaScript 对象标记法书写的**文本**，由于其语法简单，层次结构鲜明，现多用于作为数据载体，在网络中进行数据传输

定义与实例

```javascript
// 定义
var 变量名 = '{"key1":value1,"key2":value2}';
// 示例
var userStr = '{"name":"Tom","age":18,"addr":["北京","上海"]}';
```

其中 value 的数据类型为：

* 数字 (整数或浮点数)
* 字符串 (在双引号中)
* 逻辑值 (true 或 false)
* 数组 (在方括号中)
* 对象 (在花括号中)
* null

---

在 JS 中有把对象转为 JSON 字符串的方法

```javascript
var jsonStr = JSON.stringify(jsObject)
```

也有把 JSON 字符串转为对象的方法

```javascript
var userStr = '{"name":"Tom","age":18,"addr":["北京","上海"]}';
var jsObject = JSON.parse(userStr)
```

### BOM

Browser Object Model，浏览器对象模型，允许 JavaScript 与浏览器对话，JavaScript 将浏览器的各个组成部分封装成对象

* Window：浏览器窗口对象
* Navigator：浏览器对象
* Screen：屏幕对象
* History：历史记录对象
* Location：地址栏对象

#### Window

浏览器窗口对象可直接使用，其中 `window.` 可以省略。属性有

| 属性      | 描述                           |
| --------- | ------------------------------ |
| history   | 对 History 对象的只读引用      |
| location  | 用于窗口或框架的 Location 对象 |
| navigator | 对 Navigator 对象的只读引用    |

方法有

| 方法          | 描述                                             |
| ------------- | ------------------------------------------------ |
| alert()       | 显示带有一段信息和一个确认按钮的警告框           |
| confirm()     | 显示带有一段信息以及确认按钮和取消按钮的对话框   |
| setInterval() | 按照指定的周期 (以毫秒计) 来调用函数或计算表达式 |
| setTimeout()  | 在指定的毫秒数后调用函数或计算表达式             |

```javascript
// 获取window对象
window.alert("获取window对象");
// 或者省略前面
alert("省略了window");

// confirm
var flag = confirm("是否确认");
console.log(flag);

// 定时器1
var i = 0;
setInterval(function(){
    i++;
    console.log("定时器执行"+i+"次");
},2000); // 每两秒执行一次

//定时器2
setTimeout(function(){
    console.log("只会执行一次");
},3000); // 三秒后执行一次
```

#### Location

地址栏对象，使用 `window.location` 获取，其中 `window.` 可以省略

属性 `href` 可以设置或返回完整的 URL

```javascript
// 获取当前地址栏地址
console.log(location.href);
// 设置地址栏地址，会自动跳转
location.href = "https://blog.yexca.net/"
```

### DOM

Document Object Model，文档对象模型，将标记语言的各个组成部分封装为对应的对象

DOM 是 W3C 的标准，定义了访问 HTML 和 XML 文档的标准，分为三部分：

1. Core DOM - 所有文档类型的标准模型

  * Document：整个文档对象
  * Element：元素对象
  * Attribute：属性对象
  * Text：文本对象
  * Comment：注释对象
2. XML DOM - XML 文档的标准模型
3. HTML DOM - HTML 文档的标准模型
  * Image: `<img>`
  * Button: `<input type='button'>`

JS 通过 DOM，就能够对 HTML 进行操作，例如：

* 改变 HTML 元素的内容
* 改变 HTML 元素的样式 (CSS)
* 对 HTML DOM 事件做出反应
* 添加和删除 HTML 元素

HTML 中的 Element 对象可以通过 Document 对象获取，而 Document 对象是通过 window 对象获取的

Document 对象中提供了以下获取 Element 元素对象的函数

1. 根据 id 属性值获取，返回单个 Element 对象

```javascript
var app = document.getElementById('app');
```

2. 根据标签名称获取，返回 Element 对象数组

```javascript
var links = document.getElementsByTagName("a");
```

3. 根据 name 属性值获取，返回 Element 对象数组

```javascript
var hobbys = document.getElementsByName('hobby');
```

4. 根据 class 属性值获取，返回 Element 对象数组
```javascript
var classes = document.getElementsByClassName('cls');
```

以上例子 HTML

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DOM</title>
    
</head>
<body>
    <div id="app">
        <a href="#">abc</a><br>
        <input type="checkbox" name="hobby">hobby1 <br>
        <input type="checkbox" name="hobby">hobby2 <br>
        <a href="#">def</a><br>
        <div class="cls">class</div>
    </div>
</body>
<script src="./js/10-DOM.js"></script>
</html>
```

---

在获取元素后，就可以修改了，修改参考 <https://www.w3school.com.cn/jsref/index.asp> 左侧 HTML 对象

例如上例修改第一个 a 标签文字

```javascript
// 获取
var links = document.getElementsByTagName("a");
// 修改
links[0].innerHTML = "修改值";
```

## 事件监听

事件是发生在 HTML 元素上的事情，比如按钮被点击、鼠标移动到元素上、按下键盘按键等

而事件监听则指 JavaScript 可以在事件被侦测到时执行代码

### 事件绑定

事件绑定有两种方式，方式一：通过 HTML 标签中的事件属性进行绑定

```html
<button id="btn" onclick="on()">按钮</button>
<script>
	function on(){
    	alert("按钮被点击1");
	}
</script>
```

方式二：通过 DOM 元素属性绑定

```javascript
document.getElementById("btn").onclick=function(){
    alert("按钮被点击2");
}
```

### 常见事件

| 事件名      | 描述                     |
| ----------- | ------------------------ |
| onclick     | 鼠标单击事件             |
| onblur      | 元素失去焦点             |
| onfocus     | 元素获得焦点             |
| onload      | 某个页面或图像被完成加载 |
| onsubmit    | 当表单提交时触发该事件   |
| onkeydown   | 某个键盘的键被按下       |
| onmouseover | 鼠标被移到某元素之上     |
| onmouseout  | 鼠标从某元素移开         |

