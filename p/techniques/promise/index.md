## 初识 Promise ##
在上年的年末，我们工作室在学校接了一个[项目][1]，当时一[小伙伴][2]带起 NodeJS 的学习热潮，于是我们选择用 NodeJS 搭起了我们项目的后端。当时项目里引入了形形色色的模块，令我觉得眼花缭乱(当时我是这么觉得的)，其中有一个 Promise 的模块让我最为深刻，当时我们用的是 Promise 的这个 [bluebird][3] 实现。

## 什么是 Promise ##
Promise 是一种让异步代码书写起来更优雅的模式，能够让异步操作代码像同步代码那样书写并且阅读，比如下面这个异步请求的例子：

    loadImg('a.jpg', function() {
      loadImg('b.jpg', function() {
        loadImg('c.jpg', function() {
          console.log('all done!');
        });
      });
    });

使用 Promise 模式可转变为：

    var loadImage = new Promise(function(img) {
        loadImg(img);
    });
    loadImage('a.jpg').then(function() {
        return loadImage('b.jpg');
    }).then(function () {
        return loadImage('c.jpg');
    }).then(function () {
        console.log('all done!');
    });

简而言之，就是可以把异步形式的代码转变成同步代码形式。摆脱回调地狱。like this↓
曾经有个小偷潜入某神秘机构，偷出机密代码的最后一页，打开一看。

                                        });
                                    });
                                });
                            });
                        });
                    });
                });
            });
        });
    });


## 一种同步函数与异步函数联系和通信的方式 ##
同步函数最重要的两个特征
 1. 能够返回值
 2. 能够抛出异常

> 这其实和高等数学中的复合函数(function composition)很像：你可以将一个函数的返回值作为参数传递给另一个函数，并且将另一个函数的返回值作为参数再传递给下一个函数……像一条“链”一样无限的这么做下去。更重要的是，如果当中的某一环节出现了异常，这个异常能够被抛出，传递出去直到被 catch 捕获。

而在传统的异步操作中不再会有返回值，也不再会抛出异常——或者你可以抛出，但是没有人可以及时捕获。这样的结果导致必须在异步操作的回调中再嵌套一系列的回调，以防止意外情况的发生。

而 Promise 模式恰好就是为这两个缺憾准备的，它能够实现函数的复合与异常的抛出（冒泡直到被捕获）。符合 Promise 模式的函数必须返回一个 promise，无论它是 [fulfilled][4] 状态也好，还是 [failed(rejected)][5] 状态也好，我们都可以把它当做同步操作函数中的一个返回值：

    $.get("/user/784533") // promise return
        .then(function parseHandler(info) {
        var userInfo = parseData(JSON.parse(info));
        return resolve(userInfo); // promise return
    })
    .then(getCreditInfo) // promise return
    .then(function successHandler(result) {
       console.log("User credit info: ", result);
    }, function errorHandler(error) {
       console.error("Error:", error);
    })

这样以来函数复合便一目了然：

    try {
        var info = $.get("/user/784533"); //Blocking
        var userInfo = parseData(JSON.parse(info));
    
        var resolveResult = parseData(userInfo);
        var creditInfo = getCreditInfo(resolveResult); //Blocking

        console.log("User credit info: ", result);
    } cacth(e) {
        console.error("Error:", error);
    }

## AngularJS 中的 Promise ##
前端调用后端 api 的时候有一些场合也需要同步代码实现(比如在某个用户的 profile 加载完成以前，不可以渲染"用户资料")。最近学习AngularJS，所以便介绍 Angular 中的 Promise。

在 Angular 中的 Promise 实现是 [$q][6]：

    function asyncGreet(name) {
      // perform some asynchronous operation, resolve or reject the promise when appropriate.
      return $q(function(resolve, reject) {
        setTimeout(function() {
          if (okToGreet(name)) {
            resolve('Hello, ' + name + '!');
          } else {
            reject('Greeting ' + name + ' is not allowed.');
          }
        }, 1000);
      });
    }

    var promise = asyncGreet('Robin Hood');
    promise.then(function(greeting) {
      alert('Success: ' + greeting);
    }, function(reason) {
      alert('Failed: ' + reason);
    });

## 最后…… ##
鄙人才疏学浅，一点点经验，如有谬误，欢迎指正。

> [huangruichang][7]



## 参考阅读 ##

 1. [【翻译】Promises/A+ 规范][8]
 2. [Promise/A 的误区以及实践][9]
 3. [AngularJS 文档][10]


  [1]: https://github.com/node-fun/siyuan
  [2]: https://github.com/fritx
  [3]: https://github.com/petkaantonov/bluebird
  [4]: http://www.ituring.com.cn/article/66566
  [5]: http://www.ituring.com.cn/article/66566
  [6]: https://github.com/kriskowal/q
  [7]: https://coding.net/u/huangruichang
  [8]: http://www.ituring.com.cn/article/66566
  [9]: http://jishu.zol.com.cn/210138.html
  [10]: http://docs.angularjs.cn/