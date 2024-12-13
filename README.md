# Parse and display the HTTP status code
          status_code=$(echo "$resp" | grep -oP '(?<=HTTP/\d\.\d )\d+')
          echo "HTTP Status Code: $status_code"

          # Display the response body
          response_body=$(echo "$resp" | sed -n '/^{/,/^}/p')
          echo "Response Body:"
          echo "$response_body"

          if echo "$response_body" | jq -e '.hasError' > /dev/null; then
            echo "Error: Change Request creation failed."
            exit 1
          else
            echo "Change Request created successfully."
          fi

      - name: Check for Errors
        if: ${{ failure() }}
        run: |
          echo "Error detected in Change Ticket creation."
          exit 1
