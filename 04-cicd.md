# CI/CD

## Overview
GitLab CI/CD is a powerful continuous integration and deployment system built into GitLab. It uses `.gitlab-ci.yml` file to define pipelines that automatically build, test, and deploy your code.

## Table of Contents
1. [Predefined CI/CD Variables](#predefined-cicd-variables)
2. [CI/CD YAML Examples](#cicd-yaml-examples)
3. [Best Practices](#best-practices)

## Predefined CI/CD Variables

GitLab provides numerous predefined environment variables that are automatically available in your CI/CD pipelines.

### Complete Variables Reference Table

| Variable | Usage | Example Value | Description |
|----------|-------|---------------|-------------|
| `CI` | Check if running in CI | `true` | Available in all CI jobs |
| `CI_API_V4_URL` | GitLab API v4 URL | `https://gitlab.example.com/api/v4` | Base URL for API calls |
| `CI_BUILDS_DIR` | Build execution directory | `/builds` | Top-level directory for all builds |
| `CI_COMMIT_AUTHOR` | Commit author | `John Doe <john@example.com>` | Author email from git config |
| `CI_COMMIT_BEFORE_SHA` | Previous commit SHA | `1ecfd275763eff1d6b4844ea3168962458c9f27a` | SHA before changes |
| `CI_COMMIT_BRANCH` | Branch name | `main`, `feature/new-feature` | Branch being built (not for tags/MRs) |
| `CI_COMMIT_DESCRIPTION` | Commit description | `Add new feature\n\nDetailed description` | Full commit message without first line |
| `CI_COMMIT_MESSAGE` | Full commit message | `Add new feature` | Complete commit message |
| `CI_COMMIT_REF_NAME` | Branch or tag name | `main`, `v1.0.0` | Reference name being built |
| `CI_COMMIT_REF_PROTECTED` | Protected ref indicator | `true`, `false` | Whether branch/tag is protected |
| `CI_COMMIT_REF_SLUG` | Sanitized ref name | `main`, `feature-new-feature` | Lowercase, max 63 chars, slug-safe |
| `CI_COMMIT_SHA` | Commit SHA | `1ecfd275763eff1d6b4844ea3168962458c9f27a` | Full 40-character SHA |
| `CI_COMMIT_SHORT_SHA` | Short commit SHA | `1ecfd275` | First 8 characters of SHA |
| `CI_COMMIT_TAG` | Tag name | `v1.0.0`, `release-2.1` | Only available for tag pipelines |
| `CI_COMMIT_TAG_MESSAGE` | Tag message | `Version 1.0.0 release` | Annotated tag message |
| `CI_COMMIT_TIMESTAMP` | Commit timestamp | `2024-01-15T10:30:45+00:00` | ISO 8601 format |
| `CI_COMMIT_TITLE` | Commit title | `Add new feature` | First line of commit message |
| `CI_CONCURRENT_ID` | Concurrent job ID | `1`, `2`, `3` | Unique ID for concurrent jobs |
| `CI_CONCURRENT_PROJECT_ID` | Concurrent project ID | `1`, `2`, `3` | Unique ID per concurrent project |
| `CI_CONFIG_PATH` | CI config file path | `.gitlab-ci.yml`, `ci/config.yml` | Path to CI configuration |
| `CI_DEBUG_TRACE` | Debug mode | `true`, `false` | Enable verbose logging |
| `CI_DEBUG_SERVICES` | Debug services | `true`, `false` | Enable service container debugging |
| `CI_DEFAULT_BRANCH` | Default branch name | `main`, `master` | Project's default branch |
| `CI_DEPENDENCY_PROXY_GROUP_IMAGE_PREFIX` | Dependency proxy prefix | `gitlab.example.com/group/dependency_proxy/containers` | Docker image proxy prefix |
| `CI_DEPENDENCY_PROXY_PASSWORD` | Proxy password | `token-value` | Password for dependency proxy |
| `CI_DEPENDENCY_PROXY_SERVER` | Proxy server | `gitlab.example.com:443` | Dependency proxy server URL |
| `CI_DEPENDENCY_PROXY_USER` | Proxy username | `gitlab-ci-token` | Username for dependency proxy |
| `CI_DEPLOY_FREEZE` | Deployment freeze | `true`, `false` | Whether in deployment freeze period |
| `CI_DEPLOY_PASSWORD` | Deploy password | `password-value` | Deprecated, use CI_DEPLOY_TOKEN |
| `CI_DEPLOY_USER` | Deploy username | `gitlab-ci-token` | Deprecated, use CI_DEPLOY_TOKEN |
| `CI_DEPLOY_TOKEN` | Deploy token | `token-value` | Deploy token for authentication |
| `CI_DISPOSABLE_ENVIRONMENT` | Disposable env flag | `true`, `false` | Whether environment is temporary |
| `CI_ENVIRONMENT_NAME` | Environment name | `production`, `staging` | Name of the environment |
| `CI_ENVIRONMENT_SLUG` | Environment slug | `production`, `staging-123` | Sanitized environment name |
| `CI_ENVIRONMENT_URL` | Environment URL | `https://app.example.com` | URL of the environment |
| `CI_ENVIRONMENT_ACTION` | Environment action | `start`, `stop`, `prepare` | Action being performed |
| `CI_ENVIRONMENT_TIER` | Environment tier | `production`, `staging`, `development` | Environment tier |
| `CI_EXTERNAL_PULL_REQUEST_IID` | External PR ID | `123` | Pull request IID from external repo |
| `CI_EXTERNAL_PULL_REQUEST_SOURCE_BRANCH_NAME` | External PR source | `feature-branch` | Source branch of external PR |
| `CI_EXTERNAL_PULL_REQUEST_SOURCE_BRANCH_SHA` | External PR SHA | `1ecfd275...` | SHA of external PR source branch |
| `CI_EXTERNAL_PULL_REQUEST_TARGET_BRANCH_NAME` | External PR target | `main` | Target branch of external PR |
| `CI_EXTERNAL_PULL_REQUEST_TARGET_BRANCH_SHA` | External PR target SHA | `1ecfd275...` | SHA of external PR target branch |
| `CI_HAS_OPEN_REQUIREMENTS` | Open requirements flag | `true`, `false` | Whether project has open requirements |
| `CI_JOB_ID` | Job ID | `1234` | Unique job identifier |
| `CI_JOB_IMAGE` | Job image | `ruby:2.7` | Docker image used for job |
| `CI_JOB_JWT` | Job JWT token | `eyJ0eXAiOiJKV1Qi...` | JWT for authenticating with services |
| `CI_JOB_JWT_V1` | Job JWT V1 | `eyJ0eXAiOiJKV1Qi...` | JWT token version 1 |
| `CI_JOB_JWT_V2` | Job JWT V2 | `eyJ0eXAiOiJKV1Qi...` | JWT token version 2 |
| `CI_JOB_MANUAL` | Manual job flag | `true`, `false` | Whether job was manually triggered |
| `CI_JOB_NAME` | Job name | `build`, `test`, `deploy` | Name defined in .gitlab-ci.yml |
| `CI_JOB_NAME_SLUG` | Job name slug | `build`, `test-unit` | Sanitized job name |
| `CI_JOB_STAGE` | Job stage | `build`, `test`, `deploy` | Stage the job belongs to |
| `CI_JOB_STATUS` | Job status | `success`, `failed`, `canceled` | Current job status |
| `CI_JOB_STARTED_AT` | Job start time | `2024-01-15T10:30:45+00:00` | ISO 8601 timestamp |
| `CI_JOB_TOKEN` | Job token | `token-value` | Token for Git operations and API calls |
| `CI_JOB_URL` | Job URL | `https://gitlab.example.com/group/project/-/jobs/1234` | Direct link to job |
| `CI_KUBERNETES_ACTIVE` | K8s active flag | `true`, `false` | Whether Kubernetes integration is active |
| `CI_MERGE_REQUEST_APPROVED` | MR approved flag | `true`, `false` | Whether MR has required approvals |
| `CI_MERGE_REQUEST_ASSIGNEES` | MR assignees | `user1,user2` | Comma-separated assignee usernames |
| `CI_MERGE_REQUEST_DIFF_BASE_SHA` | MR diff base SHA | `1ecfd275...` | Base SHA for MR diff |
| `CI_MERGE_REQUEST_DIFF_ID` | MR diff ID | `123` | ID of the MR diff |
| `CI_MERGE_REQUEST_EVENT_TYPE` | MR event type | `detached`, `merged_result` | Type of MR pipeline |
| `CI_MERGE_REQUEST_ID` | MR ID | `123` | Internal ID of merge request |
| `CI_MERGE_REQUEST_IID` | MR IID | `42` | Project-level IID of MR |
| `CI_MERGE_REQUEST_LABELS` | MR labels | `bug,priority::high` | Comma-separated labels |
| `CI_MERGE_REQUEST_MILESTONE` | MR milestone | `v1.0` | Milestone assigned to MR |
| `CI_MERGE_REQUEST_PROJECT_ID` | MR project ID | `456` | Project ID of MR |
| `CI_MERGE_REQUEST_PROJECT_PATH` | MR project path | `group/project` | Full path of MR project |
| `CI_MERGE_REQUEST_PROJECT_URL` | MR project URL | `https://gitlab.example.com/group/project` | URL of MR project |
| `CI_MERGE_REQUEST_REF_PATH` | MR ref path | `refs/merge-requests/42/head` | Git ref path for MR |
| `CI_MERGE_REQUEST_SOURCE_BRANCH_NAME` | MR source branch | `feature-branch` | Source branch of MR |
| `CI_MERGE_REQUEST_SOURCE_BRANCH_SHA` | MR source SHA | `1ecfd275...` | SHA of MR source branch |
| `CI_MERGE_REQUEST_SOURCE_PROJECT_ID` | MR source project ID | `789` | ID of source project |
| `CI_MERGE_REQUEST_SOURCE_PROJECT_PATH` | MR source project path | `user/fork` | Path of source project |
| `CI_MERGE_REQUEST_SOURCE_PROJECT_URL` | MR source project URL | `https://gitlab.example.com/user/fork` | URL of source project |
| `CI_MERGE_REQUEST_TARGET_BRANCH_NAME` | MR target branch | `main` | Target branch of MR |
| `CI_MERGE_REQUEST_TARGET_BRANCH_SHA` | MR target SHA | `1ecfd275...` | SHA of MR target branch |
| `CI_MERGE_REQUEST_TITLE` | MR title | `Add new feature` | Title of the merge request |
| `CI_NODE_INDEX` | Node index | `1`, `2`, `3` | Index for parallel jobs |
| `CI_NODE_TOTAL` | Total nodes | `5` | Total number of parallel jobs |
| `CI_OPEN_MERGE_REQUESTS` | Open MRs | `branch-name:123,other:456` | Comma-separated branch:IID pairs |
| `CI_PAGES_DOMAIN` | Pages domain | `gitlab.io` | Domain for GitLab Pages |
| `CI_PAGES_URL` | Pages URL | `https://group.gitlab.io/project` | URL of GitLab Pages site |
| `CI_PIPELINE_CREATED_AT` | Pipeline creation time | `2024-01-15T10:00:00+00:00` | ISO 8601 timestamp |
| `CI_PIPELINE_ID` | Pipeline ID | `1000` | Unique pipeline identifier |
| `CI_PIPELINE_IID` | Pipeline IID | `50` | Project-level pipeline IID |
| `CI_PIPELINE_NAME` | Pipeline name | `Custom Pipeline Name` | Custom name for pipeline |
| `CI_PIPELINE_SOURCE` | Pipeline trigger source | `push`, `web`, `trigger`, `schedule`, `api` | How pipeline was triggered |
| `CI_PIPELINE_TRIGGERED` | Triggered flag | `true`, `false` | Whether pipeline was triggered |
| `CI_PIPELINE_URL` | Pipeline URL | `https://gitlab.example.com/group/project/-/pipelines/1000` | Direct link to pipeline |
| `CI_PROJECT_CLASSIFICATION_LABEL` | Project classification | `confidential`, `internal` | Security classification |
| `CI_PROJECT_CONFIG_PATH` | Project config path | `.gitlab-ci.yml` | Custom CI config path |
| `CI_PROJECT_DIR` | Project directory | `/builds/group/project` | Full path to project clone |
| `CI_PROJECT_ID` | Project ID | `42` | Unique project identifier |
| `CI_PROJECT_NAME` | Project name | `my-project` | Project name without namespace |
| `CI_PROJECT_NAMESPACE` | Project namespace | `group/subgroup` | Project's namespace path |
| `CI_PROJECT_NAMESPACE_ID` | Namespace ID | `123` | ID of project's namespace |
| `CI_PROJECT_PATH` | Project full path | `group/subgroup/project` | Full path including namespace |
| `CI_PROJECT_PATH_SLUG` | Project path slug | `group-subgroup-project` | Sanitized project path |
| `CI_PROJECT_REPOSITORY_LANGUAGES` | Project languages | `ruby,javascript,python` | Detected programming languages |
| `CI_PROJECT_ROOT_NAMESPACE` | Root namespace | `group` | Top-level group name |
| `CI_PROJECT_TITLE` | Project title | `My Project` | Human-readable project title |
| `CI_PROJECT_URL` | Project URL | `https://gitlab.example.com/group/project` | HTTP(S) URL to project |
| `CI_PROJECT_VISIBILITY` | Project visibility | `private`, `internal`, `public` | Project visibility level |
| `CI_REGISTRY` | Container registry | `registry.gitlab.com` | GitLab Container Registry URL |
| `CI_REGISTRY_IMAGE` | Registry image path | `registry.gitlab.com/group/project` | Full image path in registry |
| `CI_REGISTRY_PASSWORD` | Registry password | `token-value` | Password for container registry |
| `CI_REGISTRY_USER` | Registry username | `gitlab-ci-token` | Username for container registry |
| `CI_REPOSITORY_URL` | Repository URL | `https://gitlab-ci-token:[MASKED]@gitlab.example.com/group/project.git` | Git repository URL with auth |
| `CI_RUNNER_DESCRIPTION` | Runner description | `my-runner` | Description of the runner |
| `CI_RUNNER_EXECUTABLE_ARCH` | Runner architecture | `linux/amd64`, `linux/arm64` | Runner's OS/architecture |
| `CI_RUNNER_ID` | Runner ID | `1234` | Unique runner identifier |
| `CI_RUNNER_REVISION` | Runner revision | `436955cb` | GitLab Runner revision |
| `CI_RUNNER_SHORT_TOKEN` | Runner short token | `a1b2c3d4` | First 8 characters of runner token |
| `CI_RUNNER_TAGS` | Runner tags | `docker,linux,aws` | Comma-separated runner tags |
| `CI_RUNNER_VERSION` | Runner version | `14.10.1` | GitLab Runner version |
| `CI_SERVER` | Server indicator | `yes` | Always `yes` in CI |
| `CI_SERVER_HOST` | Server hostname | `gitlab.example.com` | GitLab server hostname |
| `CI_SERVER_NAME` | Server name | `GitLab` | Name of CI server (always GitLab) |
| `CI_SERVER_PORT` | Server port | `443`, `80` | GitLab server port |
| `CI_SERVER_PROTOCOL` | Server protocol | `https`, `http` | GitLab server protocol |
| `CI_SERVER_REVISION` | Server revision | `5d33c2b` | GitLab revision |
| `CI_SERVER_SHELL_SSH_HOST` | SSH host | `gitlab.example.com` | Host for Git SSH operations |
| `CI_SERVER_SHELL_SSH_PORT` | SSH port | `22` | Port for Git SSH operations |
| `CI_SERVER_TLS_CA_FILE` | TLS CA file path | `/path/to/ca.crt` | Path to TLS CA certificate |
| `CI_SERVER_TLS_CERT_FILE` | TLS cert file path | `/path/to/cert.crt` | Path to TLS certificate |
| `CI_SERVER_TLS_KEY_FILE` | TLS key file path | `/path/to/key.key` | Path to TLS key |
| `CI_SERVER_URL` | Server URL | `https://gitlab.example.com` | Full GitLab server URL |
| `CI_SERVER_VERSION` | Server version | `15.10.0` | GitLab version |
| `CI_SERVER_VERSION_MAJOR` | Major version | `15` | Major version number |
| `CI_SERVER_VERSION_MINOR` | Minor version | `10` | Minor version number |
| `CI_SERVER_VERSION_PATCH` | Patch version | `0` | Patch version number |
| `CI_SHARED_ENVIRONMENT` | Shared environment | `true`, `false` | Whether environment is shared |
| `CI_TEMPLATE_REGISTRY_HOST` | Template registry | `registry.gitlab.com` | Host for CI templates |
| `GITLAB_CI` | GitLab CI indicator | `true` | Always `true` in GitLab CI |
| `GITLAB_FEATURES` | Available features | `audit_events,burndown_charts` | Comma-separated feature flags |
| `GITLAB_USER_EMAIL` | User email | `user@example.com` | Email of user who triggered pipeline |
| `GITLAB_USER_ID` | User ID | `42` | ID of user who triggered pipeline |
| `GITLAB_USER_LOGIN` | User login | `username` | Username who triggered pipeline |
| `GITLAB_USER_NAME` | User name | `John Doe` | Name of user who triggered pipeline |

### Common Usage Patterns

#### Build Docker Images
```yaml
build:
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA $CI_REGISTRY_IMAGE:latest
```

#### Dynamic Environment URLs
```yaml
deploy:
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_COMMIT_REF_SLUG.example.com
```

#### Conditional Job Execution
```yaml
deploy_prod:
  script:
    - echo "Deploying to production"
  only:
    variables:
      - $CI_COMMIT_BRANCH == "main"
      - $CI_COMMIT_TAG =~ /^v.*/
```

## CI/CD YAML Examples

### Basic Pipeline Structure

```yaml
# Define stages
stages:
  - build
  - test
  - deploy

# Global variables
variables:
  DOCKER_DRIVER: overlay2
  APP_NAME: my-application

# Build job
build:
  stage: build
  image: node:18
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
  only:
    - branches

# Test job
test:
  stage: test
  image: node:18
  script:
    - npm install
    - npm test
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Deploy job
deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying application..."
    - ./deploy.sh
  environment:
    name: production
    url: https://example.com
  only:
    - main
```

### Docker Build and Push

```yaml
stages:
  - build
  - push

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  DOCKER_IMAGE_LATEST: $CI_REGISTRY_IMAGE:latest

before_script:
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

build_image:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker tag $DOCKER_IMAGE $DOCKER_IMAGE_LATEST
    - docker push $DOCKER_IMAGE
    - docker push $DOCKER_IMAGE_LATEST
  only:
    - main
    - tags

build_image_feature:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  except:
    - main
    - tags
```

### Multi-Environment Deployment

```yaml
stages:
  - build
  - test
  - deploy_dev
  - deploy_staging
  - deploy_production

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

test:
  stage: test
  script:
    - npm test

deploy_dev:
  stage: deploy_dev
  script:
    - ./deploy.sh dev
  environment:
    name: development
    url: https://dev.example.com
    on_stop: stop_dev
  only:
    - develop

stop_dev:
  stage: deploy_dev
  script:
    - ./cleanup.sh dev
  environment:
    name: development
    action: stop
  when: manual

deploy_staging:
  stage: deploy_staging
  script:
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main
  when: manual

deploy_production:
  stage: deploy_production
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
  only:
    - tags
  when: manual
```

### Parallel Testing

```yaml
test:
  stage: test
  parallel: 5
  script:
    - |
      # Split tests across parallel jobs
      npm test -- --shard=$((CI_NODE_INDEX))/$((CI_NODE_TOTAL))
  artifacts:
    reports:
      junit: junit-$CI_NODE_INDEX.xml

test_matrix:
  stage: test
  parallel:
    matrix:
      - NODE_VERSION: ['16', '18', '20']
        OS: ['ubuntu', 'alpine']
  image: node:${NODE_VERSION}-${OS}
  script:
    - npm test
```

### Caching and Artifacts

```yaml
.node_cache:
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
    policy: pull

build:
  extends: .node_cache
  stage: build
  script:
    - npm ci
    - npm run build
  cache:
    policy: pull-push
  artifacts:
    paths:
      - dist/
      - node_modules/
    expire_in: 1 day

test:
  extends: .node_cache
  stage: test
  needs:
    - build
  script:
    - npm test
  dependencies:
    - build
```

### Merge Request Pipelines

```yaml
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_TAG

lint:
  stage: test
  script:
    - npm run lint
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      changes:
        - "**/*.js"
        - "**/*.vue"

unit_test:
  stage: test
  script:
    - npm test
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

integration_test:
  stage: test
  script:
    - npm run test:integration
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_TAG
```

### Kubernetes Deployment

```yaml
stages:
  - build
  - deploy

variables:
  KUBECTL_VERSION: "1.28.0"
  NAMESPACE: production

.kubectl_setup: &kubectl_setup
  - curl -LO "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/amd64/kubectl"
  - chmod +x kubectl
  - mv kubectl /usr/local/bin/
  - kubectl version --client

deploy_k8s:
  stage: deploy
  image: alpine:latest
  before_script:
    - *kubectl_setup
    - echo $KUBECONFIG_BASE64 | base64 -d > /tmp/kubeconfig
    - export KUBECONFIG=/tmp/kubeconfig
  script:
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA -n $NAMESPACE
    - kubectl rollout status deployment/myapp -n $NAMESPACE
  environment:
    name: production
    kubernetes:
      namespace: $NAMESPACE
  only:
    - main
```

### Scheduled Pipelines

```yaml
nightly_build:
  script:
    - npm run build:full
    - npm run test:e2e
  only:
    - schedules
  variables:
    SCHEDULE_TYPE: nightly

weekly_cleanup:
  script:
    - ./cleanup_old_resources.sh
  only:
    variables:
      - $SCHEDULE_TYPE == "weekly"
```

### Security Scanning

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml

stages:
  - test
  - security

sast:
  stage: security

dependency_scanning:
  stage: security

secret_detection:
  stage: security

container_scanning:
  stage: security
  variables:
    CS_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
```

### Complex Multi-Project Pipeline

```yaml
trigger_downstream:
  stage: deploy
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend
  variables:
    UPSTREAM_VERSION: $CI_COMMIT_TAG
    DEPLOY_ENV: production

trigger_child_pipeline:
  stage: test
  trigger:
    include:
      - local: .gitlab-ci-child.yml
    strategy: depend

wait_for_parent:
  stage: build
  needs:
    - pipeline: $PARENT_PIPELINE_ID
      job: parent_build
```

### Dynamic Child Pipelines

```yaml
# Parent pipeline
generate_config:
  stage: generate
  script:
    - python generate_child_pipeline.py > child-pipeline.yml
  artifacts:
    paths:
      - child-pipeline.yml

trigger_child:
  stage: trigger
  trigger:
    include:
      - artifact: child-pipeline.yml
        job: generate_config
    strategy: depend
```

### Advanced Rules and Conditions

```yaml
build:
  script:
    - npm run build
  rules:
    # Run on MR pipelines
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    # Run on main branch
    - if: $CI_COMMIT_BRANCH == "main"
    # Run on tags starting with v
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
    # Run if specific files changed
    - changes:
        - src/**/*
        - package.json
      when: always
    # Don't run for scheduled pipelines
    - if: $CI_PIPELINE_SOURCE == "schedule"
      when: never

deploy:
  script:
    - ./deploy.sh
  rules:
    # Only deploy from protected branches
    - if: $CI_COMMIT_REF_PROTECTED == "true"
      when: manual
    # Allow manual deployment on any branch if needed
    - when: manual
      allow_failure: true
```

### Retry and Timeout Configuration

```yaml
flaky_test:
  script:
    - npm run test:flaky
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
      - script_failure

long_running_job:
  script:
    - npm run build:full
  timeout: 2h

quick_job:
  script:
    - npm run lint
  timeout: 5m
```

## Best Practices

### 1. Use Includes for Reusability

```yaml
include:
  - local: '/templates/.build.yml'
  - project: 'group/ci-templates'
    file: '/templates/docker.yml'
  - remote: 'https://example.com/ci/template.yml'
```

### 2. Optimize with Needs (DAG)

```yaml
build:
  stage: build
  script: npm run build

test:unit:
  stage: test
  needs: [build]
  script: npm run test:unit

test:integration:
  stage: test
  needs: [build]
  script: npm run test:integration

deploy:
  stage: deploy
  needs: ["test:unit", "test:integration"]
  script: ./deploy.sh
```

### 3. Use Anchors for DRY

```yaml
.deploy_template: &deploy_template
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST $DEPLOY_WEBHOOK

deploy_dev:
  <<: *deploy_template
  environment: development
  variables:
    DEPLOY_WEBHOOK: $DEV_WEBHOOK

deploy_prod:
  <<: *deploy_template
  environment: production
  variables:
    DEPLOY_WEBHOOK: $PROD_WEBHOOK
```

### 4. Secure Secrets

```yaml
# Use masked and protected variables
deploy:
  script:
    - echo "API_KEY is masked in logs"
    - deploy --api-key $API_KEY
  only:
    - main
```

### 5. Pipeline Efficiency

```yaml
# Use interruptible for MR pipelines
test:
  interruptible: true
  script: npm test

# Use resource_group for sequential deploys
deploy:
  resource_group: production
  script: ./deploy.sh
```

## References
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [GitLab CI/CD Variables](https://docs.gitlab.com/ee/ci/variables/)
- [GitLab CI/CD YAML Reference](https://docs.gitlab.com/ee/ci/yaml/)
- [GitLab CI/CD Examples](https://docs.gitlab.com/ee/ci/examples/)
