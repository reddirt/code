# Naming
### Tickets (Tasks in Notion)
``` 
[Area] - [Short Description]

Trips - Add fisheye effect to trip stop scrolling
```
### Branches
```
[type]/[ticketID]-[Area]-[Repo-Specific-Short-Description]

Feature/TT-100-Trips-add-db-schema-for-new-field

```
### PRs
```
[type] [ticketID] [Area] - [Full Descriptive Summary]

Feature TT-100 Trips - Improved UI with Fisheye Scrolling
```
### Release Notes
I am planning to generate relase notes based on PRs, so PR titles are important

### Definitions
- **[Type]** -  Dropdown list [Feature, Chore, Bug ...]
- **[TicketID]** - Notion task id (TT-###)
- **[Area]** - generally feature level, can be code specific if neccesary (ie component)
- **[Description]** - keep it short
- **Commit messages** - imperative tense ('Add button', 'Update README')
- **PR Titles** - Description / Past tense - what the PR does or did: 'Add trip sharing feature', 'Fix bug causing API crash', 'Updated README with setup guide'.

# Repo Names
- db-schema
- api-server
- app-web
- app-android
- app-ios

Options for additional mobile business
