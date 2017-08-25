---
title: React 项目开发中的常见问题
date: 2017-07-21
tag: 
- 前端 
- React 
---

## React 中, 在 `Controlled`(受控制)的文本框中输入中文 `onChange` 会触发多次

[Issues](https://github.com/facebook/react/issues/3926)

通过输入法输入中文,日文，不管哪种一种用拼音的、笔划的，只要按下键盘的动作都会触发文本框的 `change` 事件。 但是 `React` 中的受控组件都通过触发 `change` 事件后得到的值再去更新组件的`value`, 这样不停的触发 `change`，导致 `value` 一直在更新，组件就一直在 render,  可能会造成组件运行逻辑问题，输入框内出现很多字符，而且中文输入不进去。

在 DOM events 中另外有 3 个事件可以辅助监听一段文字的输入：

- compositionstart : 事件触发于一段文字的输入之前；
- compositionupdate : 事件触发于字符被输入到一段文字的时候；
- compositionend : 事件触发于一段文字的输入完成。

我们可以通过这 3 个事件判断当前输入是否完整的输入，再触发 `onChange` 事件，这样就能解决多次触发 `onChange` 的问题。 还有一个重要的问题，虽然 `onChange` 要等一次完整输入后才触发，但是文本框上的显示需要是实时的，所以还要考虑在 `state` 中维护一个 `innerValue` 做实时更新，这样就没有问题了。

> 另外这 3 个事件在各个浏览器上的兼容性不一样，所以还要考虑兼容性的问题。 具体详细的实现可以参考 [form-lib](https://github.com/rsuite/form-lib/blob/master/src/createFormControl.js)。




## Warning: It looks like you're using a minified copy of the development build of React. When deploying React apps to production, make sure to use the production build which skips development warnings and is faster.

当 React 升级到 v15.* 以后，可能会见到这个警告，它是意思说你在生产环境中使用的是一个开发环境 minified 的代码。 

那 React 它是怎么办判断你正在访问的是生产环境，而不是开发环境呢？ 它是通过 webpack 知道的，因为最终发布生产环境的代码都是通过 webpack 命令打包。如果你是在开发环境，那肯定是用 webpack-dev-server。  在生产环境中不应该包括开发中使用的所有额外代码，所以在 webpack 部署环境中需要添加以下配置：

```js
module.exports = {
  //...
  plugins:[
    new webpack.DefinePlugin({
      'process.env':{
        'NODE_ENV': JSON.stringify('production')
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress:{
        warnings: true
      }
    })
  ]
  //...
}
```
参考[Make your own React production version with webpack](http://dev.topheman.com/make-your-react-production-minified-version-with-webpack/)



## setState(...): Can only update a mounted or mounting component. This usually means you called setState() on an unmounted component. This is a no-op.

这个警告是说，在一个已经卸载的组件上调用 `setState` 是无效的。出现这种情况一般都是异步操作造成的，比如一个异步数据请求，响应后执行 `setState` 去更新组件，但是如果在异步请求未完成前，路由发生变化，页面当前组件被卸载，这个时候异步请求没有被中止，导致等请求完成后就会执行 `setState` 试图去更新组件，就会报当前错误。

解决办法很简单，一般都是在执行 `setState` 之前判断一下当前组件是否被卸载，如果没有被卸载，才执行 `setState`。在 ES6 以前可以通过 `this.isMounted()` 来判断当前组件是否装载，如下：

```js
if(this.isMounted()) { // This is bad.
  this.setState({...});
}
```
但是在 ES6 以后不能用这种方式，这是一种反模式，参考[isMounted is an Antipattern](https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html)

可以利用组件的生命周期自己维护一个 `isMounted` 状态， 在 `componentDidMount` 中把 `isMounted` 设置为 `true`,
在 `componentWillUnmount` 再将 `isMounted` 设置为 `false`。 通过 `isMounted` 变量来判断组件是否装载。



## Each child in an array or iterator should have a unique `key` prop.

这个警告很明确，在遍历子元素的时候，每一个子元素都应该有一个唯一的 `key`。

参考官网相关说明 [lists and keys](https://facebook.github.io/react/docs/lists-and-keys.html#keys)

但是这里需要注意，避免使用数组的 index 来作为属性 key 的值，推荐使用唯一 ID。
参考 [Index as a key is an anti-pattern](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318)

```js
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```


## 在 IE 11 控制台报错：Objects are not valid as a React child

错误的全文: [error-decoder](https://facebook.github.io/react/docs/error-decoder.html?invariant=31&args%5B%5D=object%20with%20keys%20%7B%24%24typeof%2C%20type%2C%20key%2C%20ref%2C%20props%2C%20_owner%7D&args%5B%5D=)

这问题一般会在开发环境中遇到，在使用 React 15.4 以后，如果使用了 `react-hot-loader` 则必须在热加载之前加载 `babel-polyfill`, 在你的 `webpack.config.js` 中参考如下配置:

```js
entry: [
  'babel-polyfill',
  'react-hot-loader/patch',
  'webpack-dev-server/client?http://127.0.0.1:3000',
  'webpack/hot/only-dev-server',
  path.resolve(__dirname, './scr/index')
]
```

## 在 IE 11 控制台报错：Minified exception occurred; use the non-minified dev environment for the full error message and additional helpful warnings.

如果出现这个错误是提示，在本地环境配置 `NODE_ENV=development` ，再看一下控制台是否输出了一些更详细的错误信息，参考 [Introducing React's Error Code System](https://facebook.github.io/react/blog/2016/07/11/introducing-reacts-error-code-system.html) 。

如果这个错误只在 IE 11 浏览器环境出现，一般都是 ES6+ 的语法在IE存在兼容问题，一个比较通用的解决办法就是在项目中引入一个 `babel-polyfill`

```
npm i —save babel-polyfill
```

在项目入口，引入 `babel-polyfill` 

```js
import 'babel-polyfill'
```

> 在 webpack 构建的时候会兼容性处理

## 在 IE 11 控制台报错： Promise is undefined

IE 11 不支持 Promise 对象，解决办法

```
npm i —save es6-promise
```

在项目入口，引入`es6-promise`

```js
import 'es6-promise/auto';
```

## 在 IE 11 浏览器上 FontIcon 图标不显示

这个问题和 React 没有关系，但是这里也记录一下。
在 IE11 会下载 .ttf/.woff 字体文件， 通过 Network 我们可以看到字体文件 `response headers` 中有一个 `Pragma：no-cache`,由于 IE 似乎有缓存和字体的问题，所有导致图标不能正常显示。所以删除 WEB 服务(Nginx..)中的 `Pragma：no-cache` 和 `Cache-Control：no-store` 就能正常访问。

参考 [IE and Cache-Control](https://github.com/FortAwesome/Font-Awesome/issues/6454)

## 在 React 中使用 debounce 
如果需要在 React 组件中使用 debounce 方法可以参考下面代码
```javascript
class SearchBox extends React.Component {
    constructor(props) {
        super(props);
        this.method = debounce(this.method,300);
    }
    method() {
        //...
    }
    render(){
        return(
            <input onKeyUp={e => {
              e.persist();
              this.handleKeyDown(e);
            }}/>
        )
    }
}
```
因为 `React` 中 `SyntheticEvent` 是共用的，也就是说 `SyntheticEvent`对象将会循环使用。而每次执行完事件后，它所有的属性都将会失效（所有属性的值都被置为了 `null`）。所以在调用 `debounce` 方法前，一定要先调用 `e.persist()`来移除 `SyntheticEvent` ，并允许 `event`对象被保留下来以用于你自己的代码。
参考文章:
- [Perform debounce in React.js](https://stackoverflow.com/questions/23123138/perform-debounce-in-react-js#answer-28046731)
- [React Document SyntheticEvent](https://facebook.github.io/react/docs/events.html#event-pooling)

> 本文作者：[郭小铭](https://github.com/simonguo)、[Godfery](https://github.com/hiyangguo)