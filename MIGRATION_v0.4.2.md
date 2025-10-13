# imgflo v0.4.2 Migration Guide

## What's Fixed

**Critical Issue**: The CLI `generate` command couldn't upload to S3-compatible storage (AWS S3, Tigris, Cloudflare R2, etc.). This broke the core workflow abstraction.

**v0.4.1 (Broken)**:
```bash
imgflo generate --generator openai --params '{"prompt":"sunset"}' --out s3://bucket/image.png
# ❌ Failed silently - only saved to local filesystem
```

**v0.4.2 (Fixed)**:
```bash
imgflo generate --generator openai --params '{"prompt":"sunset"}' --out s3://bucket/image.png
# ✅ Works! Detects :// protocol and uploads to S3
```

## Upgrade Steps

### 1. Update imgflo

```bash
npm install imgflo@0.4.2
# or
pnpm add imgflo@0.4.2
```

### 2. Create Configuration File

Create `imgflo.config.ts` in your project root:

```typescript
import type { ImgfloConfig } from 'imgflo';

export default {
  verbose: false,

  // OpenAI API key for AI image generation
  ai: {
    openai: {
      apiKey: process.env.OPENAI_API_KEY || 'sk-...',
    },
  },

  // Save providers (filesystem and S3)
  save: {
    default: 'fs',  // Use 'fs' for local, 's3' for cloud by default

    // Filesystem save (works out of the box)
    fs: {
      baseDir: './output',
      chmod: 0o644,
    },

    // S3-compatible storage (AWS S3, Tigris, R2, etc.)
    s3: {
      bucket: process.env.S3_BUCKET || 'my-bucket',
      region: process.env.AWS_REGION || 'us-east-1',

      // Optional: For non-AWS S3-compatible services
      endpoint: process.env.S3_ENDPOINT,
      // Examples:
      // endpoint: 'https://fly.storage.tigris.dev',               // Tigris
      // endpoint: 'https://your-account.r2.cloudflarestorage.com', // Cloudflare R2

      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID || 'your-access-key',
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY || 'your-secret-key',
      },
    },
  },
} satisfies ImgfloConfig;
```

### 3. Configure Environment Variables

Create or update `.env`:

```bash
# OpenAI (for AI image generation)
OPENAI_API_KEY=sk-proj-...

# S3-compatible storage credentials
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1
S3_BUCKET=my-bucket

# Optional: For Tigris, R2, or other S3-compatible services
S3_ENDPOINT=https://fly.storage.tigris.dev
```

## Usage Examples

### CLI: Individual Commands

```bash
# Generate AI image and save to S3
imgflo generate \
  --generator openai \
  --params '{"prompt":"A serene sunset over mountains"}' \
  --out s3://my-bucket/sunset.png

# Generate locally, then upload separately
imgflo generate --generator openai --params '{"prompt":"sunset"}' --out ./sunset.png
imgflo save --in ./sunset.png --destination s3://my-bucket/sunset.png
```

### CLI: Multi-Step Workflows (YAML Pipelines)

Create `workflow.yaml`:

```yaml
name: AI Image to S3 Workflow
steps:
  # Step 1: Generate AI image
  - kind: generate
    generator: openai
    params:
      prompt: "A serene sunset over mountains"
      size: "1024x1024"
    out: sunset_image

  # Step 2: Resize it
  - kind: transform
    in: sunset_image
    op: resize
    params:
      width: 800
      height: 600
    out: resized_image

  # Step 3: Add a caption
  - kind: transform
    in: resized_image
    op: addCaption
    params:
      text: "Beautiful Sunset"
      position: bottom
    out: final_image

  # Step 4: Upload to S3
  - kind: save
    in: final_image
    destination: s3://my-bucket/sunset.png
```

Run the workflow:

```bash
imgflo run workflow.yaml
```

### TypeScript/JavaScript: Programmatic Usage

```typescript
import createClient from 'imgflo';

const imgflo = createClient();

// Generate and save to S3 in one step
const blob = await imgflo.generate({
  generator: 'openai',
  params: { prompt: 'A serene sunset over mountains' },
});

const result = await imgflo.save(blob, 's3://my-bucket/sunset.png');
console.log(`Saved to: ${result.location}`);

// Or chain operations
const resized = await imgflo.transform(blob, {
  operation: 'resize',
  params: { width: 800, height: 600 },
});

const captioned = await imgflo.transform(resized, {
  operation: 'addCaption',
  params: { text: 'Beautiful Sunset', position: 'bottom' },
});

await imgflo.save(captioned, 's3://my-bucket/sunset-final.png');
```

### MCP: Claude Code Integration

The MCP server now consistently supports cloud storage across all operations:

```javascript
// Generate image with immediate save to S3
generate_image({
  intent: "A serene sunset over mountains",
  saveTo: "s3://my-bucket/sunset.png"
})

// Or use the pipeline API
run_pipeline({
  steps: [
    { generate: { intent: "Baseball stadium at sunset" } },
    { transform: { operation: "resize", params: { width: 800, height: 600 } } },
    { transform: { operation: "addCaption", params: { text: "Game Day" } } },
    { save: { destination: "s3://my-bucket/stadium.png" } }
  ]
})
```

## What Changed

### Technical Details

1. **CLI `generate` command** (packages/imgflo/src/cli/commands/generate.ts):
   - Now detects `://` in `--out` path
   - Routes to `client.save()` for cloud destinations
   - Uses `writeFile()` for local paths

2. **Error Messages** (packages/imgflo/src/core/errors.ts):
   - `ProviderNotFoundError` now shows exact config needed
   - Guides users to create `imgflo.config.ts`

3. **Documentation**:
   - Added `imgflo.config.example.ts` - complete config reference
   - Added `example-pipeline.yaml` - multi-step workflow example
   - Updated README with "Quick Start: Cloud Storage" section

### Breaking Changes

**None!** This is a pure bug fix release. All existing code continues to work.

### Workflow Abstraction Consistency

The `generate → transform → save` workflow now works consistently across all interfaces:

| Interface | Status |
|-----------|--------|
| JavaScript API | ✅ Always worked |
| YAML Pipelines | ✅ Always worked |
| MCP Server | ✅ Always worked |
| CLI | ✅ **Now fixed in v0.4.2** |

## Common Issues

### Issue: "Provider 's3' not found"

**Cause**: You haven't configured the S3 provider.

**Solution**: Create `imgflo.config.ts` with S3 configuration (see step 2 above).

### Issue: "Access Denied" or "Invalid credentials"

**Cause**: Environment variables not set or incorrect.

**Solution**:
1. Verify `.env` file has correct credentials
2. Ensure your Node.js process loads the `.env` file (use `dotenv`)
3. Test credentials with AWS CLI: `aws s3 ls s3://your-bucket`

### Issue: "Endpoint not found"

**Cause**: Missing or incorrect `endpoint` for non-AWS S3 services (Tigris, R2).

**Solution**: Add `endpoint` to your config:
```typescript
s3: {
  endpoint: 'https://fly.storage.tigris.dev',  // Tigris
  // or
  endpoint: 'https://your-account.r2.cloudflarestorage.com',  // Cloudflare R2
  ...
}
```

## Testing Your Setup

### Test S3 Configuration

```bash
# Test with a simple shape
imgflo generate \
  --generator shapes \
  --params '{"type":"circle","width":100,"height":100,"color":"blue"}' \
  --out s3://my-bucket/test.png

# You should see:
# Generated image saved to cloud storage!
# Provider: s3
# Location: s3://my-bucket/test.png
```

### Test YAML Pipeline

```bash
# Use the example pipeline
imgflo run example-pipeline.yaml
```

### Verify in S3 Console

Check your S3 bucket (or Tigris dashboard) to confirm the files were uploaded.

## Need Help?

- **Documentation**: https://github.com/bcooke/imgflo#readme
- **Issues**: https://github.com/bcooke/imgflo/issues
- **Example Config**: See `imgflo.config.example.ts` in the repo
- **Example Pipeline**: See `example-pipeline.yaml` in the repo

## Summary

**Before v0.4.2**:
```bash
imgflo generate --out s3://bucket/image.png  # ❌ Didn't work
```

**After v0.4.2**:
```bash
# 1. Create imgflo.config.ts with S3 credentials
# 2. Set environment variables
imgflo generate --out s3://bucket/image.png  # ✅ Works!
```

The core workflow abstraction (`generate → transform → save`) now works consistently everywhere.
