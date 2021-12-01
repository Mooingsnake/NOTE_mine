## 情况还原（因为当时config感觉没调好，就一直追着config，但是其实是localization）
![image](https://user-images.githubusercontent.com/47411365/144073659-883f1c67-8954-4858-b182-54a7952c555f.png)

![image](https://user-images.githubusercontent.com/47411365/144073763-a92893c6-e22e-4888-ad5d-4bdae682e577.png)

来看报错信息：
Can not load dictionary 'Assets/GameMain/Localization/ChineseSimplified/Dictionaries/Default.xml' from 'Assets/GameMain/Localization/ChineseSimplified/Dictionaries/Default.xml' with error message 'GameFramework.GameFrameworkException: Load data failure in data provider helper, data asset name 'Assets/GameMain/Localization/ChineseSimplified/Dictionaries/Default.xml'.
  at GameFramework.DataProvider`1[T].LoadAssetSuccessCallback (System.String dataAssetName, System.Object dataAsset, System.Single duration, System.Object userData) [0x00027] in <4bcf7fea78174bc093803aefb9e9d8a0>:0 '.
UnityEngine.Debug:LogError (object)
UnityGameFramework.Runtime.DefaultLogHelper:Log

划重点： 

1.with error message 'GameFramework.GameFrameworkException: Load data failure in data provider helper, data asset name 

2.LoadAssetSuccessCallback

## 起源

结合1.2.搜索github可得报错点：https://github.com/EllanJiang/GameFramework/blob/2ce0bf15d8f29c270c5e266b18f0c4c1840e934e/GameFramework/Base/DataProvider/DataProvider.cs#L408

原因是
```
 if (!m_DataProviderHelper.ReadData(m_Owner, dataAssetName, dataAsset, userData))
```
返回为空

向上查看定义可得这是一个接口，说明是一个类似多态的情况：

![image](https://user-images.githubusercontent.com/47411365/144074677-76603f9a-9869-4024-bc4f-f150a20d1a20.png)

去搜索这个```IDataProviderHelper```

![image](https://user-images.githubusercontent.com/47411365/144074817-1d757acb-ea0d-4154-b27c-388ee9514af2.png)

只有这三个是继承了这个接口的，根据我们的报错信息来自config，所以我们去config里面找，不幸的是这也是个抽象类，里面什么也没有，还好reference里面还有提示：

![image](https://user-images.githubusercontent.com/47411365/144075147-e4e71fd2-955c-409d-b53d-c0952e339299.png)

https://github.com/EllanJiang/UnityGameFramework/blob/86bd9dc64f8502c2e726d11bdd10db639287fcd1/Scripts/Runtime/Config/DefaultConfigHelper.cs#L38

![image](https://user-images.githubusercontent.com/47411365/144075365-71e5ca45-64f8-4070-bd7b-3f3fcfea2ace.png)

因为没有warning，所以是成功运行到了这里的：

![image](https://user-images.githubusercontent.com/47411365/144075771-68e72489-c646-409a-a1cb-b85db66eba74.png)

=============================================我是未来的画图zy，我发现这里不严谨，其实不经过component=========

进configcomponent里面看看

![image](https://user-images.githubusercontent.com/47411365/144075850-9ce7d75f-4da7-4bae-acc9-c272a5a81a1f.png)

``` private IConfigManager m_ConfigManager = null;```

再问这个接口是谁实现，要找到``` ParseData(byte[] configBytes, object userData) ```

![image](https://user-images.githubusercontent.com/47411365/144077717-ccb20bf9-b74b-4cb2-92c5-d8f10fd22a00.png)

找到它的两个子类，其中ConfigHelperBase里面的ParseData的参数列表不对，pass

![image](https://user-images.githubusercontent.com/47411365/144077919-17193cab-b24c-4dad-a6dc-9c68fd4f64b1.png)

剩下一个：

![image](https://user-images.githubusercontent.com/47411365/144077979-9ba84ff1-9243-470e-b8db-988f729df6b1.png)

噔噔咚，来看看这个m_DataProvider是何方神圣

![image](https://user-images.githubusercontent.com/47411365/144078105-d859014c-54f8-4aa3-a471-ae38ea561359.png)

来分析一下这个语法吧，也就是说要回去找DataProvider。。。

![image](https://user-images.githubusercontent.com/47411365/144079457-7dec4acf-28f5-43f2-9da7-0db78402afb5.png)

一个不成熟程序员的崩溃可能就在一瞬间。。因为log界面根本没有其他提示，所以还是这句话：
``` return m_DataProviderHelper.ParseData(m_Owner, dataBytes, startIndex, length, userData);```

![image](https://user-images.githubusercontent.com/47411365/144080198-1e7c2069-9b8b-4b4e-92af-c5d665ad5f28.png)

老天一定是在惩罚我没有好好学习。。怎么会绕回来的。。。

画图吧，我麻了、

=======================================画了图来纠正了==========================

![image](https://user-images.githubusercontent.com/47411365/144084810-22c85431-dede-4885-82d4-34a1a109cf21.png)

其实来自IConfigManager，所以我们得搜索这个类，又出来两个

![image](https://user-images.githubusercontent.com/47411365/144085053-81071660-9b8f-4ec7-8700-9af199a737ae.png)

先忽略上面那个同样继承了但是好像继承的比较复杂的情况，看看下面那个

![image](https://user-images.githubusercontent.com/47411365/144085717-a83bf70b-fec2-454a-8291-06ffd3bd0591.png)

再上去看看声明和初始化：

![image](https://user-images.githubusercontent.com/47411365/144085828-70c35ca4-b6f8-42ec-96a6-a6e579b955a6.png)

what form of power is this？

（顺带如果刚刚搜索IConfigManager的时候往下看）

![image](https://user-images.githubusercontent.com/47411365/144086173-b9c52b50-eb02-4090-b454-31e758a0dd7f.png)

为什么可以继承带自己泛型的类啊！！！！！！

不过看清楚了，因为IConfigManager的基类是DataProvider<IConfigManager>，所以子类确实可以被赋值成
  
```
  m_DataProvider = new DataProvider<IConfigManager>(this);
```

然后我搜了一下，少见多怪了，其实这个写法C++里面也有 https://stackoverflow.com/questions/8336220/how-can-a-class-inherit-from-a-template-based-on-itself
  
确实按道理说不应该能够，但是这个写法保证了这句话可以过编译，它的名字叫做 Curiously Recurring Template Pattern, or CRTP for short。
  
  关于CRTP的东西这里不再赘述，你只要知道这是一种让基类可以痛快的使用派生类的成员函数的手段就可以了（为什么要这样？某种程度而言是因为多态的虚函数消耗太大了，你就当是新型多态（学名静态多态，因为在静态编译期就可以让基类使用自己的基类方法了。））
  
  https://github.com/EllanJiang/GameFramework/blob/2ce0bf15d8f29c270c5e266b18f0c4c1840e934e/GameFramework/Base/DataProvider/DataProvider.cs#L361
  
  这里很明显就是在调用自己的派生类的成员函数了，应该？
  
  来看崩溃现场
  
  来自DataProvider 
  
![image](https://user-images.githubusercontent.com/47411365/144102435-e6beb12e-4399-467f-9a40-a83ea11ac213.png)

  ![image](https://user-images.githubusercontent.com/47411365/144102539-60b831f9-e463-4205-801d-11b1cd4b5aed.png)

你妈你们两个是互相锁死了是吧，我连夜爬上崆峒山。
 
 要相信肯定还有我不知道的展开！直接开始搜ParseData！

  ![image](https://user-images.githubusercontent.com/47411365/144102720-5e6d33b2-734e-4f72-9999-c7d1c161c9b2.png)

 落泪了，好像
  
  ![image](https://user-images.githubusercontent.com/47411365/144103064-411fac63-c5b3-4850-a441-539841b0d3c7.png)
  
https://github.com/EllanJiang/UnityGameFramework/blob/b3f3d9eb818f93082346603a7cb871a84b72e53b/Scripts/Runtime/Config/DefaultConfigHelper.cs#L131



调用者应该是在DataProvider里面的这个：
  
  ![image](https://user-images.githubusercontent.com/47411365/144106160-3ea892fc-9f95-4a9c-a1eb-2e6847098c72.png)

  给一下链接吧https://github.com/EllanJiang/GameFramework/blob/2ce0bf15d8f29c270c5e266b18f0c4c1840e934e/GameFramework/Base/DataProvider/DataProvider.cs#L361

 owner的赋值也有意思：

![image](https://user-images.githubusercontent.com/47411365/144106302-5f07f346-f48a-4c87-a52d-40c9a0f7dc5e.png)

至此应该差不多了

  ====================第二日下午 ==== 还没完======================

按理说如果真的进入的是这里，那就应该有对应的语句报错，但是其实并没有。

其实我 __漏了一点__ 

![image](https://user-images.githubusercontent.com/47411365/144176187-04b84463-2dc0-4125-a955-a292d1654380.png)

 前几个warning也同样重要，搜索前面的warning信息，我才发现报错的是这一条：https://github.com/EllanJiang/UnityGameFramework/blob/b3f3d9eb818f93082346603a7cb871a84b72e53b/Scripts/Runtime/Config/DefaultConfigHelper.cs#L100

我们解析错了！我们解析到的string，后面的解析方法，是txt的简单格式，不是xml！
 
![image](https://user-images.githubusercontent.com/47411365/144176423-2a299138-574d-4161-8620-877dd0527720.png)
 
那么哪里可以设置解析器设置成自己的xml格式呢？
  
先确认一个概念，xxxManager，比如ConfigManager的，表示的是一个人大而全的管理器，它管辖所有部分，但不不特别实现具体功能。 xxxHelper，会有defaultxxxxHelper这种作者实现好的，可以解析txt和byte的具体实现，但是如果你想实现一个xml的解析器，也可以，甚至你只需要继承DefaultxxxHelper就行了。
 
![image](https://user-images.githubusercontent.com/47411365/144177553-54150978-d9c2-46e9-b1f5-ee4c1863765f.png)

  
重读强调一遍：__Manager是全能管理器，Helper可以只实现其中的五六个函数__  当然我们需要指定对应的解析器，我知道localization的，在编辑器界面
  
![image](https://user-images.githubusercontent.com/47411365/144177417-3586de36-8a18-4ad4-8463-165453f78acc.png)

 但是StarForce里面，config的解析器用的也是默认解析器，所以问题在于。。。？
  
  奇迹出现了！！！当我加上xml解析器然后勾选上以后，其他的报错也没有了！！！！果然是这个原因！！！
 
 ![image](https://user-images.githubusercontent.com/47411365/144179269-290eee8f-59e8-4df2-ade6-91b36123511b.png)
 
 啊，真好



