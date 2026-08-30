# Installation & Setup

`gws` is the **Google Workspace CLI** (`@googleworkspace/cli`, binary `gws`). It is a
Google-maintained open-source project in the `googleworkspace` GitHub org, **not an
officially supported Google product**, and it is under active development.

> There is an unrelated AUR package literally named `gws`. It is **not** this tool.
> `yay -S gws` is wrong. Install via npm.

`gws auth setup` **requires `gcloud`** to be installed and authenticated first — it drives
the GCP project / API / OAuth-client workflow through it. Install gcloud before gws.

---

## 1. Google Cloud CLI (`gcloud`)

### Arch Linux — AUR

```bash
yay -S google-cloud-cli
gcloud --version
gcloud auth list
gcloud config get-value account
```

Package: <https://aur.archlinux.org/packages/google-cloud-cli> — provides the gcloud core
with alpha/beta commands. It **conflicts with and replaces** the older `google-cloud-sdk`
package. (Verified at version `576.0.0-1`, page last updated 15 Jul 2026; versions drift.)

### Debian / Ubuntu — official Google APT repository

Distribution URI `https://packages.cloud.google.com/apt`, component `cloud-sdk`, package
`google-cloud-cli`.

```bash
sudo apt-get update
sudo apt-get install ca-certificates gnupg curl

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
  | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list

sudo apt-get update
sudo apt-get install google-cloud-cli
```

Use the `signed-by` keyring form on modern releases. Google documents a legacy `apt-key`
path only for older systems that lack `signed-by`.

Update path:

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install google-cloud-cli
```

**India note:** there is no India-specific Google Cloud APT mirror. The global
`packages.cloud.google.com` repository is correct for Indian Debian/Ubuntu machines.
Locale and time zone affect the system and OAuth UX, not the repository.

### What the APT repo resolves to

```
/etc/apt/sources.list.d/google-cloud-sdk.list
        │
        ▼
https://packages.cloud.google.com/apt
        │
        └── cloud-sdk main
                 │
                 ▼
          google-cloud-cli
                 │
                 ▼
               gcloud
```

This is not a "Debian version" of gcloud — it is the same CLI shipped through Debian
packaging. Only the install/update/component model differs.

### Install method comparison

| Method | Update / component model | Typical use |
|---|---|---|
| APT (Debian/Ubuntu) | APT manages the package; **gcloud component manager is disabled** | System-managed workstations/servers |
| AUR (Arch) | pacman/yay manages the package | Arch desktop/server |
| Standalone `tar.gz` | `gcloud components install ...` works for many components | Portable/manual installs |

> **Component caveat:** when gcloud came from APT or yum, `gcloud components install ...`
> will not work. Install split packages through the system package manager instead —
> `google-cloud-cli-app-engine-go`, `google-cloud-cli-cbt`,
> `google-cloud-cli-docker-credential-gcr`, and `kubectl` as its own package.
> <https://docs.cloud.google.com/sdk/docs/components>

---

## 2. Google Workspace CLI (`gws`)

```bash
npm install -g @googleworkspace/cli
gws --version
```

npm may block the package's `postinstall` script:

```
added 1 package in 1s
npm warn install-scripts 1 package had install scripts blocked because they are not covered by allowScripts:
npm warn install-scripts   @googleworkspace/cli@0.22.5 (postinstall: node install.js)
```

**This is not fatal.** The package still installs, and the first `gws` run detects the
missing native binary and installs it itself — downloading the matching Linux x86_64
GitHub release and verifying its SHA256:

```
gws binary not found at /home/abhi/.npm-global/lib/node_modules/@googleworkspace/cli/bin/gws
Auto-installing...
Downloading gws from GitHub Releases
Verifying checksum ...
Checksum verified ✓
Extracting ...
gws v0.22.5 has been installed!
```

To allow the postinstall script instead:

```bash
npm install -g --allow-scripts=@googleworkspace/cli
# or persist it:
npm config set allow-scripts=@googleworkspace/cli --location=user
```

---

## 3. Complete setup flow from scratch

1. Install gcloud (`yay -S google-cloud-cli` on Arch; APT repo on Debian/Ubuntu).
2. Authenticate gcloud with the account you intend to use: `gcloud auth login`.
3. Verify: `gcloud auth list` and `gcloud config get-value account`.
4. Install gws: `npm install -g @googleworkspace/cli`.
5. Run `gws auth setup`.
6. When prompted for a GCP project ID, pick a **globally unique lowercase** ID. Do not
   reuse a taken one like `testing123`.
7. If Google reports the account has not accepted Cloud Terms of Service, open the Cloud
   Console **with that same account**, accept, then rerun setup.
8. When creating the OAuth client manually, choose **Desktop app** — never Chrome Extension.
9. Configure OAuth consent/audience. For External + Testing, add your account as a **Test user**.
10. Paste the OAuth Client ID and Client Secret when prompted. **No trailing whitespace** —
    a tracked v0.22.5 bug turns trailing spaces into `invalid_client`.
11. Let gws launch `gws auth login` and complete the browser consent flow.
12. Confirm `Authentication successful. Encrypted credentials saved.`
13. Explore: `gws <service> --help` and `gws schema <method>`.

Successful terminal result:

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

`apis_enabled: 0` alone does **not** mean gws is broken — the command ended `status: success`
and OAuth succeeded. Diagnose per-API problems from the actual gws command and Google's
response, not from this counter.

---

## 4. Known-good state (this workstation)

| Item | Value | Meaning |
|---|---|---|
| gcloud account | `abhimanhq@gmail.com` | gcloud authenticated to the intended account |
| gws project | `cli-arch` | GCP project selected by successful setup |
| gws version | `0.22.5` | Native binary auto-installed via the npm flow |
| Client config | `~/.config/gws/client_secret.json` | OAuth client configuration |
| Credentials | `~/.config/gws/credentials.enc` | Encrypted OAuth credentials |
| Encryption | AES-256-GCM, `keyring` backend | gws credential-store design |
| Drive scope | `https://www.googleapis.com/auth/drive` | Full Drive access granted |

---

## 5. Recommended workstation setup (Arch)

```bash
# 1) Google Cloud CLI
yay -S google-cloud-cli
gcloud --version
gcloud auth list
gcloud config get-value account

# 2) Google Workspace CLI
npm install -g @googleworkspace/cli
gws --version

# 3) Authenticate / repair setup
gws auth setup

# 4) Inspect Drive
gws drive --help
gws schema drive.files.list

# 5) Optional: rclone for sync/mount workflows
sudo pacman -S rclone
rclone config
```
