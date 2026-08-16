# OntoFlow 한국어 문서

한국어 | [English](../en/README.md)

이 폴더는 OntoFlow의 한국어 기준 문서입니다. 파일명은 GitHub 링크 안정성을 위해 영어 식별자를 사용하지만, 문서의 기준 언어는 한국어입니다.

## 읽는 순서

| 순서 | 문서 | 설명 |
|---:|---|---|
| 1 | [개괄 설계](01-overview.md) | OntoFlow가 해결하려는 문제와 전체 구조 |
| 2 | [상세 설계](02-architecture-spec.md) | 타입, 인스턴스, 워크플로우, 검증, 이력의 상세 계약 |
| 3 | [워크플로우 매니저](05-workflow-manager.md) | 업무 절차 실행, 입력값, 외부 호출, 실행 상태 관리 |
| 4 | [온톨로지 매니저](06-ontology-manager.md) | 업무 객체 타입, 인스턴스, 대량 등록, 변경 이력 관리 |
| 5 | [상태와 이력](07-state-history.md) | 워크플로우 임시 상태와 실제 업무 데이터 변경 추적 |
| 6 | [공개 질문](03-open-questions.md) | 아직 확정하지 않은 설계 쟁점 |
| 7 | [로드맵](04-roadmap.md) | 구현 우선순위와 후속 확장 방향 |

## 예시

| 예시 | 설명 |
|---|---|
| [병원예약 예시](../../examples/ko/hospital-reservation.md) | 사람이 버튼이나 폼으로 시작하는 업무 실행 흐름 |
| [공장 자동 정비 지시 예시](../../examples/ko/factory-maintenance-auto.md) | 센서 이벤트로 시작하는 자동화 워크플로우 |

## 언어 구조

```text
docs/ko  한국어 기준 문서
docs/en  영어 번역 또는 요약 문서
```

GitHub의 폴더 화면은 일반 홈페이지처럼 언어 선택 메뉴를 자동으로 제공하지 않습니다. 대신 각 언어 폴더의 `README.md`에서 한국어와 영어 문서로 이동할 수 있게 관리합니다.

## 용어 기준

핵심 용어는 저장소 루트의 [용어집](../../GLOSSARY.md)을 기준으로 합니다.

```text
TBox = 클래스 타입 영역
ABox = 인스턴스 영역
ObjectType = 업무 객체의 클래스 타입
ObjectInstance = 실제 업무 객체 인스턴스
ActionType = 실행 가능한 업무 작업 정의
```
