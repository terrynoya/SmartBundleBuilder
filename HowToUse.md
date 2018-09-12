# How To Use

## 使用Pipeline设计模式实现打包流程

SmartBundleBuilder使用了[Pipeline](https://medium.com/@aaronweatherall/the-pipeline-pattern-for-fun-and-profit-9b5f43a98130)设计模式，
pipeline模式使每个处理步骤更独立，更有利于重用，可以根据需要实现自己的pipeline

内置的标准的流程如图：

![GitHub](https://github.com/terrynoya/SmartBundleBuilder/raw/master/doc/pipeline.jpeg)

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
