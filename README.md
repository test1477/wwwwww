Here's a full workflow with detailed descriptions for publishing an npm package to JFrog Artifactory using GitHub Actions and OIDC authentication:

```yaml
name: Build and Publish NPM Package

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'

      - name: Setup JFrog CLI
        uses: jfrog/setup-jfrog-cli@v4
        with:
          oidc-provider-name: github-oidc-provider

      - name: Configure JFrog CLI
        run: |
          jf npm-config --repo-resolve=npm-remote --repo-deploy=npm-local

      - name: Install dependencies
        run: jf npm install

      - name: Build project
        run: npm run build

      - name: Publish to JFrog Artifactory
        run: |
          jf npm publish
        env:
          JF_URL: ${{ secrets.JF_URL }}
```

## Workflow Description:

1. **Trigger**: The workflow runs on pushes to the main branch and can be manually triggered.

2. **Permissions**: Sets necessary permissions for OIDC token generation.

3. **Checkout**: Fetches the repository code.

4. **Setup Node.js**: Installs Node.js for the build process.

5. **Setup JFrog CLI**: Installs and configures the JFrog CLI using OIDC authentication.

6. **Configure JFrog CLI**: Sets up npm configuration to use JFrog Artifactory repositories.

7. **Install dependencies**: Uses JFrog CLI to install npm dependencies from the configured remote repository.

8. **Build project**: Runs the build script defined in package.json.

9. **Publish to JFrog Artifactory**: Publishes the built package to the specified local npm repository in Artifactory.

## Setup Requirements:

1. Configure OIDC integration between GitHub and JFrog Artifactory.
2. Set up a remote npm repository in Artifactory pointing to https://registry.npmjs.org.
3. Create a local npm repository in Artifactory for publishing your packages.
4. Add the JF_URL secret in your GitHub repository settings with your JFrog Artifactory URL.

This workflow ensures secure authentication using OIDC, eliminates the need for storing API tokens, and provides a streamlined process for building and publishing npm packages to JFrog Artifactory.

Citations:
[1] https://github.com/eirslett-visma/setup-jfrog-npm
[2] https://www.youtube.com/watch?v=PYGbN_OcKX8
[3] https://stackoverflow.com/questions/70063404/publish-npm-package-to-jfrog-artifactory-using-github-action
[4] https://jfrog.com/blog/how-to-set-up-a-private-remote-and-virtual-npm-registry/
[5] https://github.com/jfrog/jfrog-cli/issues/2512
[6] https://jfrog.com/help/r/jfrog-artifactory-documentation/set-up-remote-npm-registry
[7] https://jaketrent.com/post/publish-npm-package-jfrog-artifactory/
[8] https://registry.terraform.io/providers/jfrog/artifactory/latest/docs/resources/remote_npm_repository
[9] https://github.com/jfrog/jfrog-npm-actions-example
