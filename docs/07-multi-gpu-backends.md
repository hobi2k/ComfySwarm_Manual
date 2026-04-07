# 07. 멀티 GPU 및 멀티 서버 백엔드 구성

이 문서는 여러 대의 ComfyUI 서버를 SwarmUI 백엔드로 연결해 병렬 처리 환경을 구성하는 방법을 정리한다.

관련 문서:

- 단일 서버 설치와 첫 백엔드 연결: [02. 서버 설치 및 연결](./02-server-installation.md)
- 백엔드 운영 메뉴: [04. 운영 도구와 API](./04-admin-operations.md)
- 원격 접속 보안: [05. 원격 접속과 보안](./05-remote-access-security.md)
- 장애 대응: [06. 문제 해결과 점검표](./06-troubleshooting.md)

## 1. 핵심 개념

SwarmUI는 **단일 접속점 + 다중 백엔드** 구조다.

```text
[작업자들]
    ↓ 브라우저로 접속 (SwarmUI 주소 하나)
[SwarmUI] ← 큐 관리, UI 제공
    ├── ComfyUI 백엔드 A (서버 1, GPU ×1)
    ├── ComfyUI 백엔드 B (서버 2, GPU ×1)
    └── ComfyUI 백엔드 C (서버 3, GPU ×2)
```

작업자가 이미지를 요청하면 SwarmUI가 현재 **유휴 상태인 백엔드**를 골라 요청을 넘긴다.
백엔드가 많을수록 동시에 처리할 수 있는 작업 수가 늘어난다.

### 분산의 단위: 작업(Job) 단위

분산의 단위는 **작업 단위**다.

- 이미지 1장 생성 → 한 백엔드가 전담
- 이미지 10장 배치 → 유휴 백엔드들이 나눠서 처리
- 한 이미지를 여러 GPU가 분할해서 빠르게 처리하는 방식은 아님

즉, **처리량(throughput)**이 늘어나는 구조다. 단일 작업의 생성 속도는 빨라지지 않는다.

## 2. 지원하는 멀티 백엔드 패턴

### 패턴 1: 같은 서버, 여러 GPU

GPU가 여러 장인 서버에서 각 GPU마다 별도의 ComfyUI 인스턴스를 띄우고, 각각을 백엔드로 연결한다.

```text
[서버 1대, GPU 4장]
  ├── ComfyUI :8188  (--cuda-device 0)
  ├── ComfyUI :8189  (--cuda-device 1)
  ├── ComfyUI :8190  (--cuda-device 2)
  └── ComfyUI :8191  (--cuda-device 3)
```

### 패턴 2: 여러 서버, 각 서버에 ComfyUI

서버가 여러 대 있을 때 각 서버의 ComfyUI를 백엔드로 붙인다.
Tailscale 또는 LAN으로 서로 연결되어 있어야 한다.

```text
[서버 A] ComfyUI :8188  → SwarmUI 백엔드로 등록
[서버 B] ComfyUI :8188  → SwarmUI 백엔드로 등록
[서버 C] ComfyUI :8188  → SwarmUI 백엔드로 등록
```

### 패턴 3: 다른 SwarmUI 인스턴스를 백엔드로

원격 서버에 SwarmUI까지 함께 설치된 경우, `Swarm-API-Backend` 타입으로 붙인다.
ComfyUI만 있는 경우와 달리 SwarmUI 레이어가 한 번 더 들어가는 구성이다.

## 3. 설정 방법

### 3-1. 각 ComfyUI 서버 준비

모든 ComfyUI 인스턴스에 아래 두 가지를 확인한다.

**Swarm 전용 커스텀 노드 설치**

```text
ComfyUI/custom_nodes/
  ├── SwarmComfyCommon
  └── SwarmComfyExtra
```

소스 위치:

```text
[SwarmUI 경로]/src/BuiltinExtensions/ComfyUIBackend/ExtraNodes/
```

ComfyUI `custom_nodes`에 복사하고 재시작한다.

**리슨 주소 설정**

SwarmUI가 외부에서 접근할 수 있도록 `--listen 0.0.0.0`을 실행 인수에 추가한다.

StabilityMatrix 기준:

1. ComfyUI 패키지 옆 `⋮` 클릭
2. `Launch Options` 진입
3. `--listen 0.0.0.0` 추가
4. 재시작

**포트 중복 방지 (같은 서버, 여러 인스턴스)**

같은 서버에서 ComfyUI를 여러 개 돌릴 때는 포트를 각각 다르게 지정한다.

```text
ComfyUI 인스턴스 1: --port 8188 --listen 0.0.0.0 --cuda-device 0
ComfyUI 인스턴스 2: --port 8189 --listen 0.0.0.0 --cuda-device 1
ComfyUI 인스턴스 3: --port 8190 --listen 0.0.0.0 --cuda-device 2
```

### 3-2. SwarmUI에서 백엔드 추가

1. `Server` → `Backends` 진입
2. `Add Backend` 클릭
3. 타입: `ComfyUI API By URL` 선택
4. 주소 입력

같은 서버, 포트가 다른 경우:

```text
http://127.0.0.1:8188
http://127.0.0.1:8189
```

다른 서버 (LAN):

```text
http://192.168.1.101:8188
http://192.168.1.102:8188
```

다른 서버 (Tailscale):

```text
http://100.x.x.2:8188
http://100.x.x.3:8188
```

5. 저장 후 상태가 `Connected`(녹색)인지 확인
6. 백엔드마다 반복

### 3-3. 백엔드 등록 완료 기준

아래를 모두 통과해야 정상이다.

1. 모든 백엔드 상태가 녹색이다.
2. 각 백엔드에서 테스트 생성 1건이 성공한다.
3. `Logs`에 missing node / missing model 오류가 없다.
4. 큐를 2건 이상 동시에 넣었을 때 두 백엔드가 동시에 처리되는 것이 보인다.

## 4. 반드시 지켜야 하는 운영 규칙

### 4-1. 모델 경로 통일

멀티 백엔드 환경에서 **가장 흔한 실패 원인**이다.

SwarmUI는 작업 요청 시 모델 이름을 백엔드에 그대로 넘긴다.
백엔드(ComfyUI)가 해당 이름의 파일을 찾지 못하면 생성이 실패한다.

모든 서버가 아래 조건을 만족해야 한다.

| 조건 | 설명 |
|------|------|
| 모델 파일명 동일 | 모든 서버에서 파일명이 완전히 일치 |
| 폴더 구조 동일 | `checkpoints/`, `loras/` 등 하위 폴더 구조 동일 |
| `ModelRoot` 경로 | SwarmUI `Server Configuration`에서 동일하게 설정 |

좋은 예:

```text
[서버 A] D:/AI/Models/checkpoints/sdxl_base.safetensors
[서버 B] D:/AI/Models/checkpoints/sdxl_base.safetensors
→ 파일명, 경로 구조 동일 → 정상 작동
```

나쁜 예:

```text
[서버 A] D:/AI/Models/checkpoints/sdxl_base.safetensors
[서버 B] E:/comfy/models/checkpoints/SDXL_latest.safetensors
→ 서버 A에서는 되고 서버 B에서는 실패
→ 어디서 실패했는지 찾기 어려움
```

이 규칙이 안 맞으면:

- 백엔드는 녹색이어도 특정 모델 요청이 배정된 백엔드에서 로드 실패가 난다.
- 작업자 입장에서는 "가끔 실패한다"는 증상으로 보인다.
- 어떤 백엔드에서는 되고 다른 백엔드에서는 안 되는 상태가 된다.

### 4-2. Swarm 커스텀 노드 버전 통일

모든 ComfyUI 인스턴스의 `SwarmComfyCommon`, `SwarmComfyExtra`는 같은 SwarmUI 버전에서 복사한 것을 써야 한다.

버전이 다르면 워크플로 호환 문제나 노드 동작 차이가 생길 수 있다.

SwarmUI를 업데이트하면 모든 서버의 커스텀 노드도 함께 갱신한다.

### 4-3. 큐 배분 동작 이해

- 큐가 1건씩 들어오면 백엔드 1개만 일하고 나머지는 유휴 상태다.
- 동시 요청 수가 백엔드 수보다 많아야 실질적인 분산이 이루어진다.
- 백엔드 목록 순서가 배분에 영향을 준다. 더 빠른 서버를 위에 두는 것이 유리하다.
- 백엔드가 중간에 죽으면 해당 큐 항목은 실패한다. 자동 재시도 없다.

## 5. 같은 서버에서 멀티 GPU 운영 추가 설정

이 섹션은 **컴퓨터 한 대에 GPU가 여러 장** 있는 경우의 설정이다.
GPU마다 별도의 ComfyUI 프로세스를 띄우고, 각각을 SwarmUI 백엔드로 붙이는 방식이다.

### GPU 번호 확인

NVIDIA GPU 번호는 서버에서 아래 명령으로 확인한다.

```bash
nvidia-smi
```

출력 결과에서 각 GPU의 인덱스(0, 1, 2...)와 VRAM 용량을 확인한다.

### StabilityMatrix에서 GPU별 ComfyUI 인스턴스 만들기

StabilityMatrix는 **같은 패키지를 이름만 달리해서 여러 번 설치할 수 있다.**
GPU별로 별도 패키지를 만들고 각각 다른 포트와 GPU를 지정하는 것이 권장 방법이다.

설정 예시:

| 패키지 이름 | 포트 | Launch Option |
|---|---|---|
| `ComfyUI-GPU0` | `8188` | `--listen 0.0.0.0 --cuda-device 0` |
| `ComfyUI-GPU1` | `8189` | `--listen 0.0.0.0 --cuda-device 1` |

절차:

1. StabilityMatrix `Packages` → `+ Add Package` → ComfyUI 선택
2. 패키지 이름을 `ComfyUI-GPU0`처럼 구분이 되게 짓는다.
3. 설치 후 `⋮` → `Launch Options`에서 `--listen 0.0.0.0 --cuda-device 0 --port 8188` 추가
4. GPU 1번용은 같은 방법으로 `ComfyUI-GPU1`을 별도 설치, `--cuda-device 1 --port 8189` 지정
5. 각 패키지에 `SwarmComfyCommon`, `SwarmComfyExtra`를 각각 설치한다.
6. 두 패키지를 모두 실행한 뒤 SwarmUI 백엔드에 `http://127.0.0.1:8188`, `http://127.0.0.1:8189`를 각각 추가한다.

주의: 모델 폴더는 두 패키지가 같은 경로를 바라봐야 한다. StabilityMatrix의 공유 `Models` 폴더를 쓰면 자연스럽게 해결된다.

## 6. Tailscale 기반 멀티 서버 운영 시 주의사항

서버가 물리적으로 다른 장소에 있을 경우 Tailscale로 연결하는 것이 안전하다.

- SwarmUI가 설치된 메인 서버와 추가 ComfyUI 서버 모두 같은 Tailscale 네트워크(tailnet)에 있어야 한다.
- SwarmUI → 추가 ComfyUI 서버 방향으로의 접속이기 때문에, 추가 서버의 방화벽에서 `8188` 포트를 Tailscale 대역에서 허용해야 한다.

```powershell
New-NetFirewallRule -DisplayName "ComfyUI-Tailscale" -Direction Inbound -Protocol TCP -LocalPort 8188 -Profile Private -RemoteAddress 100.64.0.0/10 -Action Allow
```

- Tailscale 연결이 릴레이(DERP)를 거치면 레이턴시가 늘어날 수 있다. P2P 직접 연결 상태인지 Tailscale 관리 콘솔에서 확인한다. 공유기 UPnP 활성화 또는 UDP 포트 41641 개방으로 P2P 연결을 개선할 수 있다.
- MagicDNS를 켜면 IP 대신 기기 이름으로 백엔드 주소를 관리할 수 있어 IP가 바뀌어도 재설정이 불필요하다.

## 7. 한계 정리

| 항목 | 실제 동작 |
|------|-----------|
| 단일 이미지 생성 속도 | 빨라지지 않음. 한 이미지는 한 백엔드가 전담 |
| 동시 처리 가능 수 | 백엔드 수만큼 늘어남 |
| 모델 자동 동기화 | 없음. 수동으로 모든 서버에 동일하게 맞춰야 함 |
| 백엔드 자동 복구 | 없음. 백엔드가 죽으면 해당 큐 항목은 실패 |
| 작업 자동 재시도 | 없음. 실패한 작업은 수동 재요청 필요 |
| 부하 기반 스마트 배분 | 없음. 큐 순서 기반으로 유휴 백엔드에 배정 |

## 8. 멀티 백엔드 배포 체크리스트

배포 전에 아래를 모두 확인한다.

- 모든 서버의 모델 파일명과 폴더 구조가 동일한가
- 모든 ComfyUI에 `SwarmComfyCommon`, `SwarmComfyExtra`가 설치되어 있는가
- 모든 ComfyUI가 `--listen 0.0.0.0`으로 실행 중인가
- 멀티 GPU 구성이라면 `--cuda-device`와 포트가 각각 다른가
- 모든 백엔드가 SwarmUI에서 녹색인가
- 각 백엔드에서 단독 테스트 생성 1건이 성공하는가
- 큐 2건 이상 동시에 넣었을 때 두 백엔드가 함께 처리되는가
- 방화벽 규칙이 Tailscale 또는 LAN 범위만 허용하는가
- 서버 절전 모드가 비활성화되어 있는가

## 9. 트러블슈팅

### "어떤 요청은 되고 어떤 요청은 실패한다"

원인은 대부분 **모델 경로 불일치**다.

1. `Server → Logs`에서 모델 로드 실패 메시지를 확인한다.
2. 실패한 백엔드의 ComfyUI 로그를 직접 확인한다.
3. 해당 모델 파일이 실패한 서버에 있는지, 경로와 파일명이 정확히 같은지 확인한다.

### "백엔드가 갑자기 빨간색이 된다"

1. 해당 서버의 ComfyUI가 살아 있는지 확인한다.
2. 절전 모드로 들어간 건 아닌지 확인한다.
3. `Server → Backends`에서 해당 백엔드를 재시작한다.
4. 네트워크 경로(Tailscale 연결 상태)를 확인한다.

### "큐를 많이 넣어도 한 백엔드만 일한다"

SwarmUI는 한 작업이 끝나야 다음 작업을 해당 백엔드에 배정한다.
실제 분산이 보이려면 **여러 작업이 동시에 큐에 들어가 있어야 한다**.
순차적으로 1건씩 넣으면 분산이 보이지 않는다.

## 다음 읽을 문서

- 단일 백엔드 설치와 연결: [02. 서버 설치 및 연결](02-server-installation.md)
- 백엔드 운영 메뉴 전체: [04. 운영 도구와 API](04-admin-operations.md)
- 원격 접속과 방화벽: [05. 원격 접속과 보안](05-remote-access-security.md)
- 장애 대응: [06. 문제 해결과 점검표](06-troubleshooting.md)
