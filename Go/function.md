## 메소드도 함수 인자로 전달할 수 있을까?
있다!     
`func (foo *Foo) Bar()`   
은 다음의 함수 시그니처를 가진다.
`func Bar(*Foo)`    
따라서 다음과 같이 메소드를 인자로 전달할 수 있다.
```go
type Foo struct{}

func (foo *Foo) Bar() {
    fmt.Println("Bar")
}

func main() {
    f := func(m func(*Foo)) {
        m()
    }
    foo := &Foo{}
    f(foo.Bar) // Bar
}
```

