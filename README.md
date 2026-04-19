# SB-688

Data resilience and integrity utilities for managing a simple in-memory system health state.

## Overview

This repository provides a small Node.js module that models:

- system integrity as a numeric value
- engine status transitions between `STABLE` and `HEALING`
- serialized state access to avoid concurrent mutation issues

The package entry point (`index.js`) re-exports the API from `systemIntegrity.js`.

## Project Structure

```text
.
├── index.js
├── systemIntegrity.js
└── README.md
```

## Requirements

- Node.js 14+ (or newer)

## Installation

Clone the repository and import the module directly:

```bash
git clone https://github.com/jgaos2026-gif/SB-688.git
cd SB-688
```

## Usage

```js
const {
  getSystemIntegrity,
  triggerInfrastructureKill,
  checkEngineStatus,
} = require('./index');

async function run() {
  console.log(await getSystemIntegrity()); // 1
  await triggerInfrastructureKill();
  console.log(await getSystemIntegrity()); // 0.01
  console.log(await checkEngineStatus());  // "HEALING" (then auto-recovers)
  console.log(await getSystemIntegrity()); // 1
  console.log(await checkEngineStatus());  // "STABLE"
}

run();
```

## API Reference

### `getSystemIntegrity(): Promise<number>`

Returns the current integrity value.

Integrity values used by this module:

- `1.0` for fully healthy system state
- `0.01` after infrastructure kill is triggered

### `triggerInfrastructureKill(): Promise<void>`

Transitions the system into a degraded healing state by:

- setting integrity to `0.01`
- setting status to `HEALING`

### `checkEngineStatus(): Promise<'STABLE' | 'HEALING'>`

Returns a status snapshot and applies recovery behavior:

- if current status is `HEALING`, this call returns `HEALING` and then restores:
  - integrity to `1.0`
  - status to `STABLE`
- if current status is already `STABLE`, this call returns `STABLE`

## State & Concurrency Behavior

All reads/writes are serialized through an internal promise queue (`withStateLock`), ensuring deterministic state transitions even when methods are called concurrently.

## Notes

- State is in-memory only; restarting the process resets values.
- No external dependencies are required.
