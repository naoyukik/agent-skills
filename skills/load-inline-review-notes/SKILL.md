---
name: load-inline-review-notes
description: Use this when a user instructs you to retrieve comments from Inline Review Notes.
metadata:
  version: "1.0"
---
Open @./.inline-review-notes/{current_branch}.json and follow the instructions provided in the unresolved comments (resolvedAt: null). Don't rely solely on your memory; always check the latest information.
For resolved items, add resolvedAt with the current date.