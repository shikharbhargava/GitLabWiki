# Users and Members

## Overview
GitLab has a comprehensive permission system with different user types, roles, and access levels. Understanding these distinctions is crucial for proper access management.

## User Types

### 1. Regular Users
- Standard users with a license seat
- Can create projects, groups, and collaborate
- Can be assigned to projects and groups
- Count towards license user limit

### 2. External Users
- Limited permissions across the GitLab instance
- Can only access projects they are explicitly members of
- Cannot create groups or projects (by default)
- Cannot see internal or public projects unless member
- **Use case**: Contractors, external consultants

### 3. Service Accounts
- Non-human users for automation
- Used for CI/CD, integrations, API access
- Should use project/group access tokens or deploy tokens

### 4. Bot Users
- Automated accounts (e.g., Alert Bot, Support Bot)
- Do not count towards license
- Created by GitLab for specific features

### 5. Administrator
- Highest level of access
- Can access all projects and groups
- Can modify instance settings
- Can impersonate other users
- **Use case**: GitLab instance management

## Project Roles and Permissions

### Role Overview

| Permission | Guest | Reporter | Developer | Maintainer | Owner |
|-----------|-------|----------|-----------|------------|-------|
| View project | ✓ | ✓ | ✓ | ✓ | ✓ |
| Create issues | ✓ | ✓ | ✓ | ✓ | ✓ |
| Comment on issues | ✓ | ✓ | ✓ | ✓ | ✓ |
| View wiki | ✓ | ✓ | ✓ | ✓ | ✓ |
| Download project | | ✓ | ✓ | ✓ | ✓ |
| Pull code | | ✓ | ✓ | ✓ | ✓ |
| Create merge requests | | | ✓ | ✓ | ✓ |
| Push to non-protected branches | | | ✓ | ✓ | ✓ |
| Manage merge requests | | | ✓ | ✓ | ✓ |
| Create tags | | | ✓ | ✓ | ✓ |
| Push to protected branches | | | | ✓ | ✓ |
| Enable/disable branch protection | | | | ✓ | ✓ |
| Add project members | | | | ✓ | ✓ |
| Manage CI/CD settings | | | | ✓ | ✓ |
| Delete project | | | | | ✓ |
| Transfer project | | | | | ✓ |

### Guest
**Permissions:**
- View project issues, boards, wiki, and snippets
- Create and comment on issues
- Leave comments on merge requests
- Cannot view code or pull repository

**Use case:** Stakeholders, project managers who need to track progress

### Reporter
**Permissions:**
- All Guest permissions
- View and download code
- Pull project repository
- View CI/CD jobs and logs
- Create and manage labels

**Use case:** QA testers, product owners, auditors

### Developer
**Permissions:**
- All Reporter permissions
- Create and push branches
- Create merge requests
- Push to non-protected branches
- Create and manage CI/CD pipelines
- Lock and unlock merge requests
- Manage merge requests
- Create and edit wiki pages

**Use case:** Software developers, contributors

### Maintainer
**Permissions:**
- All Developer permissions
- Push to protected branches
- Add and remove project members
- Manage protected branches and tags
- Edit project settings
- Configure webhooks and integrations
- Manage CI/CD variables
- View project audit events

**Use case:** Tech leads, project maintainers

### Owner (Project Level)
**Permissions:**
- All Maintainer permissions
- Transfer project to another namespace
- Delete project
- Manage project access tokens
- Change project visibility

**Use case:** Project owners, team leads

## Group Roles and Permissions

### Group Role Hierarchy

| Permission | Guest | Reporter | Developer | Maintainer | Owner |
|-----------|-------|----------|-----------|------------|-------|
| View group | ✓ | ✓ | ✓ | ✓ | ✓ |
| View group members | ✓ | ✓ | ✓ | ✓ | ✓ |
| Create projects in group | | | ✓ | ✓ | ✓ |
| Create subgroups | | | | ✓ | ✓ |
| Add group members | | | | ✓ | ✓ |
| Manage group CI/CD variables | | | | ✓ | ✓ |
| Delete group | | | | | ✓ |
| Transfer group | | | | | ✓ |

### Group Members vs Project Members
- Group membership can be inherited by projects
- Group-level role applies to all projects in the group
- Project-specific roles can override group roles
- Maximum permission applies when multiple roles exist

## Adding Members

### Add Member to Project

#### Via Web Interface
1. Navigate to project
2. Go to **Settings → Members**
3. Click **Invite members**
4. Enter username, email, or select from dropdown
5. Select role (Guest, Reporter, Developer, Maintainer)
6. Set expiration date (optional)
7. Click **Invite**

#### Via API
```bash
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "user_id=42&access_level=30" \
  "https://gitlab.example.com/api/v4/projects/:id/members"
```

**Access Levels:**
- 10 = Guest
- 20 = Reporter
- 30 = Developer
- 40 = Maintainer
- 50 = Owner

### Add Member to Group

```bash
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "user_id=42&access_level=30" \
  "https://gitlab.example.com/api/v4/groups/:id/members"
```

### Invite Members via Email
1. Go to project/group **Settings → Members**
2. Enter email address
3. Select role and expiration
4. Click **Invite**
5. User receives invitation email
6. Must create GitLab account to accept

## Managing Members

### Update Member Role

#### Via Web Interface
1. Navigate to **Settings → Members**
2. Find member in list
3. Change role from dropdown
4. Click **Update**

#### Via API
```bash
curl --request PUT \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "access_level=40" \
  "https://gitlab.example.com/api/v4/projects/:id/members/:user_id"
```

### Remove Member

#### Via Web Interface
1. Navigate to **Settings → Members**
2. Find member in list
3. Click **Remove member**
4. Confirm removal

#### Via API
```bash
curl --request DELETE \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.example.com/api/v4/projects/:id/members/:user_id"
```

### Set Expiration Date
- Automatically removes member after date
- Useful for temporary access (contractors, interns)
- Can be set during invitation or edited later

## Group and Project Sharing

### Share Project with Group

```
Project Settings → Members → Invite group
```

**Configuration:**
- Select group to invite
- Choose maximum access level
- Set expiration date (optional)

**Example:**
```
Share "web-api" project with "QA Team" group as Reporters
Result: All QA Team members get Reporter access to web-api
```

### Share Group with Group

```
Group Settings → Members → Invite group
```

- Allows entire group to access another group
- Applies to all projects in shared group
- Useful for cross-team collaboration

## External Users Configuration

### Set User as External

#### Via Admin Area
1. **Admin Area → Users**
2. Select user
3. Click **Edit**
4. Check **External** checkbox
5. Click **Save changes**

#### Via API
```bash
curl --request PUT \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "external=true" \
  "https://gitlab.example.com/api/v4/users/:id"
```

### External User Restrictions
```ruby
# In /etc/gitlab/gitlab.rb
gitlab_rails['gitlab_default_projects_features_issues'] = false
gitlab_rails['gitlab_default_projects_features_merge_requests'] = false
gitlab_rails['gitlab_default_projects_features_wiki'] = false
```

## Protected Branches and Members

### Configure Protected Branch Permissions

```
Settings → Repository → Protected branches
```

**Options:**
- **Allowed to merge:**
  - No one
  - Developers + Maintainers
  - Maintainers
  - Specific users/groups

- **Allowed to push:**
  - No one
  - Developers + Maintainers
  - Maintainers
  - Specific users/groups

- **Allowed to force push:**
  - Enabled/Disabled

**Example:**
```yaml
Branch: main
Allowed to merge: Maintainers only
Allowed to push: No one
Allowed to force push: Disabled
```

## Code Owners (Premium/Ultimate)

### CODEOWNERS File
Define automatic reviewers for specific files/directories:

```bash
# Create file: .gitlab/CODEOWNERS or CODEOWNERS

# Default owners
* @username @group/subgroup

# Frontend code
*.js @frontend-team
*.vue @frontend-team
src/components/ @frontend-team

# Backend code
*.py @backend-team
api/ @backend-team

# Database migrations
db/migrations/ @dba-team @backend-team

# CI/CD configuration
.gitlab-ci.yml @devops-team
```

### Benefits
- Automatic reviewer assignment
- Required approvals from code owners
- Clear ownership of code sections
- Enforce review process

## Merge Request Approvals (Premium/Ultimate)

### Configure Approval Rules

```
Settings → Merge requests → Approval rules
```

**Settings:**
- Number of approvals required
- Eligible approvers (specific users, groups, or roles)
- Approval rules for specific paths
- Prevent committers from approving own merge requests
- Prevent author from approving
- Remove approvals on new commits

**Example Rule:**
```yaml
Rule: "Security Team Approval"
Approvals required: 2
Eligible approvers: @security-team
Target branches: main, production
File paths: *.yml, Dockerfile, requirements.txt
```

## Audit Events (Premium/Ultimate)

Track member activities:

```
Settings → Audit Events
```

**Tracked Events:**
- Member added/removed
- Permission level changed
- Project shared/unshared
- Deploy key added/removed
- Variable added/modified/deleted

## Best Practices

### 1. Principle of Least Privilege
- Grant minimum necessary permissions
- Start with lower role and increase if needed
- Regular permission audits

### 2. Use Groups for Team Management
```
✅ Good:
Group: Engineering
├── Subgroup: Frontend (frontend-team members)
├── Subgroup: Backend (backend-team members)
└── Subgroup: DevOps (devops-team members)

Add teams to projects as needed

❌ Avoid:
Adding individual users to every project
```

### 3. Set Expiration Dates
- Always set expiration for contractors
- Set expiration for temporary access
- Review and extend as needed

### 4. Use External Users for Non-Employees
- Mark contractors as external
- Mark consultants as external
- Mark external collaborators as external

### 5. Regular Access Reviews
- Quarterly review of project members
- Remove inactive users
- Verify roles are appropriate
- Check for orphaned accounts

### 6. Document Access Policies
Create `ACCESS_POLICY.md`:
```markdown
# Access Policy

## Roles
- Guest: Stakeholders, observers
- Reporter: QA, product owners
- Developer: Engineers, contributors
- Maintainer: Tech leads, team leads
- Owner: Project owners

## Access Request Process
1. Submit issue with template
2. Manager approval required
3. Access granted with expiration
4. Quarterly review

## Offboarding
Access removed within 24 hours of departure
```

### 7. Protect Sensitive Branches
```yaml
main:
  Allowed to merge: Maintainers
  Allowed to push: No one (use MRs)
  
production:
  Allowed to merge: Owners only
  Allowed to push: No one
  
develop:
  Allowed to merge: Developers + Maintainers
  Allowed to push: Developers + Maintainers
```

### 8. Use Group Access Tokens
Instead of personal tokens:
- Create group access tokens
- Set appropriate scopes
- Set expiration dates
- Rotate regularly

## Member Management Commands

### List Project Members (API)
```bash
curl --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.example.com/api/v4/projects/:id/members"
```

### List Group Members (API)
```bash
curl --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.example.com/api/v4/groups/:id/members"
```

### Bulk Add Members (Script Example)
```bash
#!/bin/bash
PROJECT_ID=123
ACCESS_TOKEN="your-token"

# CSV format: user_id,access_level
while IFS=, read -r user_id access_level; do
  curl --request POST \
    --header "PRIVATE-TOKEN: $ACCESS_TOKEN" \
    --data "user_id=$user_id&access_level=$access_level" \
    "https://gitlab.example.com/api/v4/projects/$PROJECT_ID/members"
done < members.csv
```

## References
- [GitLab Permissions Documentation](https://docs.gitlab.com/ee/user/permissions.html)
- [GitLab Members API](https://docs.gitlab.com/ee/api/members.html)
- [Code Owners Documentation](https://docs.gitlab.com/ee/user/project/code_owners.html)
- [Approval Rules](https://docs.gitlab.com/ee/user/project/merge_requests/approvals/)
