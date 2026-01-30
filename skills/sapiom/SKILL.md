---
name: sapiom
description: Access paid services (verification, search, AI models, image generation) for AI agents via Sapiom. Use when building agents that need to verify phone/email, search the web, call AI models, or generate images — without setting up vendor accounts.
---

# Sapiom

Instant access to paid services with pay-per-use pricing. No vendor account setup required.

## Setup

1. Get your API key from https://app.sapiom.ai/settings
2. Set environment variable:
   ```bash
   export SAPIOM_API_KEY="your_key"
   ```

### Node.js (SDK)

3. Install SDK:
   ```bash
   npm install @sapiom/axios axios
   # or
   npm install @sapiom/fetch
   ```

4. Initialize client:
   ```typescript
   // Axios
   import { withSapiom } from "@sapiom/axios";
   import axios from "axios";
   const client = withSapiom(axios.create(), {
     apiKey: process.env.SAPIOM_API_KEY,
   });

   // Fetch
   import { createFetch } from "@sapiom/fetch";
   const fetch = createFetch({ apiKey: process.env.SAPIOM_API_KEY });
   ```

### Python / Other Languages (REST API)

For Python, Go, Ruby, or other languages, use the Service Proxy REST API directly. No SDK required.

**Authentication:** Include your API key as a Bearer token:
```
Authorization: Bearer YOUR_API_KEY
```

**Note:** The Service Proxy currently supports **Verify only**. Other services require the Node.js SDK.

## Services

| Service | Base URL | Description |
|---------|----------|-------------|
| Verify | `https://prelude.services.sapiom.ai` | Phone & email verification |
| Search (Linkup) | `https://linkup.services.sapiom.ai` | AI search with sourced answers |
| Search (You.com) | `https://you-com.services.sapiom.ai` | Fast search with live crawling |
| AI Models | `https://openrouter.services.sapiom.ai` | 400+ models (GPT-4, Claude, etc.) |
| Images | `https://fal-ai.services.sapiom.ai` | FLUX/SDXL generation *(coming soon)* |

## Service Proxy (Python / REST API)

For non-Node.js languages, use the Service Proxy at `https://api.sapiom.ai/v1/services/*`.

**Currently available:** Verify only. Other services coming soon.

### Verify via Service Proxy

| Step | Method | Endpoint | Body | Returns |
|------|--------|----------|------|---------|
| 1. Send code | POST | `https://api.sapiom.ai/v1/services/verify/send` | `{ "phoneNumber": "+15551234567" }` | `{ "verificationRequestId": "<uuid>" }` |
| 2. Check code | POST | `https://api.sapiom.ai/v1/services/verify/check` | `{ "verificationRequestId": "<uuid>", "code": "123456" }` | `{ "status": "success" }` |

**Python Example:**
```python
import requests
import os

API_KEY = os.environ.get("SAPIOM_API_KEY")
BASE_URL = "https://api.sapiom.ai/v1/services"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# Send verification code
response = requests.post(
    f"{BASE_URL}/verify/send",
    headers=headers,
    json={"phoneNumber": "+15551234567"}
)
request_id = response.json()["verificationRequestId"]

# Check verification code (after user enters it)
response = requests.post(
    f"{BASE_URL}/verify/check",
    headers=headers,
    json={"verificationRequestId": request_id, "code": "123456"}
)
is_verified = response.json()["status"] == "success"
```

## Endpoints (Node.js SDK)

### Verify (2-step flow)

| Step | Method | Endpoint | Body | Returns |
|------|--------|----------|------|---------|
| 1. Send code | POST | `/verifications` | `{ target: { type: "phone_number" \| "email_address", value: "+15551234567" } }` | `{ id, status }` |
| 2. Check code | POST | `/verifications/check` | `{ verificationRequestId: "<id>", code: "123456" }` | `{ id, status: "success" \| "pending" \| "failure" }` |

### Search (Linkup)

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| POST | `/v1/search` | `{ query, depth: "standard" \| "deep", outputType: "searchResults" \| "sourcedAnswer" \| "structured" }` | `{ answer?, sources?, results? }` |
| POST | `/v1/fetch` | `{ url, renderJs?: boolean }` | `{ markdown }` |

### Search (You.com)

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| POST | `/v1/search` | `{ query, count?: number, livecrawl?: "web", livecrawlFormat?: "markdown" }` | `{ results: { web: [...] } }` |
| POST | `/v1/contents` | `{ urls: string[], format?: "markdown" }` | `{ contents: [{ markdown }] }` |

### AI Models (OpenRouter)

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| POST | `/v1/chat/completions` | `{ model: "openai/gpt-4o-mini", messages: [{ role, content }], max_tokens: number }` | `{ choices: [{ message: { content } }] }` |

**Note:** `max_tokens` is required for cost calculation. Use full model name with provider prefix (e.g., `openai/gpt-4o-mini`, `anthropic/claude-3.5-sonnet`).

### Images (Fal.ai) — *coming soon*

| Method | Endpoint | Body | Returns |
|--------|----------|------|---------|
| POST | `/v1/generate` | `{ prompt, model?: "flux-dev", image_size?: "landscape_4_3" }` | `{ images: [{ url, width, height }] }` |
| POST | `/v1/transform` | `{ image_url, prompt, strength?: 0.75 }` | `{ images: [{ url }] }` |
| POST | `/v1/upscale` | `{ image_url, scale?: 2 \| 4 }` | `{ image: { url } }` |

## Example

```typescript
import { withSapiom } from "@sapiom/axios";
import axios from "axios";

const client = withSapiom(axios.create(), {
  apiKey: process.env.SAPIOM_API_KEY,
});

// Search with AI-generated answer
const { data } = await client.post(
  "https://linkup.services.sapiom.ai/v1/search",
  {
    query: "quantum computing breakthroughs",
    depth: "standard",
    outputType: "sourcedAnswer",
  }
);
console.log(data.answer);
console.log("Sources:", data.sources);

// Verify a phone number
const sendResult = await client.post(
  "https://prelude.services.sapiom.ai/verifications",
  { target: { type: "phone_number", value: "+15551234567" } }
);
// After user enters code:
const checkResult = await client.post(
  "https://prelude.services.sapiom.ai/verifications/check",
  { verificationRequestId: sendResult.data.id, code: "123456" }
);

// Call an AI model
const chatResult = await client.post(
  "https://openrouter.services.sapiom.ai/v1/chat/completions",
  {
    model: "openai/gpt-4o-mini",
    messages: [{ role: "user", content: "Hello!" }],
    max_tokens: 100,
  }
);
console.log(chatResult.data.choices[0].message.content);
```

## Pricing

| Service | Price |
|---------|-------|
| Verify | $0.015/verification |
| Search (Linkup) | $0.01 (results) - $0.08 (sourced answer) |
| Search (You.com) | $0.006/search |
| AI Models | Per-token (varies by model, see [OpenRouter](https://openrouter.ai/docs#models)) |
| Images | $0.004-$0.04/megapixel (varies by model) |
