# imgflo

> Universal image workflow engine for developers and AI agents

[![npm version](https://img.shields.io/npm/v/imgflo.svg?style=flat)](https://www.npmjs.com/package/imgflo)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**imgflo** is a **workflow execution engine** for image generation and manipulation. It provides three core operations (generate, transform, save) that work consistently across JavaScript, CLI, YAML, and MCP.

## The Problem

**LLMs are non-deterministic.** They're brilliant at understanding "create a sunset over mountains" but struggle with precise operations like "resize to exactly 800x600" or "convert to WebP at 80% quality." The same prompt can produce different results—great for creativity, terrible for reliable image processing.

**Image libraries are fragmented.** Charts need Chart.js. Diagrams need Mermaid. QR codes need node-qrcode. Screenshots need Playwright. Each has different APIs, different output formats, and no way to chain them together. A simple "generate chart → resize → upload to S3" workflow requires juggling three different libraries with manual buffer passing.

**imgflo solves both problems.** It provides a deterministic execution engine with a unified abstraction. LLMs handle the natural language; imgflo handles the precise execution. One API for all image types—charts, diagrams, QR codes, screenshots, AI images—with consistent chaining and storage.

> **Deep dive:** [Why imgflo Exists](./vault/Why-imgflo-Exists.md) explains these problems in detail with concrete examples.

## Core Concept

Every image workflow is a series of steps:
1. **generate** - Create an image (charts, QR codes, AI images, etc.)
2. **transform** - Modify an image (resize, filters, text overlays)
3. **save** - Store an image (filesystem, S3, cloud storage)

These primitives work the same way whether you're:
- Writing JavaScript code
- Running CLI commands
- Defining YAML workflows
- Using LLMs (Claude, GPT) via MCP
- Building your own APIs or tools on top of imgflo

## What imgflo Does (and Doesn't)

✅ **imgflo executes structured workflows reliably**
✅ Provides generators, transforms, and storage providers
✅ Handles image data efficiently (session workspace, no byte passing)
✅ Works consistently across all interfaces

❌ **imgflo does NOT parse natural language**
❌ Does not extract data from text descriptions
❌ Does not infer missing workflow steps

**Perfect pairing:** LLMs parse natural language → imgflo executes workflows

## Three Ways to Use imgflo

All three interfaces use the **same plugins** and **same underlying engine**. Choose based on your workflow:

### 1. 📚 As a Library (For Developers)

Install and use programmatically:

```bash
npm install imgflo imgflo-qr
```

```typescript
import createClient from 'imgflo';
import qr from 'imgflo-qr';

const imgflo = createClient();
imgflo.registerGenerator(qr());

const qrCode = await imgflo.generate({
  generator: 'qr',
  params: { text: 'https://example.com', width: 300 }
});

await imgflo.save(qrCode, './qr.png');
```

**→ Use when:** Building applications, need programmatic control, want type safety

---

### 2. 💻 As a CLI (For Scripts & Automation)

Install globally and use from terminal:

```bash
npm install -g imgflo imgflo-qr imgflo-quickchart
```

```bash
# See what's available
imgflo plugins

# Generate images
imgflo generate --generator qr --params '{"text":"https://example.com"}' --out qr.png

# Check configuration
imgflo doctor
```

**→ Use when:** Writing scripts, automating tasks, CI/CD pipelines, quick one-offs

---

### 3. 🤖 As an MCP Server (For AI Agents)

Connect to Claude Code and generate images with **natural language**:

```bash
npm install -g imgflo imgflo-qr imgflo-quickchart imgflo-d3 imgflo-mermaid
imgflo mcp install  # Generates config
```

Then just talk naturally to Claude:

> **You:** "Create a QR code for https://example.com"
>
> **Claude:** *Uses imgflo automatically → generates QR code*

> **You:** "Generate a bar chart showing Q1: 10, Q2: 20, Q3: 15"
>
> **Claude:** *Uses imgflo automatically → generates chart*

> **You:** "Generate a sunset over mountains with DALL-E"
>
> **Claude:** *Uses imgflo's OpenAI generator → generates AI image*

**→ Use when:** Working with AI assistants, describing images in natural language, rapid prototyping

**[Full MCP Setup Guide →](./packages/imgflo/docs/guides/MCP_SERVER.md)**

---

## Using with LLMs (Claude, GPT, etc.)

imgflo and LLMs complement each other perfectly:

**LLM's Job:**
- Parse user's natural language request
- Extract structured data (chart data, dimensions, colors)
- Decide which generators/operations to use
- Construct workflow steps

**imgflo's Job:**
- Execute the structured workflow reliably
- Handle image operations and storage
- Return predictable results

### Example Flow

**User to Claude:**
> "Create a bar chart with sales data, resize to 800px, upload to S3"

**Claude processes:**
1. Extracts data: `{labels: [...], values: [...]}`
2. Constructs workflow:
   ```javascript
   run_pipeline({
     steps: [
       { generate: { generator: 'quickchart', params: {...data...} } },
       { transform: { operation: 'resize', params: { width: 800 } } },
       { save: { destination: 's3://bucket/chart.png' } }
     ]
   })
   ```

**imgflo executes** and returns the final image URL.

---

## Workflow Abstraction in Action

imgflo provides consistent primitives across all interfaces:

| When an LLM sees... | It constructs workflow... | imgflo executes... |
|---------------------|---------------------------|-------------------|
| *"Create a bar chart with sales data"* | `generate(quickchart, {...data...})` | Chart.js visualization |
| *"Make a QR code for my website"* | `generate(qr, {text: url})` | Scannable QR code |
| *"Draw a flowchart, then resize it"* | `generate(mermaid, ...) → transform(resize, ...)` | Diagram + resized output |
| *"Generate AI image of mountains"* | `generate(openai, {prompt: "..."})` | DALL-E image |
| *"Screenshot site, convert to PNG"* | `generate(screenshot, ...) → transform(convert, ...)` | PNG screenshot |

**Via MCP**: LLMs parse your natural language → construct workflows → imgflo executes
**Via code/CLI**: You specify the workflow directly for precise control

---

## What Can imgflo Generate?

| Generator | Description | Natural Language Example |
|-----------|-------------|--------------------------|
| **openai** | DALL-E AI images from prompts | *"sunset over mountains"* |
| **quickchart** | Chart.js charts (bar, line, pie) | *"bar chart with Q1-Q4 revenue"* |
| **mermaid** | Flowcharts, diagrams, sequences | *"flowchart for login process"* |
| **d3** | Custom data visualizations | *"scatter plot of sales data"* |
| **qr** | QR codes | *"QR code for example.com"* |
| **screenshot** | Website captures | *"screenshot of example.com"* |
| **shapes** | SVG shapes, gradients, patterns | *"purple gradient background"* |

**All generators work with natural language (MCP), structured params (code), or CLI arguments.**

### Installation

```bash
# Core (includes shapes + openai)
npm install imgflo

# Add plugins you need
npm install imgflo-quickchart  # Charts
npm install imgflo-d3          # Custom visualizations
npm install imgflo-mermaid     # Diagrams
npm install imgflo-qr          # QR codes
npm install imgflo-screenshot  # Screenshots

# Or all at once
npm install imgflo imgflo-quickchart imgflo-d3 imgflo-mermaid imgflo-qr imgflo-screenshot
```

---

## Quick Examples

### Generate a Chart

```typescript
import createClient from 'imgflo';
import quickchart from 'imgflo-quickchart';

const imgflo = createClient();
imgflo.registerGenerator(quickchart());

const chart = await imgflo.generate({
  generator: 'quickchart',
  params: {
    type: 'bar',
    data: {
      labels: ['Q1', 'Q2', 'Q3', 'Q4'],
      datasets: [{
        label: 'Revenue',
        data: [12000, 19000, 15000, 22000]
      }]
    }
  }
});
```

### Generate a QR Code

```typescript
import qr from 'imgflo-qr';

imgflo.registerGenerator(qr());

const qrCode = await imgflo.generate({
  generator: 'qr',
  params: {
    text: 'https://example.com',
    width: 300,
    errorCorrectionLevel: 'H'
  }
});
```

### Generate a Diagram

```typescript
import mermaid from 'imgflo-mermaid';

imgflo.registerGenerator(mermaid());

const diagram = await imgflo.generate({
  generator: 'mermaid',
  params: {
    code: `
      graph TD
        A[Start] --> B[Process]
        B --> C{Decision}
        C -->|Yes| D[End]
        C -->|No| B
    `
  }
});
```

### Take a Screenshot

```typescript
import screenshot from 'imgflo-screenshot';

imgflo.registerGenerator(screenshot());

const site = await imgflo.generate({
  generator: 'screenshot',
  params: {
    url: 'https://example.com',
    width: 1920,
    height: 1080,
    fullPage: true
  }
});
```

---

## Complete Workflow

```typescript
import createClient from 'imgflo';
import quickchart from 'imgflo-quickchart';

const imgflo = createClient({
  save: {
    default: 's3',
    s3: {
      bucket: 'my-images',
      region: 'us-east-1'
    }
  }
});

imgflo.registerGenerator(quickchart());

// 1. Generate
const chart = await imgflo.generate({
  generator: 'quickchart',
  params: { type: 'bar', data: {...} }
});

// 2. Transform (optional)
const png = await imgflo.transform({
  blob: chart,
  op: 'convert',
  to: 'image/png'
});

// 3. Save
const result = await imgflo.save(png, 'charts/revenue.png');

console.log(result.url); // https://...
```

---

## CLI Usage

The CLI supports both **individual commands** and **complete workflows**.

### Individual Commands

Perfect for quick one-offs or scripting:

```bash
# Generate image (saves to local file or S3)
imgflo generate --generator openai --params '{"prompt":"sunset"}' --out sunset.png
imgflo generate --generator openai --params '{"prompt":"sunset"}' --out s3://bucket/sunset.png

# Transform image
imgflo transform --input sunset.png --operation resize --params '{"width":800}' --out resized.png

# Save to cloud storage
imgflo save --in resized.png --out s3://bucket/final.png

# Check configuration
imgflo doctor
imgflo plugins
```

### Multi-Step Workflows (YAML Pipelines)

Perfect for complex, repeatable workflows:

**Create `pipeline.yaml`:**
```yaml
name: AI Image to S3 Workflow
steps:
  - kind: generate
    generator: openai
    params:
      prompt: "A serene sunset over mountains"
      size: "1024x1024"
    out: sunset_image

  - kind: transform
    in: sunset_image
    op: resize
    params:
      width: 800
      height: 600
    out: resized_image

  - kind: transform
    in: resized_image
    op: addCaption
    params:
      text: "Beautiful Sunset"
      position: bottom
    out: final_image

  - kind: save
    in: final_image
    destination: s3://my-bucket/sunset.png
```

**Run it:**
```bash
imgflo run pipeline.yaml
```

**Output:**
```
🚀 Running pipeline: AI Image to S3 Workflow

  sunset_image:
    Type: image/png
    Size: 1024x1024

  resized_image:
    Type: image/png
    Size: 800x600

  final_image:
    Type: image/png
    Size: 800x600

  s3://my-bucket/sunset.png:
    Location: https://my-bucket.s3.amazonaws.com/sunset.png
    Provider: s3
    Size: 245678 bytes

✨ Done!
```

**Why use pipelines?**
- Define workflows once, run many times
- Version control your image generation
- Easy to share and collaborate
- Perfect for CI/CD automation

---

## MCP Usage (AI Agents)

### Setup

```bash
# Install globally with plugins
npm install -g imgflo imgflo-qr imgflo-quickchart imgflo-d3 imgflo-mermaid imgflo-screenshot

# Generate MCP config
imgflo mcp install
```

Copy the output to your Claude Code config file (location shown in output), then restart Claude.

### Usage

Just talk naturally to Claude:

- "Create a QR code for this URL"
- "Generate a bar chart with this data"
- "Take a screenshot of example.com"
- "Create a flowchart showing the login process"

The MCP server automatically:
- Detects what you want based on your intent
- Routes to the appropriate generator
- Handles generation and returns the image

**[Full MCP Documentation →](./packages/imgflo/docs/guides/MCP_SERVER.md)**

---

## Why imgflo?

### Image Generation Glue

imgflo doesn't reinvent the wheel—it connects you to the best tools:

- **QuickChart** for Chart.js charts
- **D3** for custom visualizations
- **Mermaid** for diagrams
- **QRCode** library for QR codes
- **Playwright** for screenshots
- **OpenAI** for AI images

### Pass-Through Pattern

No abstraction layer—use native library formats:

```typescript
// Pure Chart.js configuration
await imgflo.generate({
  generator: 'quickchart',
  params: {
    type: 'bar',
    data: { /* pure Chart.js */ },
    options: { /* pure Chart.js */ }
  }
});
```

### Plugin Architecture

Only install what you need:

```bash
# Minimal: just shapes and OpenAI
npm install imgflo

# Add charts
npm install imgflo-quickchart

# Add everything
npm install imgflo-quickchart imgflo-d3 imgflo-mermaid imgflo-qr imgflo-screenshot
```

---

## Configuration

### Quick Start: Cloud Storage

To upload images to S3-compatible storage (AWS S3, Tigris, R2, etc.), create an `imgflo.config.ts` file in your project root:

```typescript
export default {
  save: {
    default: 's3',  // Use S3 by default
    s3: {
      bucket: 'my-bucket',
      region: 'us-east-1',
      // Optional: endpoint for S3-compatible services
      // endpoint: 'https://fly.storage.tigris.dev',  // Tigris
      // endpoint: 'https://ACCOUNT.r2.cloudflarestorage.com',  // R2
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    },
  },
  ai: {
    openai: {
      apiKey: process.env.OPENAI_API_KEY,
    },
  },
};
```

Then set environment variables:

```bash
# .env
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
OPENAI_API_KEY=sk-...
```

Now you can use `s3://bucket/path` destinations:

```typescript
// Generate AI image and upload to S3
const image = await imgflo.generate({
  generator: 'openai',
  params: { prompt: 'sunset over mountains' }
});

await imgflo.save(image, 's3://my-bucket/sunset.png');
```

**See `imgflo.config.example.ts` for full configuration options.**

###Interactive Setup

Or use the interactive setup wizard:

```bash
imgflo config init
```

---

## Packages

| Package | Description | Install |
|---------|-------------|---------|
| **imgflo** | Core library (shapes, OpenAI) | `npm install imgflo` |
| **imgflo-quickchart** | Chart.js charts | `npm install imgflo-quickchart` |
| **imgflo-d3** | D3 data visualizations | `npm install imgflo-d3` |
| **imgflo-mermaid** | Mermaid diagrams | `npm install imgflo-mermaid` |
| **imgflo-qr** | QR code generation | `npm install imgflo-qr` |
| **imgflo-screenshot** | Website screenshots | `npm install imgflo-screenshot` |

**[View all plugin docs →](./packages/)**

---

## Documentation

- **[Complete Usage Guide](./docs/USAGE.md)** - All three usage patterns explained
- **[Quick Start](./packages/imgflo/docs/guides/QUICK_START.md)** - Get started fast
- **[MCP Server Guide](./packages/imgflo/docs/guides/MCP_SERVER.md)** - Claude Code integration
- **[Configuration](./packages/imgflo/docs/guides/CONFIGURATION.md)** - Setup and config
- **[S3 Providers](./packages/imgflo/docs/guides/S3_PROVIDERS.md)** - Cloud storage
- **[Architecture](./docs/development/GENERATOR_STRATEGY.md)** - How imgflo works
- **[Monorepo Guide](./MONOREPO.md)** - Contributing and development

---

## Features

### Current

- ✅ **Charts** - QuickChart (Chart.js) and D3
- ✅ **Diagrams** - Mermaid (flowcharts, sequence, gantt, etc.)
- ✅ **QR Codes** - Full qrcode library support
- ✅ **Screenshots** - Playwright browser automation
- ✅ **AI Images** - OpenAI DALL-E 2 & 3
- ✅ **Shapes** - SVG gradients, circles, rectangles, patterns
- ✅ **Transform** - Convert SVG → PNG/JPEG/WebP/AVIF
- ✅ **Save** - S3, R2, Tigris, filesystem (smart destination routing)
- ✅ **CLI** - Complete command-line interface
- ✅ **MCP** - Claude Code integration
- ✅ **TypeScript** - Full type safety

### Roadmap

- 🔄 **Satori** - HTML/React to image
- 🔄 **Vega** - Grammar of graphics
- 🔄 **Additional storage** - GCS, Azure Blob

---

## Philosophy

**imgflo is glue, not the engine.**

We orchestrate existing libraries rather than reimplementing them:

1. **Pass-Through Pattern** - Accept native library formats
2. **Minimal Abstraction** - Thin wrappers around existing tools
3. **Opt-In Complexity** - Install only what you need
4. **Full Capabilities** - Never neuter underlying libraries

**[Read the full strategy →](./docs/development/GENERATOR_STRATEGY.md)**

---

## Monorepo Structure

```
imgflo/
├── packages/
│   ├── imgflo/              # Core library
│   ├── imgflo-quickchart/   # Chart.js charts
│   ├── imgflo-d3/           # D3 visualizations
│   ├── imgflo-mermaid/      # Mermaid diagrams
│   ├── imgflo-qr/           # QR codes
│   └── imgflo-screenshot/   # Screenshots
├── docs/                     # Shared documentation
└── examples/                 # Example code
```

---

## Development

```bash
# Install pnpm
npm install -g pnpm

# Install dependencies
pnpm install

# Build all packages
pnpm -r build

# Run tests
pnpm -r test

# Dev mode (watch)
pnpm -r dev
```

---

## Contributing

We welcome contributions! Areas we'd love help with:

- More generators (Satori, Vega, PlantUML)
- Additional storage backends
- Tests and documentation
- Performance improvements

**[Development Guide →](./MONOREPO.md)**

---

## License

MIT © Brett Cooke

---

## Credits

Built with amazing open source tools:

- [Sharp](https://sharp.pixelplumbing.com/) & [Resvg](https://github.com/RazrFalcon/resvg) - Image processing
- [Chart.js](https://www.chartjs.org/) & [QuickChart](https://quickchart.io) - Charts
- [D3](https://d3js.org/) - Data visualizations
- [Mermaid](https://mermaid.js.org/) - Diagrams
- [QRCode](https://github.com/soldair/node-qrcode) - QR generation
- [Playwright](https://playwright.dev) - Browser automation
- [OpenAI](https://openai.com/) - AI images
- [AWS SDK](https://aws.amazon.com/sdk-for-javascript/) - S3 uploads
- [MCP](https://modelcontextprotocol.io) - AI integration

---

**Made for developers and AI agents who need powerful image workflows without the complexity.**
