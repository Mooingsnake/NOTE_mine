## 简介
Luban，一个导表工具，在配置完成后可以基于几张excel数据表生成.json/.bytes 格式的数据表，同时可以同步生成能解析该表的对应C#代码文件，功能演示如下：

基于的策划编写的excel表格：
![image](https://user-images.githubusercontent.com/47411365/140576832-a4513a6e-07a9-412b-9207-e85e95e908f2.png)


对应生成导出的Json格式的数据表：
![image](https://user-images.githubusercontent.com/47411365/140576836-2abe77a8-c719-4f46-8421-91ec1436e06d.png)

能够对应解析这个json的 C# 的代码：
![image](https://user-images.githubusercontent.com/47411365/140576851-af6fb9a1-da40-42b5-a0e2-d38ff76f42ed.png)
![image](https://user-images.githubusercontent.com/47411365/140576863-e911305e-e575-4f8b-a80f-adeb1a17e80d.png)

## 手把手教你看懂官网配置
配置可以去看官网：
https://github.com/focus-creative-games/luban/blob/main/docs/start_up.md

简单解释一下这个：
![image](https://user-images.githubusercontent.com/47411365/140576874-419279bc-6212-4f0f-804c-9d15aa2be7c7.png)

我成功的样子：
![image](https://user-images.githubusercontent.com/47411365/140576882-8f4b41eb-c135-4341-9c61-836ab8533e4d.png)


逐行解释，这是个命令行脚本，dotnet是这个脚本的运行环境，需要事先下载dotnet并且安装以后（类似java这种，不过这个很方便），之后就可以用dotnet作为开头输入参数。
第一行是set 参数，可以看到 dotnet后面接了我的Luban.ClientServer.dll
第二行是set参数替换，这里的-j应该是鲁班的参数，我没在dotnet里面看到

需要编写的部分可以看提示，这里再贴一下DOS的文件路径基本法：
路径	描述
C:\Documents\Newsletters\Summer2018.pdf	  -------------- C: 驱动器的根目录中的绝对文件路径。
\Program Files\Custom Utilities\StringFinder.exe  -----	当前驱动器根路径上的绝对路径。
2018\January.xlsx	  ----------------------------------- 指向当前目录的子目录中的文件的相对路径。
..\Publications\TravelBrochure.pdf	  ----------------  指向当前目录的同级目录中的文件的相对路径。
C:\Projects\apilibrary\apilibrary.sln  ----------------	C: 驱动器的根目录中的文件的绝对路径。
C:Projects\apilibrary\apilibrary.sln	----------------- C: 驱动器的当前目录中的相对路径。
From：https://docs.microsoft.com/zh-cn/dotnet/standard/io/file-path-formats


需要注意的事中文会乱码（但是不影响读到物品表），不过我这里的问题主要是路径名写成文件名：
![image](https://user-images.githubusercontent.com/47411365/140577043-73e4426f-b034-41a7-90b0-d258640d2d64.png)

值得注意的是我使用的是gameframework，惯用的文件存储格式是.txt和bytes,但是这里只有json和bytes

## 修改成二进制
实际上一般而言，游戏通用的存储方式是二进制，所以我们更多时候用的是bytes格式的导出方式（bin格式也是二进制存储，但是unity不认bin文件，所以这里务必检查是用bytes），别的不需要动，只需要在bat文件里修改--gen_type 并且加上后面一行--data_file_extension bytes ^就行：
![image](https://user-images.githubusercontent.com/47411365/140577053-b3b2f0ed-f0a2-4c19-84bc-0daaca385be1.png)


## 在项目文件中应当包含以下内容：

1.解析byte的库函数：（你可以在每一个示例project的Assets/LubanLib目录下面找到这几个文件）
![image](https://user-images.githubusercontent.com/47411365/140577067-1d2a2ab5-c174-481c-bf68-ca268b66e51c.png)

2.导出的cs文件和bytes文件

![image](https://user-images.githubusercontent.com/47411365/140577088-c379e2f8-9972-448f-9374-7b6200ba9ef2.png)
![image](https://user-images.githubusercontent.com/47411365/140577095-8a0c11c4-ba6a-4dd6-8080-45e9548e989f.png)
![image](https://user-images.githubusercontent.com/47411365/140577102-ab822108-62c6-4769-9a2a-ee9c9f1b8e3f.png)
![image](https://user-images.githubusercontent.com/47411365/140577112-1ae49a41-a084-474a-b720-fbb5c37ca97a.png)




没了。不需要把luban的.dll的那个目录拿进来，不要把check.bat拿进来，不要把__root__.xml拿进来，这都是是你的代码生成工具，不是你的库函数。

## 产出的数据文件，脚本文件分别对应哪几张原始excel表？这些表格应该怎么写？

### 来看__root__.xml:

![image](https://user-images.githubusercontent.com/47411365/140577122-a5d8f4ca-19ef-420c-b01a-99b7d29f5957.png)

Bat文件首先依据__root__.xml去寻找这三个xlsx文件

### 其中__tables__.xlsx文件内容如下：

![image](https://user-images.githubusercontent.com/47411365/140577163-f0717ead-f4d3-41f6-8470-19c73af60a69.png)


Full_name:这是其中一个生成的cs解析数据的文件，包含模块和名字，指的是demo是文件夹，TbItem是cs文件名，意思是tableItem
Value_type：一个实体类，代表其中另外一个生成的cs解析数据的文件，规定和解析了Item中含有Id值，Name值,price值，desc值等等

Input：这个input文件直接决定了你的生成文件，像我这里就是错的，我这里如果用的是equips.xlsx

![image](https://user-images.githubusercontent.com/47411365/140577174-95b3edb7-882e-42e9-b359-e4a2cceda892.png)


那么就会这样生成你的Item.cs（TbItem.cs同理）

![image](https://user-images.githubusercontent.com/47411365/140577191-03404d31-7598-4e2c-9de2-5659828382ee.png)


实际上input应该是填item.xlsx才能正确映射


Table.xlsx和它生成的cs文件和bytes文件能解决表的数据集合方式，也就是我们常用的数据库，比如有一个学生表，每一条都是一个学生，有很多学生的集合。

生成这两个文件后，会有一个总的Tables，每个table都会被整合到这个类里面，堪称数据库的级别。

![image](https://user-images.githubusercontent.com/47411365/140577215-2fcd806c-bd02-493d-8a84-26018d38a64a.png)



### __enum__.xlsx情况如下：

![image](https://user-images.githubusercontent.com/47411365/140577231-9d356b9b-f95e-42ae-848b-f868a62795ea.png)

同样全名是文件夹名+.cs文件名，来看它生成的cs文件：

![image](https://user-images.githubusercontent.com/47411365/140577258-359e9852-0d35-406d-b43a-84390fae58f0.png)


适合作为一个全局的工具枚举集合，比如不同的职业当作enum，不同的等级（比如稀有度这种）


### __beans.cs__文件

![image](https://user-images.githubusercontent.com/47411365/140577270-8798e1c2-01e4-4078-96e1-50a9aaac0b8a.png)


很怪的是没有生成fullname，这个beans文件好像没有被用到


## 如果你想修改namespace的话也可以，在__root__.xml里面：


![image](https://user-images.githubusercontent.com/47411365/140577280-462a34d1-8e69-48d4-a7cc-95bdf99140df.png)

![image](https://user-images.githubusercontent.com/47411365/140577291-5d0037a8-1c4b-43af-9af2-829e7aa34976.png)



