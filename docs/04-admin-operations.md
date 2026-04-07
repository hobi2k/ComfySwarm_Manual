# 04. 운영 도구와 API

이 문서는 SwarmUI를 실제 서비스처럼 운영할 때 필요한 메뉴와 도구를 정리한다.
여기서 다루는 범위는 `Utilities`, `User`, `Server`, `Backends`, `Extensions`, `Logs`, `Server Configuration`, 모델 다운로드와 메타데이터, 그리고 API다.

이 문서의 전제는 분명하다.

- 작업자는 `Generate`와 `Simple`을 주로 쓴다.
- 워크플로 설계는 빌더나 파워 유저의 역할이다.
- 운영자는 모델 경로, 백엔드, 권한, 로그, 확장, API를 관리한다.
- `ComfyUI`를 외부 백엔드로 붙일 때는 Swarm 전용 커스텀 노드가 필요하다.

관련 문서:

- 작업자용 탭은 [03. 작업자 탭 가이드](./03-worker-tabs.md)에서 본다.
- 서버 설치와 백엔드 연결은 [02. 서버 설치 및 연결](./02-server-installation.md)에서 본다.
- 원격 접속과 보안은 [05. 원격 접속과 보안](./05-remote-access-security.md)에서 본다.
- 장애 대응은 [06. 문제 해결과 점검표](./06-troubleshooting.md)에서 본다.

## 1. 이 문서에서 다루는 것

SwarmUI는 단순한 이미지 생성 버튼이 아니다. 서버, 모델, 사용자, 백엔드, API, 로그까지 같이 움직이는 운영 도구다.

이 문서가 답하는 질문은 다음과 같다.

- 모델은 어디에 넣는가?
- 모델이 잘못 인식되면 어디서 고치는가?
- 백엔드는 어떻게 추가하고 분산하는가?
- 사용자 계정과 권한은 어디서 관리하는가?
- 외부 프로그램은 어떤 API로 붙는가?
- `ComfyUI` 외부 백엔드를 쓸 때 필요한 Swarm 노드는 무엇인가?

## 2. 화면 구조 먼저 보기

운영자가 반드시 알아야 하는 화면은 다음과 같다.

| 메뉴 | 역할 |
|---|---|
| `Utilities` | 모델 다운로드, 메타데이터 재검사, 배치/보조 도구 |
| `User` | 개인 설정, 출력 경로, 비밀번호, 개인 토큰 |
| `Server` | 서버 설정, 백엔드, 사용자, 로그, 확장 관리 |
| `Backends` | GPU 서버 또는 외부 생성 엔진 연결 |
| `Extensions` | 공식/추가 확장 설치와 업데이트 |
| `Logs` | 오류, 경고, 디버깅 정보 확인 |
| `Server Configuration` | 호스트, 포트, 모델 경로, 인증, 저장 정책 |

버전에 따라 `Utilities` 일부가 `Tools`로 보이거나, 설명 문구가 조금 다를 수 있다. 그러나 기능의 성격은 같다.

## 3. `Utilities` 탭

`Utilities`는 모델과 파일, 대량 비교, 정리 작업을 위한 도구 모음이다.
일상적인 이미지 생성의 핵심 탭은 아니지만, 운영자는 반드시 익혀야 한다.

### 3-1. `Grid Generator`

`Grid Generator`는 하나의 프롬프트를 여러 조건으로 비교할 때 쓴다.

사용 순서:

1. `Utilities` 또는 `Tools`에서 `Grid Generator`를 연다.
2. 비교할 축을 하나 정한다.
3. 비교할 값들을 넣는다.
4. 필요하면 두 번째, 세 번째 축을 추가한다.
5. `Generate Grid`를 눌러 결과를 만든다.

실무 기준:

- 한 번에 너무 많은 축을 넣지 않는다.
- 비교 목적이 분명할 때만 쓴다.
- 결과는 이미지 비교용이지 최종 납품용이 아니다.
- 축이 늘어날수록 시간은 기하급수적으로 늘어난다.

### 3-2. `Image Batch Tool`

`Image Batch Tool`은 입력 이미지 묶음을 빠르게 다룰 때 쓴다.

용도 예시:

- 여러 이미지에 같은 정리 작업을 적용할 때
- 미리 준비한 입력 묶음을 한 번에 처리할 때
- 작업자 산출물을 빠르게 점검할 때

### 3-3. `Model Downloader`

`Model Downloader`는 서버 운영자가 모델을 URL 기반으로 가져올 때 쓰는 도구다.

주요 입력값:

- 다운로드 URL
- 저장할 모델 이름
- 모델 타입 또는 서브타입
- 필요 시 메타데이터

실행 순서:

1. `Utilities`에서 `Model Downloader`를 연다.
2. 모델 URL을 넣는다.
3. 저장할 파일명을 정한다.
4. 모델 타입을 고른다.
5. 다운로드를 시작한다.
6. 완료 후 모델 목록을 새로고침한다.

운영 원칙:

- 서버에 넣을 모델은 파일명 규칙을 먼저 정한다.
- 동일 모델을 여러 백엔드가 쓰면 경로와 이름을 맞춘다.
- 다운로드 후 메타데이터가 틀리면 바로 고친다.

### 3-4. `Reset All Metadata`

모델을 새로 받았거나 SwarmUI가 모델을 잘못 인식할 때 사용한다.

언제 쓰나:

- 새 모델을 추가한 뒤 타입이 이상할 때
- 업데이트 후 모델 정보가 꼬였을 때
- 썸네일, 아키텍처, 권장 해상도가 엇갈릴 때

사용 순서:

1. `Utilities`에서 `Reset All Metadata`를 실행한다.
2. 모델 목록을 다시 연다.
3. 각 모델 카드의 `Type`과 해상도를 확인한다.
4. 필요하면 `Edit Metadata`로 세부 값을 수정한다.

### 3-5. 그 밖의 도구

버전이나 확장 상태에 따라 다음 도구도 보일 수 있다.

- `CLIP Tokenization`
- `Pickle To Safetensors`
- `LoRA Extractor`
- 기타 변환/검사 도구

이런 도구는 자주 쓰지 않더라도, 운영자가 존재를 알아야 한다.

## 4. 모델 관리

SwarmUI는 모델 파일을 저장해 두는 것만으로 끝나지 않는다.
경로, 타입, 메타데이터, 다운로드 방식이 모두 맞아야 UI에서 올바르게 보인다.

### 4-1. 경로 기본값

서버 설정에서 반드시 맞춰야 하는 핵심은 다음이다.

- `ModelRoot`: 모델 루트 경로
- `SDModelFolder`: 모델 하위 폴더 이름

예시 개념:

- ComfyUI 스타일이면 `checkpoints`, `loras`, `vae` 같은 폴더 구성을 따른다.
- Auto1111 스타일이면 `Stable-diffusion` 계열 이름을 쓸 수 있다.

실무 예시:

```text
ModelRoot = D:/AI/StabilityMatrix/Data/Models
SDModelFolder = checkpoints
실제 모델 파일 = D:/AI/StabilityMatrix/Data/Models/checkpoints/sd_xl_base_1.0.safetensors
```

반대로 아래 같은 상태는 위험하다.

```text
SwarmUI는 checkpoints/sd_xl_base_1.0.safetensors 를 기대
ComfyUI는 checkpoints/SDXL_base_latest.safetensors 만 보유
```

이 경우 백엔드는 붙어도 실제 생성 시 모델 로드 실패가 날 수 있다.

### 4-2. 여러 모델 루트

SwarmUI는 여러 루트를 세미콜론으로 구분해 둘 수 있다.

장점:

- 빠른 SSD와 큰 HDD를 나눠 쓸 수 있다.
- 같은 서버에서 모델 저장소를 분리할 수 있다.
- 기존 ComfyUI 저장소를 함께 참조할 수 있다.

실무 팁:

- 첫 번째 루트는 가장 자주 쓰는 경로로 둔다.
- 여러 경로를 쓸수록 경로 규칙을 더 엄격히 맞춘다.

### 4-3. 모델 메타데이터

모델 카드에서 확인할 값은 다음이다.

- `Type` 또는 `Architecture`
- 권장 해상도
- 설명
- 태그
- 로컬/원격 여부

수정해야 하는 상황:

- 아키텍처가 잘못 감지됐을 때
- SDXL인데 다른 계열로 잡혔을 때
- 비디오 모델이 이미지 모델처럼 보일 때
- 프롬프트 힌트나 권장 해상도가 틀릴 때

### 4-4. 검증 기준

모델을 추가했으면 아래 순서로 검증한다.

1. Generate 탭의 모델 목록에 보이는가?
2. 카드의 Type이 맞는가?
3. 권장 해상도가 대략 맞는가?
4. 실제 1장 생성이 성공하는가?
5. 서버가 다른 백엔드에서도 같은 모델을 찾는가?

## 5. `Server` 탭

`Server`는 운영자 영역이다. 여기서 서버 자체를 관리한다.

### 5-1. `Server Configuration`

이 탭에서 주로 다루는 항목은 다음이다.

- `Host`
- `Port`
- `ModelRoot`
- `SDModelFolder`
- 인증 관련 옵션
- 저장 경로 관련 옵션
- 로그 레벨
- 백엔드 관련 옵션

핵심 절차:

1. `Server -> Server Configuration`으로 들어간다.
2. 네트워크에서 쓸 `Host`를 확인한다.
3. 원격 접속이 필요하면 `0.0.0.0`으로 연다.
4. 모델 경로를 맞춘다.
5. 저장하고 재시작한다.

CLI로도 덮어쓸 수 있다.

```text
--host 0.0.0.0
--port 7801
```

명심할 점:

- CLI 값이 UI 값을 덮어쓸 수 있다.
- `Settings.fds`와 `Backends.fds` 위치를 바꾸면 관련 CLI도 맞춰야 한다.
- 실서비스 운영자는 설정 파일 위치를 함부로 바꾸지 않는 편이 낫다.

### 5-2. `Server Info`

`Server Info`는 서버가 지금 어떻게 보이는지 보여 준다.

확인할 것:

- 현재 접속 주소
- LAN 주소
- 버전
- 업데이트 여부
- 시스템 상태

새 서버를 붙였을 때는 여기부터 확인한다.

### 5-3. `Backends`

`Backends`는 SwarmUI의 핵심이다. 실제 생성 엔진을 연결한다.

대표 유형:

- `ComfyUI Self-Starting`
- `ComfyUI API By URL`
- `Swarm-API-Backend`

언제 무엇을 쓰나:

- 같은 SwarmUI 인스턴스 내부에서 Comfy를 관리하고 싶으면 Self-Starting
- StabilityMatrix가 이미 ComfyUI를 관리하고 있으면 `ComfyUI API By URL`
- 다른 SwarmUI 서버를 붙이면 Swarm-API-Backend

설정 순서:

1. `Server -> Backends`를 연다.
2. `Add Backend`를 누른다.
3. 타입을 고른다.
4. 주소와 옵션을 입력한다.
5. 저장 후 상태가 `Green/Running`인지 본다.

백엔드 승인 체크리스트:

1. 백엔드가 목록에 생성되었는가
2. 상태가 `Green` 또는 `Running`으로 바뀌었는가
3. 해당 백엔드에서 사용할 모델이 실제로 보이는가
4. 테스트 1장 생성이 성공하는가
5. `Logs`에 missing node 또는 missing model 오류가 없는가
6. 여러 백엔드가 있을 때 2건 이상 큐를 넣으면 분산이 보이는가

멀티 GPU/멀티 서버 운영 원칙:

- 큐는 등록된 백엔드 순서에 영향을 받는다.
- 더 빠른 백엔드를 앞에 둔다.
- 같은 모델 경로를 유지해야 한다.
- 실제 분산은 큰 큐를 걸어야 눈에 띈다.

### 5-4. `Users`

`Users`는 계정과 권한을 관리한다.

주요 역할:

- `Owner`
- `Admin`
- `PowerUser`
- `User`

운영 기준:

- `Owner`와 `Admin`은 권한이 동일하다. 공식 문서 기준으로 둘 다 서버 전체 제어 권한을 가진다. Owner는 서버를 처음 세운 사람에게, Admin은 신뢰할 수 있는 운영자에게 부여하는 것이 일반적이다.
- `PowerUser`는 커스텀 워크플로를 다루는 신뢰 사용자다.
- 일반 작업자는 `User`로 충분하다.

기본 작업 순서:

1. `local` 계정 비밀번호를 바꾼다.
2. `AuthorizationRequired`를 켠다.
3. 필요하면 `AllowLocalhostBypass`를 끈다.
4. 사용자를 추가한다.
5. 역할을 최소 권한으로 준다.

### 5-5. `Extensions`

확장은 기능을 늘리는 곳이다.

여기서 할 수 있는 것:

- 확장 설치
- 확장 업데이트
- 확장 제거

주의:

- 확장은 편리하지만 의존성을 꼬이게 할 수 있다.
- 필요한 것만 설치한다.
- 이상이 생기면 최근 확장부터 의심한다.

### 5-6. `Logs`

문제가 생기면 `Logs`가 첫 번째다.

권장 순서:

1. `ViewType`을 `Debug`로 바꾼다.
2. 재현한다.
3. 메시지를 읽는다.
4. 필요하면 `Pastebin`으로 공유한다.

### 5-7. `Add More Backends`

여러 백엔드가 있으면 SwarmUI가 병렬로 일한다.

예:

- 같은 서버의 여러 GPU
- 다른 집 PC
- 렌트 GPU 서버
- 공용 서버 여러 대

확인 기준:

- 백엔드가 초록색인가?
- 모델이 다 같은 경로를 보나?
- 같은 프롬프트를 여러 장 큐에 넣었을 때 분산되나?

## 6. `User` 탭

`User`는 개인 설정의 경계다.

여기서 관리할 것:

- 비밀번호 변경
- 개인 출력 경로
- 프리셋
- API 토큰
- 개인 설정

공유 서버에서는 이 탭이 중요하다.

- 사용자별 출력물이 분리된다.
- 사용자별 경로 규칙이 다를 수 있다.
- 개인 API 토큰과 연결될 수 있다.

## 7. 외부 ComfyUI 백엔드의 필수 커스텀 노드

`ComfyUI API By URL`을 제대로 쓰려면 Swarm 전용 노드가 필요하다.

필수 폴더:

- `SwarmComfyCommon`
- `SwarmComfyExtra`

소스 위치:

```text
src/BuiltinExtensions/ComfyUIBackend/ExtraNodes/SwarmComfyCommon
src/BuiltinExtensions/ComfyUIBackend/ExtraNodes/SwarmComfyExtra
```

넣는 위치:

```text
ComfyUI/custom_nodes/
```

StabilityMatrix 설치 기준 경로 예시는 아래처럼 잡으면 된다.

```text
[StabilityMatrix 데이터 폴더]\Packages\SwarmUI\src\BuiltinExtensions\ComfyUIBackend\ExtraNodes
[StabilityMatrix 데이터 폴더]\Packages\ComfyUI\custom_nodes
```

설치 방법:

1. SwarmUI 저장소에서 위 두 폴더를 찾는다.
2. ComfyUI의 `custom_nodes` 아래로 복사한다.
3. 또는 심볼릭 링크로 연결한다.
4. ComfyUI를 재시작한다.

검증 방법:

- `SwarmSaveImageWS`가 보이는가?
- 필요 시 `SwarmSaveAnimationWS`가 보이는가?
- `SwarmRemBg`, `SwarmYoloDetection` 같은 Swarm 노드가 뜨는가?
- Generate 또는 custom workflow에서 에러 없이 노드가 로드되는가?
- `Server -> Logs` 또는 ComfyUI 로그에 missing node 오류가 없는가?

주의:

- `ComfyUI Manager`는 편하지만 의존성 꼬임을 만들 수 있다.
- 운영 서버에서는 꼭 필요한 노드만 넣는 편이 안전하다.
- `API-By-URL`은 model path 일치가 중요하다.

노드 문제와 모델 경로 문제를 구분하는 간단한 기준:

- 노드 문제:
  - 백엔드는 붙지만 생성 시 특정 노드 누락 오류가 난다.
  - custom workflow 로드 시 빨간 경고나 missing node 메시지가 난다.
- 모델 경로 문제:
  - 백엔드는 초록색인데 특정 모델만 생성이 실패한다.
  - 어떤 백엔드에서는 되고 어떤 백엔드에서는 안 된다.
  - 모델명이 목록엔 보여도 실제 로드가 안 된다.

## 8. API 기본

SwarmUI는 브라우저 UI만이 아니라 HTTP API와 WebSocket API도 제공한다.

API를 쓰는 이유:

- 외부 자동화
- 배치 도구
- 봇이나 스크립트
- 다른 시스템과의 연동
- 관리자 도구 통합

기본 흐름:

1. `POST /API/GetNewSession`으로 세션을 만든다.
2. 이후 요청에 `session_id`를 넣는다.
3. 인증이 켜져 있으면 `swarm_token` 쿠키가 필요하다.
4. 작업에 맞는 route를 호출한다.

간단한 확인 예시:

```bash
curl -H "Content-Type: application/json" -d "{}" -X POST http://localhost:7801/API/GetNewSession
```

이 응답에서 받은 `session_id`를 다음 요청의 본문에 넣는다.

HTTP와 WebSocket의 역할 차이도 구분해야 한다.

- `GenerateText2Image`: 한 번 요청하고 결과를 기다리는 HTTP 방식
- `GenerateText2ImageWS`: 진행률, 미리보기, 상태 업데이트를 받는 WebSocket 방식

가장 단순한 HTTP 생성 본문 예시는 아래처럼 이해하면 된다.

```json
{
  "session_id": "발급받은 세션 ID",
  "images": 1,
  "prompt": "a cat",
  "model": "OfficialStableDiffusion/sd_xl_base_1.0",
  "width": 1024,
  "height": 1024
}
```

인증이 켜진 환경에서는 세션만으로 끝나지 않는다.

- 브라우저 기반 호출이면 로그인 세션 또는 `swarm_token` 쿠키가 함께 가야 한다.
- 외부 스크립트면 세션 발급과 인증 토큰 관리 둘 다 설계해야 한다.

운영팀이 자동화 PoC를 할 때는 아래 두 흐름을 따로 검증한다.

1. 브라우저 로그인 후 개발자 도구에서 API 호출이 정상인지
2. 독립 스크립트에서 `GetNewSession` + 인증 정보로 호출이 정상인지

운영자가 API를 검증할 때는 아래 순서로 보는 것이 좋다.

1. `GetNewSession`이 성공하는가
2. `session_id`를 넣은 HTTP 생성이 성공하는가
3. 인증이 켜져 있다면 `swarm_token` 쿠키가 필요하다는 점을 확인했는가
4. WebSocket 생성에서 진행 상태가 정상적으로 오는가
5. 생성 결과가 실제 저장 경로에 남는가

대표 route 묶음:

- `BackendAPI`: 백엔드 추가, 수정, 목록, 재시작
- `ModelsAPI`: 모델 다운로드, 메타데이터, 목록
- `AdminAPI`: 사용자, 역할, 서버 설정, 로그
- `T2IAPI`: 텍스트-이미지 생성
- `BasicAPIFeatures`: 프리셋, 사용자 데이터, 기본 조작

실무에서 자주 쓰는 API:

- `GetNewSession`
- `GenerateText2Image`
- `GenerateText2ImageWS`
- `ListBackends`
- `ListBackendTypes`
- `RestartBackends`
- `ToggleBackend`
- `DoModelDownloadWS`
- `EditModelMetadata`
- `ListModels`
- `DescribeModel`
- `GetModelHash`

관리자용으로 자주 보는 route:

- `AdminListUsers`
- `AdminAddUser`
- `AdminSetUserPassword`
- `AdminEditRole`
- `ChangeServerSettings`
- `ListRecentLogMessages`
- `LogSubmitToPastebin`

운영자가 자주 자동화하는 요청 형태 예시는 아래처럼 이해하면 된다.

모델 다운로드 요청 개념 예시:

```json
{
  "session_id": "발급받은 세션 ID",
  "url": "https://huggingface.co/.../model.safetensors",
  "type": "Stable-Diffusion",
  "name": "model.safetensors"
}
```

사용자 추가 요청 개념 예시:

```json
{
  "session_id": "발급받은 세션 ID",
  "user_name": "worker01",
  "password": "초기비밀번호",
  "role_id": "user"
}
```

서버 설정 변경 요청 개념 예시:

```json
{
  "session_id": "발급받은 세션 ID",
  "settings": {
    "Network.Host": "0.0.0.0",
    "Network.Port": 7801
  }
}
```

주의:

- 실제 route 이름과 필드 스키마는 버전별로 바뀔 수 있으므로, 배포 전에는 공식 `APIRoutes` 문서와 브라우저 개발자 도구 호출을 함께 대조한다.
- 이 문서의 예시는 “어떤 수준의 데이터를 보낸다”를 설명하기 위한 운영 개념 예시다.

HTTP와 WebSocket 선택 기준:

- 결과만 받으면 되는 단순 배치: HTTP
- 진행률, 미리보기, 대기열 상태를 실시간으로 봐야 하는 작업: WebSocket
- 운영 대시보드나 진행 모니터링: WebSocket 우선
- 단건 자동화 스크립트나 야간 배치: HTTP 우선

언제 쓰나:

- 사람이 수동으로 조작하기엔 반복이 너무 많을 때
- 서버 상태를 자동 점검할 때
- 모델을 대량으로 넣거나 뺄 때
- 외부 도구에서 SwarmUI를 호출할 때

## 9. 운영 책임 분리

이 문서 기준으로 역할을 다시 정리하면 다음과 같다.

- 작업자: `Generate`, `Simple`, 개인 `User` 설정
- 빌더/파워 유저: `Comfy Workflow`
- 운영자: `Utilities`, `Server`, `Backends`, `Extensions`, `Logs`
- 자동화 담당: API

## 10. 운영 점검표

운영자라면 새 서버를 붙일 때 아래를 확인한다.

1. 모델 경로가 올바른가?
2. 백엔드가 초록색인가?
3. 외부 ComfyUI라면 Swarm 노드가 설치되어 있는가?
4. 사용자 계정과 권한이 맞는가?
5. `Logs`에 에러가 없는가?
6. API가 `GetNewSession`부터 정상 작동하는가?
