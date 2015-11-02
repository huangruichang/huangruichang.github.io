# 浏览器调试功能使用心得 #

## 前言 ##
从写程序的第一天开始，调试器就一直伴随着我们程序员的职业生涯，作为前端人员，更是如此。调试器可以帮助我们在开发过程中定位网页中的显示、网络、性能等各式各样的问题。随着浏览器的飞速发展，依附在浏览器上的调试器功能也越发强大，其中常用的有 FireFox 和 Chrome 的调试器，**本文使用 Chrome 调试器介绍浏览器调试的常用功能的使用**。

> F12 和 Ctrl + Shift + i 组合键可打开调试器。或在网页元素上邮件->审查元素。

### Elements ###
用于查看、编辑 DOM 和 CSS 的功能。
通过直接在调试器上改动 dom 和 css，以测试和调整样式结构。
可以在 Elements 面板直接改动 dom 和 css 以调整样式，把最终样式复制到 css 或 html 文件上。如此一来，就不需要每次都在编辑器里改动，然后刷新网页。大大提高了开发效率和开发体验。

![elements][1]


### Network ###
用于查看网络请求的功能。
基本涵盖查看请求和相应内容，请求时间，请求状态等功能。
![network-1][2]

点击一个请求，就可以打开请求的详细信息，有四个 tab，分别是 Headers、Preview、Response、Timing。
Headers：查看请求头、响应头、和 query string 和表单信息
Preview：按照响应 Content-Type 预览响应资源。如响应返回格式是 image/jpeg，则预览显示为一张图片
Response：与 Preview 的使用基本一样，不同点可以看[这里][3]
Timing：请求时间
![network-2][4]

### Sources ###
此功能可以获取响应的 js 和 css 文件。列表按照域名划分。
可从列表里可以打开 js 文件，并且在文件中打断点进行调试代码。
**通过 Ctrl + O 打开搜索栏，可以快速搜索出 js 文件进行调试。**
![sources][5]

### Timeline ###
Timeline面板记录和分析了web应用运行时的所有活动情况，这是研究和查找性能问题的最佳途径。
通过柱状图和时序图进行性能分析，定位耗时长的操作。
改变选区或者点击柱条，可以直接定位到特定区间分析性能。
折线图展示页面占用内存变化。
左上角垃圾桶图表会强制 V8 进行一次 gc。
![timeline][6]

### Profiles ###
时间线(timeline)告诉我们代码运行花费的时间，但是并没有帮助我们知道代码运行的时候发生了什么。我们可以做一些改动然后不断的测每次代码运行的时间，但这是盲目的。profiles给我们提供了更好的方法。profiler告诉我们哪些函数的执行占用了大部分时间。让我们切换到chrome开发者工具的“Profiles”标签页开始性能测试，这里一共提供了三种类型的性能测试。

 1.  javascript cpu 使用情况
 2. 堆栈快照
 3. 堆栈记录

![profiles-0][7]

#### javascript cpu 使用情况 ####
此功能可以展现页面中哪里的 js 函数比较消耗时间。
![profiles-1][8]

#### 堆栈快照 ####
堆栈快照可以展示你的页面中从 js 对象到 DOM 节点的内存分配情况。
![profiles-2][9]

#### 堆栈记录 ####
记录一段时间内 js 对象的内存分配，以定位内存泄露。![profiles-3][10]

### Resources ###
Resources 功能用于查看网页在本地持久化，比如 cookie、Local Storage、Web SQL、IndexedDB 等。
![resources][11]

### Audits ###
Audits 功能可以审核页面并找出页面中可以提升性能的地方。比如移除无用的 css，使用 gzip压缩传输，合并 js 文件，压缩 cookie 等建议，并且提供具体的优化位置（文件或接口）。
![audit-1][12]

![audit-2][13]


  [1]: https://dn-coding-net-production-pp.qbox.me/2c553e09-7be6-4e71-90c0-361e502a3944.png
  [2]: https://dn-coding-net-production-pp.qbox.me/bb2cc25d-367c-4ebc-87ce-828c7a8ff3b7.png
  [3]: http://jingyan.baidu.com/article/90bc8fc80bd5f3f652640c4b.html
  [4]: https://dn-coding-net-production-pp.qbox.me/5acffd40-075f-41a2-9cb9-b134d45d082f.png
  [5]: https://dn-coding-net-production-pp.qbox.me/ba13500d-6185-421b-8a07-e02b2bd16e4d.png
  [6]: https://dn-coding-net-production-pp.qbox.me/568b34ba-2707-471b-8541-742717a9053c.png
  [7]: https://dn-coding-net-production-pp.qbox.me/67c703f6-1ca1-4090-bdb3-e57efd67162d.png
  [8]: https://dn-coding-net-production-pp.qbox.me/803d5a1d-de79-4f98-bceb-52f80e4f0029.png
  [9]: https://dn-coding-net-production-pp.qbox.me/75337a19-c43c-4350-983b-652c654ad419.png
  [10]: https://dn-coding-net-production-pp.qbox.me/a601533a-cef9-4949-bc35-0c24ee70a953.png
  [11]: https://dn-coding-net-production-pp.qbox.me/db1c5089-9ff6-4286-a3ab-aa591bb4e2be.png
  [12]: https://dn-coding-net-production-pp.qbox.me/e34f3b46-e91f-4f8b-a846-c6ec80e7ed3d.png
  [13]: https://dn-coding-net-production-pp.qbox.me/ab9b0e26-7cf8-4992-a1e5-067df47a6aee.png