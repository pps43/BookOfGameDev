# How to mock Polymorphysm in Go

> interface中保存着对象的类型和指向值的指针。但要注意，在创建interface对象时，指向对象的类型需要精确指定，而不能依靠自动识别。

首先明确目标：拿到一个`*Base`对象，调用`Foo()`方法，但其实调用的是“子类”的Foo方法，这就是多态。

为此需要：
- Base类中需要利用接口对象保存“子类”的指针，该指针在创建“子类”对象时必须传入。
- 在需要实现多态的方法中，必须指明调用指针对象的同名方法。

以上缺一不可。另外在函数调用链中，一旦中途子类对象的指针被赋值给了*Base，类型信息就丢失了。

例子
```go
    package main
    import "fmt"
    
    func main() {
      var b *Base = new(Child).Create().GetBase()
      b.Foo() // Sub.Foo is called
    }
    
    type IMatch interface {
      Create() IMatch
      GetBase() *Base
      Foo()
    }
    type Base struct {
      internal IMatch
    }
    type Child struct {
        Base
      }
    // abstract
    func (this *Base) Create() IMatch {
        panic("Base.Create() should not be called")
        return this
    }
    func (this *Base) GetBase() *Base {
          return this
    }
    func (this *Base) Foo() {
      if this.internal != nil {
        this.internal.Foo()
      }
    }
    func (this *Child) Create() IMatch {
        this.internal = this
        return this
    }
    func (this *Child) Foo() {
      fmt.Println("Child.Foo is called")
    }

```