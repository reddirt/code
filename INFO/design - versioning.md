# Trippl App Versioning & Release Workflow

This document describes the **versioning and release process** for Trippl’s Dev, Staging (RC), and Production environments. It also includes the automated GitHub Actions workflows.

---

## **1. Versioning Overview**

| Environment | Version Format         | How Set / Source        | Notes |
|------------|----------------------|------------------------|------|
| **Dev**    | `X.Y.Z-dev-<shortSHA>` | From `package.json` / `VERSION` + Git SHA | Auto-generated per build, no Git tags |
| **Staging (RC)** | `X.Y.Z-RCn`          | Git tag auto-created when `main` merges into `staging` | RC number auto-increments per merge |
| **Production** | `X.Y.Z`              | Current `package.json` version used for Git tag | Release notes added in GitHub Release; app displays version & notes; `package.json` bumped after release for next cycle |

**Rules:**

1. Dev builds **never modify `package.json` or create tags**.  
2. Staging merges create **incremental RC tags** automatically.  
3. Production releases:
   - Use current `package.json` version for Git tag.
   - Create GitHub Release with release notes.
   - Build app with embedded version & notes.
   - **Then bump `package.json`** for the next development cycle.

---

## **2. Version Flow Diagram**

```
         +----------------+
         |  Dev Branches  |
         +----------------+
                 |
                 | builds: X.Y.Z-dev-SHA
                 v
         +----------------+
         |      Main      |
         +----------------+
                 |
                 | merge
                 v
         +----------------+
         |    Staging     |
         +----------------+
                 |
                 | RC tag auto: X.Y.Z-RCn
                 v
         +----------------+
         |   RC Testing   |
         +----------------+
                 |
                 | (Release approved)
                 v
         +----------------+
         |   Production   |
         +----------------+
                 |
                 | Manual workflow triggered on GitHub
                 |
                 | Build app with version & notes
                 | Use current version for Git tag
                 | Create GitHub Release with notes
                 |
                 | Bump package.json for next cycle
                 v
        Next Dev / Staging cycle starts
```

---

## **3. Notes**

- **Dev builds:** Auto version with SHA, no tagging.  
- **Staging RCs:** Auto-incremented RC tags, visible in staging.  
- **Production:** Uses current `package.json` version, release notes embedded in app, then `package.json` bumped for the next cycle.  
- **App UI:** Reads version and release notes from environment variables or embedded config at build time.
