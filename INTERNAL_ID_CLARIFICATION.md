# Internal ID Generation Clarification

**Date**: 2024-12-19  
**Purpose**: Document the clarification of internal ID generation strategy

## Key Decision

**Internal IDs are PostgreSQL SERIAL IDs** generated automatically during the Promotion stage, not computed during Transform.

## Why This Makes Sense

1. **Database Native**: Leverages PostgreSQL's built-in auto-incrementing SERIAL feature
2. **Unique by Default**: Eliminates collision concerns since SERIAL guarantees uniqueness
3. **Simplified Logic**: No need for complex deterministic ID computation in Transform stage
4. **Standard Practice**: Most applications use auto-incrementing primary keys for internal references

## Updated Workflow

### Transform Stage
- ✅ Match existing records via `source_id` lookup (bridge tables) or `match_hash` lookup (public tables)
- ✅ Set `listing_internal_id` in staging table:
  - **Existing records**: Store the known SERIAL ID from public.listings.id
  - **New records**: Leave as NULL (will be assigned during promotion)

### Promotion Stage  
- ✅ **New records**: 
  1. INSERT to production tables → PostgreSQL assigns SERIAL ID automatically
  2. Update staging table with the new SERIAL ID
  3. Populate bridge table with `source_id` → new SERIAL ID mapping
- ✅ **Existing records**: 
  1. UPDATE production record using known SERIAL ID from staging
  2. No bridge table changes needed (already exists)

## Benefits

- **Simpler Architecture**: No complex ID generation logic needed
- **Reliable**: PostgreSQL SERIAL guarantees uniqueness and consistency
- **Maintainable**: Standard database pattern that developers understand
- **Conflict-Free**: Eliminates deterministic ID collision concerns

## Implementation Impact

This clarification simplifies our implementation significantly:

- ✅ Transform stage focuses on matching and field mapping (already implemented)
- ✅ Staging stores existing SERIAL IDs for updates, NULL for new records
- ✅ Promotion stage handles SERIAL ID assignment and bridge table population
- ✅ Bridge tables contain straightforward `source_id` → SERIAL ID mappings

This is actually a **better architectural approach** than deterministic ID generation!
