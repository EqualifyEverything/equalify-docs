# User Roles and Permissions

Equalify uses a role-based system to manage what users can do within the platform. This guide explains the available roles and how team membership works.

## Roles

### Admin
The **Admin** role is automatically assigned to the first user who logs in to a new Equalify instance. Admins have full access to all platform features, including team management.

Admin capabilities:
- Create, edit, and delete audits
- View all scan results and blockers
- Invite new users to the team
- Manage team membership
- Access activity logs
- Configure audit schedules and notifications

### User
The **User** role is the default role for all invited users. Users have access to the core scanning and reporting features within their team.

User capabilities:
- Create, edit, and delete their own audits
- View scan results and blockers for audits shared with their team
- Invite new users to the team
- Access activity logs
- Configure audit schedules and notifications

## Teams

Every user in Equalify belongs to a **team**. Teams are the primary way that audit access is shared — all members of a team can view audits created by other team members.

- When the first user (Admin) creates an account, a team is automatically created.
- Invited users are added to the inviting user's team.
- Audits are visible to all members of the same team.

## Inviting Users

Any authenticated user can invite new members to join the platform:

1. Navigate to **Account** from the main menu.
2. Enter the email address of the person you want to invite.
3. Click **Invite**.
4. The invited user will receive an email with a link to log in.

Once the invited user logs in (via SSO or other authentication), they are automatically added to your team.

> **Note**: If your Equalify instance uses SSO, invited users must have an email address from an authorized domain (e.g., `@uic.edu`).

## Data Access

Equalify enforces data isolation so that users only see what they should:

- **Your audits**: You can always see audits you created.
- **Team audits**: You can see audits created by anyone on your team.
- **Activity logs**: Logs reflect actions taken across your team's audits.

This means you will not see audits or data from users outside your team.
