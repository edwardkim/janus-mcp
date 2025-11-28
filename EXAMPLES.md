# Janus MCP Converter - Usage Examples

This document provides working examples demonstrating the capabilities of Janus MCP Converter.

## Prerequisites

Ensure the MCP server is configured in your Claude Desktop:

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

---

## Example 1: Convert Multi-page TIFF to PDF

**Use Case:** Convert a scanned document stored as multi-page TIFF to PDF for easier sharing and viewing.

**Prompt:**
```
Convert the TIFF file at C:\Documents\scanned_contract.tiff to PDF format
```

**Tool Called:** `convert_tiff_to_pdf`

**Expected Input:**
```json
{
  "tiff_path": "C:\\Documents\\scanned_contract.tiff",
  "pdf_path": "C:\\Documents\\scanned_contract.pdf"
}
```

**Expected Output:**
```json
{
  "success": true,
  "message": "Conversion successful",
  "input": {
    "path": "C:\\Documents\\scanned_contract.tiff",
    "size": 1932456,
    "sizeFormatted": "1.84 MB"
  },
  "output": {
    "path": "C:\\Documents\\scanned_contract.pdf",
    "size": 1928234,
    "sizeFormatted": "1.84 MB",
    "exists": true
  },
  "conversion": {
    "pageCount": 9,
    "elapsedMs": 4,
    "avgMsPerPage": "0.44"
  }
}
```

**Key Features Demonstrated:**
- Multi-page TIFF support
- Zero-copy architecture (near-identical file sizes)
- Ultra-fast conversion (~4ms for 9 pages)

---

## Example 2: Convert PDF to TIFF for Archival

**Use Case:** Convert a PDF document to TIFF format for long-term archival in a document management system that requires TIFF.

**Prompt:**
```
I need to archive this PDF as a TIFF file. Convert C:\Archive\invoice_2024.pdf to TIFF format in the same folder.
```

**Tool Called:** `convert_pdf_to_tiff`

**Expected Input:**
```json
{
  "pdf_path": "C:\\Archive\\invoice_2024.pdf",
  "tiff_path": "C:\\Archive\\invoice_2024.tiff"
}
```

**Expected Output:**
```json
{
  "success": true,
  "message": "Conversion successful",
  "input": {
    "path": "C:\\Archive\\invoice_2024.pdf",
    "size": 524288,
    "sizeFormatted": "0.50 MB"
  },
  "output": {
    "path": "C:\\Archive\\invoice_2024.tiff",
    "size": 528640,
    "sizeFormatted": "0.50 MB",
    "exists": true
  },
  "conversion": {
    "pageCount": 3,
    "elapsedMs": 2,
    "avgMsPerPage": "0.67"
  }
}
```

**Key Features Demonstrated:**
- PDF to TIFF conversion
- Preservation of image quality
- Suitable for archival workflows

---

## Example 3: Check File Information Before Conversion

**Use Case:** Verify file existence and size before performing a batch conversion operation.

**Prompt:**
```
Before I convert these files, can you check if C:\Batch\document1.pdf and C:\Batch\document2.tiff exist and tell me their sizes?
```

**Tools Called:** `get_pdf_info` and `get_tiff_info`

**Expected Input (PDF):**
```json
{
  "pdf_path": "C:\\Batch\\document1.pdf"
}
```

**Expected Output (PDF):**
```json
{
  "success": true,
  "path": "C:\\Batch\\document1.pdf",
  "normalizedPath": "C:\\Batch\\document1.pdf",
  "exists": true,
  "size": 2097152,
  "sizeFormatted": "2.00 MB",
  "isFile": true,
  "isDirectory": false,
  "error": null
}
```

**Expected Input (TIFF):**
```json
{
  "tiff_path": "C:\\Batch\\document2.tiff"
}
```

**Expected Output (TIFF):**
```json
{
  "success": true,
  "path": "C:\\Batch\\document2.tiff",
  "normalizedPath": "C:\\Batch\\document2.tiff",
  "exists": true,
  "size": 3145728,
  "sizeFormatted": "3.00 MB",
  "isFile": true,
  "isDirectory": false,
  "error": null
}
```

**Key Features Demonstrated:**
- Pre-flight validation
- File metadata retrieval
- Path normalization for Windows

---

## Example 4: Convert JPEG2000 TIFF (Tag 34712) to Viewable PDF

**Use Case:** Open a proprietary TIFF file that uses JPEG2000 compression (tag 34712) which standard viewers cannot display.

**Prompt:**
```
I have a TIFF file that won't open in any viewer. It says "unsupported compression". Can you convert it to PDF so I can view it?
```

**Tool Called:** `convert_tiff_to_pdf`

**Expected Input:**
```json
{
  "tiff_path": "C:\\Legacy\\proprietary_scan.tiff",
  "pdf_path": "C:\\Legacy\\proprietary_scan.pdf",
  "verbose": true
}
```

**Expected Output:**
```json
{
  "success": true,
  "message": "Conversion successful",
  "input": {
    "path": "C:\\Legacy\\proprietary_scan.tiff",
    "size": 15728640,
    "sizeFormatted": "15.00 MB"
  },
  "output": {
    "path": "C:\\Legacy\\proprietary_scan.pdf",
    "size": 15732480,
    "sizeFormatted": "15.00 MB",
    "exists": true
  },
  "conversion": {
    "pageCount": 45,
    "elapsedMs": 18,
    "avgMsPerPage": "0.40"
  }
}
```

**Key Features Demonstrated:**
- JPEG2000 (tag 34712, 34713) support
- Legacy document compatibility
- Standard PDF output viewable in Chrome, Edge, Acrobat

---

## Example 5: Batch Conversion with Verbose Output

**Use Case:** Convert multiple files while monitoring progress.

**Prompt:**
```
Convert all my scanned TIFFs to PDFs with detailed progress output:
1. C:\Scans\page1.tiff → C:\PDFs\page1.pdf
2. C:\Scans\page2.tiff → C:\PDFs\page2.pdf
3. C:\Scans\page3.tiff → C:\PDFs\page3.pdf
```

**Tool Called:** `convert_tiff_to_pdf` (3 times)

**Expected Behavior:**
- Each file is converted individually
- Verbose output shows page-by-page progress
- Total time for all 3 files: typically < 15ms

**Key Features Demonstrated:**
- Batch processing support
- Verbose logging option
- Consistent high performance

---

## Supported Formats

### Input TIFF Compression Types
| Tag | Compression | Support |
|-----|-------------|---------|
| 34712 | JPEG2000 (JP2) | ✅ Full |
| 34713 | JPEG2000 (J2K) | ✅ Full |
| 33004 | libvips JPEG2000 | ✅ Full |
| 34663 | JBIG2 | ✅ Full |
| Standard | LZW, JPEG, etc. | ✅ Full |

### Output Formats
- **PDF**: Standard PDF 1.4+ compatible with all viewers
- **TIFF**: Multi-page TIFF with preserved compression

---

## Error Handling Examples

### File Not Found
```json
{
  "success": false,
  "error": "TIFF file not found: C:\\Missing\\file.tiff (original: C:\\Missing\\file.tiff)"
}
```

### Invalid Path
```json
{
  "success": false,
  "error": "Path is not a file: C:\\SomeFolder"
}
```

---

## Performance Benchmarks

| Document Type | Pages | File Size | Conversion Time |
|---------------|-------|-----------|-----------------|
| Standard TIFF | 9 | 1.9 MB | ~4 ms |
| JPEG2000 TIFF | 45 | 15 MB | ~18 ms |
| PDF to TIFF | 3 | 0.5 MB | ~2 ms |

**Note:** Times measured on Windows 11, Intel Core i7, SSD storage.
