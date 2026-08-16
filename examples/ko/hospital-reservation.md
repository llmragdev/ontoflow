# 병원예약 예시

[예시 목록](README.md) | [공장 자동 정비 지시 예시](factory-maintenance-auto.md)

이 예시는 OntoFlow가 일반적인 업무 흐름을 어떻게 모델링하는지 보여줍니다.

## 객체 타입

- `Patient`
- `Doctor`
- `Department`
- `Appointment`
- `Notification`

## 관계

- `Patient HAS_APPOINTMENT Appointment`
- `Department HAS_DOCTOR Doctor`
- `Doctor HAS_SLOT Appointment`

## 액션

- `verifyPatient`
- `findAvailableSlots`
- `createAppointment`
- `sendNotification`

## 워크플로우

```text
Start
  -> 환자 확인
  -> 진료과 또는 의사 선택
  -> 가능한 예약 시간 조회
  -> 예약 시간 확정
  -> 예약 생성
  -> 알림 발송
  -> End
```

## 상태 슬롯

```text
patient.name
patient.birth_date
patient.phone
department.code
doctor.id
appointment.requested_time
appointment.available_slots
appointment.status
notification.status
```

## 영속 지식

워크플로우 단계가 예약 값을 계산했다고 해서 곧바로 영속 지식이 바뀌지는 않습니다.

영속 지식은 Apply 경계를 통과할 때만 바뀝니다.

- `object_instances`
- `property_values`
- `relation_instances`
- `ontology_change_history`

## 대화식 입력

사용자가 다음처럼 말할 수 있습니다.

```text
내일 오전에 심장내과 예약하고 싶어요.
```

대화식 입력 컴포넌트는 다음 슬롯 후보를 만들 수 있습니다.

```text
department.code = cardiology
appointment.requested_time = 내일 오전
```

그래도 실행, 검증, 상태 슬롯, 액션 호출은 Workflow Manager가 담당합니다.
