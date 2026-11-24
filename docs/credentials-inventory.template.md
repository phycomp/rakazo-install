# Credentials Inventory (template)

Store only the **location/pattern** of credentials, never the values.

| Credential | Where stored | Env var | Notes |
|---|---|---|---|
| OpenRouter API key | Railway variables / `.env` | `OPENROUTER_API_KEY` | Model access |
| E2B API key | Railway variables / `.env` | `E2B_API_KEY` | Sandbox provider |
| Daytona API key | Railway variables / `.env` | `DAYTONA_API_KEY` | Sandbox provider |
| Box API key | Railway variables / `.env` | `BOX_API_KEY` | Sandbox provider |
| BETTER_AUTH_SECRET | Railway (auto) | `BETTER_AUTH_SECRET` | Do not regenerate |
| ENCRYPTION_KEY | Railway (auto) | `ENCRYPTION_KEY` | Do not regenerate |
| SCREEN_PROXY_SECRET | Railway (auto) | `SCREEN_PROXY_SECRET` | Shared signing secret |
| POSTGRES_PASSWORD | Railway (auto) | `POSTGRES_PASSWORD` | DB password |
| SMTP credentials | Railway / secret manager | `SMTP_URL` | Optional |
| COMPOSIO_API_KEY | Railway / secret manager | `COMPOSIO_API_KEY` | Optional |

Rules:
- Never commit real values.
- Keep secrets in Railway variables or a secret manager.
- Regenerating `ENCRYPTION_KEY` makes stored credentials undecryptable.
