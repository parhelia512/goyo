+++
title = "Page Configuration"
description = "Learn how to configure individual pages in Goyo."
weight = 5
sort_by = "weight"

[extra]
toc_expand = true
+++

Page configuration allows you to customize the behavior and appearance of individual pages in your Goyo documentation site. These settings are defined in the front matter of each content file using the `[extra]` section.

> **Note:** This page demonstrates the `toc_expand` feature - notice how the Table of Contents on the right is fully expanded by default!

## Table of Contents (TOC) Configuration

### TOC Expand
`toc_expand`

- `toc_expand`: Controls whether the Table of Contents (TOC) on the page should be expanded by default.
  - `false` (default): TOC sections are collapsed and will automatically expand/collapse based on scroll position
  - `true`: TOC sections are expanded by default and remain expanded regardless of scroll position

**Default Behavior (toc_expand = false or not set):**
```toml
+++
title = "My Page"
description = "Page description"

[extra]
# TOC will be collapsed by default and auto-expand based on scroll
+++
```

When `toc_expand` is not set or set to `false`, the TOC behaves dynamically:
- Sections with sub-headings are collapsed by default
- As you scroll, the current section automatically expands
- Other sections collapse to maintain a clean view

**Expanded TOC (toc_expand = true):**
```toml
+++
title = "My Page"
description = "Page description"

[extra]
toc_expand = true
+++
```

When `toc_expand` is set to `true`:
- All TOC sections are expanded by default
- All sections remain expanded regardless of scroll position
- Useful for pages with many short sections where you want to see all headings at once

## Example Usage

Here's a complete example of a page with expanded TOC:

```toml
+++
title = "API Reference"
description = "Complete API documentation with all endpoints visible"
weight = 10

[extra]
toc_expand = true
+++

# API Reference

This page shows all API endpoints with the TOC fully expanded.

## Authentication
Details about authentication...

## Endpoints
### GET /api/users
### POST /api/users
### DELETE /api/users

## Error Codes
### 400 Bad Request
### 401 Unauthorized
### 404 Not Found
```

With `toc_expand = true`, readers can see all endpoints and error codes in the TOC at once, making it easier to navigate the API reference.
