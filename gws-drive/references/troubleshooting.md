# Troubleshooting — every error hit during setup

Each entry is the verbatim error, what it actually meant, and the fix that worked.

---

## Quick matrix

| Symptom | Cause | Fix |
|---|---|---|
| `gws binary not found` | npm postinstall was blocked, or binary not installed yet | Just run `gws` — it auto-installs the matching release. Or reinstall allowing the script |
| `gcloud CLI not found` | `gws auth setup` needs gcloud for its project workflow | Install `google-cloud-cli` (AUR on Arch, Google APT repo on Debian/Ubuntu) |
| `project ID already in use` | GCP project IDs are globally unique | Choose a different, namespaced ID |
| `has not accepted Google Cloud Terms of Service` | The gcloud account never completed ToS acceptance | Open Cloud Console with that same account, accept, retry |
| Chrome Extension client created | Wrong OAuth client type for a local terminal flow | Create a **Desktop app** OAuth client |
| `Access blocked` / developer-approved testers | External app in Testing mode, account not a test user | Add the account as a **Test user** in OAuth audience settings |
| `401 invalid_client` after pasting Client ID | Tracked v0.22.5 bug — trailing whitespace in the saved Client ID | Paste with no trailing space; recreate the client config if needed |
| `auth login` succeeds but API calls say no credentials | Credential lookup/storage or account association problem | Rerun `gws auth login`, check `gws auth` subcommands, verify config dir / credentials path |
| `403 insufficientFilePermissions` on delete | File is not owned by you (`ownedByMe=false`) | You cannot delete it. Remove your own access via `permissions delete` |
| `403 insufficient authentication scopes` on `apps list` | Drive Apps scope was never granted | Expected. Re-run setup requesting the additional scope if you need it |
| `validationError: outside current directory` on `-o` | `--output` is sandboxed to the working directory | Write to `./file`, never `/tmp/...` |
| `500 backendError` on `files download` | Transient Google-side failure | Retry |

---

## 1. npm postinstall blocked

```
added 1 package in 1s
npm warn install-scripts 1 package had install scripts blocked because they are not covered by allowScripts:
npm warn install-scripts   @googleworkspace/cli@0.22.5 (postinstall: node install.js)
```

**Not a failure.** The JS package installed; only the native-binary download was skipped.
gws installs the binary itself on first run. To allow the script instead:

```bash
npm install -g --allow-scripts=@googleworkspace/cli
npm config set allow-scripts=@googleworkspace/cli --location=user
```

## 2. gws binary missing → self-heals

```
gws binary not found at /home/abhi/.npm-global/lib/node_modules/@googleworkspace/cli/bin/gws
Auto-installing...
Downloading gws from GitHub Releases
Verifying checksum ...
Checksum verified ✓
Extracting ...
gws v0.22.5 has been installed!
```

Installation succeeded. **Any error after this point is not an installation problem.**

## 3. `gcloud CLI not found`

```json
{
  "error": {
    "code": 400,
    "message": "gcloud CLI not found. Install it from https://cloud.google.com/sdk/docs/install",
    "reason": "validationError"
  }
}
```

`gws auth setup` drives project creation and API enablement through gcloud. Install gcloud
first — this is a hard prerequisite, not an optional integration.

## 4. Project ID collision

```
Failed to create project 'testing123' because the ID is already in use.
```

GCP project IDs are unique across **all of Google Cloud**, not just your account. Generic
IDs are always taken. Use `abhiman-gws-2026` or similar.

## 5. Cloud Terms of Service not accepted

```
Failed to create project 'abhiman-gws-2026' because the active gcloud account
has not accepted Google Cloud Terms of Service.
```

**This is not a project-ID problem** — renaming again will not help. Sign in to
<https://console.cloud.google.com/> with the exact account from `gcloud auth list`, accept
the ToS, then rerun setup. On an organization-managed account, an org admin may also need
to enable Google Cloud for the domain.

## 6. Wrong OAuth client type

The "Create OAuth client ID" page was first filled in with application type
**Chrome Extension**. Wrong for a local terminal flow.

```
Application type:  Desktop app
Name:              gws-local
```

Desktop app is what supports the `http://localhost:44789` redirect gws uses.

## 7. `Access blocked` — 403 access_denied

```
Access blocked: gws-local has not completed the Google verification process
The app is currently being tested, and can only be accessed by developer-approved testers.
Error 403: access_denied
```

The External OAuth app is in **Testing** mode, and the authorizing account was not on the
Test users list. Add `abhimanhq@gmail.com` (or whichever account) as a Test user under the
OAuth consent / audience settings. For personal development this avoids needing full public
app verification just to authorize your own client.

## 8. Success

```
Step 1/5: gcloud CLI — found
Step 2/5: Authentication — abhimanhq@gmail.com
Step 3/5: GCP project
...
Setup complete! Starting `gws auth login`...
```

```
Authentication successful. Encrypted credentials saved.
credentials_file: /home/abhi/.config/gws/credentials.enc
encryption: AES-256-GCM (key in OS keyring or local `.encryption_key`)
```

---

## Drive-runtime errors (from live command testing)

| Command | Error | Meaning |
|---|---|---|
| `files delete` on a shared doc | `403 insufficientFilePermissions` | Not owned by you. 60 owned deletes succeeded; 3 shared docs failed. Use `permissions delete` to leave the file instead |
| `files get -o /tmp/x` | `validationError: outside current directory` | `--output` must be `./something` |
| `files download` | `500 backendError` | Transient; retry |
| `about get` with no `fields` | error | `fields` is **required** |
| `comments` / `replies` with no `fields` | error | `fields` is **required** — use `"*"` |
| `drives create` with no `requestId` | `validationError: Required parameter requestId is missing` | Pass a UUID via `--params` |
| `files modifyLabels` with `setLabel` | schema validation failure | Must be a `labelModifications` array |
| `apps list` | `403 insufficient authentication scopes` | Drive Apps scope not granted |
| `channels stop` on an invalid channel | `404 Channel not found` | Expected for a fake channel ID |
