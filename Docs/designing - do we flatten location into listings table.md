```bash
The question is: should locations be a separate table, or should lat/lng (and maybe address) just live inside listings?

🔹 What you have now

listings → location_id → locations (one-to-one for most POIs).

dmos also link to locations.

This means locations is shared across different entity types.

🔹 Pros of your current design (locations as a separate table)

✅ If multiple entities share the same location (e.g., DMO office + a listing in the same building), you don’t duplicate addresses/coords.
✅ Lets you normalize address fields, so you can validate / geocode centrally.
✅ You can extend locations (e.g., add a geom column for PostGIS, bounding boxes, polygons).
✅ Clean separation: “where” is not tied to “what.”

🔹 Cons of your current design

⚠️ Almost every listing will have exactly one unique location → making locations a separate table adds a join to every geo query.
⚠️ If locations is never truly shared, it’s just overhead.
⚠️ Queries like “find listings within 5km of X” are more verbose (JOIN locations ON listings.location_id = locations.id).

🔹 When to keep the bridge table

Keep locations as its own table if:

You expect multi-location entities (chains, attractions with multiple entrances).

You expect to store complex geodata (polygons, multipoints, etc.) and reuse it across entity types.

You want to associate metadata with a location itself (e.g., geocoding source, precision, neighborhood).

🔹 When to flatten into listings

Flatten (move lat/lng into listings) if:

Every listing always has exactly one location.

locations aren’t actually shared in practice.

Simpler queries are a bigger priority than normalization.

ALTER TABLE listings
  ADD COLUMN lat decimal(10,8),
  ADD COLUMN lng decimal(11,8),
  ADD COLUMN address1 varchar,
  ADD COLUMN city varchar,
  ADD COLUMN state varchar,
  ADD COLUMN zip varchar,
  ADD COLUMN country char(2);


You can still keep locations for DMOs or special cases, but listings would have its own inline location data.

🔹 My recommendation for Trippl

Given your tourism POIs use case:

Most POIs (restaurants, attractions, hotels) = one unique location each.

You’ll be running lots of geospatial queries (map bounding box, nearby search).

You’ll rarely reuse the same location row across multiple listings.

👉 I’d flatten lat/lng directly into the listings table, and still keep a locations table for entities like DMOs, events, or shared addresses.

That way:

Your core queries (map search, “what’s nearby”) are simpler and faster.

You don’t lose flexibility if you want to use locations elsewhere.

⚖️ Hybrid option (best of both worlds):

Put lat/lng directly in listings for performance.

Keep location_id nullable if you want to attach a full structured location later.

This avoids joins in common queries but preserves normalization if needed.
```