# 06. 문제 해결과 점검표

이 문서는 SwarmUI 운영 중 자주 만나는 문제를 증상별로 정리한 복구 가이드다.

가장 먼저 기억할 원칙은 단순하다.

1. 재시작한다.
2. 로그를 본다.
3. 주소와 경로를 확인한다.
4. 모델 메타데이터를 점검한다.
5. 그래도 안 되면 설치 환경을 다시 맞춘다.

관련 문서:

- 운영 메뉴와 API는 [04. 운영 도구와 API](./04-admin-operations.md)에서 본다.
- 원격 접속과 보안은 [05. 원격 접속과 보안](./05-remote-access-security.md)에서 본다.

## 1. 가장 먼저 할 일

문제가 생기면 아래 순서로 본다.

1. 서버 PC와 작업자 PC가 둘 다 켜져 있는지 확인한다.
2. Tailscale이 살아 있는지 확인한다.
3. SwarmUI를 재시작한다.
4. ComfyUI 백엔드도 재시작한다.
5. 브라우저를 시크릿 창으로 다시 연다.
6. `Server -> Logs`에서 에러를 본다.

재시작만으로 해결되는 문제는 생각보다 많다.

## 2. 접속 문제

### SwarmUI가 안 열린다

확인 순서:

1. SwarmUI 프로세스가 살아 있는가?
2. `Host`가 `0.0.0.0` 또는 필요한 네트워크 주소인가?
3. 포트가 `7801`인가?
4. 주소를 `127.0.0.1`이 아니라 Tailscale IP로 열었는가?
5. Windows 방화벽이 막고 있지 않은가?

### ComfyUI가 안 열린다

확인 순서:

1. ComfyUI가 실제로 떠 있는가?
2. `--listen 0.0.0.0`이 필요한 경우 적용했는가?
3. 포트가 `8188`인가?
4. SwarmUI 백엔드 주소가 정확한가?
5. 서버가 절전 모드로 들어가지 않았는가?

### Tailscale로는 붙는데 화면이 안 보인다

확인 순서:

- 같은 tailnet인가?
- MagicDNS 이름이 맞는가?
- URL에 포트가 빠지지 않았는가?
- 서버가 실제로 응답하고 있는가?
- 방화벽이 TCP 7801을 막고 있지 않은가?

## 3. 백엔드 문제

백엔드가 `Red`, `Disconnected`, `Disabled` 상태면 아래를 본다.

1. `Server -> Backends`에서 주소가 맞는지 확인한다.
2. `ComfyUI API By URL`인지 확인한다.
3. 같은 머신이면 `127.0.0.1:8188`가 맞는지 확인한다.
4. 다른 머신이면 Tailscale 또는 LAN 주소인지 확인한다.
5. 모델 경로가 서로 같아야 하는지 확인한다.
6. Swarm 전용 노드가 설치되어 있는지 본다.
7. 다시 시작한다.

멀티 GPU/멀티 서버에서는 다음도 확인한다.

- 백엔드 순서가 의도한 대로인가?
- 특정 모델이 어떤 백엔드에서만 가능한가?
- 큐가 충분히 많아야 분산이 보인다.

## 4. 모델 문제

### 모델이 안 보인다

확인 순서:

1. `ModelRoot`가 맞는가?
2. `SDModelFolder`가 맞는가?
3. 파일명이 정확한가?
4. 모델 루트가 여러 개인 경우 순서가 맞는가?
5. 새로 받은 모델을 아직 새로고침하지 않았는가?

### 모델 타입이 이상하다

조치 순서:

1. `Utilities -> Reset All Metadata`를 실행한다.
2. 모델 카드의 `Edit Metadata`를 연다.
3. `Architecture`가 맞는지 확인한다.
4. 권장 해상도와 설명도 맞춘다.

### 새로 받은 모델이 이상하게 인식된다

SwarmUI 공식 문서 기준으로는 업데이트 후 다시 스캔하는 것이 중요하다.

확인할 것:

- SwarmUI가 최신인가?
- 모델을 업데이트 전에 받은 건 아닌가?
- `Reset All Metadata`를 했는가?
- `Edit Metadata`에서 수동 수정이 필요한가?

## 5. Swarm 전용 노드 문제

외부 ComfyUI를 쓸 때 Swarm 노드가 없으면 생성이 깨질 수 있다.

필수 노드:

- `SwarmComfyCommon`
- `SwarmComfyExtra`

확인 순서:

1. `ComfyUI/custom_nodes`에 있는가?
2. SwarmUI 저장소의 `src/BuiltinExtensions/ComfyUIBackend/ExtraNodes`에서 복사했는가?
3. ComfyUI를 재시작했는가?
4. 노드 목록에 `SwarmSaveImageWS`가 보이는가?
5. 필요한 경우 `SwarmSaveAnimationWS`나 `SwarmYoloDetection`도 보이는가?

문제가 계속되면:

- 최근에 넣은 커스텀 노드를 빼 본다.
- `ComfyUI Manager`가 설치를 꼬이게 했는지 본다.
- 꼭 필요한 노드만 남긴다.

## 6. 사용자와 인증 문제

로그인이 안 되거나 권한이 이상하면 아래를 본다.

1. `AuthorizationRequired`가 켜져 있는가?
2. `AllowLocalhostBypass`가 의도와 맞는가?
3. `local` 계정 비밀번호를 바꿨는가?
4. 역할이 너무 제한적이지 않은가?
5. 계정이 삭제되거나 비활성화되지 않았는가?

자기 자신을 잠갔다면:

- `Data/Settings.fds`를 직접 수정해 `AuthorizationRequired`를 잠시 끈다.
- 다시 접속한 뒤 권한을 고친다.

## 7. API 문제

API가 안 되면 세션부터 확인한다.

기본 체크:

- `GetNewSession`을 먼저 호출했는가?
- 이후 요청에 `session_id`를 넣었는가?
- 인증이 켜져 있다면 `swarm_token` 쿠키가 있는가?
- route가 맞는가?

자주 나는 실수:

- 세션 만료
- 잘못된 route 호출
- 권한 부족
- 백엔드 관리 API와 모델 API 혼동

## 8. 설치 환경 문제

### `Torch not compiled with CUDA enabled`

이 메시지는 보통 Comfy 백엔드의 Python 환경이 꼬였다는 뜻이다.

해결 원칙:

- 사용 중인 실제 ComfyUI 환경에서 torch를 다시 설치한다.
- `-s`를 붙여 현재 환경에 설치한다.
- StabilityMatrix가 관리하는 ComfyUI라면 그 패키지의 Python 환경을 쓴다.
- SwarmUI self-start 환경이라면 `dlbackend/comfy` 쪽 환경을 쓴다.

### `NuGet` 오류

네트워크, 방화벽, 일시적 장애, 캐시 손상을 의심한다.

### `dubious ownership` 오류

설치 위치가 외장 드라이브이거나 Git이 소유권을 이상하게 판단하는 경우다.
가능하면 시스템 드라이브로 옮긴다.

### 서버가 절전 모드로 꺼지는 문제

원격 접속 환경에서는 서버가 항상 켜져 있어야 한다.
GPU 서버의 절전 모드를 반드시 비활성화한다.

- Windows: `설정 → 전원 및 절전 → 절전 모드` → `없음` 설정
- 화면 끄기는 허용해도 되지만, 절전(Sleep)/최대 절전(Hibernate)은 끄는 것이 원칙이다.
- 장시간 자리를 비울 때도 서버가 꺼지지 않아야 한다.

### 잘못된 Python에 pip를 넣은 경우

가장 안전한 방법은 다음이다.

1. 현재 환경의 Python을 확인한다.
2. 그 Python으로 `python -s -m pip install ...`을 실행한다.
3. 전역 Python에 설치하지 않는다.

## 9. 로그 보는 법

문제가 반복되면 로그가 답이다.

권장 절차:

1. `Server -> Logs`를 연다.
2. `ViewType`을 `Debug`로 바꾼다.
3. 다시 문제를 재현한다.
4. 에러 메시지와 스택을 읽는다.
5. 필요하면 `Pastebin`으로 공유한다.

지원 요청 시 같이 주면 좋은 정보:

- 어떤 탭에서 막혔는지
- 어떤 버튼을 눌렀는지
- 어떤 모델을 썼는지
- 에러 메시지 원문
- 스크린샷

## 10. 복구 순서

문제가 오래가면 아래 순서로 복구한다.

1. SwarmUI와 ComfyUI를 둘 다 종료한다.
2. 최근에 추가한 확장이나 커스텀 노드를 뺀다.
3. 백엔드 주소와 포트를 다시 본다.
4. 모델 경로를 다시 맞춘다.
5. 메타데이터를 재검사한다.
6. 그래도 안 되면 설치 환경을 다시 맞춘다.

## 11. 빠른 점검표

- `Generate`가 안 되면 모델과 백엔드를 본다.
- `Server`가 안 되면 호스트, 포트, 방화벽을 본다.
- `User`가 안 되면 인증과 역할을 본다.
- `API`가 안 되면 세션과 쿠키를 본다.
- 외부 ComfyUI가 깨지면 Swarm 노드와 model path를 본다.
