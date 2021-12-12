## Action \<T>

msdn :https://docs.microsoft.com/en-us/dotnet/api/system.action-1?redirectedfrom=MSDN&view=net-6.0

 ### 定义

封装了一个方法，只有一个参数并且不返回值 

（Encapsulates a method that has a single parameter and does not return a value.） 

``` C#

public delegate void Action<in T>(T obj); 

``` 

 

T ：这个方法的参数类型,这个类型向上（抽象）兼容，也就是说可以使用T的基类（The type of the parameter of the method that this delegate encapsulates. 

（This type parameter is contravariant. That is, you can use either the type you specified or any type that is less derived. For more information about covariance and contravariance, see Covariance and Contravariance in Generics.） 


T obj ： obj就是这个函数的参数 


### 例子 

下面我们使用了Action<T>委托去打印List<T>.在这个例子里面Print函数用来展示打印list值。另外，C#例子演示了匿名函数如何打印内容。注意这个例子里面没有显式的声明Action<T>变量。它对函数传递了一个引用（这个引用携带了一个参数）并且不返回值给List<T>.ForEach()函数，重点来了，FOrEach()函数的参数就是这个Action<T>委托，这个委托变量也没有显式的实例化，但是这个匿名函数就对应了这个委托了 

``` c#

List<string> names = new List<string>(); 

names.Add("Bruce"); 

names.Add("Alfred"); 

names.Add("Tim"); 

names.Add("Richard"); 

  

// Display the contents of the list using the Print method. 

names.ForEach(Print);  

  

// The following demonstrates the anonymous method feature of C# 

// to display the contents of the list to the console. 

names.ForEach(delegate(string name)   ///就这里

{ 

    Console.WriteLine(name); 

}); 

  

void Print(string s) 

{ 

    Console.WriteLine(s); 

} 

  

/* This code will produce output similar to the following: 

* Bruce 

* Alfred 

* Tim 

* Richard 

* Bruce 

* Alfred 

* Tim 

* Richard 

*/ 

``` 

人话解释：delegate(string name){ xxxxx;}这里就是匿名函数，像个槽一样，下次就可以在ForEach()里面传无返回值，string参数类型的函数了 

 

我们可以先显式申明再赋值： 

``` c#

using System; 

using System.Windows.Forms; 

  

public class TestAction1 

{ 

   public static void Main() 

   { 

      Action<string> messageTarget; ///

  

      if (Environment.GetCommandLineArgs().Length > 1) 

         messageTarget = ShowWindowsMessage; ///

      else 

         messageTarget = Console.WriteLine; 

  

      messageTarget("Hello, World!"); 

   } 

  

   private static void ShowWindowsMessage(string message) ///

   { 

      MessageBox.Show(message); 

   } 

} 

``` 

 

我们可以显式声明以后用匿名函数： 

``` c#

using System; 

using System.Windows.Forms; 

  

public class TestAnonMethod 

{ 

   public static void Main() 

   { 

      Action<string> messageTarget;    ///

  

      if (Environment.GetCommandLineArgs().Length > 1) 

         messageTarget = delegate(string s) { ShowWindowsMessage(s); }; ///

      else 

         messageTarget = delegate(string s) { Console.WriteLine(s); }; ///

  

      messageTarget("Hello, World!"); 

   } 

  

   private static void ShowWindowsMessage(string message) 

   { 

      MessageBox.Show(message); 

   } 

} 

``` 

 

匿名函数的懒鬼lambda写法： 

``` c#

messageTarget = s => ShowWindowsMessage(s); 

``` 

 

 

 

来看具体例子(来自三个不同的文件)： 

框架内部 

``` c#

public delegate void GameFrameworkAction<in T>(T obj); 

``` 

 

DialogParams.cs  （作为一个类的成员变量了，也就是property） 

``` c#

public  GameFrameworkAction<object> OnClickConfirm { get; internal set; } 

``` 

 

MenuForm.cs  from：[GitHub地址](https://github.com/EllanJiang/StarForce/blob/d4ce803353f89852c4d58df65b2482a3b08eb11f/Assets/GameMain/Scripts/UI/MenuForm.cs#L37)
 

``` c#

OnClickConfirm = delegate(object userData) { UnityGameFramework.Runtime.GameEntry.Shutdown(ShutdownType.Quit); } 


``` 
