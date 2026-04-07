# 05. 원격 접속과 보안

이 문서는 SwarmUI를 여러 작업자가 함께 쓰는 환경에서 안전하게 여는 방법을 정리한다.

핵심 원칙은 하나다.

- 작업자 PC는 가볍게 유지한다.
- GPU 서버만 외부 접근을 받는다.
- 공개 인터넷 노출은 기본적으로 피한다.
- 사설 VPN, 특히 `Tailscale`을 우선한다.

관련 문서:

- 운영 메뉴와 API는 [04. 운영 도구와 API](./04-admin-operations.md)에서 본다.
- 장애 대응은 [06. 문제 해결과 점검표](./06-troubleshooting.md)에서 본다.

## 1. 권장 구조

권장 구조는 다음과 같다.

- 서버 PC에 `SwarmUI`와 `ComfyUI`를 둔다.
- 작업자 PC들은 브라우저로 `SwarmUI`만 쓴다.
- 실제 생성은 서버 백엔드가 맡는다.
- 원격 접속은 `Tailscale`로 한다.

이 구조의 장점:

- 작업자 PC 사양이 낮아도 된다.
- 공용 GPU 서버로 병렬 작업이 가능하다.
- 포트 포워딩 없이 접속할 수 있다.
- 사용자 권한을 계정 단위로 나눌 수 있다.

## 2. 먼저 열어야 하는 포트와 호스트

원격에서 SwarmUI를 쓰려면 서버 쪽 바인딩이 중요하다.

SwarmUI:

```text
--host 0.0.0.0
--port 7801
```

ComfyUI 직접 접속 또는 `ComfyUI API By URL` 백엔드용:

```text
--listen 0.0.0.0
```

기본 해석:

- `SwarmUI`는 작업자들이 접속하는 메인 웹 UI다.
- `ComfyUI`는 실제 생성 엔진이다.
- 일반 작업자는 SwarmUI만 보면 충분하다.
- ComfyUI 직접 노출은 관리자 확인이나 디버깅에만 필요하다.

## 3. Tailscale 설치

Tailscale은 SwarmUI 공유에 가장 단순하고 안전한 선택이다.

설치 절차:

1. 서버 PC에 Tailscale을 설치한다.
2. 작업자 PC에도 Tailscale을 설치한다.
3. 같은 계정으로 로그인한다.
4. 관리 콘솔에서 MagicDNS를 켜면 주소 관리가 쉬워진다.

기본 주소 형태:

```text
http://100.x.x.x:7801
http://my-gpu-server:7801
```

## 4. Windows 방화벽

Tailscale이 있어도 Windows 방화벽이 막으면 접속이 안 된다.

권장 규칙:

- `Private` 프로필에만 연다.
- `100.64.0.0/10` 대역만 허용한다.
- 서버 쪽에서만 연다.

예시:

```powershell
New-NetFirewallRule -DisplayName "SwarmUI-Tailscale" -Direction Inbound -Protocol TCP -LocalPort 7801 -Profile Private -RemoteAddress 100.64.0.0/10 -Action Allow
New-NetFirewallRule -DisplayName "ComfyUI-Tailscale" -Direction Inbound -Protocol TCP -LocalPort 8188 -Profile Private -RemoteAddress 100.64.0.0/10 -Action Allow
```

주의:

- 전체 LAN에 넓게 열어 두지 않는다.
- 작업자 PC에서는 방화벽을 따로 열 필요가 없는 경우가 많다.
- 원격 공유가 늘어날수록 제한 범위를 좁히는 편이 낫다.

## 5. 사용자와 권한

SwarmUI는 계정별 권한과 출력 경로를 분리할 수 있다.

권장 역할:

- `Owner` / `Admin`: 공식 문서 기준으로 두 역할의 권한은 동일하다(전체 서버 제어). Owner는 서버를 세운 사람, Admin은 신뢰할 수 있는 운영자에게 부여하는 조직적 구분이다.
- `PowerUser`: 고급 워크플로 사용 가능 사용자
- `User`: 일반 작업자

운영 순서:

1. `local` 계정의 비밀번호를 먼저 바꾼다.
2. `AuthorizationRequired`를 켠다.
3. 필요하면 `AllowLocalhostBypass`를 끈다.
4. 작업자별 계정을 만든다.
5. 역할은 최소 권한으로 준다.

공유 서버에서는 다음도 중요하다.

- 사용자별 출력 경로가 다를 수 있다.
- 사용자별 프리셋과 설정이 분리될 수 있다.
- `PowerUser`는 커스텀 워크플로를 다뤄도 되는 사람에게만 준다.

## 6. 공개 노출을 피하는 이유

SwarmUI 공식 문서도 공개 인터넷 노출은 신중하라고 말한다.

이유:

- 커스텀 노드와 워크플로의 검증 수준이 일정하지 않다.
- API와 브리지 경로가 많아 표면적이 넓다.
- 사용자 권한이 많아질수록 오남용 가능성이 커진다.
- 공개 서비스는 별도의 인증 프록시와 보안 검토가 필요하다.

이 매뉴얼의 권장 방식은 `Tailscale + 제한된 사용자 + 제한된 포트`다.

## 7. 다른 머신을 백엔드로 쓸 때

여러 GPU 서버를 함께 쓸 수 있다.

두 가지 패턴이 있다.

- 다른 SwarmUI 서버를 붙인다.
- ComfyUI만 있는 머신을 `ComfyUI API By URL`로 붙인다.

다른 머신이 SwarmUI까지 함께 돌리고 있다면 `Swarm-API-Backend`를 쓰는 편이 낫다.
반대로 그 머신이 ComfyUI만 돌리고 있다면 `ComfyUI API By URL`을 쓴다.

외부 ComfyUI를 쓸 때 반드시 확인할 것:

- 모델 경로가 서로 같아야 한다.
- 모델 파일명과 폴더 구조가 같아야 한다.
- Swarm 전용 커스텀 노드가 설치되어 있어야 한다.

## 8. 외부 ComfyUI 백엔드 필수 노드

외부 ComfyUI를 SwarmUI 백엔드로 붙이면 다음 두 폴더가 필요하다.

- `SwarmComfyCommon`
- `SwarmComfyExtra`

출처 위치:

```text
src/BuiltinExtensions/ComfyUIBackend/ExtraNodes/SwarmComfyCommon
src/BuiltinExtensions/ComfyUIBackend/ExtraNodes/SwarmComfyExtra
```

설치 위치:

```text
ComfyUI/custom_nodes/
```

설치 방법:

1. SwarmUI 저장소에서 위 두 폴더를 찾는다.
2. ComfyUI의 `custom_nodes` 아래에 복사한다.
3. 또는 심볼릭 링크로 연결한다.
4. ComfyUI를 재시작한다.

검증:

- `SwarmSaveImageWS`가 보이는가?
- `SwarmSaveAnimationWS`가 필요한 경우 보이는가?
- `SwarmRemBg`, `SwarmYoloDetection` 같은 노드가 로드되는가?
- custom workflow를 열었을 때 노드 누락이 없는가?

주의:

- `ComfyUI Manager`는 편하지만 의존성 꼬임을 만들 수 있다.
- 운영 서버에는 꼭 필요한 노드만 넣는 편이 안전하다.

## 9. 실전 접속 순서

작업자가 따라야 할 순서:

1. 서버 PC에서 SwarmUI와 Tailscale이 살아 있는지 본다.
2. 작업자 PC에서 Tailscale에 로그인한다.
3. 서버의 MagicDNS 또는 `100.x.x.x` 주소를 확인한다.
4. 브라우저에서 `http://100.x.x.x:7801`로 접속한다.
5. 필요하면 관리자만 `8188`도 확인한다.

## 9-1. 배포 후 원격 접속 검증

배포 직후에는 아래 순서대로 검수한다.

1. `Server Info`에서 서버가 인식한 LAN 주소를 확인한다.
2. 작업자 PC에서 Tailscale이 연결되어 있는지 확인한다.
3. 작업자 계정으로 로그인 화면이 뜨는지 확인한다.
4. 일반 작업자 계정이 의도한 탭만 보는지 확인한다.
5. 일반 작업자 계정으로 테스트 작업 1건이 성공하는지 확인한다.
6. 관리자만 필요 시 `8188` 직접 점검을 한다.

## 10. 보안 점검표

배포 전에 아래를 확인한다.

- `Host`가 정말 필요한 범위만 열려 있는가?
- 방화벽이 `Tailscale` 범위만 허용하는가?
- `AuthorizationRequired`가 켜져 있는가?
- `local` 비밀번호가 기본값이 아닌가?
- 공유 대상이 신뢰 가능한 사람인가?
- 외부 ComfyUI라면 model path와 custom nodes가 맞는가?

## 10-1. 보안 수용 테스트

실제 배포 전에는 아래를 운영자가 직접 검증한다.

1. 관리자 계정으로 원격 로그인할 수 있는가
2. 일반 작업자 계정으로 원격 로그인할 수 있는가
3. 일반 작업자 계정이 `Server` 같은 관리자 메뉴에 접근하지 못하는가
4. 작업자 계정으로 테스트 생성 1건이 되는가
5. 결과물이 해당 사용자 컨텍스트에서 저장되는가
6. Tailscale이 아닌 예상 밖 경로에서 접근이 차단되는가
7. `AllowLocalhostBypass` 설정이 의도대로 동작하는가
