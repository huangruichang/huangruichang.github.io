# 自制简易 redux 开发简单页面

## 前言

最近团队需要开发一个新的购买页面，在考量是否 mvvm 框架后，最终选择时间一个简单的 redux 来组织代码。

## 购买页开发难点

在腾讯云，购买页由各团队自己维护，仅提供统一组件库，所以购买页业务可以自由组织。由于历史原因，前人做的两个产品的购买页(纯 jquery)已经被改得无法维护。代码里充满了对 dom 的操作，稍微改一点规格选择的逻辑，就很容易出 bug。

为了让页面变得稳定可用，必须要摸索出一套新的方案组织购买页的代码。

## 技术选型

腾讯云里的购买页是以单个 js 文件接入的，可以理解成以单页应用的形式接入，所以可以考虑上 vue 或者 react 等 mvvm 框架，方便日后持续迭代和维护。

但是为了一个小的单页应用上 webpack + vue 或 react，有点接受不了。业务源码才几 kb，然后上一个几十上百 kb 的框架，有点杀鸡用牛刀的感觉。

仔细想想，其实我要的不是 mvvm，而是一种简洁、可维护的代码组织方式，redux 就是一个优雅的方式。

## 为什么是 redux

1. 日常开发用的是 react + redux，接口比较友好
2. redux 是  flux 架构的一个实现。可以享受单向数据流带来的好处

## 具体实现

核心代码如下

```
 //@notice simple flux
    var createStore = function (reducer, initialState) {
        return {
            state: initialState || {},
            listeners: [],
            dispatch: function (action, payload) {
                var oldState = this.getState()
                var tmp = reducer(oldState, action)
                this._setState(tmp)
                var newState = this.getState()
                _.map(this.listeners, function (value, index) {
                    value(newState, oldState)
                })
            },
            _setState: function (state) {
                this.state = state
            },
            getState: function () {
                return this.state
            },
            subscribe: function (callback) {
                this.listeners.push(callback)
            }
        }
    }
```

上面代码简单实现了 redux 的 createStore 函数，返回的 store 主要使用 dispatch、getState 和 subscribe 接口。

接下来我们看看如何使用

```
//@notice action types
    var SELECT_REGION = 'SELECT_REGION'
    var SELECT_BANDWIDTH = 'SELECT_BANDWIDTH'
    var SELECT_BANDWIDTH_PEAK = 'SELECT_BANDWIDTH_PEAK'
    var SELECT_BANDWIDTH_TRANSFLOW = 'SELECT_BANDWIDTH_TRANSFLOW'
    var SELECT_TIMESPAN = 'SELECT_TIMESPAN'
    var SHOW_OTHER_TIMESPAN = 'SHOW_OTHER_TIMESPAN'
    var CHANGE_GOODS_NUM = 'CHANGE_GOODS_NUM'
    var SET_AUTO_RENEW = 'SET_AUTO_RENEW'
    var RESET_ALL = 'RESET_ALL'

    //@notice reducer
    var reducer = function (state, action) {
        //special handler
        // if (action === actionType) ...

        if (action === SELECT_BANDWIDTH_TRANSFLOW) {

            var _state_ = _.assign({}, state)
            _.assign(_state_, action.payload)
            return _state_
        }

        //default handler
        var _state_ = _.assign({}, state)
        _.assign(_state_, action.payload)
        return _state_
    }

    var initialState = {
        regionId: 8,
        bandwidth: 20,
        bandwidthPeak: 0,
        transflow: 100,
        timespan: 1,
        otherTimespan: false,
        goodsNum: 1,
        autoRenewFlag: 0
    }

    var store = createStore(reducer, initialState)
    store.subscribe(function () {
        /* dom 挂载 */
        index.mount()
        /* 价格请求 */
        index.queryPrice()
        /* 文本设置 */
        setTransflowTips()
        setElasticLimitFluxTips()
        setCCPeakValue()
    })
```

声明 actionType 和 redux 一致，由于没有 webpack 和 babel，不能使用 es6 语法，reducers 使用 if 语句代替 switch 语句。
然后就是视图更新的方式。在这个购买页里，dom 直接就充当视图层了，视图状态直接从 store 里获取，所以把 store 的 subscribe 接口实现了，方便每次改变状态进行视图更新。

```
var index = {
    ...
    mount: function () {
        this.renderRegion()
        this.renderBandwidth()
        this.renderBandwidthPeak()
        this.renderTransflow()
        this.renderTimespan()
        this.renderNumber()
    }
    ...
}
```

状态分发主要集中在 index.mount 函数里，这个函数集中放置对各个组件进行挂载和渲染的函数，每个 render 函数的用法有固定的写法套路。比如 renderRegion：

```
var index = {
        ...
        renderRegion: function () {
            var state = store.getState()
            // state 分发，把 state 手工分发到传入组件的参数里
            setSelected(REGION_LIST, state.regionId, 'value')

            // 组件渲染
            if (!this.regionList) {
                //init
                this.regionList = regionSelector.create({
                    list: REGION_LIST,
                    renderTo: '#regionList',
                    change: function (value, name) {
                        var payload = _.assign({}, initialState)
                        // 业务操作，这里是重置规格选择
                        store.dispatch({
                            type: RESET_ALL,
                            payload: _.assign(payload, {
                                regionId: value
                            })
                        })
                       // 其他操作
                        $('#other-timespan').show();
                    }
                })
            } else {
                this.regionList.setOptions({
                    list: REGION_LIST
                })
            }
        }
        ...
    }
```

大概就是，先把从 store 里获取的 state 分发到各个业务逻辑或者组件参数，然后对组件进行挂载初始化，如果组件之前已经挂载过，就更新组件参数（前提是组件提供变更配置接口）。

组件的 change 函数必须约束所有 state 操作都经过 store.dispatch 调用。

如此一来，基本上能模仿出 redux 的形态，在编码的时候遵循上述定下的套路，基本上能做到逻辑可插拔，状态可预测，而且也不会出现不可预测和不可调试的、混乱的 dom 操作（不是说完全没有 dom 操作）。
