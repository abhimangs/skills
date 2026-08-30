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
| `-o, --output <PATH>` | Output file path for binary or downloaded responses |
| `--page-all` | Auto-paginate through all results (NDJSON format) |
| `--page-limit <N>` | Max pages to fetch when using `--page-all` (default: 10) |
| `--page-delay <MS>` | Delay in ms between page fetches (default: 100) |

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

---

## Resource Commands: `files`

Core operations on the Google Drive `files` resource.

### 1. `files list`
List or search files and folders using Drive query syntax.

```bash
gws drive files list [--params '<JSON>'] [--page-all]
```

**Common Query Parameters (`--params`):**
- `q`: Search query string (e.g. `'mimeType = "application/pdf"'`, `'name contains "report"'`, `'trashed = false'`, `"'<FOLDER_ID>' in parents"`)
- `pageSize`: Number of files to return (default: 100, max: 1000)
- `fields`: Comma-separated list of fields (e.g. `"nextPageToken, files(id, name, mimeType, size, createdTime)"`)
- `orderBy`: Sort order (e.g. `"createdTime desc"`, `"folder, modifiedTime desc"`)

**Examples:**
```bash
# List first 10 files
gws drive files list --params '{"pageSize": 10}'

# Search for folders only
gws drive files list --params '{"q": "mimeType = \"application/vnd.google-apps.folder\" and trashed = false"}'

# Search files in a specific folder
gws drive files list --params '{"q": "\"16FJQO8X943neIFkBu_bKQ5JqIKHY05gw\" in parents and trashed = false"}'
```

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

# Download binary content directly
gws drive files get --params '{"fileId": "1BeiAlDrolwQC0hHY2kf6-DdKRAG1eMGk", "alt": "media"}' -o output.json
```

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

# Upload media with specific metadata
gws drive files create --json '{"name": "Document.docx", "parents": ["FOLDER_ID"]}' --upload ./Document.docx
```

---

### 4. `files download`
Download a file's binary stream (valid 24h).

```bash
gws drive files download --params '{"fileId": "<FILE_ID>"}' -o <OUTPUT_PATH>
```

**Example:**
```bash
gws drive files download --params '{"fileId": "1x3rJ0WLS5rlZKP4eZyWuy8FKAbhZayOe"}' -o ./video.mp4
```

---

### 5. `files export`
Export a Google Docs / Sheets / Slides document to standard formats (PDF, DOCX, CSV, etc.).

```bash
gws drive files export --params '{"fileId": "<DOC_ID>", "mimeType": "application/pdf"}' -o exported.pdf
```

---

### 6. `files update`
Update a file's metadata, contents, or move between folders.

```bash
gws drive files update --params '{"fileId": "<FILE_ID>", "addParents": "<NEW_FOLDER>", "removeParents": "<OLD_FOLDER>"}' --json '{"name": "Renamed File.pdf"}'
```

---

### 7. `files copy`
Create a copy of an existing file.

```bash
gws drive files copy --params '{"fileId": "<SOURCE_FILE_ID>"}' --json '{"name": "Copy of File.pdf"}'
```

---

### 8. `files delete` & `files emptyTrash`
Permanently delete a file or empty the trash.

```bash
# Delete a file permanently
gws drive files delete --params '{"fileId": "<FILE_ID>"}'

# Empty the user's trash
gws drive files emptyTrash
```

> [!CAUTION]
> `files delete` permanently purges the file. To move to trash instead, use `files update` with `--json '{"trashed": true}'`.

---

## Resource Commands: `permissions`

Manage access control and sharing on files and shared drives.

### `permissions list`
```bash
gws drive permissions list --params '{"fileId": "<FILE_ID>"}'
```

### `permissions create`
Share a file or folder with a user, group, domain, or anyone.

```bash
# Share with a user as viewer/reader
gws drive permissions create --params '{"fileId": "<FILE_ID>"}' --json '{"role": "reader", "type": "user", "emailAddress": "user@example.com"}'

# Make public (anyone with link can view)
gws drive permissions create --params '{"fileId": "<FILE_ID>"}' --json '{"role": "reader", "type": "anyone"}'
```

### `permissions delete`
```bash
gws drive permissions delete --params '{"fileId": "<FILE_ID>", "permissionId": "<PERMISSION_ID>"}'
```

---

## Resource Commands: `drives` (Shared Drives)

Manage Google Workspace Shared Drives (Team Drives).

```bash
# List shared drives
gws drive drives list

# Create a shared drive
gws drive drives create --json '{"name": "Engineering Team Drive"}'

# Get shared drive info
gws drive drives get --params '{"driveId": "<DRIVE_ID>"}'
```

---

## Resource Commands: `comments` & `replies`

Manage comments and discussions on Drive items.

```bash
# List comments on a file (fields parameter is required)
gws drive comments list --params '{"fileId": "<FILE_ID>", "fields": "*"}'

# Add a comment
gws drive comments create --params '{"fileId": "<FILE_ID>", "fields": "*"}' --json '{"content": "Please review this draft."}'
```

---

## Summary & Safety Rules

- Always verify file IDs and parameters before running destructive operations (`files delete`, `emptyTrash`).
- To test queries without altering data, utilize `--dry-run`.
- Format JSON parameters strictly with valid JSON syntax (`--params '{"key": "value"}'`).
