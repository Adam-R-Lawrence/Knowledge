<!--
title: Personal Knowledge Database
description: A plain‑Markdown knowledge base with consistent metadata and structure.
tags: [knowledge, markdown]
-->

# Personal Knowledge Database

## Purpose
Store and organize notes on various topics in plain Markdown files.

## File Naming
- One concept per file.
- Use descriptive, kebab-case names ending in `.md` (e.g., `photopolymerization.md`).

## Metadata Front Matter
Place invisible metadata at the very top inside an HTML comment. This keeps it hidden on GitHub while remaining parseable by tools.
```html
<!--
title: Note Title
tags: [tag1, tag2]
date: YYYY-MM-DD
-->
```

## Note Structure Guidelines
- Start the main topic with a `##` heading.
- Use `###` for subtopics.
- Write lists using standard Markdown syntax.
- Avoid embedded HTML and complex tables.

## Adding or Updating Entries
1. Create or edit a Markdown file following the naming rules.
2. Optionally add YAML front matter at the top.
3. Structure content using `##`/`###` headings and lists.
4. Commit the changes to the repository.

## OneDrive Sync

To sync only this `Knowledge` directory with OneDrive, run:

```bash
onedrive --synchronize --single-directory "Knowledge"
```
