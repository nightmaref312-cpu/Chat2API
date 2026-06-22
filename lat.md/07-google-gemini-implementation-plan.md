# Google Gemini Implementation Plan

Based on the research and codebase architecture, here is the concrete implementation plan to integrate Google Gemini CLI / Code Assist OAuth into the Chat2API application as a built-in provider.

## 1. Files to Add / Change

### Shared / Store
* `src/shared/types.ts`: Add `'google-gemini'` to `ProviderVendor`. Ensure `AuthType` includes `'oauth'` or create a `'gemini_oauth'` if needed, though `'oauth'` or `'refresh_token'` is suitable.
* `src/main/store/types.ts`: Update `BUILTIN_PROVIDERS` array to include the new `google-gemini` configuration.

### Provider Config
* **Create** `src/main/providers/builtin/google-gemini.ts`: Define `id: 'google-gemini'`, `type: 'builtin'`, `authType: 'oauth'`. Endpoints will point to `https://cloudcode-pa.googleapis.com` with `chatPath: '/v1internal:streamGenerateContent?alt=sse'`.
* **Update** `src/main/providers/builtin/index.ts`: Import and add the new config to `builtinProviders` and `builtinProviderMap`.

### OAuth Layer
* **Create** `src/main/oauth/adapters/google-gemini.ts`: Implement `GoogleGeminiAdapter` extending `BaseOAuthAdapter`. It handles `startLogin` (OAuth flow for Gemini CLI), `validateToken`, and `refreshToken`.
* **Update** `src/main/oauth/adapters/index.ts`: Export `GoogleGeminiAdapter`, and register it in `createAdapter()` and `getSupportedAuthMethods()`.
* **Update** `src/main/oauth/types.ts`: Add `google-gemini` config to `MANUAL_TOKEN_CONFIGS` so users can paste JSON credential files directly.

### Proxy Adapter Layer
* **Create** `src/main/proxy/adapters/google-gemini.ts`: Implement `GoogleGeminiAdapter` class for request preparation and token injection.
* **Create** `src/main/proxy/adapters/google-gemini-stream.ts`: Implement `GoogleGeminiStreamHandler` class to parse Gemini SSE streams into OpenAI chunk formats.
* **Update** `src/main/proxy/adapters/index.ts`: Export the new adapter and stream handler.
* **Update** `src/main/proxy/forwarder.ts`: Add routing check (`GoogleGeminiAdapter.isGoogleGeminiProvider`) and implement `forwardGoogleGemini()`.

### UI Layer
* **Update** `src/renderer/src/i18n/locales/en-US.json`, `zh-CN.json`, `ru-RU.json`: Add translations under `"google-gemini": { ... }` containing name, description, auth instructions, and model names.
* **Create** `src/assets/providers/google-gemini.svg`: Add the provider icon.
* **Update** `src/renderer/src/components/providers/ProviderCard.tsx`: Map `'google-gemini'` to the new icon.
* **Create** `docs/providers/google-gemini.md`: Documentation on how to use it.

## 2. Auth Flow & Credential Shape
* **Auth Flow**: Users will authenticate using the official Google Gemini CLI / Code Assist OAuth client ID. We will use `startLogin` to open an external browser window for consent. The app then captures the OAuth callback or allows users to paste the CLI credentials JSON.
* **Credential Shape**: Stored in `Account.credentials`. It requires `access_token`, `refresh_token`, and `expires_at`.

## 3. Token Refresh Strategy
* When the proxy adapter handles a request (`chatCompletion`), it will check the `expires_at` timestamp.
* If the `access_token` is expired or within 5 minutes of expiring, the adapter calls `refreshToken()` using the `refresh_token` to fetch a new `access_token` from Google's token endpoint before making the target API call.

## 4. Request / Response Mapping
* **Request (OpenAI -> Gemini)**:
  * OpenAI `messages` map to Gemini `contents` array (`role: "user" | "model"`, `parts: [{ text: "..." }]`).
  * The `system` message maps to Gemini `systemInstruction`.
  * OpenAI parameters like `temperature` map to `generationConfig.temperature`.
* **Response (Gemini -> OpenAI)**:
  * Gemini `candidates[0].content.parts[0].text` maps to OpenAI `choices[0].message.content`.

## 5. Streaming Parser Design
* In `google-gemini-stream.ts`, listen for `data:` chunks from the `https://cloudcode-pa.googleapis.com/...alt=sse` endpoint.
* Parse the JSON inside each chunk.
* Extract `response.candidates[0].content.parts[0].text`.
* Emit a converted chunk in OpenAI SSE format (`chat.completion.chunk`) with `delta: { content: text }`.
* If `finishReason` is present in the Gemini chunk, include `finish_reason` in the final OpenAI chunk.

## 6. Model List
* `gemini-2.5-flash`
* `gemini-2.5-pro`
* `gemini-2.0-flash`
* `gemini-exp-1206` (or latest experimental model names supported by Code Assist).

## 7. Tests
* **Unit Tests**: Add `tests/providers/google-gemini-adapter.test.ts` (request mapping logic) and `tests/providers/google-gemini-stream.test.ts` (SSE parser logic).
* Ensure translation mapping covers all scenarios.

## 8. Risks
* **API Breakage**: The `cloudcode-pa` endpoint is an internal API for Google tools and might change its request/response shape.
* **Client ID Blocking**: Google may revoke or rate-limit the specific Client ID used by the Gemini CLI.
* **Refresh Token Expiry**: Refresh tokens might be invalidated if not used for extended periods, forcing users to re-authenticate.
