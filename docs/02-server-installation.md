# 02. 서버 설치 및 연결

이 장은 `SwarmUI`를 회사 작업자용 웹 UI로 쓰기 위한 서버 구축 절차를 처음부터 끝까지 정리한다.  
목표는 다음과 같다.

- GPU 서버에 `StabilityMatrix`, `ComfyUI`, `SwarmUI`를 설치한다.
- `StabilityMatrix`가 관리하는 `ComfyUI`를 SwarmUI의 외부 백엔드로 연결한다.
- 작업자 PC는 `Tailscale`로 서버에 안전하게 접속한다.
- 설치 직후 바로 검증할 수 있는 확인 절차까지 남긴다.

SwarmUI 공식 문서는 `ComfyUI Self-Starting`을 권장한다.  
그러나 이 회사 매뉴얼은 `StabilityMatrix`가 서버 측 `ComfyUI`를 관리하는 구조이므로 `ComfyUI API By URL`을 쓴다.  
이 방식에서는 외부 ComfyUI 백엔드용 필수 Swarm 노드 `SwarmComfyCommon`과 `SwarmComfyExtra`를 직접 넣어야 한다.

참고:

- 예전 글이나 일부 UI 설명에서는 같은 개념을 `ComfyUI API (External)`라고 부르기도 한다.
- 이 문서에서는 공식 소스 기준 명칭인 `ComfyUI API By URL`로 통일한다.

## 0. 설치 전에 결정할 것

설치 전에 아래 세 가지를 먼저 정한다.

1. 어떤 PC가 GPU 서버인지
2. `StabilityMatrix` 데이터 폴더를 어느 드라이브에 둘지
3. 작업자들이 어떤 방식으로 들어올지
   - 사내 같은 네트워크면 LAN
   - 외부 접속이 섞이면 `Tailscale`

권장 예시는 아래와 같다.

- 서버 경로: `D:\AI\StabilityMatrix`
- StabilityMatrix 데이터 폴더: `D:\AI\StabilityMatrix\Data`
- ComfyUI 포트: `8188`
- SwarmUI 포트: `7801`

## 1. 무엇을 어디에 둘까

- `GPU 서버`
  - `StabilityMatrix`
  - `ComfyUI`
  - `SwarmUI`
  - 모델 폴더
  - 커스텀 노드
- `작업자 PC`
  - 브라우저
  - Tailscale 클라이언트
  - 필요하면 API 토큰을 쓸 자동화 스크립트

작업자 PC가 강할 필요는 없다. 서버가 계산을 맡기 때문이다.

## 1-1. 설치 완료 후 최종 구조

```text
[GPU 서버]
  └── StabilityMatrix
        ├── Packages
        │   ├── ComfyUI
        │   │   └── custom_nodes
        │   │       ├── SwarmComfyCommon
        │   │       └── SwarmComfyExtra
        │   └── SwarmUI
        ├── Models
        └── Output

[작업자 PC]
  ├── Browser
  └── Tailscale
```

## 2. 설치 원칙

이 매뉴얼은 다음 전제를 따른다.

- `ComfyUI`는 GPU 서버에서 `StabilityMatrix`가 설치하고 실행한다.
- 작업자들은 자기 PC에서 무거운 생성 작업을 직접 돌리지 않는다.
- 작업자는 브라우저로 서버의 `SwarmUI`에 접속한다.
- `SwarmUI`는 서버 측의 외부 `ComfyUI` 백엔드에 붙는다.
- `Self-Starting`은 공식 권장 경로이지만, 이 매뉴얼은 `StabilityMatrix`가 관리하는 서버 측 `ComfyUI`를 유지하는 외부 백엔드 구성을 기준으로 한다.

## 3. 서버에 ComfyUI 설치

1. `StabilityMatrix`를 서버 PC에서 실행한다.
2. `Packages`에서 `ComfyUI`를 설치한다.
3. 한 번 실행해서 `http://127.0.0.1:8188`가 뜨는지 본다.
4. 모델 폴더가 서버에 제대로 붙어 있는지 확인한다.

권장 사항:

- 설치 경로에 한글과 공백을 넣지 않는다.
- 데이터 폴더는 용량이 충분한 SSD 또는 빠른 드라이브에 둔다.
- 모델이 많다면 `Portable` 성격으로 한 곳에 몰아 두는 편이 관리가 쉽다.

모델 경로가 이미 여러 개면 `SwarmUI`의 `ModelRoot`에 세미콜론(`;`)으로 여러 루트를 넣을 수 있다.  
같은 모델을 여러 백엔드가 읽어야 하면 파일명과 폴더 구조를 맞춰 둔다.

### 3-1. ComfyUI 1차 검증

아래 네 가지가 맞으면 1차 검증 통과다.

1. `http://127.0.0.1:8188` 접속이 된다.
2. ComfyUI 기본 화면이 뜬다.
3. 에러 없이 프로세스가 유지된다.
4. `custom_nodes` 폴더 위치를 찾을 수 있다.

### 3-2. ComfyUI 리슨 주소 설정

관리자 확인이나 원격/API 방식 운영이 필요하면 ComfyUI 실행 인수를 설정한다.

StabilityMatrix 기준 절차:

1. `Packages`에서 `ComfyUI`를 찾는다.
2. 패키지 오른쪽 `⋮` 메뉴를 연다.
3. `Launch Options` 또는 `Extra Launch Arguments`에 들어간다.
4. 아래 인수를 추가한다.

```text
--listen 0.0.0.0
```

운영자 기록용으로 아래를 같이 남기는 편이 좋다.

- 누가 이 설정을 넣었는가
- 언제 넣었는가
- 어떤 포트(`8188`)를 쓰는가
- 방화벽에서 어떤 범위만 허용하는가

주의:

- 같은 서버 안에서 `SwarmUI`가 `127.0.0.1:8188`로만 붙는 구조라면 이 설정이 항상 절대 필수인 것은 아니다.
- 하지만 이 매뉴얼처럼 `ComfyUI API By URL`, 원격 관리자 점검, Tailscale 기반 확인까지 고려하면 미리 열어 두는 편이 운영상 안정적이다.
- 이 설정은 공용 인터넷 공개를 의미하지 않는다.

## 4. SwarmUI 설치와 접속

1. 같은 서버에서 `SwarmUI`를 설치한다.
2. 첫 실행 마법사가 뜨면 초기 선택을 진행한다.
3. 첫 설정이 끝난 뒤 `Generate` 화면이 뜨는지 확인한다.
4. 서버 PC에서 `SwarmUI`가 열리면, 작업자 PC는 그 주소로 접속한다.

### 4-1. SwarmUI 첫 실행 시 권장 선택

참고: SwarmUI는 .NET 런타임이 필요하다. 설치 시 없으면 자동으로 안내된다.

이 문서 기준으로는 아래처럼 선택한다.

1. 백엔드 자동 설치/내장 백엔드 선택 단계에서는 새 ComfyUI를 추가로 설치하지 않는다.
2. 이미 `StabilityMatrix`가 관리하는 ComfyUI를 쓸 것이므로 `None`, `Skip`, 또는 그에 준하는 선택지를 택한다.
3. 모델 경로 선택 단계가 보이면 StabilityMatrix 데이터 폴더의 모델 저장소를 기준으로 맞춘다.
4. ComfyUI 스타일 폴더를 공유한다면 이후 `Server -> Server Configuration -> Paths`에서 `ModelRoot`는 StabilityMatrix 모델 루트로, `SDModelFolder`는 `checkpoints` 기준으로 맞출 준비를 한다.
5. 일단 설치를 끝내고 메인 UI 진입부터 확인한다.

설치 직후 다음 메뉴로 바로 이동한다.

1. `Server`
2. `Server Configuration`
3. `Network` 관련 그룹
4. `Host`, `Port` 확인
5. `Paths` 관련 그룹에서 모델 루트 확인
6. `ModelRoot`를 StabilityMatrix 모델 저장소로 맞춘다.
7. `SDModelFolder`가 ComfyUI 스타일 공유 환경이면 `checkpoints`인지 확인한다.

예시:

- `ModelRoot = D:/AI/StabilityMatrix/Data/Models`
- `SDModelFolder = checkpoints`

원격 또는 LAN 접속이 필요하면 `Server` -> `Server Configuration`에서 `Host`를 `0.0.0.0`으로 두고 재시작한다.  
SwarmUI 공식 문서도 LAN 접속은 이 경로를 권장한다.

```text
--host 0.0.0.0
--port 7801
```

참고:

- 공식 CLI 문서에는 Windows 개발 예시로 `--host *`가 보이기도 한다.
- 운영 문서 기준으로는 `Server Configuration`의 `Host = 0.0.0.0` 설명이 더 직관적이므로 이 문서에서는 그 표현을 기준으로 쓴다.

StabilityMatrix 패키지 실행 인수로 직접 관리하려면 아래처럼 정리한다.

1. `Packages`에서 `SwarmUI`를 찾는다.
2. 패키지 오른쪽 `⋮` 메뉴를 연다.
3. `Launch Options` 또는 `Extra Launch Arguments` 성격의 항목으로 들어간다.
4. 아래 인수를 추가한다.

```text
--host 0.0.0.0
--port 7801
```

실무 기준:

- UI 설정과 실행 인수 중 무엇을 기준으로 관리할지 팀에서 하나로 정한다.
- 두 군데를 동시에 만지면 나중에 누가 실제 값을 덮어쓰는지 헷갈릴 수 있다.

ComfyUI를 외부 API 백엔드로 붙이거나 다른 기기에서 직접 접속해야 할 때는 ComfyUI 쪽도 리슨 주소를 넓힌다.

```text
--listen 0.0.0.0
```

### 4-2. SwarmUI 1차 검증

아래 항목을 확인한다.

1. `http://127.0.0.1:7801` 접속이 된다.
2. 메인 UI가 뜬다.
3. `Server` 메뉴가 보인다.
4. `Generate` 탭이 열린다.
5. `Server -> Server Configuration`에 들어갈 수 있다.

## 5. 외부 ComfyUI 백엔드에 필요한 필수 Swarm 노드 설치

`ComfyUI API By URL`을 쓸 때는 Swarm 전용 노드를 ComfyUI에 직접 넣어야 한다.  
이 단계를 빼면 백엔드가 붙더라도 생성이 실패하거나, 특정 워크플로가 로드되지 않거나, 저장 노드가 없어서 결과가 깨질 수 있다.

필수 폴더:

- `SwarmComfyCommon`
- `SwarmComfyExtra`

소스 위치:

```text
[StabilityMatrix 데이터 폴더]\Packages\SwarmUI\src\BuiltinExtensions\ComfyUIBackend\ExtraNodes\
```

대상 위치:

```text
[StabilityMatrix 데이터 폴더]\Packages\ComfyUI\custom_nodes\
```

경로 예시:

```text
D:\AI\StabilityMatrix\Data\Packages\SwarmUI\src\BuiltinExtensions\ComfyUIBackend\ExtraNodes
D:\AI\StabilityMatrix\Data\Packages\ComfyUI\custom_nodes
```

설치 절차:

1. SwarmUI 패키지 폴더에서 `ExtraNodes` 위치를 연다.
2. `SwarmComfyCommon` 폴더를 ComfyUI `custom_nodes`에 복사한다.
3. `SwarmComfyExtra` 폴더도 같은 위치에 복사한다.
4. ComfyUI를 완전히 종료했다가 다시 실행한다.

복사 후 구조 예시는 아래와 같다.

```text
[StabilityMatrix 데이터 폴더]
└── Packages
    ├── ComfyUI
    │   └── custom_nodes
    │       ├── SwarmComfyCommon
    │       └── SwarmComfyExtra
    └── SwarmUI
        └── src
            └── BuiltinExtensions
                └── ComfyUIBackend
                    └── ExtraNodes
```

### 5-1. 필수 노드 검증

ComfyUI 재시작 후 아래를 확인한다.

1. 노드 검색에서 `SwarmSaveImageWS`가 보인다.
2. 필요한 경우 `SwarmSaveAnimationWS`도 보인다.
3. 오류 로그 없이 ComfyUI가 올라온다.
4. 이후 SwarmUI에서 백엔드를 붙였을 때 `Connected`가 뜬다.

## 6. 백엔드 연결

1. `Server` -> `Backends`로 들어간다.
2. `Add Backend`를 누른다.
3. 타입에서 `ComfyUI API By URL` 또는 필요한 다른 백엔드 타입을 고른다.
4. 주소를 입력한다.

```text
http://127.0.0.1:8188
```

5. 저장 후 상태가 `Connected`가 되는지 확인한다.

중요:

- `ComfyUI API By URL`은 SwarmUI가 ComfyUI를 직접 설치/업데이트/복구하지 않는다.
- ComfyUI 업데이트, Python 환경, 커스텀 노드 설치, 패키지 복구 책임은 운영자에게 있다.
- 백엔드가 초록색이어도 실제 생성 검증 전까지는 정상 운용으로 보면 안 된다.

백엔드는 여러 개 붙일 수 있다.

- 같은 서버에서 GPU가 여러 개면 백엔드를 여러 개 둘 수 있다.
- 다른 서버가 있으면 그 서버를 또 하나의 백엔드로 붙일 수 있다.
- 백엔드 순서는 큐 배분에 영향을 준다.

### 6-1. 백엔드 연결 검증

아래 항목이 모두 맞아야 한다.

1. `Server -> Backends`에서 상태가 `Connected` 또는 녹색으로 보인다.
2. 테스트 생성 1건이 성공한다.
3. 모델 목록이 비어 있지 않다.
4. ComfyUI 쪽 로그에 모델 없음/노드 없음 오류가 없다.

### 6-2. 모델 경로 일치 규칙

`ComfyUI API By URL`에서는 SwarmUI와 ComfyUI가 같은 모델을 같은 논리 이름으로 봐야 한다.

운영 규칙:

1. 같은 모델 파일을 같은 저장소에서 읽게 한다.
2. 상대 경로와 폴더 구조를 맞춘다.
3. 파일명이 달라지면 SwarmUI 쪽 모델 이름도 달라질 수 있다고 본다.
4. Windows에서 `\`와 `/` 차이는 허용되더라도, 논리 경로 구조는 같아야 한다.

실무 해석:

- SwarmUI가 어떤 모델을 `OfficialStableDiffusion/sd_xl_base_1.0`로 보고 있다면
- ComfyUI 쪽에서도 그에 대응되는 같은 모델을 같은 폴더 체계에서 찾아야 한다.

좋은 예시:

```text
SwarmUI ModelRoot: D:/AI/StabilityMatrix/Data/Models
ComfyUI models root: D:/AI/StabilityMatrix/Data/Models
실제 파일: D:/AI/StabilityMatrix/Data/Models/checkpoints/sd_xl_base_1.0.safetensors
```

나쁜 예시:

```text
SwarmUI ModelRoot: D:/AI/StabilityMatrix/Data/Models
ComfyUI models root: E:/Comfy/models
SwarmUI 쪽 파일명: checkpoints/sd_xl_base_1.0.safetensors
ComfyUI 쪽 파일명: checkpoints/SDXL_base_latest.safetensors
```

위와 같은 불일치가 있으면 UI에는 모델이 보여도 실제 생성 시 로드 실패가 날 수 있다.

이 규칙이 안 맞으면:

- 백엔드는 붙어도 생성 시 모델 로드 실패가 날 수 있다.
- 어떤 백엔드에서는 되고 어떤 백엔드에서는 안 되는 상태가 된다.
- 이름은 같은데 실제 다른 파일을 읽는 문제가 생긴다.

## 7. 모델 경로만 먼저 맞춘다

이 장에서 중요한 것은 “모델을 어디에 둘지”다.  
실제 다운로드 방법, 메타데이터 재검사, `Model Downloader`, `Reset All Metadata` 같은 운영 도구는 [04. 운영 도구와 API](04-admin-operations.md)에서 따로 설명한다.

여기서는 서버 설정에서 모델 경로만 먼저 맞춘다.

- `ModelRoot`는 기본 모델 루트다.
- `SDModelFolder`는 하위 폴더 이름이다.
- 여러 루트를 쓰면 첫 번째가 우선이다.

## 8. Tailscale로 연결하기

1. 서버 PC와 작업자 PC에 `Tailscale`을 설치한다.
2. 같은 계정으로 로그인한다.
3. 서버의 Tailscale 주소 또는 MagicDNS 이름을 확인한다.
4. 작업자 PC에서 그 주소의 `7801` 포트로 접속한다.

이 방식이면 포트 포워딩 없이 작업자를 붙일 수 있다.

### 8-1. 원격 접속 검증

작업자 PC에서 아래를 확인한다.

1. Tailscale이 로그인 상태다.
2. 서버 `MagicDNS` 또는 `100.x.x.x` 주소가 보인다.
3. `http://서버주소:7801` 접속이 된다.
4. 로그인 후 `Generate` 탭이 열린다.
5. `AuthorizationRequired`를 켰다면 로그인 화면이 먼저 뜬다.
6. 일반 작업자 계정으로 들어갔을 때 의도한 탭만 보인다.

### 8-2. 작업자 수용 테스트

배포 직후 실제 작업자 계정으로 아래를 한다.

1. 로그인한다.
2. 지정 모델 1개가 목록에 보이는지 확인한다.
3. 테스트 생성 1건을 보낸다.
4. 결과가 저장되는지 확인한다.
5. 문제 발생 시 운영자에게 넘기는 경로를 확인한다.

## 9. 여러 GPU와 여러 서버

SwarmUI는 여러 백엔드를 묶어 병렬 처리할 수 있다.  
`Using More GPUs` 문서의 핵심은 다음과 같다.

- 같은 서버 안의 여러 GPU도 백엔드로 나눌 수 있다.
- 다른 머신도 백엔드로 붙일 수 있다.
- 큐는 등록된 백엔드 순서에 따라 흘러간다.

즉, `SwarmUI`는 작업자 UI이고, 실제 성능 확장은 백엔드 추가로 해결한다.

## 10. API는 운영 도구 장에서 본다

외부 스크립트나 연동 도구가 필요하면 SwarmUI의 `/API/...` 구조를 쓰게 된다.  
다만 이 장은 서버를 세우고 연결하는 단계이므로, 실제 API 사용 흐름과 `GetNewSession`, `GenerateText2Image`, 인증 쿠키, `APIRoutes` 구분은 [04. 운영 도구와 API](04-admin-operations.md)에서 따로 설명한다.

## 11. 설치 완료 체크리스트

배포 전에 아래를 모두 체크한다.

- `ComfyUI`가 `127.0.0.1:8188`에서 열린다.
- `SwarmUI`가 `127.0.0.1:7801`에서 열린다.
- `SwarmComfyCommon`, `SwarmComfyExtra`가 `custom_nodes`에 있다.
- `SwarmSaveImageWS`가 ComfyUI에서 보인다.
- `Server -> Backends`가 녹색이다.
- 테스트 생성 1건이 성공한다.
- 작업자 PC에서 Tailscale로 SwarmUI 접속이 된다.

## 다음 읽을 문서

- 역할과 탭 구조: [01. 개요](01-overview.md)
- 탭별 사용법: [03. 작업자 탭 가이드](03-worker-tabs.md)
- 운영 메뉴와 API: [04. 운영 도구와 API](04-admin-operations.md)
- 공유와 권한: [05. 원격 접속과 보안](05-remote-access-security.md)
- 장애 대응: [06. 문제 해결과 점검표](06-troubleshooting.md)
