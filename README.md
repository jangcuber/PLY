# PLY

PLY는 Pleos 차량에서 동승자와 함께 쓰는 앱입니다. 동승자는 차량 화면의 QR을 스캔해 크루에 합류한 뒤 휴대폰에서 여행 지도를 보고, 이동 중에는 미니게임을 즐길 수 있습니다. 주차하면 크루와 함께 협동 게임을 할 수 있습니다.

- 팀명 · Z-RIDE
- 제출 APK · [`apk/Z-RIDE_PLY_release.apk`](apk/Z-RIDE_PLY_release.apk)
- 차량 앱 · PLY 0.6.0, 패키지 `ai.zride.demo.car`

## 설치 및 시연

1. Pleos Connect 에뮬레이터를 켭니다. Android 홈 화면이 뜨면 아래 명령으로 APK를 설치합니다. 기기가 여러 대 연결돼 있다면 `adb -s <serial>`로 설치할 기기를 지정합니다.
2. 차량 런처에서 PLY를 엽니다. 위치 권한을 요청하면 **앱 사용 중에만 허용**을 선택합니다. 기어를 P에 둔 상태에서 **크루 로비 열기**를 누른 뒤 **동승자 초대**를 선택하면 QR이 뜹니다.
3. 동승자는 휴대폰으로 QR을 스캔하고 게스트 닉네임을 입력합니다. 빈 좌석을 골라 합류하면 차량 로비에 동승자가 표시되고 휴대폰에 여행 지도가 열립니다.
4. Pleos 기본 내비에서 목적지를 설정하고 길 안내를 시작하면 경로를 볼 수 있습니다. 주차 중에는 차량 상태와 서버에서 확인한 참가 조건에 따라 게임을 열 수 있습니다.

```sh
adb install -r apk/Z-RIDE_PLY_release.apk
```

같은 패키지의 개발용 앱이 이미 설치돼 있으면 서명 차이로 업데이트에 실패할 수 있습니다. 기존 앱 데이터를 보존하려면 별도의 Pleos AVD에 설치합니다.

## 테스트 환경 및 설정

| 항목 | 환경 |
| --- | --- |
| Pleos 시스템 이미지 | Pleos Connect 2.1.0, ARM64, Android 14 (API 34) |
| Android Emulator | 37.3.6.0 (macOS, Apple M2) |
| 차량 메인 화면 | 2560 × 1440 px, 가로, 160 dpi |
| 연결 | 차량 AVD와 동승자 브라우저에 인터넷 연결 필요 |
| 기본 시연 서버 | https://hyundai.hjpark.net |

차량 앱을 실행하려면 Pleos 런타임의 차량 공유 라이브러리가 필요합니다. 동승자 화면은 휴대폰 브라우저에서 바로 열립니다.

## 소스 빌드

JDK 17, Android SDK Platform 34, Build Tools 34.0.0을 설치합니다. SDK 경로는 `ANDROID_HOME` 또는 `android/local.properties`의 `sdk.dir`에 지정합니다. Gradle Wrapper는 8.9를 사용하며, 빌드 중 Pleos SDK를 공식 Maven 저장소에서 가져옵니다.

```sh
cd android
./gradlew :carapp:assembleRelease
```

빌드가 끝나면 `android/carapp/build/outputs/apk/release/carapp-release-unsigned.apk`가 생성됩니다. 직접 빌드한 APK는 Android SDK의 `apksigner`로 보유한 키를 써서 서명한 뒤 설치합니다. 저장소에 올린 제출 APK는 서명된 파일입니다.

## 로컬 실행

서버를 로컬에서 실행하려면 Node.js 22.13 이상이 필요합니다.

```sh
cd server
npm ci --ignore-scripts --no-audit --no-fund
npm test
npm start
```

PC에서는 `http://127.0.0.1:8000`으로, Android AVD에서는 `http://10.0.2.2:8000`으로 접속합니다. Power Castle, Volt Rush, Power Tug, 충전 릴레이는 로컬 서버의 `/charge/<게임>/`에서 열립니다. 네 게임 모두 Cloudflare 없이 한 컴퓨터에서 시연할 수 있습니다.

동승자 페이지는 `/play`에서 열고, WebSocket은 `/ws`로 연결합니다. 서버 상태는 `/healthz`에서 확인할 수 있습니다. Docker로 실행하려면 `server/`에서 `docker compose up --build`를 실행합니다. [서버 설정](server/README.md) · [Android 빌드](android/README.md)

## 통신과 지도

- 차량 앱은 `vehicle.report`와 `navigation.report`로 차량 상태와 내비 정보를 서버에 보냅니다. 서버는 좌석 승인 여부와 게임 참가 조건을 확인한 뒤 승인된 클라이언트에 최신 방 상태와 재고를 전달합니다.
- 동승자 지도는 등록된 경로가 있으면 그 경로를 사용합니다. Pleos 내비로 안내할 때는 서버가 OSRM을 사용해 차량의 현재 위치에서 유효한 경유지를 순서대로 거쳐 목적지까지 가는 길을 계산합니다. 좌표를 받지 못한 경유지는 여행 지도에서 따로 안내합니다. [Pleos 내비 연동](server/docs/native-navigation.md)
- 차량 도로망과의 연결을 기준으로 OSM 객체 17,923개를 골랐습니다. 구체적인 선별 기준과 현장 확인 범위는 차량 접근 기준 문서에 정리했습니다. [장소 목록](data/places/README.md) · [차량 접근 기준](data/places/DRIVING_FILTER.md)

## 코드와 개발 문서

| 위치 | 내용 |
| --- | --- |
| [`server/`](server/README.md) | HTTP·WebSocket 서버, 동승자 웹 화면, 방과 차량 상태 관리, 아이템 판정 |
| [`android/`](android/README.md) | 제출용 차량 앱과 개발용 Android 모듈 |
| [`minigames/`](minigames/README.md), `games/` | 주행 미니게임, 충전 게임, DEADZONE |
| [`data/places/`](data/places/README.md), `tools/` | 장소 데이터와 시연·검증 도구 |

- [REST OpenAPI](server/openapi.json) · [WebSocket 프로토콜](server/protocol.json) · [앱 팀 인계서](server/docs/client-handoff.md)
- [장소와 파티](server/docs/places-parties.md) · [폰 로그인과 QR 초대](server/docs/phone-accounts.md) · [차량 효과](server/docs/vehicle-effects.md)
- [읽기 전용 관제](server/docs/observer.md) · [SQLite 저장](server/docs/persistence.md) · [Proxmox 배포](server/docs/deployment.md)
