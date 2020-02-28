# JSON笔记

JavaScript Object Notation（JavaScript对象表示法）

json是存储和佳欢文本信息的语法，类似XML，但比XML更小、更快、更易解析：

#### 使用 XML

- 读取 XML 文档
- 使用 XML DOM 来循环遍历文档
- 读取值并存储在变量中

#### 使用 JSON

- 读取 JSON 字符串
- 用 eval() 处理 JSON 字符串



不过json依然独立于语言和平台。

**JSON文件类型是".json", MIME类型是"application/json"**

json文本格式在语法上与创建JavaScript对象的代码相同。所以无需解析器，JavaScript程序能够使用内建的eval()函数，用json数据源来生成原生的JavaScript对象。如（数组）

```javascript
{
    "sites": [
    { "name":"菜鸟教程" , "url":"www.runoob.com" }, 
    { "name":"google" , "url":"www.google.com" }, 
    { "name":"微博" , "url":"www.weibo.com" }
    ]
}
```

实例：

```html
<p>
    名称：<span id = "jname"></span><br />
    URL：<span id = "jurl"></span><br/>
    slogan：<span id = "jslogan"></span><br />
</p>
<script>
    var JSONObj = {
        "name":"菜鸟教程",
        "url":"runoob.com",
        "slogan":"啊啊啊";
    };
    document.getElementById("jname").innerHTML = JSONObj.name
    document.getElementById("jurl").innerHTML = JSONOjb.url
    document.getElementById("jslogan").innerHTML = JSONObj.slogan
</script>
```



### JSON语法

- 数据在键值对中，key必须是字符串，value是合法的JSON数据类型（字符串、数字、对象、数组、布尔值、null）

- 数据由逗号分隔

- 大括号保存对象

- 中括号保存数组

- 可以使用"obj.name"或"obj[name]"或"obj.[i]"来获取对象中属性的值<br>不过使用".x"访问的话，x不可以是变量，而"[x]"可以是变量<br>

  ```js
  var myObj, x;
  myObj = { "name":"runoob", "alexa":10000, "site":null };
  x = "name";
  document.getElementById("demo").innerHTML = myObj.x;  // 结果是 undefined
  document.getElementById("demo").innerHTML = myObj[x];  // 结果是 runoob
  ```

- JSON对象可以嵌套<br>

  ```js
  myObj = {
      "name":"runoob",
      "alexa":10000,
      "sites": {
          "site1":"www.runoob.com",
          "site2":"m.runoob.com",
          "site3":"c.runoob.com"
      }
  }
  使用myObj.sites.site1访问嵌套对象
  ```

- 删除JSON对象中的属性使用delete<br>（注：delete并不会彻底删除元素，而是将值置为undefined，仍保留空间，所以不会影响数组长度。）

  ```js
  delete myObj.sites.site1;
  或
  delete myObj.sites["site1"]
  ```



**例：嵌套对象中的数组：**

```js
myObj = {
    "name":"网站",
    "num":3,
    "sites": [
        { "name":"Google", "info":[ "Android", "Google 搜索", "Google 翻译" ] },
        { "name":"Runoob", "info":[ "菜鸟教程", "菜鸟工具", "菜鸟微信" ] },
        { "name":"Taobao", "info":[ "淘宝", "网购" ] }
    ]
}
```

我们可以使用 for-in 来循环访问每个数组：

```js
for (i in myObj.sites) {
    x += "<h1>" + myObj.sites[i].name + "</h1>";
    for (j in myObj.sites[i].info) {
        x += myObj.sites[i].info[j] + "<br>";
    }
}
```



### JSON.parse()

由于JSON通常用于与服务器交换数据，而接收服务器数据时一般是字符串

可以使用JSON.parse()方法将数据转换为JavaScript对象

语法：

```js

```

- *text：必需，一个**有效的**JSON字符串

-   [reviver]：可选，一个转换结果的函数，将为对象的每个成员调用此函数

如：

```js

```

**实例**

```js
var xmlhttp = new XMLHttpRequest();
xmlhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
        myObj = JSON.parse(this.responseText);
        document.getElementById("demo").innerHTML = myObj.name;
    }
};
xmlhttp.open("GET", "/try/ajax/json_demo.txt", true);
xmlhttp.send();
```



### 解析数据

**Date对象处理**

JSON不能储存Date对象，如果需要存储Date对象，需要将他转换为字符串，需要的时候再将字符串转换为Date对象：

```js

```

可以使用第二个参数来完成：

```js

```



**函数处理**

JSON不允许包含函数，但可以将函数作为字符串存储，需要的时候再将字符串转换为函数：

（不建议在JSON中使用函数）

```js
var text = '{ "name":"Runoob", "alexa":"function () {return 10000;}", "site":"www.runoob.com"}';
var obj = JSON.parse(text);
//eval用来处理JSON字符串
obj.alexa = eval("(" + obj.alexa + ")");

document.getElementById("demo").innerHTML = obj.name + " Alexa 排名：" + obj.alexa(); 
```



### 关于eval()

eval(string)可计算某个字符串，并执行其中的JavaScript代码：

```js

```



**注1：**

对于服务器返回的JSON字符串，如果 jQuery 异步请求没做类型说明，或者以字符串方式接受，那么需要做一次对象化处理，方式不是太麻烦，就是将该字符串放于 eval()中执行一次。这种方式也适合以普通 javascipt 方式获取 json 对象，以下举例说明：

```js
var u = eval('('+user+')');
```

为什么要 eval 这里要添加 ('('+user+')') 呢？

原因在于：eval 本身的问题。 由于 json 是以 {} 的方式来开始以及结束的，在 js 中，它会被当成一个语句块来处理，所以必须强制性的将它转换成一种表达式。

加上圆括号的目的是迫使 eval 函数在处理 JavaScript 代码的时候强制将括号内的表达式（expression）转化为对象，而不是作为语句（statement）来执行。举一个例子，例如对象字面量 {}，如若不加外层的括号，那么 eval 会将大括号识别为 javascript 代码块的开始和结束标记，那么{}将会被认为是执行了一句空语句。所以下面两个执行结果是不同的：

```js
alert(eval("{}"); // return undefined
alert(eval("({})");// return object[Object]
```

**测试实例**

```js
var user = '{name:"张三",age:23,'+   
    'address:{city:"青岛",zip:"266071"},'+    'email:"iteacher@haiersoft.com.cn",'+  
    'showInfo:function(){'+  
    'document.write("姓名："+this.name+"<br/>");'+  
    'document.write("年龄："+this.age+"<br/>");'+  
    'document.write("地址："+this.address.city+"<br/>");'+  
    'document.write("邮编："+this.address.zip+"<br/>");'+  
    'document.write("E-mail："+this.email+"<br/>");} }';   
var u = eval('('+user+')');  
u.showInfo();
```

**注2：**

```
使用 JSON.parse 的 reviver 函数时一定要注意遍历到最后的顶层对象 key 为 ""，需要返回 value。

var json = '{"name":"Harvy", "age":36, "gender":"male"}';
var person = JSON.parse(json, function (key, value) {
    if(key != "")
        return "<font color=\"blue\">"+value+"</font>";
    else
        return value;
});
```



### JSON.stringify()

与JSON.parse()相反，stringify()方法将JavaScript对象转换为字符串

语法：

```

```

- value：必需，一个有效的JSON对象
- replacer：可选，用于转换结果的函数或数组
- - 如果是函数，则stringify将调用该函数，并传入每个成员的键和值，使用返回值而不是原始值。如果返回undefined则排除成员。根对象的键是一个空字符串""。
  - 如果是一个数组，则仅转换该数组中具有键值的成员，成员的转换顺序与数组中的顺序相同。当上一个参数：value参数也为数组时，将忽略replacer数组
- space：可选，文本添加缩进、空格和换行。如果space是一个数字，则返回值文本在每个级别缩进指定数目的空格（最大为10），可以使用非数字如\t。

**实例：**

对象转换：

```js

```

数组转换：

```js
var arr = [ "Google", "Runoob", "Taobao", "Facebook" ];
var myJSON = JSON.stringify(arr);
document.getElementById("demo").innerHTML = myJSON;
```



**Date对象处理**

stringify会将所有的日期转换为字符串

```js
var obj = { "name":"Runoob", "initDate":new Date(), "site":"www.runoob.com"};
var myJSON = JSON.stringify(obj);
document.getElementById("demo").innerHTML = myJSON;
```

**函数处理**

stringify会删除JavaScript对象的函数，包括key和value

```js
var obj = { "name":"Runoob", "alexa":function () {return 10000;}, "site":"www.runoob.com"};
var myJSON = JSON.stringify(obj);
document.getElementById("demo").innerHTML = myJSON;
```

结果：

```js

```



可以将函数转换成字符串来避免被删除：

（不推荐在JSON中使用函数）

```js
var obj = { "name":"Runoob", "alexa":function () {return 10000;}, "site":"www.runoob.com"};
obj.alexa = obj.alexa.toString();
var myJSON = JSON.stringify(obj);
 
document.getElementById("demo").innerHTML = myJSON;
```

结果：

```js
{"name":"Runoob","alexa":"function () {return 10000;}","site":"www.runoob.com"}
```





### JSONP

动态生成JS脚本