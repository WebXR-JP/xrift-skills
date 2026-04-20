# @xrift/sdk Type Definitions

## Client Configuration

```typescript
interface XriftClientConfig {
  token: string;
  baseUrl?: string;   // Default: 'https://api.xrift.net'
  timeout?: number;    // Default: 30000 (ms)
}
```

## Config Types

### XriftWorldConfig

```typescript
interface XriftWorldConfig {
  type: 'world';
  distDir: string;
  name: string;
  description?: string;
  thumbnailPath?: string;
  ignore: string[];
  physics?: PhysicsConfig;
  camera?: CameraConfig;
  permissions?: WorldPermissions;
  outputBufferType?: OutputBufferType;
}
```

### XriftItemConfig

```typescript
interface XriftItemConfig {
  type: 'item';
  distDir: string;
  name: string;
  description?: string;
  thumbnailPath?: string;
  ignore: string[];
  permissions?: ItemPermissions;
}
```

### XriftConfig

```typescript
type XriftConfig = XriftWorldConfig | XriftItemConfig;
```

---

## Common Types

### FileData

```typescript
type FileData = ArrayBuffer | Uint8Array;
```

Universal binary data type. Use `Uint8Array` for Node.js (`fs.readFile` result) and `ArrayBuffer` from browser File API.

### UploadFile

```typescript
interface UploadFile {
  remotePath: string;   // Relative path (e.g., 'scene.glb', 'assets/texture.png')
  size: number;         // File size in bytes
  contentType: string;  // MIME type (e.g., 'model/gltf-binary')
  data: FileData;       // Binary file data
}
```

### UploadProgress

```typescript
interface UploadProgress {
  completed: number;    // Number of files uploaded so far
  total: number;        // Total number of files
  currentFile: string;  // Path of file currently being uploaded (empty when done)
}
```

### PhysicsConfig

```typescript
interface PhysicsConfig {
  gravity?: number;            // Gravity value (default: -9.8)
  allowInfiniteJump?: boolean; // Allow jumping in mid-air
}
```

### CameraConfig

```typescript
interface CameraConfig {
  near?: number;  // Near clip distance
  far?: number;   // Far clip distance
}
```

### OutputBufferType

```typescript
type OutputBufferType = 'UnsignedByteType' | 'HalfFloatType' | 'FloatType';
```

### SignedUrlResponse

```typescript
interface SignedUrlResponse {
  path: string;       // File path
  uploadUrl: string;  // Signed upload URL (PUT)
  publicUrl: string;  // Public URL after upload
  expiresAt: string;  // URL expiration time (ISO string)
}
```

---

## World Types

### WorldPermissions

```typescript
interface WorldPermissions {
  allowedDomains?: string[];     // Allowed embed domains
  allowedCodeRules?: string[];   // Allowed code rules
}
```

### WorldUploadOptions

```typescript
interface WorldUploadOptions {
  worldId?: string;
  name: string;
  description?: string;
  thumbnailPath?: string;
  physics?: PhysicsConfig;
  camera?: CameraConfig;
  permissions?: WorldPermissions;
  outputBufferType?: OutputBufferType;
  onProgress?: (progress: UploadProgress) => void;
}
```

### WorldUploadResult

```typescript
interface WorldUploadResult {
  worldId: string;
  versionId: string;
  versionNumber: number;
  contentHash: string;
  files: UploadFile[];
}
```

---

## Item Types

### ItemPermissions

```typescript
interface ItemPermissions {
  allowedDomains?: string[];
  allowedCodeRules?: string[];
}
```

### ItemUploadOptions

```typescript
interface ItemUploadOptions {
  itemId?: string;
  name: string;
  description?: string;
  thumbnailPath?: string;
  permissions?: ItemPermissions;
  onProgress?: (progress: UploadProgress) => void;
}
```

### ItemUploadResult

```typescript
interface ItemUploadResult {
  itemId: string;
  versionId: string;
  versionNumber: number;
  contentHash: string;
  files: UploadFile[];
}
```

---

## Exports Summary

All types are exported from the package entry point:

```typescript
import type {
  // Config
  XriftConfig,
  XriftWorldConfig,
  XriftItemConfig,
  // Common
  FileData,
  UploadFile,
  UploadProgress,
  PhysicsConfig,
  CameraConfig,
  OutputBufferType,
  SignedUrlResponse,
  // Worlds
  WorldPermissions,
  WorldUploadOptions,
  WorldUploadResult,
  // Items
  ItemPermissions,
  ItemUploadOptions,
  ItemUploadResult,
} from '@xrift/sdk';
```

Node.js helpers are exported from a separate entry point:

```typescript
import {
  uploadWorldFromDirectory,
  uploadItemFromDirectory,
} from '@xrift/sdk/node';
```
