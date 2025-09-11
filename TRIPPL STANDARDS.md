# TRIPPL DEVELOPMENT STANDARDS

**Created:** 2025-01-15 
**Updated:** 2025-09-11
**Location:** Root workspace standards for all repositories

---

## 📝 **Naming Conventions**

### **Tickets (Tasks in Notion)**
```
[Area] - [Short Description]

Trips - Add fisheye effect to trip stop scrolling
```

### **Branches**
```
[type]/[ticketID]-[Area]-[Repo-Specific-Short-Description]

feat/TT-100-Trips-add-db-schema-for-new-field
```

### **PRs**
```
[type] [ticketID] [Area] - [Full Descriptive Summary]

feat TT-100 Trips - Improved UI with Fisheye Scrolling
```

### **Release Notes**
I am planning to generate release notes based on PRs, so PR titles are important

### **Definitions**
- **[Type]** - Dropdown list [Feature, Chore, Bug ...]
- **[TicketID]** - Notion task id (TT-###)
- **[Area]** - generally feature level, can be code specific if necessary (ie component)
- **[Description]** - keep it short
- **Commit messages** - imperative tense ('Add button', 'Update README')
- **PR Titles** - Description / Past tense - what the PR does or did: 'Add trip sharing feature', 'Fix bug causing API crash', 'Updated README with setup guide'.

---

## 📁 **Repository Names**
- db-schema
- api-server
- app-web
- app-android
- app-ios

Options for additional mobile business

---

## 🤖 **AI Behavior Rules**

- Always understand which repository you're working in
- Coordinate changes across multiple repositories when needed
- Use extremely brief commit messages
- Ask before generating scripts or documentation
- Do not run build commands unless specifically requested
- Do not start development servers unless explicitly requested
- Assume servers are already running with hot reload


## Code Style & Functional Programming

This codebase follows **strict functional programming patterns**. All code must adhere to these rules:

## 🚫 **CRITICAL API RULES**

### **RULE 1: No Direct Fetch Calls in Components**
**All API calls must use the existing API layer (`src/api/`) - no direct fetch calls allowed in components.**

- ✅ **CORRECT**: `import { getVersion } from '@/api/version/VersionApi';`
- ❌ **WRONG**: `fetch(\`\${config.apiUrl}/version\`)`

### **RULE 2: Environment Variables Only for Build-Time**
**Environment variables should only be used for build-time configuration, not runtime API endpoints.**

- ✅ **CORRECT**: Use centralized config system
- ❌ **WRONG**: `process.env.NEXT_PUBLIC_API_URL`

### **RULE 3: No Hardcoded URLs**
**All URLs must be defined in the centralized config system, never hardcoded.**

- ✅ **CORRECT**: `config.apiUrl` from environment-specific config
- ❌ **WRONG**: `'https://api.dev.trippl.ca/v1'`

### **RULE 4: Always Use response.isValid()**
**All API responses must be checked using `response.isValid()` before accessing data.**

- ✅ **CORRECT**: `if (response.isValid()) { const data = response.body?.result || []; }`
- ❌ **WRONG**: `const data = response.body?.result;` (direct access)

**See `src/rules/API_STANDARDS.md` for complete API rules and enforcement.**

### ✅ **Required Patterns:**

1. **Export individual functions as arrow functions:**

   ```typescript
   export const functionName = async (param: string): Promise<ReturnType> => {
     // implementation
   };
   ```

2. **Use object literals for grouped functions:**

   ```typescript
   export const GroupName = {
     function1: (param: string): void => {
       /* ... */
     },
     function2: (param: string): string => {
       /* ... */
     },
   };
   ```

3. **Use `const` for all variable declarations:**

   ```typescript
   const result = await someAsyncFunction();
   const { property1, property2 } = object;
   ```

4. **Use arrow functions for callbacks:**

   ```typescript
   array.map((item) => item.property);
   array.filter((item) => item.condition);
   ```

5. **Use `as const` for immutable constants:**
   ```typescript
   const VALIDATION_RULES = {
     minLength: 10,
     maxLength: 50,
   } as const;
   ```

### ❌ **Forbidden Patterns:**

- **No classes** - Use object literals instead
- **No `this` keyword** - Use function parameters
- **No `function` declarations** - Use arrow functions
- **No `let` declarations** - Use `const` unless reassignment is absolutely necessary
- **No object-oriented patterns** - Use functional composition

### **Examples:**

- ✅ **Good:** `src/services/RbacService.ts`
- ✅ **Good:** `src/dao/rbac/RbacDao.ts`

### **Enforcement:**

- ESLint rules enforce these patterns
- Cursor AI rules guide code generation

# Coding Conventions

Functional Programming Style

```
// Functional style
export const calculateTripDuration = (start: Date, end: Date): number =>
  Math.floor((end.getTime() - start.getTime()) / (1000 * 60 * 60 * 24));

```

Modular / Procedural Style

```
// A standalone DAO file
export function getUserById(id: string) { ... }
export function saveUser(data: User) { ... }

```

Flat / Namespace-style Exports

```
// OwnershipService.ts
export function getOwnerByListingId(...) { ... }
export function updateOwnerData(...) { ... }

// elsewhere
import * as OwnershipService from './OwnershipService';
```

If there are a few files, group in a folder. Provide an index.ts for exporting the namespace.

Avoid standalone functions and use a more object-oriented or module-based style.

```

```

Functional Programming Patterns in Our Codebase:
keep these in sync with the .cursor/rules.json file
✅ What I Should Do:
Export individual functions as arrow functions:
Use object literals for grouped functions:
Use const for all variable declarations:
Use arrow functions for callbacks:
Use functional composition patterns:
Use as const for immutable constants:
❌ What I Should NOT Do:
Don't use classes:
Don't use this keyword:
Don't use function declarations:
Don't use let unless absolutely necessary:
Don't use object-oriented patterns:
⭐️ Preferred patterns for our codebase:
Pure functions with clear inputs and outputs
Immutable data with const declarations
Arrow functions for all function definitions
Object literals for grouping related functions
Functional composition over inheritance
No classes or this keyword



# NAMING AND FOLDERS

# Component Naming and Folder Organization

maureen: 2025-08-24

✅ We are roughly following Atomic Design, but strict atomic design makes it hard to organize our domain focused UI. So we are using a hybrid approach:

- component names follow Context + Role + Type
- folders imply nesting so names don’t get absurd
- Keeps trip-specific code together (TripPanel, TripItemCard)
- Easier to navigate than separating atoms/molecules by type
- Easier to scale the app with new trip item types

✅ Naming Scheme Cheat Sheet

Panels/Columns/Bars: ExplorePanel, TripPanel, Topbar, TripMiniDashboardBar

Containers: TripItemsPanel, RightPanel

Cards: TripHeaderCard, TripItemCard, TimeDistanceCard

Variants by type: TripItem.StartCard, TripItem.EndCard, TripItem.AccommodationCard, TripItem.MoveCard, TripItem.EventCard

Parts (sub-elements): Accommodation.CheckInCard, Accommodation.CheckOutCard

Markers: TimelineDayMarker

Views: ExploreListView, ExploreDetailView

```
app-web/
  src/
    pages/
      MapPage/
        MapPage.tsx

    features/
      trip/
        types/
          trip.types.ts

        TripPanel/
          TripPanel.tsx
          TripHeaderCard/
            TripHeaderCard.tsx
            TripHeaderActions.tsx
            TripMiniDashboardBar.tsx
          TripItemsPanel/
            TripItemsPanel.tsx
            TripItemCard/
              TripItemCard.tsx    // wrapper that switches on type
              cards/
                TripItem.StartCard.tsx
                TripItem.EndCard.tsx
                TripItem.AccommodationCard.tsx
                TripItem.MoveCard.tsx
                TripItem.EventCard.tsx
              parts/
                Accommodation.CheckInCard.tsx
                Accommodation.CheckOutCard.tsx
              Timeline/
                TimelineDayMarker.tsx   // "Day 1", "Day 2" markers
              BetweenItems/
                TimeDistanceCard.tsx    // between trip items

        ExplorePanel/
          ExplorePanel.tsx
          ExploreListView/
            ExploreListView.tsx
            ExploreListItemCard.tsx
          ExploreDetailView/
            ExploreDetailView.tsx

      layout/
        Topbar/
          Topbar.tsx
        RightPanel/
          RightPanel.tsx          // switches ExploreList/Detail
        MapCanvas/
          MapCanvas.tsx

    shared/
      components/
        Buttons/
          IconActionButton.tsx
        Layout/
          Column.tsx
          Panel.tsx
        Feedback/
          EmptyState.tsx
      hooks/
        usePanelSwitch.ts
      utils/
        formatters.ts
```

# Specifically for our system

Naming convention that stays consistent

Panels = layout containers (TripPanel, RightPanel, MapPanel, Topbar, etc).

Views = swappable content inside a panel (ExploreListView, ListingDetailView).

Components = building blocks inside views (TripCard, ListingCard, etc).

✅ Left = single panel with fixed content
✅ Right = single panel with multiple views

```
src/
├─ pages/
│  ├─ map/
│  │  └─ MapPage.tsx        # Top-level layout page (TripPanel + Map + RightPanel)
│  └─ index.tsx             # Other Next.js pages
│
├─ views/
│  ├─ trip/
│  │  └─ TripView.tsx       # Content of the TripPanel (list of trip items, drag/drop, etc.)
│  ├─ explore/
│  │  ├─ ExploreListView.tsx
│  │  └─ ListingDetailView.tsx
│  └─ shared/
│     └─ MapView.tsx        # Center map view
│
├─ components/
│  ├─ panels/               # Panel-level containers
│  │  ├─ TripPanel.tsx
│  │  └─ RightPanel.tsx
│  ├─ layout/               # Shared layout elements
│  │  └─ Topbar.tsx
│  ├─ cards/                # Reusable cards
│  │  ├─ TripCard.tsx
│  │  └─ ListingCard.tsx
│  ├─ map/                  # Map-specific UI
│  │  ├─ MapMarker.tsx
│  │  └─ MapPopup.tsx
│  └─ ui/                   # Atoms / smaller reusable UI (buttons, inputs, icons, etc.)
│     ├─ Button.tsx
│     ├─ Icon.tsx
│     └─ ...
│
├─ hooks/
│  ├─ useTrip.ts
│  ├─ useExplore.ts
│  └─ useDragAndDrop.ts
│
├─ context/
│  ├─ TripContext.tsx
│  └─ ExploreContext.tsx
│
└─ utils/
   └─ formatters.ts
```

pages/map/MapPage.tsx
→ Imports Topbar, TripPanel, MapView, and RightPanel.

TripPanel
→ Always renders TripView → composed of TripCards.

RightPanel
→ Renders either ExploreListView (list of ListingCards) or ListingDetailView.

MapView
→ Handles Mapbox, markers, popups.

```
Component Tree diagram

MapPage (page)
│
├─ Topbar (layout component)
│
├─ TripPanel (panel)
│   └─ TripView (view)
│       ├─ TripCard (component) [*repeated for each trip item]
│       └─ DragAndDropLayer (component, optional)
│
├─ MapView (view)
│   ├─ MapMarker (component) [*repeated for each marker]
│   └─ MapPopup (component, optional)
│
└─ RightPanel (panel)
    ├─ ExploreListView (view)   <-- visible by default
    │   └─ ListingCard (component) [*repeated for each listing]
    │
    └─ ListingDetailView (view) <-- visible when card clicked
        ├─ ListingHeader (component)
        ├─ ListingGallery (component)
        ├─ ListingDescription (component)
        └─ ListingActions (component)
```

## Trip Planner Scaffold

```
src/
├─ pages/
│  └─ map/
│     └─ MapPage.tsx
│
├─ views/
│  ├─ trip/
│  │  └─ TripView.tsx
│  ├─ explore/
│  │  ├─ ExploreListView.tsx
│  │  └─ ListingDetailView.tsx
│  └─ shared/
│     └─ MapView.tsx
│
├─ components/
│  ├─ panels/
│  │  ├─ TripPanel.tsx
│  │  └─ RightPanel.tsx
│  ├─ layout/
│  │  └─ Topbar.tsx
│  ├─ cards/
│  │  ├─ TripCard.tsx
│  │  └─ ListingCard.tsx
│  ├─ map/
│  │  ├─ MapMarker.tsx
│  │  └─ MapPopup.tsx
│  └─ ui/
│     ├─ Button.tsx
│     └─ Icon.tsx

```

(trip-planner-scaffold.zip)

# Panels vs Views

🔹 Panels

Purpose: structural containers in your layout (left, right, bottom, etc.).

They’re usually light-weight: define width, borders, scrolling, background.

Panels shouldn’t “decide” what to render, they just provide the container.

🔹 Views

Purpose: swappable content inside a panel.

If a panel can show different states (e.g. list vs detail), then you introduce views.

Each view is self-contained and can be swapped by state or route.

🔹 When there’s only one thing to show

If a panel only ever shows one type of content (like your TripPanel), then there’s no real need for an extra TripView.

In that case, the panel is the view.

You can simplify to:
```
TripPanel
└─ TripCard(s)
```
instead of
```
TripPanel
└─ TripView
└─ TripCard(s)
```

✅ Rule of thumb

If multiple swappable states → keep a Panel wrapper and nest Views.

If only one state → let the Panel be both container and content. (No need to force a View layer just for symmetry.)

So in your app right now:

TripPanel = a panel with a fixed single view → can just render trip cards directly.

RightPanel = a panel with two views (list + detail) → needs explicit views inside.

```
✅ Revised Folder Structure
src/
├─ pages/
│  └─ map/
│     └─ MapPage.tsx          # Top-level layout page (uses panels)
│
├─ views/
│  ├─ explore/                # Only here because RightPanel has swappable states
│  │  ├─ ExploreListView.tsx
│  │  └─ ListingDetailView.tsx
│  └─ shared/
│     └─ MapView.tsx          # Map is its own "view"
│
├─ components/
│  ├─ panels/
│  │  ├─ TripPanel.tsx        # Acts as both panel + view (only 1 state)
│  │  └─ RightPanel.tsx       # Container that switches between views
│  ├─ layout/
│  │  └─ Topbar.tsx
│  ├─ cards/
│  │  ├─ TripCard.tsx
│  │  └─ ListingCard.tsx
│  ├─ map/
│  │  ├─ MapMarker.tsx
│  │  └─ MapPopup.tsx
│  └─ ui/
│     ├─ Button.tsx
│     └─ Icon.tsx
```
## 🔑 Why this is cleaner

### TripPanel

Only ever shows one thing (trip cards).

No need for TripView. The panel is the view.

If someday you add multiple modes (e.g. TripSummaryView vs TripEditView), then you can promote it into views later.

### RightPanel

Has multiple modes (list vs detail).

Keeps its two views (ExploreListView, ListingDetailView).

### MapView

Makes sense as a View, because it’s a distinct functional block that isn’t a panel but fills the center content.

## 💡 Rule you can follow

If a panel only has one content mode → implement content directly in the panel.

If a panel can switch between multiple content modes → split those modes into views/.
