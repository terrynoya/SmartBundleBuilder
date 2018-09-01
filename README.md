Smart Unity AssetBundle Builder

特性
-自动分析依赖关系，避免资源重复打包
-自动化的打包粒度管理
-支持导出dot文件，利用[graphviz](https://www.graphviz.org/)查看资源的依赖关系，最终导出AssetBundle包的依赖关系

# Troubleshooting

# 问题
为什么我打的AssetBundle包会发生资源重复（Asset Duplication）？

# 原因

[点我去看官方文档](https://docs.unity3d.com/Manual/AssetBundles-Troubleshooting.html) 

>Objects that are explicitly assigned to an AssetBundle will only be built into that AssetBundle. An Object is “explicitly assigned” when >that Object’s AssetImporter has its assetBundleName property set to a non-empty string.

>Any Object that is not explicitly assigned in an AssetBundle will be included in all AssetBundles that contain 1 or more Objects that >reference the untagged Object.

资源明确设置assetBundleName的，一定会被打到那个指定名字的AssetBundle中
没有设置assetBundleName的，会被包含在所有AssetBundle中，从而发生资源重复

# 解决方法

被多个其他资源引用的资源，需要单独打包（设置assetBundleName）

让我们先看看其他解决办法

Unity提供的AssetBundleManager

SmartBundleBuilder的解决方法

SmartBundleBuilder提供了自动分析依赖关系功能，能识别被多次依赖的资源，为它们指定AssetBundleName，从而避免了资源重复的发生

# 如何实现

## 概述
SmartBundleBuilder是如何实现分析依赖关系的？

SmartBundleBuilder利用Editor中的AssetDatabase.GetDependencies方法，对每一个需要打包的资源进行依赖关系分析，并生成一个[有向图](https://en.wikipedia.org/wiki/Directed_graph)来保存这些资源的依赖关系

图的顶点表示一个资源文件，边表示一个资源对另一个资源的依赖关系，A->B，表示A被B依赖，这条边对A来说是[出度(Out-degree)](https://zh.wikipedia.org/wiki/%E5%9B%BE_(%E6%95%B0%E5%AD%A6))，对B来说是入度

如果一个Asset节点的出度大于1，说明该节点被多个资源依赖，可能需要单独打包

