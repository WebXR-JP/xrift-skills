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

## Config Parsers

### `parseWorldConfig(json: string): XriftWorldConfig`

Parses a `xrift.json` JSON string and returns a world configuration object. Throws `XriftSdkError` if the `"world"` key is missing.

```typescript
import { parseWorldConfig } from '@xrift/sdk';

const config = parseWorldConfig(json);
// config.type === 'world', config.distDir, config.name, config.physics, ...
```

### `parseItemConfig(json: string): XriftItemConfig`

Parses a `xrift.json` JSON string and returns an item configuration object. Throws `XriftSdkError` if the `"item"` key is missing.

```typescript
import { parseItemConfig } from '@xrift/sdk';

const config = parseItemConfig(json);
// config.type === 'item', config.distDir, config.name, config.permissions, ...
```

### `filterFiles(filePaths: string[], ignorePatterns: string[]): string[]`

Filters an array of file paths, excluding files that match any of the ignore patterns.

```typescript
import { filterFiles, DEFAULT_IGNORE_PATTERNS } from '@xrift/sdk';

const filtered = filterFiles(['scene.glb', '__federation_shared_abc.js'], DEFAULT_IGNORE_PATTERNS);
// ['scene.glb']
```

---

## Node.js Helpers

Available from `@xrift/sdk/node`. These read xrift.json, collect files from the dist directory, and upload in one call.

### `uploadWorldFromDirectory(dirPath: string, options): Promise<WorldUploadResult>`

```typescript
import { uploadWorldFromDirectory } from '@xrift/sdk/node';

const result = await uploadWorldFromDirectory('./my-project', {
  token: process.env.XRIFT_TOKEN!,
  worldId: 'optional-existing-id',
  onProgress: (p) => console.log(`${p.completed}/${p.total}`),
});
```

### `uploadItemFromDirectory(dirPath: string, options): Promise<ItemUploadResult>`

```typescript
import { uploadItemFromDirectory } from '@xrift/sdk/node';

const result = await uploadItemFromDirectory('./my-project', {
  token: process.env.XRIFT_TOKEN!,
});
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
