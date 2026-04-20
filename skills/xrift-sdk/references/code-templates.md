# @xrift/sdk Code Templates

## Node.js: Upload a World from Local Files

```typescript
import { readFile } from 'node:fs/promises';
import { XriftClient, getMimeType } from '@xrift/sdk';

const client = new XriftClient({
  token: process.env.XRIFT_TOKEN!,
});

// Read file into Uint8Array
const data = new Uint8Array(await readFile('dist/scene.glb'));

const result = await client.worlds.upload(
  [
    {
      remotePath: 'scene.glb',
      data,
      size: data.byteLength,
      contentType: getMimeType('scene.glb'),
    },
  ],
  {
    name: 'My World',
    description: 'A 3D world built with XRift',
  },
);

console.log(`World ID: ${result.worldId}`);
console.log(`Version: ${result.versionNumber}`);
console.log(`Hash: ${result.contentHash}`);
```

## Node.js: Upload Multiple Files

```typescript
import { readFile } from 'node:fs/promises';
import path from 'node:path';
import { XriftClient, getMimeType, type UploadFile } from '@xrift/sdk';

const client = new XriftClient({ token: process.env.XRIFT_TOKEN! });

const filePaths = ['scene.glb', 'thumbnail.png', 'assets/texture.jpg'];
const files: UploadFile[] = await Promise.all(
  filePaths.map(async (filePath) => {
    const data = new Uint8Array(await readFile(filePath));
    return {
      remotePath: filePath,
      data,
      size: data.byteLength,
      contentType: getMimeType(filePath),
    };
  }),
);

const result = await client.worlds.upload(files, {
  name: 'Multi-file World',
  thumbnailPath: 'thumbnail.png',
});
```

## Node.js: Upload an Item

```typescript
import { readFile } from 'node:fs/promises';
import { XriftClient, getMimeType } from '@xrift/sdk';

const client = new XriftClient({ token: process.env.XRIFT_TOKEN! });

const data = new Uint8Array(await readFile('model.glb'));

const result = await client.items.upload(
  [
    {
      remotePath: 'model.glb',
      data,
      size: data.byteLength,
      contentType: getMimeType('model.glb'),
    },
  ],
  {
    name: 'My Item',
    description: 'An interactive 3D item',
  },
);

console.log(`Item ID: ${result.itemId}`);
```

## Node.js: Update an Existing World

```typescript
const result = await client.worlds.upload(files, {
  worldId: 'existing-world-id', // Specify existing world ID
  name: 'Updated World',
  description: 'Version 2 of my world',
});

console.log(`Updated version: ${result.versionNumber}`);
```

## Node.js: Upload with Physics and Camera Config

```typescript
const result = await client.worlds.upload(files, {
  name: 'Physics World',
  physics: {
    gravity: -9.8,
    allowInfiniteJump: false,
  },
  camera: {
    near: 0.1,
    far: 1000,
  },
  outputBufferType: 'HalfFloatType',
  permissions: {
    allowedDomains: ['example.com'],
  },
});
```

## Node.js: Progress Tracking

```typescript
const result = await client.worlds.upload(files, {
  name: 'My World',
  onProgress: (progress) => {
    const percent = Math.round((progress.completed / progress.total) * 100);
    if (progress.currentFile) {
      console.log(`[${percent}%] Uploading: ${progress.currentFile}`);
    } else {
      console.log(`[${percent}%] Upload complete`);
    }
  },
});
```

## Node.js: Error Handling

```typescript
import {
  XriftClient,
  XriftAuthError,
  XriftApiError,
  XriftNetworkError,
} from '@xrift/sdk';

const client = new XriftClient({ token: process.env.XRIFT_TOKEN! });

try {
  const result = await client.worlds.upload(files, options);
  console.log('Success:', result.worldId);
} catch (error) {
  if (error instanceof XriftAuthError) {
    console.error('Authentication failed. Check your API token.');
    console.error('Response:', error.responseBody);
  } else if (error instanceof XriftApiError) {
    console.error(`API error ${error.statusCode}: ${error.message}`);
    console.error('Response:', error.responseBody);
  } else if (error instanceof XriftNetworkError) {
    console.error('Network error:', error.message);
    if (error.cause) {
      console.error('Cause:', error.cause.message);
    }
  } else {
    throw error;
  }
}
```

## Browser: Upload from File Input

```typescript
import { XriftClient, getMimeType } from '@xrift/sdk';

const client = new XriftClient({ token: apiToken });

const fileInput = document.querySelector<HTMLInputElement>('#file-input')!;
const file = fileInput.files![0];
const data = new Uint8Array(await file.arrayBuffer());

const result = await client.worlds.upload(
  [
    {
      remotePath: file.name,
      data,
      size: data.byteLength,
      contentType: file.type || getMimeType(file.name),
    },
  ],
  {
    name: 'Browser Upload',
  },
);

console.log(`Uploaded: ${result.worldId}`);
```

## Browser: Upload Multiple Files with Drag & Drop

```typescript
import { XriftClient, getMimeType, type UploadFile } from '@xrift/sdk';

const client = new XriftClient({ token: apiToken });

const dropZone = document.getElementById('drop-zone')!;

dropZone.addEventListener('drop', async (event) => {
  event.preventDefault();
  const droppedFiles = event.dataTransfer!.files;

  const files: UploadFile[] = await Promise.all(
    Array.from(droppedFiles).map(async (file) => {
      const data = new Uint8Array(await file.arrayBuffer());
      return {
        remotePath: file.name,
        data,
        size: data.byteLength,
        contentType: file.type || getMimeType(file.name),
      };
    }),
  );

  const result = await client.worlds.upload(files, {
    name: 'Drag & Drop World',
  });

  alert(`World uploaded: ${result.worldId}`);
});
```

## Browser: Upload with Progress UI

```typescript
import { XriftClient, getMimeType } from '@xrift/sdk';

const client = new XriftClient({ token: apiToken });
const progressBar = document.querySelector<HTMLProgressElement>('#progress')!;
const statusText = document.getElementById('status')!;

const result = await client.worlds.upload(files, {
  name: 'My World',
  onProgress: (progress) => {
    progressBar.max = progress.total;
    progressBar.value = progress.completed;
    if (progress.currentFile) {
      statusText.textContent = `Uploading: ${progress.currentFile}`;
    } else {
      statusText.textContent = 'Upload complete!';
    }
  },
});
```

## Using xrift.json Configuration (Node.js)

### With uploadWorldFromDirectory (recommended)

```typescript
import { uploadWorldFromDirectory } from '@xrift/sdk/node';

const result = await uploadWorldFromDirectory('./my-project', {
  token: process.env.XRIFT_TOKEN!,
  onProgress: (p) => console.log(`${p.completed}/${p.total}: ${p.currentFile}`),
});

console.log(`World ID: ${result.worldId}`);
```

### With parseWorldConfig (manual control)

```typescript
import { readFile } from 'node:fs/promises';
import { XriftClient, parseWorldConfig, getMimeType, type UploadFile } from '@xrift/sdk';

const json = await readFile('xrift.json', 'utf-8');
const config = parseWorldConfig(json);

const client = new XriftClient({ token: process.env.XRIFT_TOKEN! });

// Read files from distDir
const files: UploadFile[] = [/* ... build UploadFile[] from config.distDir ... */];

const result = await client.worlds.upload(files, {
  name: config.name,
  description: config.description,
  thumbnailPath: config.thumbnailPath,
  physics: config.physics,
  camera: config.camera,
  permissions: config.permissions,
  outputBufferType: config.outputBufferType,
});
```
