# TRIPPL DEVELOPMENT STANDARDS

**Created:** 2025-01-15  
**Updated:** 2025-01-15  
**Location:** Root workspace standards for all repositories

---

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

---

## 🎯 **Functional Programming Standards**

This codebase follows **strict functional programming patterns**. All code must adhere to these rules:

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

---

## 📝 **Naming Conventions**

### **Tickets (Tasks in Notion)**
```
[Area] - [Short Description]

Trips - Add fisheye effect to trip stop scrolling
```

### **Branches**
```
[type]/[ticketID]-[area]-[repo-specific-short-description]

feat/TT-100-Trips-add-db-schema-for-new-field
```

### **PRs**
```
[type] [ticketID] [area] - [full descriptive summary]

feat TT-100 trips - improved UI with fisheye scrolling
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

### **Repo Names**
- db-schema
- api-server
- app-web
- app-android
- app-ios

---

## 🔍 **Code Review Checklist**

When reviewing code that makes API calls, check:

- [ ] Uses existing API layer (`src/api/`) instead of direct fetch
- [ ] No hardcoded URLs
- [ ] Proper error handling with `response.isValid()`
- [ ] TypeScript interfaces defined
- [ ] Environment variables only used for build-time config
- [ ] Configuration documented with comments
- [ ] Tested in multiple environments
- [ ] Follows functional programming patterns
- [ ] Uses arrow functions and `const` declarations
- [ ] No classes or `this` keyword

---

## 🛠 **ESLint Rules to Enforce**

The following ESLint rules are configured to prevent violations:

```javascript
// RULE: API Standards Enforcement
'no-restricted-syntax': [
  'error',
  {
    selector: 'CallExpression[callee.name="fetch"]',
    message: '🚫 RULE VIOLATION: Use the API layer (src/api/) instead of direct fetch calls.',
  },
  {
    selector: 'MemberExpression[object.property.name="body"][property.name="result"]',
    message: '🚫 RULE VIOLATION: Always use response.isValid() before accessing response.body.',
  },
],
```

---

## 📁 **Component Organization**

### **Naming Scheme Cheat Sheet**

- **Panels/Columns/Bars**: ExplorePanel, TripPanel, Topbar, TripMiniDashboardBar
- **Containers**: TripItemsPanel, RightPanel
- **Cards**: TripHeaderCard, TripItemCard, TimeDistanceCard
- **Variants by type**: TripItem.StartCard, TripItem.EndCard, TripItem.AccommodationCard
- **Parts (sub-elements)**: Accommodation.CheckInCard, Accommodation.CheckOutCard
- **Markers**: TimelineDayMarker
- **Views**: ExploreListView, ExploreDetailView

### **Folder Structure**
```
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

---

## 🎨 **UI Standards**

### **Material UI Only**
- ✅ **CORRECT**: Use MUI sx props, styled components, or theme overrides
- ❌ **WRONG**: Tailwind CSS classes

### **Atomic Design Pattern**
Use Atomic Design principles for components: atoms, molecules, organisms, templates, pages.

---

## 🔧 **Backend Standards**

### **No Classes in Backend Code**
Favor functional programming over class-based structure in Express routes and services.

### **Use LogUtils Instead of Console**
- ✅ **CORRECT**: `LogUtils.info()`, `LogUtils.warn()`, `LogUtils.error()`
- ❌ **WRONG**: `console.log()`, `console.warn()`, `console.error()`

### **Use accountId Instead of userId**
- ✅ **CORRECT**: `accountId`, `account_id`
- ❌ **WRONG**: `userId`, `user_id`

### **RBAC Enforcement**
Ensure protected API routes use RBAC middleware like `requirePermission` or `withRBAC`.

---

## 📊 **Database Standards**

### **Liquibase Migrations Only**
Do not alter schema directly; use Liquibase changelogs for all schema changes.

---

## 🤖 **AI Behavior Rules**

- Always understand which repository you're working in
- Coordinate changes across multiple repositories when needed
- Use extremely brief commit messages
- Ask before generating scripts or documentation
- Do not run build commands unless specifically requested
- Do not start development servers unless explicitly requested
- Assume servers are already running with hot reload
