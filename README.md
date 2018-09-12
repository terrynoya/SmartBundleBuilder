# Smart Unity AssetBundle Builder

一个聪明的Assetbundle打包系统

# 代码整理中，尚待发布

# [如何使用](/HowToUse.md)

#### 特性

*自动分析依赖关系，避免资源重复打包

*自动化的打包粒度管理

*支持导出dot文件，利用[graphviz](https://www.graphviz.org/)查看资源的依赖关系，最终导出AssetBundle包的依赖关系


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

SmartBundleBuilder利用Editor中的[EditorUtility.CollectDependencies](https://docs.unity3d.com/ScriptReference/EditorUtility.CollectDependencies.html)方法，对每一个需要打包的资源进行依赖关系分析，并生成一个[有向图](https://en.wikipedia.org/wiki/Directed_graph)来保存这些资源的依赖关系

图的顶点表示一个资源文件，边表示一个资源对另一个资源的依赖关系，A->B，表示A被B依赖，这条边对A来说是[出度(Out-degree)](https://zh.wikipedia.org/wiki/%E5%9B%BE_(%E6%95%B0%E5%AD%A6))，对B来说是入度

如果一个Asset节点的出度大于1，说明该节点被多个资源依赖，可能需要单独打包

## 步骤

SmartBundleBuilder主要分为个阶段

资源依赖关系分析阶段
1.使用EditorUtility.CollectDependencies方法，对被分析的资源的依赖关系生成有向图
2.删除重复的依赖关系

资源合并阶段

导出阶段

### 1.资源依赖关系分析阶段

#### 1.1收集依赖关系

这个阶段使用EditorUtility.CollectDependencies方法，对被分析的资源的依赖关系生成有向图
如图所示

![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/asset_depend_graph_by_api.jpg)

可以看到T0.png->Aura03_Red_cube.prefab，T0.png->Aura03_Red.mat形成了2个依赖，按照出度大于1就要被打包的原则，T0.png需要被单独打包

#### 1.2重复依赖关系删除

#### 打包粒度

##### 问题

假设我们只需要导出Aura03_Red_cube.prefab这一个AssetBundle，从[打包粒度](https://answer.uwa4d.com/question/58e5bd96e042a5c92c3484ec)上考虑，T0.png的需要单独导出会显得粒度太细。将T0.png，Aura03_Red.mat，Aura03_Red_cube.prefab合在一起打成一个包是合理的。

##### 解决方法

虽然T0.png的出度大于2，但是2个出度最终都指向了Aura03_Red_cube.prefab（直接和间接的），所以T0.png->Aura03_Red_cube.prefab的依赖可以被认为是**重复的**，需要被删除。

##### 如何实现

我们可以利用有向图的[Transitive reduction](https://en.wikipedia.org/wiki/Transitive_reduction)算法，保持到达度是一致的前提下对图进行简化。
简化后的依赖关系如图所示

![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/asset_depency_simple.jpg)


### 2.资源合并阶段

这个阶段将会对没有必要单独打包的资源进行合并，避免打包粒度过细。算法如下：

如果资源A只被资源B依赖，即A只有一个出度，那就将A与B进行合并，同时将A的入度合入新的组（即AB组）

![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/asset_depency_simple.jpg)
未合并
![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/merge_step_0.jpg)
合并第一次
![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/merge_final.jpg)
合并第二次，完成

### 3.综合阶段1和2
再来看一个复杂的例子，包含分析阶段和合并阶段

![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/complicate_start.jpg)
1.我们获得了一个资源依赖关系的有向图
![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/complicate_reduct.jpg)
2.简化重复的依赖关系
![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/complicate_step_0.jpg)
3.合并迭代一次
![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/complicate_step_2.jpg)
4.合并迭代二次
![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/complicate_final.jpg)
5.再无可合并的资源组，合并结束

### 4.打包阶段

# 参考知识链接

[资源依赖正确性测试](https://gist.github.com/QXSoftware/35a07738f481245d08b948ead3743a4b)

[JGraphT，一个强大的图结构和算法的实现库](https://github.com/jgrapht/jgrapht)

[Unity AssetBundle Reporter，冗余检测与资源分析工具 ](https://github.com/akof1314/AssetBundleReporter)

[Hero开发笔记-客户端资源更新](http://www.dpull.com/blog/2015-01-23-hero_assetbundle)



