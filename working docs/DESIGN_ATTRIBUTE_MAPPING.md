Global + Per-DMO Category Mapping Architecture

Goal

You want a system where:
	1.	Trippl has a master category taxonomy (Do, Eat, Stay, Move, Event, with subcategories).
	2.	Each DMO provides its own categories (raw strings).
	3.	You map imported categories to Trippl master categories.
	4.	Most of the mapping is global, reusable across DMOs.
	5.	Specific DMOs can override global mappings when necessary.
	6.	When importing a DMO, any new/unmapped categories trigger a UI dialog to map them.
	7.	Categories must be dynamic (DB-driven, no code deploy needed).

⸻

Database Schema (Text Descriptions)

master_categories
Stores Trippl’s official taxonomy. Supports parent/child hierarchy.
Fields:
	•	id (PK)
	•	parent_id (FK to master_categories)
	•	slug (unique)
	•	name
	•	sort_order
	•	is_active

Examples:
do, eat, stay, move, event
Subcategories: hiking (parent: do), breweries (parent: eat)

⸻

imported_categories
Stores raw category strings from each DMO’s feed.
Fields:
	•	id (PK)
	•	dmo_id
	•	raw_category
Unique on: (dmo_id, raw_category)

Examples:
DMO 5: “Pubs”
DMO 1: “Walks & Hikes”

⸻

global_category_map
Default mapping for all DMOs.
Fields:
	•	imported_category (raw text, PK)
	•	master_category_id (FK)

Examples:
“Hiking” → master category “hiking”
“Pubs” → master category “eat”

⸻

dmo_category_map
Optional override for specific DMOs.
Fields:
	•	dmo_id
	•	imported_category
	•	master_category_id
Primary key: (dmo_id, imported_category)

Examples:
For DMO 6, “Pubs” → “nightlife” instead of “eat”

⸻

Mapping Resolution Logic (Runtime)

When handling a listing:
	1.	Check dmo_category_map for an override.
	2.	If none, check global_category_map.
	3.	If none, the category is unmapped and should trigger UI mapping.

This allows:
	•	Universal mappings for common categories
	•	Per-DMO corrections
	•	Fully dynamic updates without redeploying code

⸻

Import Pipeline Flow

Step 1: Scrape DMO and extract raw categories
Already implemented in your system.

Step 2: Insert categories into imported_categories
Use “ON CONFLICT DO NOTHING” to avoid duplicates.

Step 3: Detect unmapped categories
Find any imported category where neither global nor per-DMO mapping exists.

Step 4: UI Mapping Dialog
Admin sees a list of unmapped categories. For each:
	•	Select a master category from dropdown, or
	•	Create a new master category
	•	Choose “Save as global mapping” OR “Save as DMO override”

Step 5: Continue import
Once all categories are mapped, import proceeds using the resolved IDs.

⸻

Why Global + Override Is Best

Global-only
Pros: simple
Cons: highly inaccurate for DMOs with strange category naming

Per-DMO-only
Pros: highly accurate
Cons: too much manual work per tenant

Global + override (recommended)
Pros:
	•	80% of categories require no work
	•	Flexible adjustments per DMO
	•	Perfect for SaaS
	•	Category changes are dynamic

Cons:
	•	Slightly more DB structure, but worth it

⸻

Data Flow Example
	1.	Raw category “Pubs” appears from DMO 7.
	2.	Global mapping says “Pubs → eat”.
	3.	For DMO 7, override says “Pubs → nightlife”.
	4.	System assigns the override.
	5.	UI never bothers you unless a category is unmapped.

⸻

Benefits of This Architecture
	•	Extensible: add or modify master categories at any time.
	•	Safe: mapping stored in DB, no deploy required.
	•	DMO-specific quirks handled cleanly.
	•	Supports hierarchical browsing (major categories → subcategories).
	•	Easy admin workflow for unmapped categories.
	•	Works perfectly with your import pipeline.


-- ============================================
-- MASTER CATEGORY SYSTEM (GLOBAL + PER-DMO)
-- ============================================

-- 1. Master category taxonomy
-- Hierarchical (supports major + subcategories)
CREATE TABLE master_categories (
  id SERIAL PRIMARY KEY,
  parent_id INT REFERENCES master_categories(id),
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  sort_order INT DEFAULT 0,
  is_active BOOLEAN DEFAULT TRUE
);

-- Sample inserts (for docs/testing)
INSERT INTO master_categories (slug, name) VALUES
  ('do', 'Do'),
  ('eat', 'Eat'),
  ('stay', 'Stay'),
  ('move', 'Move'),
  ('event', 'Event');

-- ============================================
-- 2. Imported categories per DMO feed
-- ============================================
CREATE TABLE imported_categories (
  id SERIAL PRIMARY KEY,
  dmo_id INT NOT NULL,
  raw_category TEXT NOT NULL,
  UNIQUE (dmo_id, raw_category)
);

-- ============================================
-- 3. Global mapping (raw value → master category)
-- Applies by default to all DMOs
-- ============================================
CREATE TABLE global_category_map (
  raw_category TEXT PRIMARY KEY,
  master_category_id INT NOT NULL REFERENCES master_categories(id)
);

-- Example inserts
INSERT INTO global_category_map (raw_category, master_category_id)
VALUES
  ('Hiking', (SELECT id FROM master_categories WHERE slug='do')),
  ('Pubs',   (SELECT id FROM master_categories WHERE slug='eat'));

-- ============================================
-- 4. Per-DMO override mapping
-- DMO-specific rules override global mapping
-- ============================================
CREATE TABLE dmo_category_map (
  dmo_id INT NOT NULL,
  raw_category TEXT NOT NULL,
  master_category_id INT NOT NULL REFERENCES master_categories(id),
  PRIMARY KEY (dmo_id, raw_category)
);

-- Example override
INSERT INTO dmo_category_map (dmo_id, raw_category, master_category_id)
VALUES (
  7,
  'Pubs',
  (SELECT id FROM master_categories WHERE slug='event')
);

-- ============================================
-- 5. Category resolution query
-- Picks DMO override first, then falls back to global mapping
-- ============================================
-- Raw SQL snippet for reference:
-- Takes dmo_id + raw_category → returns master_category_id

-- Example: finding the correct master category for an imported row
SELECT
  COALESCE(
    (SELECT master_category_id FROM dmo_category_map
     WHERE dmo_id = $1 AND raw_category = $2),
    (SELECT master_category_id FROM global_category_map
     WHERE raw_category = $2)
  ) AS resolved_category_id;

-- ============================================
-- 6. Unmapped category detection
-- Used to show UI dialog for new categories
-- ============================================
SELECT ic.raw_category
FROM imported_categories ic
LEFT JOIN dmo_category_map dcm
  ON dcm.dmo_id = ic.dmo_id AND dcm.raw_category = ic.raw_category
LEFT JOIN global_category_map gcm
  ON gcm.raw_category = ic.raw_category
WHERE dcm.master_category_id IS NULL
  AND gcm.master_category_id IS NULL;

-- ============================================
-- 7. TypeScript Example (pseudo-code)
-- Resolve a category in your Node backend
-- ============================================
/**
 * Resolve the effective master category id for a given DMO + raw category
 */
async function resolveCategory(db, dmoId, rawCategory) {
  const result = await db.oneOrNone(`
    SELECT COALESCE(
      (SELECT master_category_id FROM dmo_category_map
       WHERE dmo_id = $1 AND raw_category = $2),
      (SELECT master_category_id FROM global_category_map
       WHERE raw_category = $2)
    ) AS master_category_id
  `, [dmoId, rawCategory]);

  return result?.master_category_id || null;
}

/**
 * Fetch unmapped categories to show in admin UI
 */
async function getUnmappedCategories(db, dmoId) {
  return db.any(`
    SELECT raw_category
    FROM imported_categories ic
    LEFT JOIN dmo_category_map dcm
      ON dcm.dmo_id = ic.dmo_id AND dcm.raw_category = ic.raw_category
    LEFT JOIN global_category_map gcm
      ON gcm.raw_category = ic.raw_category
    WHERE ic.dmo_id = $1
      AND dcm.master_category_id IS NULL
      AND gcm.master_category_id IS NULL
  `, [dmoId]);
}

/**
 * Save a mapping (either global or per DMO)
 */
async function saveMapping(db, { dmoId, rawCategory, masterCategoryId, scope }) {
  if (scope === 'global') {
    await db.none(`
      INSERT INTO global_category_map (raw_category, master_category_id)
      VALUES ($1, $2)
      ON CONFLICT (raw_category) DO UPDATE
        SET master_category_id = EXCLUDED.master_category_id
    `, [rawCategory, masterCategoryId]);
  } else if (scope === 'dmo') {
    await db.none(`
      INSERT INTO dmo_category_map (dmo_id, raw_category, master_category_id)
      VALUES ($1, $2, $3)
      ON CONFLICT (dmo_id, raw_category) DO UPDATE
        SET master_category_id = EXCLUDED.master_category_id
    `, [dmoId, rawCategory, masterCategoryId]);
  }
}