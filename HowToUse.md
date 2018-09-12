# How To Use

## 使用Pipeline设计模式实现打包流程
SmartBundleBuilder使用了Pipeline设计模式，内置的标准的流程如图：

![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/pipeline.jpeg)

对应的代码如下：

每个模块可以根据需求更换，比如音乐文件不需要进行依赖分析的，可以删除前4步。
