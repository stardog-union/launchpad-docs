---
name: Launchpad Release Notes
description: Prepare release notes for a new Launchpad version. Use when the user asks to write, create, or prepare release notes for a new Launchpad release.
---

# Launchpad Release Notes

## Overview

This skill guides the process of writing release notes for a new Launchpad version. It fetches the latest application versions, collects release details from the user, and updates `release-notes.md` following the established format.

## Instructions

### Step 1: Fetch Latest Application Versions

Fetch the latest version for each bundled application from the Stardog docs site. Visit each page and extract the most recent version number:

| Component | Release Notes URL |
|-----------|-------------------|
| Designer | https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer |
| Explorer | https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer |
| Studio | https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio |
| Knowledge Catalog | https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog |

Present the fetched versions to the user for confirmation before proceeding.

### Step 2: Collect Release Details

Ask the user for the following information:

1. **Version number** (e.g., `3.8.0`)
2. **Release date** (e.g., `2026-02-19`)
3. **Changes** — Ask the user to describe the new features, bug fixes, modifications, and/or security changes included in this release. Continue collecting items until the user indicates they are done.

### Step 3: Read Existing Release Notes

Read `release-notes.md` to understand the current format and identify where to insert the new entry.

### Step 4: Update the Version Table

Add a new row to the **top** of the version table in `release-notes.md`. Follow the exact format of existing rows, including links to each component's release notes on the Stardog docs site.

The anchor link format for component versions is `#vXYZ-release` where XYZ is the version with dots removed (e.g., version `3.7.0` becomes `#v370-release`).

### Step 5: Add the Release Section

Insert a new release section **above** the previous release, immediately after the version table. Use the following structure:

```markdown
## X.Y.Z Release (YYYY-MM-DD)

### New Features

- Description of feature 1.
- Description of feature 2.

### Bug Fixes

- Description of fix.

### Modifications

- Description of modification.

### Security

- Description of security change.
```

**Rules:**
- Only include subsections (New Features, Bug Fixes, Modifications, Security) that have content.
- If there is a recommended Stardog or Voicebox version, add an `> [!IMPORTANT]` callout block at the top of the release section (see v3.7.0 and v3.6.0 entries for examples).
- Match the tone and style of existing release notes entries.

### Step 6: Verify

Read back the updated section to the user for review.
