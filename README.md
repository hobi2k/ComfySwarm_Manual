# SwarmUI Operator Manual

StabilityMatrix, ComfyUI, SwarmUI, Tailscale을 함께 쓰는 회사 작업 환경을 위한 매뉴얼 묶음이다.  
핵심 목적은 작업자 PC와 GPU 서버를 분리하는 것이다.

- 작업자는 자기 컴퓨터에서 `SwarmUI`로 접속한다.
- 실제 생성은 서버의 `ComfyUI` 백엔드가 맡는다.
- 서버는 `StabilityMatrix`로 관리한다.
- 외부 접속은 `Tailscale`로 처리한다.

이 구조를 쓰면 작업자 컴퓨터 사양이 낮아도 무거운 생성 작업을 돌릴 수 있고, 백엔드를 여러 대 붙여 GPU 작업을 병렬로 확장하기도 쉽다.

## 문서 구성

- [01. 개요](docs/01-overview.md)
- [02. 서버 설치 및 연결](docs/02-server-installation.md)
- [03. 작업자 탭 가이드](docs/03-worker-tabs.md)
- [04. 운영 도구와 API](docs/04-admin-operations.md)
- [05. 원격 접속과 보안](docs/05-remote-access-security.md)
- [06. 문제 해결과 점검표](docs/06-troubleshooting.md)

## 읽는 순서

1. 먼저 [개요](docs/01-overview.md)에서 역할 분리와 탭 구성을 잡는다.
2. 다음으로 [서버 설치 및 연결](docs/02-server-installation.md)에서 GPU 서버와 접속 경로를 만든다.
3. 실제 사용 방법은 [작업자 탭 가이드](docs/03-worker-tabs.md)에서 익힌다.
4. 운영 메뉴와 자동화 연동은 [운영 도구와 API](docs/04-admin-operations.md)에서 본다.
5. 외부 공유와 계정 운영은 [원격 접속과 보안](docs/05-remote-access-security.md)에서 정리한다.
6. 장애 대응은 [문제 해결과 점검표](docs/06-troubleshooting.md)에서 바로 확인한다.

## 출처

- 로컬 정리본: `hobi2k.github.io/_posts/2026-04-06-StabilityMatrix-SwarmUI-Tailscale.markdown`
- 공식 SwarmUI 문서: `https://github.com/mcmonkeyprojects/SwarmUI/blob/master/docs/README.md`
- ComfyUI 백엔드 문서: `https://github.com/mcmonkeyprojects/SwarmUI/blob/master/src/BuiltinExtensions/ComfyUIBackend/README.md`
