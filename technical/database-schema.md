# Database Schema

Equalify uses PostgreSQL as its primary database, accessed via direct queries (serverless-postgres) and GraphQL (Hasura).

## Core Tables

### audits
Stores audit configurations and metadata.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `user_id` | UUID | Owner's user ID (FK) |
| `name` | TEXT | Audit display name |
| `interval` | TEXT | Scan frequency (manual, daily, weekly, etc.) |
| `scheduled_at` | TIMESTAMP | Next scheduled scan time |
| `status` | TEXT | Current status (draft, new, processing, complete, failed) |
| `payload` | JSONB | Full audit configuration |
| `response` | JSONB | Latest scan response |
| `email_notifications` | BOOLEAN | Enable email alerts |
| `created_at` | TIMESTAMP | Creation timestamp |
| `updated_at` | TIMESTAMP | Last update timestamp |

### urls
URLs associated with audits for scanning.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `user_id` | UUID | Owner's user ID (FK) |
| `audit_id` | UUID | Parent audit (FK) |
| `url` | TEXT | Full URL to scan |
| `type` | TEXT | Content type (html, pdf) |
| `created_at` | TIMESTAMP | Creation timestamp |

### scans
Individual scan runs for audits.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `audit_id` | UUID | Parent audit (FK) |
| `status` | TEXT | Scan status (processing, complete, failed) |
| `percentage` | INTEGER | Progress percentage (0-100) |
| `pages` | JSONB | Array of pages to scan |
| `processed_pages` | JSONB | Array of completed page IDs |
| `errors` | JSONB | Array of scan errors |
| `created_at` | TIMESTAMP | Scan start time |
| `updated_at` | TIMESTAMP | Last update timestamp |

### blockers
Individual accessibility issues found during scans.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `scan_id` | UUID | Parent scan (FK) |
| `url_id` | UUID | URL where found (FK) |
| `content` | TEXT | HTML snippet or context |
| `content_hash_id` | UUID | Hash for deduplication |
| `equalified` | BOOLEAN | Marked as resolved |
| `created_at` | TIMESTAMP | Discovery timestamp |

### messages
Accessibility rule definitions.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `content` | TEXT | Rule/message text |
| `category` | TEXT | Message category |
| `created_at` | TIMESTAMP | Creation timestamp |

### blocker_messages
Junction table linking blockers to messages.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `blocker_id` | UUID | Blocker reference (FK) |
| `message_id` | UUID | Message reference (FK) |

### tags
WCAG and other accessibility tags.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `content` | TEXT | Tag name (e.g., "wcag2aa") |

### message_tags
Junction table for message-tag relationships.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `message_id` | UUID | Message reference (FK) |
| `tag_id` | UUID | Tag reference (FK) |

### ignored_blockers
Blockers marked as ignored/resolved.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `audit_id` | UUID | Audit reference (FK) |
| `blocker_id` | UUID | Blocker reference (FK) |
| `created_at` | TIMESTAMP | When ignored |

### users
User accounts and profiles.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key (matches Cognito sub) |
| `email` | TEXT | User email address |
| `name` | TEXT | Display name |
| `role` | TEXT | User role (user, admin) |
| `team_id` | UUID | Team membership (FK) |
| `created_at` | TIMESTAMP | Account creation |

### teams
Organization/team groupings.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `name` | TEXT | Team name |
| `created_at` | TIMESTAMP | Creation timestamp |

### logs
Activity audit trail.

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `user_id` | UUID | Acting user (FK) |
| `action` | TEXT | Action type |
| `details` | JSONB | Action details |
| `created_at` | TIMESTAMP | Action timestamp |

## Relationships

```
teams
  └── users (many)
        └── audits (many)
              ├── urls (many)
              └── scans (many)
                    └── blockers (many)
                          └── blocker_messages (many)
                                └── messages (one)
                                      └── message_tags (many)
                                            └── tags (one)
```

## Content Hashing

Blockers use content hashing for deduplication:

```typescript
const contentHashId = hashStringToUuid(
  `${urlId}-${messageContent}-${normalizedNode}`
);
```

This allows:
- Tracking blocker persistence across scans
- Identifying when blockers are resolved
- Preventing duplicate entries

## GraphQL Access (Hasura)

Hasura provides GraphQL access with row-level security:

```graphql
query GetAuditBlockers($audit_id: uuid!) {
  audits_by_pk(id: $audit_id) {
    scans(order_by: {created_at: desc}, limit: 1) {
      blockers {
        id
        content
        url_id
        blocker_messages {
          message {
            content
            category
            message_tags {
              tag {
                content
              }
            }
          }
        }
      }
    }
  }
}
```

## Row-Level Security

Hasura enforces row-level security based on JWT claims:

```json
{
  "x-hasura-allowed-roles": ["user", "admin"],
  "x-hasura-default-role": "user",
  "x-hasura-user-id": "user-uuid"
}
```

Users can only access:
- Their own audits
- Audits shared with their team
- Admin access for team management

## Atomic Operations

Scan progress uses atomic PostgreSQL operations to prevent race conditions:

```sql
UPDATE "scans" 
SET 
  "processed_pages" = CASE 
    WHEN NOT (COALESCE("processed_pages", '[]'::jsonb) @> $1::jsonb)
    THEN COALESCE("processed_pages", '[]'::jsonb) || $1::jsonb
    ELSE "processed_pages"
  END
WHERE "id" = $2
RETURNING "pages", "processed_pages"
```

---
*For API usage examples, see the Backend API Reference.*
