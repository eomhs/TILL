## Github Action에서 Docker 빌드 캐싱 하는법
```yaml
 name: Build and push
        uses: docker/build-push-action@v2
        with:
            ...
            cache-from: type=gha
            cache-to: type=gha,mode=max
```
- type은 local, registry, gha 등이 있는데 Github action cache의 10GB 제한이 충분하다면 gha가 cache export가 제일 빠르다
## 빌드 캐싱이 잘 안 될 때 확인해보기
- cache scope가 잘 되어 있는지
- docker cache layer들이 정확히 같은지
    - 특히 ARG나 ENV가 달라지면 달라져서 ARG는 사용하는 RUN 바로 위에 쓰는것이 좋음
    - https://docs.docker.com/reference/dockerfile/#impact-on-build-caching