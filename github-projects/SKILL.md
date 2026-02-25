---
name: github-projects
description: Protocol for creating GitHub issues and linking them to GitHub Projects, including setting custom fields (workstream, dates, blocked-by) via the gh CLI and GraphQL API.
---

# GitHub Issues & Projects — Standard Protocol

Use this skill whenever creating GitHub issues in a repository that has an associated GitHub Project.

---

## Prerequisites

The `project` OAuth scope is required. If not already authorised, run once:

```bash
gh auth refresh -s project
```

---

## Per-project setup (do once per project)

Discover the project number, ID, and custom field names/IDs. Field names differ across projects.

```bash
# List projects for an org
gh project list --owner <ORG>

# List custom fields for a project
gh project field-list <PROJECT_NUMBER> --owner <ORG> --format json
```

Cache the following in the project-level memory file for reuse:
- Project number and node ID (`PVT_...`)
- Field IDs for workstream/grouping, start date, end date (`PVTF_...`, `PVTSSF_...`)
- Option IDs for each single-select field option

---

## Workflow for every new issue

### Step 1 — Before creating, ask the user for:

- **Workstream** (or equivalent grouping field): list available options from the cached project fields; offer to create a new one
- **Start date** and **End date** (YYYY-MM-DD format, or leave blank)
- **Blocked by**: proactively suggest relevant blocking issues based on workflow context (e.g. a data-build issue blocks a downstream analysis issue); give the user the option to decline suggestions or add others

### Step 2 — Create the issue and add to project:

```bash
# Create and add to project in one step
gh issue create \
  --title "Issue title" \
  --body "Issue body" \
  --project "Project Name"

# Or add an existing issue to a project
gh issue edit N --add-project "Project Name"
```

### Step 3 — Get the project item ID:

```bash
gh project item-list <PROJECT_NUMBER> --owner <ORG> --format json
# Filter output with python3 or similar to find the item ID for the new issue
```

### Step 4 — Set workstream (single-select field):

```bash
gh project item-edit \
  --id <ITEM_ID> \
  --project-id <PROJECT_NODE_ID> \
  --field-id <WORKSTREAM_FIELD_ID> \
  --single-select-option-id <OPTION_ID>
```

### Step 5 — Set start and end dates:

```bash
gh project item-edit \
  --id <ITEM_ID> \
  --project-id <PROJECT_NODE_ID> \
  --field-id <START_DATE_FIELD_ID> \
  --date YYYY-MM-DD

gh project item-edit \
  --id <ITEM_ID> \
  --project-id <PROJECT_NODE_ID> \
  --field-id <END_DATE_FIELD_ID> \
  --date YYYY-MM-DD
```

### Step 6 — Set blocked-by relationships:

```bash
# Get the GraphQL node ID for any issue
gh api repos/OWNER/REPO/issues/N --jq .node_id

# Set a blocked-by relationship
gh api graphql -f query='
mutation {
  addBlockedBy(input: {
    issueId: "ISSUE_NODE_ID",
    blockedByIssueId: "BLOCKER_NODE_ID"
  }) {
    issue { number }
  }
}'

# Remove a blocked-by relationship
gh api graphql -f query='
mutation {
  removeBlockedBy(input: {
    issueId: "ISSUE_NODE_ID",
    blockedByIssueId: "BLOCKER_NODE_ID"
  }) {
    issue { number }
  }
}'
```

---

## Reading relationships and fields

```bash
# Read blocked-by / blocking for an issue
gh api graphql -f query='
{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: N) {
      blockedBy(first: 10) { nodes { number title } }
      blocking(first: 10)  { nodes { number title } }
    }
  }
}'

# List all items in a project with their fields
gh project item-list <PROJECT_NUMBER> --owner <ORG> --format json --limit 300
```

---

## Bulk operations

When adding many existing issues to a project and setting workstreams (e.g. during initial project setup):

```bash
# Add multiple issues to a project
for i in 1 2 3 4 5; do
  gh issue edit $i --add-project "Project Name"
done

# Set workstream on multiple items using Python to parse item IDs
gh project item-list <N> --owner <ORG> --format json --limit 300 > /tmp/items.json
python3 << 'EOF'
import json, subprocess
with open('/tmp/items.json') as f:
    data = json.load(f)
for item in data['items']:
    num = item.get('content', {}).get('number')
    # ... filter and call gh project item-edit for each
EOF
```

---

## Notes

- `blockedBy` and `blocking` are fully readable and writable via the GraphQL API (confirmed working as of February 2026)
- Field names and option IDs are project-specific — always look them up once with `gh project field-list` and cache them
- `jq` may not be available; use `python3 -c` or `--jq` flag on `gh api` for JSON parsing
- The `gh project item-list` output includes a `workstream` key directly if the field is set, making it easy to audit which items are missing workstreams

---

## Updating This File

After completing a task, if you encountered any new `gh` CLI behaviour, GraphQL quirk, or
useful pattern not covered above, propose a specific addition and offer to edit this file.

Before adding, check if the learning point is already covered. Prefer refining existing
entries over adding new ones.