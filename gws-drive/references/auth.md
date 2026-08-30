# Authentication & OAuth

```bash
gws auth setup    # one-time: GCP project + API enablement + OAuth client config
gws auth login    # browser-based OAuth consent flow
gws auth list     # list/configure accounts (where the installed version supports it)
gws auth logout
```

`gws auth setup` shells out to `gcloud`. Install and authenticate gcloud first — see
[setup.md](./setup.md).

---

## The four pieces

### 1. GCP project

A Google Cloud project is the **Cloud-side configuration container** used to authorize and
identify the application. It is *not* your Drive. Your files do not live in it. It exists so
Google can attach an OAuth client and API enablement to something.

Project IDs are **globally unique across all of Google Cloud** — that is why `testing123`
was rejected. Pick something namespaced: `abhiman-gws-2026`.

The successful setup here used project `cli-arch`.

### 2. OAuth consent / audience

For an **External** app in **Testing** mode, Google only permits accounts on the
**Test users** list to authorize. Add the account you will actually log in with — including
your own developer account. Skipping this produces the `Access blocked` 403 (see
[troubleshooting.md](./troubleshooting.md)).

### 3. OAuth client type — **Desktop app**

| Type | Correct? |
|---|---|
| **Desktop app** | ✅ Use this |
| Chrome Extension | ❌ Wrong — was tried first and does not work |
| Web application | ❌ Not this flow |

gws runs a local terminal flow with a **localhost redirect** — the successful authorization
URL contained `redirect_uri=http://localhost:44789`. Only the Desktop app client type
supports that. Name it something like `gws-local`.

### 4. Credential files

```
~/.config/gws/
├── client_secret.json    # OAuth client configuration
└── credentials.enc       # encrypted OAuth credentials — AES-256-GCM
```

Encryption key lives in the OS keyring, or in a local `.encryption_key` file as fallback.
The successful path in this setup reported the `keyring` backend, so `credentials.enc` is
**not** a plaintext token dump.

---

## Scopes granted by the successful login

```json
{
  "account": "abhimanhq@gmail.com",
  "credentials_file": "/home/abhi/.config/gws/credentials.enc",
  "encryption": "AES-256-GCM (key in OS keyring or local `.encryption_key`)",
  "message": "Authentication successful. Encrypted credentials saved.",
  "scopes": [
    "https://www.googleapis.com/auth/drive",
    "https://www.googleapis.com/auth/cloud-platform",
    "openid",
    "https://www.googleapis.com/auth/userinfo.email",
    "https://www.googleapis.com/auth/userinfo.profile"
  ],
  "status": "success"
}
```

Note what is **absent**: no `drive.appdata`, no Drive Apps scope. That is why
`gws drive apps list` returns `403 insufficient authentication scopes` — expected, not a bug.

---

## Environment variables

| Variable | Purpose |
|---|---|
| `GOOGLE_WORKSPACE_CLI_TOKEN` | Supply a token directly |
| `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` | Override the credentials path |
| `GOOGLE_WORKSPACE_CLI_CLIENT_ID` | OAuth client ID |
| `GOOGLE_WORKSPACE_CLI_CLIENT_SECRET` | OAuth client secret |
| `GOOGLE_WORKSPACE_CLI_CONFIG_DIR` | Override `~/.config/gws` |
| `GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND` | Set to `file` for headless environments with no OS keyring |
| `GOOGLE_WORKSPACE_PROJECT_ID` | GCP project override |

On a headless box or in CI, set `GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file`.

---

## Security checklist

- **Never** paste your OAuth Client Secret into public GitHub issues, shell history you
  publish, shared docs, or chat transcripts.
- Protect `~/.config/gws`, especially `credentials.enc` and `client_secret.json`, with
  normal Unix permissions.
- **Never commit** `client_secret.json` or `credentials.enc` to a Git repository.
- Request the **minimum scopes** you need. This setup requested Drive + identity +
  cloud-platform.
- For AI agents: a gws command with write/delete/share permission makes **real changes to a
  real Google account**. Review destructive commands before running.
- Use `--dry-run` where supported, and check `--help` or `gws schema <method>` before
  invoking an unfamiliar method.

Client **IDs** are identifiers and are not secret. Client **Secrets** and credential files are.
