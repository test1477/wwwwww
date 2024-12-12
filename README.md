The issue arises because the `jq` command is trying to parse a boolean value (`true`) as JSON and extract an `errorMessage` field from it, which is invalid. Here's how you can fix it:

### Fixed Script
Update the `Check for Errors` step to handle cases where the response might not be valid JSON:

```yaml
- name: Check for Errors
  if: ${{ contains(steps.crq.outputs.resp, 'true') || contains(steps.crq.outputs.resp, 'null') }}
  run: |
    echo "Error detected in Change Ticket creation."
    echo "Full API response: ${{ steps.crq.outputs.resp }}"
    
    # Check if the response is valid JSON
    if echo '${{ steps.crq.outputs.resp }}' | jq . > /dev/null 2>&1; then
      # Extract specific error message if present
      error_message=$(echo '${{ steps.crq.outputs.resp }}' | jq -r '.errorMessage // "No specific error message found"')
      echo "API Error Message: $error_message"
    else
      echo "The API response is not valid JSON. Response: ${{ steps.crq.outputs.resp }}"
    fi

    exit 1
```

---

### Explanation of Changes

1. **Check if the Response is Valid JSON:**
   ```bash
   if echo '${{ steps.crq.outputs.resp }}' | jq . > /dev/null 2>&1; then
   ```
   - This ensures the script only attempts to parse JSON if the response is valid JSON. Invalid JSON or other formats won't cause the script to fail.

2. **Extract Error Message with a Fallback:**
   ```bash
   error_message=$(echo '${{ steps.crq.outputs.resp }}' | jq -r '.errorMessage // "No specific error message found"')
   ```
   - If the `errorMessage` field is missing, it defaults to `"No specific error message found"`. This avoids errors like the one you encountered.

3. **Handle Non-JSON Responses:**
   ```bash
   echo "The API response is not valid JSON. Response: ${{ steps.crq.outputs.resp }}"
   ```
   - Logs the full response if it isn't valid JSON, helping to debug non-standard or unexpected responses.

---

### Output Example for Boolean Response
If the response is `true`, you'll see:
```plaintext
Error detected in Change Ticket creation.
Full API response: true
The API response is not valid JSON. Response: true
```

### Output Example for JSON Response
If the response is valid JSON, such as:
```json
{
  "errorMessage": "Invalid input data"
}
```
The output will be:
```plaintext
Error detected in Change Ticket creation.
Full API response: { "errorMessage": "Invalid input data" }
API Error Message: Invalid input data
``` 

This approach ensures robust error handling and debugging.
