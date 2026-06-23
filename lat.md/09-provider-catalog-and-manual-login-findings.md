# Provider Catalog And Manual Login Findings

This file tracks the built-in Chat2API providers and manual login findings from live provider UI checks.

## Built-In Provider Catalog

| Provider ID | Name | Login URL | API Base | Auth type | Credential fields | Default models |
| --- | --- | --- | --- | --- | --- | --- |
| `deepseek` | DeepSeek | `https://chat.deepseek.com` | `https://chat.deepseek.com/api` | `userToken` | `token` | `deepseek-v4-flash`, `deepseek-v4-pro` |
| `glm` | GLM / Zhipu Qingyan | `https://chatglm.cn` | `https://chatglm.cn/api` | `refresh_token` | `refresh_token` | `GLM-5.1` |
| `kimi` | Kimi | `https://www.kimi.com` | `https://www.kimi.com` | `jwt` | `token` | `Kimi-K2.6` |
| `minimax` | MiniMax | `https://agent.minimaxi.com` | `https://agent.minimaxi.com` | `jwt` | `token`, optional `realUserID` | `MiniMax-M2.7` |
| `mimo` | Mimo / Xiaomi AI Studio | `https://aistudio.xiaomimimo.com` | `https://aistudio.xiaomimimo.com` | `cookie` | `service_token`, `user_id`, `ph_token` | `MiMo-V2.5-Pro`, `MiMo-V2.5`, `MiMo-V2-Flash` |
| `perplexity` | Perplexity | `https://www.perplexity.ai` | `https://www.perplexity.ai` | `cookie` | `sessionToken` | `Auto` |
| `qwen` | Qwen / Tongyi | `https://www.qianwen.com` | `https://chat2.qianwen.com` | `tongyi_sso_ticket` | `ticket` | `Qwen3.6`, `Qwen3.7-Max`, `Qwen3.5-Flash`, `Qwen3-Max`, `Qwen3-Max-Thinking-Preview`, `Qwen3-Coder` |
| `qwen-ai` | Qwen AI International | `https://chat.qwen.ai` | `https://chat.qwen.ai` | `jwt` | `token`, optional `cookies` | `Qwen3.7-Max`, `Qwen3.6-Plus`, `Qwen3.6-35B-A3B`, `Qwen3.6-27B`, `Qwen3-Coder` |
| `zai` | Z.ai | `https://chat.z.ai` | `https://chat.z.ai/api` | `jwt` | `token`, optional `captcha_verify_param` | `GLM-5.1`, `GLM-5-Turbo`, `GLM-5V-Turbo`, `GLM-5`, `GLM-4.7` |

## Manual Login Findings

| Provider ID | Login URL | Gmail/Google status | Finding | Source |
| --- | --- | --- | --- | --- |
| `deepseek` | `https://chat.deepseek.com` | Available | Gmail login is available. | Manual live UI check by project owner. |
| `glm` | `https://chatglm.cn` | Not available | Gmail login is not available; login is available only through Chinese phone number or WeChat. | Manual live UI check by project owner. |
| `kimi` | `https://www.kimi.com` | Available | Gmail login is available. | Manual live UI check by project owner. |
| `minimax` | `https://agent.minimaxi.com` | Not available | Gmail login is not available. | Manual live UI check by project owner. |
| `mimo` | `https://aistudio.xiaomimimo.com` | Unknown | Site returned 404 during manual check, so login method could not be verified. | Manual live UI check by project owner. |
| `perplexity` | `https://www.perplexity.ai` | Available | Gmail login is available. | Manual live UI check by project owner. |
| `qwen` | `https://www.qianwen.com` | Not available | Gmail login is not available. | Manual live UI check by project owner. |
| `qwen-ai` | `https://chat.qwen.ai` | Available | Gmail login is available. Need to verify whether `Qwen3.7-Max` is accessible for Gmail-authenticated accounts. | Manual live UI check by project owner. |
| `zai` | `https://chat.z.ai` | Available | Gmail login is available. Need to learn how to extract/use the newer `GLM-5.2` model. | Manual live UI check by project owner. |

## Follow-Up Provider Work

1. `qwen-ai`: verify live model availability for `Qwen3.7-Max` after Gmail login and compare it with the current configured model mapping in `src/main/providers/builtin/qwen-ai.ts`.
2. `zai`: inspect live model metadata and request payloads for `GLM-5.2`; update `src/main/providers/builtin/zai.ts`, the Z.ai proxy adapter, docs, and tests once the extraction path is known.
3. `mimo`: re-check the URL or provider status because `https://aistudio.xiaomimimo.com` returned 404 during manual login verification.
4. UI should prioritize Gmail-capable providers: `deepseek`, `kimi`, `perplexity`, `qwen-ai`, and `zai`.
5. UI should clearly mark `glm`, `minimax`, and `qwen` as not Gmail-capable based on current manual checks.

## GLM Notes

`glm` in Chat2API targets Zhipu Qingyan at `chatglm.cn` and currently supports only `GLM-5.1` through a `refresh_token` flow. The app validates tokens through `/chatglm/user-api/user/refresh`, rejects guest accounts, and accepts accounts that expose phone or email in user info.

Chat2API also includes `zai`, which is a separate provider endpoint but uses GLM-family models (`GLM-5.1`, `GLM-5-Turbo`, `GLM-5V-Turbo`, `GLM-5`, `GLM-4.7`). Treat `glm` and `zai` as separate account/login surfaces even though both are GLM-family services.

## Limit Information

The repository does not document official GLM daily/monthly quotas or rate limits. Limits must be verified through the live provider UI, provider terms, or observed API responses such as HTTP `429`, quota messages, or account status responses.
