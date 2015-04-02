## 身份认证 ##
最普遍的身份认证方式就是用用户名（或 email）和密码做登陆操作。这就意味要实现一个登陆的表单，以便用户能够用他们个人信息登陆。这个表单看起来是这样的：

    <form name="loginForm" ng-controller="LoginController"
          ng-submit="login(credentials)" novalidate>
      <label for="username">Username:</label>
      <input type="text" id="username"
             ng-model="credentials.username">
      <label for="password">Password:</label>
      <input type="password" id="password"
             ng-model="credentials.password">
      <button type="submit">Login</button>
    </form>


既然这个是 Angular-powered 的表单，我们使用 ngSubmit 指令去触发上传表单时的函数。注意一点的是，我们把个人信息传入到上传表单的函数，而不是直接使用 $scope.credentials 这个对象。这样使得函数更容易进行 unit-test 和降低这个函数与当前 Controller 作用域的耦合。这个 Controller 看起来是这样的：

    .controller('LoginController', function ($scope, $rootScope, AUTH_EVENTS, AuthService) {
      $scope.credentials = {
        username: '',
        password: ''
      };
      $scope.login = function (credentials) {
        AuthService.login(credentials).then(function (user) {
          $rootScope.$broadcast(AUTH_EVENTS.loginSuccess);
          $scope.setCurrentUser(user);
        }, function () {
          $rootScope.$broadcast(AUTH_EVENTS.loginFailed);
        });
      };javascript:void(0);
    })

我们注意到这里是缺少实际的逻辑的。这个 Controller 被做成这样，目的是使身份认证的逻辑跟表单解耦。把逻辑尽可能的从我们的 Controller 里面抽离出来，把他们都放到 services 里面，这是个很好的想法。AngularJS 的 Controller 应该只管理 $scope 里面的对象（用 watching 或者 手动操作）而不是承担过多过分重的东西。

## 通知 Session 的变化 ##
身份认证会影响整个应用的状态。基于这个原因我更推荐使用事件（用 $broadcast）去通知 user session 的改变。把所有可能用到的事件代码定义在一个中间地带是个不错的选择。我喜欢用 constants 去做这个事情：

    .constant('AUTH_EVENTS', {
      loginSuccess: 'auth-login-success',
      loginFailed: 'auth-login-failed',
      logoutSuccess: 'auth-logout-success',
      sessionTimeout: 'auth-session-timeout',
      notAuthenticated: 'auth-not-authenticated',
      notAuthorized: 'auth-not-authorized'
    })

constants 有个很好的特性就是他们能随便注入到别的地方，就像 services 那样。这样使得 constants 很容易被我们的 unit-test 调用。constants 也允许你很容易地在随后对他们重命名而不需要改一大串文件。同样的戏法运用到了 user roles:

    .constant('USER_ROLES', {
      all: '*',
      admin: 'admin',
      editor: 'editor',
      guest: 'guest'
    })

如果你想给予 editors 和 administrators 同样的权限，你只需要简单地把 'editor' 改成 'admin'。

## The AuthService ##
与身份认证和授权（访问控制）相关的逻辑最好被放到同一个 service：

    .factory('AuthService', function ($http, Session) {
      var authService = {};
 
      authService.login = function (credentials) {
        return $http
          .post('/login', credentials)
          .then(function (res) {
            Session.create(res.data.id, res.data.user.id,
                           res.data.user.role);
            return res.data.user;
          });
      };
 
      authService.isAuthenticated = function () {
        return !!Session.userId;
      };
 
      authService.isAuthorized = function (authorizedRoles) {
        if (!angular.isArray(authorizedRoles)) {
          authorizedRoles = [authorizedRoles];
        }
        return (authService.isAuthenticated() &&
          authorizedRoles.indexOf(Session.userRole) !== -1);
      };
      return authService;
    })

为了进一步远离身份认证的担忧，我使用另一个 service（一个单例对象，using the [service style][1]）去保存用户的 session 信息。session 的信息细节是依赖于后端的实现，但是我还是给出一个较普遍的例子吧：

    .service('Session', function () {
      this.create = function (sessionId, userId, userRole) {
        this.id = sessionId;
        this.userId = userId;
        this.userRole = userRole;
      };
      this.destroy = function () {
        this.id = null;
        this.userId = null;
        this.userRole = null;
      };
      return this;
    })

一旦用户登录了，他的信息应该会被展示在某些地方（比如右上角用户头像什么的）。为了实现这个，用户对象必须要被 $scope 对象引用，更好的是一个可以被全局调用的地方。虽然 $rootScope 是显然易见的第一个选择，但是我尝试克制自己，不过多地使用 $rootScope（实际上我只在全局事件广播使用 $rootScope）。用我所喜欢的方式去做这个事情，就是在应用的根节点，或者在别的至少高于 Dom 树的地方，定义一个 controller 。<body> 标签是个很好的选择：

    <body ng-controller="ApplicationController">
      ...
    </body>

ApplicationController 是应用的全局逻辑的容器和一个用于运行 Angular 的 run 方法的选择。因此它要处于 $scope 树的根，所有其他的 scope 会继承它（除了隔离 scope）。这是个很好的地方去定义 currentUser 对象：

    .controller('ApplicationController', function ($scope,
                                                   USER_ROLES,
                                                   AuthService) {
      $scope.currentUser = null;
      $scope.userRoles = USER_ROLES;
      $scope.isAuthorized = AuthService.isAuthorized;
 
      $scope.setCurrentUser = function (user) {
        $scope.currentUser = user;
      };
    })

我们实际上不分配 currentUser 对象，我们仅仅初始化作用域上的属性以便 currentUser 能在后面被访问到。不幸的是，我们不能简单地在子作用域分配一个新的值给 currentUser 因为那样会造成 shadow property。这是用以值传递原始类型（strings， numbers， booleans，undefined and null）代替以引用传递原始类型的结果。为了防止 shadow property，我们要使用 setter 函数。如果想了解更多 Angular 作用域和原形继承，请阅读 [Understanding Scopes][2]。

## 访问控制 ##
身份认证，也就是访问控制，其实在 AngularJS 并不存在。因为我们是客户端应用，所有源码都在用户手上。没有办法阻止用户篡改代码以获得认证后的界面。我们能做的只是显示控制。如果你需要真正的身份认证，你需要在服务器端做这个事情，但是这个超出了本文范畴。

### 限制元素的显示 ###
AngularJS 拥有基于作用域或者表达式来控制显示或者隐藏元素的指令： ngShow， ngHide， ngIf 和 ngSwitch。前两个会使用一个 &lt<span>style<span>&gt 属性去隐藏元素，但是后两个会从 DOM 移除元素。

第一种方式，也就是隐藏元素，最好用于表达式频繁改变并且没有包含过多的模板逻辑和作用域引用的元素上。原因是在隐藏的元素里，这些元素的模板逻辑仍然会在每个 digest 循环里重新计算，使得应用性能下降。第二种方式，移除元素，也会移除所有在这个元素上的 handler 和作用域绑定。改变 DOM 对于浏览器来说是很大工作量的（在某些场景，和 ngShow/ngHide 对比），但是在很多时候这种代价是值得的。因为用户访问信息不会经常改变，使用 ngIf 或 ngShow 是最好的选择：

    <div ng-if="currentUser">Welcome, {{ currentUser.name }}</div>
    <div ng-if="isAuthorized(userRoles.admin)">You're admin.</div>
    <div ng-switch on="currentUser.role">
      <div ng-switch-when="userRoles.admin">You're admin.</div>
      <div ng-switch-when="userRoles.editor">You're editor.</div>
      <div ng-switch-default>You're something else.</div>
    </div>

### 限制路由访问 ###
很多时候你会想让整个网页都不能被访问，而不是仅仅隐藏一个元素。如果可以再路由（在UI Router 里，路由也叫状态）使用一种自定义的数据结构，我们就可以明确哪些用户角色可以被允许访问哪些内容。下面这个例子使用 [UI Router][3] 的风格，但是这些同样适用于 ngRoute。

    .config(function ($stateProvider, USER_ROLES) {
      $stateProvider.state('dashboard', {
        url: '/dashboard',
        templateUrl: 'dashboard/index.html',
        data: {
          authorizedRoles: [USER_ROLES.admin, USER_ROLES.editor]
        }
      });
    })

下一步，我们需要检查每次路由变化（就是用户跳转到其他页面的时候）。这需要监听 $routeChangStart（ngRoute 里的）或者 $stateChangeStart（UI Router 里的）事件：

    .run(function ($rootScope, AUTH_EVENTS, AuthService) {
      $rootScope.$on('$stateChangeStart', function (event, next) {
        var authorizedRoles = next.data.authorizedRoles;
        if (!AuthService.isAuthorized(authorizedRoles)) {
          event.preventDefault();
          if (AuthService.isAuthenticated()) {
            // user is not allowed
            $rootScope.$broadcast(AUTH_EVENTS.notAuthorized);
          } else {
            // user is not logged in
            $rootScope.$broadcast(AUTH_EVENTS.notAuthenticated);
          }
        }
      });
    })

## Session 时效 ##
身份认证多半是服务器端的事情。无论你用什么实现方式，你的后端会对用户信息做真正的验证和处理诸如 Session 时效和访问控制的处理。这意味着你的 API 会有时返回一些认证错误。标准的错误码就是 HTTP 状态吗。普遍使用这些错误码：

 -  401 Unauthorized — The user is not logged in
 -  403 Forbidden — The user is logged in but isn’t allowed access
 - 419 Authentication Timeout (non standard) — Session has expired
 - 440 Login Timeout (Microsoft only) — Session has expired

后两种不是标准内容，但是可能广泛应用。最好的官方的判断 session 过期的错误码是 401。无论怎样，你的登陆对话框都应该在 API 返回 401， 419， 440 或者 403 的时候马上显示出来。总的来说，我们想广播和基于这些 HTTP 返回码的时间，为此我们在 $httpProvider 增加一个拦截器：

    .config(function ($httpProvider) {
      $httpProvider.interceptors.push([
        '$injector',
        function ($injector) {
          return $injector.get('AuthInterceptor');
        }
      ]);
    })
    .factory('AuthInterceptor', function ($rootScope, $q,
                                          AUTH_EVENTS) {
      return {
        responseError: function (response) { 
          $rootScope.$broadcast({
            401: AUTH_EVENTS.notAuthenticated,
            403: AUTH_EVENTS.notAuthorized,
            419: AUTH_EVENTS.sessionTimeout,
            440: AUTH_EVENTS.sessionTimeout
          }[response.status], response);
          return $q.reject(response);
        }
      };
    })

这只是一个认证拦截器的简单实现。有个很棒的[项目][4]在 Github ，它做了相同的事情，并且使用了 httpBuffer 服务。当返回 HTTP 错误码时，它会阻止用户进一步的请求，直到用户再次登录，然后继续这个请求。

### 登录对话框指令 ###
当一个 session 过期了，我们需要用户重新进入他的账号。为了防止他丢失他当前的工作，最好的方法就是弹出登录登录对话框，而不是跳转到登录页面。这个对话框需要监听 notAuthenticated 和 sessionTimeout 事件，所以当其中一个事件被触发了，对话框就要打开：

    .directive('loginDialog', function (AUTH_EVENTS) {
      return {
        restrict: 'A',
        template: '<div ng-if="visible"
                        ng-include="\'login-form.html\'">',
        link: function (scope) {
          var showDialog = function () {
            scope.visible = true;
          };
  
          scope.visible = false;
          scope.$on(AUTH_EVENTS.notAuthenticated, showDialog);
          scope.$on(AUTH_EVENTS.sessionTimeout, showDialog)
        }
      };
    })

只要你喜欢，这个对话框可以随便扩展。主要的思想是重用已存在的登陆表单模板和 LoginController。你需要在每个页面写上如下的代码：

    <div login-dialog ng-if="!isLoginPage"></div>

注意 isLoginPage 检查。一个失败了的登陆会触发 notAuthenticated 时间，但我们不想在登陆页面显示这个对话框，因为这很多余和奇怪。这就是为什么我们不把登陆对话框也放在登陆页面的原因。所以在 ApplicationController 里定义一个 $scope.isLoginPage 是合理的。

## 保存用户状态 ##
在用户刷新他们的页面，依旧保存已登陆的用户信息是单页应用认证里面狡猾的一个环节。因为所有状态都存在客户端，刷新会清空用户信息。为了修复这个问题，我通常实现一个会返回已登陆的当前用户的数据的 API （比如 /profile），这个 API 会在 AngularJS 应用启动（比如在 "run" 函数）。然后用户数据会被保存在 Session 服务或者 $rootScope，就像用户已经登陆后的状态。或者，你可以把用户数据直接嵌入到 index.html，这样就不用额外的请求了。第三种方式就是把用户数据存在 cookie 或者 LocalStorage，但这会使得登出或者清空用户数据变得困难一点。

## 最后…… ##
鄙人才疏学浅，一点点经验，这是一篇翻译的文章，如有谬误，欢迎指正。

[huangruichang][5]

### 参考阅读 ###

 -  [Techniques for authentication in AngularJS applications][6]


  [1]: https://gist.github.com/Mithrandir0x/3639232
  [2]: https://github.com/angular/angular.js/wiki/Understanding-Scopes
  [3]: https://github.com/angular-ui/ui-router
  [4]: https://github.com/witoldsz/angular-http-auth
  [5]: https://coding.net/u/huangruichang
  [6]: https://medium.com/opinionated-angularjs/techniques-for-authentication-in-angularjs-applications-7bbf0346acec