```
git clone https://github.com/microsoft/graphrag
cd graphrag
uv sync
source .venv/bin/activate

#Create a .env file with
GRAPHRAG_API_KEY="ollama"

# To generate the setting.yaml
graphrag init --root .

# Edit settings.yaml to support Ollama
# IMPORTANT: Change the model accoding accordingly
models:
  default_chat_model:
    type: chat
    model_provider: openai
    auth_type: api_key # or azure_managed_identity
    api_key: ${GRAPHRAG_API_KEY} # This will read "ollama" from your .env file
    model: "granite4:350m"                # CHANGED: Your Ollama chat model
    api_base: "http://localhost:11434/v1" # CHANGED: Point to local Ollama API
    # api_version: 2024-05-01-preview
    model_supports_json: true # Assumes your model (e.g., Llama 3) supports JSON mode
    concurrent_requests: 4    # CHANGED: Lowered to avoid overloading local machine
    async_mode: threaded # or asyncio
    retry_strategy: exponential_backoff
    max_retries: 10
    tokens_per_minute: null
    requests_per_minute: null
  default_embedding_model:
    type: embedding
    model_provider: openai
    auth_type: api_key
    api_key: ${GRAPHRAG_API_KEY} # This will read "ollama" from your .env file
    model: "nomic-embed-text"    # CHANGED: Your Ollama embedding model
    api_base: "http://localhost:11434/v1" # CHANGED: Point to local Ollama API
    # api_version: 2024-05-01-preview
    concurrent_requests: 4 # CHANGED: Lowered to avoid overloading local machine
    async_mode: threaded # or asyncio
    retry_strategy: exponential_backoff
    max_retries: 10
    tokens_per_minute: null
    requests_per_minute: null

rm -rf output/ cache/ logs/

# GraphRAG looks at Ollama to create text embeddings
ollama pull nomic-embed-text

# granite4 is very small model from IBM you can use
# use to do a quick test
# Note: Quality is directly proportional to Model size 
# (and time).     
ollama run granite4:350m

# Make sure Ollama model is reachable
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "granite4:350m",
        "messages": [
          {
            "role": "system",
            "content": "You are a helpful assistant."
          },
          {
            "role": "user",
            "content": "Hello!"
          }
        ]
      }'

# Uncomment fo you want to see the ollama logs
# tail -f ~/.ollama/logs/server.log
# tail -f logs/indexing-engine.log

# Test
mkdir -p input
echo "Alice is a software engineer at Microsoft. Bob is a data scientist at Google. Alice and Bob are colleagues." > input/test.txt
graphrag index --root . --verbose
graphrag query --root . --method local --query "What is the relationship between Alice and Bob?"

# Test 2
curl https://www.gutenberg.org/cache/epub/24022/pg24022.txt -o ./input/book.txt

graphrag index --root .

graphrag query \
--root ./ \
--method local \
--query "Who is Scrooge and what are his main relationships?"
```