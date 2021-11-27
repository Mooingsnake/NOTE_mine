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
