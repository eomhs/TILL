## Go 프로젝트 레이아웃
- 참조링크: https://github.com/golang-standards/project-layout/blob/master/README_ko.md
### /internal
- internal 디렉토리에 있는 패키지는 다음의 경우에만 참조 가능함
    - 부모 패키지
    - 부모 패키지의 하위 패키지
    - internal의 하위 패키지

