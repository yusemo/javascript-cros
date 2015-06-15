# javascript-cros
跨域的一些问题和方法总结


一直都说想找个大总结的文章关于跨域的处理奈何度娘上的资料太老旧，自己对于新的知识点又不是十分十分有研究。因此一直让懒癌发作下去。奈何最近做了双失青年，因此有了空余的时间去研究研究知识点和梳理一下，顺便自己动手风衣足食。废话不多说请看下文。

为什么要跨域？

##何谓同源:

URL由协议、域名、端口和路径组成，如果两个URL的协议、域名和端口相同，则表示他们同源。简单哪来说就是域名，协议，端口都相同。

同源策略(Same origin policy):
浏览器的同源策略，限制了来自不同源的"document"或脚本，对当前"document"读取或设置某些属性。 （白帽子讲web安全[1]）
从一个域上加载的脚本不允许访问另外一个域的文档属性。
因为有了这样的安全策略我们偶而需要数据交互的时候就会导致非常的不便。因而为了绕过这个安全机制策略而且产生了跨域。

 ##什么叫跨域，跨域定义？

简单的来说，出于安全方面的考虑，页面中的JavaScript无法访问其他服务器上的数据，即“同源策略”。而跨域就是通过某些手段来绕过同源策略限制，实现不同域之间通信的效果。

定义：只要协议、域名、端口有任何一个不同，都被当作是不同的域。

如图：

URL

说明

是否允许通信

http://www.a.com/a.js

http://www.a.com/b.js

同一域名下

允许

http://www.a.com/lab/a.js

http://www.a.com/script/b.js

同一域名下不同文件夹

允许

http://www.a.com:8000/a.js

http://www.a.com/b.js

同一域名，不同端口

不允许

http://www.a.com/a.js

https://www.a.com/b.js

同一域名，不同协议

不允许

http://www.a.com/a.js

http://70.32.92.74/b.js

域名和域名对应ip

不允许

http://www.a.com/a.js

http://script.a.com/b.js

主域相同，子域不同

不允许

http://www.a.com/a.js

http://a.com/b.js

同一域名，不同二级域名（同上）

不允许（cookie这种情况下也不允许访问）

http://www.cnblogs.com/a.js

http://www.a.com/b.js

不同域名

不允许

 

##1、document.domain+iframe的设置

对于主域相同而子域不同的可以采用此类办法。具体如下：

即域a页面http://a.com/aa.html 的页面上写一个隐藏的iframe
```
<html>
<head></head>
<body>
<script type="text/javascript" ><!--
Document.domain="a.com";
Var remoteHtml=document.getElementById("domainId");
remoteHtml.src="b.com/bb.html";//这里访问域b的链接
var document=remoteHtml.ContentDocument; //这里就可以使用document来操作域B中页面bb.html的数据了

// --></script>

<iframe id="domainId" src="" style="diapay:none" style="diapay:none"/>
</body>
</html>
```
注意：这里http://b.com/bb.html 页面也需要设置document.domain="a.com"， 这种方法才能奏效。

之所以这种iframe的方法不适合不同父域之间的跨域，是因为设置document.domain只能设置为自己的父域，而不是能设置为其他域，例如:a.com只能设置document.domain="a.com"，而不能是document.domain="b.com"；ps：所谓的父域（主域）是不带www，例如www.a.com其实是二级域名。 domain只能设置为主域名，不可以在b.a.com中将domain设置为c.a.com。

优点：隐藏iframe方式也很简单，它可以处理任何返回的数据格式，适用于{www.a.com, a.com, script.a.com, css.a.com}中的任何页面相互通信。

缺点：但它只适用在具有同一个父域下的跨域请求上，并且要求其他域得配合开发，即需要设置document.domain,安全性能会差些。

 

##2、location.hash＋iframe设置

在url： http://a.com#helloword中的‘#helloworld’就是location.hash，改变hash并不会导致页面刷新，所以可以利用hash值来进行数据传递，当然数据容量是有限的。

www.baidu.com#abc  就是location.hash

假如a.com要使用b.com中的值。

1)首先a.com创建iframe，并指向 b.com#parms

a.com用js定时检测iframe中src中的location.hash是否改变了，如果改变则获取其值则获取新location.hash值。

IE，chrome浏览器不支持跨域修改parent.location.hash值，所以需要代理。但是firefox支持修改。

a.com中的代码
```
<script>

function startRequest(){
var ifr = document.createElement('iframe');
ifr.style.display = 'none';
ifr.src = 'http://www.cnblogs.com/lab/cscript/cs2.html#paramdo';   //根据paramdo来决定获取啥值
document.body.appendChild(ifr);
}

//每2秒检测一次，是否变动了值

function checkHash() {

try {
var data = location.hash ? location.hash.substring(1) : '';
if (console.log) //console.log javascript中调试输出信息，比alert好

{

console.log('Now the data is '+data);

}

} catch(e) {};

}

setInterval(checkHash, 2000);

</script>
```
b.com页面的代码如下

//模拟一个简单的参数处理操作
```
switch(location.hash){

    case '#paramdo':

        callBack();

        break;

    case '#paramset':

        //do something……

        break;

}

//设置a.com iframe src的location.hash

function callBack(){

    try {

//此时是火狐正常执行

        parent.location.hash = 'somedata';

    } catch (e) {

        // ie、chrome的安全机制无法修改parent.location.hash，

        // 所以要利用一个中间的cnblogs域下的代理iframe

        var ifrproxy = document.createElement('iframe');

        ifrproxy.style.display = 'none';

        ifrproxy.src = 'http://a.com/test/cscript/cs3.html#somedata';    // 注意该文件在"a.com"域下

        document.body.appendChild(ifrproxy);

    }

}
```
a.com域名下 cs3.html中的代码

//因为parent.parent和自身属于同一个域，所以可以改变其location.hash的值

```parent.parent.location.hash = self.location.hash.substring(1);```

优点：可以解决不同主域名下的数据通信。

缺点：子窗口无法改变无窗口的hash值，除非是同一域名下的，而且hash有长度限制，数据暴露，数据类型有限制

 

##3、跨域资源共享（CORS）

CORS（Cross-Origin Resource Sharing）跨域资源共享，定义了必须在访问跨域资源时，浏览器与服务器应该如何沟通。CORS背后的基本思想就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功还是失败。

支持的浏览器版本：

因此对于数据量大的移动端可以考虑利用cors ，post数据到服务器。当然对于小量数据就没必要折腾post了，get即可。

对于服务器：设置“Access-Control-Allow-Origin：a.com”即可请求来自a域的数据。

对于客户端：
```
<script type="text/javascript">

    var xhr = new XMLHttpRequest();

    xhr.open("GET", "http://a.com/a/",true);

    xhr.send();

</script>
```
ajax访问的时候需要用到绝对路径，也就是你要跨域访问的接口地址。

优点：cors在移动终端支持的不错，可以考虑在移动端全面尝试；对于大数据量可以选择cors，对比起jsonp的get形式再大数据量是利用cors是不二的选择。配合新的JSAPI(fileapi、xhr2等)一起使用，实现强大的新体验功能。

缺点：兼容无法做到pc上全兼容和完美支持。ie8+ 利用cors需要利用XDR(XDomainRequest)类型。
```
<script type="text/javascript">

    var xdr = new XDomainRequest();

    xdr.open("GET", "http://a.com/a/",true);

    xdr.send();

</script>
```
##4、script标签

判断script节点加载完毕：ie只能通过script的readystatechange属性，其它浏览器是script的load事件。

以下是部分判断script加载完毕的方法。
```
js.onload = js.onreadystatechange = function() {

    if (!this.readyState || this.readyState === 'loaded' || this.readyState === 'complete') {

        // callback在此处执行

        js.onload = js.onreadystatechange = null;

    }

};
```
##5、jsonp

JSONP(JSON with Padding)是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题。由于同源策略，一般来说位于 server1.example.com 的网页无法与不是 server1.example.com的服务器沟通，而 HTML 的<script> 元素是一个例外。利用 <script> 元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。

JSONP由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数，而数据就是传入回调函数中的JSON数据。

JSONP也叫填充式JSON，是应用JSON的一种新方法，只不过是被包含在函数调用中的JSON，例如：

callback({"name","trigkit4"});

JSONP由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数，而数据就是传入回调函数中的JSON数据。

在js中，我们直接用XMLHttpRequest请求不同域上的数据时，是不可以的。但是，在页面上引入不同域上的js脚本文件却是可以的，jsonp正是利用这个特性来实现的。 例如：

<script type="text/javascript">

    function dosomething(jsondata){

        //处理获得的json数据

    }
</script>

<script src="http://example.com/data.php?callback=dosomething"></script>

js文件载入成功后会执行我们在url参数中指定的函数，并且会把我们需要的json数据作为参数传入。所以jsonp是需要服务器端的页面进行相应的配合的。
```
<?php

$callback = $_GET['callback'];//得到回调函数名

$data = array('a','b','c');//要返回的数据

echo $callback.'('.json_encode($data).')';//输出

?>
```
最终，输出结果为：dosomething(['a','b','c']);

如果你的页面使用jquery，那么通过它封装的方法就能很方便的来进行jsonp操作了。
```
<script type="text/javascript">

    $.getJSON('http://example.com/data.php?callback=?,function(jsondata)'){

        //处理获得的json数据

    });

</script>
```
jquery会自动生成一个全局函数来替换callback=?中的问号，之后获取到数据后又会自动销毁，实际上就是起一个临时代理函数的作用。$.getJSON方法会自动判断是否跨域，不跨域的话，就调用普通的ajax方法；跨域的话，则会以异步加载js文件的形式来调用jsonp的回调函数。

JSONP的优点是：它不像XMLHttpRequest对象实现的Ajax请求那样受到同源策略的限制；它的兼容性更好，在更加古老的浏览器中都可以运行，不需要XMLHttpRequest或ActiveX的支持；并且在请求完毕后可以通过调用callback的方式回传结果。

JSONP的缺点则是：它只支持GET请求而不支持POST等其它类型的HTTP请求；它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行JavaScript调用的问题。


##6、window.name实现的跨域数据传输

对那个实现小改了一下，加了浏览器本地设置 Window.name 的功能，实现浏览器“本地”异域数据的传递。

下面贴出代码。。。

（注意：本地需要创建一个名为 proxy.html 的空文件） 

Javascript代码
```
(function() {  
  
    var _isIE = (  
        navigator.appName == "Microsoft Internet Explorer"  
    );  
  
    var _removeNode = _isIE ? function() {  
        var d;  
        return function(n) {  
            if(n && n.tagName != 'BODY') {  
                d = d || document.createElement('div');  
                d.appendChild(n);  
                d.innerHTML = '';  
            }  
        }  
    }() : function(n) {  
        if(n && n.parentNode && n.tagName != 'BODY') {  
            n.parentNode.removeChild(n);  
        }  
    };  
  
  
/* [ Request by window.name ] 
 * **************************************************************************** 
   借助 Window.name 实现 Js 的跨域访问。 
   1、 url 向外传值， callback 处理返回结果。 
   2、 返回页面中 JS 对 window.name 赋值。 
 
   返回页 
   <script language="JavaScript"> 
       window.name = ...  // 支持 JSON 字符串，可达~2MB 
   </script> 
 
   若需同时进行多个请求，回调函数应是不同的函数实例。 
   iframe 的自由载入形成了异步机制。 
*/  
  
    wnRequest = {  
        _doc: document,  
        _proxyUrl: 'proxy.html'  
    };  
  
    wnRequest.send = function( url, callback )  
    {  
        if(! url || typeof url !== 'string') {  
            return;  
        }  
        url += (url.indexOf('?') > 0 ? '&' : '?') + 'windowname=get';  
  
        var frame = this._doc.createElement('iframe');  
        frame._state = 0;  
        this._doc.body.appendChild(frame);  
        frame.style.display = 'none';  
  
        (function( el, type, fn ) {  
            if (_isIE) {  
                el.attachEvent('on' + type, fn);  
            } else {  
                el.addEventListener(type, fn, false);  
            }  
        })(frame, 'load', function() {  
            if(frame._state === 1) {  
                _getData(frame, callback);  
            } else if(frame._state === 0) {  
                frame._state = 1;  
                frame.contentWindow.location = wnRequest._proxyUrl;  
            }  
        });  
  
        frame.src = url;  
    };  
  
    //  
    // 设置异域 Js 可访问的本地数据，客户端直接站间转递数据  
    // 注：  
    // 即浏览器直接将数据转递给另一个域的窗口，数据不上网。  
    // 返回页代码：  
    // <script type="text/javascript">  
    //     if (window.name) {  
    //         //... 处理 name 值  
    //         window.name = null;  
    //     }  
    //     // 升为顶级窗口，完成数据转递  
    //     try {  
    //         top.location.hostname;  
    //         if (top.location.hostname != window.location.hostname) {  
    //             top.location.href =window.location.href;  
    //         }  
    //     } catch(e) {  
    //         top.location.href = window.location.href;  
    //     }  
    // </script>  
    //  
    //  
    wnRequest.setname = function( name, url ) {  
        if(! url || typeof url !== 'string') {  
            return;  
        }  
        url += (url.indexOf('?') > 0 ? '&' : '?') + 'windowname=loc';  
  
        var frame = this._doc.createElement('iframe');  
        frame._count = 0;  
        this._doc.body.appendChild(frame);  
        frame.style.display = 'none';  
        if (_isIE) {  
            frame.name = name;  
        } else {  
            frame.contentWindow.name = name;  
        }  
        frame.src = url;  
    };  
  
    //  
    // 私用辅助  
    //  
    var _clear = function(frame) {  
        try {  
            frame.contentWindow.document.write('');  
            frame.contentWindow.close();  
            _removeNode(frame);  
        } catch(e) {}  
    }  
  
    var _getData = function(frame, callback) {  
        try {  
            var da = frame.contentWindow.name;  
        } catch(e) {}  
        _clear(frame);  
        if(callback && typeof callback === 'function') {  
            callback(da);  
        }  
    }  
  
})(); 
```
使用： 

如果需要同时访问多个异域文件，可以像下面这样写回调函数，浏览器异步载入 iframe 的机制形成了天生的 JS 跨域异步访问。 

这是跨域请求的主页面 JS 调用： 

Javascript代码
```
<script language="javascript">  
    var _str = '', _cnt = 0;  
  
    function myfunc( id ) {  
        return  function( data ) {  
            _str += id + ':' + data + '\n';  
            ++_cnt;  
            if (_cnt >= 4)  alert(_str);  
        };  
    }  
  
    var _links = [  
        { id: 4, url: 'http://www.aaa.com/test4.html' },  
        { id: 5, url: 'http://www.bbb.com/test5.html' },  
        { id: 6, url: 'http://www.ccc.com/test6.html' },  
        { id: 7, url: 'http://www.ddd.com/test7.html' }  
    ];  
    function dosome() {  
        for (var _i=0; _i<_links.length; ++_i) {  
            wnRequest.send(_links[_i].url, myfunc(_links[_i].id));  
        }  
        // 跨域本地数据转递  
        wnRequest.setname('这里可能是一串加密用的密钥哦，俺从 https 那边过来滴！', 'http://www.eee.com/test8.html');  
    }  
</script>  
http://www.aaa.com/test4.html 中的内容：（跨域网络数据传递） 
```
Javascript代码
```
<script type="text/javascript">  
    window.name='返回的数据，可以是 JSON 格式';  
</script>  
http://www.eee.com/test8.html 中的内容：（跨域本地数据转递应用。注意：这里是普通的 http 协议） 
```
Javascript代码
```
<script type="text/javascript">  
    if (window.name) {  
        alert(window.name);  
        // 存储或处理 name 值  
        // 可存在 Cookie 中，如果不希望 Cookie 上传泄露出去，可设置其 secure 属性  
        window.name = null;  
    }  
    /* 
    try { 
        top.location.hostname; 
        if (top.location.hostname != window.location.hostname) { 
            top.location.href =window.location.href; 
        } 
    } catch(e) { 
        top.location.href = window.location.href; 
    } 
    */  
</script>  
```
window.name的优点：name 值在不同的页面（甚至不同域名）加载后依旧存在，并且可以支持非常长的 name 值（2MB）

window.name的缺点：

7、flash

查看以下方式##flash跨域策略文件crossdomain.xml配置详解(http://www.2cto.com/Article/201108/100008.html)和YUI3的IO组件中看到的办法，具体可见http://developer.yahoo.com/yui/3/io/。

8、HTML5的window.postMessage方法跨域

查看http://www.cnblogs.com/dolphinX/p/3464056.html和http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html#m6 里面也有详细的解决方法就不一一复制过来了。

9、图像Ping

大部分的跨域技术都是结合xhr对象，有木有一种方法是可以不依赖xhr发送某种请求呢？答案是：有的。

在dom中img标签常常可以利用src发送请求就可以加载图像。那么我们可以利用动态创建图像即可发送请求。而动态创建图像经常用于图像 Ping.图像Ping是与服务器进行简单、单向的跨域通信的一种方式。请求的数据是通过查询字符串形式发送的，而响应可以是任何内容，但通常是像素图或204响应。通过图像Ping，浏览器得不到任何具体数据，但通过监听load和error事件，它能知道响应是什么时候收到的。具体请看：
```
var img = new Image();
img.onload = img.onerror = function(){
   alert('complte');   
};

img.src = 'http://a.com/test?name=a';
```
这里创建了一个Image的实例，然后把onload和onerror事件处理程序指定为同一函数。这里无论是什么响应，只要请求完成，就能得到通知。请求从设置src属性那一刻开始，而这个例子在请求发送了一个name参数。

图像Ping优点：就像前面所说的可以不依赖xhr对象，不需要修改服务器代码，简单发送请求即可。常用于跟踪用户点击或者动态广告曝光次数。

图像Ping缺点：一是只可以发送get请求，二是无法访问到服务器的响应文本。因此，图像Ping只能用于浏览器与服务器间的单向通信。

10、web sockets

web sockets是一种浏览器的API，它的目标是在一个单独的持久连接上提供全双工、双向通信。(同源策略对web sockets不适用)

web sockets原理：在JS创建了web socket之后，会有一个HTTP请求发送到浏览器以发起连接。取得服务器响应后，建立的连接会使用HTTP升级从HTTP协议交换为web sockt协议。
```
var socket = new WebSockt('ws://www.baidu.com');//http->ws; https->wss
socket.send('hello WebSockt');
socket.onmessage = function(event){
    var data = event.data;
}
```
web sockets优点：定义里面已经说明。

web sockets缺点：只有在支持web socket协议的服务器上才能正常工作。

 11、canvas跨域

最近html5大行其道，很多人也利用canvas做各种效果动画因此碰上如何为其跨域的问题。

比如：图片和网站放在不同的域下，想把图片存放到localStorage中，想到的办法是先用canvas的toDataUrl取到图片的数据，再把数据存放到localStorage中，但是toDataUrl不能跨域，所以一调用这个方法的时候就报错，还有什么好的方法取得不同域下的图片数据，或者有什么更好的办法把图片存到localStorage中？

解决办法：在网站的服务器上做代理，给网站服务器发请求，让网站服务器请求图片给前端。

还有例如：html5的getImageData方法来实现刮刮卡的效果，后台上传图片，手机端用手刮。提示"Uncaught SecurityError: Failed to execute 'getImageData' on 'CanvasRenderingContext2D': The canvas has been tainted by cross-origin data.”错误。这是因为getImageData此方法不允许操作非此域名外的图片资源，即使是子域也不行。

解决办法：

利用php把图片编码嵌入到html
```
<?php  
$pic = 'http://avatar.csdn.net/7/5/0/1_molaifeng.jpg';  
$arr = getimagesize($pic);  
$pic = "data:{$arr['mime']};base64," . base64_encode(file_get_contents($pic));  
?>  
<img src="<?php echo $pic ?>" /> 
```
很多时候说canvas使用image不能跨域其实是不对的，利用canvas 使用了没有权限的跨域图片在使用canvas.toDataURL()等数据导出函数的时候会报错！

因此可以```img.crossOrigin = "Anonymous"``` 做了巨大的贡献，它开启了本地的跨域允许。当然服务器存储那边也要开放相应的权限才行，如果是设置了防盗链的图片在服务端就没有相应的权限的话你本地端开启了权限也是没有用的。因此如果遇到Uncaught SecurityError: Failed to execute 'toDataURL' on 'HTMLCanvasElement': Tainted canvases may not be exported. 这样的报错可以加上 img.crossOrigin = “*"。

详细参考资料：

http://segmentfault.com/a/1190000000718840

 http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html


