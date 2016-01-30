# fiddler 使用

## 前言 ##
Fiddler 是一个 http 协议调试代理工具，它能够记录并检查你的电脑和互联网之间的 http 通讯，设置断点，查看所有的"进出" Fiddler 的数据（指 cookie，html，js，css 等文件，这些都可以让你胡乱修改的意思）。

**本文将介绍 Fiddler 几个使用场景。**

## 基础面板 ##

如下图，开启 Fiddler 后，需要从面板里的“打开代理浏览器”才可以捕捉浏览器的 http 请求。

![basic-panel][1]

## 抓包分析 ##
在右边面板里，可以看到 http 请求报文和 http 响应报文，和浏览器中的调试器可以看到的数据差不多，但是可以配合 http 断点，对请求报文和响应报文进行修改，或 AutoResponder 功能，进行 http 请求代理。

![data-package][2]

## AutoResponder 请求代理 ##
通过在 AutoResponder 里配置代理 url，并勾选 Enable rules 打开 AutoResponder 功能，就可以代理指定 url。

如图，配置 `http://haha.com` 响应一张图片。

除了替换响应资源，还可以指定 url 响应状态，比如 200，304 等等。

![request-proxy][3]

![request-proxy-result][4]

## 低网速环境模拟 ##
限速对于web前端研发是非常重要的，由于开发者的机器一般配置都很高，并且是在localhost下来调试程序，所以很难模拟到用户的真实使用情 况，如正在下载JS,css等静态资源的时候，页面的一个渲染情况。当网速很慢的时候，我们更希望看到的是先渲染出用户界面，而不是让用户看到一片空白。 那么这个时候，网络限速就能很方便在localhost针对类似的情况来做性能调试与优化。

启动方法：Rules → Performances → Simulate Modem Speeds

![rules-start][5]

如果需要模拟更低网速，可以修改模拟参数。
找到如下代码，修改这两个参数，保存退出。
**重新走一次启动方法，因为改动参数后模拟功能会自动关闭。**
模拟 2G 网络一般需要把参数设置在 20-50k。
![rules-code][6]


## 请求断点 ##
Rules → Automatic Breakpoints 里，前三个是断点控制选项。分别是 Before Requests(请求前断点)、 After Reponses(响应后断点) 和 Disabled。

通过设置断点，Fiddler可以做到：

1. 修改HTTP请求头信息。例如修改请求头的UA, Cookie, Referer 信息，通过“伪造”相应信息达到达到相应的目的（调试，模拟用户真实请求等）。

2. 构造请求数据，突破表单的限制，随意提交数据。避免页面js和表单限制影响相关调试。

3. 拦截响应数据，修改响应实体。

为什么以上方法是重要的？假设js前端程序员和服务器程序员是分工合作的，js程序员想要调试Ajax请求的功能，这样便不必等待服务器端程序员开发好所有接口之后再开始开发js端的ajax请求功能，因为通过“模拟”真实的服务器端的响应，便可以保证功能的正确性，而服务器端开发程序员，只要保证最终的响应是符合规定的即可。这大大简化了程序开发的效率，当然也降低了不同业务线程序员联调的难度。


参考：

 1. [用Fiddler模拟低速网络环境][7]
 2. [【HTTP】Fiddler（三）- Fiddler命令行和HTTP断点调试][8]


  [1]: https://dn-coding-net-production-pp.qbox.me/b9536b1a-5d6c-42fb-9fbc-635e7b87a7f2.png
  [2]: https://dn-coding-net-production-pp.qbox.me/37eb9bbd-3712-435f-bc0c-6d7cf6d9244e.png
  [3]: https://dn-coding-net-production-pp.qbox.me/62c0fd61-c111-4c51-b117-fb335efcc4a5.png
  [4]: https://dn-coding-net-production-pp.qbox.me/4cd05d94-b1a4-495a-8b38-a386657f030f.png
  [5]: https://dn-coding-net-production-pp.qbox.me/142fe135-8a38-4bde-94ba-8c2be7203bb9.png
  [6]: https://dn-coding-net-production-pp.qbox.me/4a7ac81b-89e1-4f9d-911e-c39be70040f9.png
  [7]: http://caibaojian.com/fiddler.html
  [8]: http://blog.csdn.net/ohmygirl/article/details/17855031