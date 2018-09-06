# 深入剖析Unity协程

---
- [深入剖析Unity协程](#%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90unity%E5%8D%8F%E7%A8%8B)
    - [从`IEnumerator/IEnumerable` 到`Yield`](#%E4%BB%8Eienumeratorienumerable-%E5%88%B0yield)
    - [Unity协程的实现](#unity%E5%8D%8F%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0)
    - [参考资料](#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)
---


## 从`IEnumerator/IEnumerable` 到`Yield`

c#语言中，迭代器这个特性大家不会陌生，最常见的莫过于`foreach`了。`foreach`能够对一个实现了`IEnumerable`接口的对象`dataSource`进行遍历访问其中的元素，过程可以拆解为：
```cs
//using foreach
foreach (var item in dataSource)
{
    Console.WriteLine(item.ToString());
}
//equals to:
IEnumerator iterator = dataSource.GetEnumerator(); 
while (iterator.MoveNext()) 
{ 
    Console.WriteLine(iterator.ToString());
}
```

细心的读者会发现，为什么迭代器要涉及`IEnumerable` 和 `IEnumerator`两个接口呢？为什么不可以直接在`dataSource`中实现`MoveNext`和`Current`？

这z正是迭代器模式的精髓。这个模式将存储数据和遍历数据的职责分离，在c#中对应为`IEnumerable` 和 `IEnumerator`两个接口（其实还有两个泛型接口：IEnumerable\<T>, IEnumerator\<T>，不加 T 的话则默认为IEnumerator\<object>）。本文对该模式不做展开讨论。

如何利用`IEnumerable` 和 `IEnumerator`，使自己的类支持`foreach`呢？

在c#1.0中，你只能这样做：
```cs
public class DataSource : IEnumerable
{
    public IEnumerator GetEnumerator()
    {
        Enumerator enumerator = new Enumerator(0);
        return enumerator;
    }

    public class Enumerator : IEnumerator, IDisposable
    {
        private int state;
        private object current;

        public Enumerator(int state)
        {
            this.state = state;
        }

        public bool MoveNext()
        {
            switch (state)
            {
                case 0:
                    current = "Hello";
                    state = 1;
                    return true;
                case 1:
                    current = "World";
                    state = 2;
                    return true;
                case 2:
                    break;
            }
            return false;
        }

        public void Reset()
        {
            throw new NotSupportedException();
        }

        public object Current
        {
            get { return current; }
        }
        public void Dispose()
        {
        }
    }

}

class Program
{
    static void Main(string[] args)
    {
        DataSource dataSource = new DataSource();
        foreach (string s in dataSource)
        {
            Console.WriteLine(s);
        }
    }
}
```
可以看到，需要在`DataSource`类中实现一个`Enumerator`类，其采用状态机的方式实现`MoveNext`的逻辑，稍有不慎就会产生差错。

c#2.0后引入了语法糖`yield`，较完美的解决了迭代器模式的易用性这个问题。同样的功能，只需寥寥几行便可实现。
```cs
public class DataSource : IEnumerable
{
    public IEnumerator GetEnumerator()
    {
        yield return "Hello";
        yield return "World";
    }
}
```
好奇的读者从这个例子中，其实已经在猜想到`yield return`是如何实现等价效果的：对于一个位于返回值为`IEnumerator`的函数里的yield，做如下展开：

1. 静默创建了一个`IEnumerator`对象
~~2. 立刻调用了这个对象的`MoveNext()`方法~~
3. `MoveNext()`方法中保存了当前状态，为`Current`赋值（正是`yield return`后面的值）。最后一个`yield return`语句，对应的`MoveNext()`返回false，否则为true。
4. 外界访问了迭代器的`Current`状态。
5. 如果刚才`MoveNext()`返回了true，则重复执行第二步，直到第三步返回false。~




## Unity协程的实现

## 参考资料
- 社区讨论
- 博客
- 书籍