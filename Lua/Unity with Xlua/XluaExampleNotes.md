# Xlua Examples 学习

## 01_HellowWorld
> 主要看 new LuaEnv() 里面具体干了什么

```csharp
void Start () {
    LuaEnv luaenv = new LuaEnv(); //新建一个lua虚拟机
    luaenv.DoString("CS.UnityEngine.Debug.Log('hello world')");
    luaenv.Dispose();             //关闭lua虚拟机
}
```





## 02_U3DScripting
> 主要看 LuaBehaviour.cs 绑定.lua.txt的方法

### 为什么要把`.lua`转化为`.txt`加载
先说结论：

不一定要转化为txt后使用。例子中仅仅是为了方便，把lua文件后加`.txt`后缀作为`TextAsset`，就可以被unity识别。

下面说细节：

- Unity识别不了`.lua`这种资源，因此既不能拖动到编辑器里，也不能像普通资源那样被识别出依赖关系并打包。
- 标准做法是：lua文件通过自己的`custumLoader`加载进来。lua文件本身存放在哪里呢？
- 放到 Resources 目录下这种方案**不行**，虽然会被打入包中，但和其他东西一起变成了整体，无法通过文件名提取出来。
- 放到 StreamingAsset 目录下这种方案**貌似可以**，由于其不压缩，故可以保留原始的目录信息。但由于该目录只读，因此不可用于存放热更时新下载下来的lua脚本。而且还是无法用Unity提供的加载API，需要自定义的loader加载。


---
#### 【Unity知识补充】`Resources` vs `StreamingAssets`
**Resources 和 StreamingAssets 相同点**

- 都会将文件夹中的资源打包


**Resources 和 StreamingAssets 区别**

- R 和 S 的打包结果不同。打包后，R 里面的内容压缩成一团， S 保持完整路径 （但只读）
> 例如ios下需要播放的影片必须放在有目录结构的文件夹中

- R 和 S 的加载方式不同。R用同步的`Resource.Load`，S 里面，如果是assetbundle，可以用`AssetBundle.LoadFromFile`同步加载；否则要区分平台？（android下还要解压） https://forum.unity3d.com/threads/assetbundle-load-question.402376/#post-2679871  这里也说了一个5.4之前版本加载StreamingAsset的问题： http://www.xuanyusong.com/archives/4033

---



### LuaBehaviour.cs 做了什么
- 所有LuaBehaviour共用一个Lua虚拟机（名叫 `luaEnv`），lua全局空间叫`luaEnv.Global`。注意并没有在mono脚本中关闭lua虚拟机。

- 一个LuaBehaviour拥有自己的一个LuaTable（名叫 `scriptEnv`）作为lua本地空间，并且其元表的__index设置为`luaEnv.Global`。在mono自己OnDestroy的时候，调用`scriptEnv.Dispose()`释放回收。

- 往lua本地空间里塞了c#的引用
```cs
scriptEnv.Set("self",this); 
//这样可以在绑定的lua脚本中通过 self 访问到LuaBehaviour这个mono类。
```

- 从lua本地空间里获取了函数的引用。并在需要的时候调用。
```cs
//这两种Get效果一样
luaAwake = scriptEnv.Get<Action>("awake");
scriptEnv.Get("start", out luaStart);
//xlua还提供一种GetInPath可以按照A.B.C访问lua的表，更多见《Xlua_API》.doc
```


### 为什么用 `[System.Serializable]` 修饰一个非mono类
首先贴代码。
```csharp
[System.Serializable]
public class Injection
{
    public string name;
    public GameObject value;
}
```
在LuaBehaviour的Awake方法中有一段：
```csharp
public Injection[] injections;

void Awake()
{
    foreach (var injection in injections)
    {
        scriptEnv.Set(injection.name, injection.value);
    }
}
```
可以看到 Injection类提供了将gameObject自动注册到本地空间的功能。在编辑器中，可以将gameObject拖动到LuaBehaviour暴露的变量上面。这就要求 Injection **可序列化**。因此必须用`[System.Serializable]`


### 绑定的lua脚本里写了什么
- 读写UnityC#的静态属性、方法，使用前缀`CS.`。因为在新建LuaEnv的时候，在init_xlua中将这些都存在了名为CS的表中。

```lua
local GameObject = CS.UnityEngine.GameObject  -- 先用局部变量引用后访问，能提高性能
GameObject.Find('helloworld')

local r = CS.UnityEngine.Vector3.up * CS.UnityEngine.Time.deltaTime * speed
self.transform:Rotate(r)   -- 访问成员函数用冒号
```






## 03_UIEvent
> 主要掌握给按钮绑定lua回调的方法。注意冒号和点号的使用。

```lua
self:GetComponent("Button").onClick:AddListener(function() print("ddd") end)
-- GetComponent(string)本来就是C#的写法，和GetComponent<T>()效果一样
-- 这里的self是LuaBehaviour中传入的指向该cs脚本的this，而不是lua这边的语法糖什么的
```








## 04_LuaObjectOrented
> 主要掌握：
1. c#中将lua函数添加到c#的委托。举一反三：lua中将lua函数添加到c#的委托。
2. c#接口调用lua函数。

### 为什么c#的接口定义前和委托定义前要加 `[CsharpCallLua]`？
首先，我们这里的目的是 **c#访问lua**。

- 对于接口，映射到lua是`table`。在《Xlua教程》.doc中说了“访问一个全局的table”的几种方法。如果是interface，其依赖于生成代码，所以要加[CsharpCallLua]。
- 对于委托，映射到lua是`function`。在《Xlua教程》.doc中说了“访问一个全局的function”。如果是delegate，其依赖于生成代码，所以要加[CsharpCallLua]。
> 【补充】除了用c#委托调用lua函数，还可以用Xlua提供的`LuaFunction`调用lua函数，不依赖生成代码，而且可以传任意类型、个数的参数。返回值是object数组。但性能较低、类型不安全。


### 在c#中将lua函数加入c#的委托列表
假设我们有一个定义好的c#委托类型和实例（甚至已经绑定了别的函数）
```csharp
[CsharpCallLua]
public delegate void Func(int a, int b);

void Start()
{
    Func funcObj = null;
    funcObj += FuncFromCS;//别的函数
}
```
和要绑定的Lua函数，
```lua
function FuncFromLua(a,b)
    print('a-b=' .. a-b)
end
```

现在想在c#中将这个lua函数加入该委托列表中。然后触发即可。
```csharp
Func func += luaenv.Global.Get<Func>("FuncFromLua");

func(3,1);  //会打印出 a-b=2

```

注意最终将lua函数添加到委托列表里是在c#里完成的。**实际可能还有另一种需求：动态地将某个lua函数添加到c#的委托上。这就要在lua里完成**。在《Xlua教程》.doc中的“Lua调用c#”中，介绍了这种方法。继续我们的例子加以说明。现在，`InvokeLua.cs`里有一个成员变量是一个event对象 JTestAction：
```csharp
[CSharpCallLua]
public delegate void JTest(int a, int b);

public event JTest JTestAction;

private string script = @"
    -- jony test
    
    function JTestLuaActionHandler(a,b)
        print('JTestLuaActionHandler,a+b=' .. (a+b))
    end
    
    csScript:JTestAction('+', JTestLuaActionHandler)  --注意是冒号
";
	        
void Start()
{
    LuaEnv luaenv = new LuaEnv();
    luaenv.Global.Set("csScript", this);
    luaenv.DoString(script);
    JTestAction(3, 4);      //会调用lua的JTestLuaActionHandler函数。
}

```
如果不用 `csScript:JTestAction('+', JTestLuaActionHandler)` 这样的语法，那么也可以这么写（见Tutorial的LuaCallCSharp）：
```lua
local TestClass = CS.Tutorial.TestClass
local testobj = TestClass()
testobj.TestDelegate = lua_delegate + testobj.TestDelegate --combine，这里演示的是C#delegate作为右值，左值也支持
testobj.TestDelegate = testobj.TestDelegate - lua_delegate --remove
```


#### 【C#知识补充】delegate vs event
> 看了上面的例子，估计会对c#的event和delegate有点迷糊，为什么需要event这个东西？

一言以蔽之：event是对delegeate的又一层封装，好处是：http://csharpindepth.com/Articles/Chapter2/Events.aspx (what's the point部分); http://stackoverflow.com/questions/29155/what-are-the-differences-between-delegates-and-events; http://stackoverflow.com/questions/4482613/creating-delegates-manually-vs-using-action-func-delegates

另外，event还有以下特点：
1、一个delegate原型可以对应多种event
2、通过event对象来触发消息只能在该对象所在的类中，不可在外面调用。
3、只能对event对象增加或减少某个监听，不可直接赋值（防止覆盖掉别人的监听）。



### 在c#中将table映射到c#的接口
接口比类更灵活，里面可以有方法，也可以有get/set。从文档中得知那么lua中的table可以映射到多种c#类型，使用上有什么限制？参考Examples里面的CSCallLua.cs。
```lua
-- d中包含两个string-value键值对，3个数字下标的整数，1个函数
d = {
   f1 = 12, f2 = 34, 
   1, 2, 3,
   add = function(self, a, b) 
      print('d.add called')
      return a + b 
   end
}
```

```csharp
//映射到class
public class DClass
{
    public int f1;
    public int f2;
}
DClass d0 = luaenv.Global.Get<DClass>("d");//映射到有对应字段，by value。d0.f1 = 12; d0.f2=34


//映射到interface
[CSharpCallLua]
public interface ItfD
{
    int f1 { get; set; }
    int f2 { get; set; }
    int add(int a, int b);
}
ItfD d3 = luaenv.Global.Get<ItfD>("d"); //映射到interface实例，by ref。注意接口要注明[CSharpCallLua]。d3.Add(6,7)会调用d.Add(6,7)


//映射到dictionary
Dictionary<string, double> d1 = luaenv.Global.Get<Dictionary<string, double>>("d");//映射到字典，by value。d1["f1"]=12;  d1["f2"]=34


//映射到List
List<double> d2 = luaenv.Global.Get<List<double>>("d"); //映射到列表，by value。d2[0]=1,d2[1]=2,d2[2]=3

//映射到Luatable（Xlua提供的）
LuaTable d4 = luaenv.Global.Get<LuaTable>("d");//映射到LuaTable，by ref。d4.Get<int>("f1")将返回12


```

### lua函数传入参数个数不定的映射
```csharp
void funcCS(int a, params string[] strs);
funcCS(1,"a","b","haha");
```
```lua
function funcLua(a, ...)
funcLua(1,"a","b","haha")
```

### lua函数返回参数有多个的映射
参考Examples里面的CSCallLua.cs。
```csharp
[CSharpCallLua]
public delegate int FDelegate(int a, string b, out DClass c);

public class DClass
{
    public int f1;
    public int f2;
}
    
FDelegate f = luaenv.Global.Get<FDelegate>("f");
int f_ret = f(100, "John", out d_ret);
//f_ret = 1 ; d_ret.f1=1024; d_ret.f2 = 0（默认值）

```
```lua
function funcLua(a, b)
    print('a', a, 'b', b)
    return 1, {f1 = 1024} -- 1将会映射到c#函数的返回值，后面的映射到out参数
end
```