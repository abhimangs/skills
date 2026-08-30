# Appendix — verbatim terminal log

The actual machine state reached during setup, preserved in order. Useful for recognising
an error by its exact wording.

> OAuth **Client IDs** are identifiers and appear here safely. **Client Secrets** and
> credential files were never printed and must stay private.

---

### Install the npm package

```console
$ npm install -g @googleworkspace/cli
added 1 package in 1s
npm warn install-scripts 1 package had install scripts blocked because they are not covered by allowScripts:
npm warn install-scripts   @googleworkspace/cli@0.22.5 (postinstall: node install.js)
```

### Native binary auto-install on first run

```console
gws binary not found at /home/abhi/.npm-global/lib/node_modules/@googleworkspace/cli/bin/gws
Auto-installing...
Downloading gws from GitHub Releases
Verifying checksum ...
Checksum verified ✓
Extracting ...
gws v0.22.5 has been installed!
```

### First `gws auth setup` — gcloud missing

```json
{
  "error": {
    "code": 400,
    "message": "gcloud CLI not found. Install it from https://cloud.google.com/sdk/docs/install",
    "reason": "validationError"
  }
}
```

### After installing gcloud — confirm the account

```console
$ gcloud auth list
Credentialed Accounts
ACTIVE  ACCOUNT
*       abhimanhq@gmail.com
```

### Setup attempt 2 — project ID taken

```console
gws auth setup
...
Failed to create project 'testing123' because the ID is already in use.
```

### Setup attempt 3 — Cloud ToS not accepted

```console
gws auth setup
...
Failed to create project 'abhiman-gws-2026' because the active gcloud account
has not accepted Google Cloud Terms of Service.
```

### OAuth consent blocked

```
Access blocked: gws-local has not completed the Google verification process
The app is currently being tested, and can only be accessed by developer-approved testers.
Error 403: access_denied
```

### Setup succeeds

```console
gws auth setup
...
Step 1/5: gcloud CLI — found
Step 2/5: Authentication — abhimanhq@gmail.com
Step 3/5: GCP project
...
Setup complete! Starting `gws auth login`...
```

```json
{
  "account": "abhimanhq@gmail.com",
  "apis_enabled": 0,
  "apis_failed": [],
  "apis_skipped": 1,
  "client_config": "/home/abhi/.config/gws/client_secret.json",
  "message": "Setup complete! Starting `gws auth login`...",
  "project": "cli-arch",
  "status": "success"
}
```

### OAuth login succeeds

```console
Authentication successful. Encrypted credentials saved.
credentials_file: /home/abhi/.config/gws/credentials.enc
encryption: AES-256-GCM (key in OS keyring or local `.encryption_key`)
```

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

### First real Drive call

```console
$ gws drive --help
$ gws schema drive.files.list
$ gws drive files list --params '{"pageSize":10}'
```
