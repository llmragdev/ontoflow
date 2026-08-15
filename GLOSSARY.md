# 용어집

[English](GLOSSARY.en.md)

## OntoFlow

온톨로지 정본과 워크플로우 실행을 결합한 업무 자동화 아키텍처입니다.

## Ontology Manager

타입 정의, 영속 객체, 속성값, 관계 인스턴스, 일괄 등록, 온톨로지 변경 이력을 관리하는 컴포넌트입니다.

## Workflow Manager

워크플로우 정의, 역할 선언, 노드 선언, 상태 슬롯, 단계 실행, 액션 실행 로그를 관리하는 컴포넌트입니다.

## ObjectType

업무 객체의 타입입니다. 예: `Patient`, `Doctor`, `Appointment`, `Equipment`, `WorkOrder`.

## PropertyDefinition

ObjectType에 속한 속성 정의입니다. 값 타입, 필수 여부, 허용값, 범위, 패턴 같은 검증 기준을 가집니다.

## RelationType

두 ObjectType 사이의 관계 정의입니다.

## ActionType

실행 가능한 액션의 타입 계약입니다. 필요한 객체 역할, 생성하는 객체 역할, 입력 스키마, 출력 스키마, 커넥터, 재시도 정책, dry-run 동작을 정의합니다.

## State Slot

워크플로우 실행 중 사용하는 상태값입니다. 실행, 객체 역할, 슬롯 이름으로 식별합니다.

## State Slot History

워크플로우 실행 중 상태 슬롯이 어떻게 바뀌었는지 남기는 이력입니다.

## Persistent Knowledge

영속 지식입니다. 객체 인스턴스, 속성값, 관계 인스턴스, 온톨로지 변경 이력을 포함합니다.

## Apply Boundary

영속 지식을 바꾸는 단일 쓰기 경계입니다. 일괄 등록, 직접 편집, 워크플로우 WriteBack은 모두 이 경계를 통과합니다.

## Unified Validator

슬롯 쓰기, 일괄 등록, 직접 편집, dry-run, 승격 판정, 타입 변경 영향 분석이 함께 사용하는 단일 검증기입니다.

## Import Job

파일 또는 API로 들어온 대량 정의·데이터를 검증하고 적용하기 위한 추적 단위입니다.

## TBox

타입 계층입니다. 객체 타입, 속성, 관계, 액션 타입, 워크플로우 타입을 포함합니다.

## ABox

인스턴스 계층입니다. 객체 인스턴스, 속성값, 관계 인스턴스를 포함합니다.

## Gate

워크플로우에서 만들어진 변경 후보를 승인 경계에 올리는 확장 컴포넌트입니다.

## Change Candidate

승인 대기 중인 변경 제안입니다. 반려될 수 있으므로 실제 변경 이력과 다릅니다.

## Ontology Change History

TBox 또는 ABox가 실제로 바뀐 사실을 남기는 불변 이력입니다.

## Conversational Input

사용자 발화를 워크플로우 후보와 슬롯 후보로 바꾸는 앞단 입력 컴포넌트입니다. 온톨로지 정본을 직접 바꾸거나 액션을 직접 실행하지 않습니다.

## Dry-Run

영속 반영 없이 매핑, 누락값, 검증 실패, 예상 변경을 확인하는 실행 또는 검증 모드입니다.
