Here's the updated GitHub Actions workflow, incorporating the configuration needed to publish an NPM package to your JFrog Artifactory repository. It uses the `publishConfig` in `package.json` or the `--registry` flag for publishing:

```yaml
name: Build and Upload NPM Project

on:
  workflow_dispatch: # Allows manual trigger of the workflow

env:
  JF_URL: https://frigate.jfrog.io  # Replace with your JFrog Artifactory URL
  JF_TARGET_REPO: npm-test-project  # Replace with your target NPM repository
  JF_ARTIFACTORY_API_TOKEN: ${{ secrets.JF_ARTIFACTORY_API_TOKEN }} # Store the token as a GitHub secret

jobs:
  build-and-upload:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16' # Use the desired Node.js version
          cache: 'npm'

      - name: Configure NPM Registry for Artifactory
        run: |
          echo "//frigate.jfrog.io/artifactory/api/npm/${{ env.JF_TARGET_REPO }}/:_authToken=${{ secrets.JF_ARTIFACTORY_API_TOKEN }}" > ~/.npmrc

      - name: Install Dependencies
        run: |
          npm install

      - name: Build the NPM Project
        run: |
          npm run build

      - name: Publish to Artifactory
        run: |
          npm publish --registry=https://frigate.jfrog.io/artifactory/api/npm/${{ env.JF_TARGET_REPO }}/

      - name: Verify Published Package
        run: |
          echo "Checking if the package was published successfully..."
          echo "Published to: https://frigate.jfrog.io/artifactory/webapp/#/artifacts/browse/tree/General/${{ env.JF_TARGET_REPO }}"
```

### Key Updates
1. **Registry Configuration (`.npmrc`)**:
   - Explicitly sets the registry for Artifactory using `npmrc` with an API token.

2. **Publish Command**:
   - Uses `npm publish` with the `--registry` flag to target the correct Artifactory repository.

3. **Verification**:
   - Includes a message with the Artifactory URL where you can verify the published package.

---

### Notes:
- **Authentication**: Ensure that the API token has sufficient permissions to publish packages to the target repository.
- **Secrets**: Use GitHub Secrets for storing sensitive information like `JF_ARTIFACTORY_API_TOKEN`.
- **Build Script**: Ensure that your `package.json` includes a `build` script if you’re using `npm run build`.

Let me know if you need further assistance!
