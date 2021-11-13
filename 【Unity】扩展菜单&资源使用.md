[太长不看版](#taichang)
## 扩展菜单基本法
from：https://docs.unity3d.com/ScriptReference/MenuItem.html
```
using UnityEditor;  // 需要引用的库
using UnityEngine;
public class MenuTest : MonoBehaviour   // 不知道为什么需要MonoBehaviour，我自己不用也是可以的
{
    // Add a menu item named "Do Something" to MyMenu in the menu bar.
    [MenuItem("MyMenu/Do Something")]  // 菜单的名字和路径
    static void DoSomething()
    {
        Debug.Log("Doing Something..."); // 要做的事情
    }

    // Validated menu item.
    // Add a menu item named "Log Selected Transform Name" to MyMenu in the menu bar.
    // We use a second function to validate the menu item 
    // so it will only be enabled if we have a transform selected.
    [MenuItem("MyMenu/Log Selected Transform Name")]
    static void LogSelectedTransformName()
    {
        Debug.Log("Selected Transform is on " + Selection.activeTransform.gameObject.name + ".");
    }

    // Validate the menu item defined by the function above.
    // The menu item will be disabled if this function returns false.
    [MenuItem("MyMenu/Log Selected Transform Name", true)]
    static bool ValidateLogSelectedTransformName()
    {
        // Return false if no transform is selected.
        return Selection.activeTransform != null;
    }

    // Add a menu item named "Do Something with a Shortcut Key" to MyMenu in the menu bar
    // and give it a shortcut (ctrl-g on Windows, cmd-g on macOS).
    [MenuItem("MyMenu/Do Something with a Shortcut Key %g")]
    static void DoSomethingWithAShortcutKey()
    {
        Debug.Log("Doing something with a Shortcut Key...");
    }

    // Add a menu item called "Double Mass" to a Rigidbody's context menu.
    [MenuItem("CONTEXT/Rigidbody/Double Mass")]
    static void DoubleMass(MenuCommand command)
    {
        Rigidbody body = (Rigidbody)command.context;
        body.mass = body.mass * 2;
        Debug.Log("Doubled Rigidbody's Mass to " + body.mass + " from Context Menu.");
    }

    // Add a menu item to create custom GameObjects.
    // Priority 1 ensures it is grouped with the other menu items of the same kind
    // and propagated to the hierarchy dropdown and hierarchy context menus.
    [MenuItem("GameObject/MyCategory/Custom Game Object", false, 10)]
    static void CreateCustomGameObject(MenuCommand menuCommand)
    {
        // Create a custom game object
        GameObject go = new GameObject("Custom Game Object");
        // Ensure it gets reparented if this was a context click (otherwise does nothing)
        GameObjectUtility.SetParentAndAlign(go, menuCommand.context as GameObject);
        // Register the creation in the undo system
        Undo.RegisterCreatedObjectUndo(go, "Create " + go.name);
        Selection.activeObject = go;
    }
}
```
## 可以对场景-资源Asset 进行操作

关于Resource的解释：https://docs.unity3d.com/ScriptReference/Resources.html
```
using UnityEngine;
using System.Collections;

public class ExampleClass : MonoBehaviour
{
    void Start()
    {
        GameObject go = GameObject.CreatePrimitive(PrimitiveType.Plane);
        Renderer rend = go.GetComponent<Renderer>();
        rend.material.mainTexture = Resources.Load("glass") as Texture; // 这里就是读取资源。有时候前面的类型定义太长可以直接用object

    }
    
}
```

下面是常用的路径表示和类型总结
```
        ⛎这个函数只允许寻找Asset/Resource目录下面的文件，我们一般不用这个函数，而且这玩意不需要后缀
        来看文件路径格式
        //Load a text file (Assets/Resources/Text/textFile01.txt)   // ⬛就所有数据类型都是textAsset，.bytes也是
        var textFile = Resources.Load<TextAsset>("Text/textFile01");  

        //Load text from a JSON file (Assets/Resources/Text/jsonFile01.json)  //⬛ json也归属于textAsset
        var jsonTextFile = Resources.Load<TextAsset>("Text/jsonFile01");
        //Then use JsonUtility.FromJson<T>() to deserialize jsonTextFile into an object

        //Load a Texture (Assets/Resources/Textures/texture01.png)
        var texture = Resources.Load<Texture2D>("Textures/texture01");  //⬛ 图片是Texture2D

        //Load a Sprite (Assets/Resources/Sprites/sprite01.png)
        var sprite = Resources.Load<Sprite>("Sprites/sprite01");  // ⬛sprite 是Sprite
  

        //Load an AudioClip (Assets/Resources/Audio/audioClip01.mp3)  //⬛ 音频文件是 AudioClip
        var audioClip = Resources.Load<AudioClip>("Audio/audioClip01");
```



## 太长不看版
<span id="taichang"></span>
如果Resource.Load不好用，可以用这个：(from:https://docs.unity3d.com/ScriptReference/AssetDatabase.LoadAssetAtPath.html)
```
using UnityEngine;
using UnityEditor;

public class MyPlayer : MonoBehaviour
{
    [MenuItem("AssetDatabase/LoadAssetExample")]
    static void ImportExample()
    {
        Texture2D t = (Texture2D)AssetDatabase.LoadAssetAtPath("Assets/Textures/texture.jpg", typeof(Texture2D));
    }
}
```
