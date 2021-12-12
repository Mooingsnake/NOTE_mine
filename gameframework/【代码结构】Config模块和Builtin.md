## Config和Builtin的业务逻辑区别
全局配置 (Config) - 存储一些全局的只读的游戏配置，如玩家初始速度、游戏初始音量等。    ——这是官方的原话。

我在学习的时候发现官方的框架提供了Config的组件，但是自己又实现了一个类似的“BuiltinData”组件，来看看二者的区别：

Config 主要设置了不可变的配置信息（指scene的文件地址和unity build的编号），比如：

![image](https://user-images.githubusercontent.com/47411365/143689680-8f65c214-d2ea-4a82-a8a6-fedee2a279c2.png)


BuiltinData主要设置了可以变的信息（图一在ProcedureLaunch.cs里面，图二在自己实现的BuiltinDataComponent.cs里面），比如：

![image](https://user-images.githubusercontent.com/47411365/143689768-1be7939d-8138-4348-b2cc-bb77adf58355.png)

![image](https://user-images.githubusercontent.com/47411365/143690355-dd13550c-c628-4f4e-939e-d04ede88eefe.png)


## 为什么会有这样的差别
这两个真的有差别吗？我可以在config里面放可变数据吗？

config实际上存储的是一个字典表：

![image](https://user-images.githubusercontent.com/47411365/143691530-9e522008-f171-42a6-a8f6-45e87802da0c.png)


实际上，Config里面读取配置信息的时候调用的也是DataProvider ，也就是DataTableComponent的套路。

![image](https://user-images.githubusercontent.com/47411365/143691115-7ae143d1-176c-4b34-8bac-0ccc7db97dbf.png)

from:ConfigManager.cs

因为走了框架所以需要随时监听有没有加载完成：

![image](https://user-images.githubusercontent.com/47411365/143691472-f002a2f5-ec38-4c4d-af7b-b854dfe3c87c.png)


===================分割线=========================

BuiltinData里面的InitDictionary方法则是需要解析XML文件，所以还需要接入一个xml解析器：

![image](https://user-images.githubusercontent.com/47411365/143691208-c602c72d-de77-46e5-980b-1cf76ea5f74d.png)

并没有通过gf的资源加载方法，而是使用了微软给C#的自带xml解析器：

![image](https://user-images.githubusercontent.com/47411365/143691310-884c7d23-3a67-4dca-8d32-da18de84d2ab.png)

xmlDocument就是官方提供的xml对象，xmlDocument.LoadXml(dictionaryString); 这句就是系统的加载函数，所以不存在异步，可以放心使用。

因为加载没走框架库，所以在ProcedureLaunch.cs没有提到任何订阅.Event.Subcribe()这样的函数。（__下图打脸现场__）

![image](https://user-images.githubusercontent.com/47411365/143766959-fb635068-791f-459c-a2c8-3b95884210c9.png)


================？？？============

等下，为什么这里还有Dictionary的事件监听?如果是全部自己搞的那就不可能呀！

![image](https://user-images.githubusercontent.com/47411365/143691726-085b4f2e-5a1f-4f25-a5c5-7ff4114d7c49.png)

首先可以肯定的是Dictionary也来自最普通的数据读取，我们需要知道它哪里调用了这个东西。

然后我搜了一下，貌似并没有用XMLHelper里面的那个函数，或者说整个这一整个类都没有用到，用的还是LocalizationComponent里面的内容。而这个组件里面的确有关于字典加载成功事件的注册：

![image](https://user-images.githubusercontent.com/47411365/143765903-6ff0e38e-98bf-48f6-a0f5-bccb8525f817.png)

需要注意的是github里面BuiltinDataComponent里面的ParseData函数跳转到了XmlLocalizationHelper，但是其实不是这个函数，需要做一个区分。
