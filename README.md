The error occurs because the syntax for configuring the JFrog CLI server is incorrect. The JFrog CLI command for adding a server configuration requires a specific structure. Let's fix it.

Here's the **correct and complete workflow** to build an npm project, fetch dependencies from a JFrog remote repository, and upload the build to an Artifactory repository.

---

### **Workflow YAML**

```yaml
name: Build and Upload to JFrog

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 16
        cache: npm

    - name: Install dependencies
      run: npm ci

    - name: Build project
      run: npm run build

    - name: Install JFrog CLI
      run: |
        curl -fL https://getcli.jfrog.io | sh
        chmod +x jfrog
        ./jfrog --version

    - name: Configure JFrog CLI
      run: |
        ./jfrog config add my-server \
          --url=https://frigate.jfrog.io \
          --access-token=${{ secrets.JFROG_TOKEN }} \
          --interactive=false

    - name: Authenticate npm with JFrog
      run: ./jfrog npm-config --server-id-resolve=my-server --repo-resolve=my-npm-remote

    - name: Publish npm package to JFrog Artifactory
      run: ./jfrog npm-publish --server-id-deploy=my-server --repo-deploy=my-npm-local

    - name: Capture version
      run: |
        VERSION=$(node -p "require('./package.json').version")
        echo "Package version: $VERSION"
```

---

### **Key Points in the Workflow**

1. **Setup Node.js and npm Cache**
   - Ensure you use `actions/setup-node` to install Node.js and enable npm caching for faster builds.

2. **Install JFrog CLI**
   - Use the official script to install JFrog CLI and make it executable.

3. **Configure JFrog CLI**
   - Add a JFrog server configuration using `jfrog config add`.
   - Use `--access-token` to authenticate securely (retrieve the token from GitHub Secrets).

4. **Authenticate npm with JFrog**
   - Configure npm to resolve dependencies from the JFrog remote repository (e.g., `my-npm-remote`).

5. **Publish to Artifactory**
   - Use `jfrog npm-publish` to upload the build to the JFrog local repository (e.g., `my-npm-local`).

6. **Capture the Version**
   - Extract the version from `package.json` using Node.js and log it for confirmation.

---

### **Repository Setup**
1. **GitHub Secrets**
   - Add the following secrets in your GitHub repository:
     - `JFROG_TOKEN`: Your API token for JFrog.
     - `JFROG_URL`: The base URL of your JFrog instance (e.g., `https://frigate.jfrog.io`).

2. **Jfrog Repositories**
   - **my-npm-remote**: Remote repository for npm dependencies.
   - **my-npm-local**: Local repository for publishing npm packages.

---

### **Common Errors and Fixes**

1. **Wrong Number of Arguments (Config Add)**
   - Ensure all required arguments (`--url`, `--access-token`) are passed correctly.

2. **No Lock File Error**
   - Ensure `package-lock.json` exists in the repository by running `npm install` locally and committing the file.

3. **Repository Not Found**
   - Verify the repository names (`my-npm-remote`, `my-npm-local`) in the JFrog Artifactory instance.

---

Let me know if you need further adjustments!
