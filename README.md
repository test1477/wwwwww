Yes, you can use an API token instead of a username and password for authentication with JFrog Artifactory. Here's an updated Python script that uses an API token:

```python
import requests
import json

# Artifactory connection details
artifactory_url = "http://your-artifactory-instance/artifactory"
api_token = "your_api_token"

# API endpoint for searching artifacts
search_endpoint = f"{artifactory_url}/api/search/aql"

# AQL query to fetch all artifacts
aql_query = 'items.find()'

# Headers for the request
headers = {
    "Content-Type": "text/plain",
    "X-JFrog-Art-Api": api_token
}

# Send the request to Artifactory
response = requests.post(search_endpoint, 
                         headers=headers, 
                         data=aql_query)

# Check if the request was successful
if response.status_code == 200:
    # Parse the JSON response
    artifacts = response.json()["results"]
    
    # Write the artifacts to a JSON file
    with open("artifacts_export.json", "w") as outfile:
        json.dump(artifacts, outfile, indent=2)
    
    print(f"Exported {len(artifacts)} artifacts to artifacts_export.json")
else:
    print(f"Failed to fetch artifacts. Status code: {response.status_code}")
    print(response.text)
```

This script uses the API token for authentication by including it in the `X-JFrog-Art-Api` header[5]. Make sure to replace `your-artifactory-instance` and `your_api_token` with your actual Artifactory URL and API token.

Citations:
[1] https://jfrog.com/help/r/jfrog-rest-apis/create-a-token
[2] https://docs.mend.io/wsk/python-with-jfrog-artifactory-host-rule-implementa
[3] https://jfrog.com/help/r/jfrog-rest-apis/create-token
[4] https://pypi.org/project/PyArtifactory/
[5] https://jfrog.com/help/r/jfrog-platform-administration-documentation/api-key
[6] https://jfrog.com/help/r/jfrog-platform-administration-documentation/access-tokens
[7] https://jfrog.com/help/r/how-to-generate-an-access-token-video
