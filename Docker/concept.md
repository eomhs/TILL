 # Docker & Linux 컨테이너 핵심 정리

---

## 1. Docker Image

이미지는 컨테이너를 실행하기 위한 **읽기전용 파일시스템 스냅샷**이다.
실체는 `/var/lib/docker/overlay2/` 아래의 디렉토리 묶음이며, 별도의 실행 단위가 아니라 그냥 파일들의 집합이다.

### 레이어 구조

Dockerfile의 각 명령어(`FROM`, `COPY`, `RUN` 등)마다 **파일시스템 diff 스냅샷**이 하나의 레이어로 저장된다.

```
FROM golang:1.22-alpine   → alpine OS 레이어 + Go 런타임 레이어 (읽기전용)
WORKDIR /app              → /app 디렉토리 레이어 (읽기전용)
COPY . .                  → 소스코드 레이어 (읽기전용)
RUN go build -o app .     → 바이너리 레이어 (읽기전용)
CMD ["/app/app"]          → 레이어 없음, config JSON에 메타데이터만 저장
```

- **캐시 무효화**: 상위 레이어가 바뀌면 그 아래 모든 레이어의 캐시가 날아간다.
- **레이어 식별**: 내용의 `sha256` 해시로 식별 → 같은 레이어는 여러 이미지가 공유 가능.
- `CMD`, `ENV`, `EXPOSE`는 파일을 건드리지 않으므로 레이어를 생성하지 않는다.

### 로컬 캐시와 pull

`docker build` 시 `FROM`을 만나면:
1. 로컬 캐시(`/var/lib/docker/overlay2/`) 확인
2. 있으면 네트워크 없이 재사용
3. 없으면 Docker Hub(또는 registry)에서 레이어 pull 후 저장

---

## 2. docker build — 임시 컨테이너

각 `RUN` 명령어마다:
1. OverlayFS 마운트 (`lowerdir` = 이전 레이어들, `upperdir` = 빈 새 디렉토리)
2. `clone(CLONE_NEWNS | CLONE_NEWPID | ...)` 로 격리된 임시 프로세스 생성
3. 명령어 실행 — 변경사항은 `upperdir`에만 기록
4. `upperdir` 내용을 새 읽기전용 레이어로 커밋
5. 임시 컨테이너 삭제

---

## 3. docker run — 컨테이너 실행

```
이미지 레이어들 (읽기전용)
        ↓
OverlayFS 마운트 (lowerdir + upperdir)
        ↓
merged/ → 프로세스의 / (pivot_root)
        ↓
exec → 컨테이너 프로세스 (PID 1)
```

1. **OverlayFS 마운트**: 이미지 레이어들을 `lowerdir`에 쌓고, 새 R/W 레이어를 `upperdir`에 얹어 `merged/`라는 통합 뷰 생성
2. **mnt namespace + pivot_root**: `merged/`를 이 프로세스의 `/`로 교체
3. **exec**: 격리된 환경 안에서 바이너리 실행

컨테이너를 종료하면 R/W 레이어(변경사항)는 사라진다.

---

## 4. 컨테이너 = 격리된 프로세스

컨테이너는 VM이 아니다. **Linux 커널 기능에 격리 플래그를 달고 태어난 평범한 프로세스**다.

```bash
# 호스트에서 보면
ps aux | grep nginx   # PID 3847 — 일반 프로세스

# 컨테이너 안에서 보면
ps aux                # PID 1 — 같은 프로세스, 다르게 보임
```

Docker가 내부적으로 하는 것:
```c
clone(child_fn, CLONE_NEWPID | CLONE_NEWNS | CLONE_NEWNET | ..., ...);
```
일반 `fork()`와 다른 건 `CLONE_NEW*` 플래그뿐이다.

---

## 5. Linux Namespace — 격리 수단

| namespace | 격리 대상 |
|-----------|-----------|
| `pid` | 프로세스 ID — 컨테이너 안에서 자신이 PID 1로 보임 |
| `mnt` | 파일시스템 마운트 — 각 컨테이너가 독립된 `/` 가짐 |
| `net` | 네트워크 인터페이스 — 컨테이너마다 독립 IP |
| `uts` | hostname · domainname |
| `ipc` | 공유 메모리 · 세마포어 |
| `user` | UID · GID |

namespace는 **"보이는 것"만 격리**한다. 자원 사용량 제한(CPU, 메모리)은 **cgroup**이 담당한다.

> 컨테이너 = namespace(격리) + cgroup(제한) + OverlayFS(파일시스템)

---

## 6. Linux vs macOS/Windows에서의 Docker

| | Linux | macOS / Windows |
|---|---|---|
| 커널 | 호스트 커널 직접 사용 | 내부 Linux VM 커널 사용 |
| VM | 없음 | Apple Hypervisor / WSL2 |
| `/var/lib/docker/` | 호스트 파일시스템에 존재 | VM 내부에만 존재 |
| 컨테이너 기동 속도 | ms 단위 | VM 오버헤드 있음 |

macOS/Windows는 Linux 커널이 없어서 namespace, OverlayFS를 쓸 수 없다. Docker Desktop이 경량 Linux VM을 내부적으로 띄우고 그 안에서 Docker daemon을 실행한다.

---

## 7. OS = 커널 + 유저랜드

컨테이너에서 "다른 OS"처럼 보이는 이유:

- **커널**: 모든 컨테이너가 호스트 Linux 커널 하나를 공유
- **유저랜드**: 각 이미지의 레이어에 담긴 파일들(`/bin`, `/lib`, `/etc` 등)이 다름

`FROM ubuntu`나 `FROM alpine`을 써도 커널은 교체되지 않는다. 유저랜드만 바뀌는 것이다.
`FROM windows`가 Linux 호스트에서 불가능한 이유는 Windows NT 커널의 syscall을 Linux가 처리할 수 없기 때문이다.

---

## 8. Alpine vs Ubuntu

두 이미지의 가장 중요한 차이는 **C 표준 라이브러리(libc)** 다.

| | Alpine | Ubuntu |
|---|---|---|
| 이미지 크기 | ~5 MB | ~80 MB |
| libc | **musl** | **glibc** |
| 기본 쉘 | `/bin/sh` (busybox ash) | `bash` |
| 패키지 매니저 | `apk` | `apt` |

glibc로 동적 링크된 바이너리(Python, Node.js, JVM 등)는 Alpine에서 실행하면 오류가 난다.
Go처럼 **정적 바이너리**를 만드는 언어는 libc 의존성이 없어서 Alpine이 최적이다.

---

## 9. 모든 것은 파일이다

Linux의 핵심 철학: **Everything is a file**

| 종류 | 예시 | 설명 |
|---|---|---|
| 일반 파일 | `/lib/libc.so.6` | 디스크에 저장 |
| 디렉토리 | `/etc/` | 파일의 목록 |
| 디바이스 파일 | `/dev/sda`, `/dev/null` | 하드웨어 드라이버와 연결 |
| 가상 파일 | `/proc/meminfo`, `/proc/$$` | 커널이 실시간 생성, 디스크에 없음 |
| 소켓/파이프 | `/var/run/docker.sock` | IPC 수단 |

라이브러리도 파일이다. 프로그램 실행 시 동적 링커가 `/lib/libc.so.6`을 `open()`으로 열어 메모리에 매핑한다.
Docker 이미지 레이어가 담고 있는 것도 결국 이 파일들의 묶음이다.

---

## 10. Multi-stage build 패턴

Go 바이너리는 정적 링크되므로 `scratch`(완전 빈 이미지)에 바이너리만 복사해 배포할 수 있다.

```dockerfile
# Stage 1: 빌드 (Go 컴파일러 포함, 크기 큼)
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o app .

# Stage 2: 실행 (바이너리만, 수 MB)
FROM scratch
COPY --from=builder /app/app /app
CMD ["/app"]
```

최종 이미지에 컴파일러, 소스코드, 중간 파일이 남지 않는다.