# Creating Repos and Projects

## Overview
In GitLab, repositories are organized within projects, and projects are organized within groups. Understanding this hierarchy is essential for effective project management.

## Project Hierarchy

```
GitLab Instance
├── Groups (optional)
│   ├── Subgroups (optional)
│   │   └── Projects
│   └── Projects
└── Projects (personal namespace)
```

## Creating a New Project

### Via Web Interface

1. **Navigate to Projects**
   - Click the `+` icon in the top navigation bar
   - Select "New project/repository"

2. **Choose Creation Method**
   - **Create blank project**: Start from scratch
   - **Create from template**: Use predefined templates
   - **Import project**: Import from GitHub, Bitbucket, etc.
   - **Run CI/CD for external repository**: Connect external repos

3. **Configure Project Settings**
   - **Project name**: Display name for the project
   - **Project slug**: URL-friendly name (auto-generated)
   - **Project URL**: Choose namespace (user or group)
   - **Visibility level**: Private, Internal, or Public
   - **Initialize repository**: Add README, .gitignore, LICENSE

4. **Click "Create project"**

### Via API

```bash
# Create a new project
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "My Project",
    "description": "Project description",
    "visibility": "private",
    "initialize_with_readme": true
  }' \
  "https://gitlab.example.com/api/v4/projects"
```

### Via GitLab CLI

```bash
# Install glab CLI
brew install glab  # macOS
# or download from: https://gitlab.com/gitlab-org/cli

# Authenticate
glab auth login

# Create project
glab repo create my-project --public --description "My project description"
```

## Project Visibility Levels

### Private
- Only project members can access
- Not visible in public project lists
- **Best for**: Internal company projects, sensitive code

### Internal
- Any authenticated user can access
- Visible to logged-in users only
- **Best for**: Company-wide projects on private GitLab instance

### Public
- Anyone can access without authentication
- Visible in public project lists
- Indexed by search engines
- **Best for**: Open source projects, public documentation

## Project Templates

GitLab provides built-in templates for common project types:

### Available Templates
- **Ruby on Rails**: Rails application with GitLab CI
- **Spring**: Spring Boot application
- **Express**: Node.js Express application
- **Django**: Python Django application
- **Go Micro**: Go microservice
- **Netlify Jekyll**: Static site with Jekyll
- **Hexo**: Static blog with Hexo
- **Hugo**: Static site with Hugo
- **Pages/Plain HTML**: Simple static HTML pages
- **GitBook**: GitBook documentation

### Using Templates

```bash
# Via API
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "name=my-rails-app&template_name=rails" \
  "https://gitlab.example.com/api/v4/projects"
```

## Creating Groups

### Purpose of Groups
- Organize related projects together
- Manage permissions at group level
- Share CI/CD variables across projects
- Aggregate issues, merge requests, and milestones

### Create a Group

1. **Via Web Interface**
   - Click `+` → "New group/subgroup"
   - Enter group name and slug
   - Set visibility level
   - Click "Create group"

2. **Via API**
```bash
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "name=MyGroup&path=my-group&visibility=private" \
  "https://gitlab.example.com/api/v4/groups"
```

### Subgroups
Create hierarchical structure for better organization:
- Maximum depth: 20 levels
- Inherit permissions from parent groups
- Can have their own projects and subgroups

## Importing Projects

### From GitHub

1. **Via Web Interface**
   - New project → "Import project" → "GitHub"
   - Authenticate with GitHub
   - Select repositories to import
   - Map to GitLab namespace

2. **Via API**
```bash
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "personal_access_token=<github_token>&repo_id=123456&target_namespace=my-group" \
  "https://gitlab.example.com/api/v4/import/github"
```

### From Bitbucket

```bash
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  --data "bitbucket_username=user&bitbucket_app_password=pass&repo_slug=repo&target_namespace=group" \
  "https://gitlab.example.com/api/v4/import/bitbucket_server"
```

### From Git Repository (URL)

1. New project → "Import project" → "Repository by URL"
2. Enter Git repository URL
3. Configure project settings
4. Click "Create project"

## Project Configuration

### General Settings

```
Settings → General
```

**Key Settings:**
- **Project name and description**: Update anytime
- **Project avatar**: Upload project icon
- **Topics**: Add tags for discoverability
- **Visibility**: Change visibility level
- **Transfer project**: Move to different namespace
- **Archive project**: Make read-only
- **Delete project**: Permanent deletion (can be restored by admin)

### Repository Settings

```
Settings → Repository
```

**Protected Branches:**
```yaml
# Protect master/main branch
Branch: main
Allowed to merge: Maintainers
Allowed to push: No one
Allowed to force push: No
```

**Protected Tags:**
```yaml
# Protect release tags
Tag: v*
Allowed to create: Maintainers
```

**Push Rules (Premium/Ultimate):**
- Reject unsigned commits
- Reject commits by committers not verified
- Reject commits that aren't DCO certified
- Branch name pattern
- Commit message pattern
- File name pattern

### Repository Mirroring

#### Pull Mirroring
Keep GitLab repository in sync with external repository:

```bash
# Configure via Settings → Repository → Mirroring repositories
# Add:
# Git repository URL: https://github.com/user/repo.git
# Mirror direction: Pull
# Authentication method: Password
# Password: <access_token>
# Update frequency: Every 5 minutes
```

#### Push Mirroring
Push changes from GitLab to external repository:

```bash
# Configure via Settings → Repository → Mirroring repositories
# Add:
# Git repository URL: https://github.com/user/repo.git
# Mirror direction: Push
# Authentication method: SSH public key
# Only mirror protected branches: Enabled
```

## Project Access Tokens

Create tokens for automation and CI/CD:

```
Settings → Access Tokens
```

**Configure:**
- Token name
- Expiration date
- Select scopes: api, read_repository, write_repository, etc.
- Select role: Developer, Maintainer

**Usage:**
```bash
git clone https://project-token-name:<token>@gitlab.example.com/group/project.git
```

## Deploy Keys

Add SSH keys for read-only or read-write repository access:

```
Settings → Repository → Deploy Keys
```

**Generate deploy key:**
```bash
ssh-keygen -t ed25519 -C "deploy-key-name"
```

**Add to GitLab:**
- Copy public key content
- Add via Settings → Repository → Deploy Keys
- Select "Write access" if needed

## Repository Housekeeping

### Repository Size

```
Settings → General → Repository size
```

- View repository size
- Cleanup: Remove unreferenced objects
- Remove cached artifacts

### Repository Maintenance

```bash
# Via API - Trigger repository housekeeping
curl --request POST \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.example.com/api/v4/projects/:id/housekeeping"
```

## Best Practices

### 1. Naming Conventions
```
✅ Good:
- my-awesome-project
- web-api-service
- mobile-app-ios

❌ Avoid:
- Project1
- test
- asdf
```

### 2. Project Structure
```
project-root/
├── .gitlab-ci.yml       # CI/CD configuration
├── .gitignore           # Git ignore rules
├── README.md            # Project documentation
├── LICENSE              # License file
├── CONTRIBUTING.md      # Contribution guidelines
├── CHANGELOG.md         # Version history
├── docs/                # Additional documentation
├── src/                 # Source code
├── tests/               # Test files
└── scripts/             # Utility scripts
```

### 3. README Template
```markdown
# Project Name

Brief description of the project.

## Features
- Feature 1
- Feature 2

## Installation
\`\`\`bash
npm install
\`\`\`

## Usage
\`\`\`bash
npm start
\`\`\`

## Contributing
See [CONTRIBUTING.md](CONTRIBUTING.md)

## License
This project is licensed under MIT License.
```

### 4. Use Labels
Create consistent labels across projects:
- Priority: P1-Critical, P2-High, P3-Medium, P4-Low
- Status: To Do, In Progress, Review, Done
- Type: Bug, Feature, Enhancement, Documentation

### 5. Enable Branch Protection
- Protect main/master branch
- Require merge requests for changes
- Require approval before merging
- Enable merge request pipelines

### 6. Set Up CI/CD Early
Add `.gitlab-ci.yml` from the start:
```yaml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  script:
    - echo "Running tests..."
```

## Cloning and Working with Projects

### Clone via HTTPS
```bash
git clone https://gitlab.example.com/group/project.git
cd project
```

### Clone via SSH
```bash
# Add SSH key to GitLab first
git clone git@gitlab.example.com:group/project.git
cd project
```

### Clone specific branch
```bash
git clone -b develop https://gitlab.example.com/group/project.git
```

### Clone with depth (shallow clone)
```bash
git clone --depth 1 https://gitlab.example.com/group/project.git
```

## Project Export and Import

### Export Project
```
Settings → General → Advanced → Export project
```

- Includes: repository, wiki, issues, merge requests, labels, milestones
- Download archive when ready
- Archive expires after 24 hours

### Import Project
```
New project → Import project → GitLab export
```

- Upload exported archive
- Configure project settings
- Click "Import project"

## References
- [GitLab Projects Documentation](https://docs.gitlab.com/ee/user/project/)
- [GitLab API - Projects](https://docs.gitlab.com/ee/api/projects.html)
- [GitLab CLI Documentation](https://gitlab.com/gitlab-org/cli)
