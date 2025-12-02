# Node.js Developer Guide for Janus Converter

**Last Updated**: 2025-11-30
**Audience**: Node.js backend developers, document processing pipeline builders

---

## Introduction

`@janus-mcp/converter` is not just a Claude Desktop MCP server—it's a **high-performance TIFF ↔ PDF conversion library** that you can use directly in your Node.js applications.

> Use **@janus-mcp/converter** for TIFF ↔ PDF conversion just like you use **sharp** for image processing.

### Why @janus-mcp/converter?

| Feature | ImageMagick | sharp | @janus-mcp/converter |
|---------|-------------|-------|----------------------|
| TIFF→PDF Speed | Slow (decode/encode) | Not supported | **1,400+ pages/sec** |
| PDF→TIFF Speed | Slow | Not supported | **1,900+ pages/sec** |
| JPEG2000 Support | Limited | Not supported | **Full (Zero-Copy)** |
| Multi-page TIFF | Supported | Not supported | **Native support** |
| Installation | Complex (system deps) | Easy | **Easy (prebuilds included)** |

---

## Quick Start

### 1. Installation

```bash
npm install @janus-mcp/converter
```

### 2. Basic Usage

```javascript
// converter.js
const path = require('path');

// Load native addons directly
const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

async function convertPdfToTiff(pdfPath, tiffPath) {
  const result = await pdf2tiff.ConvertAsync(pdfPath, tiffPath, false);

  if (result.success) {
    console.log(`Conversion successful: ${result.pageCount} pages`);
    return result;
  } else {
    throw new Error(result.message);
  }
}

async function convertTiffToPdf(tiffPath, pdfPath) {
  const result = await tiff2pdf.ConvertAsync(tiffPath, pdfPath, 0);

  if (result.success) {
    console.log(`Conversion successful: ${result.pageCount} pages`);
    return result;
  } else {
    throw new Error(result.message);
  }
}

// Usage example
(async () => {
  try {
    // PDF → TIFF
    await convertPdfToTiff('./input.pdf', './output.tif');

    // TIFF → PDF
    await convertTiffToPdf('./input.tif', './output.pdf');
  } catch (err) {
    console.error('Conversion failed:', err.message);
  }
})();
```

---

## API Reference

### PDF → TIFF Conversion

#### ConvertAsync (Asynchronous, Recommended)

```javascript
const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');

const result = await pdf2tiff.ConvertAsync(pdfPath, tiffPath, verbose);
```

**Parameters**:
| Name | Type | Description |
|------|------|-------------|
| `pdfPath` | string | Input PDF file path |
| `tiffPath` | string | Output TIFF file path |
| `verbose` | boolean | Enable detailed logging |

**Return Value**:
```javascript
{
  success: true,          // Success status
  pageCount: 9,           // Number of converted pages
  message: "Conversion successful"
}
```

#### ConvertSync (Synchronous)

```javascript
// Blocks event loop - only recommended for batch scripts
const result = pdf2tiff.ConvertSync(pdfPath, tiffPath, verbose);
```

### TIFF → PDF Conversion

#### ConvertAsync (Asynchronous, Recommended)

```javascript
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

const result = await tiff2pdf.ConvertAsync(tiffPath, pdfPath, verboseInt);
```

**Parameters**:
| Name | Type | Description |
|------|------|-------------|
| `tiffPath` | string | Input TIFF file path |
| `pdfPath` | string | Output PDF file path |
| `verboseInt` | number | Verbose logging (0: off, 1: on) |

#### JPEG2000 Normalization Options

```javascript
const result = await tiff2pdf.ConvertAsync(tiffPath, pdfPath, {
  normalizeJpeg2000: true,    // Convert 33004, 34713 → 34712
  normalizeFallback: 1        // 0=error, 1=keep original, 2=uncompressed
});

console.log(`Normalized: ${result.normalizeSuccessCount} pages`);
console.log(`Fallback: ${result.normalizeFailedCount} pages`);
```

---

## Practical Examples

### Example 1: Express File Upload Conversion Server

```javascript
// server.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs').promises;

const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

const app = express();
const upload = multer({ dest: 'uploads/' });

// PDF → TIFF conversion endpoint
app.post('/convert/pdf-to-tiff', upload.single('file'), async (req, res) => {
  try {
    const inputPath = req.file.path;
    const outputPath = inputPath + '.tif';

    const result = await pdf2tiff.ConvertAsync(inputPath, outputPath, false);

    if (result.success) {
      res.download(outputPath, 'converted.tif', async () => {
        // Clean up temp files
        await fs.unlink(inputPath);
        await fs.unlink(outputPath);
      });
    } else {
      res.status(500).json({ error: result.message });
    }
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// TIFF → PDF conversion endpoint
app.post('/convert/tiff-to-pdf', upload.single('file'), async (req, res) => {
  try {
    const inputPath = req.file.path;
    const outputPath = inputPath + '.pdf';

    const result = await tiff2pdf.ConvertAsync(inputPath, outputPath, 0);

    if (result.success) {
      res.download(outputPath, 'converted.pdf', async () => {
        await fs.unlink(inputPath);
        await fs.unlink(outputPath);
      });
    } else {
      res.status(500).json({ error: result.message });
    }
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(3000, () => {
  console.log('Conversion server running at http://localhost:3000');
});
```

### Example 2: Batch Conversion

```javascript
// batch-convert.js
const fs = require('fs').promises;
const path = require('path');
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

async function batchConvertTiffToPdf(inputDir, outputDir) {
  const files = await fs.readdir(inputDir);
  const tiffFiles = files.filter(f => /\.(tif|tiff)$/i.test(f));

  console.log(`Starting conversion of ${tiffFiles.length} files...`);

  const results = {
    success: 0,
    failed: 0,
    totalPages: 0,
    errors: []
  };

  const startTime = Date.now();

  // Parallel conversion (5 concurrent)
  const concurrency = 5;
  for (let i = 0; i < tiffFiles.length; i += concurrency) {
    const batch = tiffFiles.slice(i, i + concurrency);

    await Promise.all(batch.map(async (file) => {
      const inputPath = path.join(inputDir, file);
      const outputPath = path.join(outputDir, file.replace(/\.(tif|tiff)$/i, '.pdf'));

      try {
        const result = await tiff2pdf.ConvertAsync(inputPath, outputPath, 0);

        if (result.success) {
          results.success++;
          results.totalPages += result.pageCount;
        } else {
          results.failed++;
          results.errors.push({ file, error: result.message });
        }
      } catch (err) {
        results.failed++;
        results.errors.push({ file, error: err.message });
      }
    }));

    console.log(`Progress: ${Math.min(i + concurrency, tiffFiles.length)}/${tiffFiles.length}`);
  }

  const elapsed = (Date.now() - startTime) / 1000;

  console.log('\n=== Conversion Complete ===');
  console.log(`Success: ${results.success}`);
  console.log(`Failed: ${results.failed}`);
  console.log(`Total pages: ${results.totalPages}`);
  console.log(`Elapsed: ${elapsed.toFixed(2)}s`);
  console.log(`Throughput: ${(results.totalPages / elapsed).toFixed(0)} pages/sec`);

  if (results.errors.length > 0) {
    console.log('\nFailed files:');
    results.errors.forEach(e => console.log(`  - ${e.file}: ${e.error}`));
  }

  return results;
}

// Run
batchConvertTiffToPdf('./input', './output');
```

### Example 3: Stream-based Processing (Using Temp Files)

```javascript
// stream-convert.js
const fs = require('fs');
const path = require('path');
const os = require('os');
const { v4: uuidv4 } = require('uuid');

const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');

async function convertPdfBufferToTiff(pdfBuffer) {
  // Save buffer to temp file
  const tempDir = os.tmpdir();
  const tempPdf = path.join(tempDir, `${uuidv4()}.pdf`);
  const tempTiff = path.join(tempDir, `${uuidv4()}.tif`);

  try {
    // Write PDF buffer to file
    fs.writeFileSync(tempPdf, pdfBuffer);

    // Run conversion
    const result = await pdf2tiff.ConvertAsync(tempPdf, tempTiff, false);

    if (!result.success) {
      throw new Error(result.message);
    }

    // Read result as buffer
    const tiffBuffer = fs.readFileSync(tempTiff);

    return {
      buffer: tiffBuffer,
      pageCount: result.pageCount
    };
  } finally {
    // Clean up temp files
    try { fs.unlinkSync(tempPdf); } catch {}
    try { fs.unlinkSync(tempTiff); } catch {}
  }
}

// Usage example
async function example() {
  const pdfBuffer = fs.readFileSync('./sample.pdf');
  const result = await convertPdfBufferToTiff(pdfBuffer);

  console.log(`Conversion complete: ${result.pageCount} pages, ${result.buffer.length} bytes`);
  fs.writeFileSync('./output.tif', result.buffer);
}
```

### Example 4: Worker Threads for Parallel Processing

```javascript
// worker-convert.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const path = require('path');
const os = require('os');

if (isMainThread) {
  // Main thread
  async function parallelConvert(files) {
    const numCPUs = os.cpus().length;
    const chunkSize = Math.ceil(files.length / numCPUs);

    const workers = [];

    for (let i = 0; i < numCPUs && i * chunkSize < files.length; i++) {
      const chunk = files.slice(i * chunkSize, (i + 1) * chunkSize);

      const worker = new Worker(__filename, {
        workerData: { files: chunk }
      });

      workers.push(new Promise((resolve, reject) => {
        worker.on('message', resolve);
        worker.on('error', reject);
      }));
    }

    const results = await Promise.all(workers);
    return results.flat();
  }

  // Usage
  const files = [
    { input: './file1.tif', output: './file1.pdf' },
    { input: './file2.tif', output: './file2.pdf' },
    // ...
  ];

  parallelConvert(files).then(results => {
    console.log('All conversions complete:', results);
  });

} else {
  // Worker thread
  const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

  async function processChunk(files) {
    const results = [];

    for (const file of files) {
      try {
        const result = await tiff2pdf.ConvertAsync(file.input, file.output, 0);
        results.push({ file: file.input, success: result.success });
      } catch (err) {
        results.push({ file: file.input, success: false, error: err.message });
      }
    }

    return results;
  }

  processChunk(workerData.files).then(results => {
    parentPort.postMessage(results);
  });
}
```

---

## Performance Optimization Tips

### 1. Use Async API

```javascript
// Recommended: Async (no event loop blocking)
const result = await pdf2tiff.ConvertAsync(input, output, false);

// Not recommended: Sync (blocks event loop)
const result = pdf2tiff.ConvertSync(input, output, false);
```

### 2. Set Appropriate Concurrency

```javascript
// Adjust concurrency based on CPU cores
const concurrency = Math.min(os.cpus().length, 8);
```

### 3. Use File-based Processing for Large Files

```javascript
// Memory efficient processing
// File → Convert → File (minimize buffers)
await tiff2pdf.ConvertAsync(inputPath, outputPath, 0);
```

---

## Web Frontend Integration (REST API)

Browsers cannot use Native Addons directly. Build a **REST API server** to integrate with web applications.

> **Why no WASM?** WebAssembly would eliminate Zero-Copy architecture benefits (no direct file system access, requires memory copying). By design principle, we don't provide a WASM version.

### Complete REST API Server Example

```javascript
// janus-api-server.js
const express = require('express');
const multer = require('multer');
const cors = require('cors');
const path = require('path');
const fs = require('fs').promises;
const os = require('os');
const crypto = require('crypto');

const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

const app = express();

// CORS configuration
app.use(cors({
  origin: ['http://localhost:3000', 'https://your-frontend.com'],
  methods: ['POST', 'GET']
}));

// File upload configuration
const storage = multer.diskStorage({
  destination: os.tmpdir(),
  filename: (req, file, cb) => {
    const uniqueName = `${crypto.randomUUID()}${path.extname(file.originalname)}`;
    cb(null, uniqueName);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 100 * 1024 * 1024 }, // 100MB limit
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['.pdf', '.tif', '.tiff'];
    const ext = path.extname(file.originalname).toLowerCase();
    if (allowedTypes.includes(ext)) {
      cb(null, true);
    } else {
      cb(new Error('Unsupported file type.'));
    }
  }
});

// Temp file cleanup function
async function cleanup(...files) {
  for (const file of files) {
    try { await fs.unlink(file); } catch {}
  }
}

// PDF → TIFF conversion API
app.post('/api/convert/pdf-to-tiff', upload.single('file'), async (req, res) => {
  const inputPath = req.file?.path;
  if (!inputPath) {
    return res.status(400).json({ success: false, error: 'File required.' });
  }

  const outputPath = inputPath.replace(/\.pdf$/i, '.tif');

  try {
    const startTime = Date.now();
    const result = await pdf2tiff.ConvertAsync(inputPath, outputPath, false);

    if (!result.success) {
      await cleanup(inputPath);
      return res.status(500).json({ success: false, error: result.message });
    }

    const elapsedMs = Date.now() - startTime;

    res.setHeader('X-Page-Count', result.pageCount);
    res.setHeader('X-Elapsed-Ms', elapsedMs);
    res.download(outputPath, 'converted.tif', async (err) => {
      await cleanup(inputPath, outputPath);
    });

  } catch (err) {
    await cleanup(inputPath, outputPath);
    res.status(500).json({ success: false, error: err.message });
  }
});

// TIFF → PDF conversion API
app.post('/api/convert/tiff-to-pdf', upload.single('file'), async (req, res) => {
  const inputPath = req.file?.path;
  if (!inputPath) {
    return res.status(400).json({ success: false, error: 'File required.' });
  }

  const outputPath = inputPath.replace(/\.(tif|tiff)$/i, '.pdf');

  try {
    const startTime = Date.now();
    const result = await tiff2pdf.ConvertAsync(inputPath, outputPath, 0);

    if (!result.success) {
      await cleanup(inputPath);
      return res.status(500).json({ success: false, error: result.message });
    }

    const elapsedMs = Date.now() - startTime;

    res.setHeader('X-Page-Count', result.pageCount);
    res.setHeader('X-Elapsed-Ms', elapsedMs);
    res.download(outputPath, 'converted.pdf', async (err) => {
      await cleanup(inputPath, outputPath);
    });

  } catch (err) {
    await cleanup(inputPath, outputPath);
    res.status(500).json({ success: false, error: err.message });
  }
});

// Health check
app.get('/api/health', (req, res) => {
  res.json({
    status: 'ok',
    version: '1.0.0',
    timestamp: new Date().toISOString()
  });
});

// Error handler
app.use((err, req, res, next) => {
  console.error('Server Error:', err);
  res.status(500).json({ success: false, error: err.message });
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, () => {
  console.log(`Janus Converter API server running at: http://localhost:${PORT}`);
  console.log('Endpoints:');
  console.log('  POST /api/convert/pdf-to-tiff  - PDF → TIFF');
  console.log('  POST /api/convert/tiff-to-pdf  - TIFF → PDF');
  console.log('  GET  /api/health               - Health check');
});
```

### Frontend Integration (React/JavaScript)

```javascript
// Browser API call
async function convertPdfToTiff(file) {
  const formData = new FormData();
  formData.append('file', file);

  const response = await fetch('http://localhost:4000/api/convert/pdf-to-tiff', {
    method: 'POST',
    body: formData
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error);
  }

  // Extract metadata from response headers
  const pageCount = response.headers.get('X-Page-Count');
  const elapsedMs = response.headers.get('X-Elapsed-Ms');
  console.log(`Conversion complete: ${pageCount} pages, ${elapsedMs}ms`);

  // Download as Blob
  const blob = await response.blob();
  const url = URL.createObjectURL(blob);

  // Auto download
  const a = document.createElement('a');
  a.href = url;
  a.download = 'converted.tif';
  a.click();

  URL.revokeObjectURL(url);
}

// File input event handler
document.getElementById('fileInput').addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (file) {
    try {
      await convertPdfToTiff(file);
    } catch (err) {
      alert('Conversion failed: ' + err.message);
    }
  }
});
```

### cURL Testing

```bash
# PDF → TIFF conversion
curl -X POST -F "file=@input.pdf" http://localhost:4000/api/convert/pdf-to-tiff -o output.tif

# TIFF → PDF conversion
curl -X POST -F "file=@input.tif" http://localhost:4000/api/convert/tiff-to-pdf -o output.pdf

# Health check
curl http://localhost:4000/api/health
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:20-slim

# Install dependencies (Linux)
RUN apt-get update && apt-get install -y \
    libtiff-dev \
    libopenjp2-7-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 4000
CMD ["node", "janus-api-server.js"]
```

```bash
# Build and run
docker build -t janus-api .
docker run -p 4000:4000 janus-api
```

---

## Supported Platforms

| Platform | Architecture | Prebuilt | Notes |
|----------|--------------|----------|-------|
| Windows | x64 | ✅ | No additional dependencies |
| Linux | x64 | ✅ | libtiff, openjpeg required |
| macOS | ARM64 | ✅ | Install via Homebrew |

---

## Troubleshooting

### Module Load Failure

```javascript
// Check path
const addonPath = require.resolve('@janus-mcp/converter/build/Release/pdf2tiff.node');
console.log('Addon path:', addonPath);
```

### Linux Library Errors

```bash
# Install libtiff
sudo apt-get install libtiff-dev libopenjp2-7-dev libjpeg-turbo8-dev zlib1g-dev
```

### libtiff.so.5 Not Found Error (Ubuntu 24.04, Debian 12, RHEL 9+)

You may encounter this error on newer Linux distributions:

```
error while loading shared libraries: libtiff.so.5: cannot open shared object file
```

**Cause**: Prebuilds were compiled on Ubuntu 20.04 (libtiff.so.5), but newer distros use libtiff.so.6.

**Distribution libtiff Versions**:

| Distribution | libtiff Version | File Name | Action |
|--------------|-----------------|-----------|--------|
| Ubuntu 20.04/22.04 | 4.1~4.3 | libtiff.so.5 | Works |
| **Ubuntu 24.04+** | 4.5+ | libtiff.so.6 | **Symlink required** |
| Debian 11 | 4.2 | libtiff.so.5 | Works |
| **Debian 12+** | 4.5+ | libtiff.so.6 | **Symlink required** |
| RHEL/Rocky 8 | 4.0 | libtiff.so.5 | Works |
| **RHEL/Rocky 9+** | 4.4+ | libtiff.so.6 | **Symlink required** |
| Fedora 38- | 4.4 | libtiff.so.5 | Works |
| **Fedora 39+** | 4.5+ | libtiff.so.6 | **Symlink required** |

**Solution (Create Symlink)**:

```bash
# 1. Find libtiff.so.6 location
find /usr -name "libtiff.so.6*" 2>/dev/null
# Example output: /usr/lib/x86_64-linux-gnu/libtiff.so.6

# 2. Create symlink
sudo ln -s /usr/lib/x86_64-linux-gnu/libtiff.so.6 /usr/lib/x86_64-linux-gnu/libtiff.so.5

# 3. Update library cache
sudo ldconfig

# 4. Verify
ls -la /usr/lib/x86_64-linux-gnu/libtiff.so.5
# Output: libtiff.so.5 -> libtiff.so.6
```

**For ARM64 (aarch64) systems**:

```bash
sudo ln -s /usr/lib/aarch64-linux-gnu/libtiff.so.6 /usr/lib/aarch64-linux-gnu/libtiff.so.5
sudo ldconfig
```

> **Note**: libtiff.so.5 and libtiff.so.6 have ABI compatibility for the functions used by this package, so the symlink works without issues.

### Debug Conversion Failures

```javascript
// Enable verbose mode
const result = await tiff2pdf.ConvertAsync(input, output, 1);
// Detailed logs printed to console
```

---

## License

MPL 2.0 - Free for commercial use.

---

**Contact**: tangokorea@gmail.com
**Issues**: https://github.com/anthropics/janus-mcp/issues
