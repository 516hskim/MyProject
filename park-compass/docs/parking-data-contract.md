# 주차장 데이터 API 계약 초안

이 문서는 아파트·주상복합 운영사가 실시간 주차 정보를 제공한다는 가정 아래 작성한 연동 제안입니다. 엔드포인트와 필드는 실제 운영사 API 명세가 아닙니다.

## 기본 식별자와 좌표

- `facilityId`: 시설 고유 ID
- `floorId`: 층 고유 ID (`B2`, `1F` 같은 표기와 분리 가능)
- `stallId`: 주차면 ID
- `nodeId`: 차량 통로 또는 보행 네트워크 노드
- `mapVersion`: 층별 지도 버전
- 좌표: `mapVersion`에 속하는 로컬 좌표 `x`, `y`, `heading`

주차면 좌표와 통로 좌표는 동일한 로컬 좌표계에 있어야 합니다. 층을 넘는 램프는 층 간 연결 edge로 표현합니다.

## 제안 API

### 시설 지도

```http
GET /v1/facilities/{facilityId}/map?mapVersion=current
```

반환 예시:

```json
{
  "facilityId": "apt-001",
  "mapVersion": "2026-11-01",
  "floors": [{"floorId": "B2", "label": "지하 2층"}],
  "nodes": [{"nodeId": "b2-corridor-12", "floorId": "B2", "x": 42.5, "y": 18.0, "type": "LANE"}],
  "edges": [{"from": "b2-entry", "to": "b2-corridor-12", "direction": "FORWARD", "lengthM": 21.0}],
  "stalls": [{"stallId": "B2-C12", "floorId": "B2", "nodeId": "b2-corridor-12", "x": 45.0, "y": 19.0, "allowedTypes": ["RESIDENT", "VISITOR"]}]
}
```

### 빈자리 상태

```http
GET /v1/facilities/{facilityId}/availability?floorId=B2&accessType=VISITOR&destinationId=102
```

반환 예시:

```json
{
  "facilityId": "apt-001",
  "mapVersion": "2026-11-01",
  "generatedAt": "2026-11-15T12:15:20Z",
  "stalls": [
    {"stallId": "B2-C12", "status": "FREE", "updatedAt": "2026-11-15T12:15:18Z", "features": ["VISITOR"]},
    {"stallId": "B2-C13", "status": "TAKEN", "updatedAt": "2026-11-15T12:15:16Z", "features": []}
  ]
}
```

허용 상태는 `FREE`, `TAKEN`, `RESERVED`, `RESTRICTED`, `UNKNOWN`으로 정규화합니다. 상태와 함께 `updatedAt`을 보내야 앱이 오래된 `FREE`를 최신 정보처럼 안내하지 않습니다.

### 변경 이벤트

```text
WebSocket /v1/facilities/{facilityId}/availability/stream
```

이벤트에는 `facilityId`, `mapVersion`, `stallId`, `status`, `updatedAt`, `eventId`를 포함합니다. 연결 실패 시 단기 폴링으로 복구하고, 마지막 수신 시각을 화면에 표시합니다.

### 주차 세션 저장

```http
POST /v1/parking-sessions
```

```json
{
  "sessionId": "ephemeral-session-id",
  "facilityId": "apt-001",
  "stallId": "B2-C12",
  "parkedAt": "2026-11-15T12:20:00Z",
  "mapVersion": "2026-11-01"
}
```

차량 번호는 기본 데이터 모델에 넣지 않습니다. 운영사가 접근 제어에 별도 식별자를 요구하면 목적과 보존 기간을 정한 뒤 분리 저장해야 합니다.

## 추천 처리 규칙

1. 시설이 반환한 이용 자격과 차량 제한을 먼저 필터링합니다.
2. `FREE`이며 데이터 갱신이 허용 시간 안인 면만 경로 후보로 삼습니다.
3. 차량 진입점에서 후보면까지의 통로 그래프 경로와 목적 동까지의 보행 경로를 계산합니다.
4. 최단 주행, 짧은 보행, 적은 회전, 데이터 최신성, 위치 신뢰도를 합산해 순위를 정합니다.
5. 안내 직전에 상태를 다시 확인하고, 점유로 바뀌면 재탐색합니다.
6. 예약 API가 없는 시설에서는 주차면 예약을 보장한다고 표현하지 않습니다.

## 오류 처리

| 상황 | 앱 동작 |
|---|---|
| 상태 갱신이 늦음 | `정보가 지연됨` 표시, 구역 단위 추천 |
| 주차면 상태가 `UNKNOWN` | 추천 제외 |
| 선택 면이 점유됨 | 다음 후보 계산, 음성/큰 카드로 안내 |
| 실내 위치 신뢰도 낮음 | 층/구역을 확인 요청 |
| 지도 버전 불일치 | 특정 면 경로 중지, 시설 지도 다시 동기화 |
| 네트워크 단절 | 마지막 갱신 시각을 보여주고 실시간 확정 표현 금지 |

