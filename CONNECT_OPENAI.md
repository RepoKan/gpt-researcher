# Connecting GPT Researcher with OpenAI

This guide provides step-by-step instructions to configure and connect GPT Researcher with OpenAI's API.

## Prerequisites

- GPT Researcher installed and ready to use
- An OpenAI account with API access
- Python 3.11 or later
- Active OpenAI API key with appropriate billing setup

## Step 1: Obtain Your OpenAI API Key

1. Visit [OpenAI Platform](https://platform.openai.com/)
2. Sign in or create an account
3. Navigate to **API Keys** in the left sidebar
4. Click **Create new secret key**
5. Copy the key (you won't see it again)
6. Store it securely

## Step 2: Configure Environment Variables

### Option A: Using `.env` File (Recommended for Local Development)

1. Navigate to the GPT Researcher root directory
2. Create a `.env` file from the template:
   ```bash
   cp .env.example .env
   ```

3. Open `.env` and add your OpenAI API key:
   ```bash
   OPENAI_API_KEY=sk-your-api-key-here
   ```

4. (Optional) Add a secondary search provider:
   ```bash
   TAVILY_API_KEY=tvly-your-tavily-key-here
   ```

### Option B: Using Environment Variables (CLI)

For temporary sessions, export directly:

```bash
export OPENAI_API_KEY=sk-your-api-key-here
export TAVILY_API_KEY=tvly-your-tavily-key-here
```

### Option C: Using Docker Environment

Edit your `docker-compose.yml`:

```yaml
services:
  gpt-researcher:
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      TAVILY_API_KEY: ${TAVILY_API_KEY}
```

Then create a `.env` file in the project root with your keys.

## Step 3: Verify Connection

### Test Locally

Create a test script `test_openai_connection.py`:

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

# Initialize OpenAI client
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# Test connection
try:
    response = client.models.list()
    print("✅ Connection successful!")
    print(f"Available models: {len(response.data)}")
except Exception as e:
    print(f"❌ Connection failed: {e}")
```

Run the test:
```bash
python test_openai_connection.py
```

### Test with GPT Researcher

```python
import asyncio
from gpt_researcher import GPTResearcher

async def test_gptr():
    researcher = GPTResearcher(query="What is artificial intelligence?")
    report = await researcher.conduct_research()
    return report

# Run the test
result = asyncio.run(test_gptr())
print(result)
```

## Step 4: Configure LLM Settings (Optional)

Edit your `.env` file to customize the OpenAI model and parameters:

```bash
# Default LLM Model (Fast Mode)
FAST_LLM=openai:gpt-4o-mini

# Smart LLM Model (Balance)
SMART_LLM=openai:gpt-4o

# Strategic LLM Model (High Quality)
STRATEGIC_LLM=openai:gpt-4-turbo

# Embedding Model
EMBEDDING_MODEL=openai:text-embedding-3-small

# Token Limits
FAST_TOKEN_LIMIT=3000
SMART_TOKEN_LIMIT=6000
STRATEGIC_TOKEN_LIMIT=4000

# Temperature (0.0 - 1.0, disabled for reasoning models)
# LLM_TEMPERATURE=0.7
```

## Step 5: Start GPT Researcher

### Local Development

```bash
python -m uvicorn main:app --reload
```

Access at: `http://localhost:8000`

### Using Docker

```bash
docker-compose up --build
```

Access API at: `http://localhost:8000`  
Access Frontend at: `http://localhost:3000`

### Using Python Package

```python
from gpt_researcher import GPTResearcher

async def main():
    query = "Latest AI trends in 2025"
    researcher = GPTResearcher(query=query)
    report = await researcher.conduct_research()
    print(report)

import asyncio
asyncio.run(main())
```

## Advanced Configuration

### Custom OpenAI Base URL

For local models or custom deployments:

```bash
OPENAI_BASE_URL=http://localhost:8000
OPENAI_API_KEY=your-key-or-dummy-key
```

### Different Model for Embeddings

```bash
# Use a different embedding provider
EMBEDDING_MODEL=ollama:nomic-embed-text
OLLAMA_BASE_URL=http://localhost:11434
```

### Enable Observability (LangSmith)

```bash
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your-langsmith-key
LANGCHAIN_PROJECT=gpt-researcher
```

### Token Limits by Model

Update `.env` based on your model's context window:

```bash
# GPT-4o (128k context)
SMART_TOKEN_LIMIT=16000

# GPT-4 Turbo (128k context)
STRATEGIC_TOKEN_LIMIT=16000

# GPT-4o Mini (128k context)
FAST_TOKEN_LIMIT=8000
```

## Troubleshooting

### Issue: `AuthenticationError: Incorrect API key provided`

**Solution:**
- Verify API key is correct in `.env`
- Check that API key has not been rotated/revoked
- Ensure no extra spaces around the key

```bash
# Check your .env file
cat .env | grep OPENAI_API_KEY
```

### Issue: `RateLimitError: 429 Resource has been exhausted`

**Solution:**
- Implement rate limiting in `.env`:
```bash
SCRAPER_RATE_LIMIT_DELAY=6.0  # 1 request per 6 seconds
```

- Adjust token limits for faster responses:
```bash
FAST_TOKEN_LIMIT=2000
SMART_TOKEN_LIMIT=4000
```

### Issue: `Model not found: gpt-4o`

**Solution:**
- Verify the model name is correct
- Check your OpenAI account has access to the model
- Use available models:
```bash
# List available models
python -c "from openai import OpenAI; import os; client = OpenAI(); print([m.id for m in client.models.list()])"
```

### Issue: `Connection timeout`

**Solution:**
- Check OpenAI API status: https://status.openai.com
- Verify internet connectivity
- For custom base URLs, ensure the server is running

## Best Practices

### 1. **Security**
- Never commit `.env` to version control
- Use `.gitignore` to exclude `.env`:
  ```bash
  echo ".env" >> .gitignore
  ```
- Rotate API keys regularly
- Use environment-specific keys for production

### 2. **Cost Management**
- Set token limits appropriately
- Use `FAST_TOKEN_LIMIT` for quick queries
- Monitor usage in OpenAI dashboard
- Consider rate limiting to avoid excessive API calls

### 3. **Performance**
- Use appropriate LLM models for your use case:
  - **FAST**: Quick, cost-effective (gpt-4o-mini)
  - **SMART**: Balanced (gpt-4o)
  - **STRATEGIC**: High quality (gpt-4-turbo)

### 4. **Error Handling**
- Enable logging to debug issues:
  ```bash
  LOGGING_LEVEL=DEBUG
  ```
- Check logs in `logs/app.log`

## API Usage Examples

### Basic Research Query

```python
from gpt_researcher import GPTResearcher
import asyncio

async def research():
    researcher = GPTResearcher(
        query="What are the latest breakthroughs in quantum computing?",
        report_type="research_report"
    )
    report = await researcher.conduct_research()
    return report

result = asyncio.run(research())
print(result)
```

### Custom Report with Options

```python
async def advanced_research():
    researcher = GPTResearcher(
        query="Best practices for Python development",
        report_type="detailed_report",
        source_urls=["github.com", "python.org"],
        tone="formal",
        max_iterations=10
    )
    report = await researcher.conduct_research()
    md_report = await researcher.write_report()
    return md_report

result = asyncio.run(advanced_research())
```

## Testing Your Setup

Run the built-in test suite:

```bash
# Run all tests
docker-compose --profile test up

# Or locally
pytest tests/report-types.py -v
```

## Additional Resources

- [OpenAI Documentation](https://platform.openai.com/docs)
- [GPT Researcher Docs](https://docs.gptr.dev)
- [LangChain OpenAI](https://python.langchain.com/docs/integrations/llms/openai)
- [Discord Community](https://discord.gg/QgZXvJAccX)

## Support

If you encounter issues:

1. Check the troubleshooting section above
2. Review logs: `tail -f logs/app.log`
3. Visit [GitHub Issues](https://github.com/assafelovic/gpt-researcher/issues)
4. Ask in [Discord Community](https://discord.gg/QgZXvJAccX)

---

**Last Updated:** 2026-01-09  
**GPT Researcher Version:** 0.14.7+  
**OpenAI API Version:** Compatible with v1.0+
