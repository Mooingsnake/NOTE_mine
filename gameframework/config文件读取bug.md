## 情况还原
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









