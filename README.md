# localai-github-action

A dead simple Github action which allows to run [LocalAI](https://github.com/mudler/LocalAI) in the Github action runner workflow.

Example usage:
```yaml
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      - uses: actions/checkout@v4
      - id: foo
        uses: mudler/localai-github-action@v1
        with:
          model: 'hermes-2-theta-llama-3-8b' # Any from models.localai.io, or from huggingface.com with: "huggingface://<repository>/file"
      - name: Greet
        id: greet
        run: |
                input="Hello!"

                # Define the LocalAI API endpoint
                API_URL="http://localhost:8080/chat/completions"

                # Create a JSON payload using jq to handle special characters
                json_payload=$(jq -n --arg input "$input" '{
                model: "'$MODEL_NAME'",
                messages: [
                    {
                    role: "system",
                    content: "You are a friendly companion"
                    },
                    {
                    role: "user",
                    content: $input
                    }
                ]
                }')

                # Send the request to LocalAI
                response=$(curl -s -X POST $API_URL \
                -H "Content-Type: application/json" \
                -d "$json_payload")

                # Extract the result from the response
                result="$(echo $response | jq -r '.choices[0].message.content')"

                # Print the result
                echo "Result:"
                echo "$result"
                echo "payload sent"
                echo "$json_payload"
                {
                    echo 'message<<EOF'
                    echo "$result"
                    echo EOF
                } >> "$GITHUB_OUTPUT"
                # now the output can be consumed with ${{ steps.summarize.outputs.message }} by other steps
```