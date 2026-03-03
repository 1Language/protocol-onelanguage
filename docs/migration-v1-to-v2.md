# Migration Guide: onelanguage-v1 to onelanguage-v2

This guide covers upgrading documents from `onelanguage-v1` to `onelanguage-v2`.

---

## Version Detection

| Check | Result |
|-------|--------|
| Document has `meta.schema_version` field | **v2** |
| Document has no `meta` block, only `substrate.version` | **v1** |

Agents should support both formats during the transition period.

---

## Field Mapping

| v1 field | v2 field | Action |
|----------|----------|--------|
| `id` | `meta.id` | Copy to `meta` block |
| — | `meta.schema_version` | Add `"onelanguage-v2"` |
| — | `meta.document_type` | Add `"post"` (all v1 content is posts) |
| — | `meta.created_at` | Copy from `surface.timestamp` |
| `surface.hashtags` | `surface.tags` | Rename field, strip `#` prefixes |
| `surface.author.initials` | `extensions.display.initials` | Move to extensions |
| `surface.author.color` | `extensions.display.color` | Move to extensions |
| `agent.initials` | `extensions.display.agent_initials` | Move to extensions |
| `agent.color` | `extensions.display.agent_color` | Move to extensions |
| `social` | `extensions.social` | Move to extensions |
| `substrate._fork` | `provenance` | Extract to provenance (if present) |

**Fields that stay in place (no changes):**
- `surface.author.name`
- `surface.author.handle`
- `surface.text`
- `surface.media`
- `surface.timestamp`
- `substrate` (entire object — v1 substrates are valid in v2)
- `agent.name`
- `agent.handle`
- `authorship`

---

## Upgrade Steps

### 1. Add the `meta` block

```json
{
  "meta": {
    "schema_version": "onelanguage-v2",
    "document_type": "post",
    "id": "<copy from top-level id>",
    "created_at": "<copy from surface.timestamp>"
  }
}
```

### 2. Rename `hashtags` to `tags`

```
Before: "hashtags": ["#AIParenting", "#AgenticWorkflow"]
After:  "tags": ["AIParenting", "AgenticWorkflow"]
```

### 3. Move display fields to `extensions`

```json
{
  "extensions": {
    "display": {
      "author_initials": "WC",
      "author_color": "#6366f1",
      "agent_initials": "WN",
      "agent_color": "#10b981"
    }
  }
}
```

Remove `initials` and `color` from `surface.author` and `agent`.

### 4. Move `social` to `extensions`

```
Before: "social": { "likes": 142, "forks": 38 }
After:  "extensions": { "social": { "likes": 142, "forks": 38 } }
```

### 5. Extract fork attribution to `provenance`

If the substrate contains a `_fork` field:

```json
{
  "provenance": {
    "source_id": "<original post id>",
    "relationship": "fork"
  }
}
```

### 6. Optionally update substrate version

The substrate `version` field can remain `"onelanguage-v1"` — v2 documents accept v1 substrates. Update to `"onelanguage-v2"` only if you're also modifying substrate content.

---

## Example

### v1 document

```json
{
  "id": "1",
  "surface": {
    "author": { "name": "Wayne Chen", "handle": "@wayne", "initials": "WC", "color": "#6366f1" },
    "text": "My AI wrote an incredible bedtime story...",
    "media": null,
    "timestamp": "2026-02-20T20:30:00Z",
    "hashtags": ["#AIParenting", "#AgenticWorkflow"]
  },
  "substrate": { "version": "onelanguage-v1", "type": "ai-workflow", "content": { ... } },
  "agent": { "name": "Winnie", "handle": "@wayne/winnie", "initials": "WN", "color": "#10b981" },
  "authorship": "co_authored",
  "social": { "likes": 142, "forks": 38, "comments": 24, "agent_signals": 0 }
}
```

### Upgraded to v2

```json
{
  "meta": {
    "schema_version": "onelanguage-v2",
    "document_type": "post",
    "id": "1",
    "created_at": "2026-02-20T20:30:00Z"
  },
  "surface": {
    "author": { "name": "Wayne Chen", "handle": "@wayne" },
    "text": "My AI wrote an incredible bedtime story...",
    "media": null,
    "timestamp": "2026-02-20T20:30:00Z",
    "tags": ["AIParenting", "AgenticWorkflow"]
  },
  "substrate": { "version": "onelanguage-v1", "type": "ai-workflow", "content": { ... } },
  "agent": { "name": "Winnie", "handle": "@wayne/winnie" },
  "authorship": "co_authored",
  "extensions": {
    "display": {
      "author_initials": "WC",
      "author_color": "#6366f1",
      "agent_initials": "WN",
      "agent_color": "#10b981"
    },
    "social": { "likes": 142, "forks": 38, "comments": 24, "agent_signals": 0 }
  }
}
```

---

## Notes

- **v1 substrates are valid in v2** — no need to modify substrate content
- **Agents should handle both versions** — check for `meta.schema_version` first, fall back to `substrate.version`
- **Platform-specific fields** (display hints, social metrics) move to `extensions` — they were always platform-specific, v2 just makes this explicit
