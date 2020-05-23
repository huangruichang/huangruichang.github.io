# 使用 electron 抓取动态页面数据

## 背景

团队小伙伴有个需要抓取页面的需求，该页面是 websocket 直接推送到页面，并且刷新频率非常快。不熟悉 websocket 抓取数据的我，只能另辟蹊径，尝试从 dom 上下手。首先需要解决两个问题：

1.  页面刷新频率太快了，如何保证获取数据的正确性
2.  成功抓取数据后，如何落地

## 解决方案

### MutationObserver

在平时开发中，我们经常会使用到控制台的'审查元素'这个功能。在审查元素中的页面源码，我们随便选中一个 dom，右键，会发现有个 break on 功能，可以让 dom 产生变化的时候进行断点。所以，是不是 js 也有一个接口，可以让我获取到 dom 的变化？是的，正是 [MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver "MutationObserver") 接口（此前还有 [Mutation Events](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Mutation_events "Mutation Events") 接口，不过已被 mdn 标记为废弃）。使用这个接口，就可以获取到正确的数据流水了，解决了问题 1。

### electron

electron 是一个基于 chromium(chrome 的实验版本) 和 nodejs 的，可以使用前端技术开发跨平台桌面应用的工具。因为他是基于 chromium 和 nodejs，所以不仅拥有网页 javascript 的所有 api，并且可以使用 nodejs 接口进行文件操作使得获取到的数据能够落地，这样就决绝了问题 2。

## 代码

```javascript
const { app, BrowserWindow } = require("electron");
const { ipcMain } = require("electron");
const fs = require("fs");
function createWindow() {
  win = new BrowserWindow({ width: 800, height: 600 });
  win.loadURL("http://ssa.yundun.com/DDOS");
  //使用 electron 的接口，在页面加载后执行抓取脚本
  win.webContents
    .executeJavaScript(
      `
        setTimeout(() => {
        // ========= 抓取脚本开始 ============
        if (document.cookie.indexOf('china') === -1) {
            document.querySelector('#mapSwitchBtn').click()
        }
        const { ipcRenderer } = require('electron');
        let realTimeDdosAttackDetails = document.querySelector('#realTimeDdosAttackDetails');
        let mo = new MutationObserver(function () {
        let ols = realTimeDdosAttackDetails.querySelectorAll('ol');
        if (ols) {
        const result = [];
        for (let ol of ols) {
        console.log(ol.title);
        result.push(ol.title);
        }
        // ========= 抓取脚本结束 ============
        
        //通过 electron 的 ipc 接口，把数据传到 nodejs
        ipcRenderer.send('asynchronous-message', result.join(';'))
        }
        });
        let options = {
            'childList': true,
            'arrtibutes': true
        };
        
        // 使用 MutationObserver 监听 dom 变化
        mo.observe(realTimeDdosAttackDetails, options);
        }, 3000)
  `
    )
    .then(() => {
      console.log("start!!");
    });
}
ipcMain.on("asynchronous-message", (event, arg) => {
  //收到数据，并且落地到本地文件
  const records = arg.split(";");
  records.map((record) => {
    const lines = record.split("\n");
    const attackInfo = lines[0];
    const srcInfo = lines[1];
    const dstInfo = lines[2];
    const attackTokens = attackInfo.split("\t");
    const srcTokens = srcInfo.split("\t");
    const dstTokens = dstInfo.split("\t");
    const time = attackTokens[0];
    const mbps = attackTokens[1];
    const attack_type = attackTokens[2];
    const src_ip = srcTokens[0];
    const src_ip_loc = srcTokens[1];
    const dst_ip = dstTokens[0];
    const dst_ip_loc = dstTokens[1];
    const item = `"time":"${time}","mbps":"${mbps}","attack_type":"${attack_type}","src_ip":"${src_ip}","src_ip_loc":"${src_ip_loc}","dst_ip":"${dst_ip}","dst_ip_loc":"${dst_ip_loc}"\n`;
    let fd;
    try {
      fd = fs.openSync("yundun.txt", "a");
      fs.appendFileSync(fd, item, "utf8");
      console.log(`running:${new Date()}`);
    } catch (err) {
      console.log(err);
    } finally {
      if (fd !== undefined) fs.closeSync(fd);
    }
  });
});
app.on("ready", createWindow);
```

![](https://user-images.githubusercontent.com/4167510/82728245-c7cffe00-9d21-11ea-8c3a-648c8108c80f.gif)

## 如何跑在服务器

在很早以前，有开发者基于 electron 开发出自动化测试工具 [nightmare](https://github.com/segmentio/nightmare "nightmare")。作为自动化测试工具，肯定存在需要无头运行的场景（headless）。在 nightmare 的项目 [issue#224](https://github.com/segmentio/nightmare/issues/224#issuecomment-141575361 "issue#224") 里，有人已经解决了这个问题，并且成功地跑在了 ubuntu 上。借鉴这个方案，我也成功地把抓取脚本跑在了 CentOS 上。

## 优缺点

### good

1. 贴近前端技术，抓包脚本可以轻松扩展
2. 不需要分析 http api 各种参数和头，直接用选择器就能获取到数据，简单直接

### bad

运行的是 chromiun，并且运行时需要加载完整页面，性能比不上直接请求 http api，并且占用较多计算机资源。
