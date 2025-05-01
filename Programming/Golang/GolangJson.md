# How to use json.Marshal in Go

`Marshal` traverses the value v recursively.
- If an encountered value implements the `Marshaler` interface and is not a nil pointer, Marshal calls its `MarshalJSON` method to produce JSON. 
- If no MarshalJSON method is present but the value implements `encoding.TextMarshaler` instead, Marshal calls its `MarshalText` method and encodes the result as a JSON string

也就是说，对于自定义类型，要支持json序列化，要实现以下接口：

```go
type Marshaler interface {
    MarshalJSON() ([]byte, error)
}
    
type TextMarshaler interface {
    MarshalText() (text []byte, err error)
}
```