# @xrift/sdk API Reference

## XriftClient

Entry point of the SDK.

```typescript
import { XriftClient } from '@xrift/sdk';

const client = new XriftClient({
  token: string;       // Required - API authentication token
  baseUrl?: string;    // Default: 'https://api.xrift.net'
  timeout?: number;    // Default: 30000 (ms)
});
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `worlds` | `WorldsApi` | World operations API |
| `items` | `ItemsApi` | Item operations API |

---

## WorldsApi

Access via `client.worlds`.

### `upload(files: UploadFile[], options: WorldUploadOptions): Promise<WorldUploadResult>`

Integrated upload flow: create → hash → get URLs → upload → complete.

**Parameters:**

- `files: UploadFile[]` — Files to upload
- `options: WorldUploadOptions` — Upload configuration

**WorldUploadOptions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `worldId` | `string` | No | Existing world ID (creates new if omitted) |
| `name` | `string` | Yes | World name |
| `description` | `string` | No | Description |
| `thumbnailPath` | `string` | No | Thumbnail file path (must be in files list) |
| `physics` | `PhysicsConfig` | No | Physics settings |
| `camera` | `CameraConfig` | No | Camera settings |
| `permissions` | `WorldPermissions` | No | Permission settings |
| `outputBufferType` | `OutputBufferType` | No | Output buffer type |
| `onProgress` | `(progress: UploadProgress) => void` | No | Progress callback |

**Returns: `WorldUploadResult`**

| Field | Type | Description |
|-------|------|-------------|
| `worldId` | `string` | World ID |
| `versionId` | `string` | Version ID |
| `versionNumber` | `number` | Version number |
| `contentHash` | `string` | Content hash (12-char hex) |
| `files` | `UploadFile[]` | Uploaded files |

### `create(): Promise<CreateWorldResponse>`

Creates a new world resource.

**Returns:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | World ID |
| `ownerId` | `string` | Owner user ID |
| `createdAt` | `string` | ISO date string |
| `updatedAt` | `string` | ISO date string |

### `getUploadUrls(worldId: string, request: WorldUploadUrlsRequest): Promise<WorldUploadUrlsResponse>`

Gets signed upload URLs for files.

**WorldUploadUrlsRequest:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | Yes | World name |
| `description` | `string` | No | Description |
| `thumbnailPath` | `string` | No | Thumbnail path |
| `physics` | `PhysicsConfig` | No | Physics config |
| `camera` | `CameraConfig` | No | Camera config |
| `permissions` | `WorldPermissions` | No | Permissions |
| `outputBufferType` | `OutputBufferType` | No | Buffer type |
| `contentHash` | `string` | Yes | Content hash |
| `fileSize` | `number` | Yes | Total file size in bytes |
| `files` | `Array<{path, contentType}>` | Yes | File metadata |

**Returns: `WorldUploadUrlsResponse`**

| Field | Type | Description |
|-------|------|-------------|
| `uploadUrls` | `SignedUrlResponse[]` | Signed URLs for each file |
| `versionId` | `string` | Version ID |
| `contentHash` | `string` | Content hash |
| `versionNumber` | `number` | Version number |

### `complete(worldId: string, versionId: string): Promise<CompleteWorldUploadResponse>`

Notifies the server that all files have been uploaded.

**Returns: `CompleteWorldUploadResponse`**

| Field | Type | Description |
|-------|------|-------------|
| `versionId` | `string` | Version ID |
| `worldId` | `string` | World ID |
| `name` | `string` | World name |
| `description` | `string` | Description |
| `contentHash` | `string` | Content hash |
| `fileSize` | `number` | Total file size |
| `status` | `string` | Upload status |
| `versionNumber` | `number` | Version number |
| `owner` | `{id, displayName}` | Owner info |
| `createdAt` | `string` | ISO date string |
| `updatedAt` | `string` | ISO date string |

---

## ItemsApi

Access via `client.items`. API structure mirrors WorldsApi.

### `upload(files: UploadFile[], options: ItemUploadOptions): Promise<ItemUploadResult>`

Integrated upload flow for items.

**ItemUploadOptions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `itemId` | `string` | No | Existing item ID (creates new if omitted) |
| `name` | `string` | Yes | Item name |
| `description` | `string` | No | Description |
| `thumbnailPath` | `string` | No | Thumbnail path |
| `permissions` | `ItemPermissions` | No | Permission settings |
| `onProgress` | `(progress: UploadProgress) => void` | No | Progress callback |

**Returns: `ItemUploadResult`**

| Field | Type | Description |
|-------|------|-------------|
| `itemId` | `string` | Item ID |
| `versionId` | `string` | Version ID |
| `versionNumber` | `number` | Version number |
| `contentHash` | `string` | Content hash |
| `files` | `UploadFile[]` | Uploaded files |

### `create(): Promise<CreateItemResponse>`

Creates a new item resource.

**Returns:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Item ID |
| `ownerId` | `string` | Owner user ID |
| `createdAt` | `string` | ISO date string |
| `updatedAt` | `string` | ISO date string |

### `getUploadUrls(itemId: string, request: ItemUploadUrlsRequest): Promise<ItemUploadUrlsResponse>`

Gets signed upload URLs for item files.

### `complete(itemId: string, versionId: string): Promise<CompleteItemUploadResponse>`

Notifies upload completion.

---

## Error Classes

### `XriftSdkError`

Base error class for all SDK errors.

```typescript
class XriftSdkError extends Error {
  name: 'XriftSdkError';
}
```

### `XriftApiError` extends `XriftSdkError`

Thrown when the API returns a non-OK response.

```typescript
class XriftApiError extends XriftSdkError {
  readonly statusCode: number;
  readonly responseBody?: unknown;
}
```

### `XriftAuthError` extends `XriftApiError`

Thrown on HTTP 401 (authentication failure).

```typescript
class XriftAuthError extends XriftApiError {
  // statusCode is always 401
}
```

### `XriftNetworkError` extends `XriftSdkError`

Thrown on network connectivity issues (DNS failure, timeout, etc.).

```typescript
class XriftNetworkError extends XriftSdkError {
  readonly cause?: Error;
}
```

---

## Utility Functions

### `calculateContentHash(files, configValues?): Promise<string>`

Calculates a SHA-256 content hash (first 12 hex characters) from file data and optional config values.

```typescript
import { calculateContentHash } from '@xrift/sdk';

const hash = await calculateContentHash(
  [{ remotePath: 'scene.glb', data: uint8Array }],
  { physics: { gravity: -9.8 } }
);
// Returns: '3a7f2b1c9d0e' (12 hex chars)
```

- Files are sorted by `remotePath` before hashing
- Config values are JSON-serialized with sorted keys
- Uses `node:crypto` in Node.js, Web Crypto API in browsers

### `getMimeType(filePath: string): string`

Returns the MIME type for a file based on its extension.

```typescript
import { getMimeType } from '@xrift/sdk';

getMimeType('scene.glb');     // 'model/gltf-binary'
getMimeType('texture.png');   // 'image/png'
getMimeType('audio.mp3');     // 'audio/mpeg'
getMimeType('unknown.xyz');   // 'application/octet-stream'
```

**Supported extensions:**

| Extension | MIME Type |
|-----------|----------|
| `.glb` | `model/gltf-binary` |
| `.gltf` | `model/gltf+json` |
| `.png` | `image/png` |
| `.jpg`, `.jpeg` | `image/jpeg` |
| `.webp` | `image/webp` |
| `.json` | `application/json` |
| `.js`, `.mjs` | `application/javascript` |
| `.html` | `text/html` |
| `.css` | `text/css` |
| `.txt` | `text/plain` |
| `.bin` | `application/octet-stream` |
| `.wasm` | `application/wasm` |
| `.svg` | `image/svg+xml` |
| `.mp3` | `audio/mpeg` |
| `.ogg` | `audio/ogg` |
| `.wav` | `audio/wav` |
| `.mp4` | `video/mp4` |
| `.webm` | `video/webm` |
| `.ktx2` | `image/ktx2` |
| `.basis`, `.hdr`, `.exr` | `application/octet-stream` |
