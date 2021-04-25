# 六个必备设计模式

本文介绍游戏客户端开发中几个十分朴实但实用的设计模式，是个人在经历不同游戏项目后的经验总结。
每种模式在阐明用途后，均给出参考代码（以Unity引擎为例）。在实践中，需要根据项目需求进行扩展或简化。

## 单例
单例可能是被使用最多也是被误用最多的一种模式。由于管理游戏对象时势必需要各类“Manager”，且这些Manager类对象通常生命周期比较长，如果用一个个全局单例来实现，不管在哪里都能访问，似乎十分方便。然而，笔者并不鼓励这种依赖多个全局单例的写法，这样会造成模块调用关系混乱而难以维护。推荐的方法是只有一个单例，并通过依赖注入的方式传递给需要访问它的类。

这并不是否认单例写法的使用面，甚至在早期的小规模预研中，全局单例的写法有助于尽快搭建框架进行玩法验证。用Unity引擎进行原型开发时，由于只有主线程能访问`GameObject`这类Native资源，所以`写法三`通常已经足够。

```cs
//写法一：c# 线程安全的单例
public sealed class Singleton
{
    private static volatile Singleton instance;
    private static object syncRoot = new object();

    private Singleton(){} //防止外界创建该类对象

    public static Singleton Instance
    {
        get
        {
            if (instance == null) //提升性能，不用每次加锁
            {
                lock (syncRoot)
                {
                    if (instance == null)
                    {
                        instance = new Singleton();
                    }
                }
            }

            return instance;
        }
    }
}

//写法二：c# 线程安全的单例，利用了 Lazy<T> 内部已有的多线程处理逻辑。适用 .Net4 以上版本
public class Singleton
{
    private static readonly Lazy<Singleton> instance = new Lazy<Singleton>(() => new Singleton());

    private Singleton() { }

    public static Singleton Instance => instance.Value;
}

//写法三：继承MonoBehaviour以用于Unity引擎，单线程版。
public abstract class USingleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;

    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                T[] objs = GameObject.FindObjectsOfType<T>(true);
                if (objs.Length > 0)
                {
                    instance = objs[0];
                    for (int i = 1; i < objs.Length; i++)
                    {
                        GameObject.Destroy(objs[i].gameObject); //去除重复
                    }
                }
                else
                {
                    GameObject newObj = new GameObject(typeof(T).Name); //自动创建 object
                    DontDestroyOnLoad(newObj); //生命周期跨场景
                    instance = newObj.AddComponent<T>();
                }
            }

            return instance;
        }
    }
}
//使用方法
public class GameManager : USingleton<GameManager>
{

}

```

## 抽象工厂

## 对象池

## 命令模式
例一个简易的输入系统

## 状态模式
一个简易的状态机

## 观察者模式
一个简易的消息分发系统

## 更多资料
- 《Game Programming Patterns》一书较为系统的介绍了游戏开发中常见的设计模式，以上介绍的几种模式在书中亦有提及。在线阅读地址：http://gameprogrammingpatterns.com/