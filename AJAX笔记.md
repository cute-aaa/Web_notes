# AJAX笔记

AJAX = Asynchronous JavaScript and XML（异步的JavaScript和XML）

AJAX不是新的编程语言，而是一种使用现有标准的新方法。

AJAX最大的优点是通过在后台与服务器进行少量数据交换，在不重新加载整个页面的情况下，可以于服务器交换数据并更新部分网页内容

AJAX不需要任何浏览器插件，但需要用户允许JavaScript再浏览器上执行，因此AJAX应用程序与浏览器和平台无关。



**核心**

```javascript
var xmlhttp;
	if (window.XMLHttpRequest){
		//  IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
		xmlhttp=new XMLHttpRequest();
	}else{
		// IE6, IE5 浏览器执行代码
		xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
	}
	xmlhttp.onreadystatechange=function(){
		if (xmlhttp.readyState==4 && xmlhttp.status==200){
			document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
		}
	}
	xmlhttp.open("GET","/try/ajax/ajax_info.txt",true);
	xmlhttp.send();
```





## AJAX 应用

- 运用XHTML+CSS来表达资讯；
- 运用JavaScript操作DOM（Document Object Model）来执行动态效果；
- 运用XML和XSLT操作资料;
- 运用XMLHttpRequest或新的Fetch API与网页服务器进行异步资料交换；
- 注意：AJAX与Flash、Silverlight和Java Applet等RIA技术是有区分的。

搜索建议使它流行了起来



工作原理：

![AJAX](http://www.runoob.com/images/ajax.gif)

可以



### 正文

AJAX的基础是XMLHttpRequest对象。所有现代浏览器均支持XMLHttpRequest对象（IE5、6使用ActiveXObject）。

XMLHttpRequest对象用于在后台与服务器交换数据。以此为基础在不重新加载整个网页的情况下对网页的某部分进行更新

**创建XMLHttpRequest对象**

```javascript
var xmlhttp
if(window.XMLHttpRequest){
    //IE7+, firefox, chrome, opera, safari
    xmlhttp = new XMLHttpRequest();
}else {
    //IE5,6
    xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
```

**向服务器发送请求**

```javascript
//请求类型， URL， 是否异步
xmlhttp.open("GET", "/try/ajax/ajax_info.txt", true);
xmlhttp.send();
```

| 方法                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| open(*method*,*url*,*async*) | 规定请求的类型、URL 以及是否异步处理请求。*method*：请求的类型；GET 或 POST*url*：文件在服务器上的位置*async*：true（异步）或 false（同步） |
| send(*string*)               | 将请求发送到服务器。*string*：仅用于 POST 请求               |

| 方法                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| open(*method*,*url*,*async*) | 规定请求的类型、URL 以及是否异步处理请求。*method*：请求的类型；GET 或 POST*url*：文件在服务器上的位置*async*：true（异步）或 false（同步） |
| send(*string*)               | 将请求发送到服务器。*string*：仅用于 POST 请求               |

open方法的URL时服务器上文件的地址，可以是任何类型的文件或脚本，比如.php（在传回响应之前能够在服务器上执行任务）

**异步true还是false**
AJAX指的时异步JavaScript和XML（Asynchronous JavaScript and XML）
XMLHttpRequest对象如果要用于AJAX的话，open方法的async参数必须设置为true
异步请求是一个巨大的进步。很多在服务器执行的任务都相当费时，这可能会引起应用挂起或停止。
通过AJAX，JavaScript无需等待服务器的响应，而是：

- 在等待服务器响应时执行其他脚本
- 当响应就绪后对响应进行处理

使用异步时，请规定响应处于onreadystatechange事件中的就绪状态时执行的函数：
```javascript
xmlhttp.onreadystatechange = function(){
    if(xmlhttp.readyState == 4 && xmlhttp.status == 200){
        document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
    }
}
xmlhttp.open(...);
xmlhttp.send();
```

不推荐异步=false，但对于一些小型的请求也是可以的，js会等到服务器响应就绪才继续执行。如果服务器繁忙或缓慢，应用程序可能会挂起或停止。
当异步=false时，不要编写onreadystatechange函数，并且把要执行的代码放到send语句后面
```javascript
xmlhttp.open(...);
xmlhttp.send();
document.getElementById("myDiv".innerHTML = xmlhttp.responseText);
```

**GET还是POST？**

GET更简单快速，并且大部分情况下都能用，但以下情况请使用post：

- 无法使用缓存文件时（更新服务器上的文件或数据库）
- 向服务器发送大量数据（post没有数据量限制）
- 发送包含位置字符的用户输入时，post更稳定可靠

一个简单的get请求：
```javascript
xmlhttp.open("GET", "/try/ajax/get.php", true);
xmlhttp.send();
```
上面得到的可能是缓存结果，可以向URL添加一个唯一的ID来避免这种情况
```javascript
xmlhttp.open("GET", "/try/ajax/get.php?t=" + Math.random(), true);
xmlhttp.send();
```
还可以通过GET方法发送消息：
```javascript
xmlhttp.open("GET", "/try/ajax/get.php?name=sure", true);
xmlhttp.send();
```
**post**
最基本的post请求与get一样

```javascript
xmlhttp.open("post", "try/ajax/post.php", true);
```
但如果要向HTML表单那样post数据，需要使用setRequest()来添加HTTP头，然后在send方法中规定希望发送的数据。
```javascript
xmlhttp.open("post", "try/ajax/post.php", true);
//向请求中添加HTTP头(header, value)：规定头的名称、规定头的值
xmlhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xmlhttp.send("name=sure");
```



### 接收服务器响应

使用XMLHttpRequest对象的responseText（获取字符串形式的数据）或responseXML（获取XML格式的响应数据）属性

**responseText**

如果来自服务器的响应并非XML就使用responseText

```javascript
document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
```

**responseXML**

如果来自服务器的响应是XM了，且需要作为XML对象进行解析，请使用responseXML属性

不过格式全靠string控制

```javascript
var txt, x, i;
//每次readyState改变时就会触发onreadystatechange事件
xmlhttp.onreadystatechange = function(){
    //如果服务端响应完成且正确
    if(xmlhttp.readyState == 4 && xmlhttp.status == 200){
        xmlDoc = xmlhttp.responseXML;
        txt = "";
        //获取所有artist标签组成的数组
        x = xmlDoc.getElementSByTagName("ARTIST");
        for(i = 0; i < x.length; i++){
            txt = txt + x[i].childNodes[0].nodeValue + "<br>";
        }
        document.getElementById("myDiv").innerHTML = txt;
    }
}
xmlhttp.open("GET", "cd.xml", true);
xmlhttp.send();
```

readyState存有XMLHttpRequest的状态信息

XMLHttpRequest对象的重要属性：

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| onreadystatechange | 存储函数（或函数名），每当 readyState 属性改变时，就会调用该函数。所以这个函数会被触发4次（0-4） |
| readyState         | 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。0: 请求未初始化1: 服务器连接已建立2: 请求已接收3: 请求处理中4: 请求已完成，且响应已就绪 |
| status             | 200: "OK" 404: 未找到页面                                    |

因此一般在onreadystatechange事件中规定当服务器响应以做好被处理的准备时所执行的任务

**附：状态的详细变化**

创建一个XMLHttpRequest对象后，xmlHttp.readyState的值是0

然后xmlHttp.onreadystatechange = handlestatechange没有运行。

紧接着是open()，这个函数发生之后xmlHttp.ready.State的值变为1

紧接着又返回到xmlHttp.onreadystatechange = handlestatechange运行

再执行send()，send发生之后xmlHttp.readyState的值变为2

回到xmlHttp.onreadystatechange = handlestatechange运行，以此类推



**回调函数**

回调函数是一种以参数形式传递给另一个函数的函数

如果网站上存在多个AJAX任务，那么应该为创建XMLHttpRequest对象编写一个标准的函数，并未每个AJAX任务调用该函数，如：

```javascript
<script>
    var xmlhttp;
    function myFunction() {
        `loadXMLDoc("/try/ajax/info.txt"k, function(){
         if(xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
         }
         })`
    }
    function loadXMLDoc(url, cfunc) {
        if(window.XMLHttpRequest) {
            xmlhttp = new XMLHttpRequest();
        }else{
            xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
        }
        xmlhttp.onreadystatechange = cfunc;
        xmlhttp.open("GET", url, true);
        xmlhttp.send();
    }
<script>
```

在html中定义div后使用动作标签如button，设定事件为myFunction()就可以使用了



### 动态网页实例

动态页面多类于此，执行的方法都交给了控制器

```javascript
function showHint(str){
    xmlhttp.onreadystatechange = function(){
    if(... 4 && ... 200){
            document.getElementById("txtHint").innerHTML = xmlhttp.responseText;
        }
    }
    //用这里的URL来指定发送路径从而获得接收的信息吗？
    xmlhttp.open("GET", "/try/ajax/gethint.php?name=" + str, true);
    xmlhttp.send();
}
```

```html
<form action = "">
    输入姓名：
    //传参！！！！！！！！！！！！！！！！！！
    <input type = "text" id = "txt1" onkeyup = "showHint(this.value)" />
    <p>
        提示信息：<span id = "txtHint"></span>
    </p>
</form>
```











服务器常用的状态码及其对应的含义如下：

-  200：服务器响应正常。
-  304：该资源在上次请求之后没有任何修改（这通常用于浏览器的缓存机制，使用GET请求时尤其需要注意）。
-  400：无法找到请求的资源。
-  401：访问资源的权限不够。
-  403：没有权限访问资源。
-  404：需要访问的资源不存在。
-  405：需要访问的资源被禁止。
-  407：访问的资源需要代理身份验证。
-  414：请求的URL太长。
-  500：服务器内部错误。





简单的AJAX（差不多是全部了）

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script>
function loadXMLDoc()
{
	var xmlhttp;
	if (window.XMLHttpRequest)
	{
		//  IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
		xmlhttp=new XMLHttpRequest();
	}
	else
	{
		// IE6, IE5 浏览器执行代码
		xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
	}
	xmlhttp.onreadystatechange=function()
	{
		if (xmlhttp.readyState==4 && xmlhttp.status==200)
		{
			document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
		}
	}
	xmlhttp.open("GET","/try/ajax/ajax_info.txt",true);
	xmlhttp.send();
}
</script>
</head>
<body>

<div id="myDiv"><h2>使用 AJAX 修改该文本内容</h2></div>
<button type="button" onclick="loadXMLDoc()">修改内容</button>

</body>
</html>
```

较复杂的AJAX（复杂的也不是它）

```html
<html><!DOCTYPE html>
<html>
<head>
<script>
function loadXMLDoc(url)
{
var xmlhttp;
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
xmlhttp.onreadystatechange=function()
  {
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
    document.getElementById('p1').innerHTML="Last modified: " + xmlhttp.getResponseHeader('Last-Modified');
    }
  }
xmlhttp.open("GET",url,true);
xmlhttp.send();
}
</script>
</head>
<body>

<p id="p1">The getResponseHeader() function is used to return specific header information from a resource, like length, server-type, content-type, last-modified, etc.</p>
<button onclick="loadXMLDoc('/try/ajax/ajax_info.txt')">Get "Last-Modified" information</button>

</body>
</html>

```





