## DI (Dependency Injection)
아래의 코드에서 `B`는 `A`에 의존하고 있음
```Go
type A struct {}

type B struct {
    a *A
}

func NewB() *B {
    a := &A{}
    return &B{a: a}
}
```
아래의 코드에서 `B`는 `AImpl`에 의존하지 않음
```Go
type A interface {
    DoSomething()
}

type AImpl struct {}

func (a *AImpl) DoSomething() {}

type B struct {
    a A
}

func NewB(a A) *B {
    return &B{a: a}
}
```
이처럼 인터페이스를 이용해서 DI를 사용할 수 있고, 프레임워크를 사용해서 할 수도 있음.(DI를 쓴다고 하면 이쪽이 일반적) 
`NewB`에서 `A`에 대한 생성은 외부에 맡기는데, 필요한 종속성을 코드 내에서 만들지 않고 외부에 맡긴다는 의미에서 이를 IoC(Inversion of Control)라고 함.

