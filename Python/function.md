## Default parameter가 실행되는 시점
- 함수 호출이 아니라 함수가 정의될 때(import) 실행됨
```python
def f1(p=f2()):
    ...
```
- 이런 예시에서 f2의 실행 시점을 잘 고려해야함