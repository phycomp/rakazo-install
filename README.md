# Rakazo Install

Installation and configuration guide for deploying Rakazo (persistent AI
teammates platform) on Railway, Docker Compose, or GitHub Codespaces.

See [INSTALL.md](./INSTALL.md) for the full step-by-step guide.

## Files

- `INSTALL.md` — full deployment guide
- `.env.example` — environment variable template (no real secrets)
- `docs/credentials-inventory.template.md` — credentials location inventory

## Security

- Never commit real API keys or secrets.
- Copy `.env.example` to `.env` and fill in values locally.
- Keep `ENCRYPTION_KEY` stable after first deploy.
