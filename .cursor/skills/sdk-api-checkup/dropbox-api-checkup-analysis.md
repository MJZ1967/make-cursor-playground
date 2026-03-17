# Dropbox app (v5) — Code analysis vs API documentation

**App:** dropbox v5 (IPME: 5.10.10)  
**Analysis date:** 2026-03-17  
**Reference:** [Dropbox developers](https://dropbox.tech/developers), [Certificate changes 2026](https://dropbox.tech/developers/api-server-certificate-changes)

---

## 1. App structure (from downloaded code)

| Layer | Details |
|-------|---------|
| **Base** | `baseUrl`: `https://api.dropboxapi.com/2`; Bearer auth; header `dropbox-api-path-root`: `pathRootJsonString(connection.root_namespace_id)` |
| **Connections** | 2: `dropbox` (personal, require_role), `dropbox2` (offline token, scopes: account_info.read + params) |
| **Modules** | 22 (trigger: watchFiles2; actions: list, create, delete, move, copy, restore, search, share, file requests, upload, etc.) |
| **RPCs** | 8 (listFolders, listFiles, listFolderIDs, listFolderForMovingTo, listFileRequests, listShareLinks, searchFolders, paramUpdateRevokeShareLink) |
| **Functions** | 25 (pathRootJsonString, upload session helpers, share link helpers, etc.) |

---

## 2. API surface used

**Hosts:**

- `https://api.dropboxapi.com/2` — RPC-style JSON (base + most modules)
- `https://api.dropboxapi.com/oauth2/token` — OAuth token/refresh
- `https://www.dropbox.com/oauth2/authorize` — OAuth authorize
- `https://content.dropboxapi.com/2` — download, upload, upload_session (binary)

**Endpoints (summary):**

- **Auth:** `/oauth2/token`, `/2/users/get_current_account`, `/2/auth/token/revoke`
- **Files:** `/files/list_folder`, `list_folder/continue`, `list_revisions`, `create_folder_v2`, `delete_v2`, `move_v2`, `copy_v2`, `restore`, `search_v2`, `search/continue_v2`
- **Content:** `content.dropboxapi.com`: `/2/files/download`, `/2/files/upload`, `/2/files/upload_session/start`, `append_v2`, `finish`
- **Sharing:** `/sharing/list_shared_links`, `create_shared_link_with_settings`, `modify_shared_link_settings`, `revoke_shared_link`, `get_folder_metadata`
- **File requests:** `/file_requests/create`, `get`, `list_v2`, `list/continue`, `update`, `delete`, `delete_all_closed`

**Auth / identity:**

- OAuth2 (authorize + token + refresh for dropbox2); Bearer in `Authorization`
- `root_namespace_id` from `get_current_account` → `dropbox-api-path-root` (team/personal root); built via `pathRootJsonString()`.

---

## 3. Impact vs announcements

### 3.1 Dropbox API server root certificate changes (2026)

- **Announcement:** Certificates issued from a new root on or after 2026-01-01. [Source](https://dropbox.tech/developers/api-server-certificate-changes).
- **SDK impact:** Only official SDKs that **pin certificates** need updates (.NET, Java, older Python). Our app does **not** use any Dropbox SDK; it uses direct HTTPS to `api.dropboxapi.com` and `content.dropboxapi.com`.
- **Conclusion:** No app code change. Relies on platform TLS/CA store. **Action:** Before/after Jan 2026, verify Make runtime (Node/OpenSSL) trusts the new root (OS or bundled CAs). Document in ticket; no code change in IMLJSON/functions.

### 3.2 API updates to better support team spaces

- **Announcement:** API updates for team spaces (Nov 2023); “Listing the contents of all team-accessible namespaces” (Mar 2024).
- **Current usage:** App already uses `root_namespace_id` and `dropbox-api-path-root` for list/create/delete/move/copy/restore/search and sharing. No explicit “list team-accessible namespaces” call in current code.
- **Conclusion:** **Possible** impact if Dropbox changes behavior of `list_folder` / path_root for team spaces or adds new requirements. **Action:** Review [Dropbox team spaces docs](https://dropbox.tech/developers) when doing manual check; confirm existing path_root usage still matches API spec.

### 3.3 Customizing scopes in the OAuth app authorization flow

- **Announcement:** Customizing scopes in OAuth flow (Sep 2024).
- **Current usage:**  
  - `dropbox`: authorize with `scope: join(oauth.scope, ',')`, `require_role: "personal"`.  
  - `dropbox2`: `scope: merge(oauth.scope, parameters.scopes)`, `token_access_type: "offline"`.  
  Connection scope configs: dropbox `[]`, dropbox2 `["account_info.read"]`.
- **Conclusion:** **Possible** impact if Dropbox renamed scopes or added required scopes. **Action:** Confirm current scope sets still match [Dropbox OAuth docs](https://www.dropbox.com/developers/documentation/http/documentation#authorization); adjust connection scopes if needed.

### 3.4 Minor: invalidate URL in connection `dropbox`

- **Finding:** `connections/dropbox/api.imljson` → `invalidate.url`: `"api.dropboxapi.com/2/auth/token/revoke"` (no `https://`).  
- **Risk:** Some clients may treat as relative or fail. dropbox2 correctly uses `"https://api.dropboxapi.com/2/auth/token/revoke"`.
- **Suggestion:** Use `"https://api.dropboxapi.com/2/auth/token/revoke"` in dropbox connection for consistency and reliability.

---

## 4. Scope (components to keep in mind)

**Modules (22):** copyFileFolder, createFileRequest, createFolder, createPaperDoc, createShareLink, deleteFile, deleteFileRequests, getAFolderMetadata, getFile, getFileRequest, listAllFilesSubfoldersInFolder, listFileRevisions, listFileRequests, makeApiCall, moveFileFolder, renameFileFolder, restoreFile, searchFilesFolders, updateFileRequest, updateShareLink, uploadLargeFile, watchFiles2.

**RPCs (8):** listFiles, listFolderForMovingTo, listFolderIDs, listFolders, listFileRequests, listShareLinks, paramUpdateRevokeShareLink, searchFolders.

**Connections (2):** dropbox, dropbox2.

**Relevant functions:** pathRootJsonString (path_root for API), upload session and share-link helpers.

---

## 5. Summary

| Topic | Impact | Code change needed? |
|-------|--------|----------------------|
| Certificate change 2026 | Platform TLS only | No (verify runtime CA store) |
| Team spaces API | Possible behavior/contract change | Maybe (review path_root / list_folder) |
| OAuth scopes | Possible scope name/requirements change | Maybe (review connection scopes) |
| invalidate URL (dropbox) | Bug/low risk | Yes (add https://) |

**Overall:** No mandatory code change for certificate change. Recommend: (1) add https:// to revoke URL in dropbox connection, (2) manual review of team spaces and OAuth scope docs, (3) re-test after Jan 2026 for TLS.
