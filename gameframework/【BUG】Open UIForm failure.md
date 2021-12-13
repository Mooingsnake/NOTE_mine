## 报错信息
![image](https://user-images.githubusercontent.com/47411365/145720574-85e88604-4dd2-4e87-a7d5-dff7920ea82a.png)

最重要的就是开头这句话：
```
Open UI form failure, asset name 'StartMenu', UI group name 'Default', pause covered UI form 'True', error message 'Load UI form failure, asset name 'StartMenu', status 'NotExist', error message 'Asset name 'StartMenu' is invalid.'.'.
UnityEngine.Debug:LogWarning (object)
```

## 看源码
### 一，找出message的源头
感谢作者大大优秀的log打印技巧，让我能够找到出错的地方：

复制开头的```Load UI form failure, asset name``` 到GitHub上去:

![image](https://user-images.githubusercontent.com/47411365/145720729-bd04927c-c7f4-46fd-9c90-11263f347497.png)

好消息是只有唯一的报错信息：https://github.com/EllanJiang/GameFramework/blob/2ce0bf15d8f29c270c5e266b18f0c4c1840e934e/GameFramework/UI/UIManager.cs#L1015

我们可以看到这是一个在加载失败的时候发生的回调函数

### 二， 谁，在什么调用了这个函数？

![image](https://user-images.githubusercontent.com/47411365/145720900-ef231de5-8ef0-4ded-a047-a22cb9d50b4f.png)

因为报错内容明确告知了报错类型，所以我们可以通过这个信息快速到达报错地点：https://github.com/EllanJiang/GameFramework/blob/2ce0bf15d8f29c270c5e266b18f0c4c1840e934e/GameFramework/Resource/ResourceManager.ResourceLoader.cs#L315

### 三，找到车祸现场

```C#
           /// <summary>
            /// 异步加载资源。
            /// </summary>
            /// <param name="assetName">要加载资源的名称。</param>
            /// <param name="assetType">要加载资源的类型。</param>
            /// <param name="priority">加载资源的优先级。</param>
            /// <param name="loadAssetCallbacks">加载资源回调函数集。</param>
            /// <param name="userData">用户自定义数据。</param>
            public void LoadAsset(string assetName, Type assetType, int priority, LoadAssetCallbacks loadAssetCallbacks, object userData)
            {
                ResourceInfo resourceInfo = null;
                string[] dependencyAssetNames = null;
                if (!CheckAsset(assetName, out resourceInfo, out dependencyAssetNames))  // 就是这个函数返回错了
                {
                    string errorMessage = Utility.Text.Format("Can not load asset '{0}'.", assetName);
                    if (loadAssetCallbacks.LoadAssetFailureCallback != null)
                    {
                        loadAssetCallbacks.LoadAssetFailureCallback(assetName, resourceInfo != null && !resourceInfo.Ready ? LoadResourceStatus.NotReady : LoadResourceStatus.NotExist, errorMessage, userData);
                        return;
                    }

                    throw new GameFrameworkException(errorMessage);
                }

                if (resourceInfo.IsLoadFromBinary)
                {
                    string errorMessage = Utility.Text.Format("Can not load asset '{0}' which is a binary asset.", assetName);
                    if (loadAssetCallbacks.LoadAssetFailureCallback != null)
                    {
                        loadAssetCallbacks.LoadAssetFailureCallback(assetName, LoadResourceStatus.TypeError, errorMessage, userData);
                        return;
                    }

                    throw new GameFrameworkException(errorMessage);
                }
```

去看看checkAsset(和上文代码同文件夹)：
```
            private bool CheckAsset(string assetName, out ResourceInfo resourceInfo, out string[] dependencyAssetNames)
            {
                resourceInfo = null;
                dependencyAssetNames = null;

                if (string.IsNullOrEmpty(assetName))
                {
                    return false;  //不可能，assetName是有字的
                }

                AssetInfo assetInfo = m_ResourceManager.GetAssetInfo(assetName);
                if (assetInfo == null)
                {
                    return false;
                }

                resourceInfo = m_ResourceManager.GetResourceInfo(assetInfo.ResourceName);
                if (resourceInfo == null)
                {
                    return false;
                }

                dependencyAssetNames = assetInfo.GetDependencyAssetNames();
                return m_ResourceManager.m_ResourceMode == ResourceMode.UpdatableWhilePlaying ? true : resourceInfo.Ready;
            }
```

死胡同
### 四，从log着手再找新的原发地
![image](https://user-images.githubusercontent.com/47411365/145721962-b8f34372-ebcd-4392-99ec-4dffdd08878e.png)
来自这行代码：
```
GameFramework.UI.UIManager:LoadAssetFailureCallback (string,GameFramework.Resource.LoadResourceStatus,string,object)
UnityGameFramework.Runtime.EditorResourceComponent:LoadAsset (string,System.Type,int,GameFramework.Resource.LoadAssetCallbacks,object) (at Assets/GameFramework/Scripts/Runtime/Resource/EditorResourceComponent.cs:1036)
```

```c#
            if (!assetName.StartsWith("Assets/", StringComparison.Ordinal))
            {
                if (loadAssetCallbacks.LoadAssetFailureCallback != null)
                {
                    loadAssetCallbacks.LoadAssetFailureCallback(assetName, LoadResourceStatus.NotExist, Utility.Text.Format("Asset name '{0}' is invalid.", assetName), userData);
                }

                return;
            }
```
https://github.com/EllanJiang/UnityGameFramework/blob/b2d2ef63517d2cab5f0def57e691a2d794b6a7f5/Scripts/Runtime/Resource/ResourceComponent.cs#L1244

这里就可以看清楚了，我们缺了Assets/ ，回去一看果然是UIextension.cs这里面没有好好加上路径名
