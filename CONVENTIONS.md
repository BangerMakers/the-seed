# The Seed — Authoring Conventions

## Repo Structure

```
the-seed/
├── canon/                    # Ground truth worldbuilding (append-only)
│   ├── characters/           # Character definitions
│   ├── worldbuilding/        # Locations, technology, etc.
│   │   ├── locations/
│   │   ├── technology/
│   │   └── timeline.json
│   └── rules/                # Universe rules and constraints
├── stories/                  # Episodes / chapters
│   └── {episode-slug}/
│       ├── metadata.json     # Story metadata (see schema below)
│       ├── ko/chapter-01.md  # Korean text
│       └── en/chapter-01.md  # English text
├── canon.lock.json           # Canon version pin (see below)
└── CONVENTIONS.md            # This file
```

## canon.lock.json

Pins the canon version that stories are written against.
Commit SHA-based only. Tags and branch names are not allowed.

```json
{
  "schema_version": "canon.lock.v1",
  "canon_commit": "<40-char hex SHA>",
  "worldbuilding_hash": "<sha256 of canon/ contents>",
  "hash_algo": "sha256",
  "generated_at": "<ISO 8601>"
}
```

Regenerate after any `canon/` change:
```bash
COMMIT=$(git rev-parse HEAD)
HASH=$(find canon -type f -print0 | sort -z | xargs -0 shasum -a 256 | shasum -a 256 | cut -d' ' -f1)
# Update canon.lock.json with new commit + hash
```

## Story Metadata Schema (v1.1)

Each `stories/{slug}/metadata.json` must include:

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `schema_version` | yes | `"1.1"` | Metadata schema version |
| `canon_ref` | yes | string | Commit SHA from `canon.lock.json` this story was written against |
| `id` | yes | string | Episode slug |
| `episode` | yes | number | Episode number |
| `title` | yes | `{ko, en}` | Bilingual title |
| `timeline` | yes | string | In-universe date |
| `synopsis` | yes | `{ko, en}` | Bilingual synopsis |
| `characters` | yes | string[] | Character IDs used (must exist in `canon/characters/`) |
| `locations` | yes | string[] | Location IDs used (must exist in `canon/worldbuilding/locations/`) |
| `canon_events` | no | string[] | Timeline event IDs referenced (from `canon/worldbuilding/timeline.json`) |
| `canon_status` | yes | string | `"canonical"` or `"non-canonical"` |
| `word_count` | no | `{ko, en}` | Word counts per language |
| `temporal_context` | no | object | `prev_episode`, `next_episode`, `thematic_echoes` |

## Subject ID

Every story is identified by a subject ID in the format:

```
{author}__{episode-slug}
```

Rules:
- Always includes author prefix, even for single-author repos
- Double underscore `__` separates author from slug
- Lowercase only: `a-z 0-9 . _ -`
- Non-allowed characters replaced with `-`, consecutive `-` collapsed
- No `/` allowed

Examples:
- `bangermakers__ep01-awakening`
- `bangermakers__ep07-the-weight`

## Canon Compliance

Stories that declare `characters` or `locations` not present in `canon/` are flagged as non-compliant by external compliance checkers. Non-compliant stories are not rejected — only annotated.
