## 定论
委托就是一个函数指针，我们可以在C++里面找到对应的部分，是lambda和函数指针：
```
    sort(arr.begin(), arr.end(), [](int a, int b){return a>b;})
```
![image](https://user-images.githubusercontent.com/47411365/140749624-1b7ca5e5-5eea-4d76-9f67-05afde3b7cef.png)

## 基本用法
微软官方的说法是调用者自己也提供了一部分的算法实现，下面是bilibili的视频：

From：https://www.bilibili.com/video/BV1t3411176r/?spm_id_from=333.788.recommend_more_video.0

![image](https://user-images.githubusercontent.com/47411365/140749969-d11b015a-9585-4f5e-bfa2-607cd214342d.png)

这张图就能看到，函数Foo和Bar被当成了变量了。

这就像一个基类，声明的时候保证了基本特征（参数和返回值一致），那么像C++的多态一样是可以用爹的名义做儿子的事（？

我们甚至可以把变量的特征应用到函数参数上去：

![image](https://user-images.githubusercontent.com/47411365/140752455-94d5e9ff-08d5-41e8-a0ee-ce36b5ffe3cb.png)

语义上和上文一样。

我们来简化这里的代码：

![image](https://user-images.githubusercontent.com/47411365/140888005-bc8ae699-48c4-4823-868d-e59c3b2d26e3.png)

1号和2号高度相似，我们应该怎么才能重用呢？

![image](https://user-images.githubusercontent.com/47411365/140888050-260f5180-9be6-48cc-aa83-ec33bdb5be7b.png)

我的感觉是，委托的那个类更像是一个代表函数指针的槽，声明了以后就可以放在函数列表里面，然后可以拿同类（返回值和参数列表相同就算同类）的lambda填入

![image](https://user-images.githubusercontent.com/47411365/140888091-54511cca-74af-40a3-9dbd-fc38c80fa28a.png)

当然C++的lambda代表一个函数指针

C#的lambda代表一个委托。

C++ 用void (* f)(int a, int b);表示的函数指针，在C#里是用delegate写的，同样要列出对应的返回值和参数列表。

![image](https://user-images.githubusercontent.com/47411365/140891773-e8ef3192-e038-48e1-8d68-40854d6acd9d.png)

## 具体使用（gameframework）
https://github.com/EllanJiang/GameFramework/search?q=LoadAssetSuccessCallback

我们来看gf是怎么实现资产的加载回调函数的

定义：
```
namespace GameFramework.Resource
{
    /// <summary>
    /// 加载资源成功回调函数。
    /// </summary>
    /// <param name="assetName">要加载的资源名称。</param>
    /// <param name="asset">已加载的资源。</param>
    /// <param name="duration">加载持续时间。</param>
    /// <param name="userData">用户自定义数据。</param>
    public delegate void LoadAssetSuccessCallback(string assetName, object asset, float duration, object userData);
}
```

集成了一个函数集：
```
 public sealed class LoadAssetCallbacks
    {
        private readonly LoadAssetSuccessCallback m_LoadAssetSuccessCallback;
        private readonly LoadAssetFailureCallback m_LoadAssetFailureCallback;
        private readonly LoadAssetUpdateCallback m_LoadAssetUpdateCallback;
        private readonly LoadAssetDependencyAssetCallback m_LoadAssetDependencyAssetCallback;

        /// <summary>
        /// 初始化加载资源回调函数集的新实例。
        /// </summary>
        /// <param name="loadAssetSuccessCallback">加载资源成功回调函数。</param>
        public LoadAssetCallbacks(LoadAssetSuccessCallback loadAssetSuccessCallback)
            : this(loadAssetSuccessCallback, null, null, null)
        {
        }

```

在需要的地方编写函数的实现方式（ internal sealed partial class EntityManager ）：
```
        ...
        
        private readonly LoadAssetCallbacks m_LoadAssetCallbacks;
        
        ...
        
        public EntityManager()
        {
           ...
            m_LoadAssetCallbacks = new LoadAssetCallbacks(LoadAssetSuccessCallback, LoadAssetFailureCallback, LoadAssetUpdateCallback, LoadAssetDependencyAssetCallback);
           ...
        }
        
        ...
        
        private void LoadAssetSuccessCallback(string entityAssetName, object entityAsset, float duration, object userData)
        {
            ShowEntityInfo showEntityInfo = (ShowEntityInfo)userData;
            if (showEntityInfo == null)
            {
                throw new GameFrameworkException("Show entity info is invalid.");
            }

            if (m_EntitiesToReleaseOnLoad.Contains(showEntityInfo.SerialId))
            {
                m_EntitiesToReleaseOnLoad.Remove(showEntityInfo.SerialId);
                ReferencePool.Release(showEntityInfo);
                m_EntityHelper.ReleaseEntity(entityAsset, null);
                return;
            }

            m_EntitiesBeingLoaded.Remove(showEntityInfo.EntityId);
            EntityInstanceObject entityInstanceObject = EntityInstanceObject.Create(entityAssetName, entityAsset, m_EntityHelper.InstantiateEntity(entityAsset), m_EntityHelper);
            showEntityInfo.EntityGroup.RegisterEntityInstanceObject(entityInstanceObject, true);

            InternalShowEntity(showEntityInfo.EntityId, entityAssetName, showEntityInfo.EntityGroup, entityInstanceObject.Target, true, duration, showEntityInfo.UserData);
            ReferencePool.Release(showEntityInfo);
        }
```

## 心跳写的
cfg是luban导表工具生成的读表函数，ByteBuf也是luban导表工具的库函数提供的

https://github.com/cnImpulse/AGame/blob/2c3e12d2dd8023e4256a183f150fce392287c10e/Assets/GameMain/Scripts/Cfg/CfgComponent.cs

```
       public void LoadTables()
        {
            for (int i = 0; i < Cfg.Tables.Assets.Length; ++i)
            {
                GameEntry.Resource.LoadAsset($"Assets/GameMain/GameData/CfgData/{Cfg.Tables.Assets[i]}.bytes",
                    new LoadAssetCallbacks(OnAssetLoadScuess));
            }
        }
        
        ···
        
      public void OnAssetLoadScuess(string assetName, object asset, float duration, object userData)
        {
            TextAsset textAsset = asset as TextAsset;
            ByteBufList.Add(textAsset.name, new ByteBuf(textAsset.bytes));

            if (!EndLoad && ByteBufList.Count == Cfg.Tables.Assets.Length)
            {
                Tables = new Cfg.Tables(LoadByteBuf);  --这里也是函数指针参数，原定义是Func<string,ByteBuf>,目测是为了方便表示，和委托性质差不多，一环套一环了属于是
                EndLoad = true;
            }
        }
        
     private ByteBuf LoadByteBuf(string file)
        {
            return ByteBufList[file];
        }
```
关于string为什么这么写，走这里：https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated

