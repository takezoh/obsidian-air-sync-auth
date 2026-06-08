# air-sync-auth

OAuth relay server for [Air Sync for Obsidian](https://github.com/takezoh/obsidian-air-sync). Performs server-side Google OAuth token exchange so the Client Secret stays off the client.

## Overview

Google OAuth requires redirect URIs to use `https://` — custom schemes like `obsidian://` are not allowed for Web application clients. This Cloudflare Worker receives the OAuth callback, exchanges the authorization code for tokens using the server-held Client Secret, and redirects to `obsidian://` with the tokens.

```
[Plugin] → [Google OAuth] → [Worker: /google/callback]
                                 ↓ code → token exchange
                            [obsidian://air-sync-auth?access_token=...&refresh_token=...]
```

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/google/callback` | Google OAuth redirect → token exchange → `obsidian://` redirect |
| POST | `/google/token/refresh` | Refresh token → new access token (JSON) |

## `docs/callback/`

Custom OAuth redirect page for users who bring their own Google OAuth credentials. Hosted on GitHub Pages at `airsync.takezo.dev/callback/`.

When a custom OAuth user completes Google sign-in, Google redirects to this page with `?code=...&state=...`. The page then redirects to `obsidian://air-sync-auth?code=...&state=...` so the plugin can exchange the code for tokens directly (with PKCE), without going through the auth server.

Unlike the built-in flow (`/google/callback` on the Worker), no server-side token exchange happens — the authorization code is passed through as-is.

## Infrastructure

| Domain | Host | Purpose |
|--------|------|---------|
| `airsync.takezo.dev` | GitHub Pages | Landing page, privacy policy, terms of service, custom-OAuth callback |
| `auth-airsync.takezo.dev` | Cloudflare Workers | OAuth token exchange relay |

## Local development

The worker has its own toolchain (Wrangler). Work on it from `worker/`:

```bash
cd worker
npm install
npm run dev      # wrangler dev
npx tsc -noEmit  # type-check
```

## License

MIT
