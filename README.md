To fetch JSON for all artifacts from all repositories in Artifactory, you can enhance the script to:

1. Fetch all repository names using the `/api/repositories` endpoint.
2. Iterate through each repository to fetch artifacts using the `/api/storage` endpoint.

Here's the updated script:

### Python Script: Fetch JSON for All Artifacts from All Repositories

```python
import requests
import json

# Artifactory Configuration
ARTIFACTORY_URL = "https://your-artifactory-instance/artifactory"  # Replace with your Artifactory URL
API_KEY = "your-api-key"  # Replace with your API key

def get_repositories(artifactory_url, api_key):
    """Fetch all repositories from Artifactory."""
    url = f"{artifactory_url}/api/repositories"
    headers = {"Authorization": f"Bearer {api_key}"}
    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Error fetching repositories: {e}")
        return []

def fetch_artifacts(repo_name, artifactory_url, api_key):
    """Fetch all artifacts for a given repository."""
    artifacts = []
    url = f"{artifactory_url}/api/storage/{repo_name}"
    headers = {"Authorization": f"Bearer {api_key}"}

    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        repo_data = response.json()

        # Recursively fetch all artifacts in the repository
        def fetch_children(data_url):
            child_response = requests.get(data_url, headers=headers)
            child_response.raise_for_status()
            data = child_response.json()

            if "children" in data:
                for child in data["children"]:
                    child_url = f"{data_url}{child['uri']}"
                    if child["folder"]:
                        fetch_children(child_url)  # Recurse into subfolders
                    else:
                        artifacts.append(child_url)

        fetch_children(url)

    except requests.exceptions.RequestException as e:
        print(f"Error fetching artifacts for {repo_name}: {e}")

    return artifacts

def save_to_file(data, file_name="all_artifacts.json"):
    """Save JSON data to a file."""
    try:
        with open(file_name, "w") as f:
            json.dump(data, f, indent=4)
        print(f"All artifacts saved to {file_name}")
    except IOError as e:
        print(f"Error saving file: {e}")

if __name__ == "__main__":
    all_artifacts = {}
    repositories = get_repositories(ARTIFACTORY_URL, API_KEY)

    if repositories:
        for repo in repositories:
            repo_name = repo["key"]
            print(f"Fetching artifacts for repository: {repo_name}")
            artifacts = fetch_artifacts(repo_name, ARTIFACTORY_URL, API_KEY)
            all_artifacts[repo_name] = artifacts
    else:
        print("No repositories found or error occurred.")

    save_to_file(all_artifacts)
```

### Instructions:
1. **Update Configuration**:
   - Replace `ARTIFACTORY_URL` with your Artifactory URL.
   - Replace `API_KEY` with your Artifactory API key.

2. **Run the Script**:
   Save the script as `fetch_all_artifacts.py` and execute it:
   ```bash
   python fetch_all_artifacts.py
   ```

3. **Output**:
   - The script will fetch JSON data for artifacts in all repositories and save it in `all_artifacts.json`.

### How It Works:
1. **Get Repositories**: It fetches a list of all repositories from the `/api/repositories` endpoint.
2. **Fetch Artifacts**: For each repository, it recursively fetches all artifact URLs using the `/api/storage` endpoint.
3. **Save JSON**: The data is stored in a dictionary with repository names as keys and lists of artifact URLs as values.

### Sample Output (`all_artifacts.json`):
```json
{
    "repo1": [
        "https://your-artifactory-instance/artifactory/repo1/path/to/artifact1",
        "https://your-artifactory-instance/artifactory/repo1/path/to/artifact2"
    ],
    "repo2": [
        "https://your-artifactory-instance/artifactory/repo2/path/to/artifact1"
    ]
}
```

Let me know if you need further customizations!
