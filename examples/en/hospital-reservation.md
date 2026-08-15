# Hospital Reservation Example

[한국어 정본](../ko/hospital-reservation.md)

This is an English translation of the canonical Korean example.

This example shows how OntoFlow models a common business workflow.

## Object Types

- `Patient`
- `Doctor`
- `Department`
- `Appointment`
- `Notification`

## Relations

- `Patient HAS_APPOINTMENT Appointment`
- `Department HAS_DOCTOR Doctor`
- `Doctor HAS_SLOT Appointment`

## Actions

- `verifyPatient`
- `findAvailableSlots`
- `createAppointment`
- `sendNotification`

## Workflow

```text
Start
  -> verify patient
  -> select department or doctor
  -> find available slots
  -> confirm requested time
  -> create appointment
  -> send notification
  -> End
```

## State Slots

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

## Persistent Knowledge

The appointment is not persistent just because a workflow step calculated it.

It becomes persistent only when an Apply request creates or updates:

- `object_instances`
- `property_values`
- `relation_instances`
- `ontology_change_history`

## Conversational Input

A user may say:

```text
I want to book a cardiology appointment tomorrow morning.
```

The conversational component may propose:

```text
department.code = cardiology
appointment.requested_time = tomorrow morning
```

The Workflow Manager still owns the run, validation, state slots, and execution.
