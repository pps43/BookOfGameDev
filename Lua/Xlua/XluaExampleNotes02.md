# Xlua Examples 学习（二）


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

lua中：
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

c#中：
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
lua中：
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
lua中：
```lua
function funcLua(a, b)
    print('a', a, 'b', b)
    return 1, {f1 = 1024} -- 1将会映射到c#函数的返回值，后面的映射到out参数
end
```