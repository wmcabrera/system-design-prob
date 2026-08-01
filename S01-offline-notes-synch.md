# System Design Practice: Offline Notes Synchronization

## Problem

Design an **offline-first notes application** that synchronizes notes between two devices: a phone and a tablet.

Both devices store notes locally and must continue working without an internet connection. When connectivity becomes available, the devices synchronize their changes and eventually reach the same state.

Each note contains:

```json
{
  "noteId": "N-101",
  "title": "Shopping list",
  "content": "Milk and bread",
  "updatedAt": "2026-08-01T10:00:00Z",
  "deleted": false
}
```

## Functional Requirements

The system must support:

* Creating a note
* Reading notes locally
* Editing a note
* Deleting a note
* Working while disconnected
* Synchronizing changes between the phone and tablet
* Handling duplicate synchronization messages
* Handling messages delivered out of order
* Resolving concurrent updates deterministically
* Ensuring both devices eventually contain the same data

## Conflict Scenario 1: Different Fields

Both devices initially contain:

```text
Title: Shopping list
Content: Milk
```

While disconnected, the phone changes the content:

```text
Content: Milk and bread
```

At the same time, the tablet changes the title:

```text
Title: Weekend shopping
```

After synchronization, the system should preserve both updates:

```text
Title: Weekend shopping
Content: Milk and bread
```

Explain how your design merges changes made to different fields.

## Conflict Scenario 2: Same Field

While disconnected, both devices modify the `content` field.

The phone writes:

```text
Milk and bread
```

The tablet writes:

```text
Milk and eggs
```

For this exercise, use a **last-write-wins register for each field**.

Explain:

* How the system determines which update wins
* How it avoids depending only on physical device clocks
* How it guarantees that both devices select the same winner

## Questions to Address

### 1. High-Level Architecture

Describe the main components running on each device.

A possible structure is:

```text
Notes Application
       |
Local Notes Database
       |
Change Log
       |
Synchronization Service
```

Explain the responsibility of each component.

### 2. Local Write Path

Describe what happens when a user edits a note while offline.

Your answer should address:

* Updating the local database
* Assigning a version to the change
* Recording the change durably
* Returning success to the application
* Avoiding any dependency on the other device

### 3. Versioning

Design a version identifier for each field update.

For example:

```text
(logical counter, device ID)
```

Explain:

* How the logical counter is maintained
* Why the device ID is needed
* How two versions are compared
* What happens when the counters are equal

### 4. Synchronization Protocol

Describe what happens when the phone and tablet connect.

Your protocol should cover:

1. Authenticating the other device
2. Determining which changes are missing
3. Sending missing changes
4. Applying received changes
5. Comparing field versions
6. Acknowledging received updates
7. Retrying after an interrupted connection

### 5. Duplicate Messages

Each change should have a unique identifier, such as:

```text
changeId = deviceId + local sequence number
```

Example:

```text
phone:42
tablet:19
```

Explain how the receiving device detects a duplicate change and why applying the same change multiple times must be safe.

### 6. Out-of-Order Messages

Suppose a device receives version 10 before version 8.

Explain why version 8 must not overwrite version 10 and how the stored version metadata prevents this.

### 7. Deletion

Treat deletion as a versioned update rather than immediately removing the note.

For example:

```json
{
  "deleted": {
    "value": true,
    "logicalCounter": 20,
    "deviceId": "phone"
  }
}
```

Explain:

* Why the system needs a tombstone
* How the tombstone prevents an old copy from recreating a deleted note
* How deletion competes with a concurrent edit
* When the tombstone may be permanently removed

For this simplified system, assume that a tombstone may be removed after both known devices acknowledge the deletion.

### 8. Failure Cases

Explain how your design handles these situations:

* The connection fails halfway through synchronization.
* The same change is delivered multiple times.
* Changes arrive in a different order on each device.
* One device has an incorrect physical clock.
* A device restarts before sending its local changes.
* A device reconnects with an old copy of a deleted note.

## Constraints

Assume:

* There are exactly two devices.
* Both devices store all of the user’s notes.
* Each device has a permanent unique identifier.
* Local writes should succeed immediately.
* Network connections may be unreliable.
* Messages may be duplicated or reordered.
* Automatic text merging is not required.
* Conflict resolution is last-write-wins per field.
* A logical clock should be used instead of relying exclusively on wall-clock timestamps.

## Expected Design

A reasonable architecture may look like this:

```text
┌────────────── Phone ──────────────┐
│ Notes application                 │
│ Local database                    │
│ Per-field version metadata        │
│ Durable change log                │
│ Logical clock                     │
│ Synchronization service           │
└────────────────┬──────────────────┘
                 │
           Wi-Fi / Internet
                 │
┌────────────────┴──────────────────┐
│ Tablet                            │
│ Notes application                 │
│ Local database                    │
│ Per-field version metadata        │
│ Durable change log                │
│ Logical clock                     │
│ Synchronization service           │
└───────────────────────────────────┘
```

## Core Principle

Your design should follow this principle:

> Update locally, exchange changes later, and use deterministic version metadata so both replicas eventually reach the same state.
