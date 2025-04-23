# lua的学习资源
- [官方教材](http://www.lua.org/pil/)

  详实准确，但是英文的。
  
- [中文网站](http://www.runoob.com/lua/lua-tutorial.html)

  简易实用，可以快速浏览。

# 数据类型
- **nil**

a=nil 等效为删除a。未赋值的区域是nil的海洋

- **boolean**

只有 nil 和 false 会被判断为假

- **number**

lua5.3引入了整数，其类型也是number，但支持位操作。例如 2 | 3 = 3; 1 & 2=0;

- **string**

```lua
string.char(97)       --'a'
string.byte('a')      -- 97
string.format("my name is:%s" , nameStr) --格式化输出
table.concat({"a","b","cd"} , "-")       --字符串拼接"a-b-cd"

-- 字符串分割比较麻烦，见 http://lua-users.org/wiki/SplitJoin
```

- **thread**

lua的 `thread` 其实是协程。其和真正的线程差不多，拥有独立的栈、局部变量和指针，可以和其他thread共享全局变量。但是，真正的线程可以同时多个运行，而协程任意时刻只能运行一个。

- **userdata**



# 数据结构
- **table**

lua唯一的数据结构。
```lua
a = {}
a = {x=1.1, y=2.2}               --调用时，a.k 等同于 a["k"]，可理解为一种简化写法
a = {4, 'abc', {2}}              --a[1]等于4，下标默认从1开始

b = a                            --b和a指向同一个地址
b = {table.unpack(a)}            --b复制a（但注意不是递归的复制，所以若有子表还是会指向同一个地址）

table.unpack
table.concat                     --要求其所有元素都是字符串或数字
table.insert(tb,[pos], value)    --默认末尾插入，pos = [1, #tb+1]，超出会报错
table.remove(tb,[pos])           --默认末尾移除，pos = [1, #tb]，太大返回 nil，太小报错
table.sort(tb, [compfunc])       --compfunc(t1,t2) ，返回 true 表明 t1在t2的前面。不稳定排序
table.copy                       --没有这种，要自己实现。参见 http://lua-users.org/wiki/CopyTable
```




# 变量
- 默认为全局，需要显式声明为 `local` 才为局部变量。
- **访问局部变量更快**。如有必要，可将全局变量的**引用**保存到局部，加快访问速度，也可同步更改全局变量的值。
```lua
a = {1,2,3} -- 全局
do -- 强制局部作用域
    local la = a  -- 对table是引用而不是拷贝（对字符串、数字、布尔等类型都是拷贝，故不适用于此例）
    la[1]=9
    
    -- call some func. 如果func里面更改了a，这里的la也会改变
    
    for k,v in pairs(la) do  
        print(v) -- pairs迭代器会遍历table的所有键值，ipair只会遍历连续的、1开始的、数字键值
    end
end
```




# 函数
- 可作为参数传递
- 支持匿名函数
- 默认为 public 函数，用local声明则为该模块的私有函数
- 【语法糖】`func({1,2,3})`可简写为`func{1,2,3}`
- 【语法糖】
  - 定义时。`function t:func(p1,p2)` 等效为 `function t.func(self,p1,p2)`
  - 调用时。`t:func(1,2)` 等效为 `t.func(t,1,2)`


# 模块与包
- [模块（module）](http://www.runoob.com/lua/lua-modules-packages.html)就是一个定义了一个table的文件，最后 return 该模块名称。
- 包（package）就是一系列的模块（module）
- require "module_name"，会先从`package.loaded`中找，再用lua加载器从`package.path`中找，再用c加载器从`package.cpath`中找，再用[一体化加载器](http://www.runoob.com/manual/lua53doc/manual.html#pdf-package.searchers)

# 环境
- 引用一个叫 var 的自由名字（指在任何层级都未被声明的名字）在句法上都被翻译为 `_ENV.var` 。 此外，每个被编译的 Lua 代码块都会有一个外部的局部变量叫 _ENV。 
- `_ENV`是一个table，被称为`环境`
- `全局环境`叫做`_G`，被保存在c注册表的一个特别索引下。而且，_G._G == _G。
- 所有的标准库都被加载入全局环境

```lua
for k,n in pairs(_G) do print(k, '\t', n) end  --查看全局变量
```

- lua53去掉了setfenv和getfenv，转而用_ENV和upvalue的概念。[参考这里](http://leafo.net/guides/setfenv-in-lua52-and-above.html)


# 元表
```lua
setmetatable(t, meta1)
getmetatable(t)
print(t)   -- 会调用 meta1.__tostring，可以自定义为一个返回字符串的函数
t+t2       -- 会调用 meta1.__add
```

- 若干table可以共用一个元表，用于共享一些自定义的行为，如两个表相加。
- 一个表的元表可以设定为自己


- `__index`元方法。当我们对一个表进行键值访问`t.anonexsitkey`，内部触发的逻辑顺序是：
 - 在表t中找anonexsitkey这个键，找到则返回对应值，找不到执行下一步
 - 找t的元表并判断元表有没有__index，找不到返回nil，有则执行下一步
 - __index若是函数则调用。其若是表就在这个表中找键值（这其实也可以理解为一个语法糖，因为用的太频繁）。
 - 注：不想触发__index则用`rawget(t,key)`


- `__newindex`元方法。当我们对一个表的键赋值`t.anonexsitkey = myValue`，内部触发的逻辑顺序是：
 - 在表t中找anonexsitkey这个键，找到则替换成myValue，找不到执行下一步
 - 找t的元表并判断元表有没有__newindex，找不到则替换成myValue，有则执行下一步
 - __newindex若是函数则调用（myValue就不执行赋值），若是表则将myValue更新到这个表里而不是t中（有点像直接改变基类的成员）。
 - 注：不想触发__newindex则用`rawset(t,key,value)`

```lua
-- 只读table的实现。思路巧妙，用了一个空表，把数据放在了元表的__index里。
function readOnly (t)
  local proxy = {}
  local mt = {       -- create metatable
    __index = t,
    __newindex = function (t,k,v)
      error("attempt to update a read-only table", 2)
    end
  }
  setmetatable(proxy, mt)
  return proxy
end

days = readOnly{'mon','tues','wed','thurs','fri'}

```

- `__metatable`元方法。若mt是t的元表，则保护t的元表不被改变，可以 `mt.__metatable = 'i am hiden and non-changable'`。这样getmetatable(t)会给出这句话，setmetatable(t,other)会报错。[参见](https://www.lua.org/pil/13.3.html)

- 更多元方法简介见 http://lua-users.org/wiki/MetatableEvents

# 错误处理
- `assert( l1, msgStr)` l1为false则打印msgStr
- `error(msgStr, level)` level=0,1,2
- `pcall(func, param1, param2)` 保护模式运行func，忽略func内部的错误处理，返回true或false
- `xpcall(func, errhandle, param1,param2)` 和pcall类似，但自定义错误处理函数。结合：
 - debug.traceback()
 - 更多[调试库函数](http://www.runoob.com/lua/lua-debug.html)

# 垃圾回收
```lua
collectgarbage("count")     --memory(KB) used by lua
collectgarbage()            --full gc
collectgarbage("count")
```


# 面向对象
- lua没有class，而是类似prototype-based. 所谓prototype， 其实就是一个普通的实体，但只有一个作用：提供公共的行为。所以要切换一下思维：lua中的“类”不是一个抽象的概念，而是一个个活生生的原型实体。

下面的写法让b成为a的prototype（也可以称作b是一个class）. a的metatable在这里是匿名的。
```lua
setmetatable(a, {__index = b})
```
但这样每次以b为原型创建对象时都会新建一个metatable，挺浪费的。优化的写法是：
```lua
function b:new(o)
    o = o or {}
    setmetatable(o, self)  --如果没有上下文，这里self就是b. b直接作为o的元表而不是用一个匿名的表
    self.__index = self  --这里可理解为 b.__index = b.
    return o
end

a = b:new()
```
接着上面，如果b这张表里定义了一个键对应一个函数，比如 `b:func`
```lua
fcuntion b:func()
    print("defined by b")
end
```

则现在a也可以调用
```lua
a:func() -- 等效为a.func(a)。a里没有func这个键，则转化为 getmetatable(a).__index.func(a)，也就是b.func(a)
```
如果a也想作为一个类，并且继承b呢？下面将a打造成b的子类，改变func，创造出以a为原型的c
```lua
fcuntion a:func()
    print("defined by a")
end

c = a:new() 
-- 等效为a.new(a,nil)，a里没有new这个键，转化为getmetatable(a).__index.new(a)，转化为b.new(a,nil)
-- 所以现在，c的metatable变成了a，a.__index = a。与此同时，并没有修改a的metatable相关的东西。

c:func() -- getmetatable(c).__index.func(c)，即a.func(c)
```
上述会输出`defined by a`而不是会输出`defined by b`，妙就妙在b:new中用了self。如果把self直接改写为b，那么由于b就是个实体，那么，b.new(a,nil)就还是会把c的metatable变成b，b.__index=b。


# 底层交互
- **lua和c存在两种调用关系**
 - lua作为主程序，调用c编译出的库（例子，算法加速）
 - c作为主程序，调用lua编译出的库（例子，lua翻译器本身）

- lua和c通信通过一个**虚拟栈**，来解决两种语言的差异：
 - GC。虚拟栈由lua维护，所以知道哪些正被C使用，故不清除。
 - 变量类型。主要是解决了若干种C类型和若干种Lua类型关联的[组合爆炸](https://www.lua.org/pil/24.2.html)。每种类型只需要实现和虚拟栈中统一类型的转化函数。

- 虚拟栈有两种索引顺序：
 - 1.栈底(1)->栈顶(n)
 - 2.栈顶(-1)->栈底(-n)
 - C可以访问和修改栈任意位置的值，lua只能按照LIFO原则访问修改栈顶。

## 压栈和出栈取值
- 压栈
 - lua->栈的`lua_pushunumber`
 - c->栈，`userdata`?稍后讨论
 - 一次压栈太多要防止超过容量。`lua_checkstack`
- 取值
 - lua从栈上取值要判断类型，利用`lua_isnumber`之类的
 - c从栈上取值，利用`lua_tonumber`之类的，将返回double
 - 如果取的是table的某个键值，这里有一个特殊处理（假设lua中有一个bgcolor = {r=1, g=0, b = 0}）：

- 例子1

```c
//------------------------------------------------------
//假设L当前栈顶是bgcolor这张表。c中要这么获得 bgcolor.r
lua_getgloble(L, "bgcolor");   //将lua全局变量get到lua环境(L)中

if( !lua_istable(L, -1))
    error(L, " not a table")

char * key = "r";
lua_pushstring(L, key);
lua_gettable(L, -2)         //这个函数会找到这个table，并pop出栈顶为key去访问table，再将结果压栈

if( !lua_isnumber(L, -1))   // 先判断上面压栈的结果是不是数字
    error(L, "don't have key")

int result = (int)lua_tonumber(L, -1);   
lua_pop(L,1);

//------------------------------------------------------
//假设L当前栈顶是bgcolor这张表。c中要这么改变 bgcolor.g
lua_pushstring(L, "g")     // 压栈key
lua_pushnumber(L, 0.555)   // 压栈value
lua_settable(L, -3)        // 因为-3是现在bgcolor在L上的位置


//------------------------------------------------------
//如果是c中要在lua里面新建一个table变量 mytable，还要
lua_newtable(L)
{
 //... 执行刚才 settable 类似的赋值
}
lua_setglobal(L, "mytable")
```
 
- 例子2
c调用lua的一个函数myluafunc

 

## [**一个简化的c程序调用luaAPI的流程**](https://www.lua.org/pil/24.1.html)
 - open lua state L (`lua_open`) 注：创建了一个全新的独立的lua环境
 - load lib into L (`luaL_openlibs`)
 - compile commandStr in buff , and push compiled code to L's stack  - (`luaL_loadbuffer`)
 - pop code from L's stack and execute in protect mode (`lua_pcall`)
 - if lua throws error msg in L, print & pop it (`lua_tostring`, `lua_pop`)
 - close lua state L(`lua_close`)
 


## C_API 错误处理
- lua调用c库，
  c库会报错导致程序crash，标准的处理是让c库调用`lua_error`
- c调用lua，
  分配栈空间不足也会报错导致crash（除非使用了保护模式即`lua_pcall`，其会返回错误码）