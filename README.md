# CI Catalog

Repository to store all GitHub Actions to be re-used by all other repositories.  
As such they all trigger on `workflow_call`.  
If building a pipeline it is encouraged to call them with `if` conditions to control when they execute.

You can read more about re-usable workflows in [GitHub's Documentation](https://docs.github.com/en/actions/sharing-automations/reusing-workflows).

**Notes:**
- External workflows must be called using the keyword `uses` with the following syntax `<owner>/<repository>/<path_to_file>@<ref>`, where `ref` can be a commit SHA, a release tag or a branch name.  
Example usage:
```
    uses: n-alonso/GH_ACTIONS-ci-catalog/.github/workflows/branch-names.yml@v1.0.0
```
- All workflows that require secrets need to include the necessary permissions to access them when calling them:
```
    permissions:
      contents: read
```
- All workflows that require to create any resources in Github need to include the necessary permissions when calling them:
```
    permissions:
      contents: write
```

## Available Workflows

### Branch Names

Checks that the branches follow an appropriate pattern that adheres to [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) development convention.  
Pattern: `<prefix>/<jira_ticket>-<description>`  
Prefixes allowed: `feature|release|bugfix|hotfix|support`.  
Jira ticket: Has to conform to Jira tickets naming convention (Ie. `<project-name>-<ticket_number>`).  
**Notes:**  
- **Should be called on push**  

Example usage:
```
  check_branch:
    if: github.event_name == 'push'
    uses: n-alonso/GH_ACTIONS-ci-catalog/.github/workflows/branch-names.yml@main
```

### Commit Names

Checks that the commit messages follow an appropriate pattern that adheres to [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) development convention.  
Pattern: `<type>[!]: <message>`.  
Types allowed: `feat|fix|docs|style|refactor|test|chore|build|ci|perf`.  
**Notes:**  
- **Should be called on pull requests**  
- **The use of parenthesis to define an optional scope is not supported**  

Example usage:
```
  check_commits:
    if: github.event_name == 'pull_request'
    uses: n-alonso/GH_ACTIONS-ci-catalog/.github/workflows/commit-names.yml@main
```

### Docker Build

Builds and pushes a Docker Image.  
Inputs:  
- environment   

GitHub Environment Variables:
- SERVICE_USER
- DOCKER_REGISTRY_URL  

GitHub Environment Secrets:  
- SERVICE_PASSWORD  

Example usage:
```
  build_dev:
    if: github.event_name == 'pull_request' && github.base_ref == 'develop'
    uses: n-alonso/GH_ACTIONS-ci-catalog/.github/workflows/docker-build.yml@main
    permissions:
      contents: read
    with:
      environment: develop
    secrets:
      SERVICE_PASSWORD: ${{ secrets.SERVICE_PASSWORD }}
```

### OpenShift Deploy

Deploys the application to OpenShift.  
Inputs:  
- environment   

GitHub Environment Variables:
- SERVICE_USER
- OPENSHIFT_URL
- OPENSHIFT_ROUTE
- DOCKER_REGISTRY_URL  

GitHub Environment Secrets:  
- SERVICE_PASSWORD  

Example usage:
```
  deploy_dev:
    if: github.event_name == 'pull_request' && github.base_ref == 'develop'
    uses: n-alonso/GH_ACTIONS-ci-catalog/.github/workflows/openshift-deploy.yml@main
    needs: build_dev
    permissions:
      contents: read
    with:
      environment: develop
    secrets:
      SERVICE_PASSWORD: ${{ secrets.SERVICE_PASSWORD }}
```

### Create Release

It creates a release by catching a new tag being pushed and creating a differential of the commits between this and the previous tag found in the commit history.
**Notes:**
- **It MUST be called on a `push` for a `git tag`**
- **The repository MUST contain at least 2 tags (including the current one being pushed)**

Example usage:
```
  create_release:
    if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/')
    uses: n-alonso/GH_ACTIONS-ci-catalog/.github/workflows/create-release.yml@main
```
