Here's the updated workflow file with a simpler and more concise configuration, leveraging the internal repository for the JFrog CLI setup and removing unnecessary steps.

```yaml
name: Build and Upload NPM Project

on:
  workflow_dispatch: # Allows manual trigger of the workflow

env:
  JF_URL: https://frigate.jfrog.io  # Replace with your JFrog Artifactory URL
  JF_REPO: npm-remote-repo         # Replace with your remote NPM repository
  JF_TARGET_REPO: npm-local-repo   # Replace with your local NPM repository
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
          node-version: '16' # Specify the Node.js version
          cache: 'npm'

      - name: Setup JFrog CLI
        uses: jfrog/setup-jfrog-cli@v4
        with:
          version: 'latest' # Automatically fetch the latest JFrog CLI version

      - name: Configure JFrog CLI
        run: |
          jf config add my-server \
            --url=${{ env.JF_URL }} \
            --apikey=${{ env.JF_ARTIFACTORY_API_TOKEN }} \
            --interactive=false

      - name: Install Dependencies from Artifactory
        run: |
          jf rt npm-config --server-id-resolve=my-server --repo-resolve=${{ env.JF_REPO }}
          npm install

      - name: Build the NPM Project
        run: npm run build

      - name: Publish to Artifactory
        run: |
          jf rt npm-publish --server-id-deploy=my-server --repo-deploy=${{ env.JF_TARGET_REPO }}

      - name: Capture Version and Upload Metadata
        run: |
          PACKAGE_VERSION=$(jq -r '.version' package.json)
          echo "Built version: $PACKAGE_VERSION"
          echo "version=$PACKAGE_VERSION" >> $GITHUB_ENV
```

### Key Updates:
1. **Removed External JFrog CLI Installation**:
   - The workflow uses `jfrog/setup-jfrog-cli@v4` from your internal repository.

2. **Streamlined Configuration**:
   - The `jf config add` command is pre-configured with `apikey` authentication.

3. **Capturing Version**:
   - Reads the `version` field from `package.json` and stores it in GitHub Actions' environment (`$GITHUB_ENV`) for later use.

4. **Simplified Commands**:
   - Ensures compatibility with your internal `setup-jfrog-cli` setup.

Let me know if any additional adjustments are needed!
