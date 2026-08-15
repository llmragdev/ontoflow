# OntoFlow

한국어 | [English](README.en.md)

OntoFlow는 온톨로지 정본과 결정론적 워크플로우 실행을 결합해 설명 가능한 업무 자동화를 만들기 위한 공개 아키텍처 프로젝트입니다.

## 왜 필요한가

대화형 AI나 도구 호출 체인만으로 업무 자동화를 만들면 다음 질문에 답하기 어려워집니다.

- 어떤 업무 객체의 어떤 값이 바뀌었는가?
- 그 값은 어느 워크플로우 단계에서 만들어졌는가?
- REST API, MCP 도구, 내부 서비스 호출의 입력과 결과는 어떻게 추적되는가?
- 어떤 검증 규칙이 통과하거나 실패했는가?
- 후보 변경은 언제 영속 지식으로 반영되는가?

OntoFlow는 이 질문을 제품 구조의 핵심으로 다룹니다.

## 핵심 아이디어

- **Ontology Manager**: 객체 타입, 속성, 관계, 액션 타입, 영속 객체, 일괄 등록, 변경 이력을 관리합니다.
- **Workflow Manager**: 워크플로우 정의, 역할 바인딩, 상태 슬롯, 단계 실행, 액션 로그를 관리합니다.
- **Unified Validator**: 슬롯 쓰기, 일괄 등록, 직접 편집, dry-run, 승격 판정이 같은 검증기를 사용합니다.
- **Apply Boundary**: 영속 지식을 바꾸는 단일 쓰기 경계입니다.
- **Composable Components**: 대화식 입력, Gate, WriteBack, 자동화 트리거는 코어 바깥에 조립합니다.

## 문서

한국어 문서가 정본입니다. 영어 문서는 번역 또는 요약본입니다.

| 문서 | 한국어 | English |
|---|---|---|
| 개괄 설계 | [docs/ko/01-overview.md](docs/ko/01-overview.md) | [docs/en/01-overview.md](docs/en/01-overview.md) |
| 상세 설계 | [docs/ko/02-architecture-spec.md](docs/ko/02-architecture-spec.md) | [docs/en/02-architecture-spec.md](docs/en/02-architecture-spec.md) |
| 공개 질문 | [docs/ko/03-open-questions.md](docs/ko/03-open-questions.md) | [docs/en/03-open-questions.md](docs/en/03-open-questions.md) |
| 로드맵 | [docs/ko/04-roadmap.md](docs/ko/04-roadmap.md) | [docs/en/04-roadmap.md](docs/en/04-roadmap.md) |
| 병원예약 예시 | [examples/ko/hospital-reservation.md](examples/ko/hospital-reservation.md) | [examples/en/hospital-reservation.md](examples/en/hospital-reservation.md) |

용어는 [GLOSSARY.md](GLOSSARY.md)를 먼저 참고하세요.

## 현재 공개 범위

이 저장소는 설계 문서부터 공개합니다. 구현 코드는 아키텍처와 공개 기여 모델이 안정된 뒤 별도 범위로 추가합니다.

2026년 코어 범위:

- 상용 수준의 Workflow Manager 설계
- 상용 수준의 Ontology Manager 설계
- 버튼/폼 기반 업무 실행
- REST, MCP, 내부 서비스 액션 계약
- 일괄 등록, 검증, Apply, 변경 이력 계약

확장 범위:

- 대화식 입력 모듈
- Gate / WriteBack 프로토타입
- 후속 자동화 트리거

## 참여 방법

- GitHub Discussions에 설계 질문을 남겨주세요.
- 실제 업무 자동화 시나리오를 제안해주세요.
- 온톨로지와 워크플로우 경계를 검토해주세요.
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

## 라이선스

문서는 CC BY 4.0으로 공유하는 것을 기본으로 합니다. 예제 코드는 추가 시 Apache License 2.0을 기본으로 합니다.

현재 릴리즈는 설계 문서와 예시만 포함하며 운영 소스 코드는 포함하지 않습니다.
