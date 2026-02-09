---
name: sapiom
description: Access paid services (verification, search, AI models, images, audio, browser automation) for AI agents via Sapiom. Use when building agents that need to verify phone/email, search the web, call AI models, generate images, convert text-to-speech, or automate browsers — without setting up vendor accounts.
---

# Sapiom

Instant access to paid services with pay-per-use pricing. No vendor account setup required.

## Documentation

**Always fetch the markdown docs for endpoint details, parameters, and examples.**

| Service | Documentation |
|---------|---------------|
| Verify | https://docs.sapiom.ai/capabilities/verify.md |
| Search | https://docs.sapiom.ai/capabilities/search.md |
| AI Models | https://docs.sapiom.ai/capabilities/ai-models.md |
| Images | https://docs.sapiom.ai/capabilities/images.md |
| Audio | https://docs.sapiom.ai/capabilities/audio.md |
| Browser | https://docs.sapiom.ai/capabilities/browser.md |
| Using Services | https://docs.sapiom.ai/using-services.md |

## How It Works

The SDK wraps your HTTP client (Axios or Fetch) and automatically handles payment negotiation with Sapiom service gateways. Each service is hosted at `{provider}.services.sapiom.ai` and proxies to the underlying vendor API.

**When to use:** Node.js/TypeScript projects. For Python or other languages, use the SDK patterns with standard HTTP clients — the payment flow requires the SDK.

```typescript
import { withSapiom } from "@sapiom/axios";
import axios from "axios";

const client = withSapiom(axios.create(), {
  apiKey: process.env.SAPIOM_API_KEY,
});

// Search the web with Linkup
const { data } = await client.post(
  "https://linkup.services.sapiom.ai/v1/search",
  { q: "quantum computing", depth: "standard", outputType: "sourcedAnswer" }
);
```

**Service URLs:**

| Service | Base URL |
|---------|----------|
| Verify | `https://prelude.services.sapiom.ai` |
| Search (Linkup) | `https://linkup.services.sapiom.ai` |
| Search (You.com) | `https://you-com.services.sapiom.ai` |
| AI Models | `https://openrouter.services.sapiom.ai` |
| Images | `https://fal.services.sapiom.ai` |
| Audio | `https://elevenlabs.services.sapiom.ai` |
| Browser | `https://anchor-browser.services.sapiom.ai` |

## Setup

1. Get your API key from https://app.sapiom.ai/settings
2. Set environment variable:
   ```bash
   export SAPIOM_API_KEY="your_key"
   ```
3. Install the SDK:
   ```bash
   npm install @sapiom/axios axios
   # or
   npm install @sapiom/fetch
   ```

## Quick Reference

### Verify (Prelude)

```typescript
// Send code
await client.post("https://prelude.services.sapiom.ai/verifications", {
  target: { type: "phone_number", value: "+15551234567" },
});
// Check code
await client.post("https://prelude.services.sapiom.ai/verifications/check", {
  verificationRequestId: "id-from-send", code: "123456",
});
```

### Search (Linkup)

```typescript
await client.post("https://linkup.services.sapiom.ai/v1/search", {
  q: "query", depth: "standard", outputType: "sourcedAnswer",
});
```

### AI Models (OpenRouter) — OpenAI-compatible

```typescript
await client.post("https://openrouter.services.sapiom.ai/v1/chat/completions", {
  model: "openai/gpt-4o-mini",
  messages: [{ role: "user", content: "Hello" }],
  max_tokens: 100,
});
```

### Images (Fal.ai)

```typescript
await client.post("https://fal.services.sapiom.ai/v1/run/fal-ai/flux/dev", {
  prompt: "A mountain landscape at sunset", image_size: "landscape_4_3",
});
```

### Audio (ElevenLabs)

```typescript
// TTS — returns binary audio
await client.post(
  "https://elevenlabs.services.sapiom.ai/v1/text-to-speech/EXAVITQu4vr4xnSDxMaL",
  { text: "Hello world", model_id: "eleven_multilingual_v2" },
  { responseType: "arraybuffer" }
);
```

### Browser (Anchor)

```typescript
// Extract content
await client.post("https://anchor-browser.services.sapiom.ai/v1/tools/fetch-webpage", {
  url: "https://example.com", format: "markdown",
});
// Screenshot
await client.post("https://anchor-browser.services.sapiom.ai/v1/tools/screenshot", {
  url: "https://example.com", width: 1280, height: 720,
});
```

## Navigating Documentation

Append `.md` to any docs URL to get the raw markdown:

| HTML Page | Markdown |
|-----------|----------|
| `https://docs.sapiom.ai/capabilities/search` | `https://docs.sapiom.ai/capabilities/search.md` |
| `https://docs.sapiom.ai/using-services` | `https://docs.sapiom.ai/using-services.md` |

**For full endpoint details, parameters, and pricing — always fetch the docs.**
