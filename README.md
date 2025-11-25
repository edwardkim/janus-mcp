# 🏛️ Janus: Native Document Capability for Claude
>The Zero-Copy Gateway for Instant TIFF ↔ PDF Conversion \
>"Don't just convert files. Give your AI the power to read history."

> "I started this project because I was tired of installing proprietary viewers just to check a simple document in banking projects. I wanted a way to view these 'weird' TIFFs in Chrome, instantly."

[![License: MPL 2.0](https://img.shields.io/badge/License-MPL%202.0-brightgreen.svg)](https://opensource.org/licenses/MPL-2.0)
[![Node.js Version](https://img.shields.io/badge/node-%3E%3D18.17.0-brightgreen)](https://nodejs.org/)

## Overview

`@janus-mcp/converter` is a unified MCP (Model Context Protocol) server that provides bidirectional TIFF ↔ PDF conversion with **zero-copy architecture** for maximum performance. This package combines the functionality of `mcp-janus-pdf2tiff` and `mcp-janus-tiff2pdf` into a single, easy-to-use tool.

> **"Input is Innovation, Output is Delegation."**
> We handle the complexity of legacy formats so you can simply delegate tasks to AI.

![Janus Workflow](https://unpkg.com/@janus-mcp/converter@latest/janus-workflow.png)

### Key Features

- ✅ **Zero-Copy Architecture**: Direct stream extraction without decode/re-encode
- ✅ **Ultra-Fast Performance**: ~4ms for 9-page documents (1.9MB)
- ✅ **Non-Blocking Async**: Uses libuv thread pool for async conversion
- ✅ **JPEG2000 Support**: JP2 (34712), J2K (34713), libvips (33004)
- ✅ **JBIG2 Support**: Tag 34663
- ✅ **JPEG Support**: Tag 7
- ✅ **Multi-Page Support**: Convert multi-page TIFFs to multi-page PDFs
- ✅ **MPL 2.0 License**: Open-source with copyleft protection
- ✅ **4 Tools**: PDF→TIFF, TIFF→PDF, PDF info, TIFF info

### Why @janus-mcp/converter?

#### 🔓 Use Case: The "Universal Viewer" for Proprietary TIFFs

**Are you struggling with proprietary TIFF files (Tag 34712, 34713) that won't open in standard image viewers?**

In sectors like Finance and Government (especially in South Korea), TIFFs often use vendor-specific tags wrapping standard JPEG2000 streams. These require expensive, OS-dependent proprietary viewers.

**Janus solves this instantly:**
1.  **Extracts** the standard JPEG2000 stream from the proprietary TIFF container.
2.  **Embeds** it into a standard PDF container using Zero-Copy.
3.  **Result:** You can now view these files in **ANY standard PDF Viewer** (Chrome, Edge, Acrobat, Preview).

> **Effect:** Janus acts as a high-performance, open-source converter that turns any PDF reader into a free viewer for legacy proprietary documents.


Q: I can't open my TIFF file. It shows a black screen or error. 

A: If your TIFF uses proprietary tags like 34712 or 34713 , standard viewers cannot open it. Janus converts these into standard PDFs, allowing you to view them in Chrome, Edge, or Acrobat for free.

## Installation

### Prerequisites

#### Node.js
- **Node.js**: ≥18.17.0, ≥20.3.0, or ≥21.0.0

#### Platform-Specific Dependencies

##### Linux (Ubuntu/Debian/Mint)

The package includes prebuilt binaries for Linux x64 (glibc ≥ 2.31), but requires system libraries:

**Check if libraries are installed:**
```bash
# Check all required libraries at once
pkg-config --modversion libtiff-4 libopenjp2 libjpeg libpng zlib

# Expected output:
# 4.1.0 (or higher)
# 2.3.1 (or higher)
# 2.0.3 (or higher)
# 1.6.37 (or higher)
# 1.2.11 (or higher)
```

**If any library is missing, install development packages:**
```bash
sudo apt-get update
sudo apt-get install -y \
  libtiff-dev \
  libopenjp2-7-dev \
  libjpeg-dev \
  libpng-dev \
  zlib1g-dev
```

**Verify installation:**
```bash
# Check library versions
pkg-config --modversion libtiff-4 libopenjp2 libjpeg libpng zlib

# Expected output (versions may vary):
# libtiff: 4.1.0+
# openjpeg: 2.3.1+
# libjpeg: 2.0.3+
# libpng: 1.6.37+
# zlib: 1.2.11+
```

**Distribution Compatibility:**
- ✅ Ubuntu 20.04 LTS or later
- ✅ Ubuntu 22.04 LTS, 24.04 LTS
- ✅ Linux Mint 20 or later
- ✅ Pop!_OS 20.04 or later
- ✅ Debian 11 (Bullseye) or later
- ✅ Fedora 35 or later
- ✅ RHEL 9 or later

##### macOS

```bash
# Install dependencies via Homebrew
brew install libtiff openjpeg
```

##### Windows

**Prebuilt binaries included** - no additional dependencies needed for Windows x64.

The package includes statically-linked binaries with all dependencies embedded.

### NPM Installation

```bash
npm install @janus-mcp/converter
```

### Build from Source

```bash
git clone https://github.com/edwardkim/janus-mcp.git
cd mcp-janus-converter
npm install
npm run build
```

## Usage

### Claude Desktop Configuration

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "janus-converter": {
      "command": "npx",
      "args": ["-y", "@janus-mcp/converter"]
    }
  }
}
```

### Available Tools

#### 1. `convert_pdf_to_tiff`

Convert PDF to multi-page TIFF.

**Input Schema**:
```json
{
  "pdf_path": "/path/to/input.pdf",
  "tiff_path": "/path/to/output.tif",
  "verbose": false
}
```

**Output**:
```json
{
  "success": true,
  "message": "Conversion successful",
  "input": {
    "path": "/path/to/input.pdf",
    "size": 1048576,
    "sizeFormatted": "1.00 MB"
  },
  "output": {
    "path": "/path/to/output.tif",
    "size": 2097152,
    "sizeFormatted": "2.00 MB",
    "exists": true
  },
  "conversion": {
    "pageCount": 9,
    "elapsedMs": 4,
    "avgMsPerPage": "0.44"
  }
}
```

#### 2. `convert_tiff_to_pdf`

Convert TIFF to multi-page PDF.

**Input Schema**:
```json
{
  "tiff_path": "/path/to/input.tif",
  "pdf_path": "/path/to/output.pdf",
  "verbose": false
}
```

**Output**: Same structure as `convert_pdf_to_tiff`

#### 3. `get_pdf_info`

Get PDF file information without conversion.

**Input Schema**:
```json
{
  "pdf_path": "/path/to/file.pdf"
}
```

**Output**:
```json
{
  "success": true,
  "path": "/path/to/file.pdf",
  "exists": true,
  "size": 1048576,
  "sizeFormatted": "1.00 MB",
  "isFile": true,
  "isDirectory": false,
  "error": null
}
```

#### 4. `get_tiff_info`

Get TIFF file information without conversion.

**Input Schema**: Same as `get_pdf_info` but with `tiff_path`

## API Reference

### Native Addon (pdf2tiff.node)

#### `ConvertSync(pdfPath, tiffPath, verbose)`

Synchronous conversion (blocks event loop).

**Parameters**:
- `pdfPath` (string): Input PDF file path
- `tiffPath` (string): Output TIFF file path
- `verbose` (boolean, optional): Enable verbose output

**Returns**:
```javascript
{
  success: boolean,
  pageCount: number,
  message: string
}
```

#### `ConvertAsync(pdfPath, tiffPath, verbose)`

Asynchronous conversion (non-blocking, returns Promise).

**Parameters**: Same as `ConvertSync`

**Returns**: `Promise<{success, pageCount, message}>`

### Native Addon (tiff2pdf.node)

#### `ConvertSync(tiffPath, pdfPath, verboseInt)`

Synchronous conversion (blocks event loop).

**Parameters**:
- `tiffPath` (string): Input TIFF file path
- `pdfPath` (string): Output PDF file path
- `verboseInt` (number): Verbose flag (0 or 1)

**Returns**:
```javascript
{
  success: boolean,
  pageCount: number,
  message: string
}
```

#### `ConvertAsync(tiffPath, pdfPath, verboseInt)`

Asynchronous conversion (non-blocking, returns Promise).

**Parameters**: Same as `ConvertSync`

**Returns**: `Promise<{success, pageCount, message}>`

## Performance Characteristics

### Zero-Copy Architecture (Strip-based TIFF)

| Conversion | Input Size | Output Size | Pages | Time | Throughput |
|------------|------------|-------------|-------|------|------------|
| PDF→TIFF (strip) | 48 MB | 48 MB | 121 | **63ms** | 1920 pages/sec |
| TIFF→PDF (strip) | 48 MB | 48 MB | 121 | **86ms** | 1407 pages/sec |

### Fallback Mode (Tiled TIFF)

| Conversion | Input Size | Output Size | Pages | Time | Throughput |
|------------|------------|-------------|-------|------|------------|
| TIFF→PDF (tiled 33004) | - | 48 MB | 121 | **22s** | 5.5 pages/sec |

**Performance Impact**: Strip-based vs Tiled = **74× faster** (0.3s vs 22s for 121 pages)

### Memory Usage

| Scenario | Separate Packages | Unified Package | Savings |
|----------|-------------------|-----------------|---------|
| **Idle** | 40 MB × 2 = 80 MB | 40 MB | **50%** |
| **Peak (10 concurrent)** | 160 MB × 2 = 320 MB | 180 MB | **44%** |

### Concurrent Performance

| Requests | Sequential (Sync) | Parallel (Async) | Improvement |
|----------|-------------------|------------------|-------------|
| 1 | 4 ms | 4 ms | 0% |
| 10 | 40 ms | 22 ms | **45%** |
| 100 | 400 ms | 220 ms | **45%** |

## Supported Formats

### PDF → TIFF Conversion

Extracts images from PDF and saves as multi-page TIFF with **zero-copy** architecture:

| PDF Image Filter | TIFF Compression | Tag | Performance | Notes |
|------------------|------------------|-----|-------------|-------|
| **DCTDecode** | JPEG | 7 | ✅ Zero-copy | Most common format |
| **JPXDecode** (JP2) | JPEG2000 JP2 | 34712 | ✅ Zero-copy | InziSoft format |
| **JPXDecode** (J2K) | JPEG2000 J2K | 34713 | ✅ Zero-copy | Minervasoft/Kakadu |
| **JBIG2Decode** | JBIG2 | 34663 | ✅ Zero-copy | Bilevel images |
| **CCITTFaxDecode** | CCITT Group 4 | 4 | ✅ Zero-copy | Fax compression |
| **CCITTFaxDecode** | CCITT Group 3 | 3 | ✅ Zero-copy | Legacy fax |

**Features**:
- Automatic JBIG2 file header insertion (13 bytes) when needed
- Preserves original compression (no re-encoding)
- Multi-page PDF → Multi-page TIFF
- Photometric interpretation handling

### TIFF → PDF Conversion

Embeds TIFF images into PDF with **intelligent routing**:

| TIFF Compression | Tag | PDF Filter | Performance | Notes |
|------------------|-----|------------|-------------|-------|
| **CCITT Group 3** | 3 | CCITTFaxDecode | ✅ Zero-copy | FAX compression, K=0/4 |
| **CCITT Group 4** | 4 | CCITTFaxDecode | ✅ Zero-copy | FAX compression, K=-1 |
| **JPEG** | 7 | DCTDecode | ✅ Zero-copy | Most common format |
| **DEFLATE** | 8 | FlateDecode | ✅ Zero-copy | ZIP compression |
| **JPEG2000 (libvips)** | 33004 | JPXDecode | ✅ Zero-copy | libvips encoding |
| **JPEG2000 (JP2)** | 34712 | JPXDecode | ✅ Zero-copy | InziSoft format |
| **JPEG2000 (J2K)** | 34713 | JPXDecode | ✅ Zero-copy | Minervasoft/Kakadu |
| **JBIG2** | 34663 | JBIG2Decode | ✅ Zero-copy | Bilevel compression |
| **Uncompressed** | 1 | FlateDecode | 🔄 Re-encode | Decode + Deflate |
| **CCITT RLE** | 2 | FlateDecode | 🔄 Re-encode | Modified Huffman |
| **LZW** | 5 | FlateDecode | 🔄 Re-encode | Lempel-Ziv-Welch |
| **PACKBITS** | 32773 | FlateDecode | 🔄 Re-encode | Apple RLE |

**Legend**:
- ✅ **Zero-copy**: Direct stream extraction (ultra-fast, ~100ms for 121 pages)
- 🔄 **Re-encode**: Decode with libtiff + Deflate compression (fast, handles 1-bit bilevel images)

**Advanced Features**:
- **1-bit Bilevel Handling**: Automatically converts 1-bit bilevel images to 8-bit grayscale for PDF compatibility
- **ColorSpace Detection**: Uses `samples_per_pixel` to determine `/DeviceGray` vs `/DeviceRGB`
- **CCITT Group 3 K Parameter**: Detects 1D/2D encoding from Group3Options tag (K=0/4)
- **JPEG ColorTransform**: Automatic detection from component IDs (1,2,3 vs 82,71,66)
- **Automatic MCT Detection**: Parses JPEG2000 COD marker to detect MCT=0/1
- **Smart Re-encoding**: Re-encodes MCT=0 streams to MCT=1 for proper color rendering
- **JP2 IHDR Parsing**: Extracts true dimensions from JP2 container (fixes padding issues)
- **JBIG2 Header Stripping**: Removes 13-byte file header for PDF embedding
- **Photometric Correction**: Inserts `/Decode [1 0]` for min-is-black interpretation

### Performance Comparison

| Operation | Format | Structure | Time (121 pages) | Throughput |
|-----------|--------|-----------|------------------|------------|
| **PDF → TIFF** | JPEG/JBIG2 | Strip-based | **63ms** | 1,920 pages/sec |
| **TIFF → PDF** | 33004/34713 | Strip-based | **86ms** | 1,407 pages/sec |
| **TIFF → PDF** | 33004 | Tiled 128×128 | **22s** | 5.5 pages/sec |

**Key Insight**: Strip-based layout provides **74× performance improvement** over tiled layout

## ⚡ Performance: The 74× Speed Hack

### 🚀 Achieving Maximum Performance

The converter automatically detects TIFF structure and uses the optimal path:

- **Strip-based TIFF** → Zero-copy (recommended, 74× faster)
- **Tiled TIFF** → Decode/re-encode fallback (compatible but slower)

### ⚡ Recommended TIFF Creation Settings

For maximum conversion performance, create TIFFs with **strip-based** layout:

#### Using libvips CLI

```bash
# JPEG2000 J2K format (recommended)
vips tiffsave input.jpg output.tif \
  --compression=j2k-minervasoft \
  --Q=85 \
  --tile=false

# JPEG2000 JP2 format
vips tiffsave input.jpg output.tif \
  --compression=jp2-inzisoft \
  --Q=85 \
  --tile=false
```

#### Using Python (pyvips)

```python
import pyvips

image = pyvips.Image.new_from_file('input.jpg')
image.tiffsave('output.tif',
    compression='j2k-minervasoft',  # or 'jp2-inzisoft'
    Q=85,
    tile=False,                      # Force strip-based
    pyramid=False)
```

#### Using libtiff

```c
#include <tiffio.h>

TIFF *tif = TIFFOpen("output.tif", "w");
TIFFSetField(tif, TIFFTAG_IMAGEWIDTH, width);
TIFFSetField(tif, TIFFTAG_IMAGELENGTH, height);
TIFFSetField(tif, TIFFTAG_COMPRESSION, 34713);  // J2K
TIFFSetField(tif, TIFFTAG_PHOTOMETRIC, PHOTOMETRIC_RGB);
TIFFSetField(tif, TIFFTAG_ROWSPERSTRIP, height);  // Full-height strip
// ... write data
TIFFClose(tif);
```

### 📊 Performance Comparison

**Benchmark**: 121-page document conversion (816×1056 pixels per page)

| TIFF Format | Structure | Conversion Time | Performance |
|-------------|-----------|-----------------|-------------|
| **34713 (J2K) strip-based** | Rows/Strip = height | **0.3 seconds** | ⚡⚡⚡ Optimal |
| **34712 (JP2) strip-based** | Rows/Strip = height | **0.3 seconds** | ⚡⚡⚡ Optimal |
| **33004 strip-based** | Rows/Strip = height | **0.3 seconds** | ⚡⚡⚡ Optimal |
| **33004 tiled** | Tile 128×128 | **22 seconds** | 🐌 74× slower |

### ✅ Verification

Check if your TIFF is optimized:

```bash
tiffinfo your-file.tif | grep -E "(Tile|Rows/Strip)"

# Optimal (strip-based):
#   Rows/Strip: 1056  (equals image height)

# Slow (tiled):
#   Tile Width: 128 Tile Length: 128
```

### 💡 Recommendations for Vendors/Tools

If you're generating TIFFs for high-volume PDF conversion:

1. **Use strip-based layout** instead of tiled
2. **Set Rows/Strip = image height** (entire image as one strip)
3. **Prefer compression 34713** (J2K codestream) for best compatibility
4. **Alternative: 34712** (JP2 container) also supports zero-copy

**Impact**: Switching from tiled to strip-based provides **74× performance improvement** with no quality loss.

## Architecture

### Project Structure

```
mcp-janus-converter/
├── package.json
├── README.md
├── LICENSE (MPL 2.0)
├── NOTICE
├── index.js (Unified MCP Server)
├── binding.gyp (Multi-target build)
├── src/
│   ├── pdf2tiff/
│   │   ├── addon.c (N-API wrapper)
│   │   ├── pdf_parser.c (Custom PDF parser)
│   │   ├── pdf_parser.h
│   │   ├── pdf2tiff.c (TIFF generation)
│   │   └── pdf2tiff.h
│   └── tiff2pdf/
│       ├── addon.c (N-API wrapper)
│       ├── tiff_parser.c (TIFF metadata parser)
│       ├── tiff2pdf.c (PDF generation)
│       └── tiff2pdf.h
└── build/Release/
    ├── pdf2tiff.node
    └── tiff2pdf.node
```

### Multi-Target Build

The `binding.gyp` configuration builds **two independent native addons** from a single project:

1. **pdf2tiff.node**: PDF → TIFF conversion
2. **tiff2pdf.node**: TIFF → PDF conversion

**Benefits**:
- Independent versioning and updates
- Modular architecture (can use separately if needed)
- Easier testing and debugging
- Clear separation of concerns

### Async Architecture

Both native addons use **N-API Async** with the following pattern:

```c
typedef struct {
  napi_async_work work;
  napi_deferred deferred;

  /* Input parameters */
  char source_path[4096];
  char target_path[4096];
  int verbose;

  /* Output results */
  int result_code;
  int page_count;
  char error_message[256];
} ConvertAsyncData;
```

**Execution Flow**:
1. **Main Thread**: Parse arguments, create Promise
2. **Worker Thread** (libuv): Execute conversion
3. **Main Thread**: Resolve/reject Promise

## ⚖️ License & Philosophy

**@janus-mcp/converter is licensed under the Mozilla Public License 2.0 (MPL 2.0).**

**Copyright (c) 2025 Janus Project**

### What this means for you:

✅ **You CAN** use this package in your commercial/proprietary applications freely.
✅ **You CAN** link against this package without opening your application's source code.
❌ **You CANNOT** modify source files and distribute them as closed source.
❌ **You CANNOT** create a "Pro" version by hiding our core optimizations.

### Why MPL 2.0?

We chose this license to escape the restrictiveness of AGPL (like MuPDF) while preventing
unconscionable vendors from exploiting our **Zero-Copy Innovation** to create locked-in,
closed derivatives.

**"Input is Innovation, Output is Delegation."** We share our innovation freely, provided
you respect the openness of the core technology.

### Important Notice

See [NOTICE](./NOTICE) file for SDK vendor warnings.

## Development

### Build Commands

```bash
# Clean build
npm run clean

# Build native addons
npm run build

# Rebuild (clean + build)
npm install
```

### Testing

```bash
# Test PDF → TIFF conversion
node -e "
const pdf2tiff = require('./build/Release/pdf2tiff.node');
(async () => {
  const result = await pdf2tiff.ConvertAsync('input.pdf', 'output.tif', false);
  console.log(result);
})();
"

# Test TIFF → PDF conversion
node -e "
const tiff2pdf = require('./build/Release/tiff2pdf.node');
(async () => {
  const result = await tiff2pdf.ConvertAsync('input.tif', 'output.pdf', 0);
  console.log(result);
})();
"
```

### Debugging

```bash
# Enable verbose output
node index.js --verbose

# Check native addon loading
node -e "
const pdf2tiff = require('./build/Release/pdf2tiff.node');
const tiff2pdf = require('./build/Release/tiff2pdf.node');
console.log('pdf2tiff version:', pdf2tiff.GetVersion());
console.log('tiff2pdf version:', tiff2pdf.GetVersion());
"
```

### 🚀 Synergy with Modern Browsers (Chrome OCR)

**Janus + Chrome = Instant OCR & Searchability**

Recently, Google Chrome added built-in AI OCR for image-based PDFs. This creates a powerful synergy with Janus:

1.  **Janus** converts unreadable proprietary TIFFs into standard PDFs in milliseconds.
2.  **Chrome** opens the PDF and automatically recognizes text within the images.
3.  **Result:** You can **select, copy, and search text** from legacy scanned documents without any heavy OCR software on your server.

> **"Zero-Server-Load OCR"**: Janus prepares the document, Chrome provides the intelligence.

### ❓ Troubleshooting / FAQ

**Q: I can't open my TIFF file. It shows a black screen or error.**

> **A:** This is likely due to proprietary compression tags.
> If your TIFF uses tags like **34712 (InziSoft)** or **34713 (Minervasoft)**, standard image viewers cannot decode them.
>
> **Janus fixes this:** It extracts the hidden JPEG2000 stream and wraps it in a standard PDF, making it viewable in Chrome, Edge, or Acrobat.

### Native Addon Build Failures

**Error**: `Cannot find module 'build/Release/pdf2tiff.node'`

**Solution**:
```bash
npm run build
```

### libtiff Not Found

**Error**: `error while loading shared libraries: libtiff.so.6`

**Solution**:
```bash
# Ubuntu/Debian
sudo apt-get install libtiff-dev

# macOS
brew install libtiff

# Set LD_LIBRARY_PATH (if needed)
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```

### Conversion Failures

**Error**: `Failed to open PDF file` or `Failed to open TIFF file`

**Checklist**:
1. Verify input file exists and is readable
2. Check file permissions
3. Ensure file path is absolute or correct relative path
4. Validate file format (not corrupted)

## Contributing

We welcome contributions! Please read our [Contributing Guide](CONTRIBUTING.md) before submitting pull requests.

### Development Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Commit with conventional commits (`git commit -m "feat: add amazing feature"`)
5. Push to your branch (`git push origin feature/amazing-feature`)
6. Open a Pull Request

## Roadmap

- [x] **v1.0.0**: Windows x64 prebuilt binaries
- [x] **v1.1.0**: Linux x64 prebuilt binaries (Ubuntu 20.04+, glibc 2.31+)
- [ ] **v1.2.0**: macOS ARM prebuilt binaries
- [ ] **v1.3.0**: Advanced PDF metadata extraction
- [ ] **v1.4.0**: TIFF annotation rendering
- [ ] **v2.0.0**: Support for additional compression formats

## Credits

- **libvips**: Demand-driven image processing library
- **libtiff**: TIFF reading/writing library
- **OpenJPEG**: JPEG 2000 codec library
- **MCP SDK**: Model Context Protocol by Anthropic

## Support

- **Issues**: [GitHub Issues](https://github.com/edwardkim/janus-mcp/issues)
- **Discussions**: [GitHub Discussions](https://github.com/edwardkim/janus-mcp/discussions)
- **Email**: tangokorea@gmail.com

## License

This project is licensed under the Mozilla Public License 2.0 - see the [LICENSE](LICENSE) file for details.

---

**Made with ❤️ by the Janus Project**
