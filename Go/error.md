## 에러 관련 이것저것
- [cockroachdb/errors](https://github.com/cockroachdb/errors) 패키지 좋음(stack trace 잘해줌)
- errors.Wrap 하면 "outer: inner" 형태로 찍힘
- `fmt.Printf("%+v", err)` 형태로 찍으면 stack trace 다 보여줌

