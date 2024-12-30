## Go에서 enum 사용하는법
```Go
type enumType string
```
처럼 type을 하나 새로 생성한다.  
주의할 점은 다음과 같이 하면 새 타입이 아니라 alias임
```Go
type enumType = string
```
그 다음 const로 enum 값들을 대입하면 됨
```Go
type enumType string

const (
    enumValue1 enumType = "v1"
    enumValue2 enumType = "v2"
    enumValue3 enumType = "v3"
)
```
이렇게 enum처럼 사용해서 값을 제한하는 데 쓰거나 리팩토링시 이 부분만 변경하면 돼서 쉬워짐