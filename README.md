# OntoFlow

한국어 | [English](README.en.md)

OntoFlow는 업무 데이터 모델과 업무 절차 실행을 연결해, 사람이 이해하고 추적할 수 있는 업무 자동화를 만들기 위한 공개 아키텍처 프로젝트입니다.

## 왜 필요한가

대화형 AI나 여러 도구를 이어 붙이는 방식만으로 업무 자동화를 만들면 다음 질문에 답하기 어려워집니다.

- 어떤 업무 데이터의 어떤 값이 바뀌었는가?
- 그 값은 어느 업무 단계에서 만들어졌는가?
- 외부 API, MCP 도구, 내부 서비스 호출의 입력과 결과는 어떻게 추적되는가?
- 어떤 검증 규칙이 통과하거나 실패했는가?
- 후보 변경은 언제 실제 업무 데이터에 반영되는가?

OntoFlow는 이 질문을 제품 구조의 핵심으로 다룹니다.

## 핵심 아이디어

- **업무 데이터 모델 관리기(Ontology Manager)**: 업무 객체, 속성, 관계, 대량 등록, 변경 이력을 관리합니다.
- **업무 절차 실행기(Workflow Manager)**: 업무 단계, 입력값, 상태 변화, 실행 로그를 관리합니다.
- **공통 검증기(Unified Validator)**: 입력, 대량 등록, 직접 수정, 사전 점검이 같은 기준으로 검증되게 합니다.
- **반영 경계(Apply Boundary)**: 실제 업무 데이터를 바꾸는 통로를 하나로 모읍니다.
- **조립형 확장(Composable Components)**: 대화식 입력, 승인 화면, 외부 반영, 자동 실행은 핵심 기능 바깥에 붙일 수 있게 둡니다.

## 문서

한국어 문서가 기준 문서입니다. 영어 문서는 번역 또는 요약본입니다.

문서 홈: [한국어](docs/ko/README.md) | [English](docs/en/README.md)

| 문서 | 한국어 | English |
|---|---|---|
| 개괄 설계 | [docs/ko/01-overview.md](docs/ko/01-overview.md) | [docs/en/01-overview.md](docs/en/01-overview.md) |
| 상세 설계 | [docs/ko/02-architecture-spec.md](docs/ko/02-architecture-spec.md) | [docs/en/02-architecture-spec.md](docs/en/02-architecture-spec.md) |
| 워크플로우 매니저 | [docs/ko/05-workflow-manager.md](docs/ko/05-workflow-manager.md) | - |
| 온톨로지 매니저 | [docs/ko/06-ontology-manager.md](docs/ko/06-ontology-manager.md) | - |
| 상태와 이력 | [docs/ko/07-state-history.md](docs/ko/07-state-history.md) | - |
| 공개 질문 | [docs/ko/03-open-questions.md](docs/ko/03-open-questions.md) | [docs/en/03-open-questions.md](docs/en/03-open-questions.md) |
| 로드맵 | [docs/ko/04-roadmap.md](docs/ko/04-roadmap.md) | [docs/en/04-roadmap.md](docs/en/04-roadmap.md) |
| 병원예약 예시 | [examples/ko/hospital-reservation.md](examples/ko/hospital-reservation.md) | [examples/en/hospital-reservation.md](examples/en/hospital-reservation.md) |
| 공장 자동 정비 지시 예시 | [examples/ko/factory-maintenance-auto.md](examples/ko/factory-maintenance-auto.md) | [examples/en/factory-maintenance-auto.md](examples/en/factory-maintenance-auto.md) |
| 운영 원칙 | [GOVERNANCE.md](GOVERNANCE.md) | [GOVERNANCE.en.md](GOVERNANCE.en.md) |

용어는 [GLOSSARY.md](GLOSSARY.md)를 먼저 참고하세요.

## 현재 공개 범위

이 저장소는 설계 문서부터 공개합니다. 구현 코드는 아키텍처와 공개 기여 모델이 안정된 뒤 별도 범위로 추가합니다.

2026년 코어 범위:

- 실무 적용 수준의 업무 절차 실행기 설계
- 실무 적용 수준의 업무 데이터 모델 관리기 설계
- 버튼/폼 기반 업무 실행
- 외부 API, MCP, 내부 서비스 실행 계약
- 대량 등록, 검증, 반영, 변경 이력 계약

확장 범위:

- 대화식 입력 모듈
- 승인 / 외부 반영 (초기 검증 범위)
- 자동화 트리거 (후속 범위)

## 참여 방법

- GitHub Discussions에 설계 질문을 남겨주세요.
- 실제 업무 자동화 시나리오를 제안해주세요.
- 업무 데이터 모델과 업무 절차의 경계를 검토해주세요.
- 더 명확한 용어를 제안해주세요.
- 문서나 예시를 개선해주세요.

참여 설문:

```text
설문 링크가 준비되면 여기에 추가합니다.
```

## 저장소

| 저장소 | 공개 여부 | 역할 |
|---|---|---|
| `ontoflow` | Public | 공개 문서, 예시, Discussions, 릴리즈 |
| `ontoflow-lab` | Private | 초안, 심층 리뷰, 릴리즈 큐레이션 |

Private lab 초대는 공개 토론 참여 또는 설문 응답 이후 검토합니다.

## 운영 원칙

OntoFlow는 공개 기여를 환영합니다. 기여자는 기여 내용에 따라 공개적으로 인정받을 수 있습니다.

공식 저장소, 문서 방향, 릴리즈 기준은 프로젝트 리드가 관리합니다. 기여자 표기는 프로젝트 대표 권한이나 저장소 관리 권한을 의미하지 않습니다.

자세한 내용은 [GOVERNANCE.md](GOVERNANCE.md)를 참고하세요.

## 라이선스

현재 라이선스 정책은 검토 중입니다.

문서, 예시, 핵심 소스는 서로 다른 조건으로 분리될 수 있습니다. 정식 라이선스가 확정되기 전에는 설계 문서와 공개 가능한 예시 중심으로 운영합니다.

자세한 현재 기준은 [LICENSE-POLICY.md](LICENSE-POLICY.md)를 참고하세요.
