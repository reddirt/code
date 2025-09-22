# Project Decisions

This file tracks architectural, API, and workflow decisions for the trippl project.

---

## [2025-09-22] Choice of API versioning strategy
- **Decision:** Use URI-based versioning (`/v1/...`) for the API.
- **Context / Problem:** We need a way to introduce breaking changes without affecting existing clients.
- **Considered Alternatives:**  
  1. Header-based versioning  
  2. Query param versioning (`?v=1`)  
- **Rationale:** URI-based versioning is easiest to document and most compatible with tools like Postman.
- **Implications / Next Steps:** Update API docs and add middleware to route `/v1/...` requests correctly.
- **Decision Maker(s):** Me, Alice, Bob

---

## [2025-09-20] Database schema for listings
- **Decision:** Split listings into `listings` and `listings_info` tables.
- **Context:** We need to store scraped data separately from imported customer data.
- **Alternatives:** Single table with nullable columns for all data.
- **Rationale:** Improves
