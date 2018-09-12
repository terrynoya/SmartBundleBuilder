# How To Use

## 使用Pipeline设计模式实现打包流程

SmartBundleBuilder使用了[Pipeline](https://medium.com/@aaronweatherall/the-pipeline-pattern-for-fun-and-profit-9b5f43a98130)设计模式，
pipeline模式使每个处理步骤更独立，更有利于重用，可以根据需要实现自己的pipeline

## Pipeline结构

### Pipeline
Pipeline是一个list容器，list中存放ValeBase节点

### ValeBase
ValeBase是pipeline中的节点

派生子类实现具体的需求，例如FileDependcyAnylize是ValvBase的子类之一，用来生成文件依赖关系有向图

### Payload
Payload是pipeline中需要处理的上下文，BundlePayload是payload的子类，用来存放pipeline处理后产生的数据，或者提供参数

pipeline.Process()

内置的标准的流程如图：

![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/pipeline.jpeg)

pipeline中的处理节点都派生自ValveBase，按照需求实现对payload进行处理

### 实现一个处理文件依赖关系分析的ValvBase代码

---
```C#
/// <summary>
/// 资源依赖关系分析
/// </summary>
namespace SmartBundle
{
    public class FileDependcyAnylize : ValveBase
    {
        public override void Excute(Payload payload)
        {
            BundlePayload context = (BundlePayload) payload;
            MAssetFileDependencyGraphGenerator fileGraphGen =
                new MAssetFileDependencyGraphGenerator(context.FileDependcyGraph);
            string[] filePathList = context.Files;
            for (int i = 0; i < filePathList.Length; i++)
            {
                string filePath = filePathList[i];
                fileGraphGen.AddFileFromPath(filePath);
            }
            MDirectedGraph graph = fileGraphGen.Graph;
            //删除多余依赖关系
            TransitiveReduction.Reduce(graph);

            this.Complete();
        }
    }
}
```


## 代码实现

---
```C#
/// <summary>
/// 标准处理管道
/// </summary>
/// <param name="files"></param>
/// <param name="outputPath"></param>
/// <returns></returns>
public static Pipeline CreateStandardPipeline(string[] files, string outputPath)
{
    Pipeline pipe = new Pipeline();

    pipe.Pipe(new FileDependcyAnylize())
        .Pipe(new LogFileDependcyGraph())
        .Pipe(new MergeFileToPackage())
        .Pipe(new PackageGraphCircleCheck())
        .Pipe(new ClearBundleNames())
        .Pipe(new SetBundleNames())
        .Pipe(new BuildBundle())
        .Pipe(new GenerateHintFile())
        .Pipe(new RenameManifestBinaryFile())
        .Pipe(new RemoveAllManifestTextFiles());

    return pipe;
}
```


每个模块可以根据需求更换，比如音乐文件不需要进行依赖分析的，可以删除前4步。
