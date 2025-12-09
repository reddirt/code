# Pipeline API Mappings System

## Overview

The Pipeline Mappings system uses a **single source of truth** on the backend, with the frontend fetching mappings dynamically from the API. This ensures consistency across the application and makes it easy to update mappings without redeploying the frontend.

The system supports a two-stage mapping process:
- **Source → Import Fields**: Map WordPress fields (or other sources) to standard import column names
- **Import Fields → Stage Fields**: Map normalized import columns to pipeline stage record fields

This design allows any import source (CSV, WordPress, API, Drupal, etc.) to normalize to the same intermediate import format, ensuring consistent processing downstream.

## Architecture

### Backend (`api-server`)

**Location:** `/src/constants/pipeline.ts`

Contains all canonical pipeline field mappings and constants:

#### 1. Import Field Mappings

- `IMPORT_FIELD_NAMES`: Array of standard import column names
  ```
  ['rec_num', 'dmo_id', 'source_id', 'name', 'address', 'city', ...]
  ```
- `IMPORT_FIELD_DISPLAY`: Human-readable labels for import columns
  ```
  { 'name': 'Name', 'address': 'Address (Line 1)', ... }
  ```

#### 2. Mapping Defaults

- `DEFAULT_SOURCE_TO_IMPORT_MAPPING`: Identity mapping for source → import fields
  ```
  { 'name': 'name', 'address': 'address', ... }
  ```
- `DEFAULT_IMPORT_TO_STAGE_MAPPING`: Maps import fields to stage record fields
  ```
  { 'name': 'l_name', 'address': 'loc_address1', ... }
  ```
- `CSV_TO_STAGE_FIELD_MAPPING`: Legacy alias maintained for backward compatibility

#### 3. Stage Fields

- `STAGE_FIELD_LIST`: All stage record target fields (string values)
- `STAGE_FIELD_DISPLAY`: Human-readable labels for stage fields

#### 4. Category Mappings

- `DEFAULT_CATEGORY_MAPPING`: Maps import categories to attributes
  ```
  { 'stay': [{name: 'lodging'}, {name: 'establishment'}], ... }
  ```

**API Endpoint:**
```
GET /v1/pipeline/mappings
```

Returns:
```json
{
  "statusCode": 200,
  "message": "OK",
  "body": {
    "importFieldNames": ["rec_num", "source_id", "name", ...],
    "importFieldDisplay": {"name": "Name", ...},
    "defaultSourceToImportMapping": {"name": "name", ...},
    "defaultImportToStageMapping": {"name": "l_name", ...},
    "stageFieldNames": ["rec_num", "source_id", "l_name", ...],
    "stageFieldDisplay": {"l_name": "Listing Name", ...},
    "defaultCategoryMapping": {"stay": [...], ...}
  }
}
```

### Frontend (`app-web`)

#### 1. **Custom Hook: `usePipelineMappings`**

**Location:** `/src/hooks/usePipelineMappings.ts`

Provides a clean interface for fetching and caching pipeline mappings:

```typescript
const { mappings, loading, error } = usePipelineMappings();

// Access different mapping types
mappings?.importFieldNames;            // Array of import column names
mappings?.importFieldDisplay;          // Record<importField, displayName>
mappings?.defaultSourceToImportMapping; // Record<sourceField, importField>
mappings?.defaultImportToStageMapping;  // Record<importField, stageField>
mappings?.stageFieldNames;             // Array of stage field names
mappings?.stageFieldDisplay;           // Record<stageField, displayName>
mappings?.defaultCategoryMapping; // Record<category, attributes>
```

#### 2. **Endpoint: `PipelineMappingsEndpoint`**

**Location:** `/src/api/pipeline/endpoints/PipelineMappingsEndpoint.ts`

Type-safe API communication interface.

#### 3. **API Function: `getPipelineMappings`**

**Location:** `/src/api/pipeline/PipelineApi.ts`

Simple API function to fetch mappings.

## Data Flow Examples

### CSV Import
```
CSV File
  ↓ (raw columns: name, address, city)
Stage Processing (uses DEFAULT_FIELD_MAPPINGS)
  ↓ name → l_name, address → loc_address1, city → loc_city
Stage Records
  ↓
Production Database
```

### WordPress Import
```
WordPress API
  ↓ (data: title.rendered, meta.city, content.rendered)
User Mapping (via config.transform.fieldMappings)
  ↓ title.rendered → name, meta.city → city, content.rendered → description
CSV Column Names (standard intermediate format)
  ↓
Stage Processing (uses DEFAULT_FIELD_MAPPINGS)
  ↓ name → l_name, city → loc_city, description → li_description
Stage Records
  ↓
Production Database
```

## Usage Examples

### WordPress Field Mapper (in PipelineConfigForm)

```typescript
import { usePipelineMappings } from '@/hooks/usePipelineMappings';

const PipelineConfigForm = ({ ... }) => {
  const { mappings: pipelineMappings } = usePipelineMappings();

  return (
    <FieldMapper
      sourceFields={wordpressFields}        // WordPress API field names
      targetFields={pipelineMappings?.importFieldNames || []}  // Standard import columns
      mappings={fieldMappings}               // User-created mappings
      onMappingsChange={setFieldMappings}
      sourceLabel="WordPress Fields"
      targetLabel="Import Fields"
      targetFieldDisplayNames={pipelineMappings?.importFieldDisplay || {}}
    />
  );
};
```

### Stage Processing (in PipelineStageService)

```typescript
import { DEFAULT_IMPORT_TO_STAGE_MAPPING } from '../../constants/pipeline';

// Map CSV columns to stage fields
for (const [importField, stageField] of Object.entries(DEFAULT_IMPORT_TO_STAGE_MAPPING)) {
  if (csvRecord[importField] !== undefined) {
    const transformedValue = transformFieldValue(stageField, csvRecord[importField]);
    stageRecord[stageField] = transformedValue;
  }
}
```

## Configuration Overrides

Both field mappings and category mappings can be overridden in the import configuration:

### Field Mappings Override

In the Import Configuration dialog, under "Transform (JSON)":

```json
{
  "sourceToImportMapping": {
    "meta.city": "city",
    "meta.street": "address",
    "featured_media": "images",
    "title.rendered": "name",
    "content.rendered": "description"
  },
  "importToStageMapping": {
    "name": "l_name",
    "address": "loc_address1",
    "city": "loc_city",
    "description": "li_description"
  }
}
```

> **Note:** The snake_case names (`source_to_import_mapping`, `import_to_stage_mapping`) are still supported for backward compatibility but camelCase is preferred.

### Category Mappings Override

In the Import Configuration dialog, under "Transform (JSON)":

```json
{
  "categoryMapping": {
    "do": ["point_of_interest"],
    "eat": ["restaurant", "food"],
    "stay": ["lodging", "establishment"],
    "airport": ["airport"],
    "camping": ["campground"]
  }
}
```

> **Note:** The snake_case name (`category_mapping`) is still supported for backward compatibility but camelCase is preferred.

### Field Transforms Override

Use `fieldTransforms` to normalize import values before they are stored in stage records. Each key is a stage field and the value is an array (or single object) of transform definitions applied in order.

```json
{
  "fieldTransforms": {
    "i_images": [
      { "wrapInArray": true },
      { "csvToArray": { "delimiter": "|", "trim": true } }
    ],
    "ci_socials": [
      {
        "stringsToArray": {
          "fields": ["facebook", "instagram", "youtube"],
          "includeStageValue": false
        }
      }
    ],
    "li_description": [
      { "stripHtml": { "preserveLineBreaks": true } }
    ],
    "ci_website_url": [
      {
        "regexReplace": {
          "pattern": "\\s",
          "replacement": ""
        }
      }
    ]
  }
}
```

Supported operations (order matters):

- `wrapInArray`: Wrap a single value in an array (optionally keep empty strings).
- `regexReplace`: Apply JavaScript-style regex replacements (`pattern`, `replacement`, optional `flags`, `applyToEach`).
- `stripHtml`: Remove HTML tags and optionally preserve line breaks.
- `csvToArray`: Split comma/pipe separated strings into arrays (configurable delimiter, trimming, dedupe).
- `stringsToArray` / `combineImportFields`: Gather multiple import fields into a single array of strings.

> **Note:** snake_case (`field_transforms`) is also supported for backward compatibility.

## Single Source of Truth

All mappings are defined in **one place**: `/src/constants/pipeline.ts` in the backend.

### To Update Mappings

1. **Add a new import column**: 
   - Add to `IMPORT_FIELD_NAMES`
   - Add to `IMPORT_FIELD_DISPLAY`
   - Add mapping in `DEFAULT_SOURCE_TO_IMPORT_MAPPING`
   - Update mapping in `DEFAULT_IMPORT_TO_STAGE_MAPPING`

2. **Change a display name**: 
   - Update `IMPORT_FIELD_DISPLAY` or `STAGE_FIELD_DISPLAY`

3. **Update default categories**: 
   - Update `DEFAULT_CATEGORY_MAPPING`

Changes automatically propagate to all frontend components through the API.

## Performance Considerations

### Caching Strategy

1. **First Request**: Fetches from API, caches in memory
2. **Subsequent Requests**: Returns cached value immediately
3. **In-Flight Requests**: Multiple components requesting simultaneously get one API call
4. **Error Handling**: Failed requests clear cache to allow retry

### API Design

The `/v1/pipeline/mappings` endpoint requires **no authentication**, allowing it to be cached by browsers and CDNs for maximum performance.

## Migration Checklist

- [x] Restructure CSV_TO_STAGE_FIELD_MAPPING as DEFAULT_IMPORT_TO_STAGE_MAPPING
- [x] Create IMPORT_FIELD_NAMES and IMPORT_FIELD_DISPLAY
- [x] Update backend endpoint to export CSV column mappings
- [x] Create PipelineMappingsEndpoint with extended interface
- [x] Update getPipelineMappings() API function
- [x] Update usePipelineMappings() hook
- [x] Add targetFieldDisplayNames prop to FieldMapper
- [x] Update PipelineConfigForm to use import columns for WordPress mapping
- [x] Document configuration override examples

## Future Enhancements

- Cache mappings in browser localStorage for offline support
- Add version tracking to detect when mappings need refresh
- Create admin UI to manage field mappings without code changes
- Support custom field mapping templates for different import sources
- Add mapping validation and conflict detection
