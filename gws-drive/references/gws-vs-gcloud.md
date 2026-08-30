# gws vs gcloud vs rclone

Three different tools, three different planes. Confusing them is the single most common
source of wasted time in this setup.

| Tool | Command | Purpose | Role here |
|---|---|---|---|
| Google Workspace CLI | `gws` | Drive, Gmail, Calendar, Docs, Sheets, Slides, Chat, Apps Script | **Primary tool** for Workspace data/automation |
| Google Cloud CLI | `gcloud` | GCP projects, IAM, APIs, Cloud resources, quotas | Required by `gws auth setup`; also for actual Cloud work |
| rclone | `rclone` | File-oriented cloud storage copy/sync/mount | Optional — when you want Drive as a filesystem |

```
Your Linux machine
│
├── gcloud  ─────────────── Google Cloud control plane
│      ├── projects
│      ├── APIs
│      ├── IAM / credentials
│      ├── Cloud resources
│      └── quotas / configuration
│
├── gws  ───────────────── Google Workspace API control plane
│      ├── Drive
│      ├── Gmail
│      ├── Calendar
│      ├── Docs
│      ├── Sheets
│      ├── Slides
│      ├── Chat
│      └── Apps Script / other Workspace APIs
│
└── rclone (optional) ───── Storage/file-sync plane
       ├── copy
       ├── sync
       ├── mount
       └── bulk filesystem-style workflows
```

**The rule:** `gws` for API-level Workspace operations and automation. `rclone` when the job
is moving/syncing/mounting Drive storage. `gcloud` for Google Cloud infrastructure and
project/API administration.

---

## What gws is good at on Drive

- Listing and searching files/folders with Drive query syntax
- Creating folders and files through API methods
- Reading metadata — IDs, names, MIME types, parents, owners, permissions, timestamps
- Upload/download where the Drive method supports it
- Permission and sharing operations
- Combining Drive with Gmail, Calendar, Sheets, Docs, Slides, Chat and Apps Script in one
  automation
- Emitting JSON that shell scripts and AI agents can parse

## Why you may still want rclone

For bulk copy, synchronization, and mounting a remote as a filesystem path, rclone is far
more convenient than API calls.

```bash
rclone copy ~/Documents gdrive:Documents
rclone sync ~/Photos gdrive:Photos
rclone mount gdrive: ~/GoogleDrive
```

> **Care with `sync`:** by design it deletes files on the destination to make it match the
> source. Keep mount/copy/sync mentally separate from Workspace API operations.

---

## gcloud commands you actually need

```bash
gcloud auth list                    # list credentialed accounts
gcloud auth login                   # authenticate an account
gcloud config get-value account     # show the configured account
gcloud config list                  # inspect active configuration
gcloud projects list                # list accessible projects
gcloud projects create PROJECT_ID   # create a project (if permitted and ID is free)
gcloud services enable ...          # enable Google APIs/services
```

Everything else in gcloud — Compute Engine, IAM, networking, storage, GKE — is irrelevant
to gws unless you are separately doing Cloud work.

---

## The gws command surface

gws builds its commands **dynamically from Google's Discovery Service**, so don't guess
flags — inspect them:

```bash
gws drive --help
gws schema drive.files.list
```

General form:

```
gws <service> <resource> [sub-resource] <method> [flags]
```

Output formats: `json` (default), `table`, `yaml`, `csv`. `--params` carries URL/query
parameters, `--json` carries the request body.

---

## Why gws suits AI coding agents

The gws repository explicitly describes itself as built for humans **and AI agents**: it
emits structured JSON, derives its command surface from Discovery, and ships per-service
"skills" documents in-repo. That makes it a clean tool layer for terminal agents, given
shell access and authenticated credentials.

```
AI agent
  │
  ├── inspect: gws drive --help
  ├── inspect: gws schema drive.files.list
  ├── execute: gws drive files list --params '{...}'
  ├── parse JSON
  └── perform next approved action
```

Because gws performs writes and permission changes, treat it as a **privileged tool**: reads
are low-risk, writes/deletes/shares deserve explicit review.

---

## Sources

- gws repository — <https://github.com/googleworkspace/cli>
- gws shared/auth skill — <https://github.com/googleworkspace/cli/blob/main/skills/gws-shared/SKILL.md>
- gws package metadata — <https://github.com/googleworkspace/cli/blob/main/package.json>
- gws changelog — <https://github.com/googleworkspace/cli/blob/main/CHANGELOG.md>
- AUR `google-cloud-cli` — <https://aur.archlinux.org/packages/google-cloud-cli>
- Google Cloud CLI install — <https://docs.cloud.google.com/sdk/docs/install-sdk>
- Google Cloud CLI components — <https://docs.cloud.google.com/sdk/docs/components>
- Google Cloud Console — <https://console.cloud.google.com/>
