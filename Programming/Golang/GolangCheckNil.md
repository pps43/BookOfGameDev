# How to check nil in Go

> 涉及interface时，由于interface对象里包含了类型指针和真实对象指针，所以只对interface对只判断nil是不够的。下面列举三种容易踩坑的情况以及正确的判空方法。最后一个最隐蔽也最容易出错：当不得以要返回一个interface返回值时，要格外注意判空方法！

## Check `Type Assertion`
知道具体类型，转换成具体类型判断：
```go
a, ok := obj.(*SomeType)
if ok && a!= nil {
    // really ok
}
```

## Check `interface{}`
不知道具体类型，只能从interface对象判断真实对象是否有效：
```go
func isNil(a interface{}) bool {
    defer func() {
        recover()
    }()
    return a == nil || reflect.ValueOf(a).IsNil()
}
```

## Check `Return Value`

函数返回一个interface的情况本质上不复杂，但很容易被调用方忽视。总结概括为：要么转成具体类型判断，要么使用反射`reflect.ValueOf(a).IsNil()`判断。

下面是具体例子。
```go
package main
import (
    "fmt"
    "reflect"
)
    
type IMatchTeam interface {
    GetTeamIndex() uint32
}

type MatchTeam struct {
}

func (this *MatchTeam) GetTeamIndex() uint32 {
    return 0
}
    
// this function return an interface
func ChangeTeam() IMatchTeam {
    var t *MatchTeam
    return t // "t" is nil, but Go wraps this "typed nil" into a interface object, which is not nil itself
}
    
func main() {
    res := ChangeTeam() // res now is an interface object

    if res != nil {
        fmt.Println("This is true!")
            
        ti, ok := res.(IMatchTeam)
        if ok {
            fmt.Println("This is true! Suprise!")
        }
        if ti != nil {
            fmt.Println("This is true! Suprise!!")
        }
        if !reflect.ValueOf(ti).IsNil() { // the only right way to check underlying value of ti
            fmt.Println("Never happen")
        }
    
        t, ok2 := res.(*MatchTeam)
        if ok2 {
            fmt.Println("This is true! Suprise!!!")
        }
        if t != nil {
            fmt.Println("Never happen")
        }
    }
}
```