# Simple Listings Pipeline Implementation Plan

**Goal**: Add scraping/ingestion capability to existing listings system without breaking anything

**Approach**: Incremental, safe, following existing codebase patterns

---

## Phase 1: Minimal Viable Implementation (1 week)

### **Task 1.1: Extend Existing Import System**
**Time**: 2 days  
**Risk**: Low (extends existing working system)

#### **What to do**:
1. **Add source_type column** to existing `dmos` table:
   ```sql
   -- Add source_type column to dmos table
   ALTER TABLE dmos ADD COLUMN source_type VARCHAR(20) DEFAULT 'API';
   
   -- Add constraint for valid source types
   ALTER TABLE dmos ADD CONSTRAINT chk_dmo_source_type 
     CHECK (source_type IN ('WORDPRESS', 'WEBSITE', 'API', 'DRUPAL'));
   ```

3. **Create scrape configuration table**:
   ```sql
   -- Create table for DMO-specific scrape configurations
   CREATE TABLE dmo_scrape_configs (
     id SERIAL PRIMARY KEY,
     dmo_id INTEGER NOT NULL REFERENCES dmos(id) ON DELETE CASCADE,
     category VARCHAR(100) NOT NULL,  -- 'accommodations', 'attractions', 'restaurants', etc.
     path TEXT NOT NULL,              -- '/accommodations', '/eat-drink', '/do', etc.
     field_mappings JSONB NOT NULL,   -- CSS selectors, API field mappings, etc.
     active BOOLEAN DEFAULT TRUE,
     last_run TIMESTAMP,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
     updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   -- Add indexes for performance
   CREATE INDEX idx_dmo_scrape_configs_dmo_id ON dmo_scrape_configs (dmo_id);
   CREATE INDEX idx_dmo_scrape_configs_active ON dmo_scrape_configs (active) WHERE active = TRUE;
   CREATE UNIQUE INDEX idx_dmo_scrape_configs_unique ON dmo_scrape_configs (dmo_id, category);
   ```

2. **Update DMO interfaces** to include source_type:
   ```typescript
   // In src/dao/dmo/DmoInterfaces.ts
   export enum SourceType {
     WORDPRESS = 'WORDPRESS',
     WEBSITE = 'WEBSITE', 
     API = 'API',
     DRUPAL = 'DRUPAL'
   }
   
   export interface DmoDb {
     id: number;
     name: string;
     dmoTypeId: number;
     sourceType: SourceType; // Add this field
   }
   
   export interface DmoTransport {
     id: number;
     name: string;
     dmoTypeId: number;
     sourceType: SourceType; // Add this field
     // ... existing fields
   }
   ```

3. **Create scrape configuration interfaces**:
   ```typescript
   // In src/dao/dmo/DmoScrapeConfigInterfaces.ts
   export interface DmoScrapeConfigDb {
     id: number;
     dmoId: number;
     category: string;
     path: string;
     fieldMappings: Record<string, any>;
     active: boolean;
     lastRun?: Date;
     createdAt: Date;
     updatedAt: Date;
   }
   
   export interface DmoScrapeConfigTransport {
     id: number;
     dmoId: number;
     category: string;
     path: string;
     fieldMappings: Record<string, any>;
     active: boolean;
     lastRun?: string;
     createdAt: string;
     updatedAt: string;
   }
   
   // Field mapping examples for different source types
   export interface WordPressFieldMappings {
     name: string;           // 'title.rendered'
     description: string;    // 'content.rendered'
     address: string;        // 'acf.address'
     phone: string;          // 'acf.phone'
     website: string;        // 'acf.website'
     latitude: string;       // 'acf.latitude'
     longitude: string;      // 'acf.longitude'
   }
   
   export interface WebsiteFieldMappings {
     name: string;           // 'h1'
     description: string;    // '.field-name-body'
     address: string;        // '.field-name-field-address'
     phone: string;          // '.field-name-field-phone'
     website: string;        // '.field-name-field-website'
     hours: string;          // '.field-name-field-hours'
   }
   
   export interface ApiFieldMappings {
     name: string;           // 'name'
     description: string;    // 'description'
     address: string;        // 'address'
     phone: string;          // 'phone'
     website: string;        // 'website'
     latitude: string;       // 'lat'
     longitude: string;      // 'lng'
   }
   ```

4. **Extend existing ImportService** with pipeline methods:
   ```typescript
   // In src/scripts/services/ImportService.ts
   export const importFromDmo = async (dmoId: number) => {
     // Get DMO configuration from database
     const dmo = await DmoDao.getDmoById(dmoId);
     if (!dmo) {
       throw new Error(`DMO ${dmoId} not found`);
     }
     
     // Get active scrape configurations for this DMO
     const configs = await DmoScrapeConfigDao.getActiveConfigsByDmoId(dmoId);
     if (configs.length === 0) {
       throw new Error(`No active scrape configurations found for DMO ${dmoId}`);
     }
     
     // Route to appropriate import method based on source_type
     switch (dmo.sourceType) {
       case SourceType.WORDPRESS:
         return await importFromWordPress(dmo, configs);
       case SourceType.WEBSITE:
         return await importFromWebsite(dmo, configs);
       case SourceType.API:
         return await importFromApi(dmo, configs);
       case SourceType.DRUPAL:
         return await importFromDrupal(dmo, configs);
       default:
         throw new Error(`Unsupported source type: ${dmo.sourceType}`);
     }
   };
   
   const importFromWordPress = async (dmo: DmoTransport, configs: DmoScrapeConfigTransport[]) => {
     const results = [];
     
     for (const config of configs) {
       // Build WordPress API URL
       const baseUrl = dmo.domainUrl || 'https://example.com';
       const apiUrl = `${baseUrl}/wp-json/wp/v2${config.path}`;
       
       // Fetch data from WordPress API
       const response = await fetch(apiUrl);
       const data = await response.json();
       
       // Transform using field mappings
       const transformedData = data.map(item => transformWordPressItem(item, config.fieldMappings));
       
       // Import using existing importListings pattern
       const result = await importListings(transformedData);
       results.push({ category: config.category, count: result.count });
       
       // Update last_run timestamp
       await DmoScrapeConfigDao.updateLastRun(config.id);
     }
     
     return { totalProcessed: results.reduce((sum, r) => sum + r.count, 0), results };
   };
   
   const importFromWebsite = async (dmo: DmoTransport, configs: DmoScrapeConfigTransport[]) => {
     const results = [];
     
     for (const config of configs) {
       // Build website URL
       const baseUrl = dmo.domainUrl || 'https://example.com';
       const scrapeUrl = `${baseUrl}${config.path}`;
       
       // Scrape using field mappings (CSS selectors)
       const scrapedData = await scrapeWebsite(scrapeUrl, config.fieldMappings);
       
       // Import using existing importListings pattern
       const result = await importListings(scrapedData);
       results.push({ category: config.category, count: result.count });
       
       // Update last_run timestamp
       await DmoScrapeConfigDao.updateLastRun(config.id);
     }
     
     return { totalProcessed: results.reduce((sum, r) => sum + r.count, 0), results };
   };
   
   const importFromDrupal = async (dmo: DmoTransport, configs: DmoScrapeConfigTransport[]) => {
     // Similar to WordPress but using Drupal REST API
     // Implementation would use Drupal's JSON API endpoints
   };
   
   // Helper function to transform WordPress data using field mappings
   const transformWordPressItem = (item: any, mappings: Record<string, string>) => {
     const transformed: any = {};
     
     for (const [field, mapping] of Object.entries(mappings)) {
       // Handle nested property access (e.g., 'title.rendered', 'acf.latitude')
       const value = getNestedProperty(item, mapping);
       transformed[field] = value;
     }
     
     return transformed;
   };
   
   // Helper function to get nested properties
   const getNestedProperty = (obj: any, path: string) => {
     return path.split('.').reduce((current, key) => current?.[key], obj);
   };
   ```

5. **Example configuration data**:
   ```sql
   -- Example: Booking PEI (WordPress)
   INSERT INTO dmo_scrape_configs (dmo_id, category, path, field_mappings) VALUES
   (1, 'accommodations', '/accommodations', '{
     "name": "title.rendered",
     "description": "content.rendered", 
     "address": "acf.address",
     "phone": "acf.phone",
     "website": "acf.website",
     "latitude": "acf.latitude",
     "longitude": "acf.longitude"
   }'),
   (1, 'attractions', '/attractions', '{
     "name": "title.rendered",
     "description": "content.rendered",
     "address": "acf.address", 
     "phone": "acf.phone",
     "website": "acf.website",
     "latitude": "acf.latitude",
     "longitude": "acf.longitude"
   }');
   
   -- Example: Fredericton Capital Region (Website Scraping)
   INSERT INTO dmo_scrape_configs (dmo_id, category, path, field_mappings) VALUES
   (2, 'do', '/do', '{
     "name": "h1",
     "description": "div.field-name-body",
     "address": "div.field-name-field-address",
     "phone": "div.field-name-field-phone", 
     "website": "div.field-name-field-website",
     "hours": "div.field-name-field-hours"
   }'),
   (2, 'eat', '/eat-drink', '{
     "name": "h1",
     "description": "div.field-name-body",
     "address": "div.field-name-field-address",
     "phone": "div.field-name-field-phone",
     "website": "div.field-name-field-website"
   }'),
   (2, 'stay', '/stay', '{
     "name": "h1", 
     "description": "div.field-name-body",
     "address": "div.field-name-field-address",
     "phone": "div.field-name-field-phone",
     "website": "div.field-name-field-website"
   }');
   ```

### **Task 1.2: Add Simple API Endpoints**
**Time**: 1 day  
**Risk**: Low (follows existing route patterns)

#### **What to do**:
1. **Add listings import routes** to existing structure:
   ```typescript
   // In src/routes/v1/index.ts
   import { router as listingsImportRouter } from './listings-import';
   
   // Add to router
   router.use('/listings-import', listingsImportRouter);
   ```

2. **Create listings import routes**:
   ```typescript
   // src/routes/v1/listings-import.ts
   import express from 'express';
   import { rbacMiddleware, requirePermissions } from '../../middleware/rbac/rbacMiddleware';
   import { Permissions } from '../../constants/p-ermissions';
   import { asyncHandler } from '../../utils/asyncHandler';
   import { importFromDmo } from '../../scripts/services/ImportService';
   
   const router = express.Router();
   
   // POST /v1/listings-import/:dmoId
   router.post('/:dmoId', 
     rbacMiddleware,
     requirePermissions([Permissions.LISTINGS_CREATE]),
     asyncHandler(async (req: IAuthRequest, res: express.Response) => {
       const { dmoId } = req.params;
       
       // Basic validation
       if (!dmoId || isNaN(parseInt(dmoId, 10))) {
         return res.status(400).json({ error: 'Invalid DMO ID' });
       }
       
       const result = await importFromDmo(parseInt(dmoId, 10));
       
       return res.status(200).json({
         success: true,
         message: `Imported ${result.count} listings for DMO ${dmoId}`,
         data: result
       });
     })
   );
   
   export { router };
   ```

### **Task 1.3: Add Simple Admin Interface**
**Time**: 2 days  
**Risk**: Low (extends existing admin patterns)

#### **What to do**:
1. **Add listings import page** to existing admin structure:
   ```typescript
   // In app-web/src/app/admin/listings-import/page.tsx
   'use client';
   
   import { useState } from 'react';
   import { Button, Card, CardContent, CardHeader, CardTitle } from '@mui/material';
   import { useRbac } from '@/hooks/useRbac';
   import { Permissions } from '@/constants/permissions';
   
   export default function ListingsImportPage() {
     const { hasPermission } = useRbac();
     const [loading, setLoading] = useState(false);
     
     if (!hasPermission(Permissions.LISTINGS_CREATE)) {
       return <div>Access denied</div>;
     }
     
     const handleWordPressImport = async () => {
       setLoading(true);
       try {
         const response = await fetch('/api/v1/listings-import/5', {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' }
         });
         
         const result = await response.json();
         alert(`Success: ${result.message}`);
       } catch (error) {
         alert(`Error: ${error.message}`);
       } finally {
         setLoading(false);
       }
     };
     
     return (
       <Card>
         <CardHeader>
           <CardTitle>Listings Import</CardTitle>
         </CardHeader>
         <CardContent>
           <Button 
             onClick={handleWordPressImport}
             disabled={loading}
             variant="contained"
           >
             Import Booking PEI Listings
           </Button>
         </CardContent>
       </Card>
     );
   }
   ```

### **Task 1.4: Add Basic Sync Capability**
**Time**: 2 days  
**Risk**: Medium (new functionality)

#### **What to do**:
1. **Add sync method** to existing ImportService:
   ```typescript
   // In src/scripts/services/ImportService.ts
   export const syncListings = async (dmoId: number, config: any) => {
     // Get existing listings for this DMO
     const existingListings = await ListingDao.getListingsByDmoId(dmoId);
     
     // Fetch fresh data from source
     const freshData = await fetchFromSource(config);
     
     // Compare and update
     const updates = compareListings(existingListings, freshData);
     
     // Apply updates using existing patterns
     for (const update of updates) {
       if (update.action === 'CREATE') {
         await importListings([update.data]);
       } else if (update.action === 'UPDATE') {
         await ListingDao.updateListing(update.id, update.data);
       }
     }
     
     return { updated: updates.length };
   };
   ```

---

## Phase 2: Enhanced Features (1 week)

### **Task 2.1: Add Visibility Controls**
**Time**: 2 days

#### **What to do**:
1. **Add columns** to existing tables:
   ```sql
   -- Add to existing listings table
   ALTER TABLE listings ADD COLUMN listing_visible BOOLEAN DEFAULT TRUE;
   ALTER TABLE listings ADD COLUMN listing_status VARCHAR(50) DEFAULT 'ACTIVE';
   
   -- Add to existing listing_info table  
   ALTER TABLE listing_info ADD COLUMN info_visible BOOLEAN DEFAULT TRUE;
   ALTER TABLE listing_info ADD COLUMN info_status VARCHAR(50) DEFAULT 'ACTIVE';
   ```

2. **Update existing queries** to respect visibility:
   ```typescript
   // In src/dao/listing/ListingQueries.ts
   export const GET_LISTINGS_NEAR_POINT = `
     ${GET_LISTINGS_NEAR_POINT_BASE_QUERY}
     WHERE l.listing_visible = TRUE 
     AND li.info_visible = TRUE
     AND l.listing_status = 'ACTIVE'
     AND li.info_status = 'ACTIVE'
   `;
   ```

### **Task 2.2: Add Simple Queue System**
**Time**: 2 days

#### **What to do**:
1. **Use existing patterns** for background jobs:
   ```typescript
   // In src/services/PipelineService.ts
   export const scheduleImport = async (dmoId: number, config: any) => {
     // Use existing cron patterns from your codebase
     cron.schedule('0 */6 * * *', async () => {
       try {
         await syncListings(dmoId, config);
         LogUtils.info(`Scheduled sync completed for DMO ${dmoId}`);
       } catch (error) {
         LogUtils.error(`Scheduled sync failed for DMO ${dmoId}: ${error}`);
       }
     });
   };
   ```

### **Task 2.3: Add Admin Dashboard**
**Time**: 3 days

#### **What to do**:
1. **Extend existing admin** with pipeline status:
   ```typescript
   // In app-web/src/app/admin/listings/page.tsx
   // Add pipeline status section to existing listings admin
   ```

---

## Phase 3: Production Ready (1 week)

### **Task 3.1: Error Handling & Logging**
**Time**: 2 days

#### **What to do**:
1. **Use existing error patterns**:
   ```typescript
   // Follow existing ServiceResponse pattern
   // Use existing LogUtils for logging
   // Use existing asyncHandler for route error handling
   ```

### **Task 3.2: Testing**
**Time**: 2 days

#### **What to do**:
1. **Follow existing test patterns**:
   ```typescript
   // Use existing test structure
   // Test new functions in isolation
   // Integration tests for new endpoints
   ```

### **Task 3.3: Documentation**
**Time**: 1 day

#### **What to do**:
1. **Document new endpoints** in existing API docs
2. **Add admin user guide** for pipeline features

---

## Key Benefits of This Approach

### ✅ **Safe & Incremental**
- Builds on existing working systems
- Low risk of breaking anything
- Can be implemented piece by piece

### ✅ **Follows Existing Patterns**
- Uses existing service structure
- Uses existing error handling
- Uses existing RBAC patterns
- Uses existing database patterns

### ✅ **Realistic Timeline**
- 3 weeks total vs 3+ months for complex plan
- Can deliver value quickly
- Can iterate based on feedback

### ✅ **Maintainable**
- Simple, focused code
- Easy to understand and modify
- Follows your team's existing patterns

---

## Migration Path from Complex Plan

If you later want the complex features from the original plan:

1. **Start with this simple plan** (3 weeks)
2. **Get it working in production** 
3. **Add complex features incrementally**:
   - Add S3 storage for raw data
   - Add advanced deduplication
   - Add complex admin workflows
   - Add image processing pipeline

This way you get **working functionality quickly** and can **evolve it safely** over time.

---

## Next Steps

1. **Review this simplified plan**
2. **Start with Phase 1, Task 1.1** (extend existing import system)
3. **Test each piece thoroughly** before moving to next task
4. **Get feedback from team** on each phase

This approach will give you a **working pipeline in 3 weeks** instead of a **broken complex system in 3 months**.
