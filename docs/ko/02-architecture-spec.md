---
title: "OntoFlow Architecture Specification"
author: OntoFlow maintainers
status: final
created_at: 2026-08-16 03:08
scope: "타입 기준, 영속 지식, Apply 경계, 단일 검증기, 일괄 등록, 바인딩, 실행 상태, 근거, 대화 입력 계약, Gate 확장 계약의 구현 기준"
audience: "백엔드·프론트엔드 개발자, DB 설계자, 오픈소스 기여자"
overview_doc: "01-overview.md"
license_note: "본 문서는 공개 배포를 전제로 작성한다. 특정 기업의 소스·식별자·제품명·제3자 실명을 포함하지 않는다."
---

# OntoFlow 상세 설계

> 구현자가 테이블과 계약으로 바로 읽는 문서다.
> 제품 방향과 컴포넌트 경계는 개괄 설계 문서에서 다룬다.
> 확정된 계약은 §1–§18, §20에, 확정 대기 항목은 §19에 있다.

---

## 1. 문서 범위

### 1.1 구현 수준 표기

```text
◆ 코어      2026년 실무 구현 범위
▲ 모듈      조립형 입력 컴포넌트. 계약 고정, 구현은 모듈 단위
○ 확장      초기 검증 범위 + 계약 고정. 고도화는 별도 범위
```

### 1.2 확장 계약을 먼저 고정하는 이유

```text
Gate와 대화 입력은 선택 컴포넌트지만 실행 상태와 슬롯 계약에 직접 연결된다.
따라서 코어 단계에서 다음 계약을 먼저 고정한다.

Gate / WriteBack 이 추가되어도 실행 상태 생명주기는 유지된다.
Conversational Input 이 추가되어도 State Slot 구조는 유지된다.
```

**구현 우선순위와 무관하게 외부 컴포넌트가 붙는 계약면은 유지한다.**

### 1.3 명명 규칙

```text
api_name       식별자. 영문·숫자·언더스코어. 변경하지 않는다
display_name   화면 표기. 다국어 대상. 자유롭게 바꾼다
```

정의 테이블의 유일 키는 `api_name`으로 잡는다. `display_name`이 바뀌어도 참조는 그대로 유지된다.

---

## 2. 계층 구조

### 2.1 5계층

```text
[L1] 타입 기준                   ◆ 온톨로지 매니저              → §3
     ObjectType · PropertyDefinition · SharedPropertyDefinition
     RelationType · ActionType · WorkflowType

[L2] 실행 상태                   ◆ 워크플로우 매니저            → §9
     WorkflowRun · StepRun · RoleBinding · StateSlot
     StateSlotHistory · ActionExecutionLog

[L3] 변경 후보                   ○ Gate 확장                   → §15
     CandidateGroup · ChangeCandidate

[L4] 영속 지식                   ◆ 온톨로지 매니저              → §4
     ObjectInstance · PropertyValue · RelationInstance
     OntologyChangeHistory

[L5] 근거                        ◆ 코어                        → §11
     문서 근거 · 온톨로지 경로 근거 · 상태변경 근거 · 발화 근거
```

**L3만 확장이다.** L4에는 네 진입 경로가 있고(§4.2), Gate는 그중 하나에만 개입한다.

### 2.2 생명주기

| 계층 | 수명 | 변경 통제 |
|---|---|---|
| L1 | 영속, 버전 관리 | 제안 → 영향 분석 → 승인 → 적용 (§7) |
| L2 | 실행 단위, 보존 정책 적용 | 실행 중 쓰기 가능, 종료 후 정리 |
| L3 | 승인·반려·반영까지 | 상태 전이만 |
| L4 | 영속, 이력 누적 | 모든 경로가 Apply 경계 통과 (§4.3) |
| L5 | 영속 또는 감사 보존 기간 | **생성 후 수정 금지** |

### 2.3 이전값

| 용도 | 출처 | 성격 |
|---|---|---|
| 승인 화면의 이전값 | 후보 생성 시점의 L4 현재값 | **불변 스냅샷** |
| 이력 조회의 직전값 | 변경 이력의 직전 레코드 | 시간축 조회용 |

승인 대기 중 영속 값이 바뀌면 충돌로 분기한다(§15.5).

---

## 3. 타입 기준 ◆

### 3.1 스코프 — 전역과 프로젝트

**전역 기준은 예약 프로젝트에 담는다.**

```text
project_id = "__base__"   전역 기본 기준. 읽기 전용
project_id = <프로젝트>    해당 프로젝트의 정의
```

| 규칙 | 내용 |
|---|---|
| 상속 | 프로젝트는 전역 기준을 상속한다 |
| 확장 | 속성·관계·액션을 추가할 수 있다 |
| 약화 금지 | 전역이 정한 `required`·범위 제약을 **완화할 수 없다** |
| 이름 충돌 | 프로젝트 정의가 우선하되 충돌을 경고로 표시한다 |

**`project_id`를 null로 두지 않는다.** null이 유일 키에 들어가면 유일성 강제가 무너지고 모든 조회 경로에 null 분기가 생긴다.

### 3.2 상태 두 축

```text
정의 수명주기   status          DRAFT | ACTIVE | DEPRECATED | RETIRED
변경 절차       proposal_state  PROPOSED | IMPACT_ANALYZED | APPROVED | REJECTED
```

| `status` | 의미 |
|---|---|
| `DRAFT` | 제안 승인 전. 참조 가능하나 승격 대상 불가 |
| `ACTIVE` | 정상 사용 |
| `DEPRECATED` | 신규 참조 금지, 기존 참조 유지 |
| `RETIRED` | 참조 불가. 이력 조회만 |

`proposal_state`는 §7의 변경 제안이 갖는다. 정의 자체가 갖지 않는다.

### 3.3 `object_types`

```text
object_types
  project_id / object_type_id
  api_name / display_name / description
  parent_type_id            상속이 필요할 때만
  primary_key_property_id   외부 식별에 쓰는 속성
  title_property_id         화면 표기에 쓰는 속성 (§4.4)
  status / version
  created_at / updated_at
유일 키  (project_id, api_name)
```

하위 타입은 상위 타입의 필수 속성을 약화할 수 없다.

### 3.4 `shared_property_definitions`

```text
shared_property_definitions
  project_id / shared_property_id
  api_name / display_name / description
  value_type / unit
  min_value / max_value / enum_values / pattern
  source_reference
  status / version
유일 키  (project_id, api_name)
```

| 규칙 | 내용 |
|---|---|
| 파생 | `property_definitions.shared_property_id`로 연결한다 |
| 제약 강화 | 파생 측은 범위를 **좁힐 수만** 있다 |
| 제약 완화 | 금지. `value_type` 변경도 금지 |
| 영향 분석 | 공유 속성 변경은 파생 전체를 대상으로 계산한다 |

### 3.5 `property_definitions`

```text
property_definitions
  project_id / property_id / object_type_id
  shared_property_id        공유 속성에서 파생된 경우
  api_name / display_name
  value_type                string | number | boolean | date | enum | reference
  unit / required
  min_value / max_value / enum_values / pattern
  source_reference          이 정의의 출처
  status / version
유일 키  (project_id, object_type_id, api_name)
```

**JSON 내부 필드가 아니라 별도 행으로 관리한다.** 슬롯 선언·드롭다운·값 검증·영향 분석이 모두 이 행을 직접 참조한다.

### 3.6 `relation_types`

```text
relation_types
  project_id / relation_type_id
  api_name / display_name
  domain_type_id / range_type_id
  cardinality               1:1 | 1:N | N:N
  from_side_api_name / from_side_display_name
  to_side_api_name   / to_side_display_name
  source_reference
  status / version
유일 키  (project_id, api_name)
```

**양방향 탐색을 위해 side 메타데이터를 둔다.** 역관계 이름 하나만으로는 화면과 API 양쪽을 안정적으로 지원하기 어렵다.

### 3.7 `action_types`

```text
action_types
  project_id / action_type_id
  api_name / display_name / description
  kind                      rest_call | tool_call | internal_service
                            | rule_eval | human_task
  connector_id              호출 대상 (§3.8)
  input_schema              논리 입력 이름 + 필수 여부
  output_schema             논리 출력 이름 + 타입
  required_object_roles     이 액션이 소비하는 대상 역할
  produced_object_roles     이 액션이 생성하는 대상 역할 (§3.9)
  side_effect_level         read_only | writes_external
                            | writes_ontology | unknown        (§12.5)
  dry_run_behavior          simulate | execute | forbid
  writeback_required        영속 반영을 요구하는가
  approval_required         승인 필수 여부
  idempotency_key_spec      재실행 안전성
  max_attempts / timeout_policy
  status / version
유일 키  (project_id, api_name)
```

`kind` 5종을 한 체계로 관리한다. **노드는 액션 유형만 고르고 호출 프로토콜을 알 필요가 없다.**

**`side_effect_level`은 `kind`와 다른 축이다.** `rest_call`이 조회일 수도 변경일 수도 있으므로 부작용 수준을 따로 선언한다. 기본값은 `unknown`이며, `unknown`인 액션은 dry-run에서 실행되지 않는다.

### 3.8 `action_connectors`

```text
action_connectors
  project_id / connector_id
  connector_type            rest | tool | internal
  api_name / display_name
  target_ref                엔드포인트 식별자 또는 도구 이름
  environment               dev | stage | prod
  auth_profile_ref
  request_schema / response_schema
  sandbox_target_ref        dry-run 예외 허용 시 사용 (§12.5)
  timeout_ms / retry_policy / rate_limit
  status
유일 키  (project_id, api_name, environment)
```

**호출 주소와 인증 정보를 타입 기준에 직접 넣지 않는다.**

| 이유 | 설명 |
|---|---|
| 보안 | 기준 쓰기 권한이 곧 임의 주소 호출 권한이 되는 것을 막는다 |
| 환경 분리 | 같은 식별자가 환경마다 다른 주소를 가리킨다 |
| 변경 격리 | 주소 변경이 타입 변경 절차를 타지 않는다 |

### 3.9 `produced_object_roles`

액션은 기존 객체를 소비할 뿐 아니라 **새 객체를 만들 수 있다.**

```text
createAppointment
  required_object_roles   patient, doctor
  produced_object_roles   appointment
```

| 규칙 | 내용 |
|---|---|
| 바인딩 시점 | 액션이 성공한 뒤 해당 역할에 새 객체가 바인딩된다 |
| 역할 선언 | 그 역할의 `binding_source`는 `created_by_action`이어야 한다(§10.1) |
| 정적 검사 | 생성 역할과 소비 역할이 모두 정의에 선언되어야 한다(§10.5) |
| 실패 시 | 액션이 실패하면 역할은 `unbound`로 남는다 |
| 생성 경로 | 액션이 Apply 경계를 호출해 `object_instances` 행을 만든다(§4.3) |

### 3.10 `workflow_types`

```text
workflow_types
  project_id / workflow_type_id
  api_name / display_name / category
  allowed_action_types      사용 가능한 ActionType 목록
  allowed_object_types      역할이 받을 수 있는 타입 범위
  required_object_roles     정의가 반드시 선언해야 할 역할 (§10.5)
  required_slot_declarations
  approval_policy
  default_timeout_policy
  status / version
유일 키  (project_id, api_name)
```

### 3.11 실행 전용 슬롯

슬롯 유형을 별도 기준으로 두지 않는다. 영속 지식과 연결되는 슬롯은 `(object_type, property_definition)`에서 파생된다.

```text
transient 슬롯
  기준 연결 없음
  영속 반영 불가
  runtime_type 기준 검증 (§5.4)
  실행 종료 시 정리 대상
```

---

## 4. 영속 지식 ◆

### 4.1 저장소

#### `object_instances`

```text
object_instances
  project_id / object_id / object_type_id
  external_key              외부 시스템 식별자
  title                     title_property 에서 파생된 표기 (§4.4)
  title_source              property | external_key | object_id
  lifecycle_status          active | inactive | archived | deleted
  source_system / source_record_ref
  current_version
  created_at / updated_at
유일 키  (project_id, object_type_id, external_key)
```

워크플로우 실행 슬롯의 `target_object_id`가 가리키는 대상이 이 테이블의 `object_id`다.

#### `property_values`

```text
property_values
  project_id / property_value_id
  object_id / property_id
  value_json / value_type / unit
  value_status              current | superseded | deleted
  source_system / source_record_ref
  valid_from / valid_to
  version / updated_at
유일 키       (project_id, property_value_id)
현재값 유일   (project_id, object_id, property_id) where value_status = current
이력 인덱스   (project_id, object_id, property_id, updated_at)
```

**현재값은 `current` 행 하나로 유지한다.** 값이 바뀌면 기존 `current`를 `superseded`로 닫고 새 `current`를 만든다.

#### `relation_instances`

```text
relation_instances
  project_id / relation_id / relation_type_id
  from_object_id / to_object_id
  relation_status           active | inactive | deleted
  source_system / source_record_ref
  valid_from / valid_to
  version / updated_at
유일 키       (project_id, relation_id)
활성 중복방지  (project_id, relation_type_id, from_object_id, to_object_id)
              where relation_status = active
이력 인덱스   (project_id, relation_type_id, from_object_id, to_object_id, updated_at)
```

### 4.2 네 진입 경로

**L4를 바꾸는 주체는 넷이다.** 경로마다 앞단 절차가 다르지만 **모두 Apply 경계에서 만난다.**

```text
① ABox 일괄 등록
   파일 / API / UI  →  ImportJob  →  Validator  →  Apply
                                                   origin_kind = import_abox

② ABox 직접 편집
   관리자 화면  →  ontology_edit_allowed 확인  →  Validator  →  Apply
                                                   origin_kind = direct_edit_abox

③ TBox 변경          (일괄·단건·직접 편집 전부 이 경로)
   제안 생성  →  영향 분석  →  승인  →  Apply
                                                   origin_kind = type_change_approval

④ 워크플로우 승격
   StateSlot  →  승격 판정  →  Gate 후보  →  승인  →  WriteBack  →  Apply
                                                   origin_kind = workflow_writeback
```

| | 같다 | 다르다 |
|---|---|---|
| 검증 기준 | **동일** — Unified Validator(§5) | — |
| Apply 경계 | **동일** — 같은 저장소, 같은 제약(§4.3) | — |
| 변경 이력 | **동일** — `ontology_change_history`(§4.5) | — |
| 입력 형식 | — | 파일 행 / 폼 필드 / 제안 정의 / 슬롯값 |
| 실패 처리 | — | 항목 스킵 / 저장 거부 / 승인 불가 / 승격 불가 |
| 사전 절차 | — | 없음 / 권한 확인 / **제안·영향분석·승인** / Gate 승인 |

**TBox는 예외 없이 ③을 탄다.** 일괄 등록이든 관리자 화면의 단건 수정이든, 정의를 바꾸는 모든 요청은 `type_change_proposals`를 생성하고 승인을 거친다.

### 4.3 Apply 경계

모든 경로가 통과하는 단일 진입점이다.

```text
ApplyRequest
  project_id
  change_scope        tbox | abox
  target_kind         object_type | property_definition | shared_property_definition
                      | relation_type | action_type | workflow_type
                      | object_instance | property_value | relation_instance
  change_type         create | update | deactivate | delete | restore
  target_id           update 이상일 때
  payload             적용할 내용
  origin_kind         import_abox | direct_edit_abox
                      | type_change_approval | workflow_writeback
  origin_ref
  proposal_id         change_scope = tbox 일 때 필수
  import_job_id       import 경로일 때
  candidate_id        workflow_writeback 일 때
  provenance_id
  changed_by / change_reason
```

**Apply 절차**

```text
0. change_scope = tbox 이면 proposal_id 필수. 없으면 거부
1. change_scope = tbox 이면 type_change_proposals.proposal_state = APPROVED 확인
2. change_scope = abox 이면 proposal_id 금지
3. Unified Validator 호출                          (§5)
4. 변경 전 스냅샷 확보
5. 저장소 갱신
     property_values     기존 current 를 superseded 로 닫고 새 current 생성
     relation_instances  활성 중복 방지 제약 확인 후 갱신
     object_instances    title 갱신 규칙 적용                (§4.4)
6. ontology_change_history 기록                    (§4.5)
7. 결과 반환
```

**0~2단계가 TBox 승인을 데이터로 강제한다.** 승인된 제안 없이는 정의가 바뀌지 않는다.

**검증에 실패하면 5단계에 도달하지 않는다.** Apply 경계에는 우회가 없다.

### 4.4 `title` 갱신 규칙

`object_instances.title`은 `title_property_id`에서 파생된 값이다. **파생값을 저장하므로 갱신 규칙이 필요하다.**

```text
property_value Apply 시
  변경된 property_id 가 object_types.title_property_id 와 같으면
  같은 트랜잭션에서 object_instances.title 을 새 current 값에서 재계산한다.
```

| 상황 | 처리 | `title_source` |
|---|---|---|
| `title_property` 값 변경 | 같은 트랜잭션에서 `title` 갱신 | `property` |
| `title_property` 값 삭제 | fallback 적용 | `external_key` |
| `title_property` 미설정 | `external_key`, 없으면 `object_id` | `external_key` / `object_id` |
| **title 계산 실패** | **`title_property`가 설정된 경우 blocking error** | — |

`title_property`를 설정해 두고 계산에 실패하면 Apply를 중단한다. 조용히 fallback으로 넘기면 화면에 잘못된 표기가 남고 원인을 추적할 수 없다.

### 4.5 `ontology_change_history`

```text
ontology_change_history
  change_id / project_id
  change_scope              tbox | abox
  target_kind
  target_id
  change_type               create | update | deactivate | delete | restore
  previous_snapshot / next_snapshot
  origin_kind               import_abox | direct_edit_abox
                            | type_change_approval | workflow_writeback
  proposal_id
  import_job_id
  workflow_run_id / workflow_step_run_id / action_execution_id
  candidate_id
  provenance_id
  changed_by / change_reason
  changed_at
인덱스  (project_id, change_scope, target_kind, target_id, changed_at)
        (project_id, import_job_id)
        (project_id, workflow_run_id)
        (project_id, proposal_id)
```

**생성 후 수정하지 않는다.**

### 4.6 후보와 이력은 다르다

| 대상 | 성격 | 생성 시점 | 소유 |
|---|---|---|---|
| `change_candidates` | **제안** — 승인 대기 | 워크플로우 승격 시 | Gate 확장 |
| `ontology_change_history` | **사실** — 실제 일어난 변경 | **Apply 될 때마다 항상** | 온톨로지 매니저 |

**후보는 반려될 수 있으므로 이력이 아니다.** 이력은 승인 여부와 무관하게 남는다.

```text
① ABox 일괄 등록  → 후보를 거치지 않고 이력을 남긴다
② ABox 직접 편집  → 후보를 거치지 않고 이력을 남긴다
③ TBox 변경       → 제안 승인 후 이력을 남긴다 (proposal_id 로 연결)
④ 워크플로우      → 후보가 COMMITTED 될 때 이력을 남긴다 (candidate_id 로 연결)
```

### 4.7 두 이력의 구분

| 이력 | 의미 | 소유 |
|---|---|---|
| `state_slot_history` | 워크플로우 실행 중 슬롯값 변경 (§9.10) | Workflow Manager |
| `ontology_change_history` | 기준 또는 영속 인스턴스가 **실제로** 바뀐 기록 | Ontology Manager |

워크플로우가 값을 계산했다고 해서 곧바로 영속 인스턴스가 바뀌지 않는다. **영속 반영은 Apply 경계에서만 일어난다.**

---

## 5. Unified Validator ◆

### 5.1 검증기는 하나다

타입·범위·열거값·패턴·필수값·참조 무결성·관계 카디널리티 검증은 **하나의 컴포넌트가 담당한다.**

```text
Validator (단일)
  ├─ Apply 경계가 호출                   (§4.3)
  ├─ 슬롯 쓰기가 호출                    (§9.7)
  ├─ 워크플로우 dry-run 이 호출           (§12.5)
  ├─ 승격 판정이 호출                    (§12.2)
  ├─ 일괄 등록 검증이 호출                (§6)
  └─ TBox 변경 제안의 영향 분석이 호출     (§7.3)
```

### 5.2 호출자마다 다른 것은 실패 처리뿐이다

| 호출자 | 실패 처리 |
|---|---|
| Apply 경계 | 적용 중단 |
| 슬롯 쓰기 | 거부 또는 보류 |
| dry-run | 리포트에 실패 항목 기록 |
| 일괄 등록 | `atomic`이면 전체 실패, `partial`이면 항목 스킵 |
| 승격 판정 | 승격 불가 |
| TBox 영향 분석 | 영향 분석 실패 또는 승인 불가 |

### 5.3 `ValidationResult`

```text
ValidationResult
  valid                 통과 여부
  severity              error | warning | info
  rule_id               적용된 규칙 식별자
  rule_type             type | required | enum | range | pattern
                        | reference | uniqueness | cardinality | inheritance
  target_kind
  target_id
  object_type_id / property_id / relation_type_id
  expected              기대값 또는 제약
  actual                실제값
  message
  blocking              진행을 막는가
```

**모든 리포트가 같은 형식을 쓴다.** dry-run 리포트, 일괄 등록 리포트, 영향 분석 리포트가 하나의 표시 컴포넌트를 공유한다.

`severity = warning`이고 `blocking = false`인 항목은 기록하되 진행을 막지 않는다.

### 5.4 검증 기준의 두 축

`slot_scope`는 **기준 연결 여부**, `validation_basis`는 **실제 검증 경로**다.

| 검증 경로 | `slot_scope` | `validation_basis` | `schema_version_id` | 검증 내용 |
|---|---|---|:---:|---|
| 기준 검증 | `bound` | `tbox_schema` | **필수** | 타입·범위·열거·필수·참조·카디널리티 |
| 런타임 검증 | `transient` | `runtime_type` | null | 값 타입 일치만 |
| 우회 | `bound`/`transient` **유지** | `bypassed` | null | 검증 안 함. 사유 필수 |

**우회는 슬롯의 종류를 바꾸지 않는다.** 원래 `slot_scope`를 유지해야 재검증 배치가 "기준 검증을 받아야 했던 슬롯"을 찾을 수 있다.

`bypassed`는 슬롯 쓰기 경로에만 존재한다. **Apply 경계에는 우회가 없다.**

### 5.5 규칙 유형

| `rule_type` | 검사 | 최소 규칙셋 |
|---|---|:---:|
| `type` | `value_type` 일치 | ● |
| `required` | 필수 속성 존재 | ● |
| `enum` | `enum_values` 포함 | ● |
| `range` | `min_value` / `max_value` | ● |
| `pattern` | 정규식 일치 | ● |
| `reference` | 참조 대상 객체·타입 존재와 `status` | ● |
| `uniqueness` | `external_key`, `api_name` 중복 | ● |
| `cardinality` | 관계 `1:1` / `1:N` / `N:N` 위반 | ● |
| `inheritance` | 하위 타입의 필수 속성 약화, 공유 속성 제약 완화 | |

**최소 규칙셋 8종이 Core-3의 범위다**(§18.2). 이후 단계에서는 호출자와 리포트가 늘어날 뿐 **검증 결과 형식과 규칙 식별자는 바뀌지 않는다.**

---

## 6. 일괄 등록 ◆

### 6.1 `ontology_import_jobs`

```text
ontology_import_jobs
  project_id / import_job_id
  import_kind               tbox | abox | mixed
  input_channel             file | api | ui | system
  source_name / source_payload_ref
  requested_by
  dry_run
  apply_mode                atomic | partial
  proposal_id               TBox 경로에서 생성된 변경 제안 (§7)
  status                    RECEIVED | VALIDATING | VALIDATED
                            | PROPOSAL_CREATED | APPROVED
                            | APPLYING | APPLIED | PARTIAL_FAILED
                            | FAILED | CANCELLED
  total_count / valid_count / invalid_count
  applied_count / error_count
  created_at / validated_at / applied_at
```

`PROPOSAL_CREATED`와 `APPROVED`는 **TBox 경로에서만** 사용한다.

### 6.2 `ontology_import_items`

```text
ontology_import_items
  project_id / import_job_id / import_item_id
  row_no
  target_kind               object_type | property_definition
                            | shared_property_definition
                            | relation_type | action_type | workflow_type
                            | object_instance | property_value | relation_instance
  target_api_name
  target_object_external_key
  normalized_payload
  validation_status         PENDING | VALID | INVALID | WARNING
  validation_errors         ValidationResult 배열 (§5.3)
  apply_status              PENDING | APPLIED | SKIPPED | FAILED
  target_id                 적용 후 생성·갱신된 대상
  created_change_id         ontology_change_history 참조
유일 키  (project_id, import_job_id, import_item_id)
```

### 6.3 처리 흐름

```text
입력 수신
  → 파싱
  → 정규화                    ontology_import_items 생성
  → TBox / ABox 대상 분류
  → Unified Validator 호출     (§5)
  → dry-run 리포트             ValidationResult 집계
  → 분기
       ABox  →  Apply 호출     origin_kind = import_abox
       TBox  →  type_change_proposals 생성  (§7)
                → 영향 분석 → 승인
                → Apply 호출  origin_kind = type_change_approval
  → ontology_change_history 기록
```

### 6.4 ABox와 TBox의 통제 강도

| | ABox 일괄 적재 | TBox 일괄 변경 |
|---|---|---|
| 대상 | 인스턴스·속성값·관계 | 객체·속성·공유속성·관계·액션·워크플로우 타입 |
| dry-run | 가능 | 가능 |
| `apply_mode` | `atomic` / `partial` 선택 | **`atomic` 고정** |
| 승인 | **불필요** | **필수** |
| Apply `origin_kind` | `import_abox` | `type_change_approval` |
| `status` 흐름 | `VALIDATED → APPLYING → APPLIED` | `VALIDATED → PROPOSAL_CREATED → APPROVED → APPLYING → APPLIED` |

### 6.5 TBox 일괄 변경이 승인을 경유하는 이유

**속성 하나를 바꾸는 데도 승인이 필요한데, 500개를 한 번에 바꾸는 것이 검증만으로 통과하면 통제 강도가 거꾸로 된다.**

```text
TBox 는 검증 기준 자체를 바꾼다.
기준이 바뀌면 기존 데이터 전체의 유효성이 흔들린다.
규모가 클수록 영향 분석이 더 중요해진다.
```

### 6.6 `atomic` / `partial`

| 모드 | 의미 | 적용 대상 |
|---|---|---|
| `atomic` | 하나라도 실패하면 전체 미적용 | **TBox 전체**, 관계 구조 변경 |
| `partial` | `VALID` 항목만 적용하고 실패 항목 보고 | ABox 대량 적재 |

ABox도 사용자가 `atomic`을 선택할 수 있다. **TBox는 `partial`을 선택할 수 없다** — 정의 일부만 적용되면 기준이 불완전한 상태로 남는다.

### 6.7 재적용 안전성

```text
dry_run = true 로 검증한 뒤 같은 source_payload_ref 로 적용한다
검증 시점과 적용 시점 사이에 기준이 바뀌면 재검증한다
status = APPLIED 인 job 은 다시 적용할 수 없다
```

`import_job_id`가 재적용 단위다.

---

## 7. 타입 변경 절차 ◆

### 7.1 `type_change_proposals`

```text
type_change_proposals
  proposal_id / project_id
  proposed_by / proposed_at
  origin_kind               single_edit | import
  import_job_id             origin_kind = import 일 때 (§6.1)
  change_kind               add_object_type | add_property | add_shared_property
                            | add_relation | add_action | add_workflow_type
                            | modify_constraint | deprecate | retire
  target_object_type_id / target_api_name
  proposed_definition
  rationale
  impact_summary
  affected_definitions      참조 중인 정의·노드·인스턴스 목록
  proposal_state            PROPOSED | IMPACT_ANALYZED | APPROVED | REJECTED
  decided_by / decided_at
  applied_change_ids        Apply 후 생성된 이력 참조
```

### 7.2 TBox를 바꾸는 유일한 경로

**정의를 바꾸는 모든 요청이 이 절차를 탄다.**

```text
단건 편집    관리자 화면  →  제안 생성  →  영향 분석  →  승인  →  Apply
일괄 변경    ImportJob    →  제안 생성  →  영향 분석  →  승인  →  Apply
```

**TBox 직접 편집 화면은 Apply를 직접 호출하지 않는다.** 화면이 하는 일은 제안을 만드는 것까지다. `ontology_edit_allowed`가 있어도 정의는 바뀌지 않는다(§13).

### 7.3 상태 전이

| 현재 | 사건 | 다음 | 필요 권한 |
|---|---|---|---|
| — | 제안 생성 (단건 또는 import) | `PROPOSED` | `type_change_propose_allowed` |
| `PROPOSED` | 영향 분석 완료 | `IMPACT_ANALYZED` | — |
| `IMPACT_ANALYZED` | 승인 | `APPROVED` | `type_change_approve_allowed` |
| `IMPACT_ANALYZED` | 반려 | `REJECTED` | `type_change_approve_allowed` |
| `PROPOSED` / `IMPACT_ANALYZED` | 철회 | `REJECTED` | 제안자 또는 승인자 |

**`APPROVED` 없이 Apply 되지 않는다**(§4.3-1). `IMPACT_ANALYZED`를 건너뛴 승인도 허용하지 않는다.

### 7.4 영향 분석의 범위

Unified Validator를 **가상 적용 모드**로 호출해 계산한다.

| 변경 | 역조회 대상 |
|---|---|
| 속성 제약 변경 | 이 속성을 참조하는 슬롯 선언 · 현재 슬롯값 · `property_values` |
| 공유 속성 변경 | 파생 속성 정의 전체 + 그 아래 모든 참조 |
| 관계 타입 변경 | `relation_instances` · 규칙 경로 |
| 액션 타입 변경 | `node_action_bindings` · 진행 중 실행 |
| 워크플로우 타입 변경 | 소속 정의 · 역할 선언 |
| 정의 폐기 | 위 전부 + `DEPRECATED` 유예 대상 |
| 객체 타입 변경 | 하위 타입 · 인스턴스 · 모든 속성 정의 |

`affected_definitions`는 §3.5가 속성을 별도 행으로 둔 덕에 역조회로 계산된다.

### 7.5 `pending_type`

| 상태 | 조건 | 처리 |
|---|---|---|
| `validated` | 기준 존재 + 검증 통과 | 승격 가능 |
| `pending_type` | 기준 없음, 제안 대기 | **값 보관 가능, 승격 불가** |
| `invalid` | 기준 존재 + 검증 실패 | 승격 불가 |

`pending_type`은 초기 설계 마찰을 줄이는 완충 장치일 뿐 기준 우회로가 아니다. **실행 단위별 비율 상한**을 둔다(D-6).

---

## 8. 값 전달과 역할 참조 ◆

### 8.1 이름 기반 Blackboard

| 방식 | 값 전달 | 캔버스 |
|---|---|---|
| Dataflow | 노드 출력 → 연결선 → 노드 입력 | 데이터 선 + 제어 선 |
| **Blackboard** | 실행 상태에 이름으로 쓰고 읽는다 | **제어 선만** |

**단 자유 문자열 키가 아니라 기준과 역할에 결속된 참조만 허용한다.**

### 8.2 공용 상태 패턴의 위험과 대응

| 위험 | 대응 |
|---|---|
| 이름 충돌 | 슬롯 이름을 타입 기준에 결속(§12.1) |
| 타입 불일치 | Unified Validator(§5) |
| 유일성 없음 | 실행·역할·이름 복합 키(§9.7) |
| 병렬 쓰기 덮어쓰기 | 행 단위 저장. 충돌이 키 위반으로 드러난다 |
| 누락 무시 | Fail-Closed 승격 조건(§12.2) |
| **요청 간 간섭** | **실행 단위 식별자를 키에 포함**(§9.7) |
| 흐름 불명 | `filled_by` + `filled_by_node_id` 역조회 |

여섯 번째가 특히 중요하다. 공용 상태를 다루는 구현에서 요청·실행 단위 격리 실패는 흔한 결함이며, 병렬 처리를 도입할수록 발생 확률이 올라간다.

### 8.3 역할 한정 슬롯 참조

```text
SlotRef
  object_role
  slot_name

축약형   subject.장력 · source.장력 · target.장력
```

**역할을 생략하면 `subject`로 해석한다.** 단일 대상 워크플로우는 `장력`으로 읽히고, 다중 대상만 역할을 드러낸다. 축약 문자열을 입력에서 허용하더라도 **저장은 구조화된 `{object_role, slot_name}`으로 정규화한다.**

### 8.4 제어 선

```text
control_edges
  project_id / workflow_definition_id / edge_id
  from_node_id / to_node_id
  edge_kind          sequence | branch | loop_back | parallel_split | parallel_join
  condition_expr     branch 일 때의 분기 조건
  order_index
유일 키  (project_id, workflow_definition_id, edge_id)
```

캔버스의 선은 **실행 순서만** 뜻한다.

### 8.5 단계 결과 객체를 두지 않는다

액션 응답은 `output_schema`와 노드의 `result_mapping`에 따라 곧바로 슬롯에 기록된다. 원응답은 액션 실행 로그(§9.11)에 남는다.

---

## 9. 실행 상태 계층 ◆

### 9.1 `workflow_definitions`

```text
workflow_definitions
  project_id / workflow_definition_id / workflow_type_id
  api_name / display_name / description
  status                    DRAFT | ACTIVE | DEPRECATED | RETIRED
  version
  created_at / updated_at
유일 키  (project_id, api_name)
```

### 9.2 `workflow_nodes`

```text
workflow_nodes
  project_id / workflow_definition_id / node_id
  node_type                 action | decision | input | output
                            | human_task | subworkflow
  display_name / config
  position_x / position_y
유일 키  (project_id, workflow_definition_id, node_id)
```

### 9.3 `workflow_runs`

```text
workflow_runs
  project_id / run_id
  parent_run_id / run_kind      root | child
  workflow_definition_id / workflow_definition_version
  workflow_type_id
  iteration_key                 하위 실행의 순회 키
  trigger_kind                  button | conversation | schedule | event | api
  run_mode                      normal | dry_run
  session_id                    대화 세션에서 시작된 경우 (§14.2)
  status                        RUNNING | WAITING | FAILED | COMPLETED | CANCELLED
  started_by / started_at / ended_at
```

**`trigger_kind`와 `run_mode`는 다른 축이다.** 대화로 시작한 dry-run이 표현 가능해야 한다.

### 9.4 순회는 하위 실행으로 나눈다

```text
부모 실행   대상 목록 생성
   ├─ 하위 실행 1   subject → A-04    슬롯 집합 독립
   ├─ 하위 실행 2   subject → B-11    슬롯 집합 독립
   └─ 하위 실행 3   subject → C-27    슬롯 집합 독립
```

**역할을 `subject_1`, `subject_2`로 늘리지 않는다.** 선언이 대상 개수에 묶여 정의를 재사용할 수 없게 된다.

### 9.5 `workflow_step_runs`

```text
workflow_step_runs
  project_id / run_id / step_run_id
  node_id / attempt_no
  status                    PENDING | RUNNING | SUCCESS | FAILED | SKIPPED
  entered_at / started_at / ended_at
  error_code / error_message
유일 키  (project_id, run_id, step_run_id)
```

**액션이 없는 노드도 여기 기록된다.** 분기·검증·사람 작업 노드의 진입 이력이 §12.3 실행 경로 판정의 근거가 된다.

### 9.6 `workflow_run_role_bindings`

```text
workflow_run_role_bindings
  project_id / run_id / object_role
  object_type_id
  target_object_id
  binding_status            unbound | bound | locked
  bound_at / locked_at
  locked_by_slot_state_id
  created_by_step_run_id    produced_object_roles 로 생성된 경우 (§3.9)
유일 키  (project_id, run_id, object_role)
```

**역할 바인딩 불변 규칙**

| 현재 | 사건 | 다음 | 조건 |
|---|---|---|---|
| `unbound` | 인스턴스 확정 | `bound` | 타입이 `object_type_id`와 일치 |
| `unbound` | 액션이 객체 생성 | `bound` | `produced_object_roles`에 선언됨 |
| `bound` | 인스턴스 교체 | `bound` | **슬롯이 하나도 채워지지 않았을 때만** |
| `bound` | 첫 슬롯 기록 | **`locked`** | `locked_by_slot_state_id` 기록 |
| `locked` | 인스턴스 교체 | — | **거부. 실행 오류** |

**`locked`는 되돌아가지 않는다.** 풀 수 있게 하면 불변 규칙이 우회 가능한 권고로 전락한다.

### 9.7 `workflow_state_slots`

```text
workflow_state_slots
  slot_state_id / project_id / run_id / session_id
  object_role
  slot_name
  object_type_id / property_id
  target_object_id              역할 바인딩의 복사본 (§9.8)
  slot_scope                    bound | transient
  value_type
  target_value / origin_value / unit
  filled_by                     §9.9
  filled_by_node_id / filled_by_step_run_id
  confidence                    추출 신뢰도. 확정 입력은 null (§14.5)
  ask_state                     not_asked | asked | answered | refused | skipped
  ask_count
  validated
  validation_basis              tbox_schema | runtime_type | bypassed
  schema_version_id
  validation_detail             ValidationResult 배열 (§5.3)
  confirmed
  filled_at / updated_at
유일 키  (project_id, run_id, object_role, slot_name)
```

**`ask_state`가 없으면 "아직 안 물어봄"과 "물어봤는데 답을 안 함"이 구분되지 않는다.**

### 9.8 `target_object_id`는 복사본이다

```text
외래 키    (project_id, run_id, object_role) → workflow_run_role_bindings
기록 시점  슬롯에 첫 값이 채워질 때 1회 복사
이후       수정 불가. 역할 바인딩도 이 시점에 locked 가 된다
```

두 값이 다르면 데이터 오류이며 승격 조건(§12.2-8)에서 걸린다.

### 9.9 `filled_by` — 채움 경로

| 값 | 의미 | `confidence` | 문서 근거 |
|---|---|:---:|:---:|
| `user_input` | 버튼·폼 직접 입력 | null | 없음 |
| **`conversation`** | **대화 발화에서 추출** | **필수** | 없음 |
| `extraction` | 문서에서 추출 | 필수 | **필수** |
| `action_response` | 외부 호출 응답 | null | 선택 |
| `prior_step` | 이전 슬롯 참조 | 상속 | 상속 |
| `rule_eval` | 규칙 평가 결과 | null | **필수** |
| `default_value` | 선언의 기본값 | null | 없음 |

### 9.10 `state_slot_history`

```text
state_slot_history
  history_id / project_id / run_id
  slot_state_id / seq
  object_role / slot_name
  previous_target_value / next_target_value
  previous_origin_value / next_origin_value
  filled_by / confidence
  changed_by_node_id / changed_by_step_run_id
  changed_by_action_execution_id
  changed_by_turn_id            대화 입력일 때 (§14.3)
  change_reason
  validated / validation_basis / validation_detail
  changed_at
유일 키  (project_id, slot_state_id, seq)
인덱스   (project_id, run_id, object_role, slot_name, changed_at)
```

### 9.11 `action_execution_logs`

```text
action_execution_logs
  action_execution_id / project_id
  run_id / step_run_id / node_id
  action_type_id / connector_id
  attempt_no / idempotency_key
  run_mode                      normal | dry_run
  request_payload_ref           마스킹 정책 적용
  response_payload_ref
  response_origin               live | simulated | sandbox      (§12.5)
  status                        PENDING | RUNNING | SUCCESS | FAILED
                                | TIMEOUT | CANCELLED
  started_at / ended_at / duration_ms
  error_code / error_message
  resulting_slot_state_ids
```

**`response_origin`으로 모의 응답과 실제 응답을 구분한다.** dry-run 로그가 실제 호출 로그와 섞이면 감사에서 구분할 수 없다.

### 9.12 `rerun_requests`

```text
rerun_requests
  rerun_id / project_id
  original_run_id / original_step_run_id
  rerun_type                manual | auto
  rerun_seq
  use_definition_version    원본 버전 유지 또는 신규 버전 선택
  requested_by / reason / created_at
  new_run_id
```

**재수행은 같은 입력과 같은 워크플로우 버전을 기준으로 시작한다.**

### 9.13 정규화값과 원본값

```text
origin_value : 입력된 그대로
target_value : 동의어·정규식·타입 변환·단위 변환을 거친 대표값
```

정규화 규칙이 바뀌면 `origin_value`로 재처리할 수 있다.

---

## 10. 바인딩 계층 ◆

| 바인딩 | 축 | 정의/노드 ↔ 기준 |
|---|---|---|
| 역할 선언 | 대상 | 정의 ↔ `ObjectType` |
| 슬롯 선언 | 데이터 | 노드 ↔ `PropertyDefinition` |
| 액션 바인딩 | 실행 | 노드 ↔ `ActionType` |

### 10.1 `workflow_definition_roles`

```text
workflow_definition_roles
  project_id / workflow_definition_id
  object_role
  object_type_id            이 역할이 받을 수 있는 타입
  requirement               required | optional
  binding_source            caller_input | prior_slot | query
                            | conversation | created_by_action
  produced_by_action_type_id  binding_source = created_by_action 일 때 (§3.9)
  declared_at
유일 키  (project_id, workflow_definition_id, object_role)
```

### 10.2 `node_slot_declarations`

```text
node_slot_declarations
  declaration_id / project_id
  workflow_definition_id / node_id
  direction               read | write
  object_role
  slot_name
  object_type_id / property_id
  requirement             mandatory | optional
  writeback_target        write 일 때 영속 반영 대상인지
  default_source
  ask_order               되묻기 우선순위 (§14.4)
  ask_policy              always | if_missing | if_low_confidence | never
  prompt_template         질문 문구 템플릿
  confirm_policy          never | if_low_confidence | always
  declared_at
유일 키  (project_id, workflow_definition_id, node_id,
          direction, object_role, slot_name)
```

**`write`가 곧 승격은 아니다.** 중간 계산값과 `transient`는 `writeback_target = false`다.

### 10.3 `node_action_bindings`

```text
node_action_bindings
  binding_id / project_id
  workflow_definition_id / node_id
  action_type_id
  invocation_mode         sync | async | deferred
  parameter_mapping       action_input_name  -> { object_role, slot_name }
  result_mapping          action_output_name -> { object_role, slot_name }
  produced_role_mapping   produced_object_role -> object_role (§3.9)
  on_failure              halt | retry | skip | branch
  retry_policy
유일 키  (project_id, workflow_definition_id, node_id)
```

> **노드 하나는 액션 하나를 실행한다.** 유일 키가 이를 강제한다.
> 액션 둘이 필요하면 노드를 나눈다.

### 10.4 `action_types.input_schema`와의 관계

| 대상 | 층 | 의미 |
|---|---|---|
| `input_schema` | 타입 | 액션 유형이 요구하는 논리 입력 |
| `parameter_mapping` | 인스턴스 | 그 입력을 어느 슬롯 참조로 채우는지 |
| `node_slot_declarations` | 인스턴스 | 노드가 실제로 읽고 쓰는 슬롯 |

`input_schema`가 필수로 정한 입력은 노드에서 optional로 낮출 수 없다.

### 10.5 정적 검사

워크플로우 저장 또는 실행 전에 검사한다.

#### 정의 수준

| # | 검사 | 실패 처리 |
|---:|---|---|
| 1 | `workflow_definition.status`가 `ACTIVE`인가 | 실행 거부 |
| 2 | **`workflow_types.required_object_roles`가 전부 `workflow_definition_roles`에 선언되었는가** | **저장 거부** |
| 3 | **역할의 `object_type_id`가 `allowed_object_types` 범위 안인가** | **저장 거부** |
| 4 | 제어 선이 도달 불가 노드를 남기지 않는가 | 경고 |

#### 노드 수준

| # | 검사 | 실패 처리 |
|---:|---|---|
| 5 | `action_type_id`가 `allowed_action_types` 안에 있는가 | 저장 거부 |
| 6 | **`action_types.required_object_roles`가 정의 역할 안에 있는가** | **저장 거부** |
| 7 | **`action_types.produced_object_roles`가 정의 역할로 선언되어 있는가** | **저장 거부** |
| 8 | **`binding_source = created_by_action` 역할을 생산하는 ActionType이 존재하는가** | **저장 거부** |
| 9 | `parameter_mapping`의 `{object_role, slot_name}`이 `read` 선언에 있는가 | 저장 거부 |
| 10 | `result_mapping`의 `{object_role, slot_name}`이 `write` 선언에 있는가 | 저장 거부 |
| 11 | 선언의 `slot_name`이 해당 객체 타입의 속성 정의에 있는가 | 타입 추가 제안(§7) |
| 12 | 선언의 `object_role`이 `workflow_definition_roles`에 있는가 | 저장 거부 |
| 13 | 참조하는 기준 정의의 `status`가 `RETIRED`가 아닌가 | 저장 거부 |
| 14 | `input_schema`의 필수 입력이 전부 매핑되었는가 | 저장 거부 |

**6·7·8이 `produced_object_roles`와 `created_by_action`을 양방향으로 묶는다.** 액션이 만든다고 선언한 역할과 정의가 생성을 기대하는 역할이 서로를 가리켜야 한다.

**역할까지 대조하므로 다음이 실행 전에 잡힌다.**

```text
선언 :  read  source.장력
매핑 :  액션 입력 ← target.장력      ← 선언되지 않은 역할. 거부
```

---

## 11. 근거 계층 ◆

### 11.1 세 종류

| 근거 | 답하는 질문 |
|---|---|
| 문서 근거 | 이 지식이 원문 어디에 있는가 |
| 온톨로지 경로 근거 | 이 판단이 어떤 타입·관계 경로로 연결되는가 |
| 상태변경 근거 | 이 후보가 어떤 이전값·이후값·승인 상태를 갖는가 |

**서로 대체하지 않는다.** "문서에 있다"와 "기준으로 승인되었다"는 다른 말이다.

| 하나만 있을 때 | 위험 |
|---|---|
| 문서 근거만 | 유사 문장은 찾았으나 대상 객체와 연결되는 경로가 없다 |
| 경로 근거만 | 관계는 있으나 왜 기준인지 설명할 수 없다 |
| 상태변경 근거만 | 후보값은 있으나 기준값·출처가 없어 승인자가 판단할 수 없다 |

### 11.2 `provenance_records`

```text
provenance_records
  provenance_id / project_id
  provenance_kind    document | ontology_path | state_change | rule
                     | action_log | utterance | import
  source_ref / source_location / source_excerpt
  created_by / created_at
  immutable_hash
  supersedes_id      정정 시 이전 근거 참조
```

**생성 후 수정하지 않는다.** 정정이 필요하면 새 근거를 만들고 `supersedes_id`로 관계를 남긴다.

### 11.3 `slot_provenance_links`

```text
slot_provenance_links
  link_id / project_id
  slot_state_id / provenance_id
  evidence_kind      document | ontology_path | state_change | utterance
  weight
유일 키  (project_id, slot_state_id, provenance_id)
```

### 11.4 근거 계약

```text
[필수]   모든 슬롯은 "이 값이 왜 여기 있는지" 를 설명할 수 있어야 한다.
[조건부] 그 설명이 반드시 문서 근거 참조일 필요는 없다.
```

**항상 채워지는 설명 필드:** `filled_by` · `filled_by_node_id` · `filled_at` · `validation_basis` · `validated`/`validation_detail`.

### 11.5 `filled_by`별 근거 요구

| `filled_by` | 설명 형태 | 문서 근거 |
|---|---|:---:|
| `user_input` | 사용자 식별자 + 시각 | 없음 |
| `conversation` | 발화 근거 + 신뢰도 | 없음 (발화 근거 필수) |
| `extraction` | 문서 위치 + 원문 조각 | **필수** |
| `action_response` | 실행 로그 참조 | 선택 |
| `prior_step` | 이전 슬롯 참조 | 상속 |
| `rule_eval` | 적용 규칙 식별자 | **필수** |
| `default_value` | 선언 참조 | 없음 |

---

## 12. 경계 규칙과 승격 판정 ◆

### 12.1 기준 사용 규칙

| 행위 | 처리 |
|---|---|
| 노드 추가, 제어 선 연결, 배치 수정 | 워크플로우 정의 변경. 가볍게 |
| 기존 슬롯·액션 타입 참조 | 허용 |
| 기준에 없는 슬롯 이름 사용 | 타입 추가 제안(§7) |
| 허용되지 않은 액션 타입 사용 | 거부 |
| 선언되지 않은 역할 한정 참조 사용 | 거부 |
| 속성 타입·제약 변경 | 타입 변경 절차 |
| 관계 타입·액션 타입 신설 | 타입 변경 절차 |

### 12.2 Fail-Closed 승격 조건

```text
1. 실행된 제어 경로의 mandatory read 슬롯이 전량 존재한다        (§12.3)
2. 승격 대상 슬롯이 전량 validated = true 이며
   validation_basis 가 bypassed 가 아니다                       (§12.4)
3. confirm_policy 가 요구하는 슬롯은 confirmed = true 이다
4. filled_by 별 근거 요구 수준을 충족한다                        (§11.5)
5. 승격 대상은 writeback_target = true 인 write 슬롯뿐이다        (§10.2)
6. 승격 대상 슬롯에 object_role 과 target_object_id 가 있다      (§9.7)
7. 참조한 기준 정의의 status 가 RETIRED 가 아니다               (§3.2)
8. 역할 바인딩이 locked 이며 슬롯의 target_object_id 와 일치한다  (§9.8)
9. ask_state 가 refused 인 mandatory 슬롯이 없다                (§14.4)
10. confidence 임계 이하인 슬롯은 confirmed = true 이다          (§14.5)
11. run_mode 가 dry_run 이 아니다                              (§12.5)
```

**이 조건을 평가하는 것은 코어다.** Gate가 없어도 워크플로우 매니저는 "이 실행이 반영 가능한 상태를 만들었는가"를 답해야 한다.

```text
코어      조건 평가 → 통과/미통과 + 미충족 항목 목록
Gate 확장 통과한 것을 후보로 만들어 승인 절차에 올림
```

### 12.3 검사 범위는 실행된 경로다

```text
      ┌── B (mandatory: 원인코드)
A ────┤
      └── C (mandatory: 반려사유)

B 로 분기한 실행에서 C 의 반려사유를 요구하면 승격이 영원히 불가능하다.
```

```text
실행 경로 = 이 run 에서 workflow_step_runs 에 기록된 노드의 집합   (§9.5)
반복 노드는 여러 번 진입해도 한 번으로 센다
진입했으나 중단된 노드는 포함한다
```

### 12.4 우회는 승격 경로가 아니다

우회(`validation_basis = bypassed`)는 **슬롯을 채우는 단계 전용**이다.

**긴급 반영이 필요하면 슬롯을 우회하지 않는다.** §4.2의 ② ABox 직접 편집 경로를 쓰거나, 승인 권한자가 `origin_kind = manual` 변경 후보를 만든다(§15.1).

| 경로 | 검증 | 승인 | 이력 |
|---|:---:|:---:|:---:|
| 정상 승격 | ○ | ○ | ○ |
| 슬롯 우회 | ✕ | — | ○ (승격 불가) |
| ABox 직접 편집 | **○** | — | ○ |
| 수동 후보 | ✕ | **○** | ○ |

우회 시 다음을 기록한다.

```text
수행자 · 사유 · 이전값 · 이후값 · 시각 · 건너뛴 검증 항목
```

### 12.5 dry-run과 부작용 정책

```text
run_mode = dry_run 일 때
  상태 슬롯은 정상적으로 채운다
  Unified Validator 를 호출한다
  액션은 side_effect_level 과 dry_run_behavior 에 따른다
  Fail-Closed 조건을 평가한다
  변경 후보를 만들지 않는다
  Apply 경계를 호출하지 않는다
```

**`side_effect_level`이 `dry_run_behavior`를 제한한다.**

| `side_effect_level` | 허용 dry-run | 기본값 |
|---|---|---|
| `read_only` | `simulate` 또는 `execute` | `execute` |
| `writes_external` | 기본 `simulate`. 명시 허용 없으면 `execute` 금지 | `simulate` |
| `writes_ontology` | **`execute` 금지** | `simulate` |
| `unknown` | **`execute` 금지** | `simulate` |

```text
dry_run_behavior = execute 는 side_effect_level = read_only 일 때만 기본 허용한다.
writes_external 인 액션은 connector 의 sandbox_target_ref 가 있을 때만 예외 허용한다.
dry-run 중 writes_ontology 는 항상 금지한다 — Apply 경계를 통하지 않는 쓰기는 없다.
```

`dry_run_behavior = forbid`인 노드에 dry-run이 도달하면 실행을 중단하고 리포트에 사유를 남긴다.

**검증 리포트 항목**

| 항목 | 출처 |
|---|---|
| 실행된 제어 경로 | §9.5 |
| 채워진 슬롯 / 미충족 필수 슬롯 | §10.2 + §12.2-1 |
| 슬롯별 `ValidationResult` | §5.3 |
| 근거가 붙은 슬롯 / 붙지 않은 슬롯 | §11.3 |
| 액션 호출 요약과 `response_origin` | §9.11 |
| **승격 가능 여부와 미충족 항목 목록** | §12.2 |
| 반영되었다면 바뀌었을 값 | §15.2 형태로 **계산만** |

마지막 항목은 **저장하지 않는다.**

---

## 13. 권한 축

### 13.1 네 갈래

| 권한 | 대상 | 의미 | 범위 |
|---|---|---|:---:|
| `workflow_allowed` | 워크플로우 실행 | 절차를 실행해도 되는가 | ◆ |
| `ontology_edit_allowed` | **ABox 직접 편집** | 영속 인스턴스 값을 직접 수정할 수 있는가 | ◆ |
| `type_change_propose_allowed` | **TBox 제안** | 타입 변경 제안을 만들 수 있는가 | ◆ |
| `type_change_approve_allowed` | **TBox 승인** | 영향 분석된 타입 변경을 승인할 수 있는가 | ◆ |
| `writeback_allowed` | Gate WriteBack | 승인 후보를 영속 반영할 수 있는가 | ○ |

**`ontology_edit_allowed`로는 정의를 바꿀 수 없다.** TBox는 제안 권한과 승인 권한을 따로 탄다(§7.2).

### 13.2 확인 지점

```text
ABox 직접 편집
  관리자 화면
    → ontology_edit_allowed 확인          ← 실패하면 ApplyRequest 를 만들지 않는다
    → Unified Validator 호출
    → ApplyRequest(origin_kind = direct_edit_abox)
    → ontology_change_history 기록

TBox 변경
  관리자 화면 또는 ImportJob
    → type_change_propose_allowed 확인
    → type_change_proposals 생성
    → 영향 분석
    → type_change_approve_allowed 확인
    → ApplyRequest(origin_kind = type_change_approval, proposal_id)
```

**권한 실패 시 `ApplyRequest`를 만들지 않는다.** Apply 경계에 도달한 뒤 거부하는 것이 아니라 그 앞에서 막는다.

### 13.3 권한과 준비 상태

```text
[주체 권한] writeback_allowed    이 사용자가 후보를 반영해도 되는가    ○
[후보 상태] writeback_ready      이 후보가 반영 가능한 상태인가        ○

writeback_executable = writeback_allowed AND writeback_ready
```

| 축 | 바뀌는 조건 | 누가 물어도 같은 답인가 |
|---|---|:---:|
| 주체 권한 | 역할·정책이 바뀔 때 | **아니다** — 사용자마다 다르다 |
| 후보 상태 | 승인 생명주기가 진행될 때 | **그렇다** — 후보에만 의존 |

**검색·추천 응답에 실리는 것은 `writeback_ready`다.** 응답은 후보 상태를 말할 뿐 그 사용자의 권한을 말하지 않는다.

---

## 14. 대화식 입력 계약 ▲

> 대화 모듈은 **부품**이다. 코어는 이 장 없이 동작해야 한다.

### 14.1 진입 순서

```text
1. 의도 인식        어느 workflow_definition 인가
2. 실행 시작        run 발급 (trigger_kind = conversation)
3. 역할 바인딩      추출된 대상을 역할에 연결
4. 슬롯 채움        filled_by = conversation
5. 되묻기 루프      미충족 mandatory read (§14.4)
6. 실행 제안        사용자 확인 후 노드 실행
```

**의도 인식이 실행 시작보다 앞선다.** 어느 워크플로우인지 정해져야 `run_id`가 생기고, 그래야 슬롯을 담을 곳이 생긴다.

### 14.2 `conversation_sessions`

```text
conversation_sessions
  project_id / session_id
  user_id
  status                        COLLECTING | RESOLVED | WAITING_FOR_INPUT
                                | COMPLETED | CANCELLED
  resolved_workflow_definition_id
  resolved_run_id
  started_at / last_activity_at
```

| 상태 | 의미 |
|---|---|
| `COLLECTING` | 의도 미확정. 값은 후보로만 존재 |
| `RESOLVED` | 워크플로우 확정. 후보가 슬롯으로 이관됨 |
| `WAITING_FOR_INPUT` | 되묻기 중 |
| `COMPLETED` / `CANCELLED` | 종료. 후보는 정리 대상 |

### 14.3 `conversation_turns`와 후보 버퍼

```text
conversation_turns
  project_id / session_id / turn_id
  user_utterance / normalized_text
  detected_intent / workflow_candidate_id
  confidence
  status                    PARSED | NEEDS_CLARIFICATION | CONFIRMED | REJECTED
  created_at

conversation_slot_candidates
  project_id / session_id / turn_id / candidate_id
  object_role / slot_name
  candidate_object_type_id / candidate_property_id
  origin_value / target_value
  confidence
  extraction_basis
  accepted / accepted_at
  transferred_to_slot_state_id
```

**후보값은 검증되지 않았다.** 기준 연결도 역할 바인딩도 없다. **이관 시점에 비로소 Unified Validator를 통과한다.**

### 14.4 되묻기는 선언에서 파생된다

```text
무엇을 언제 물을지   →  node_slot_declarations 가 정한다   (코어)
어떻게 말할지        →  대화 모듈이 정한다                 (부품)
```

| 선언 필드 | 대화 모듈의 사용 |
|---|---|
| `ask_order` | 질문 순서 |
| `ask_policy` | 물을 조건 |
| `prompt_template` | 질문 문구의 원형 |
| `confirm_policy` | 확인 질문이 필요한지 |

**대화 모듈은 렌더러가 된다.** §12.5 dry-run이 만드는 미충족 항목 목록이 그대로 되묻기 큐다.

| 현재 | 사건 | 다음 |
|---|---|---|
| `not_asked` | 질문 발화 | `asked` |
| `asked` | 값 획득 | `answered` |
| `asked` | 사용자가 거절·회피 | `refused` |
| `asked` | 재질문 | `asked` (`ask_count` 증가) |
| 임의 | 정책상 묻지 않음 | `skipped` |

`refused`인 mandatory 슬롯이 있으면 승격되지 않는다(§12.2-9).

### 14.5 신뢰도

```text
confidence  0.0 ~ 1.0.  filled_by = conversation | extraction 일 때 필수
```

| 활용 | 규칙 |
|---|---|
| 확인 질문 | `confirm_policy = if_low_confidence` + 임계 이하 → 확인 |
| 되묻기 | `ask_policy = if_low_confidence` + 임계 이하 → 재질문 |
| 승격 | 임계 이하이고 `confirmed = false`면 승격 불가(§12.2-10) |

임계값은 프로젝트 설정이다(D-12).

### 14.6 발화 근거

```text
provenance_records
  provenance_kind = utterance
  source_ref       session_id
  source_location  turn_id
  source_excerpt   해당 발화 원문
```

`slot_provenance_links.evidence_kind = utterance`로 연결한다.

### 14.7 대화 모듈이 하지 않는 것

```text
워크플로우 정의를 바꾸지 않는다
타입 기준을 바꾸지 않는다
영속 지식에 직접 반영하지 않는다
액션을 우회 실행하지 않는다
되묻기 순서를 스스로 정하지 않는다
슬롯을 검증 없이 확정하지 않는다
```

**실패 시 폴백:** 대화가 실패해도 사용자는 버튼 화면에서 같은 슬롯을 직접 채운다. 그때 `filled_by`는 `user_input`으로 기록되고 `confidence`는 null이 된다.

---

## 15. Gate 확장 계약 ○

> 코어의 핵심 기능은 **이 장 없이 동작해야 한다** — §4.2의 ①②③ 경로가 영속 지식을 관리한다.

| 초기 검증 범위 | 후속 고도화 범위 |
|---|---|
| 후보 저장소, 그룹, 최소 승인 상태 | 다단계 승인, 권한 세분화 |
| 이전값·이후값·근거 화면 | 운영 대시보드, 감사 리포트 |
| WriteBack 모의 흐름 | 실제 운영 반영, 롤백 오케스트레이션 |

### 15.1 `candidate_groups`

```text
candidate_groups
  group_id / project_id
  origin_kind        workflow | manual
  run_id             origin_kind = workflow 일 때만
  workflow_definition_id / node_id
  created_by / created_at
  atomicity          all_or_nothing | independent
  status
  submitted_at / decided_by / decided_at
```

| `atomicity` | 승인 | 반영 |
|---|---|---|
| **`all_or_nothing`** (기본) | 그룹 전체를 함께 승인·반려 | 하나라도 실패하면 전체 롤백 |
| `independent` | 후보별 개별 승인 | 후보별 개별 반영 |

### 15.2 `change_candidates`

```text
change_candidates
  candidate_id / project_id
  group_id                    필수. 단건도 그룹 하나를 갖는다
  object_role / target_object_id
  target_kind                 property | relation

  # target_kind = property
  target_property_id
  previous_value / proposed_value

  # target_kind = relation
  target_relation_type_id
  relation_direction          outgoing | incoming
  operation                   add | remove | replace
  previous_object_id / proposed_object_id

  validation_status / writeback_ready
  status                      §15.4 의 7개 상태
  source_slot_state_id
  applied_change_id           COMMITTED 후 ontology_change_history 참조
  created_at / committed_at
```

### 15.3 `candidate_provenance_links`

```text
candidate_provenance_links
  link_id / project_id
  candidate_id / provenance_id
  evidence_kind
  copied_from_slot_state_id
유일 키  (project_id, candidate_id, provenance_id)
```

후보 근거는 **승인자가 본 근거**이므로 후보 생성 후 바꾸지 않는다.

### 15.4 승인 생명주기

> **초기 검증 범위는 `DRAFT` · `SUBMITTED` · `APPROVED` · `REJECTED`다.**
> `COMMITTED` · `COMMIT_FAILED` · `CONFLICTED`는 **계약만 고정**하고 고도화 시 구현한다.

```text
DRAFT ──▶ SUBMITTED ──▶ APPROVED ──▶ COMMITTED ●
                            ▲   │
               재시도 ───────┘   ├──▶ COMMIT_FAILED ──┐
                                 │         └── 재시도 ─┘
                                 └──▶ CONFLICTED ──▶ 새 후보 (DRAFT)

반려 가능 지점  SUBMITTED · APPROVED · COMMIT_FAILED · CONFLICTED  ──▶ REJECTED ●

● 종단 상태
```

| 현재 | 사건 | 다음 | 조건 |
|---|---|---|---|
| `DRAFT` | 제출 | `SUBMITTED` | 승인 최소 근거 충족(§15.6) |
| `SUBMITTED` | 승인 | `APPROVED` | 승인 권한 |
| `SUBMITTED` | 반려 | `REJECTED` | |
| `APPROVED` | 반영 성공 | `COMMITTED` | `writeback_executable` |
| `APPROVED` | 반영 오류 | `COMMIT_FAILED` | |
| `APPROVED` | 현재값 불일치 | `CONFLICTED` | |
| `APPROVED` | 승인 철회 | `REJECTED` | 반영 전에만 |
| `COMMIT_FAILED` | 재시도 | `APPROVED` | 원인 해소 |
| `COMMIT_FAILED` | 포기 | `REJECTED` | |
| `CONFLICTED` | 재판정 | **새 후보 `DRAFT`** | **자동 재시도 금지** |
| `CONFLICTED` | 포기 | `REJECTED` | |

**`CONFLICTED`는 같은 후보로 재시도하지 않는다.** 승인자가 본 이전값이 더 이상 사실이 아니므로 그 승인은 유효한 근거가 아니다.

`all_or_nothing` 그룹에서 후보 하나가 `CONFLICTED`가 되면 **그룹 전체가 `CONFLICTED`**다.

### 15.5 WriteBack

```text
전제
  status = APPROVED
  writeback_ready = true
  writeback_allowed = true           호출 시점에 호출자 기준으로 평가
  현재값이 previous_value 또는 기준 버전과 일치

수행
  Apply 경계 호출                    (§4.3, origin_kind = workflow_writeback)
  applied_change_id 기록

결과
  성공          COMMITTED
  처리 오류      COMMIT_FAILED
  현재값 불일치  CONFLICTED
```

**WriteBack은 저장소를 직접 건드리지 않는다.** Apply 경계를 호출하며, 검증과 이력 기록은 다른 경로와 동일하게 이루어진다.

### 15.6 승인 최소 근거

```text
대상 객체 · 대상 속성 또는 관계 · 이전값 · 이후값
생성 주체 · 검증 상태
문서 근거 또는 "문서 근거 없음"의 사유
온톨로지 경로 근거 · 적용한 기준 규칙
```

---

## 16. 자동화 트리거 계약 ○

**계약은 한 줄이다.**

```text
트리거는 실행을 시작한다. 그 뒤는 버튼식 실행과 동일하다.
```

```text
workflow_runs.trigger_kind = schedule | event
workflow_runs.parent_run_id = null
역할 바인딩은 트리거 페이로드가 제공한다
```

트리거 정의·조건식·재시도 정책·중복 억제는 후속 범위다.

---

## 17. 화면 계약

### 17.1 노드 선택 패널

| 항목 | 출처 | 범위 |
|---|---|:---:|
| 대상 역할 | `workflow_definition_roles.object_role` | ◆ |
| 대상 객체 | `workflow_run_role_bindings.target_object_id` | ◆ |
| 바인딩 상태 | `binding_status` | ◆ |
| 액션 유형 | `node_action_bindings` → `action_types` | ◆ |
| 부작용 수준 | `side_effect_level` | ◆ |
| 읽는 슬롯 / 쓰는 슬롯 | `node_slot_declarations` | ◆ |
| 슬롯 현재값 / 원본값 | `target_value` / `origin_value` | ◆ |
| 신뢰도 | `confidence` | ▲ |
| 질문 상태 | `ask_state` / `ask_count` | ▲ |
| 검증 상태 | `ValidationResult` 목록 | ◆ |
| 근거 | `slot_provenance_links` | ◆ |
| 슬롯 변경 이력 | `state_slot_history` | ◆ |
| 액션 호출 이력 | `action_execution_logs` + `response_origin` | ◆ |
| **승격 가능 여부** | **Fail-Closed 판정 + 미충족 항목**(§12.2) | **◆** |
| 이전값 → 이후값 | `change_candidates` 스냅샷 | ○ |
| 승인 상태 | 후보 및 그룹 생명주기 | ○ |
| 반영 가능 여부 | `writeback_ready` · `writeback_allowed` | ○ |

**Gate 행이 없어도 패널이 성립한다.**

### 17.2 표시 상태

| 상태 | 조건 | 화면 문구 |
|---|---|---|
| 정상 | 값 있음, 근거 접근 가능 | 정상 표시 |
| 미수집 | 슬롯이 비어 있음 | 입력 대기 — 오류가 아님 |
| 질문 대기 | `ask_state = asked` | 답변 대기 중 |
| 답변 거절 | `ask_state = refused` | 값을 받지 못함 — 대체 입력 필요 |
| 낮은 신뢰도 | `confidence` 임계 이하 | 확인 필요 |
| **검증 실패** | `validated = false`, `basis ≠ bypassed` | 검증 실패 + 위반 내역 |
| **검증 우회** | `validation_basis = bypassed` | **검증하지 않음 — 판단 근거로 사용되지 않음** |
| **모의 응답** | `response_origin ≠ live` | 모의 실행 결과 |
| 타입 미등록 | `pending_type` | 타입 추가 제안 필요 |
| 실행 전용 | `slot_scope = transient` | 실행 전용, 반영 대상 아님 |
| 대상 미지정 | `binding_status = unbound` | 대상 없음 |
| 근거 비공개 | 노출 정책에 따라 원문 숨김 | 정책에 따라 원문 비공개 |
| 충돌 | 현재값이 승인 당시 이전값과 다름 | 현재값 변경 — 재판정 필요 |

**검증 실패와 검증 우회를 같은 문구로 쓰지 않는다.** 우회는 실패가 아니라 미실시다.

### 17.3 버튼 상태

| 조건 | 반영 버튼 |
|---|---|
| `writeback_ready = false` | 비활성 + **후보 상태 사유** |
| `ready = true`, `allowed = false` | 비활성 + **권한 없음** |
| 둘 다 `true` | 활성 |

### 17.4 리포트 화면

**dry-run·일괄 등록·영향 분석이 같은 `ValidationResult` 형식을 쓴다**(§5.3). 하나의 표시 컴포넌트를 공유한다.

| 항목 | 출처 |
|---|---|
| 총 항목 수 / 유효 / 무효 / 경고 | 집계 |
| 항목별 검증 결과 | `ValidationResult` 배열 |
| 적용 모드와 예상 결과 | `apply_mode` |
| TBox 경로면 생성될 변경 제안 | `proposal_id` |
| 적용 후 결과 | `apply_status` / `created_change_id` |

---

## 18. 도입 순서

### 18.1 의존 관계

```text
타입 기준 조회 계층    →  슬롯 결속, 정적 검사, 검증 기준
영속 지식 저장소       →  Apply 경계의 대상
Apply 경계            →  영속 지식 저장소 + 변경 이력
Unified Validator     →  타입 기준 + 영속 지식
일괄 등록             →  Validator + Apply 경계
기준 변경·영향 분석    →  Validator (가상 적용 모드) + Apply 경계
워크플로우 정의·제어 선  →  실행 경로 기록의 전제
바인딩 계층           →  타입 기준 + 워크플로우 정의
역할 바인딩           →  실행 단위 + 역할 선언
상태 슬롯             →  역할 바인딩 + Validator + 근거 저장
─────────────────── 이하 조립 컴포넌트 ───────────────────
대화 세션·후보 버퍼    →  워크플로우 정의 + 슬롯 선언의 ask_* 필드
변경 후보 저장소      →  상태 슬롯 + 근거 저장 + Apply 경계
```

### 18.2 코어 단계와 산출물

| 단계 | 내용 | 이 문서의 절 |
|:---:|---|---|
| **Core-1** | 타입 기준 기본 구조 | §3.1 ~ §3.6, §3.10 |
| **Core-2** | 영속 지식 저장소 + Apply 경계 + 변경 이력 | §4.1, §4.3 ~ §4.5 |
| **Core-3** | **Unified Validator 최소 규칙셋** | §5 |
| **Core-4** | 일괄 등록 — Job·Item, ABox 적재, dry-run | §6.1 ~ §6.3, §6.6 ~ §6.7 |
| **Core-5** | TBox 변경 제안 · 영향 분석 · 승인 | §6.4 ~ §6.5, §7 |
| **Core-6** | 워크플로우 정의 · 역할 · 노드 · 제어 선 | §8.4, §9.1 ~ §9.2, §10.1 |
| **Core-7** | 슬롯 선언 · 액션 바인딩 · 정적 검사 | §3.7 ~ §3.9, §10.2 ~ §10.5 |
| **Core-8** | 실행 · 단계 실행 · 역할 바인딩 · 상태 슬롯 | §9.3 ~ §9.9 |
| **Core-9** | 슬롯 변경 이력 · 액션 실행 로그 · 근거 · 재수행 | §9.10 ~ §9.12, §11 |
| **Core-10** | 버튼식 실행 · dry-run 리포트 | §12.5, §17 |

### 18.3 조립 컴포넌트 단계

| 단계 | 내용 | 이 문서의 절 |
|:---:|---|---|
| Conv-P1 | 의도 인식 · 세션 · 후보 버퍼 | §14.1 ~ §14.3 |
| Conv-P2 | 되묻기 · 정규화 · 신뢰도 | §14.4, §14.5 |
| Conv-P3 | 발화 근거 · 실행 제안 | §14.6 |
| Gate-P1 | 변경 후보 최소 저장 | §15.1, §15.2 |
| Gate-P2 | 최소 승인 화면 | §15.3, §15.6 |
| Gate-P3 | WriteBack 모의 흐름 | §15.4, §15.5 |

### 18.4 순서의 근거

```text
Core-2 가 Core-6 보다 앞서는 이유
  워크플로우 실행 결과가 갈 곳이 없으면 검증도 승격도 판정할 수 없다.

Core-3 이 Core-4 보다 앞서는 이유
  일괄 등록이 첫 대량 검증 호출자다.
  검증기 없이 대량 데이터를 받으면 이후 판정 결과가 달라진다.
  Core-3 은 최소 규칙셋 8종(§5.5)으로 시작하되 인터페이스와
  ValidationResult 형식은 처음부터 확정한다.

Core-5 가 Core-6 보다 앞서는 이유
  워크플로우가 기준을 소비하려면 기준 변경 절차가 먼저 서 있어야 한다.
```

---

## 19. 확정 대기 항목

설계가 결론을 강요하지 않고 남긴 항목이다. 항목 번호는 `D-N`으로 고정하며, 추가·삭제 시에도 기존 번호를 재사용하지 않는다.

| # | 판단 | 선택지 | 범위 | 시점 |
|---|---|---|:---:|---|
| **D-1** | 전역 기준의 버전 승격 | 프로젝트가 명시적으로 올림 / 자동 추종 | ◆ | Core-1 |
| **D-2** | 공유 속성의 스코프 | 전역만 / 프로젝트 확장 허용 | ◆ | Core-1 |
| **D-3** | `DEPRECATED` 유예 기간 | 무기한 / 기간 후 자동 `RETIRED` | ◆ | Core-1 |
| **D-4** | 영속 값 이력 보존 기간 | 무기한 / 기간 후 아카이브 | ◆ | Core-2 |
| **D-5** | 일괄 등록 원본 보존 | 참조만 / 사본 보관 | ◆ | Core-4 |
| **D-6** | `pending_type` 비율 상한 | 미정 | ◆ | Core-7 |
| **D-7** | TBox 일괄 변경의 승인 단위 | Job 전체 / 항목별 | ◆ | Core-5 |
| **D-8** | **영향 분석의 인스턴스 검사 범위** | 전량 / 표본 / 비동기 전량 | ◆ | **Core-5** |
| **D-9** | 역할 바인딩 시점 | 실행 시작 시 일괄 / 노드 진입 시 지연 | ◆ | Core-8 |
| **D-10** | `transient` 슬롯 정리 정책 | 실행 종료 시 삭제 / 기간 보존 | ◆ | Core-8 |
| **D-11** | 액션 로그 페이로드 마스킹 | 전량 저장 / 필드 마스킹 / 미저장 | ◆ | Core-9 |
| **D-12** | 신뢰도 임계값 | 전역 고정 / 프로젝트별 / 슬롯별 | ▲ | Conv-P2 |
| **D-13** | 세션 버퍼 보존 기간 | 세션 종료 시 삭제 / 기간 보존 | ▲ | Conv-P1 |
| **D-14** | dry-run 리포트 보존 | 휘발 / 실행 단위 보존 / 추세 비교 | ◆ | Core-10 |
| **D-15** | 근거 원문 노출 정책 | 기본 차단 / 길이 제한 / 마스킹 | ◆ | Core-10 |
| **D-16** | `writes_external` sandbox 예외 승인 주체 | 액션 정의자 / 운영 관리자 | ◆ | Core-10 |
| **D-17** | 하위 실행의 승격 단위 | 하위 실행별 / 부모에서 일괄 | ○ | Gate-P1 |
| **D-18** | `CONFLICTED` 판정 기준 | 값 비교 / 버전·타임스탬프 비교 | ○ | Gate-P3 |
| **D-19** | 슬롯 조회 비용 | 미측정 | ◆ | §19.2 |

### 19.1 D-8은 Core-5 착수 전에 반드시 닫아야 한다

가장 이른 항목은 아니다. D-1~D-3이 Core-1이므로 시점상 앞선다. **다만 D-8은 난이도와 영향이 커서 해당 단계에 와서 결정하기에는 늦다.**

TBox 변경의 영향 분석이 **인스턴스를 전량 검사해야 하는가**는 성능과 안전성이 정면으로 부딪히는 지점이다.

```text
전량 검사   정확하지만 대량 데이터에서 승인이 오래 걸린다
표본 검사   빠르지만 승인 후 적용 시점에 실패가 드러난다
비동기 전량 승인은 빠르나 결과 통지 흐름이 별도로 필요하다
```

선택에 따라 `type_change_proposals`의 상태 집합과 화면 흐름이 달라진다. **Core-5 설계 착수 전에 확정한다.**

### 19.2 성능에 관한 주의

```text
타입 기준 결속은 쓰기 경로마다 조회를 늘린다.
슬롯 하나를 검증할 때 기준 조회 · 근거 연결 · 버전 확인 · 바인딩 확인이 붙는다.
Apply 경계는 여기에 스냅샷 확보와 이력 기록을 더한다.

Blackboard 패턴의 가치는 단순함이었다. 그 단순함을 잃지 않으려면
결속 요소를 한 번에 다 필수로 두지 말고 단계적으로 도입해야 한다.
```

인덱스나 조회 구조를 추가할 때는 **조회 개선과 쓰기 비용을 함께 측정해 보고한다.** 한쪽만 측정한 변경은 채택하지 않는다.

---

## 20. 용어

### 20.1 고지

> **본 문서의 용어는 이 설계의 자체 정의다.**
> `State Slot`, `WriteBack`, `Gate`, `ObjectType` 등 일부는 운영형 데이터·온톨로지 플랫폼의 어휘와
> 겹쳐 보일 수 있으나, **특정 제품의 공식 용어를 인용하거나 그 정의를 따르지 않는다.**

특히 `State Slot`은 널리 통용되는 표준 용어가 아니다. 운영형 플랫폼에서는 이 계층의 역할이 액션 파라미터·객체 속성값·제출 조건·실행 컨텍스트·감사 로그로 **나뉘어** 있는 경우가 많다. 한 계층으로 모으는 이유는 흩어져 있으면 "실행 중 값"과 "조직 지식"의 경계가 지점마다 다르게 그어지기 때문이다.

### 20.2 용어표

| 용어 | 의미 | 범위 |
|---|---|:---:|
| 타입 기준 | 승인된 객체·속성·관계·액션·워크플로우 정의 | ◆ |
| 공유 속성 | 여러 객체 유형이 함께 쓰는 속성 정의 | ◆ |
| 영속 지식 | 객체 인스턴스·속성값·관계 인스턴스 | ◆ |
| Apply 경계 | 영속 지식을 바꾸는 단일 진입점 | ◆ |
| Unified Validator | 모든 경로가 공유하는 단일 검증기 | ◆ |
| 변경 제안 | 타입 기준을 바꾸기 위한 승인 대상 | ◆ |
| 변경 이력 | 기준·영속 지식이 실제로 바뀐 기록 | ◆ |
| 일괄 등록 | 파일·API로 들어오는 대량 정의·데이터 | ◆ |
| 부작용 수준 | 액션이 외부·기준에 미치는 영향의 등급 | ◆ |
| 역할 | 실행 안에서 대상 객체가 맡는 이름 | ◆ |
| 역할 바인딩 | 역할을 실제 인스턴스에 묶는 실행 상태 | ◆ |
| 역할 한정 슬롯 참조 | `{object_role, slot_name}` | ◆ |
| 상태 슬롯 | 실행 중 값의 격리 저장 단위 | ◆ |
| 제어 선 | 실행 순서를 잇는 선 | ◆ |
| 바인딩 계층 | 절차가 기준을 참조하는 지점 | ◆ |
| dry-run | 반영 없이 실행해 품질을 확인하는 것 | ◆ |
| Provenance | 판단과 변경의 근거 | ◆ |
| 세션 버퍼 | 의도 확정 전 발화값을 담는 임시 저장 | ▲ |
| 되묻기 | 미충족 필수 슬롯을 질문하는 절차 | ▲ |
| 변경 후보 / 후보 그룹 | 반영 전 승인 대상이 되는 변경안 | ○ |
| Gate | 변경 후보가 승인 경계를 통과하는 확장 절차 | ○ |
| WriteBack | 승인된 후보를 Apply 경계로 보내는 확장 행위 | ○ |

---

## 21. 요약

### 21.1 범위

```text
1. 코어는 온톨로지 매니저와 워크플로우 매니저다. 실무 적용 수준으로 만든다.
2. 대화 입력은 앞단 부품, Gate 는 뒷단 부품, 자동화는 측면 부품이다.
3. 부품은 현재 범위와 후속 범위를 분리하되, 계약은 코어 단계에서 고정한다.
4. 코어는 부품 없이 완결된다. 영속 지식은 일괄 등록·직접 편집·TBox 변경으로 관리된다.
5. 승격 조건을 판정하는 것은 코어, 후보를 만들어 승인받는 것은 Gate 다.
```

### 21.2 온톨로지 매니저

```text
 6. 타입 기준은 객체·속성·공유속성·관계·액션·워크플로우 유형이다.
 7. 전역 기준은 예약 프로젝트에 담고, 프로젝트는 상속·확장만 하며 약화할 수 없다.
 8. 정의 수명주기(status)와 변경 절차 상태(proposal_state)는 다른 축이다.
 9. L4 를 바꾸는 경로는 넷이며 모두 Apply 경계에서 만난다.
10. change_scope = tbox 인 Apply 는 승인된 proposal_id 없이는 불가능하다.
11. TBox 직접 편집도 Apply 를 직접 호출하지 않고 변경 제안을 만든다.
12. ABox 직접 편집은 ontology_edit_allowed 를 확인한 뒤 Apply 를 호출한다.
13. 검증기는 하나이며, 호출자마다 다른 것은 실패 처리 정책뿐이다.
14. 검증 결과 형식도 하나다. dry-run·일괄 등록·영향 분석 리포트가 같은 형식을 쓴다.
15. ABox 일괄 적재는 승인 없이 적용하고, TBox 일괄 변경은 반드시 승인을 경유한다.
16. title 은 파생값이므로 title_property 변경 시 같은 트랜잭션에서 갱신한다.
17. 후보는 제안이고 이력은 사실이다. 후보는 반려될 수 있으므로 이력이 아니다.
```

### 21.3 워크플로우 매니저

```text
18. 값은 데이터 선이 아니라 역할 한정 슬롯 참조로 읽고 쓴다.
19. 역할은 정의에 선언하고 인스턴스는 실행이 묶는다. 값이 채워지면 잠긴다.
20. WorkflowType 이 요구한 역할은 정의가 반드시 선언해야 한다.
21. 액션이 생성하는 역할과 정의가 기대하는 역할은 서로를 가리켜야 한다.
22. 여러 객체를 순회할 때는 역할을 늘리지 않고 하위 실행으로 나눈다.
23. 노드는 액션 하나를 실행한다. 매핑은 {object_role, slot_name} 으로 정적 검사한다.
24. 슬롯 변경은 이력으로, 노드 진입은 단계 실행으로, 외부 호출은 실행 로그로 남긴다.
25. 승격 판정은 Fail-Closed 다. 검사 범위는 실행된 제어 경로다.
26. 우회는 슬롯 채우기 전용이며 승격 경로가 아니다. Apply 경계에는 우회가 없다.
27. dry-run 은 Apply 경계를 호출하지 않으며, side_effect_level 이 실행 여부를 제한한다.
28. 문서 근거, 온톨로지 경로 근거, 상태변경 근거는 서로 대체하지 않는다.
```

### 21.4 조립 컴포넌트

```text
29. 대화는 의도 인식 → 실행 시작 → 슬롯 채움 순서를 지킨다.
30. 의도 확정 전 발화값은 후보 버퍼에 담고, 이관 시점에 검증한다.
31. 무엇을 언제 물을지는 선언이 정하고, 어떻게 말할지는 대화 모듈이 정한다.
32. 버튼 입력과 대화 입력은 다른 채움 경로로 기록하고 신뢰도를 함께 남긴다.
33. 대화가 실패해도 사용자는 버튼으로 같은 슬롯을 채울 수 있어야 한다.
34. Gate 초기 검증 범위의 상태는 DRAFT, SUBMITTED, APPROVED, REJECTED 다.
35. COMMITTED, COMMIT_FAILED, CONFLICTED 는 계약만 고정하고 고도화 시 구현한다.
36. WriteBack 은 저장소를 직접 건드리지 않고 Apply 경계를 호출한다.
37. 자동화는 실행을 시작할 뿐 엔진을 우회하지 않는다.
```
