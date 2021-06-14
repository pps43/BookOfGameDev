# struct灵魂拷问

以下内容灵感来源于一次技术交流，基于`CLR`运行时。

### struct也继承自System.Object，与class的区别到底在哪里？
struct继承自`ValueType`（所有值类型的基类），class属于引用类型，而值类型与引用类型的本质区别在于前者在拷贝时是按值拷贝，后者拷贝的是地址。其他区别还有：struct对象一般在线程栈上（除非嵌入了引用类型中作为字段），class总是在托管堆上由GC管理；struct对象不可能为null，总是初始化为0（不能自定义无参构造器）；struct不支持继承；等。

### struct为什么不能被继承（从语法上和设计上回答）
从语法上来说，struct继承`ValueType`的时候是隐式密封的，观察相应的IL代码即可发现`sealed`关键字。
从设计上来说，struct多用于纯数据的集合（也可以有逻辑），内存布局比较简单，没有类型对象指针和同步块索引。因此不能直接支持多态。


### struct 可以实现接口吗
可以，系统里很多就是这么做的，比如 System.Int32
实际上CLR中的接口只是对一组方法签名进行了统一命名，这些方法不提供任何实现。


### struct 可以自定义无参构造器吗
c#中不可以，会报错`CS0568: Struct cannot contain explicit parameterless constructors`。但CLR中其实没有这个限制。
如下代码中，CLR为了高效，会直接将内存空间置0，并不会自动调用1000次MyStruct的构造函数。如果c#允许定义无参构造器，那么在这里创建出来的`MyStruct`对象大概率是不合格的，会让使用者更加困惑。
```cs
MyStruct[] foo = new MyStruct[1000];
```
多说一句，c#有一个设计理念是：类型的默认值不应当依赖于初始化。


### struct何时被装箱
其实就是问值类型何时被装箱。
1. 将值类型对象赋值/转换给一个接口（接口也是引用类型）
2. 将值类型对象赋值/转换为一个`object`（转换大多发生在函数参数传递）
3. 调用`Object`中的非虚方法`GetType()`
4. 调用`ValueType`或`Object`中的虚方法（例如`ToString()`），但自身没有实现`ToString()`。

其他情形，例如struct作为字典的key，可以归属到上述的`4`中。下面通过代码解释最难理解的第`4`种情况，其余三种可以仿效下面的方法自行检验，或者见 [.NET 装箱/拆箱机制](dotNetBoxing.md)

```cs
public struct MyStruct0
{
    public string Name { get; set; }
    // Call ToString() will call ValueType.ToString(), which has boxing
}
public struct MyStruct1
{
    public string Name { get; set; }
    public override string ToString()
    {
        return base.ToString(); // boxing
    }
}
public struct MyStruct2
{
    public string Name;
    public override string ToString()
    {
        return Name; // no boxing
    }
}

MyStruct0 m0 = new MyStruct0();
MyStruct1 m1 = new MyStruct1();
MyStruct2 m2 = new MyStruct2();
m0.ToString(); // boxing
m1.ToString(); // boxing
m2.ToString(); // no boxing, see ToString() defined in MyStruct2
```

在调用ToString()方法时，IL代码并无不同，都是`constrained.`跟着`callvirt`。`callvirt`并不代表只有引用类型可以用，值类型也可以，在前面加上`constrained.`就行。而且`callvirt`的出现并不意味着装箱。这个IL指令会另起一篇文章详细阐述。

```cs
/* (76,13)-(76,27) E:\local\CSharpNetFramework\Program.cs */
/* 0x00000346 1206         */ IL_008A: ldloca.s  m0
/* 0x00000348 FE1603000002 */ IL_008C: constrained. CSharpProj.MyStruct0
/* 0x0000034E 6F1400000A   */ IL_0092: callvirt  instance string [mscorlib]System.Object::ToString()
/* 0x00000353 26           */ IL_0097: pop
/* (77,13)-(77,27) E:\local\CSharpNetFramework\Program.cs */
/* 0x00000354 1207         */ IL_0098: ldloca.s  m1
/* 0x00000356 FE1604000002 */ IL_009A: constrained. CSharpProj.MyStruct1
/* 0x0000035C 6F1400000A   */ IL_00A0: callvirt  instance string [mscorlib]System.Object::ToString()
/* 0x00000361 26           */ IL_00A5: pop
/* (78,13)-(78,27) E:\local\CSharpNetFramework\Program.cs */
/* 0x00000362 1208         */ IL_00A6: ldloca.s  m2
/* 0x00000364 FE1605000002 */ IL_00A8: constrained. CSharpProj.MyStruct2
/* 0x0000036A 6F1400000A   */ IL_00AE: callvirt  instance string [mscorlib]System.Object::ToString()
/* 0x0000036F 26           */ IL_00B3: pop
```

虽然上面的IL中看不到`box`，展开进一步的分析。
m0没有自己的Override，因此会调用父类ValueType的ToString()，为此一定要box的；
m1在自己的Override中调用了base.ToString()，因此从IL代码中直接可以看到box这条指令；
m2在自己的Override中直接返回一个string，因此不需要box；

```cs
//m1.ToString()的IL代码
// Token: 0x06000007 RID: 7 RVA: 0x00002074 File Offset: 0x00000274
.method public hidebysig virtual 
	instance string ToString () cil managed 
{
	// Header Size: 12 bytes
	// Code Size: 22 (0x16) bytes
	// LocalVarSig Token: 0x11000001 RID: 1
	.maxstack 1
	.locals init (
		[0] string
	)

	/* (29,9)-(29,10) E:\local\CSharpNetFramework\Program.cs */
	/* 0x00000280 00           */ IL_0000: nop
	/* (30,13)-(30,36) E:\local\CSharpNetFramework\Program.cs */
	/* 0x00000281 02           */ IL_0001: ldarg.0
	/* 0x00000282 7104000002   */ IL_0002: ldobj     CSharpProj.MyStruct1
	/* 0x00000287 8C04000002   */ IL_0007: box       CSharpProj.MyStruct1
	/* 0x0000028C 281100000A   */ IL_000C: call      instance string [mscorlib]System.ValueType::ToString()
	/* 0x00000291 0A           */ IL_0011: stloc.0
	/* 0x00000292 2B00         */ IL_0012: br.s      IL_0014

	/* (31,9)-(31,10) E:\local\CSharpNetFramework\Program.cs */
	/* 0x00000294 06           */ IL_0014: ldloc.0
	/* 0x00000295 2A           */ IL_0015: ret
} // end of method MyStruct1::ToString

//m2.ToString()的IL代码
// Token: 0x06000008 RID: 8 RVA: 0x00002098 File Offset: 0x00000298
.method public hidebysig virtual 
	instance string ToString () cil managed 
{
	// Header Size: 12 bytes
	// Code Size: 12 (0xC) bytes
	// LocalVarSig Token: 0x11000001 RID: 1
	.maxstack 1
	.locals init (
		[0] string
	)

	/* (39,9)-(39,10) E:\local\CSharpNetFramework\Program.cs */
	/* 0x000002A4 00           */ IL_0000: nop
	/* (40,13)-(40,25) E:\local\CSharpNetFramework\Program.cs */
	/* 0x000002A5 02           */ IL_0001: ldarg.0
	/* 0x000002A6 7B03000004   */ IL_0002: ldfld     string CSharpProj.MyStruct2::Name
	/* 0x000002AB 0A           */ IL_0007: stloc.0
	/* 0x000002AC 2B00         */ IL_0008: br.s      IL_000A

	/* (41,9)-(41,10) E:\local\CSharpNetFramework\Program.cs */
	/* 0x000002AE 06           */ IL_000A: ldloc.0
	/* 0x000002AF 2A           */ IL_000B: ret
} // end of method MyStruct2::ToString
```


### 同一个struct对象分别经历两次装箱，装箱后的对象地址相同吗
由装箱的过程可知，每次都是在堆内存上新申请一块空间并将值拷贝过去，因此两次装箱的对象地址不可能相同。

### 两个引用类型的对象默认是怎么比较相同的
通过比较二者的内存地址。参见`Object.Equals`。

### struct作为字典的key产生了装箱后，而且装箱后地址不同，但字典似乎依然正常执行逻辑。它是如何判定key相等的？
既然讨论的是发生了装箱，也就是该struct并没有override自己的`Equals`，因此装箱后的对象调用的是`ValueType.Equals`。该方法正确处理了box后的对象的相等性判断，避免了使用`Object.Equals`带来的问题。具体来说，其过程是：
- 如果obj为`null`，直接返回`false`；
- 如果二者的类型不同，直接返回`false`；
- 尝试进行内存逐bit快速比较；
- 逐一比较每个字段，调用其对应类型的`Equals`方法。

`ValueType.Equals`反编译后的源码如下：

```cs
// System.ValueType
/// <summary>Indicates whether this instance and a specified object are equal.</summary>
/// <param name="obj">The object to compare with the current instance. </param>
/// <returns>
///     <see langword="true" /> if <paramref name="obj" /> and this instance are the same type and represent the same value; otherwise, <see langword="false" />. </returns>
// Token: 0x06001570 RID: 5488 RVA: 0x0003EC40 File Offset: 0x0003CE40
[SecuritySafeCritical]
[__DynamicallyInvokable]
public override bool Equals(object obj)
{
	if (obj == null)
	{
		return false;
	}
	RuntimeType runtimeType = (RuntimeType)base.GetType();
	RuntimeType left = (RuntimeType)obj.GetType();
	if (left != runtimeType)
	{
		return false;
	}
	if (ValueType.CanCompareBits(this))
	{
		return ValueType.FastEqualsCheck(this, obj);
	}
	FieldInfo[] fields = runtimeType.GetFields(BindingFlags.Instance | BindingFlags.Public | BindingFlags.NonPublic);
	for (int i = 0; i < fields.Length; i++)
	{
		object obj2 = ((RtFieldInfo)fields[i]).UnsafeGetValue(this);
		object obj3 = ((RtFieldInfo)fields[i]).UnsafeGetValue(obj);
		if (obj2 == null)
		{
			if (obj3 != null)
			{
				return false;
			}
		}
		else if (!obj2.Equals(obj3))
		{
			return false;
		}
	}
	return true;
}

```

---
灵魂拷问结束。

如果对.NET CLR没有系统性的了解，看到这里可能已经满头大汗。推荐看《CLR via C#》第二部分（尤其第5章），相信阅读后会豁然开朗。

同时极力推荐使用`dnSpy`作为IL查看工具，在实践中反思书本上的知识有没有过时，亦或是自己的理解有误。例如在观察是否有装箱时，笔者之前有一个不正确的理解：看不到`box`指令，就没有装箱。因此在分析一个没有实现自己的`ToString()`的struct对象调用`ToString()`时，因为没有观察到`box`，对书本和ECMS335标准上的说法产生了困惑，以为是编译器做了某种神奇优化。正确的理解是：实际会调用到`ValueType.ToString()`，为此struct一定会装箱。