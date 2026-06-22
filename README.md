[README.md](https://github.com/user-attachments/files/29188779/README.md)
# 반려동물 카메라 — 개발 키트

두 가지가 들어 있습니다.

1. `test-app/` — 브라우저에서 바로 돌아가는 **검증용 프로토타입**
2. `ios-skeleton/` — 실제 iOS 앱의 **핵심 코드 뼈대** (Swift)

---

## 1. 테스트 앱 실행하기 (`test-app/index.html`)

카메라(`getUserMedia`)는 보안 컨텍스트에서만 동작합니다. `file://`로 그냥 열면
카메라가 안 켜질 수 있어요. 아래 둘 중 하나로 여세요.

### A. 데스크탑 (웹캠으로 빠르게 확인)
```bash
cd test-app
python3 -m http.server 8000
```
브라우저에서 `http://localhost:8000` 접속 → `localhost`는 카메라 허용됩니다.

### B. 폰에서 실제 반려동물로 테스트 (권장)
폰 카메라를 쓰려면 HTTPS가 필요합니다. 가장 쉬운 방법:
- **GitHub Pages**: 레포에 `index.html` 올리고 Pages 켜면 끝 (무료, HTTPS 자동)
- 또는 `npx serve` + `ngrok http 3000` 으로 임시 HTTPS 주소 생성

### 사용법
- 화면 하단 소리 버튼으로 반려동물 시선을 끕니다 (지금은 합성음, 실제 앱은 녹음 파일 사용).
- 강아지/고양이가 **충분히 크고 + 중앙에 + 안정적으로** 잡히면 자동 연사 → 가장 선명한 컷 자동 선택.
- 패널의 슬라이더 4개로 임계값을 **실제 반려동물 앞에서** 조정하세요. 이게 이 앱의 핵심 검증입니다.
- 각 컷의 숫자는 선명도 점수입니다. 흔들린 사진은 점수가 확 낮아집니다.

### 무엇을 검증하나
- 우리 견종/묘종, 우리 집 조명에서 인식이 잘 되는가?
- 자동 셔터가 "쳐다본 순간"을 실제로 잡는가, 헛발질하는가?
- 어떤 임계값 조합이 가장 잘 맞는가? (→ 이 값을 iOS `AutoShutterController`에 그대로 이식)

> 참고: 테스트 앱은 범용 모델(coco-ssd)을 씁니다. 실제 iOS 앱은 Apple Vision의
> 전용 동물 탐지를 써서 더 빠르고 정확합니다. 테스트 앱은 "개념과 임계값" 검증용입니다.

---

## 2. iOS 코드 뼈대 (`ios-skeleton/`)

| 파일 | 역할 |
|---|---|
| `CameraManager.swift` | AVFoundation 캡처 세션, 매 프레임 전달 |
| `AnimalDetector.swift` | Vision 내장 개/고양이 탐지 (온디바이스) |
| `AutoShutterController.swift` | **핵심** — 발사 판단 + 연사 + 선명도 선별 |
| `ContentView.swift` | SwiftUI로 전체 파이프라인 연결 + 프리뷰 |

### Xcode 세팅
1. Xcode에서 새 iOS App (SwiftUI) 프로젝트 생성.
2. 위 4개 파일을 프로젝트에 추가.
3. `Info.plist`에 카메라 권한 설명 추가 (없으면 앱이 즉시 종료됩니다):
   - Key: `NSCameraUsageDescription`
   - Value: `반려동물 사진을 찍기 위해 카메라를 사용합니다.`
4. 저장 기능을 쓰려면 `NSPhotoLibraryAddUsageDescription`도 추가.
5. **실기기에서 실행** (시뮬레이터엔 카메라가 없습니다).

### 튜닝 포인트
`AutoShutterController`의 상단 변수들(`minBoxArea`, `maxCenterOffset`,
`requiredGoodFrames`, `minConfidence`)을 테스트 앱에서 찾은 값으로 맞추세요.

### 다음에 채울 것 (v1 → 정식)
- 시선 유도음 녹음 파일 + 재생 UI (와이어프레임 하단 행)
- `AVCapturePhotoOutput`으로 풀해상도 스틸 (지금은 비디오 프레임 사용)
- 반려동물 보정 필터 (검은 털 디테일·눈 선명도)
- 연사 결과 필름스트립 리뷰 UI
