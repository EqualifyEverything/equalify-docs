# Backend API Reference

The Equalify backend is a single AWS Lambda function that acts as a router, directing requests to appropriate handlers based on the URL path and authentication status.

## Entry Point

The main handler (`apps/backend/index.ts`) routes requests based on path prefixes:

| Path Prefix | Router | Authentication |
|-------------|--------|----------------|
| `/public` | `publicRouter` | None |
| `/auth` | `authRouter` | JWT required |
| `/internal` | `internalRouter` | Internal only |
| `/scheduled` | `scheduledRouter` | EventBridge |
| `/hasura` | `hasuraRouter` | Webhook secret |
| Cognito triggers | `cognitoRouter` | Cognito events |

## Route Organization

### Public Routes (`/public`)
Endpoints that don't require authentication:

- **`POST /public/scanWebhook`** - Receives scan results from scanner Lambdas
- **`POST /public/requestAccess`** - Submits a self-registration access request (name + email). On SSO instances, the email must belong to an authorized domain (`SSO_EMAIL_DOMAINS`); duplicate requests, existing invites, and existing accounts are rejected

### Auth Routes (`/auth`)
Endpoints requiring a valid JWT token:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `getAccount` | GET | Retrieve user account details |
| `updateUser` | POST | Update user profile |
| `saveAudit` | POST | Create a new audit |
| `updateAudit` | POST | Modify an existing audit |
| `deleteAudit` | POST | Delete an audit |
| `rescanAudit` | POST | Trigger a new scan for an audit |
| `getAuditDetails` | GET | Get audit configuration |
| `getAuditResults` | GET | Get scan results with blockers |
| `getAuditChart` | GET | Get chart data for blockers over time |
| `getAuditTable` | GET | Get paginated blocker table data |
| `getAuditProgress` | GET | Get current scan progress |
| `getLogs` | GET | Get activity logs |
| `inviteUser` | POST | Invite a new user to the team |
| `getAccessRequests` | POST | List pending self-registration access requests (admin only) |
| `reviewAccessRequest` | POST | Approve or deny an access request; approval creates an invite and emails the requester (admin only) |
| `trackUser` | POST | Track user analytics events |
| `chatWithAudit` | POST | AI-powered audit chat interface |

### Scheduled Routes (`/scheduled`)
Endpoints triggered by AWS EventBridge:

- **`runEveryMinute`** - Processes scheduled audits and triggers scans

### Hasura Routes (`/hasura`)
Webhook endpoints for Hasura event triggers (protected by webhook secret).

## Authentication Flow

### JWT Verification

```typescript
// For Cognito tokens
const verifier = CognitoJwtVerifier.create({
  userPoolId: process.env.USER_POOL_ID,
  tokenUse: "id",
  clientId: process.env.WEB_CLIENT_ID
});
const claims = await verifier.verify(token);

// For SSO tokens
const rawClaims = await verifySsoToken(token);
const { normalizedClaims, hasuraClaims } = await ensureSsoUser(rawClaims);
```

### Token Structure

Tokens include Hasura claims for GraphQL authorization:
```json
{
  "sub": "user-uuid",
  "email": "user@example.com",
  "https://hasura.io/jwt/claims": {
    "x-hasura-allowed-roles": ["user", "admin"],
    "x-hasura-default-role": "user",
    "x-hasura-user-id": "user-uuid"
  }
}
```

## Database Access

### Direct PostgreSQL
Using `serverless-postgres` for connection pooling:

```typescript
import { db } from '#src/utils';

await db.connect();
const result = await db.query({
  text: `SELECT * FROM "audits" WHERE "id" = $1`,
  values: [auditId],
});
await db.clean();
```

### GraphQL via Hasura
For complex queries with relationships:

```typescript
import { graphqlQuery } from '#src/utils';

const response = await graphqlQuery({
  query: `query ($audit_id: uuid!) {
    audits_by_pk(id: $audit_id) {
      scans(order_by: {created_at: desc}, limit: 1) {
        blockers { ... }
      }
    }
  }`,
  variables: { audit_id: auditId },
});
```

## Creating an Audit

When a user creates an audit (`saveAudit`):

1. Insert audit record with configuration
2. Insert all URLs associated with the audit
3. If `saveAndRun` is true:
   - Create a new scan record
   - Invoke the scan router Lambda asynchronously

```typescript
await lambda.send(new InvokeCommand({
  FunctionName: "aws-lambda-scan-sqs-router",
  InvocationType: "Event",
  Payload: JSON.stringify({
    urls: urls.map(url => ({
      auditId, scanId, urlId: url.id,
      url: url.url, type: url.type, isStaging
    }))
  })
}));
```

## Webhook Processing

The `scanWebhook` endpoint processes results from scanners:

1. Validate required fields (`auditId`, `urlId`)
2. Handle failed scans (log error, update progress)
3. For successful scans:
   - Store blockers with content hashing for deduplication
   - Link blockers to messages (accessibility rules)
   - Update scan progress atomically
   - Mark scan complete when all URLs processed

## Environment Variables

| Variable | Description |
|----------|-------------|
| `DB_USER` | PostgreSQL username |
| `DB_HOST` | PostgreSQL host |
| `DB_NAME` | Database name |
| `DB_PASSWORD` | Database password / Hasura admin secret |
| `USER_POOL_ID` | Cognito User Pool ID |
| `WEB_CLIENT_ID` | Cognito Web Client ID |
| `SSO_ENABLED` | Enable SSO authentication |
| `WEBHOOKSECRET` | Secret for Hasura webhooks |

---
*For frontend integration details, see the Frontend Guide.*
