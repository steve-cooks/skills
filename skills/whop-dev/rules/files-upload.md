---
name: files-upload
description: Uploading files using Whop's native file storage
metadata:
  tags: files, upload, storage, images
---

Whop provides native file storage via the SDK. No external storage service required.

## Upload Files

```typescript
import fs from "fs";
import { whopsdk } from "@/lib/whop-sdk";

const file = await whopsdk.files.create({
  file: fs.readFileSync("./photo.jpg"),
  filename: "photo.jpg",
});

console.log(file.id);  // file_xxxxxxxxxxxxx
console.log(file.url); // URL to access the file
```

## From Form Data (API Route)

```typescript
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const headersList = await headers();
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  const formData = await request.formData();
  const file = formData.get("file") as File;
  
  if (!file) {
    return NextResponse.json({ error: "No file provided" }, { status: 400 });
  }
  
  // Convert to buffer
  const buffer = Buffer.from(await file.arrayBuffer());
  
  // Upload to Whop
  const uploaded = await whopsdk.files.create({
    file: buffer,
    filename: file.name,
  });
  
  return NextResponse.json({ 
    fileId: uploaded.id,
    url: uploaded.url 
  });
}
```

## Using Uploaded Files

Use the file ID in any API call that accepts file attachments:

```typescript
// Attach to forum post
await whopsdk.forumPosts.create({
  experience_id: "exp_xxx",
  content: "Check out this image",
  attachments: [{ id: file.id }],
});

// Attach to chat message
await whopsdk.messages.create({
  channel_id: "channel_xxx",
  content: "Here's the file",
  attachments: [{ id: file.id }],
});
```

## File Response Object

```typescript
{
  id: "file_xxxxxxxxxxxxx",
  filename: "photo.jpg",
  content_type: "image/jpeg",
  byte_size: 102400,
  url: "https://...",
  upload_status: "ready" // pending, processing, ready, failed
}
```

## Retrieve File

```typescript
const file = await whopsdk.files.retrieve("file_xxxxx");
console.log(file.url);
```

## Client-Side Upload Component

```typescript
"use client";

import { useState } from "react";
import { Button } from "@whop/react";

export function FileUploadButton({ 
  onUpload 
}: { 
  onUpload: (fileId: string, url: string) => void 
}) {
  const [uploading, setUploading] = useState(false);
  
  const handleUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    
    setUploading(true);
    try {
      const formData = new FormData();
      formData.append("file", file);
      
      const res = await fetch("/api/upload", {
        method: "POST",
        body: formData,
      });
      
      if (!res.ok) throw new Error("Upload failed");
      
      const { fileId, url } = await res.json();
      onUpload(fileId, url);
    } catch (error) {
      console.error("Upload error:", error);
    } finally {
      setUploading(false);
    }
  };
  
  return (
    <div>
      <input 
        type="file" 
        onChange={handleUpload} 
        disabled={uploading}
        className="hidden"
        id="file-upload"
      />
      <label htmlFor="file-upload">
        <Button as="span" disabled={uploading}>
          {uploading ? "Uploading..." : "Upload File"}
        </Button>
      </label>
    </div>
  );
}
```

## Image Preview

```typescript
"use client";

import { useState } from "react";

export function ImageUploader() {
  const [preview, setPreview] = useState<string | null>(null);
  const [fileId, setFileId] = useState<string | null>(null);
  
  const handleUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    
    // Show preview immediately
    setPreview(URL.createObjectURL(file));
    
    // Upload to server
    const formData = new FormData();
    formData.append("file", file);
    
    const res = await fetch("/api/upload", {
      method: "POST",
      body: formData,
    });
    
    const { fileId: id, url } = await res.json();
    setFileId(id);
    setPreview(url); // Replace with permanent URL
  };
  
  return (
    <div>
      <input type="file" accept="image/*" onChange={handleUpload} />
      {preview && <img src={preview} alt="Preview" className="max-w-xs mt-4" />}
    </div>
  );
}
```

## Reference

- [Upload Files Guide](https://docs.whop.com/developer/guides/upload-files)
- [Create File API](https://docs.whop.com/api-reference/files/create-file)
- [Retrieve File API](https://docs.whop.com/api-reference/files/retrieve-file)
