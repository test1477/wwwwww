The error you're encountering is because the response `'true'` is a boolean and not valid JSON. Attempting to process it with `jq` results in an error. Here’s a fixed version of your script to handle this scenario gracefully:

### Corrected Script

```bash
echo "Error detected in Change Ticket creation."
echo "Full API response: true"

# Check if the response is valid JSON
if echo 'true' | jq . > /dev/null 2>&1; then
  # Extract specific error message if present
  error_message=$(echo 'true' | jq -r '.errorMessage // "No specific error message found"')
  echo "API Error Message: $error_message"
else
  # Handle non-JSON responses
  if [[ 'true' == "true" ]]; then
    echo "The API response is a boolean value: true"
  elif [[ 'true' == "false" ]]; then
    echo "The API response is a boolean value: false"
  else
    echo "The API response is not valid JSON. Response: true"
  fi
fi

exit 1
```

### Key Fixes

1. **Proper JSON Parsing Check:**
   - The `if echo 'true' | jq . > /dev/null 2>&1` block ensures the script attempts JSON parsing only if the response is JSON.

2. **Correct String Comparison for Booleans:**
   - The condition `[[ 'true' == "true" ]]` ensures correct comparison without syntax issues.

3. **Avoid Processing Booleans with `jq`:**
   - `jq` is not used on boolean values, avoiding the error `Cannot index boolean with string "errorMessage"`.

### Output Scenarios
- If the response is `true` or `false`, the script will identify it and log the value.
- If the response is valid JSON, the `errorMessage` is extracted.
- For any other non-JSON responses, an appropriate message is logged. 

### Testing
Run the script with different responses (`true`, `false`, valid JSON, invalid JSON) to ensure it behaves as expected.
