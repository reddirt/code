# Regions - Neighbourhoods - Districts

## 1️⃣ The problem
- We have:
- Listings → shared location (canonical) info
- Listing_info / DMO_info → per-DMO metadata
- Neighborhoods / regions → human-friendly local area labels

**Question: should neighborhood live in:**
- listing_info (per DMO) → allows the same listing to appear in different neighborhoods depending on the DMO
- listing (canonical) → single neighborhood for all DMOs

## 5️⃣ DECISION - Implement Simple Approach Now (Nov 2025)
- If in your current MVP all DMOs mostly agree on neighborhoods, you could store neighborhood in listing to start.
- Later, when a white-label needs a different neighborhood, migrate neighborhood into listing_info with minimal schema changes.


## 2️⃣ Option 1: Neighborhood in listing_info (per DMO)

Pros:
- Full flexibility for white-label: “Cavendish Golf” can be in “Anne’s Land” for DMO A and “PEI” for DMO B.
- Matches reality if different DMOs classify regions differently.
- Minimal impact on canonical location data.

Cons:
- Slightly more complex schema and queries: you must join listing_info → neighborhoods per DMO.
- If you want to show a map with neighborhoods, some listings may “jump” depending on DMO.

Example:
```
listings
  id: 1, name: Cavendish Golf, location_id: 10

listing_info
  id: 100, listing_id: 1, dmo_id: 1, neighborhood_id: 20 ("Anne's Land")
  id: 101, listing_id: 1, dmo_id: 2, neighborhood_id: 21 ("PEI")
  ```
## 3️⃣ Option 2: Neighborhood in listing (canonical)

Pros:
- Schema is simpler: listing → neighborhood
- Easy for mapping / filtering across all DMOs
- Queries are simpler

Cons:
- You lose flexibility for white-labels: different DMOs can’t show different neighborhoods.
- If two DMOs disagree about what neighborhood a listing belongs to, you must pick one.


## 4️⃣ Recommendation

Since you specifically mention white-labels may want different data per DMO, the more flexible and correct approach is to keep neighborhood in listing_info (per DMO).
- This keeps canonical location clean: latitude/longitude, street, city, province.
- Neighborhood becomes per DMO metadata, just like category, status, or quality_score.
- If most of your listings are shared across DMOs but a few neighborhoods differ, this design handles it naturally.
- Optionally keep the neighbourhoods in a bridge table to allow a listing_info to be in more than one region.


## 💡 TL;DR (eventual implementation):
- White-label flexibility matters: put neighborhood in listing_info (per DMO).
- Canonical location stays in listing → location.
- Mapping and queries can join neighborhoods through listing_info when needed.

----

## DETAILS
```
1️⃣ Simple Version (Neighborhood in listings)

-- Canonical location
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    street TEXT,
    city TEXT,
    province TEXT,
    country TEXT,
    postal_code TEXT,
    latitude NUMERIC NOT NULL,
    longitude NUMERIC NOT NULL
);

-- Neighborhood as part of canonical listing
CREATE TABLE neighborhoods (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

-- Listings
CREATE TABLE listings (
    id SERIAL PRIMARY KEY,
    location_id INT REFERENCES locations(id),
    name TEXT NOT NULL,
    neighborhood_id INT REFERENCES neighborhoods(id),
    description TEXT
);

-- DMOs / white labels
CREATE TABLE dmos (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

-- Listing info / DMO metadata
CREATE TABLE listing_info (
    id SERIAL PRIMARY KEY,
    listing_id INT REFERENCES listings(id),
    dmo_id INT REFERENCES dmos(id),
    category TEXT,
    status TEXT,
    quality_score INT,
    manual_review_required BOOLEAN DEFAULT FALSE
);
```

```
2️⃣ Recommended Version (Neighborhood per DMO / per listing_info)
-- Canonical location
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    street TEXT,
    city TEXT,
    province TEXT,
    country TEXT,
    postal_code TEXT,
    latitude NUMERIC NOT NULL,
    longitude NUMERIC NOT NULL
);

-- Neighborhoods (normalized)
CREATE TABLE neighborhoods (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

-- Listings
CREATE TABLE listings (
    id SERIAL PRIMARY KEY,
    location_id INT REFERENCES locations(id),
    name TEXT NOT NULL,
    description TEXT
);

-- DMOs / white labels
CREATE TABLE dmos (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

-- Listing info / DMO metadata
CREATE TABLE listing_info (
    id SERIAL PRIMARY KEY,
    listing_id INT REFERENCES listings(id),
    dmo_id INT REFERENCES dmos(id),
    category TEXT,
    status TEXT,
    quality_score INT,
    manual_review_required BOOLEAN DEFAULT FALSE
);

-- Neighborhood links (per DMO)
CREATE TABLE listing_info_neighborhoods (
    listing_info_id INT REFERENCES listing_info(id) ON DELETE CASCADE,
    neighborhood_id INT REFERENCES neighborhoods(id) ON DELETE CASCADE,
    PRIMARY KEY (listing_info_id, neighborhood_id)
);
```