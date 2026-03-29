## GOMAXPROCS
- 프로세스가 사용할 수 있는 최대 CPU 수
- go 1.25 이전에는 시스템의 총 CPU 수를 그대로 사용했으나
- go 1.25 이후부터 CPU limit을 기본값으로 사용
    - 따라서 컨테이너 환경에서 똑똑해졌음
    - 특히 k8s에서 node의 cpu 수를 그대로 쓰는 것이 아닌 해당 pod의 cpu limit을 사용함