# Aztec Sentinel Module

The Sentinel module provides validator monitoring and performance tracking capabilities for the Aztec network. It tracks validator behavior, computes performance metrics, and provides RPC endpoints for retrieving validator statistics.

## Overview

The Sentinel monitors validator activity by:
- Tracking block proposals and attestations
- Computing performance metrics per validator
- Storing historical data for analysis
- Providing RPC endpoints for querying validator stats
- Detecting and reporting validator misbehavior

## Architecture

### Core Components

- **`Sentinel`** - Main class that monitors validator activity
- **`SentinelStore`** - Persistent storage for validator history and performance data
- **`SentinelConfig`** - Configuration options for the Sentinel module
- **`factory.ts`** - Factory function for creating Sentinel instances

### Key Features

1. **Real-time Monitoring**: Tracks validator activity as blocks are produced
2. **Historical Analysis**: Maintains configurable history length for performance analysis
3. **Performance Metrics**: Computes missed proposals, attestations, and overall performance
4. **RPC Interface**: Provides endpoints for querying validator statistics
5. **Slashing Detection**: Identifies validators that should be slashed for inactivity

## Configuration

The Sentinel can be configured via environment variables:

```bash
# Enable/disable the Sentinel
SENTINEL_ENABLED=true

# Number of epochs to keep in history (default: 24)
SENTINEL_HISTORY_LENGTH_IN_EPOCHS=24
```

## RPC Methods

### `node_getValidatorStats`

Retrieves performance statistics for a single validator.

**Parameters:**
- `validatorAddress` (string): The validator's Ethereum address
- `fromSlot` (optional bigint): Start slot for filtering historical data  
- `toSlot` (optional bigint): End slot for filtering historical data

**Returns:** `SingleValidatorStats | undefined`

```typescript
interface SingleValidatorStats {
  address: string;
  currentEpoch: bigint;
  totalSlots: number;
  missedProposals: number;
  missedAttestations: number;
  lastProposal?: bigint;
  lastAttestation?: bigint;
  history: ValidatorStatusHistory[];
}
```

**Example Usage:**

```bash
# Get all stats for a validator
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "node_getValidatorStats",
    "params": ["0x1234567890123456789012345678901234567890"],
    "id": 1
  }'

# Get stats for specific slot range
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0", 
    "method": "node_getValidatorStats",
    "params": ["0x1234567890123456789012345678901234567890", "100", "200"],
    "id": 1
  }'
```

## Usage

### Starting with Sentinel Enabled

```bash
# Via environment variable
SENTINEL_ENABLED=true yarn start start --node --archiver

# Via Docker
docker run -e SENTINEL_ENABLED=true aztecprotocol/aztec:latest start --node --archiver
```

### Programmatic Usage

```typescript
import { createSentinel } from './factory.js';

const sentinel = await createSentinel(
  epochCache,
  archiver, 
  p2p,
  config,
  logger
);

if (sentinel) {
  await sentinel.start();
  
  // Get stats for a validator
  const stats = await sentinel.getValidatorStats(validatorAddress);
}
```

## Development

### Running Tests

```bash
cd yarn-project/aztec-node
yarn test --testNamePattern="sentinel"
```

### Key Files

- `sentinel.ts` - Main Sentinel implementation
- `store.ts` - Persistent storage layer
- `config.ts` - Configuration definitions
- `factory.ts` - Factory for creating instances
- `sentinel.test.ts` - Test suite

## Implementation Details

### Data Storage

The Sentinel uses LMDB for persistent storage with two main maps:
- **History Map**: Stores `ValidatorStatusHistory` per validator
- **Proven Map**: Stores historical epoch performance data

### Performance Tracking

For each validator, the Sentinel tracks:
- Block proposal attempts and successes
- Attestation attempts and successes  
- Historical performance over configurable epochs
- Computed metrics like miss rates and last activity

### Slashing Integration

The Sentinel integrates with the slashing system by:
- Emitting `WANT_TO_SLASH_EVENT` for inactive validators
- Computing penalty amounts based on configuration
- Tracking offense types (currently INACTIVITY)

## Future Enhancements

Potential areas for expansion:
- Additional validator metrics (response times, peer connectivity)
- More sophisticated slashing conditions
- Performance analytics and reporting
- Integration with governance for validator management
- Archival and compression of historical data
