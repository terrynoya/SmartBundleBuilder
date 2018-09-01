Smart Unity AssetBundle Builder

特性
自动分析依赖关系，避免资源重复打包
自动化的打包粒度管理

Troubleshooting

问题
为什么我打的AssetBundle包会发生资源重复（Asset Duplication）？

原因

[点我去看官方文档](https://docs.unity3d.com/Manual/AssetBundles-Troubleshooting.html) 

>Objects that are explicitly assigned to an AssetBundle will only be built into that AssetBundle. An Object is “explicitly assigned” when >that Object’s AssetImporter has its assetBundleName property set to a non-empty string.

>Any Object that is not explicitly assigned in an AssetBundle will be included in all AssetBundles that contain 1 or more Objects that >reference the untagged Object.

资源明确设置assetBundleName的，一定会被打到那个指定名字的AssetBundle种
没有设置assetBundleName的，会被包含在所有AssetBundle中，从而发生资源重复

解决方法
被多个其他资源引用的资源，需要单独打包（设置assetBundleName）

让我们先看看其他解决办法
Unity提供的AssetBundleManager

SmartBundleBuilder的解决方法
SmartBundleBuilder提供了自动分析依赖关系功能，能识别被多次依赖的资源，为它们指定AssetBundleName，从而避免了资源重复的发生

SmartBundleBuilder是如何实现依赖关系的？
SmartBundleBuilder利用Editor中的AssetDatabase.GetDependencies，对每一个需要打包的资源进行依赖关系分析，并生成一个[有向图](https://en.wikipedia.org/wiki/Directed_graph)来保存这些资源的依赖关系


