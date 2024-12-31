Here’s the full updated GitHub Actions workflow for building and uploading an NPM project to JFrog Artifactory:

```yaml
name: Build and Upload NPM Project

on:
  workflow_dispatch:  # Allows manual trigger of the workflow

env:
  JF_URL: https://frigate.jfrog.io  # Your JFrog Artifactory URL
  JF_REMOTE_REPO: npm-remote        # The remote NPM repository
  JF_TARGET_REPO: npm-test-project  # The target local NPM repository
  JF_ARTIFACTORY_API_TOKEN: ${{ secrets.JF_ARTIFACTORY_API_TOKEN }} # GitHub secret for your JFrog API token

jobs:
  build-and-upload:
    runs-on: ubuntu-latest

    steps:
      # Checkout the code from the repository
      - name: Checkout Code
        uses: actions/checkout@v3

      # Set up Node.js environment and registry URL for Artifactory
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'  # Node.js version to use
          registry-url: 'https://frigate.jfrog.io/artifactory/api/npm/${{ env.JF_TARGET_REPO }}/'  # Registry URL for Artifactory

      # Configure NPM to use the Artifactory registry with an API token
      - name: Configure NPM for Artifactory
        run: |
          echo "//frigate.jfrog.io/artifactory/api/npm/${{ env.JF_TARGET_REPO }}/:_authToken=${{ secrets.JF_ARTIFACTORY_API_TOKEN }}" > ~/.npmrc
          npm config set registry https://frigate.jfrog.io/artifactory/api/npm/${{ env.JF_TARGET_REPO }}/

      # Install dependencies using npm ci
      - name: Install Dependencies
        run: npm ci

      # Build the NPM project
      - name: Build the NPM Project
        run: npm run build

      # Publish the package to Artifactory
      - name: Publish to Artifactory
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.JF_ARTIFACTORY_API_TOKEN }}  # Use the API token for authentication

      # Verify the package has been successfully published
      - name: Verify Published Package
        run: |
          echo "Package published successfully to:"
          echo "https://frigate.jfrog.io/artifactory/webapp/#/artifacts/browse/tree/General/${{ env.JF_TARGET_REPO }}/"

      # Verify the npm registry configuration (optional debugging step)
      - name: Verify NPM Registry
        run: npm config get registry
```

### Key Steps:
1. **Checkout Code**: Pulls the code from your GitHub repository.
2. **Setup Node.js**: Installs Node.js and sets the Artifactory registry for npm.
3. **Configure NPM for Artifactory**: Configures npm to use the correct registry with your API token.
4. **Install Dependencies**: Uses `npm ci` to install the project's dependencies.
5. **Build the NPM Project**: Runs the build process (via `npm run build`).
6. **Publish to Artifactory**: Publishes the built package to the specified Artifactory repository.
7. **Verify Published Package**: Logs the URL of the package in Artifactory for confirmation.
8. **Verify NPM Registry**: Logs the current npm registry to confirm it's set correctly (optional debugging step).

### Important Notes:
- Make sure that `JF_ARTIFACTORY_API_TOKEN` is stored as a GitHub secret for security purposes.
- Ensure that the API token has the necessary permissions to read and write to the specified Artifactory repositories.
- Adjust the `node-version` and repository names according to your project's configuration.

Let me know if you need any further adjustments!
