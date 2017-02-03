
# 前端性能 checklist 2017

> 你已经开始使用渐进式启动了吗？React 和 Angular 里的 tree-shaking 和 code-splitting 呢？你安装了 Brotli 或者 Zopfli 压缩算法了吗，OCSP 封套和 HPACK 压缩算法呢？还有，资源提示，client hints，CSS Containment 呢 － 更别提 IPv6，HTTP/2 和 service workers 了。

在过去，性能通常只是事后想法。通常延后到项目的最后，一般归结为，压缩，连接，资源文件优化和一些可能可以调整的服务器相关的配置文件。现在看来，今非昔比。

性能不仅仅是技术要关心的事情：它非常重要，并且当合并到工作流中，设计决策需要了解他们对性能的含义。性能需要持续的被测试，监控和改进，还有持续增长的网页复杂度带来指标难以追踪的难题，因为指标在不同的设备，浏览器，协议，网络类型和延迟相差极大(CDN，运营商，缓存，代理，防火墙，负载均衡和服务器都在性能中扮演重要的角色)。

所以，当提高性能时，我们打造一个这些我们要铭记于心的东西的概览－从项目最初到站点发布的最后版本－看起来会是个什么样子呢？下面你会看到一个(希望是没有偏见和客观的)**2017的前端性能 checklist**－一个关于如何保证你的响应时间和站点流畅性的问题的概览。

## Front-End Performance Checklist 2017 ##
小的优化对保持性能是好事，但它对心中有明确目标的要求非常高－可测试的目标会彻底影响整个过程。这有好几种模型，以下讨论的几种是武断的－只需要确保你自己的优先顺序即可。

### *准备好，设定目标* ###

### 01.比你最快的对手还要快上 20% ###
根据[心理调查][1]，如果你想让用户感觉你的网站比其他人的网站快的话，你需要至少快上 20%。整页加载时间和指标无关，比如开始渲染时间。[首屏时间][2](需要展示页面主要内容的时间)和[可交互时间][3](单页应用可以开始交互的时间)。

选一台 Moto G，一台中档三星和一台大众机，比如 Nexus ，或者[oepn device lab][4] 测试开始渲染时间(用[WebPageTest][5])和首屏时间(用[Lighthouse][6])－在正常 3G、4G 和 WiFi 连接下。

![lighthouse-demo][7]

*Lighthouse, Google 新出的性能审计工具。*

从数据里分析出你的用户所在。

看到你的用户在哪个数据范围所在，你可以模仿百分之九十的的用户使用来测试。收集数据，制作一个 [speedsheet][8]，砍下 20%，并且以这种方法给自己一个目标(像 [performance budget][9])。那么现在，你已拥有了一些可以测试的东西了。如果你把这些测试内容谨记于心并且尝试把这些落地到最小化脚本，那么你就走在正确的道路上了！

![在这里输入图片描述][10]

*[Performance budget builder][11] builder by Brad Frost*

**与你的同事分享 checklist 。**确保每个你的团队里的成员都对 checklist 熟悉，以免造成对内容的误解。每个决策都有性能含义，并且当理念，用户体验和可视化设计在决定时，经过前端开发者积极地参与，项目将会极大地获得好处。把设计的决定转换到性能预算并且定义在 checklist 里的优先级。

### 02.响应时间 100ms，60帧每秒 ###

[RAIL performance model][12] 给了你好的目标：在初始化输入以后，以你做大的能力提供少于100ms的响应。为了能让响应 < 100ms，网页必须在每个 < 50ms 的间隔里把控制交回给主线程。对于高压力的点比如动画，最好不要在这个点做你能控制的事情和绝对的最少的你不能控制的事情。

另外，每一帧动画应该在16ms内完成，因此可以达到60帧每秒。(1s ÷ 60 = 16.6ms)－低于10ms更好。因为浏览器需要时间去画在屏幕上，你的代码需要在16.6ms这个点前完成运行。乐观点，明智地使用线程的空闲时间。显然易见地，这些目标适用于运行时性能，而不是记载时性能。

### 03.首屏时间低于1.25秒，SpeedIndex 少于 1000 ###

尽管这可能很难达到，你的最终目标应该是少于1s的开始渲染时间和低于1000的 SpeedIndex 值(在一个很快的连接)。对于首屏时间，最普遍的是以1.25s为标准。对于移动设备，少于3s的开始渲染时间在3G情况下是可以接受的。稍微高一点的值作为标准也是可以的，但是最好把这些值压的越低越好。

### *定义环境* ###

### 04.选择和安装你的工具 ###

今时今日不要花太多注意力在据说很酷炫的东西。忠于你的环境去构建，可能是 Grunt，Gulp，Webpack，PostCSS 或他们的组合。只要你可以快速地获得结果并且在维护构建过程的时候没有问题，就行了。

### 05.渐进式强化 ###

保持[渐进式强化][13]作为前端架构和部署的准则是非常稳的赌博。设计和构建核心体验优先，然后用浏览器兼容的高级特性去增强用户体验，创造[弹性][14]体验。如果你的网站在一台又慢、屏幕又差、网络又不怎么给力的机子上跑得快，那在给力一点的机子上跑得一顶更快。

### 06.Angular, React, Ember and co ###

选一个可以服务端渲染的框架。确保在用上框架前测试好在移动设备上服务端和客户端渲染模式的启动时间。(因为由于性能问题在后面换掉框架是非常困难的)。如果你确实用了一个 javascript 框架，要确保你的选择是被广而告之而且深思熟虑的。不同的框架在性能方面会有不同的副作用，而且需要不同的优化策略，所以你要对你在使用的框架知根知底。当构建一个网页应用时，看看 [PRPL pattern][15] 和 [application shell architecture][16]。

![在这里输入图片描述][17]

> PRPL 标准，意思是推送重要资源，渲染初始路由，预缓存剩下的路由和在需要的时候懒加载剩下的路由。

![在这里输入图片描述][18]

> 一个应用的外壳是一个支撑用户交互的最少 html，css 和 javascript。

### 07.AMP 还是 Instant Articles ###

取决于你的组织的优先事项和策略，你可能考虑像用 Google 的 [AMP][19] 或者 Facebook 的 Instant Articles。不用他们你也可以达到很好的性能，但是 AMP 用免费的 CDN 提供牢固的性能框架，而 Instant Articles 会在 Facebook 上提高你的性能。你也可以构建[progressive web AMPs][20]

### 08.明智地选择你的 CDN ###

取决于你有多少的动态数据，你可能可以外置静态内容到[静态网站生成器][21]，把它放到 CDN 上并且在之上提供静态版本，以此避免数据库请求。你甚至可以选择一个基于 CDN [静态部署平台][22]，用你的组件去丰富页面交互([JAMStack][23])

注意，CDN 也可以提供(卸下)动态内容。所以没必要把 CDN 限制在静态资源上。在 CDN 端，复核是否你的 CDN 展示压缩后内容和转换后内容，使用智能 HTTP/2 传输，使用[ESI(edge-side includes)][24] 这些组装页面动态内容和静态内容的技术，和其他任务。

### *构建优化* ###

### 09.清楚地设定你的优先事项 ###

弄清楚你要优先完成的事情是个好的点子。用一个清单记录你的所有资源(JavaScript，图片，字体，第三方脚本和页面上“昂贵”的模块，比如轮播图、复杂的图表和多媒体)，并且把他们按组分类。

搞一张电子表格。说明传统浏览器的基本的体验(也就是完全可访问内容)，兼容的浏览器的增强体验(就是增强过的、全面的体验)，和其他(那些不是绝对需要加载并且可以懒加载的，比如字体，非必要样式，轮播脚本，video 播放器，社交媒体按钮，大图)。我们发行过一篇关于"[提高 Smashing Magazine 站点性能][25]"的文章，这篇文章记录了分类方法的详细做法。

### 10.使用"cutting-the-mustard"技术 ###

使用 [cutting-the-mustard][26] 技术去为传统浏览器和现代浏览器分发基本体验和增强体验。严格对待内容加载:马上加载基本体验，增强的就放到 ````DomContentLoaded```` 里，其他的都放在 ```` load ```` 事件里。

注意，这项技术是根据浏览器版本去推断出设备能力。但是我们不能在今时今日这样做了。举个例子，在发展中国家的低端安卓机器大部分都运行着 Chrome 而且会 cut the mustard(屏蔽可以屏蔽的一切)，不管他们内存上线和 CPU 的能力。请意识到，我们别无选择，这项技术的使用在最近变得越来越受阻。

### 11.考虑小优化和渐进式启动 ###

在一些应用，在你渲染页面之前你可能需要一些时间去初始化应用。展示[骨架页][27]去代替 loading 指示。寻找库或者技术去提高初始化的渲染速度(比如，[tree-shaking][28] 和 [code-splitting][29])。因为大部分性能问题都是出现在初始化解析到应用启动的这段时间。另外，使用[预编译器][30]去[转移一些客户端渲染][31]到[服务端][32]，因此，输出页面就更快了。最后，为了更快的初始化 loading，可以考虑使用[Optimize.js][33]来热切地优化 IIFE 函数。

![在这里输入图片描述][34]

*[渐进式启动][35]指的是使用服务器渲染去得到更快的首屏时间，但也需要包括一些最小化的 JavaScript 去保持可交互时间紧贴首屏时间。*

客户端渲染还是服务端渲染？在两种情况下，我们的目标是达到[渐进式启动][36]:使用服务器渲染去得到更快的首屏时间，但也需要包括一些最小化的 JavaScript 去保持可交互时间紧贴首屏时间。然后我们可以，在要求或者时间允许，启动应用的非必需部分。不幸的是，如 [Paul Lewis ][37] 说的，框架通常不知道浮现于开发者的优先事项，因此渐进式启动在大部分库和框架里很难实现。如果你有时间和资源，最后可以使用这个策略去提高性能。

### 12.HTTP 缓存正确设置了吗 ###

复查 ```` expires ````，```` cache-control ````，```` max-age ```` 和其他 HTTP 缓存是否正确地设置。大体上，任何资源都可以缓存一段非常短的时间(如果资源频繁改动)或者永远(如果资源是静态的)。－你可以在需要的时候通过改变 URL 来更新资源版本。

如果可以，使用为指纹静态资源设计的```` Cache-control: immutable ````，来避免二次验证(2016年12月，[只在火狐][38]的 https 事务支持)。你可以用 [Heroku’s primer on HTTP caching headers][39]，Jake Archibald 的 [Caching Best Practices][40] 和 Ilya Grigorik 的 [HTTP caching primer][41] 来作为指导。

### 13.限制第三方库，异步加载 JavaScript ###

当用户请求页面，浏览器获取 HTML 和构造 DOM，然后获取 CSS 和构造 CSSDOM，然后用 DOM 和 CSSDOM 生成渲染树。如果有任何 JavaScript 需要解析，在代码解析完之前，浏览器会停止渲染页面，因此会延迟渲染。作为开发者，我们要明确地告诉不要等，去渲染。用 HTML 的 ```` defer ```` 和 ```` async ```` 去达到这个目的。

在实战中，我们应该[更喜欢  ```` defer ```` 胜过  ```` async ````][42]([在用户使用 <= IE9 成本][43]，因为你可能把脚本拆开几块)。并且，减少第三方库和脚本的冲击，尤其是社交按钮和嵌入的 ```` iframe ```` (比如地图)。你可以使用[静态社交分享按钮][44](比如 [SSBG][45])和使用[交互式地图的静态链接][46]替代。

### 14.图片适当地优化了吗 ###

尽可能地，使用用上了 ```` srcset ````，```` sizes ```` 的[响应式图片][47]和```` <picture> ````元素。当你已经这样做了，你可以用 [WebP 格式][48]，通过用提供 WebP 的 ```` <picture> ```` 元素和一个 JPEG 的降级处理(见 Andreas Bovens 的 [code snippet][49])或者用内容协商(使用 ```` Accept ```` 头)来达到。Sketch 原生支持 WebP，而且 WebP 图片也可以通过 PhotoShop 的插件 [WebP plugin for Photoshop][50] 来导出  WebP。[其他方法也是可以的][51]。

![在这里输入图片描述][52]

*[Responsive Image Breakpoints Generator][53] 自动化图片和标签生成器。*

你也可以使用 [client hints][54]，[它如今正逐渐得到浏览器的支持][55]。没有足够的资源去烘焙出一个老司机级别的响应式图片的标签？请用 [Responsive Image Breakpoints Generator][56] 或者 [Cloudinary][57] 这样的服务去自动优化图片。另外，在很多情况下，单独使用 ```` srcset ```` 和 ```` sizes ```` 将会得到重大的好处。在 Smashing Magazine，我们用后缀 ```` -opt ```` 为图片命名－比如，```` brotli-compression-opt.png ````；无论何时一张图片包含了后缀，团队里的每个人都知道这张图已经优化过了。

### 15.图片优化进阶 ###

当你在开发首页的时候，开发那种特定图片需要飞快加载的网页，要确保 JPEG 是渐进式的并且用 [mozJPEG][58] 压缩过了，或者用 [Pingo][59] 处理过的 PNG，[Lossy GIF][60] 处理过的 GIF 和 [SVGOMG][61] 处理过的 SVG。把图片非必要的部分模糊掉(用高斯模糊)以减少文件体积，最终你可能把图片转成黑白来进一步减少文件体积。对于背景图，用 PhotoShop 导出 0~10% 品质的图已经绝对地可以接受了。

觉得还不够好？好吧，你也可以用 [multiple][62] [background][63] [images][64] [technique][65] 来提高性能。

### 16.页面字体经过优化了吗 ###

很大可能你在使用象形文字和额外未背使用的特性。你可以要求你的字体制作组把字体分组，[或者你自己来][66]。如果你在用开源字体(比如，把拉丁文和带口音的文字放到一个文件)来压缩字体文件大小。[WOFF 支持][67]非常好，你可以用 WOFF 和 OTF 在那些不支持的浏览器作为降级处理。另外，选择一个 Zach Leatherman 的 [Comprehensive Guide to Font-Loading Strategies][68] 里的策略，用 service worker 去持久地缓存字体。需要快速见效？Pixel Ambacht 有一个[快速引导和案例学习][69]来让你的字体井然有序。

![在这里输入图片描述][70]

*Zach Leatherman 的 [Comprehensive Guide to Font-Loading Strategies][71] 提供了很多选择去优化网页字体奋发。*

如果你不能从你的服务器获取字体，并且依赖第三方主机，确保使用了 [Web Font Loader][72]。[FOUT 比 FOIT 要好][73]；请用正确的方式在降级处理时渲染文本，和异步加载字体－你也可以用 [loadCSS][74] 来做这个。你可能也可以[远离本地安装字体][75]了。

### 17.快速推送 critical CSS ###

为了保证浏览器尽可能快地渲染你的页面，收集需要渲染时第一眼看到的所有 CSS (公认为"critical CSS"或者"首屏 CSS")并且把它们加入到 ```` <head> ```` 行内是[普遍实践][76]，因此减少往返交换。由于在缓慢的开始阶段交换包大小的限制，你给 Critical CSS 的预算大约有14KB。如果你超过了这个限制，浏览器会需要额外的交换去获取更多样式。库[CriticalCSS][77] 和 [Critical][78] 让你可以这样做。你可能需要为每个模板弄一次 critical CSS。如果可以，考虑使用 Filament Group 青睐的 [conditional inlining approach][79]。

在 HTTP/2 下，critical CSS 可以被存放到分隔的 CSS 文件并且可以通过服务器推送分发而不需要让 HTML 变得臃肿。但是不足的是，服务器推送没有被一贯地支持，而且存在缓存问题(请看 [Hooman Beheshti 的 PPT][80]，从 114 页开始)。事实上，它带来的副作用会，带来[消极影响][81]而且会使网络变得臃肿，会阻碍文档真正的帧的传输。因为 TCP 的缓慢启动，服务器推送在[热连接上非常有效][82]。所以，你可能需要创建[可缓存的 HTTP/2 服务器推送机制][83]。谨记，尽管新的```` cache-digest ````[声明][84]可以不用手动地构建这种“可缓存”的服务器。

### 18.使用 tree-shaking 和 code-splitting 来减少负载 ###

[Tree-shaking][85]是一个以只包括生产环境使用的代码来清理你的代码的方式。你可以[使用 Webpack2 来清除没有用的输出][86]和用 [UnCSS][87] 或 [Helium][88] 来从 CSS 移除不需要的样式。另外，你可能想考虑学习怎样[写有效的 CSS 选择器][89]和怎样[避免臃肿和昂贵的样式][90]。

[Code-splitting][91] 是另一个 Webpack 特性，它会把你的代码分割成在需要时才加载的“块”。一旦你定义了你代码的分割点，Webpack 关心依赖和输出文件。基本地当应用请求时，它可以让你保持初始下载很小和在需要的时候请求。

注意到 [Rollup][92] 显著地表现得要比 Browserify  好。当我们在用 Rollup 时，你可能想看看 [Rollupify][93]，它会把 ES2015 模块合成一个大的 CommonJS 模块－因为小的模块可以有一个[惊人地高的性能成本][94]，取决于你选择打包还是模块系统。

### 19.提高渲染性能 ###

用 [CSS Containment][95] 使昂贵的组件脱离出来－比如，为了限制浏览器样式的作用域，布局的作用域和 off-canvas 导航的绘制，或第三方组件。确保当滚动的时候，动画没有延迟，而且你要持续保持60帧每秒。如果这不可能，那至少使每秒帧数达到 15~ 60。使用 CSS 的 ```` will-change ```` 属性来告诉哪个浏览器的元素和属性会变化。

另外，测量[运行时渲染属性][96](比如，[在 DevTools][97])。为了开始，请看看 Paul Lewis 免费的[优达学城上的浏览器渲染优化的课程][98]。我们也有一篇 Sergey Chikuyonok 的[关于怎么正确用 GPU 运行动画][99]的文章。

### 20.加热连接来加快传输 ###

用骨架屏幕，和懒加载所有昂贵的组件，比如字体，JavaScript，轮播图，视频和 iframe。使用 [resource hint][100]来节省 ```` dns-prefetch ````(在后台做的 dns 查找)，```` preconnect ````(要求浏览器在后台开始握手(DNS，TCP，TLS))，```` prefetch ````(要求浏览器请求资源)，```` prerender ````(要求浏览器在后台渲染特定页面)，和 ```` preload ````(prefetch 资源而不执行它们，包括其他东西)。注意到在实际中，取决于浏览器的支持，你会喜欢 ```` preconnect ```` 胜过 ```` dns-prefetch ````，并且你会小心使用 ```` prefetch ```` 和 ```` prerender  ````－后者应该在你对用户下一步是什么这件事非常有自信才能使用(比如，在购买漏斗模型)。

### *HTTP/2* ###

### 21.HTTP/2 准备 ###

在 Google 的[向更安全的网络进发][101]和 Chrome 上最终的所有 HTTP 页面处理都是"不安全的"，你需要决定是否继续使用 HTTP/1.1 还是搭建一个 [HTTP/2 环境][102]。HTTP/2 的[支持非常好][103]。它不会走远，在大多数情况下，使用它会让你情况好转。这个投资相当重要，但你迟早都要换上 HTTP/2。除了上述，你在使用 service workder 和服务器推送能得到[重大的性能提升][104](至少在长连接方面)。

![在这里输入图片描述][105]

*最终，Goolgle 计划把所有 HTTP 页面打上不安全的标签，和把 HTTP 安全提示改成那个用来表示不安全 HTTPS 的红色三角形。([图片来源][106])*

下载端需要你来迁移 HTTPS，而且取决于你的 HTTP/1.1 的用户基数(就是在用老旧操作系统或老旧浏览器的用户)，你需要分发不同的构建版本，也可能需要你适配一个[不同的构建进程][107]。注意：配置迁移和新的构建进程可能会很困难和花时间。在这篇文章的后面，我会假设你正在或者已经使用 HTTP/2。

### 22.正确地部署 HTTP/2 ###

再次，[通过 HTTP/2 分发资源][108]需要对你目前提供的资源进行彻底检查。你需要找到一个打包模块和加载小模块并行的平衡点。

一方面，你可能想避免所有资源完全打包，而是把你的整个界面拆分成多个小的模块，在构建进程里把它们压缩到一块，用 ["scout" 的方式][109]并且并行地加载它们。一个文件的改动不会引发整个样式或者 JavaScript 重新下载。

另一方面，[打包仍然很重要][110]，因为给浏览器分发小模块仍然存在问题。首先，**压缩是一场灾难**。大文件的压缩可以从字典复用中获得好处，然后小的分离的包就不能。有标准工序去处理它，但是对于现在来说太新潮了。再者，浏览器还没有为这种工作流作出优化。例如，Chrome 根据资源数直线上升地会触发[进程间通讯][111](IPCs)，所以包含上百个资源时浏览器会产生运行时成本。

![在这里输入图片描述][112]

*为了达到 HTTP/2 最好的结果，考虑[渐进地加载 CSS][113]，就像 Chrome’s Jake Archibald 的建议。*

你仍然可以[渐进地加载 CSS][114]。显然地，这样做会积极地处罚 HTTP/1.1 的用户，所以你可能需要生成和提供不同的构建给浏览器来作为你部署进程的一部分，这样会轻微地变得复杂。你可以从 [HTTP/2 连接合并][115]中获得侥幸，它能让你在 HTTP/2 中使用域名碎片来获得好处，但是实际操作非常困难。

那咋弄呢？如果你在使用 HTTP/2，发送**10个包左右**看起来像是个正确的妥协(并且对老旧浏览器不会太差)。实验和测试来找到你网站的平衡点。

### 23.确保你服务器的安全性是坚实的 ###

所有实现了 HTTP/2 的浏览器都在 TLS 上运行，所以你可能会想避免安全警告和一些元素在你的页面上不生效的情况。复查[头里的安全字段是否设置正确][116]，[排查已知攻击点][117]，和[检查你的证书][118]。

还没有迁移到 HTTPS？看看 [HTTPS-Only 标准][119]这个全面指导。另外，确保所有外在的插件和跟踪脚本都使用 HTTPS 加载，确保不允许跨站脚本和确保 [HTTP 严格传输安全头][120]和[内容安全策略头][121]是否设置正确。

### 24.你的服务器和 CDN 支持 HTTP/2 吗 ###

不同服务器和 CDN 可能对 HTTP/2 的支持也不同。以 [Is TLS Fast Yet?][122] 这篇文章为指导去检查你的设置，或者扫一眼你的服务器的运行情况和什么你期望的特性会得到支持。

![在这里输入图片描述][123]

*[Is TLS Fast Yet?][124] 能让在切换 HTTP/2 的时候你查看你服务器和 CDN 的设置。*

### 25.Brotli 或者 Zopfli 压缩算法用上了吗 ###

上一年，Google [推出的][125] [Brotil][126]，一个新的开源无损数据格式，如今已被 Chrome，FireFox 和 Opera [广泛地支持][127]了。实际上，Brotil 表现得比 Gzip 和 Deflate [还好][128]。它压缩的时候可能很慢，但又由于设置，更慢的压缩最终会带来更高的压缩率。还有，它解压时很快。由于来自 Google 的算法，浏览器只在用户浏览 HTTPS 网站时才会接受这种格式就变得不足为奇了。－还有确实，在技术上也存在一些原因。美中不足的是，Brotil 如今没有在大部分的服务器上预装，而且如果不用自己编译的 nginx 或 Ubuntu 的话安装起来非常困难。然而，你仍然可以在不支持 Brotil 的 CDN 上使用它(用一个 service worker)。

要不，你可以看看 [Zopfli 的压缩算法][129]，它会对数据进行 Deflate，Gzip 和 Zlib 格式的压缩。任何 Gzip 压缩的资源会从 Zopfli 的改良的 Deflate 编码中获得好处，因为文件会比 Zlib 的最大压缩小 3%~8%。不足的是，文件压缩的时间大概是80倍。这就是在不经常改变的资源和那些压缩一次下载多次的文件上可以使用 Zopfli 的原因。

### 26.OCSP stapling 开启了吗 ###

在[在你服务器上开启 OCSP stapling][130]中，你可以为你的 TLS 握手加速。Online Certificate Status Protocol (OCSP) 被创造来作为 Certificate Revocation List (CRL) 协议的替代品。两个协议都是用于检查 SSL 证书是否被取消。然而，OCSP 协议不需要要求浏览器去花费时间去下载和查询证书信息列表，因此减少了握手的时间。

### 27.你采用了 IPv6 了吗 ###

因为我们的 [IPv4 空间正在消耗殆尽][131]了，并且移动网络迅速地才用 IPv6(美国[已经有][132]50%的 IPv6 使用率了)，[把你的 DNS 升级到 IPv6][133]来保持服务器未来的坚挺是个好点子。只需要保证访问的双重网络得到支持就可以了－这就可以让 IPv6 和 IPv4 同时各自运行了。毕竟，IPv6 不会向上兼容。另外，[研究表明][134]由于邻居发现he路由优化，那些网站快上了10~15%。

### 28.HPACK 压缩在使用了吗 ###

如果你在用 HTTP/2 了，复查你的服务器[实现了 HPACK 压缩][135]，用于响应头来减少不必要开支。用 HPACK 来作例子，因为现在 HTTP/2 服务器相当新潮，他们可能没有完全支持这个规范。[H2spec][136] 是个非常好的工具用于检查 [HPACK 生效][137]。

![在这里输入图片描述][138]

*H2spec([查看大图][139])([图片来源][140])*

### 29.service worker 用来缓存和网络降级处理了吗 ###

没有任何网络优化比得上在用户机器上缓存。如果你的网站正在用 HTTPS，按照"[Pragmatist’s Guide to Service Workers][141]"来用 service worker 缓存静态资源和存储掉线的降级处理(甚至是掉线页面)并且从用户的机器上获取他们，而不是再走一次网络。另外，看看 Jake 的[掉线 Cookbook][142]和优达学城的免费课程"[掉线的网页应用][143]"。浏览器支持？在[这里][144]，但无论如何降级处理都网络。

### *测试与监控* ###

### 30.监控混合内容警告 ###

如果你最近从 HTTP 迁移到 HTTPS，用类似于 [Report-URI.io][145] 的工具确保监控活跃和不活跃的混合内容警告。你也可以使用[混合内容扫描][146]来扫描你开启了 HTTPS 的网站的混合内容。

### 31.你的在 DevTools 里的部署工作流经过优化了吗 ###

选择一个调试工具并且按下每一个按钮。确保你清楚怎样分析性能和控制台输出，和怎样调试 JavaScript 代码和编辑 CSS 样式。Umar Hansa 最近准备了一个(巨型的)[PPT][147]和[演讲][148]，覆盖了很多在调试和测试中费解的提示和要去注意的技术。

### 32.你曾经在代理浏览器和老旧浏览器上测试过吗 ###

只在 Chrome 和 FireFox 上测试是不够的。观察你的网页在代理浏览器和老旧浏览器上的工作情况。比如，UC 浏览器和 Opera Mini，[在亚洲占据很大的市场份额][149](在亚洲高达35%)。在你的有效的国家里[测量平均网速][150]来避免日后发生大惊喜。用节流网络和模拟的高 DPI 设备来测试。[BroswerStack][151]非常棒，并且也可以在真机上测试。

### 33.持续监控配置好了吗 ###

有一个私有的 [WebPagetest][152] 实例总是对快速和无上限的测试有好处的。用自动报警配置持续的性能预算监控。把你的用户时间打点设置来测试和监控业务相关的性能。用 [SpeedCurve][153] 来监控随着时间的过去性能的改变，或者/和使用 [New Relic][154] 去了解到那些 WebPagetest 不能提供的数据。另外，看看 [SpeedTracker][155]，[Lighthouse][156] 和 [Calibre][157]。

## 快速见效 ##

这个列表是相当容易理解的，并且相比所有优化只取相当一部分。所以，如果你只有一个小时去获得重大的改善，你会怎么做？让我们凝练到10条唾手可得的建议把。显然地，在你开始前和完成时，测量结果，包括在 3G 上和有线网络上的开始渲染时间和 SpeedIndex 。

 1. 你的目标是开始渲染时间在有线网络上低于1s和在 3G 上低于3s，SpeedIndex 在1000以下。优化开始渲染时间和可交互时间。
 2. 给你主模版准备好 critical CSS，并且在页面的 ```` <head> ```` 里包含它(你的预算是 14KB)。
 3. 尽可能地延迟和懒加载大量的脚本，包括你的第三方脚本－特别是媒体按钮，视频播放器和昂贵的 JavaScript。
 4. 用 dns-lookup，preconnect，prefetch，preload 和 prerender  加上资源提示来加快传输速度。
 5. 把字体分组并且异步加载它们(或者用系统字体代替)。
 6. 优化图片，在关键页考虑使用 WebP(比如首页)。
 7. 检查 HTTP 缓存和安全头字段设置正确。
 8. 打开服务器上的 Brotil 或 Zopfli 压缩(如果不行，不要忘了打开 Gzip)。
 9. 如果 HTTP/2 可以，打开 HPACK 压缩和开始监控混合内容警告。如果你跑在 LTS 上，也要打开 OCSP stapling。
 10. 如果可以，尽可能地把诸如字体，样式，JavaScript 和图片这些资源缓存起来－用 service worker。

## 最后 ##

译自[《Front-End Performance Checklist 2017 》][158]，如有纰漏，欢迎指正。


  [1]: https://www.smashingmagazine.com/2015/09/why-performance-matters-the-perception-of-time/#the-need-for-performance-optimization-the-20-rule
  [2]: https://developers.google.com/web/tools/lighthouse/audits/first-meaningful-paint
  [3]: https://developers.google.com/web/tools/lighthouse/audits/time-to-interactive
  [4]: https://www.smashingmagazine.com/2016/11/worlds-best-open-device-labs/
  [5]: http://www.webpagetest.org/
  [6]: https://github.com/GoogleChrome/lighthouse
  [7]: https://dn-coding-net-production-pp.qbox.me/0aa8c7ca-01ab-4a06-a188-82636517f83e.png
  [8]: http://danielmall.com/articles/how-to-make-a-performance-budget/
  [9]: http://bradfrost.com/blog/post/performance-budget-builder/
  [10]: https://dn-coding-net-production-pp.qbox.me/52d7f598-6d51-415e-8985-9e39a087565d.png
  [11]: http://bradfrost.com/blog/post/performance-budget-builder/
  [12]: https://www.smashingmagazine.com/2015/10/rail-user-centric-model-performance/
  [13]: https://www.aaron-gustafson.com/notebook/insert-clickbait-headline-about-progressive-enhancement-here/
  [14]: https://resilientwebdesign.com/
  [15]: https://developers.google.com/web/fundamentals/performance/prpl-pattern/
  [16]: https://developers.google.com/web/updates/2015/11/app-shell
  [17]: https://dn-coding-net-production-pp.qbox.me/e2658e49-aea9-41a1-81c1-59b0b9972dc1.png
  [18]: https://dn-coding-net-production-pp.qbox.me/c1dde813-fe71-483c-8610-04c4bc64714e.jpg
  [19]: https://www.ampproject.org/
  [20]: https://www.smashingmagazine.com/2016/12/progressive-web-amps/
  [21]: https://www.smashingmagazine.com/2015/11/static-website-generators-jekyll-middleman-roots-hugo-review/
  [22]: https://www.smashingmagazine.com/2015/11/modern-static-website-generators-next-big-thing/
  [23]: https://jamstack.org/
  [24]: http://baike.baidu.com/link?url=uN6MJKpAq9eC6jpdnrg_BatRD1AG9RMwVYydhCypbKtDJ817gUoZm4OXjdVTTNI8ahdj53hSoksnk9B9plzzH_
  [25]: https://www.smashingmagazine.com/2014/09/improving-smashing-magazine-performance-case-study/
  [26]: http://responsivenews.co.uk/post/18948466399/cutting-the-mustard
  [27]: https://twitter.com/lukew/status/665288063195594752
  [28]: https://medium.com/@richavyas/aha-moments-from-ngconf-2016-part-1-angular-2-0-compile-cycle-6f462f68632e#.8b9afnsub
  [29]: https://webpack.github.io/docs/code-splitting.html
  [30]: https://www.lucidchart.com/techblog/2016/09/26/improving-angular-2-load-times/
  [31]: https://www.smashingmagazine.com/2016/03/server-side-rendering-react-node-express/
  [32]: http://redux.js.org/docs/recipes/ServerRendering.html
  [33]: https://github.com/nolanlawson/optimize-js
  [34]: https://dn-coding-net-production-pp.qbox.me/d9bb69f5-9217-4c4d-aed3-46c53f9fb71b.jpeg
  [35]: https://aerotwist.com/blog/when-everything-is-important-nothing-is/
  [36]: https://aerotwist.com/blog/when-everything-is-important-nothing-is/
  [37]: https://aerotwist.com/blog/when-everything-is-important-nothing-is/#which-to-use-progressive-booting
  [38]: https://aerotwist.com/blog/when-everything-is-important-nothing-is/#which-to-use-progressive-booting
  [39]: https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers
  [40]: https://jakearchibald.com/2016/caching-best-practices/
  [41]: https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=en
  [42]: https://calendar.perfplanet.com/2016/prefer-defer-over-async/
  [43]: https://github.com/h5bp/lazyweb-requests/issues/42
  [44]: https://www.savjee.be/2015/01/Creating-static-social-share-buttons/
  [45]: https://simplesharingbuttons.com/#intro
  [46]: https://developers.google.com/maps/documentation/static-maps/intro
  [47]: https://www.smashingmagazine.com/2014/05/responsive-images-done-right-guide-picture-srcset/
  [48]: https://www.smashingmagazine.com/2015/10/webp-images-and-performance/
  [49]: https://dev.opera.com/articles/responsive-images/#different-image-types-use-case
  [50]: http://telegraphics.com.au/sw/product/WebPFormat#webpformat
  [51]: https://developers.google.com/speed/webp/docs/using
  [52]: https://dn-coding-net-production-pp.qbox.me/b0e8446d-2796-4be9-8182-9f74953d1d20.jpeg
  [53]: http://www.responsivebreakpoints.com/
  [54]: https://www.smashingmagazine.com/2016/01/leaner-responsive-images-client-hints/
  [55]: http://caniuse.com/#search=client-hints
  [56]: http://www.responsivebreakpoints.com/
  [57]: http://cloudinary.com/documentation/solution_overview#account_and_api_setup
  [58]: https://github.com/mozilla/mozjpeg
  [59]: http://css-ig.net/pingo
  [60]: https://kornel.ski/lossygif
  [61]: https://jakearchibald.github.io/svgomg/
  [62]: https://csswizardry.com/2016/10/improving-perceived-performance-with-multiple-background-images/
  [63]: https://jmperezperez.com/medium-image-progressive-loading-placeholder/
  [64]: https://manu.ninja/dominant-colors-for-lazy-loading-images#tiny-thumbnails
  [65]: https://css-tricks.com/the-blur-up-technique-for-loading-background-images/
  [66]: https://www.fontsquirrel.com/tools/webfont-generator
  [67]: http://caniuse.com/#search=woff2
  [68]: https://www.zachleat.com/web/comprehensive-webfonts/
  [69]: https://pixelambacht.nl/2016/font-awesome-fixed/
  [70]: https://dn-coding-net-production-pp.qbox.me/c036d5ca-1a6a-45e9-8622-610c16ebf442.png
  [71]: https://www.zachleat.com/web/comprehensive-webfonts/
  [72]: https://github.com/typekit/webfontloader
  [73]: https://www.filamentgroup.com/lab/font-events.html
  [74]: https://github.com/filamentgroup/loadCSS
  [75]: https://www.smashingmagazine.com/2015/11/using-system-ui-fonts-practical-guide/
  [76]: https://www.smashingmagazine.com/2015/08/understanding-critical-css/
  [77]: https://github.com/filamentgroup/criticalCSS
  [78]: https://github.com/addyosmani/critical
  [79]: https://www.filamentgroup.com/lab/performance-rwd.html
  [80]: http://www.slideshare.net/Fastly/http2-what-no-one-is-telling-you
  [81]: http://calendar.perfplanet.com/2016/http2-push-the-details/
  [82]: https://docs.google.com/document/d/1K0NykTXBbbbTlv60t5MyJvXjqKGsCVNYHyLEXIxYMv0/edit
  [83]: https://css-tricks.com/cache-aware-server-push/
  [84]: https://calendar.perfplanet.com/2016/cache-digests-http2-server-push/
  [85]: https://medium.com/@roman01la/dead-code-elimination-and-tree-shaking-in-javascript-build-systems-fb8512c86edf
  [86]: http://www.2ality.com/2015/12/webpack-tree-shaking.html
  [87]: https://github.com/giakki/uncss
  [88]: https://github.com/geuis/helium-css
  [89]: http://csswizardry.com/2011/09/writing-efficient-css-selectors/
  [90]: https://benfrain.com/css-performance-revisited-selectors-bloat-expensive-styles/
  [91]: https://webpack.github.io/docs/code-splitting.html
  [92]: http://rollupjs.org/
  [93]: https://github.com/nolanlawson/rollupify
  [94]: https://nolanlawson.com/2016/08/15/the-cost-of-small-modules/
  [95]: http://caniuse.com/#search=contain
  [96]: https://aerotwist.com/blog/my-performance-audit-workflow/#runtime-performance
  [97]: https://developers.google.com/web/tools/chrome-devtools/rendering-tools/
  [98]: https://www.udacity.com/course/browser-rendering-optimization--ud860
  [99]: https://www.smashingmagazine.com/2016/12/gpu-animation-doing-it-right/
  [100]: https://w3c.github.io/resource-hints
  [101]: https://security.googleblog.com/2016/09/moving-towards-more-secure-web.html
  [102]: https://http2.github.io/faq/
  [103]: http://caniuse.com/#search=http2
  [104]: https://www.youtube.com/watch?v=RWLzUnESylc&t=1s&list=PLNYkxOF6rcIBTs2KPy1E6tIYaWoFcG3uj&index=25
  [105]: https://dn-coding-net-production-pp.qbox.me/04578842-7e9a-45ea-91ab-41c7cdea8b23.png
  [106]: https://security.googleblog.com/2016/09/moving-towards-more-secure-web.html
  [107]: https://rmurphey.com/blog/2015/11/25/building-for-http2
  [108]: https://www.youtube.com/watch?v=yURLTwZ3ehk
  [109]: https://rmurphey.com/blog/2015/11/25/building-for-http2
  [110]: http://engineering.khanacademy.org/posts/js-packaging-http2.htm
  [111]: https://www.chromium.org/developers/design-documents/inter-process-communication
  [112]: https://dn-coding-net-production-pp.qbox.me/965a3c0f-7680-4c04-b85a-bf3d2e836e9e.png
  [113]: https://jakearchibald.com/2016/link-in-body/
  [114]: https://jakearchibald.com/2016/link-in-body/
  [115]: https://daniel.haxx.se/blog/2016/08/18/http2-connection-coalescing/
  [116]: https://securityheaders.io/
  [117]: https://www.smashingmagazine.com/2016/01/eliminating-known-security-vulnerabilities-with-snyk/
  [118]: https://www.ssllabs.com/ssltest/
  [119]: https://https.cio.gov/faq/
  [120]: https://www.owasp.org/index.php/HTTP_Strict_Transport_Security_Cheat_Sheet
  [121]: https://content-security-policy.com/
  [122]: https://istlsfastyet.com/
  [123]: https://dn-coding-net-production-pp.qbox.me/785e36b4-aa9c-4c92-b80d-9cb4c1c539aa.png
  [124]: https://istlsfastyet.com/
  [125]: https://opensource.googleblog.com/2015/09/introducing-brotli-new-compression.html
  [126]: https://github.com/google/brotli
  [127]: http://caniuse.com/#search=brotli
  [128]: https://samsaffron.com/archive/2016/06/15/the-current-state-of-brotli-compression
  [129]: https://blog.codinghorror.com/zopfli-optimization-literally-free-bandwidth/
  [130]: https://www.digicert.com/enabling-ocsp-stapling.htm
  [131]: https://en.wikipedia.org/wiki/IPv4_address_exhaustion
  [132]: https://www.google.com/intl/en/ipv6/statistics.html#tab=ipv6-adoption&tab=ipv6-adoption
  [133]: https://www.paessler.com/blog/2016/04/08/monitoring-news/ask-the-expert-current-status-on-ipv6
  [134]: https://www.cloudflare.com/ipv6/
  [135]: https://blog.cloudflare.com/hpack-the-silent-killer-feature-of-http-2/
  [136]: https://github.com/summerwind/h2spec
  [137]: https://www.keycdn.com/blog/http2-hpack-compression/
  [138]: https://dn-coding-net-production-pp.qbox.me/39d8c3f5-2159-4611-85de-fa612d87e44b.png
  [139]: https://www.smashingmagazine.com/wp-content/uploads/2016/12/h2spec-example-large-opt.png
  [140]: https://github.com/summerwind/h2spec
  [141]: https://github.com/lyzadanger/pragmatist-service-worker
  [142]: https://jakearchibald.com/2014/offline-cookbook/
  [143]: https://www.udacity.com/course/offline-web-applications--ud899
  [144]: http://caniuse.com/#search=serviceworker
  [145]: https://report-uri.io/
  [146]: https://github.com/bramus/mixed-content-scan
  [147]: https://umaar.github.io/devtools-optimise-your-web-development-workflow-2016/#/
  [148]: https://www.youtube.com/watch?v=N33lYfsAsoU
  [149]: http://gs.statcounter.com/#mobile_browser-as-monthly-201511-201611
  [150]: https://www.webworldwide.io/
  [151]: https://www.browserstack.com/
  [152]: http://www.webpagetest.org/
  [153]: https://speedcurve.com/
  [154]: https://newrelic.com/browser-monitoring
  [155]: https://speedtracker.org/
  [156]: https://github.com/GoogleChrome/lighthouse
  [157]: https://calibreapp.com/
  [158]: https://www.smashingmagazine.com/2016/12/front-end-performance-checklist-2017-pdf-pages/?from=message&isappinstalled=0#getting-ready-and-setting-goals
