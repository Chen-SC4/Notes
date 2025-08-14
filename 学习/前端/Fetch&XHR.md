在使用浏览器的开发者工具时，我们有的时候会在网络板块检查网络请求，当网络请求比较多时，我们往往会使用 `Fetch/XHR` 作为过滤条件。什么是 `Fetch/XHR` ？它过滤了什么？

## Ajax

`Ajax (Asynchronous JavaScript and XML)` ，它的核心思想是：在不加载整个网页的前提下，与服务器进行数据交换并更新部分网页的内容。而实现 `Ajax` 这个思想 (方法) 的技术就是 `XMLHttpRequest (XHR)`。

## Fetch&XHR

随着技术的发展，开发者逐渐觉得 `XHR` 用起来有些复杂和不方便。所以，又出现了一个更加现代化的 `API`，它就是 `Fetch`。

在浏览器的开发者工具中，过滤条件 `Fetch/XHR` 就表示 `Fetch` 和 `XHR` 这两种技术所发出的**异步请求**。