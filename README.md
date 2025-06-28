---
title: Gemini CLI to API Proxy
emoji: 🤖
colorFrom: blue
colorTo: purple
sdk: docker
pinned: false
license: mit
app_port: 7860
---

# Gemini CLI to API Proxy (geminicli2api)

A FastAPI-based proxy server that converts the Gemini CLI tool into both OpenAI-compatible and native Gemini API endpoints. This allows you to leverage Google's free Gemini API quota through familiar OpenAI API interfaces or direct Gemini API calls.

## 🚀 Features

- **OpenAI-Compatible API**: Drop-in replacement for OpenAI's chat completions API
- **Native Gemini API**: Direct proxy to Google's Gemini API
- **Streaming Support**: Real-time streaming responses for both API formats
- **Multimodal Support**: Text and image inputs
- **Authentication**: Multiple auth methods (Bearer, Basic, API key)
- **Docker Ready**: Containerized for easy deployment
- **Hugging Face Spaces**: Ready for deployment on Hugging Face

## 🔧 Environment Variables

### Required
- `GEMINI_AUTH_PASSWORD`: Authentication password for API access

### Optional Credential Sources (choose one)
- `GEMINI_CREDENTIALS`: JSON string containing Google OAuth credentials
- `GOOGLE_APPLICATION_CREDENTIALS`: Path to Google OAuth credentials file
- `GOOGLE_CLOUD_PROJECT`: Google Cloud project ID
- `GEMINI_PROJECT_ID`: Alternative project ID variable

### Example Credentials JSON
```json
{
  "client_id": "your-client-id",
  "client_secret": "your-client-secret", 
  "token": "your-access-token",
  "refresh_token": "your-refresh-token",
  "scopes": ["https://www.googleapis.com/auth/cloud-platform"],
  "token_uri": "https://oauth2.googleapis.com/token"
}
```

## 📡 API Endpoints

### OpenAI-Compatible Endpoints
- `POST /v1/chat/completions` - Chat completions (streaming & non-streaming)
- `GET /v1/models` - List available models

### Native Gemini Endpoints  
- `GET /v1beta/models` - List Gemini models
- `POST /v1beta/models/{model}:generateContent` - Generate content
- `POST /v1beta/models/{model}:streamGenerateContent` - Stream content
- All other Gemini API endpoints are proxied through

### Utility Endpoints
- `GET /health` - Health check for container orchestration

## 🔐 Authentication

The API supports multiple authentication methods:

1. **Bearer Token**: `Authorization: Bearer YOUR_PASSWORD`
2. **Basic Auth**: `Authorization: Basic base64(username:YOUR_PASSWORD)`
3. **Query Parameter**: `?key=YOUR_PASSWORD`
4. **Google Header**: `x-goog-api-key: YOUR_PASSWORD`

## 🐳 Docker Usage

```bash
# Build the image
docker build -t geminicli2api .

# Run on default port 8888 (compatibility)
docker run -p 8888:8888 \
  -e GEMINI_AUTH_PASSWORD=your_password \
  -e GEMINI_CREDENTIALS='{"client_id":"...","token":"..."}' \
  -e PORT=8888 \
  geminicli2api

# Run on port 7860 (Hugging Face compatible)
docker run -p 7860:7860 \
  -e GEMINI_AUTH_PASSWORD=your_password \
  -e GEMINI_CREDENTIALS='{"client_id":"...","token":"..."}' \
  -e PORT=7860 \
  geminicli2api
```

### Docker Compose

```bash
# Default setup (port 8888)
docker-compose up -d

# Hugging Face setup (port 7860)
docker-compose --profile hf up -d geminicli2api-hf
```

## 🤗 Hugging Face Spaces

This project is configured for Hugging Face Spaces deployment:

1. Fork this repository
2. Create a new Space on Hugging Face
3. Connect your repository
4. Set the required environment variables in Space settings:
   - `GEMINI_AUTH_PASSWORD`
   - `GEMINI_CREDENTIALS` (or other credential source)

The Space will automatically build and deploy using the included Dockerfile.

## 📝 OpenAI API Example

```python
import openai

# Configure client to use your proxy
client = openai.OpenAI(
    base_url="http://localhost:8888/v1",  # or 7860 for HF
    api_key="your_password"  # Your GEMINI_AUTH_PASSWORD
)

# Use like normal OpenAI API
response = client.chat.completions.create(
    model="gemini-2.0-flash-exp",
    messages=[
        {"role": "user", "content": "Hello, how are you?"}
    ],
    stream=True
)

for chunk in response:
    print(chunk.choices[0].delta.content, end="")
```

## 🔧 Native Gemini API Example

```python
import requests

headers = {
    "Authorization": "Bearer your_password",
    "Content-Type": "application/json"
}

data = {
    "contents": [
        {
            "role": "user", 
            "parts": [{"text": "Hello, how are you?"}]
        }
    ]
}

response = requests.post(
    "http://localhost:8888/v1beta/models/gemini-2.0-flash-exp:generateContent",  # or 7860 for HF
    headers=headers,
    json=data
)

print(response.json())
```

## 🎯 Supported Models

- `gemini-2.0-flash-exp`
- `gemini-1.5-flash`
- `gemini-1.5-flash-8b`
- `gemini-1.5-pro`
- `gemini-1.0-pro`

## 📄 License

MIT License - see LICENSE file for details.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.