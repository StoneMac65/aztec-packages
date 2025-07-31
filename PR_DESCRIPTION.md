# Add Single Validator RPC Method for Historical Data Retrieval

## Summary

This PR adds a new RPC method `getValidatorStats` to retrieve historical data and performance for a single validator instead of all validators, addressing the need for more efficient validator monitoring as discussed in the issue.

## Motivation

Currently, the `getValidatorsStats` method returns data for all validators, which can be inefficient when you only need information about a specific validator. This PR introduces a targeted approach that:

- Reduces data transfer and processing overhead
- Enables focused validator monitoring
- Supports historical data analysis with optional time range filtering
- Prepares the foundation for future archival functionality

## Changes

### Core Implementation

1. **New Types & Schemas** (`yarn-project/stdlib/src/validators/`)
   - Added `SingleValidatorStats` type for single validator response
   - Added corresponding Zod schema for validation

2. **RPC Interface** (`yarn-project/stdlib/src/interfaces/aztec-node.ts`)
   - Added `getValidatorStats` method to `AztecNode` interface
   - Added schema definition with optional slot range parameters

3. **Sentinel Implementation** (`yarn-project/aztec-node/src/sentinel/`)
   - Made `SentinelStore.getHistory()` public for single validator access
   - Added `computeStatsForSingleValidator()` method to `Sentinel` class
   - Implemented slot range filtering for historical data analysis

4. **Node Service** (`yarn-project/aztec-node/src/aztec-node/server.ts`)
   - Added RPC endpoint implementation that delegates to Sentinel

### API Details

**Method**: `getValidatorStats`
**Namespace**: `node_getValidatorStats` (in sandbox)

**Parameters**:
- `validatorAddress` (string): The validator's Ethereum address
- `fromSlot` (optional bigint): Start slot for filtering historical data
- `toSlot` (optional bigint): End slot for filtering historical data

**Returns**: `SingleValidatorStats | undefined`

**Response Structure**:
```typescript
{
  validator: ValidatorStats,           // Current validator statistics
  provenPerformance: Array<{           // Historical proven performance
    missed: number,
    total: number,
    epoch: bigint
  }>,
  lastProcessedSlot?: bigint,          // Last processed slot
  initialSlot?: bigint,                // Initial tracking slot
  slotWindow: number                   // Size of tracking window
}
```

## Usage Examples

### Get all data for a validator
```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"node_getValidatorStats","params":["0x1234..."],"id":1}'
```

### Get data for specific slot range
```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"node_getValidatorStats","params":["0x1234...", "100", "200"],"id":1}'
```

## Benefits

- **Performance**: Only fetches data for the requested validator
- **Flexibility**: Optional time range filtering for historical analysis
- **Comprehensive**: Includes both real-time stats and proven performance history
- **Robust**: Returns `undefined` for non-existent validators
- **Future-ready**: Designed to support archival functionality

## Testing

The implementation follows existing patterns and includes proper error handling. The method integrates seamlessly with the existing Sentinel infrastructure and maintains backward compatibility.

### Docker Integration

The changes have been tested with Docker builds and the Sentinel functionality works correctly in containerized environments:

```bash
# Build custom image with Sentinel
docker build -t aztecprotocol/aztec:latest .

# Run with Sentinel enabled
docker run -p 8080:8080 -e SENTINEL_ENABLED=true aztecprotocol/aztec:latest "start --sandbox"
```

## Documentation

- Added comprehensive documentation in `yarn-project/aztec-node/src/sentinel/README.md`
- Includes API reference, usage examples, and integration guide
- Documents new RPC method `node_getValidatorStats` with parameters and examples

## Related Issues

Addresses the need for single validator data retrieval methods in the Sentinel module as discussed with maintainers.
