---
name: gws-drive
description: "Google Drive: Comprehensive command suite and API resource management via the official gws CLI."
metadata:
  version: "0.22.5"
  openclaw:
    category: "productivity"
    requires:
      bins:
        - gws
    cliHelp: "gws drive --help"
---

# gws drive

> **PREREQUISITE:** Ensure authentication is set up with `gws auth setup` and `gws auth login`. Encrypted credentials are saved automatically.

The `gws drive` command suite interacts with the Google Drive v3 API to manage files, folders, permissions, comments, shared drives, and revisions.

---

## Global Options

The following options are supported across `gws drive` subcommands:

| Option | Description |
|---|---|
| `--format <FORMAT>` | Output format: `json` (default), `table`, `yaml`, `csv` |
| `--dry-run` | Validate the request locally without sending it to the API |
| `--sanitize <TEMPLATE>` | Sanitize responses via Model Armor (format: `projects/PROJECT/locations/LOCATION/templates/TEMPLATE`) |
| `-o, --output <PATH>` | Output file path for binary or downloaded responses — **must be inside current directory** (e.g. `./exported.txt`, not `/tmp/...`) |
| `--page-all` | Auto-paginate through all results (NDJSON format) |
| `--page-limit <N>` | Max pages to fetch when using `--page-all` (default: 10) |
| `--page-delay <MS>` | Delay in ms between page fetches (default: 100) |
| `--params '<JSON>'` | URL/Query parameters as JSON |
| `--json '<JSON>'` | Request body as JSON |
| `--upload <PATH>` | Local file to upload as media content |
| `--upload-content-type <MIME>` | MIME type override (auto-detected if omitted) |

---

## Helper Commands

### `+upload`
Uploads a local file with automatic MIME-type detection and metadata inference.

```bash
gws drive +upload <file> [--name <NAME>] [--parent <FOLDER_ID>]
```

**Options:**
- `<file>`: Path to local file to upload (Required)
- `--name <NAME>`: Target filename in Drive (defaults to local basename)
- `--parent <ID>`: Parent folder ID in Google Drive

**Examples:**
```bash
# Upload a file to root
gws drive +upload ./report.pdf

# Upload with custom name into a specific folder
gws drive +upload ./data.csv --name "Q3 Sales Data.csv" --parent 16FJQO8X943neIFkBu_bKQ5JqIKHY05gw
```

**Tested:** upload to folder `1dFk_KvuCbqPf3iMGJQgeGZt1doMaY1YU` succeeded; returns `id`, `name`, `mimeType`.

---

## Resource Commands: `about`

Get user, Drive, and system capabilities. `fields` param is **required**.

```bash
gws drive about get --params '{"fields": "user, storageQuota"}'
```

**Tested:** `2026-08-30` returns `user.emailAddress=abhimanhq@gmail.com`, `storageQuota.limit=5497558138880`, `usageInDrive=0` after deletions.

---

## Resource Commands: `files`

Core operations on the Google Drive `files` resource.

### 1. `files list`
List or search files and folders using Drive query syntax.

```bash
gws drive files list [--params '<JSON>'] [--page-all]
```

**Common Query Parameters (`--params`):**
- `q`: Search query string (e.g. `'mimeType = "application/pdf"'`, `'name contains "report"'`, `'trashed = false'`, `"'<FOLDER_ID>' in parents"`, `'"me" in owners'`)
- `pageSize`: Number of files to return (default: 100, max: 1000)
- `fields`: Comma-separated list of fields (e.g. `"nextPageToken, files(id, name, mimeType, size, createdTime)"`)
- `orderBy`: Sort order (e.g. `"createdTime desc"`, `"folder, modifiedTime desc"`)
- `corpora`, `includeItemsFromAllDrives`, `supportsAllDrives` for shared drives

**Examples:**
```bash
# List first 10 files
gws drive files list --params '{"pageSize": 10}'

# Search for folders only
gws drive files list --params '{"q": "mimeType = \"application/vnd.google-apps.folder\" and trashed = false"}'

# Search files in a specific folder
gws drive files list --params '{"q": "\"16FJQO8X943neIFkBu_bKQ5JqIKHY05gw\" in parents and trashed = false"}'

# List owned files only
gws drive files list --params '{"q": "trashed = false and \"me\" in owners"}' --page-all
```

**Tested:** `pageSize`/`q`/`fields` work; `--page-all` returns NDJSON; after mass deletion `owned by me = 0`, `total trashed=false = 2` (shared docs).

---

### 2. `files get`
Retrieve file metadata or download file content by file ID.

```bash
gws drive files get --params '{"fileId": "<FILE_ID>", ...}'
```

**Examples:**
```bash
# Get file metadata
gws drive files get --params '{"fileId": "1BeiAlDrolwQC0hHY2kf6-DdKRAG1eMGk", "fields": "id,name,mimeType,size,createdTime"}'

# Download binary content directly (output MUST be inside cwd)
gws drive files get --params '{"fileId": "1BeiAlDrolwQC0hHY2kf6-DdKRAG1eMGk", "alt": "media"}' -o ./output.json
```

**Tested:** `fields` filtering works; `alt=media` + `-o ./file` works only inside cwd (`/tmp` fails with `validationError: outside current directory`).

---

### 3. `files create`
Create a new file, folder, or upload content with specific metadata.

```bash
gws drive files create [--json '<JSON>'] [--upload <PATH>] [--upload-content-type <MIME>]
```

**Examples:**
```bash
# Create a folder
gws drive files create --json '{"name": "New Folder", "mimeType": "application/vnd.google-apps.folder"}'

# Create a Google Doc
gws drive files create --json '{"name": "gws-test-doc", "mimeType": "application/vnd.google-apps.document"}'

# Upload media with specific metadata
gws drive files create --json '{"name": "Document.docx", "parents": ["FOLDER_ID"]}' --upload ./Document.docx
```

**Tested:** folder `gws-test-folder-*` and doc `1g-4MXhKhYmu5HBuadu7AKj_YCn81RuCRKsTD4WhoUuE` created successfully.

---

### 4. `files download` & `files export` & `files get` (media)
Download/export byte content.

```bash
gws drive files download --params '{"fileId": "<FILE_ID>"}' -o ./<OUTPUT_PATH>
gws drive files export --params '{"fileId": "<DOC_ID>", "mimeType": "application/pdf"}' -o ./exported.pdf
```

**Examples:**
```bash
gws drive files download --params '{"fileId": "1x3rJ0WLS5rlZKP4eZyWuy8FKAbhZayOe"}' -o ./video.mp4
gws drive files export --params '{"fileId": "1g-4MXhKhYmu5HBuadu7AKj_YCn81RuCRKsTD4WhoUuE", "mimeType": "text/plain"}' -o ./exported.txt
```

**Tested:** `export` to `./exported.txt` succeeded (3 bytes); `download` can return `500 backendError` transiently; both require `-o` inside cwd.

---

### 5. `files update`
Update a file's metadata, contents, or move between folders. Trash/untrash via `trashed`.

```bash
gws drive files update --params '{"fileId": "<FILE_ID>", "addParents": "<NEW_FOLDER>", "removeParents": "<OLD_FOLDER>"}' --json '{"name": "Renamed File.pdf"}'
```

**Examples:**
```bash
# Rename
gws drive files update --params '{"fileId": "11r1aQOIBU35sHdPNRF2htdpAnHXeJngj"}' --json '{"name": "gws-test-file-renamed.txt"}'

# Move to trash (recoverable)
gws drive files update --params '{"fileId": "<FILE_ID>"}' --json '{"trashed": true}'

# Update content
gws drive files update --params '{"fileId": "<FILE_ID>"}' --upload ./new-content.txt
```

**Tested:** rename succeeded; `trashed:true` moves to trash (30d recoverable) unlike `delete`.

---

### 6. `files copy`
Create a copy of an existing file.

```bash
gws drive files copy --params '{"fileId": "<SOURCE_FILE_ID>"}' --json '{"name": "Copy of File.pdf"}'
```

**Tested:** copy `11r1aQOIBU35sHdPNRF2htdpAnHXeJngj` → `1ednl8d-E6q-pKaHg69ZGEB6zm5ybnTVy` succeeded; appears in same parent folder.

---

### 7. `files delete` & `files emptyTrash`
Permanently delete a file or empty the trash.

```bash
# Delete a file permanently (must be owned by you)
gws drive files delete --params '{"fileId": "<FILE_ID>"}'

# Empty the user's trash
gws drive files emptyTrash
```

> [!CAUTION]
> `files delete` permanently purges the file. To move to trash instead, use `files update` with `--json '{"trashed": true}'`.
> Non-owned files (`ownedByMe=false`) return `403 insufficientFilePermissions` — you can only remove your permission via `permissions delete`, not delete the file itself. Verified: 60 owned deletes succeeded, 3 shared docs failed with 403.

---

### 8. `files generateIds`
Generates file IDs for create/copy requests.

```bash
gws drive files generateIds --params '{"count": 2}'
```

**Tested:** returns `ids: ["1CB-K2h0LjHxDDaeO8SIkNs15yyJNtRHL", ...]`, `kind=drive#generatedIds`.

---

### 9. `files listLabels` & `files modifyLabels`
Manage Drive Labels.

```bash
gws drive files listLabels --params '{"fileId": "<FILE_ID>"}'
gws drive files modifyLabels --params '{"fileId": "<FILE_ID>"}' --json '{"labelModifications": [...]}'
```

**Tested:** `listLabels` returns `labels: []`; `modifyLabels` requires `labelModifications` array — `setLabel` fails schema validation.

---

### 10. `files watch` & `files generateCseToken`
Subscribe to file changes / CSE.

```bash
gws drive files watch --params '{"fileId": "<FILE_ID>"}' --json '{"id": "test-id", "type": "web_hook", "address": "https://example.com"}'
gws drive files generateCseToken --params '{"fileId": "<FILE_ID>"}'
```

**Tested:** both support `--dry-run` (no live call); watch requires `id`, `type`, `address`.

---

## Resource Commands: `permissions`

Manage access control and sharing on files and shared drives.

### `permissions list`
```bash
gws drive permissions list --params '{"fileId": "<FILE_ID>", "fields": "permissions(id, type, role)"}'
```

### `permissions create`
Share a file or folder with a user, group, domain, or anyone.

```bash
# Share with a user as viewer/reader
gws drive permissions create --params '{"fileId": "<FILE_ID>"}' --json '{"role": "reader", "type": "user", "emailAddress": "user@example.com"}'

# Make public (anyone with link can view)
gws drive permissions create --params '{"fileId": "<FILE_ID>"}' --json '{"role": "reader", "type": "anyone"}'

# Share as commenter
gws drive permissions create --params '{"fileId": "<FILE_ID>"}' --json '{"role": "commenter", "type": "anyone"}'
```

### `permissions get`
```bash
gws drive permissions get --params '{"fileId": "<FILE_ID>", "permissionId": "<PERMISSION_ID>", "fields": "*"}'
```

### `permissions update`
```bash
gws drive permissions update --params '{"fileId": "<FILE_ID>", "permissionId": "<PERMISSION_ID>"}' --json '{"role": "commenter"}'
```

### `permissions delete`
```bash
gws drive permissions delete --params '{"fileId": "<FILE_ID>", "permissionId": "<PERMISSION_ID>"}'
```

**Tested:** full lifecycle on `11r1aQOIBU35sHdPNRF2htdpAnHXeJngj`: create `anyoneWithLink/reader` → list → get → update to `commenter` → delete. All succeeded. Deleting `anyoneWithLink` removes public sharing. For non-owned files, `permissions delete` with your own `permissionId` removes you from the file; but if you lack `permissions list` scope (see 2 docs owned by others), you get `403 insufficientFilePermissions` — remove via Drive UI.

---

## Resource Commands: `drives` (Shared Drives)

Manage Google Workspace Shared Drives (Team Drives).

```bash
# List shared drives (empty in test account)
gws drive drives list

# Create a shared drive (requestId required via --params)
gws drive drives create --params '{"requestId": "uuid-here"}' --json '{"name": "Engineering Team Drive"}'

# Get / update / hide / unhide / delete
gws drive drives get --params '{"driveId": "<DRIVE_ID>"}'
gws drive drives update --params '{"driveId": "<DRIVE_ID>"}' --json '{"name": "New Name"}'
gws drive drives hide --params '{"driveId": "<DRIVE_ID>"}'
gws drive drives unhide --params '{"driveId": "<DRIVE_ID>"}'
gws drive drives delete --params '{"driveId": "<DRIVE_ID>"}'
```

**Tested:** `drives list` returns `drives: []`; `create` without `requestId` fails `validationError: Required parameter requestId is missing`; `--dry-run` validates locally. `hide`/`unhide`/`update`/`delete` require `organizer` role and empty drive for delete.

---

## Resource Commands: `comments` & `replies`

Manage comments and discussions on Drive items.

### `comments`

```bash
# List comments on a file (fields parameter is required)
gws drive comments list --params '{"fileId": "<FILE_ID>", "fields": "*"}'

# Get / create / update / delete
gws drive comments get --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>", "fields": "*"}'
gws drive comments create --params '{"fileId": "<FILE_ID>", "fields": "*"}' --json '{"content": "Please review this draft."}'
gws drive comments update --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>", "fields": "*"}' --json '{"content": "Updated content"}'
gws drive comments delete --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>"}'
```

### `replies` (nested under comments)

```bash
gws drive replies list --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>", "fields": "*"}'
gws drive replies get --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>", "replyId": "<REPLY_ID>", "fields": "*"}'
gws drive replies create --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>", "fields": "*"}' --json '{"content": "Reply text"}'
gws drive replies update --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>", "replyId": "<REPLY_ID>", "fields": "*"}' --json '{"content": "Updated reply"}'
gws drive replies delete --params '{"fileId": "<FILE_ID>", "commentId": "<COMMENT_ID>", "replyId": "<REPLY_ID>"}'
```

**Tested:** full lifecycle on `11r1aQOIBU35sHdPNRF2htdpAnHXeJngj`: comment `AAACGOGelyw` create → list → get → update (`content` → "test comment updated") → reply `AAACGOGely0` create → list → get → update → delete → comment delete. All succeeded. Field `fields="*"` is **required** or API returns error.

---

## Resource Commands: `revisions`

Manage file version history (binary files like images/videos retain revisions).

```bash
gws drive revisions list --params '{"fileId": "<FILE_ID>", "fields": "revisions(id, mimeType)"}'
gws drive revisions get --params '{"fileId": "<FILE_ID>", "revisionId": "<REVISION_ID>", "fields": "*"}'
gws drive revisions update --params '{"fileId": "<FILE_ID>", "revisionId": "<REVISION_ID>"}' --json '{"keepForever": true}'
gws drive revisions delete --params '{"fileId": "<FILE_ID>", "revisionId": "<REVISION_ID>"}'
```

**Tested:** on `11r1aQOIBU35sHdPNRF2htdpAnHXeJngj` → list returned 1 revision `0BymAXG_k-xEtK1R6dzFjbXhPTytiVFppcXpIVEs3YmxXbkhRPQ` with `md5Checksum`, `size=47`; get/update succeeded.

---

## Resource Commands: `about`, `changes`, `channels`, `operations`

```bash
# About (fields required)
gws drive about get --params '{"fields": "user, storageQuota"}'

# Changes (retrieve Drive changes)
gws drive changes getStartPageToken
gws drive changes list --params '{"pageToken": "<TOKEN>", "pageSize": 10}'
gws drive changes watch --params '{"pageToken": "<TOKEN>"}' --json '{"id": "...", "type": "web_hook", "address": "https://..."}'

# Channels
gws drive channels stop --json '{"id": "test", "resourceId": "test"}'

# Operations (long-running)
gws drive operations get --params '{"name": "operations/<ID>"}'
```

**Tested:** `about get` OK; `changes getStartPageToken` → `1404`; `changes list` with token → `changes:[]`; `channels stop` with invalid channel → `404 Channel not found`; `operations get` supports `--dry-run`.

---

## Resource Commands: `apps`, `accessproposals`, `approvals`, `teamdrives`

```bash
# Apps (installed Drive apps)
gws drive apps list
gws drive apps get --params '{"appId": "<APP_ID>"}'

# Access Proposals (pending access requests)
gws drive accessproposals list --params '{"fileId": "<FILE_ID>"}'
gws drive accessproposals get --params '{"fileId": "<FILE_ID>", "proposalId": "<ID>"}'
gws drive accessproposals resolve --params '{"fileId": "<FILE_ID>", "proposalId": "<ID>"}' --json '{"action": "approve"}'

# Approvals (file approval workflows)
gws drive approvals list --params '{"fileId": "<FILE_ID>"}'
gws drive approvals get --params '{"fileId": "<FILE_ID>", "approvalId": "<ID>"}'
gws drive approvals start --params '{"fileId": "<FILE_ID>"}' --json '{"reviewers": [...]}'
gws drive approvals approve --params '{"fileId": "<FILE_ID>", "approvalId": "<ID>"}' --json '{}'
gws drive approvals decline --params '{"fileId": "<FILE_ID>", "approvalId": "<ID>"}' --json '{}'
gws drive approvals cancel --params '{"fileId": "<FILE_ID>", "approvalId": "<ID>"}' --json '{}'
gws drive approvals comment --params '{"fileId": "<FILE_ID>", "approvalId": "<ID>"}' --json '{"comment": "lgtm"}'
gws drive approvals reassign --params '{"fileId": "<FILE_ID>", "approvalId": "<ID>"}' --json '{"reviewers": [...]}'

# Team Drives (deprecated — use drives instead)
gws drive teamdrives list
gws drive teamdrives get --params '{"teamDriveId": "<ID>"}'
```

**Tested:** `apps list` → `403 insufficient authentication scopes` (expected without scope); `approvals list` → empty `approvalList`; `accessproposals list` / `channels stop` tested; `teamdrives list` returns `teamDrives:[]` but marked deprecated.

---

## Summary & Safety Rules

- Always verify file IDs and parameters before running destructive operations (`files delete`, `emptyTrash`, `revisions delete`, `drives delete`). Non-owned files cannot be deleted — use `permissions delete` to leave them.
- `--output` must be inside current directory (`./file`), not `/tmp` — validationError otherwise.
- `fields` is required for `about get`, `comments`, `replies`.
- `requestId` (UUID) required via `--params` for `drives create`.
- To test queries without altering data, utilize `--dry-run` (supports `files watch`, `operations get`, `generateCseToken`, etc.).
- Format JSON parameters strictly with valid JSON syntax (`--params '{"key": "value"}'`).
- Shared drives must be empty before `drives delete` and you must be `organizer`.
- After mass deletion, `storageQuota.usageInDrive` drops to `0` (verified).
