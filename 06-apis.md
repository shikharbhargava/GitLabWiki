# GitLab APIs

## Overview
GitLab provides a comprehensive REST API and GraphQL API that allow you to interact with GitLab programmatically. These APIs enable automation, integration with external tools, and custom workflows.

## Table of Contents
1. [Authentication](#authentication)
2. [REST API](#rest-api)
3. [GraphQL API](#graphql-api)
4. [Common API Operations](#common-api-operations)
5. [Best Practices](#best-practices)

## Authentication

### Personal Access Tokens (PAT)

#### Create Personal Access Token
1. Navigate to **User Settings → Access Tokens**
2. Enter token name and expiration date
3. Select scopes:
   - `api`: Complete API access
   - `read_api`: Read-only API access
   - `read_repository`: Read repository content
   - `write_repository`: Write repository content
   - `read_registry`: Read container registry
   - `write_registry`: Write container registry
4. Click **Create personal access token**
5. Copy and save the token (shown only once)

#### Using PAT in API Calls

**Header Authentication:**
```bash
curl --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.example.com/api/v4/projects"
```

**Query Parameter:**
```bash
curl "https://gitlab.example.com/api/v4/projects?private_token=<your_access_token>"
```

### OAuth 2.0 Applications

```bash
# Authorization Code Flow
# Step 1: Redirect user to authorization URL
https://gitlab.example.com/oauth/authorize?client_id=APP_ID&redirect_uri=REDIRECT_URI&response_type=code&scope=api

# Step 2: Exchange code for access token
curl --data "client_id=APP_ID&client_secret=APP_SECRET&code=AUTH_CODE&grant_type=authorization_code&redirect_uri=REDIRECT_URI" \
  --request POST "https://gitlab.example.com/oauth/token"
```

### Project/Group Access Tokens

```bash
# Create project access token via API
curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "automation-token",
    "scopes": ["api", "read_repository"],
    "expires_at": "2025-01-01",
    "access_level": 40
  }' \
  "https://gitlab.example.com/api/v4/projects/:id/access_tokens"
```

### CI Job Token

```yaml
# Available automatically in CI/CD pipelines
script:
  - curl --header "JOB-TOKEN: $CI_JOB_TOKEN" \
      "https://gitlab.example.com/api/v4/projects/$CI_PROJECT_ID"
```

## REST API

### API Versioning
Current stable version: **v4**
Base URL: `https://gitlab.example.com/api/v4`

### Common HTTP Methods
- `GET`: Retrieve resources
- `POST`: Create resources
- `PUT`: Update resources (full update)
- `PATCH`: Update resources (partial update)
- `DELETE`: Remove resources

### Response Codes
- `200 OK`: Successful GET, PUT, PATCH, DELETE
- `201 Created`: Successful POST
- `204 No Content`: Successful DELETE (no body)
- `400 Bad Request`: Invalid request
- `401 Unauthorized`: Authentication failed
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource doesn't exist
- `422 Unprocessable Entity`: Validation failed
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

### Pagination

```bash
# Default: 20 items per page
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects?per_page=50&page=2"

# Response headers contain pagination info
# X-Total: 100
# X-Total-Pages: 2
# X-Per-Page: 50
# X-Page: 2
# X-Next-Page: 3
# X-Prev-Page: 1
```

### Rate Limiting

**Default Limits:**
- Authenticated users: 2,000 requests per minute
- Unauthenticated: 1,000 requests per minute

**Headers:**
```
RateLimit-Limit: 2000
RateLimit-Remaining: 1999
RateLimit-Reset: 1639065600
```

## Common API Operations

### Projects

#### List All Projects
```bash
# List accessible projects
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects"

# With filters
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects?visibility=public&order_by=created_at&sort=desc"
```

#### Get Single Project
```bash
# By project ID
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123"

# By URL-encoded path
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/group%2Fsubgroup%2Fproject"
```

#### Create Project
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "My New Project",
    "description": "Project description",
    "visibility": "private",
    "initialize_with_readme": true,
    "namespace_id": 123
  }' \
  "https://gitlab.example.com/api/v4/projects"
```

#### Update Project
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "description": "Updated description",
    "visibility": "internal"
  }' \
  "https://gitlab.example.com/api/v4/projects/123"
```

#### Delete Project
```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123"
```

### Repository Files

#### Get File Content
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/repository/files/path%2Fto%2Ffile.txt?ref=main"
```

#### Create File
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "branch": "main",
    "content": "File content here",
    "commit_message": "Add new file",
    "encoding": "text"
  }' \
  "https://gitlab.example.com/api/v4/projects/123/repository/files/path%2Fto%2Ffile.txt"
```

#### Update File
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "branch": "main",
    "content": "Updated content",
    "commit_message": "Update file"
  }' \
  "https://gitlab.example.com/api/v4/projects/123/repository/files/path%2Fto%2Ffile.txt"
```

#### Delete File
```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "branch": "main",
    "commit_message": "Delete file"
  }' \
  "https://gitlab.example.com/api/v4/projects/123/repository/files/path%2Fto%2Ffile.txt"
```

### Commits

#### List Commits
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/repository/commits?ref_name=main&per_page=10"
```

#### Get Single Commit
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/repository/commits/a3f5b8d"
```

#### Create Commit with Multiple Files
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "branch": "main",
    "commit_message": "Update multiple files",
    "actions": [
      {
        "action": "create",
        "file_path": "new_file.txt",
        "content": "New file content"
      },
      {
        "action": "update",
        "file_path": "existing_file.txt",
        "content": "Updated content"
      },
      {
        "action": "delete",
        "file_path": "old_file.txt"
      }
    ]
  }' \
  "https://gitlab.example.com/api/v4/projects/123/repository/commits"
```

### Branches

#### List Branches
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/repository/branches"
```

#### Create Branch
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --data "branch=feature-branch&ref=main" \
  "https://gitlab.example.com/api/v4/projects/123/repository/branches"
```

#### Delete Branch
```bash
curl --request DELETE --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/repository/branches/feature-branch"
```

### Merge Requests

#### List Merge Requests
```bash
# All project MRs
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/merge_requests?state=opened"

# All MRs assigned to me
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/merge_requests?scope=assigned_to_me&state=opened"
```

#### Create Merge Request
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "source_branch": "feature-branch",
    "target_branch": "main",
    "title": "Add new feature",
    "description": "This MR adds a new feature",
    "remove_source_branch": true,
    "assignee_id": 42
  }' \
  "https://gitlab.example.com/api/v4/projects/123/merge_requests"
```

#### Update Merge Request
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "title": "Updated title",
    "labels": "bug,priority::high"
  }' \
  "https://gitlab.example.com/api/v4/projects/123/merge_requests/42"
```

#### Merge a Merge Request
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  --data "merge_commit_message=Merge feature branch" \
  "https://gitlab.example.com/api/v4/projects/123/merge_requests/42/merge"
```

#### Approve Merge Request
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/merge_requests/42/approve"
```

### Issues

#### List Issues
```bash
# Project issues
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/issues?state=opened"

# All issues assigned to me
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/issues?scope=assigned_to_me"
```

#### Create Issue
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "title": "Bug: Login not working",
    "description": "Users cannot log in",
    "labels": "bug,priority::high",
    "assignee_ids": [42, 43]
  }' \
  "https://gitlab.example.com/api/v4/projects/123/issues"
```

#### Update Issue
```bash
curl --request PUT --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "state_event": "close",
    "labels": "bug,resolved"
  }' \
  "https://gitlab.example.com/api/v4/projects/123/issues/42"
```

### Pipelines

#### List Pipelines
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/pipelines?ref=main&status=success"
```

#### Get Pipeline Details
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/pipelines/1000"
```

#### Trigger Pipeline
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --form "ref=main" \
  --form "variables[DEPLOY_ENV]=production" \
  --form "variables[VERSION]=1.0.0" \
  "https://gitlab.example.com/api/v4/projects/123/pipeline"
```

#### Cancel Pipeline
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/pipelines/1000/cancel"
```

#### Retry Pipeline
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/pipelines/1000/retry"
```

### Jobs

#### List Pipeline Jobs
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/pipelines/1000/jobs"
```

#### Get Job Details
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/jobs/5000"
```

#### Get Job Logs
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/jobs/5000/trace"
```

#### Retry Job
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/123/jobs/5000/retry"
```

### Users

#### List Users
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/users"
```

#### Get Current User
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/user"
```

#### Get User by ID
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/users/42"
```

### Groups

#### List Groups
```bash
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/groups"
```

#### Create Group
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "My Group",
    "path": "my-group",
    "visibility": "private"
  }' \
  "https://gitlab.example.com/api/v4/groups"
```

## GraphQL API

### Endpoint
```
https://gitlab.example.com/api/graphql
```

### Authentication
```bash
curl "https://gitlab.example.com/api/graphql" \
  --header "Authorization: Bearer <your_access_token>" \
  --header "Content-Type: application/json" \
  --data '{"query": "query { currentUser { username } }"}'
```

### Basic Query Examples

#### Get Current User
```graphql
query {
  currentUser {
    id
    username
    name
    email
    avatarUrl
  }
}
```

#### Get Project Details
```graphql
query {
  project(fullPath: "group/project") {
    id
    name
    description
    visibility
    createdAt
    repository {
      rootRef
      tree {
        blobs {
          nodes {
            name
            path
          }
        }
      }
    }
  }
}
```

#### List Project Issues
```graphql
query {
  project(fullPath: "group/project") {
    issues(first: 10, state: opened) {
      nodes {
        iid
        title
        description
        state
        createdAt
        author {
          name
          username
        }
        labels {
          nodes {
            title
            color
          }
        }
      }
    }
  }
}
```

#### List Merge Requests
```graphql
query {
  project(fullPath: "group/project") {
    mergeRequests(first: 10, state: opened) {
      nodes {
        iid
        title
        description
        sourceBranch
        targetBranch
        state
        author {
          name
          username
        }
        approvedBy {
          nodes {
            name
          }
        }
      }
    }
  }
}
```

### Mutation Examples

#### Create Issue
```graphql
mutation {
  createIssue(input: {
    projectPath: "group/project"
    title: "New bug report"
    description: "Description of the bug"
    labels: ["bug", "priority::high"]
  }) {
    issue {
      iid
      title
      webUrl
    }
    errors
  }
}
```

#### Update Merge Request
```graphql
mutation {
  mergeRequestUpdate(input: {
    projectPath: "group/project"
    iid: "42"
    title: "Updated MR title"
    description: "Updated description"
  }) {
    mergeRequest {
      id
      title
    }
    errors
  }
}
```

## API Client Libraries

### Python (python-gitlab)
```python
import gitlab

# Create connection
gl = gitlab.Gitlab('https://gitlab.example.com', private_token='token')

# Get project
project = gl.projects.get('group/project')

# List issues
issues = project.issues.list(state='opened')
for issue in issues:
    print(f"#{issue.iid}: {issue.title}")

# Create merge request
mr = project.mergerequests.create({
    'source_branch': 'feature',
    'target_branch': 'main',
    'title': 'New feature',
})

# Trigger pipeline
pipeline = project.pipelines.create({'ref': 'main'})
```

### JavaScript/Node.js (@gitbeaker/node)
```javascript
const { Gitlab } = require('@gitbeaker/node');

const api = new Gitlab({
  host: 'https://gitlab.example.com',
  token: 'your-token',
});

// Get project
const project = await api.Projects.show('group/project');

// List issues
const issues = await api.Issues.all({ projectId: project.id });

// Create merge request
const mr = await api.MergeRequests.create(
  project.id,
  'feature-branch',
  'main',
  'New Feature'
);
```

### Go (go-gitlab)
```go
import "github.com/xanzy/go-gitlab"

git, err := gitlab.NewClient("your-token", gitlab.WithBaseURL("https://gitlab.example.com"))

// Get project
project, _, err := git.Projects.GetProject("group/project", nil)

// List issues
issues, _, err := git.Issues.ListProjectIssues(project.ID, &gitlab.ListProjectIssuesOptions{
    State: gitlab.String("opened"),
})

// Create MR
mr, _, err := git.MergeRequests.CreateMergeRequest(project.ID, &gitlab.CreateMergeRequestOptions{
    Title:        gitlab.String("New Feature"),
    SourceBranch: gitlab.String("feature"),
    TargetBranch: gitlab.String("main"),
})
```

### Ruby (gitlab gem)
```ruby
require 'gitlab'

Gitlab.configure do |config|
  config.endpoint = 'https://gitlab.example.com/api/v4'
  config.private_token = 'your-token'
end

# Get project
project = Gitlab.project('group/project')

# List issues
issues = Gitlab.issues(project.id, state: 'opened')

# Create MR
mr = Gitlab.create_merge_request(project.id, 'New Feature',
  source_branch: 'feature',
  target_branch: 'main'
)
```

## Webhooks

### Create Webhook
```bash
curl --request POST --header "PRIVATE-TOKEN: <token>" \
  --header "Content-Type: application/json" \
  --data '{
    "url": "https://example.com/webhook",
    "push_events": true,
    "merge_requests_events": true,
    "issues_events": true,
    "token": "secret-token",
    "enable_ssl_verification": true
  }' \
  "https://gitlab.example.com/api/v4/projects/123/hooks"
```

### Webhook Payload Example
```json
{
  "object_kind": "push",
  "event_name": "push",
  "before": "95790bf891e76fee5e1747ab589903a6a1f80f22",
  "after": "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
  "ref": "refs/heads/main",
  "user_name": "John Doe",
  "user_username": "jdoe",
  "project": {
    "id": 123,
    "name": "My Project",
    "namespace": "group",
    "web_url": "https://gitlab.example.com/group/project"
  },
  "commits": [
    {
      "id": "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
      "message": "Update README",
      "timestamp": "2024-01-15T10:30:00+00:00",
      "author": {
        "name": "John Doe",
        "email": "john@example.com"
      }
    }
  ]
}
```

## Best Practices

### 1. Use Appropriate Authentication
```python
# ✅ Good: Use project tokens for automation
headers = {"PRIVATE-TOKEN": os.environ['PROJECT_TOKEN']}

# ❌ Avoid: Hardcoding tokens
headers = {"PRIVATE-TOKEN": "hardcoded-token-123"}
```

### 2. Handle Rate Limiting
```python
import time
import requests

def api_call_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 60))
            time.sleep(retry_after)
            continue
        
        return response
    
    raise Exception("Max retries exceeded")
```

### 3. Use Pagination Efficiently
```python
def get_all_projects(gl):
    projects = []
    page = 1
    
    while True:
        batch = gl.projects.list(page=page, per_page=100)
        if not batch:
            break
        projects.extend(batch)
        page += 1
    
    return projects
```

### 4. Error Handling
```python
try:
    project = gl.projects.get('group/project')
except gitlab.exceptions.GitlabGetError as e:
    if e.response_code == 404:
        print("Project not found")
    elif e.response_code == 403:
        print("Access denied")
    else:
        print(f"Error: {e}")
```

### 5. Use Specific Scopes
```bash
# ✅ Good: Minimal required scope
# Create token with only read_api scope for read operations

# ❌ Avoid: Overly broad permissions
# Using 'api' scope when only reading data
```

### 6. Cache Responses
```python
import requests_cache

# Cache API responses for 5 minutes
requests_cache.install_cache('gitlab_cache', expire_after=300)
```

### 7. Batch Operations
```python
# ✅ Good: Batch multiple file operations in one commit
actions = [
    {'action': 'create', 'file_path': 'file1.txt', 'content': 'content1'},
    {'action': 'create', 'file_path': 'file2.txt', 'content': 'content2'},
]
project.commits.create({
    'branch': 'main',
    'commit_message': 'Add multiple files',
    'actions': actions
})

# ❌ Avoid: Multiple single-file commits
for file in files:
    project.commits.create({...})  # Multiple API calls
```

## Troubleshooting

### Common Issues

**401 Unauthorized:**
- Check token validity
- Verify token has required scopes
- Ensure token hasn't expired

**403 Forbidden:**
- Check user permissions on resource
- Verify token scopes include required permissions

**404 Not Found:**
- Verify resource exists
- Check project path encoding (use URL encoding)
- Ensure proper namespace in path

**429 Too Many Requests:**
- Implement rate limiting handling
- Use `Retry-After` header value
- Reduce request frequency

## References
- [GitLab REST API Documentation](https://docs.gitlab.com/ee/api/api_resources.html)
- [GitLab GraphQL API Documentation](https://docs.gitlab.com/ee/api/graphql/)
- [API Rate Limits](https://docs.gitlab.com/ee/user/admin_area/settings/rate_limits_on_raw_endpoints.html)
- [python-gitlab Documentation](https://python-gitlab.readthedocs.io/)
- [GitLab Webhooks](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html)
