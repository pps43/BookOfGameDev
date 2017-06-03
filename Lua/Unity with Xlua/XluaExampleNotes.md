# Xlua Examples 学习（一）

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



下一篇将继续分析示例代码。




