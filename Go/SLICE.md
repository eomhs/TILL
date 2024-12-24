## 슬라이스를 인자로 넘겨서 callee에서 append를 해도 caller에서는 반영이 안 되는 것일까?
- 참조링크: https://go.dev/blog/slices-intro
- 보통 Go에서 슬라이스는 다음과 같이 사용한다
```Go
func caller() {
    arr := make([]int, 0, 10)
    arr = callee(arr)
	for _, n := range arr {
		fmt.Println(n)
	}
}

func callee(arr []int) []int {
	arr = append(arr, 1)
	return arr
}

stdout: 1
```

- 이건 왜 안될까? arr은 결국 포인터도 가지고 있는데
```Go
func caller() {
    arr := make([]int, 0, 10)
    callee(arr)
	for _, n := range arr {
		fmt.Println(n)
	}
}

func callee(arr []int) {
	arr = append(arr, 1)
}

stdout: 
```
- 답은 슬라이스는 pointer, length, capacity 세 가지를 가지고 있어서
- caller의 length가 아직 0이라서 그렇다!
- 실제로 pointer가 가리키는 슬라이스에는 1 이 추가되어 있다
- 다음의 코드로, length를 1로 재설정하면 접근 가능
```Go
func caller() {
    arr := make([]int, 0, 10)
    callee(arr)
    arr = arr[:1]
	for _, n := range arr {
		fmt.Println(n)
	}
}

func callee(arr []int) {
	arr = append(arr, 1)
}

stdout: 1
```
