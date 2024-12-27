Here’s a GitHub Actions workflow for building an NPM project, fetching dependencies from a JFrog Artifactory remote repository, and uploading the built package to a specific repository in Artifactory. The workflow uses JFrog CLI and an API token for authentication.

---

### **GitHub Actions Workflow File**

Save this workflow in your repository as `.github/workflows/npm-artifactory-build.yml`.

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
          node-version: '16' # Use the desired Node.js version
          cache: 'npm'

      - name: Install JFrog CLI
        uses: jfrog/setup-jfrog-cli@v4
        with:
          version: 'latest'
      
      - name: Configure JFrog CLI
        run: |
          jfrog config add my-server \
            --url=${{ env.JF_URL }} \
            --apikey=${{ env.JF_ARTIFACTORY_API_TOKEN }} \
            --interactive=false

      - name: Install Dependencies from Artifactory
        run: |
          jf rt npm-config --server-id-resolve=my-server --repo-resolve=${{ env.JF_REPO }}
          npm install

      - name: Build the NPM Project
        run: |
          npm run build

      - name: Publish to Artifactory
        run: |
          jf rt npm-publish --server-id-deploy=my-server --repo-deploy=${{ env.JF_TARGET_REPO }}

      - name: Capture Version and Upload Metadata
        run: |
          PACKAGE_VERSION=$(jq -r '.version' package.json)
          echo "Built version: $PACKAGE_VERSION"
          echo "version=$PACKAGE_VERSION" >> $GITHUB_ENV
```

---

### **Explanation**

1. **Triggers:**
   - The `workflow_dispatch` event allows manual triggering from the GitHub Actions UI.

2. **Environment Variables:**
   - `JF_URL`: Your JFrog Artifactory URL.
   - `JF_REPO`: The Artifactory remote repository used for dependency resolution.
   - `JF_TARGET_REPO`: The Artifactory local repository where the built package is uploaded.
   - `JF_ARTIFACTORY_API_TOKEN`: An API token for authenticating with Artifactory (stored as a secret).

3. **Steps:**
   - **Checkout Code:** Clones the repository into the workflow runner.
   - **Setup Node.js:** Installs the required Node.js version.
   - **Install JFrog CLI:** Sets up the JFrog CLI tool.
   - **Configure JFrog CLI:** Configures the CLI with the Artifactory URL and API token.
   - **Install Dependencies:** Fetches NPM dependencies from the Artifactory remote repository.
   - **Build the Project:** Runs the build script (assumes a script named `build` is in `package.json`).
   - **Publish to Artifactory:** Uploads the built package to the specified Artifactory local repository.
   - **Capture Version:** Extracts the `version` field from `package.json` and logs it for traceability.

4. **Secrets Management:**
   - Add your JFrog API token as a GitHub secret named `JF_ARTIFACTORY_API_TOKEN`.

---

### **How to Trigger the Workflow**
1. Navigate to the "Actions" tab in your GitHub repository.
2. Select this workflow and click **"Run workflow"**.
3. The workflow will:
   - Install dependencies.
   - Build the project.
   - Publish the package to Artifactory.
   - Log the built version.

Let me know if you need additional customizations!
