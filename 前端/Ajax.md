### Ajax

***

#### 1、什么是Ajax

​	Ajax是与服务器交换数据并更新部分网页的艺术，在不重新加载整个页面的情况下。

#### 2、Ajax的使用

- 创建一个异步对象

~~~html
var xmlHttp = new XMLHttpRequest()
~~~

- 设置请求方式和请求地址

| 方法                           | 描述                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| `open(*method*,*url*,*async*)` | 规定请求的类型、URL 以及是否异步处理请求。*method*：请求的类型；GET 或 POST URL：文件在服务器上的位置 `async`：true（异步）或 false（同步） |
| send(*string*)                 | 将请求发送到服务器。*string*：仅用于 POST 请求               |

~~~html
/*当使用 async=true 时，请规定在响应处于 onreadystatechange 事件中的就绪状态时执行的函数*/
xmlhttp.open("GET","test1.txt",true);
~~~

- 发送请求

~~~html
xmlHttp.send()
~~~

- 监听状态的变化

| 属性                 | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| `onreadystatechange` | 存储函数（或函数名），每当 `readyState` 属性改变时，就会调用该函数。 |
| `readyState`         | 存有 `XMLHttpRequest `的状态。从 0 到 4 发生变化。0: 请求未初始化  1: 服务器连接已建立  2: 请求已接收  3: 请求处理中  4: 请求已完成，且响应已就绪 |
| status               | 200: "OK"404: 未找到页面                                     |

~~~html
xmlHttp.onreadystatechange = function(ev){
	if(xmlHttp.readyState === 4){
		if(xmlHttp.status >= 200 && xmlHttp.status <=300 || xmlHttp.status == 304)
		/*处理返回的结果*/
			console.log("收到服务器返回的结果");
         else
            console.log("没有收到服务器返回的结果");            
	}
}
~~~

- 处理返回的结果

