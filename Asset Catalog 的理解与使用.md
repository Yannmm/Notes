# Asset Catalog 的理解与使用



> asset catalog 是 iOS app 开发必不可少的组成部分。我们一般将图片存放其中，但殊不知，其用途远不止于此。



# ![图1](https://ws2.sinaimg.cn/large/006tNbRwly1fwklrh4p92j307g0bdtca.jpg)

---

### 概述

资源目录（asset catalog）是 Xcode  提供的项目资源管理工具，其`核心理念`在于：以设备特征（[traits](https://help.apple.com/xcode/mac/current/#/devb19ffeb37)）为单位配置资源，包括但不限于图片，颜色，材质，数据。既让开发者免于代码配置资源的烦恼，也让苹果能够更好的控制 .ipa 包。例如，苹果在 iOS 9 支持 [on-demand resource](https://help.apple.com/xcode/mac/current/#/devaf48822a6) 和 [slicing](https://help.apple.com/xcode/mac/current/#/dev2b279877a)，前者意味着将资源和二进制代码彻底分离；后者以根据具体设备打包资源和二进制代码，减少 .ipa 体积。

资源集（asset set）是聚合资源（asset）的基本单位，一个或多个资源构成一个资源集。资源集分为若干类型，取决于资源的具体类型（[asset types](https://help.apple.com/xcode/mac/current/#/dev70f9ac19b)）。此外还可以建立子文件夹，进一步聚合资源集。

asset catalog 本质是一个文件目录。经测试，无论使用几个资源目录，打包后总以单个 `Asset.car` 文件的形式存在。通过命令 `xcrun --sdk iphoneos assetutil --info Assets.car` 进行验证，获取目录内所有资源集信息，例如：



```shell
xcrun --sdk iphoneos assetutil --info Assets.car
[
  {
    "AssetStorageVersion" : "IBCocoaTouchImageCatalogTool-9.4.1",
    "Authoring Tool" : "@(#)PROGRAM:CoreThemeDefinition  PROJECT:CoreThemeDefinition-339.7\n",
    "CoreUIVersion" : 493,
    "DumpToolVersion" : 493.39999999999998,
    "Key Format" : [
      "kCRThemeScaleName",
      "kCRThemeIdiomName",
      "kCRThemeSubtypeName",
      "kCRThemeDeploymentTargetName",
      "kCRThemeGraphicsClassName",
      "kCRThemeMemoryClassName",
      "kCRThemeDisplayGamutName",
      "kCRThemeDirectionName",
      "kCRThemeSizeClassHorizontalName",
      "kCRThemeSizeClassVerticalName",
      "kCRThemeIdentifierName",
      "kCRThemeElementName",
      "kCRThemePartName",
      "kCRThemeStateName",
      "kCRThemeValueName",
      "kCRThemeDimension1Name",
      "kCRThemeDimension2Name"
    ],
    "MainVersion" : "@(#)PROGRAM:CoreUI  PROJECT:CoreUI-493.4\n",
    "Platform" : "ios",
    "PlatformVersion" : "10.0",
    "SchemaVersion" : 2,
    "StorageVersion" : 14,
    "ThinningParameters" : "optimized <idiom 1> <subtype 569> <scale 2> <gamut 1> <graphics 4> <graphicsfallback (3,2,1,0)> <memory 2> <deployment *> <hostedIdioms (4,5)>"
  },
  {
    "AssetType" : "Data",
    "Compression" : "uncompressed",
    "Data Length" : 27,
    "Idiom" : "universal",
    "Name" : "sample-data",
    "Scale" : 1,
    "SizeOnDisk" : 251,
    "UTI" : "UTI-Unknown"
  },
  {
    "AssetType" : "MultiSized Image",
    "Idiom" : "phone",
    "Name" : "AppIcon",
    "Scale" : 1,
    "Sizes" : [
      "20x20 index:0 idiom:phone",
      "29x29 index:1 idiom:phone",
      "40x40 index:2 idiom:phone",
      "57x57 index:4 idiom:phone",
      "60x60 index:5 idiom:phone"
    ]
  },
  {
    "AssetType" : "Image",
    "BitsPerComponent" : 8,
    "ColorModel" : "RGB",
    "Encoding" : "JPEG",
    "Idiom" : "universal",
    "Image Type" : "kCoreThemeOnePartScale",
    "Name" : "Bonsai",
    "Opaque" : true,
    "PixelHeight" : 1024,
    "PixelWidth" : 1024,
    "RenditionName" : "Bonsai.jpg",
    "Scale" : 3,
    "SizeOnDisk" : 330548
  }
]
```



### 使用

##### App Icon

iOS 项目创建时附带一个名为 `Assets.xcassets` 的资源目录，包含一个名为 AppIcon 的空资源集，类型为 `iOS App Icon`。



![图3](https://ws2.sinaimg.cn/large/006tNbRwly1fwkr4be7sjj30ty0u4qe6.jpg)



显然，这个类型的资源集是用来管理 app 图标的。首先，打开右侧的 attribute inspector 查看当前配置：本资源集适用于 iPhone 和 iPad 设备。

中间区域罗列了所需资源，每个插槽代表一个，需要手动拖入，下方文字说明用途。例如，iPhone Notificaiton 指通知栏的 app 图标；iPhone App iOS 7-11 60pt 表示需要大小为60pt的图标，以便在用户桌面显示 app。



##### Launch Image

苹果推荐使用 .xib 制作启动界面，但同时仍允许我们静态图片。首先建立类型为  `iOS Launch Image` 的资源集；然后从设备类型，iOS版本等方面配置；最后填满所有插槽。



![图添加launchimage](https://ws4.sinaimg.cn/large/006tNbRwly1fwle1bcq5jj30tv0f7q7o.jpg)



从上图不难看出，这组启动图支持 iOS 7.0 及以后的竖屏 iPhone 设备。



##### Image



![图imageassetset](https://ws1.sinaimg.cn/large/006tNbRwly1fwlezheu3nj30760ut79q.jpg)

图片素材管理是我们最常用的功能。将一个或一组图片拖入 asset catalog，Xcode 会根据名称创建资源集。查看右侧配置面板，有许多选项，每组选项代表一个设备特征。通过这种方式，达到因材施教的目的。

这里着重讲一下 Slicing，即图片切分。asset catalog 允许我们通过所见即所得的方式切分图片，无需编写代码。具体操作如下：

1. 选中一个图片素材资源集，点击中央区域最下方的 Show Slicing；
2. 点击图片上的 Start Slicing，开始切分操作；
3. 切分方式有三种：水平切分（horizontal），垂直切分（vertical），水平 + 垂直；
4. 以水平切分为例，一开始有三条竖线，将整个图片分为四部分，从左到右依次为：
   - 左1：左端点（end cap），水平方向保持不变；
   - 左2：水平形变区域；
   - 左3：水平固定区域；
   - 左4：右端点（end cap），水平方向保持不变；
5. 调整三条线的位置，确定各区域的大小。



此外，还可以在配置面板选择具体形变方式：拉伸 / 平铺。



##### data

二进制数据作为资源的一种，也可以通过 asset catalog 管理！通过配置面板中的 memory 一栏，根据设备内存使用不同数据。



![图dataset](https://ws2.sinaimg.cn/large/006tNbRwly1fwlg1xm83ij30tx0iwgqi.jpg)



###  参考

- 示例 app：[NCCBF-iOS](https://github.com/keitaito/NCCBF-iOS)

- [Xcode Help - WORK WITH ASSETS](https://help.apple.com/xcode/mac/current/#/dev10510b1f7)

- [Analysing Assets.car file in iOS](https://stackoverflow.com/a/44597439)
