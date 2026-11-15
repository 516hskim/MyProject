# 주차나침반 Park Compass

아파트·주상복합 주차장의 빈자리 데이터와 차량 내 위치·길 안내를 연결하는 차량용 주차 동행 앱 제안 및 시제품입니다.

## 빠른 실행

`prototype/index.html`을 Chrome 또는 Edge에서 여세요. 이 시제품은 외부 설치나 서버 없이 동작하는 화면 흐름 데모입니다.

1. 방문객 또는 입주민을 선택합니다.
2. `해오름 주상복합`의 주차장으로 이동합니다.
3. 층과 추천 주차면을 선택하고 `주차 완료`를 누릅니다.
4. 귀환 화면에서 저장된 위치와 가까운 엘리베이터 동선을 확인합니다.

표시되는 시설과 점유 정보는 가상 시연 데이터입니다. 실제 아파트·주상복합 주차 데이터, 차량 INS, Pleos Connect SDK 또는 운영사 서버와 연결되어 있지 않습니다.

## 제안 요약

- 입주민은 귀가 시간대에 빈자리를 찾아 여러 층을 도는 시간을 줄입니다.
- 방문객은 초대받은 단지와 목적 동을 기준으로 방문 허용 구역의 자리를 찾습니다.
- 앱은 층·면 단위 점유 상태와 갱신 시각을 확인하고, 주차장 도로 그래프에서 목적지까지의 주행 경로를 계산합니다.
- 주차 후에는 층·주차면·가까운 엘리베이터를 저장해 차량과 출입구를 다시 찾도록 돕습니다.
- 조향·자동 주차 제어는 하지 않습니다. 안내 위치의 신뢰도가 낮으면 특정 면 대신 구역 단위로 안내합니다.

## 기술 전제

시설 측에서 아래 데이터를 앱이 쓸 수 있는 표준 API로 제공한다고 가정합니다.

- 주차장, 층, 입구와 출구, 주차면 식별자 및 실내 좌표
- 실시간 점유 상태, 데이터 갱신 시각, 예약/통제 여부
- 입주민·방문객·전기차 등 사용자와 차량별 이용 제한
- 차량 통행로, 램프, 보행로, 엘리베이터, 동 출입구의 연결 그래프
- 방문 허용 구역을 판정할 수 있는 권한 또는 초대 정보

시설 API의 빈자리 데이터는 “어느 면이 사용 가능한가”를 알려주고, INS·차량 위치는 “현재 차량이 어느 통로의 어느 위치인가”를 알려줍니다. 둘을 지도상의 주행 가능 그래프에 맞춰야 특정 빈자리까지 유도할 수 있습니다. 차량용 앱에 원시 INS/휠 오도메트리가 노출되지 않는다면 이 기능을 기본 제공된다고 가정하면 안 됩니다.

## SDK 확인 사항

- Pleos NaviHelper는 목적지·경유지 설정과 경로 안내 연동에 사용합니다.
- 차량 상태 API는 지원 여부와 권한을 프로젝트 및 차량별로 확인한 뒤 선택 기능으로 연결합니다.
- 공개 FusedLocation 자료에 표시된 데이터는 위도, 경도, 고도, 진행 방향, 신뢰도, 시각입니다. 원시 INS 또는 층·주차면 위치가 보장된다는 뜻은 아닙니다.
- Pleos 개발자 포털의 프로젝트 SDK 접근 안내는 현재 FusedLocation을 `Not Available Yet`으로, Vehicle State와 NaviHelper를 `Ready to Use`로 표시합니다. Playground의 실제 프로젝트 상태를 먼저 확인합니다.
- 접근 권한이 없으면 시제품처럼 위치 트레이스를 재생해 시연하고, 실제 기능에서는 수동 층/구역 선택과 구역 단위 안내를 제공합니다.

## 저장소 구성

```text
prototype/index.html          클릭 가능한 화면 시제품
docs/technical-feasibility.md INS·위치·경로 알고리즘 및 제한
docs/parking-data-contract.md 주차장 API 데이터 계약 초안
docs/application-content.md  신청서에 입력한 본문
```

## 기술 문서

- [기술 구현과 가능성](docs/technical-feasibility.md)
- [주차장 데이터 API 계약](docs/parking-data-contract.md)
- [신청서 본문 초안](docs/application-content.md)

## 공식 참고 자료

- [공모전 참가 가이드](https://appdevchallenge.auto-hmg.io/guide/overview)
- [공모전 심사 기준](https://appdevchallenge.auto-hmg.io/guide/judging-awards)
- [Pleos 프로젝트 SDK 접근 상태](https://document.pleos.ai/en/docs/pleos-only/create-my-project)
- [Pleos FusedLocation SDK](https://document.pleos.ai/docs/pleos-only/connect-sdk-pleos/FusedLocation/intro)
- [Pleos NaviHelper SDK](https://document.pleos.ai/api-reference/connect-sdk-pleos/NaviHelper/intro)

## GitHub에 올리기

이 묶음은 GitHub에 올릴 수 있는 로컬 저장소 초안입니다. 원격 저장소는 생성하거나 공개하지 않았습니다.

```bash
git init -b main
git add .
git commit -m "Add Park Compass prototype and proposal"
git remote add origin <your-repository-url>
git push -u origin main
```

공개 전에 실제 시설의 주차면 좌표, 차량 위치 로그, 방문자 정보, 개인 연락처가 저장소에 포함되지 않았는지 확인하세요. 현재 시제품에는 가상 데이터만 들어 있습니다.

