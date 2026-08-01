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

### 2. Local Write Path

### 3. Versioning

### 4. Synchronization Protocol

### 5. Duplicate Messages

### 6. Out-of-Order Messages

Suppose a device receives version 10 before version 8.

Explain why version 8 must not overwrite version 10 and how the stored version metadata prevents this.

### 7. Deletion

### 8. Failure Cases



## Core Principle (Hint)

Your design should follow this principle:

> Update locally, exchange changes later, and use deterministic version metadata so both replicas eventually reach the same state.
