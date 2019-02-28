# Miller547719886的博客

## Vue2.6源码解析系列

引用一位阿里前端大牛的话：

如果你想阅读一个比较复杂的框架的源码，并有很透彻的理解，比如 Vue源码，那么至少你需要满足如下几个条件：

1. 前端基础知识扎实，JS基础扎实，flow，Rollup等要熟悉
2. 有大型项目的架构能力，理解如何进行封装,解耦，复用，理解如何进行分层设计，知道大型项目如何处理依赖、组件通信等问题。
3. 对Vue本身要很熟悉，有项目开发经验，写过插件等
4. 可以先看一些别人的源码分析文章
5 有时间和耐心
6 读源码并不是一个简单的工作，如果说jQuery源码的复杂度是1，那么我觉得Vue应该是10。有一个小技巧就是，在读源码的时候，自己创建一个项目，把不太理解的地方写出来，然后断点调试，可能会有意想不到的发现。

本源码解析将与[simple-vue](https://github.com/Miller547719886/simple-vue/tree/master)的实现同步进行。
