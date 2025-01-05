---
slug: 135
# layout: post
title: "Ajax 与 Axios"
author: yexca
date: 2023-12-21T13:32:15+08:00
# permalink: /archives/135
categories:
    - 前端
    - JavaScript
tags:
    - Ajax
    - Axios
---

Asynchronous JavaScript And XML，异步的 JS 和 XML。作用：

* 数据交换：通过 Ajax 可以给服务器发送请求，并获取服务器响应的数据
* 异步交互：可以在不重新加载整个页面的情况下，与服务器交换数据并更新部分网页的技术

使用场景：搜索联想、用户名是否可用等

## 同步与异步

同步指在访问网页时进行某操作需要请求服务器，在服务器处理时网页不可操作，直到服务器响应客户端时才可继续操作

而异步在请求服务器的同时，客户端可以执行其他操作

## 原生 Ajax

首先创建 XMLHttpRequest 对象 (用于和服务器交换数据)，然后向服务器发送请求，最后获取服务器响应数据

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ajax</title>
</head>
<body>
    <input type="button" value="获取数据" onclick="getDate()">
    <div id="div1"></div>
</body>
<script>
    function getDate(){
        // 创建XMLHttpRequest
        var xmlHttpRequest = new XMLHttpRequest();

        // 发送异步请求
        xmlHttpRequest.open('GET', '//127.0.0.1:8080/listEmp');
        xmlHttpRequest.send();

        // 获取服务响应数据
        xmlHttpRequest.onreadystatechange=function(){
            if(xmlHttpRequest.readyState==4 && xmlHttpRequest.status==200){
                document.getElementById("div1").innerHTML=xmlHttpRequest.responseText;
            }
        }
    }
</script>
</html>
```

> 详细参考：<https://www.w3school.com.cn/js/js_ajax_intro.asp>

## Axios

Axios 对原生的 Ajax 进行了封装，简化书写，快速开发。官网：<https://www.axios-http.cn/>

使用 Axios 需要先引入 Axios 文件

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

使用 Axios 发送请求并响应结果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Axios</title>
    <!-- 文件下载到了本地 -->
    <script src="./js/axios-0.18.0.js"></script>
</head>
<body>
    <input type="button" value="获取数据" onclick="getData()">
</body>
<script>
    function getData(){
        axios({
            method: "get",
            url: "//127.0.0.1:8080/listEmp"
        }).then(result => {
            console.log(result.data);
        })
    }
</script>
</html>
```

上述是 GET 方法，POST 方法需要添加数据

```javascript
function deleteData(){
    axios({
        method: "POST",
        url: "//127.0.0.1:8080/deleteEmpById",
        data: "id=1"
    }).then(result => {
        console.log(result.data);
    })
}
```

---

当然，这样写还是过于繁琐，Axios 提供了别名

* axios.get(url[, config])
* axios.delete(url[, config])
* axios.post(url[, data[, config]])
* axios.put(url[, data[, config]])

```javascript
function getData(){
    // axios({
    //     method: "get",
    //     url: "//127.0.0.1:8080/listEmp"
    // }).then(result => {
    //     console.log(result.data);
    // })

    axios.get("//127.0.0.1:8080/listEmp").then(result => {
        console.log(result.data);
    })
}

function deleteData(){
    // axios({
    //     method: "POST",
    //     url: "//127.0.0.1:8080/deleteEmpById",
    //     data: "id=1"
    // }).then(result => {
    //     console.log(result.data);
    // })
    
    axios.post("//127.0.0.1:8080/deleteEmpById","id=1").then(result => {
        console.log(result.data);
    })
}
```

