# Public Private CI-CD

## How Public-Private Repo Combination Enables Deployment

This repository demonstrates a pattern where a **public repository** is used to orchestrate CI/CD workflows that deploy code from **private repositories**. The public repo contains the workflow definitions, while the actual application code remains secure in private repos.

To enable this, the workflow in the public repo uses a **Personal Access Token (PAT)** with appropriate permissions. This token is stored as a GitHub secret (e.g., `GET_CODE_FROM_PRIVATE_REPO`) and allows the workflow to:

- Authenticate and securely access the private repository.
- Check out code from the private repo using the `actions/checkout` action and the `repository` parameter.
- Build, test, and deploy the application as defined in the workflow steps.

This approach allows teams to keep sensitive code private while leveraging the visibility, auditability, and modularity of public CI/CD orchestration. The PAT must have at least `repo` scope to allow read access to private repositories and should be managed securely.

**Note:** Never expose the PAT directly in workflow files. Always use GitHub secrets to reference sensitive credentials.

## Overview

This repository acts as a public CI/CD orchestrator. The workflows here securely fetch code from private repos, build, and deploy it to cloud infrastructure (Kubernetes, Azure, etc.).

## How Private Repos Are Deployed

- **Manual Trigger:** Workflows are triggered manually (`workflow_dispatch`) with an input for the target branch of the private repo.
- **Secure Checkout:** Uses `actions/checkout` with the `repository` parameter set to the private repo (e.g., `The-Bike-Rentals/tbr-backend`). Access is granted via a secret token (`GET_CODE_FROM_PRIVATE_REPO`).
- **Build & Deploy:** After checkout, the workflow runs build steps, logs into Azure, and deploys to Kubernetes or uploads artifacts. All sensitive credentials are managed via GitHub secrets.
- **Reusable Workflows:** For Android, a reusable workflow (`tbr-android-reusable.yaml`) is called with secrets and inputs, handling build and upload steps for different environments.

### Example: Backend Deployment
```yaml
- name: Checkout
  uses: actions/checkout@v4.1.7
  with:
    repository: The-Bike-Rentals/tbr-backend
    ref: ${{ env.target_branch }}
    token: ${{ secrets.GET_CODE_FROM_PRIVATE_REPO }}
```

### Example: Android Frontend (Reusable Workflow)
```yaml
jobs:
  development:
    uses: ./.github/workflows/tbr-android-reusable.yaml
    with:
      environment: Frontend_Development
      target_branch: ${{ github.event.inputs.target_branch }}
      folder: tst
    secrets: 
      GET_CODE_FROM_PRIVATE_REPO: ${{ secrets.GET_CODE_FROM_PRIVATE_REPO }}
      # ...other secrets...
```

## Pros and Cons of This Approach

### Pros
- **Centralized CI/CD:** All deployment logic is managed in one public repository, simplifying maintenance and visibility.
- **Security:** Secrets are used to access private repos and cloud resources, keeping sensitive data out of workflow files.
- **Modularity:** Reusable workflows allow for consistent build/deploy steps across multiple services.
- **Auditability:** Public workflows make it easy to review and audit deployment logic.
- **Separation of Concerns:** Keeps deployment logic separate from application code.

### Cons
- **Secret Management:** Exposing secret references in a public repo requires careful management to avoid leaks.
- **Coupling:** Changes in private repo structure or access may require updates in the public workflows.
- **Limited Debug Context:** The public repo does not contain the actual code, so debugging build/deploy issues may require access to the private repo.
- **Manual Triggers:** Reliance on manual workflow dispatch may slow down automation compared to push-based triggers in private repos.
- **Visibility:** Some deployment details (e.g., build failures) may not be visible to developers without access to the public CI/CD repo.

## Common Workflow Functions

- **Build and Test:** Automatically build the project and run tests to ensure code quality and prevent regressions.
- **Linting:** Check code for style and formatting issues.
- **Deployment:** Deploy the application to staging or production environments when changes are merged or released.
- **Notifications:** Send alerts or status updates to team members or external services.
- **Release Management:** Automate the creation of releases and tagging of versions.


## Best Practices

- Keep workflows modular and focused on specific tasks.
- Use secrets for sensitive information.
- Regularly review and update workflows to match project requirements.

## Additional Resources
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---
For questions or improvements, please open an issue or submit a pull request.
