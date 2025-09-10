# TRIPPL DEVELOPMENT STANDARDS

**Created:** 2025-01-15  
**Updated:** 2025-01-15  
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
