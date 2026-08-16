# 공장 자동 정비 지시 예시

[예시 목록](README.md) | [병원예약 예시](hospital-reservation.md)

이 예시는 OntoFlow가 센서 이벤트를 받아 자동으로 정비 지시를 만드는 흐름을 보여줍니다.

병원 예약 예시가 사람이 버튼이나 폼으로 실행하는 업무 흐름이라면, 이 예시는 설비 상태 변화가 워크플로우를 시작하는 자동화 흐름입니다.

이 예시는 별도의 승인 절차를 포함하지 않습니다. 센서 이벤트와 기준 조건이 충족되면 `Workflow Manager`가 정비 지시를 자동 생성합니다. 단, 실제 설비 제어처럼 위험도가 높은 작업은 후속 승인 컴포넌트와 결합할 수 있습니다.

## 시나리오

컨베이어 설비에서 짧은 시간 안에 반복 정지가 감지됩니다. 시스템은 설비 이벤트를 받아 증상과 원인 후보를 확인하고, 정비팀에 작업 지시를 생성합니다.

```text
센서 이벤트 수신
  -> 반복 장애 조건 확인
  -> 설비 타입과 증상 기준 조회
  -> 원인 후보와 조치 후보 선택
  -> 정비 지시 생성
  -> 설비 상태와 정비 이력 갱신
  -> 담당자 알림
```

## 객체 타입

- `Equipment`
- `SensorEvent`
- `FaultSymptom`
- `FaultCause`
- `MaintenanceAction`
- `WorkOrder`
- `Technician`
- `Notification`

## 객체 인스턴스 예시

```text
Equipment           Conveyor-01
SensorEvent         EVT-2026-001
FaultSymptom        repeated_stop
FaultCause          belt_tension_low
MaintenanceAction   adjust_belt_tension
WorkOrder           WO-2026-001
```

## 관계

- `Equipment HAS_EVENT SensorEvent`
- `Equipment HAS_SYMPTOM FaultSymptom`
- `FaultSymptom INDICATES_CAUSE FaultCause`
- `FaultCause SUGGESTS_ACTION MaintenanceAction`
- `WorkOrder TARGETS Equipment`
- `WorkOrder REQUESTS MaintenanceAction`
- `WorkOrder ASSIGNED_TO Technician`

## 액션

- `receiveSensorEvent`
- `detectRepeatedFault`
- `classifyFaultSymptom`
- `selectMaintenanceAction`
- `createWorkOrder`
- `notifyTechnician`
- `updateEquipmentStatus`

## 워크플로우

```text
SensorEventReceived
  -> 반복 장애 조건 확인
  -> 증상 분류
  -> 원인 후보 선택
  -> 조치 후보 선택
  -> 정비 지시 생성
  -> 설비 상태 갱신
  -> 담당자 알림
  -> End
```

## 상태 슬롯

```text
equipment.id
equipment.status
sensor_event.id
sensor_event.event_type
sensor_event.detected_at
fault.count_in_window
fault.symptom_code
fault.cause_code
maintenance.action_code
work_order.id
work_order.status
notification.status
```

## 자동 실행 조건

```text
event_type = stop
same_equipment_stop_count >= 3
time_window_minutes = 30
equipment.status != maintenance_in_progress
```

조건이 충족되면 워크플로우가 자동으로 시작됩니다.

## 생성되는 정비 지시 예시

```text
작업 지시: WO-2026-001
대상 설비: Conveyor-01
감지 증상: 30분 내 반복 정지 3회
추정 원인: 벨트 장력 저하
권장 조치: 벨트 장력 점검 및 기준 범위로 조정
우선순위: 높음
상태: created
```

## 상태 변경 이력

이 예시의 핵심은 자동 실행 자체가 아니라, 자동 실행 중 값과 상태가 추적된다는 점입니다.

```text
State Slot History
  fault.count_in_window: 2 -> 3
  fault.symptom_code: null -> repeated_stop
  fault.cause_code: null -> belt_tension_low
  work_order.status: null -> created

Action Execution Log
  detectRepeatedFault    success
  selectMaintenanceAction success
  createWorkOrder        success
  notifyTechnician       success

Ontology Change History
  Equipment.status: running -> maintenance_requested
  WorkOrder.status: null -> created
```

## 병원 예약 예시와의 차이

| 구분 | 병원 예약 | 공장 자동 정비 지시 |
|---|---|---|
| 시작 방식 | 사람이 버튼이나 폼으로 시작 | 센서 이벤트가 자동 시작 |
| 주요 입력 | 환자, 의사, 예약 시간 | 설비, 이벤트, 반복 장애 조건 |
| 주요 결과 | 예약 생성 | 정비 지시 생성 |
| 핵심 설명 | 사람 주도 업무 실행 | 이벤트 기반 자동화 |
| 추적 대상 | 예약 상태와 알림 상태 | 설비 상태, 장애 판정, 정비 이력 |

두 예시는 같은 구조를 공유합니다.

```text
입력 발생
  -> 상태 슬롯 채움
  -> 온톨로지 기준 검증
  -> 액션 실행
  -> 실제 업무 데이터 반영
  -> 이력 저장
```

차이는 입력 방식입니다. 병원 예약은 사람이 시작하고, 공장 정비 지시는 이벤트가 시작합니다.
