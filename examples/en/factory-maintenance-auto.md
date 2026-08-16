# Factory Maintenance Automation Example

[Examples](README.md) | [Hospital Reservation Example](hospital-reservation.md)

[한국어](../ko/factory-maintenance-auto.md) | English

This example shows how OntoFlow can create a maintenance work order from equipment events.

The hospital reservation example shows a human-driven button/form workflow. This example shows an event-driven automation workflow.

This example does not include a separate approval flow. When sensor events and rule conditions are met, `Workflow Manager` creates a maintenance work order automatically. Higher-risk actions, such as direct equipment control, can be combined with an approval component later.

## Scenario

A conveyor repeatedly stops within a short time window. The system receives equipment events, classifies the symptom, selects a likely cause and maintenance action, and creates a work order for the maintenance team.

```text
Sensor event received
  -> Check repeated-fault condition
  -> Look up equipment type and symptom rules
  -> Select cause candidate and action candidate
  -> Create work order
  -> Update equipment status and maintenance history
  -> Notify technician
```

## Object Types

- `Equipment`
- `SensorEvent`
- `FaultSymptom`
- `FaultCause`
- `MaintenanceAction`
- `WorkOrder`
- `Technician`
- `Notification`

## Example Object Instances

```text
Equipment           Conveyor-01
SensorEvent         EVT-2026-001
FaultSymptom        repeated_stop
FaultCause          belt_tension_low
MaintenanceAction   adjust_belt_tension
WorkOrder           WO-2026-001
```

## Relations

- `Equipment HAS_EVENT SensorEvent`
- `Equipment HAS_SYMPTOM FaultSymptom`
- `FaultSymptom INDICATES_CAUSE FaultCause`
- `FaultCause SUGGESTS_ACTION MaintenanceAction`
- `WorkOrder TARGETS Equipment`
- `WorkOrder REQUESTS MaintenanceAction`
- `WorkOrder ASSIGNED_TO Technician`

## Actions

- `receiveSensorEvent`
- `detectRepeatedFault`
- `classifyFaultSymptom`
- `selectMaintenanceAction`
- `createWorkOrder`
- `notifyTechnician`
- `updateEquipmentStatus`

## Workflow

```text
SensorEventReceived
  -> Check repeated-fault condition
  -> Classify symptom
  -> Select cause candidate
  -> Select action candidate
  -> Create work order
  -> Update equipment status
  -> Notify technician
  -> End
```

## State Slots

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

## Automation Condition

```text
event_type = stop
same_equipment_stop_count >= 3
time_window_minutes = 30
equipment.status != maintenance_in_progress
```

When the condition is met, the workflow starts automatically.

## Example Work Order

```text
Work order: WO-2026-001
Target equipment: Conveyor-01
Detected symptom: 3 stops within 30 minutes
Likely cause: Low belt tension
Recommended action: Inspect belt tension and adjust to the target range
Priority: High
Status: created
```

## State Change History

The point of this example is not automation alone. The important part is that values and state changes remain traceable.

```text
State Slot History
  fault.count_in_window: 2 -> 3
  fault.symptom_code: null -> repeated_stop
  fault.cause_code: null -> belt_tension_low
  work_order.status: null -> created

Action Execution Log
  detectRepeatedFault     success
  selectMaintenanceAction success
  createWorkOrder         success
  notifyTechnician        success

Ontology Change History
  Equipment.status: running -> maintenance_requested
  WorkOrder.status: null -> created
```

## Difference From the Hospital Reservation Example

| Area | Hospital Reservation | Factory Maintenance Automation |
|---|---|---|
| Start | User starts with a button or form | Sensor event starts the workflow |
| Main input | Patient, doctor, requested time | Equipment, event, repeated-fault condition |
| Result | Appointment created | Work order created |
| Main point | Human-driven workflow execution | Event-driven automation |
| Tracked state | Appointment and notification status | Equipment status, fault decision, maintenance history |

Both examples share the same structure.

```text
Input occurs
  -> Fill state slots
  -> Validate against ontology definitions
  -> Execute actions
  -> Apply actual business data changes
  -> Store history
```

The difference is how the workflow starts. The hospital workflow starts from a person. The factory workflow starts from an event.
