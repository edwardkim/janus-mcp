# Node.js 개발자를 위한 Janus Converter 사용 가이드

**작성일**: 2025-11-30
**대상**: Node.js 백엔드 개발자, 문서 처리 파이프라인 구축자

---

## 소개

`@janus-mcp/converter`는 Claude Desktop MCP 서버뿐만 아니라 **Node.js 애플리케이션에서 직접 사용할 수 있는 고성능 TIFF ↔ PDF 변환 라이브러리**입니다.

> **sharp**를 이미지 처리에 사용하듯, **@janus-mcp/converter**를 TIFF ↔ PDF 변환에 사용하세요.

### 왜 @janus-mcp/converter를 사용해야 할까요?

| 비교 항목 | ImageMagick | sharp | @janus-mcp/converter |
|-----------|-------------|-------|----------------------|
| TIFF→PDF 속도 | 느림 (decode/encode) | 미지원 | **1,400+ pages/sec** |
| PDF→TIFF 속도 | 느림 | 미지원 | **1,900+ pages/sec** |
| JPEG2000 지원 | 제한적 | 미지원 | **완벽 지원 (Zero-Copy)** |
| 멀티페이지 TIFF | 지원 | 미지원 | **네이티브 지원** |
| 설치 복잡도 | 높음 (시스템 의존성) | 낮음 | **낮음 (prebuild 포함)** |

---

## 빠른 시작

### 1. 설치

```bash
npm install @janus-mcp/converter
```

### 2. 기본 사용법

```javascript
// converter.js
const path = require('path');

// Native addon 직접 로드
const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

async function convertPdfToTiff(pdfPath, tiffPath) {
  const result = await pdf2tiff.ConvertAsync(pdfPath, tiffPath, false);

  if (result.success) {
    console.log(`변환 성공: ${result.pageCount} 페이지`);
    return result;
  } else {
    throw new Error(result.message);
  }
}

async function convertTiffToPdf(tiffPath, pdfPath) {
  const result = await tiff2pdf.ConvertAsync(tiffPath, pdfPath, 0);

  if (result.success) {
    console.log(`변환 성공: ${result.pageCount} 페이지`);
    return result;
  } else {
    throw new Error(result.message);
  }
}

// 사용 예시
(async () => {
  try {
    // PDF → TIFF
    await convertPdfToTiff('./input.pdf', './output.tif');

    // TIFF → PDF
    await convertTiffToPdf('./input.tif', './output.pdf');
  } catch (err) {
    console.error('변환 실패:', err.message);
  }
})();
```

---

## 상세 API

### PDF → TIFF 변환

#### ConvertAsync (비동기, 권장)

```javascript
const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');

const result = await pdf2tiff.ConvertAsync(pdfPath, tiffPath, verbose);
```

**매개변수**:
| 이름 | 타입 | 설명 |
|------|------|------|
| `pdfPath` | string | 입력 PDF 파일 경로 |
| `tiffPath` | string | 출력 TIFF 파일 경로 |
| `verbose` | boolean | 상세 로그 출력 여부 |

**반환값**:
```javascript
{
  success: true,          // 성공 여부
  pageCount: 9,           // 변환된 페이지 수
  message: "Conversion successful"
}
```

#### ConvertSync (동기)

```javascript
// 이벤트 루프 차단 - 배치 작업에만 권장
const result = pdf2tiff.ConvertSync(pdfPath, tiffPath, verbose);
```

### TIFF → PDF 변환

#### ConvertAsync (비동기, 권장)

```javascript
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

const result = await tiff2pdf.ConvertAsync(tiffPath, pdfPath, verboseInt);
```

**매개변수**:
| 이름 | 타입 | 설명 |
|------|------|------|
| `tiffPath` | string | 입력 TIFF 파일 경로 |
| `pdfPath` | string | 출력 PDF 파일 경로 |
| `verboseInt` | number | 상세 로그 (0: 끔, 1: 켬) |

#### JPEG2000 정규화 옵션

```javascript
const result = await tiff2pdf.ConvertAsync(tiffPath, pdfPath, {
  normalizeJpeg2000: true,    // 33004, 34713 → 34712 변환
  normalizeFallback: 1        // 0=에러, 1=원본 유지, 2=비압축
});

console.log(`정규화 성공: ${result.normalizeSuccessCount}페이지`);
console.log(`정규화 실패(폴백): ${result.normalizeFailedCount}페이지`);
```

---

## 실전 예제

### 예제 1: Express 파일 업로드 변환 서버

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

// PDF → TIFF 변환 엔드포인트
app.post('/convert/pdf-to-tiff', upload.single('file'), async (req, res) => {
  try {
    const inputPath = req.file.path;
    const outputPath = inputPath + '.tif';

    const result = await pdf2tiff.ConvertAsync(inputPath, outputPath, false);

    if (result.success) {
      res.download(outputPath, 'converted.tif', async () => {
        // 임시 파일 정리
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

// TIFF → PDF 변환 엔드포인트
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
  console.log('변환 서버 실행 중: http://localhost:3000');
});
```

### 예제 2: 대량 배치 변환

```javascript
// batch-convert.js
const fs = require('fs').promises;
const path = require('path');
const tiff2pdf = require('@janus-mcp/converter/build/Release/tiff2pdf.node');

async function batchConvertTiffToPdf(inputDir, outputDir) {
  const files = await fs.readdir(inputDir);
  const tiffFiles = files.filter(f => /\.(tif|tiff)$/i.test(f));

  console.log(`${tiffFiles.length}개 파일 변환 시작...`);

  const results = {
    success: 0,
    failed: 0,
    totalPages: 0,
    errors: []
  };

  const startTime = Date.now();

  // 병렬 변환 (동시 5개)
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

    console.log(`진행: ${Math.min(i + concurrency, tiffFiles.length)}/${tiffFiles.length}`);
  }

  const elapsed = (Date.now() - startTime) / 1000;

  console.log('\n=== 변환 완료 ===');
  console.log(`성공: ${results.success}개`);
  console.log(`실패: ${results.failed}개`);
  console.log(`총 페이지: ${results.totalPages}페이지`);
  console.log(`소요 시간: ${elapsed.toFixed(2)}초`);
  console.log(`처리량: ${(results.totalPages / elapsed).toFixed(0)} pages/sec`);

  if (results.errors.length > 0) {
    console.log('\n실패 목록:');
    results.errors.forEach(e => console.log(`  - ${e.file}: ${e.error}`));
  }

  return results;
}

// 실행
batchConvertTiffToPdf('./input', './output');
```

### 예제 3: 스트림 기반 처리 (임시 파일 사용)

```javascript
// stream-convert.js
const fs = require('fs');
const path = require('path');
const os = require('os');
const { v4: uuidv4 } = require('uuid');

const pdf2tiff = require('@janus-mcp/converter/build/Release/pdf2tiff.node');

async function convertPdfBufferToTiff(pdfBuffer) {
  // 임시 파일에 버퍼 저장
  const tempDir = os.tmpdir();
  const tempPdf = path.join(tempDir, `${uuidv4()}.pdf`);
  const tempTiff = path.join(tempDir, `${uuidv4()}.tif`);

  try {
    // PDF 버퍼를 파일로 저장
    fs.writeFileSync(tempPdf, pdfBuffer);

    // 변환 실행
    const result = await pdf2tiff.ConvertAsync(tempPdf, tempTiff, false);

    if (!result.success) {
      throw new Error(result.message);
    }

    // 결과를 버퍼로 읽기
    const tiffBuffer = fs.readFileSync(tempTiff);

    return {
      buffer: tiffBuffer,
      pageCount: result.pageCount
    };
  } finally {
    // 임시 파일 정리
    try { fs.unlinkSync(tempPdf); } catch {}
    try { fs.unlinkSync(tempTiff); } catch {}
  }
}

// 사용 예시
async function example() {
  const pdfBuffer = fs.readFileSync('./sample.pdf');
  const result = await convertPdfBufferToTiff(pdfBuffer);

  console.log(`변환 완료: ${result.pageCount}페이지, ${result.buffer.length} bytes`);
  fs.writeFileSync('./output.tif', result.buffer);
}
```

### 예제 4: Worker Threads를 활용한 병렬 처리

```javascript
// worker-convert.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const path = require('path');
const os = require('os');

if (isMainThread) {
  // 메인 스레드
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

  // 사용
  const files = [
    { input: './file1.tif', output: './file1.pdf' },
    { input: './file2.tif', output: './file2.pdf' },
    // ...
  ];

  parallelConvert(files).then(results => {
    console.log('모든 변환 완료:', results);
  });

} else {
  // 워커 스레드
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

## 성능 최적화 팁

### 1. 비동기 API 사용

```javascript
// 권장: 비동기 (이벤트 루프 차단 없음)
const result = await pdf2tiff.ConvertAsync(input, output, false);

// 비권장: 동기 (이벤트 루프 차단)
const result = pdf2tiff.ConvertSync(input, output, false);
```

### 2. 적절한 동시성 설정

```javascript
// CPU 코어 수에 맞춰 동시성 조절
const concurrency = Math.min(os.cpus().length, 8);
```

### 3. 대용량 파일은 스트리밍 처리

```javascript
// 메모리 효율적인 처리
// 파일 → 변환 → 파일 (버퍼 최소화)
await tiff2pdf.ConvertAsync(inputPath, outputPath, 0);
```

---

## 웹 프론트엔드 연동 (REST API)

브라우저에서는 Native Addon을 직접 사용할 수 없습니다. **REST API 서버**를 구축하여 웹 애플리케이션과 연동하세요.

> **WASM 미지원 이유**: WebAssembly로 빌드하면 Zero-Copy 아키텍처의 성능 이점이 사라집니다 (파일 시스템 직접 접근 불가, 메모리 복사 필수). 설계 원칙에 따라 WASM 버전은 제공하지 않습니다.

### REST API 서버 (완전한 예제)

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

// CORS 설정 (프론트엔드 도메인 허용)
app.use(cors({
  origin: ['http://localhost:3000', 'https://your-frontend.com'],
  methods: ['POST', 'GET']
}));

// 파일 업로드 설정
const storage = multer.diskStorage({
  destination: os.tmpdir(),
  filename: (req, file, cb) => {
    const uniqueName = `${crypto.randomUUID()}${path.extname(file.originalname)}`;
    cb(null, uniqueName);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 100 * 1024 * 1024 }, // 100MB 제한
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['.pdf', '.tif', '.tiff'];
    const ext = path.extname(file.originalname).toLowerCase();
    if (allowedTypes.includes(ext)) {
      cb(null, true);
    } else {
      cb(new Error('지원하지 않는 파일 형식입니다.'));
    }
  }
});

// 임시 파일 정리 함수
async function cleanup(...files) {
  for (const file of files) {
    try { await fs.unlink(file); } catch {}
  }
}

// PDF → TIFF 변환 API
app.post('/api/convert/pdf-to-tiff', upload.single('file'), async (req, res) => {
  const inputPath = req.file?.path;
  if (!inputPath) {
    return res.status(400).json({ success: false, error: '파일이 필요합니다.' });
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

// TIFF → PDF 변환 API
app.post('/api/convert/tiff-to-pdf', upload.single('file'), async (req, res) => {
  const inputPath = req.file?.path;
  if (!inputPath) {
    return res.status(400).json({ success: false, error: '파일이 필요합니다.' });
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

// 헬스 체크
app.get('/api/health', (req, res) => {
  res.json({
    status: 'ok',
    version: '1.0.0',
    timestamp: new Date().toISOString()
  });
});

// 에러 핸들러
app.use((err, req, res, next) => {
  console.error('Server Error:', err);
  res.status(500).json({ success: false, error: err.message });
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, () => {
  console.log(`Janus Converter API 서버 실행 중: http://localhost:${PORT}`);
  console.log('엔드포인트:');
  console.log('  POST /api/convert/pdf-to-tiff  - PDF → TIFF (파일 다운로드)');
  console.log('  POST /api/convert/tiff-to-pdf  - TIFF → PDF (파일 다운로드)');
  console.log('  GET  /api/health               - 헬스 체크');
});
```

### 프론트엔드 연동 예제 (React/JavaScript)

```javascript
// 웹 브라우저에서 API 호출
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

  // 응답 헤더에서 메타데이터 추출
  const pageCount = response.headers.get('X-Page-Count');
  const elapsedMs = response.headers.get('X-Elapsed-Ms');
  console.log(`변환 완료: ${pageCount}페이지, ${elapsedMs}ms`);

  // Blob으로 다운로드
  const blob = await response.blob();
  const url = URL.createObjectURL(blob);

  // 자동 다운로드
  const a = document.createElement('a');
  a.href = url;
  a.download = 'converted.tif';
  a.click();

  URL.revokeObjectURL(url);
}

// 파일 입력 이벤트 핸들러
document.getElementById('fileInput').addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (file) {
    try {
      await convertPdfToTiff(file);
    } catch (err) {
      alert('변환 실패: ' + err.message);
    }
  }
});
```

### cURL 테스트

```bash
# PDF → TIFF 변환
curl -X POST -F "file=@input.pdf" http://localhost:4000/api/convert/pdf-to-tiff -o output.tif

# TIFF → PDF 변환
curl -X POST -F "file=@input.tif" http://localhost:4000/api/convert/tiff-to-pdf -o output.pdf

# 헬스 체크
curl http://localhost:4000/api/health
```

### Docker 배포

```dockerfile
# Dockerfile
FROM node:20-slim

# 의존성 설치 (Linux)
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
# 빌드 및 실행
docker build -t janus-api .
docker run -p 4000:4000 janus-api
```

---

## 지원 플랫폼

| 플랫폼 | 아키텍처 | 사전 빌드 | 비고 |
|--------|----------|-----------|------|
| Windows | x64 | ✅ | 추가 의존성 없음 |
| Linux | x64 | ✅ | libtiff, openjpeg 필요 |
| macOS | ARM64 | ✅ | Homebrew로 의존성 설치 |

---

## 문제 해결

### 모듈 로드 실패

```javascript
// 경로 확인
const addonPath = require.resolve('@janus-mcp/converter/build/Release/pdf2tiff.node');
console.log('Addon 경로:', addonPath);
```

### Linux에서 라이브러리 오류

```bash
# libtiff 설치
sudo apt-get install libtiff-dev libopenjp2-7-dev libjpeg-turbo8-dev zlib1g-dev
```

### libtiff.so.5 not found 오류 (Ubuntu 24.04, Debian 12, RHEL 9+)

최신 Linux 배포판에서 다음 오류가 발생할 수 있습니다:

```
error while loading shared libraries: libtiff.so.5: cannot open shared object file
```

**원인**: prebuild는 Ubuntu 20.04 (libtiff.so.5)에서 빌드되었지만, 최신 배포판은 libtiff.so.6을 사용합니다.

**배포판별 libtiff 버전**:

| 배포판 | libtiff 버전 | 파일명 | 조치 |
|--------|-------------|--------|------|
| Ubuntu 20.04/22.04 | 4.1~4.3 | libtiff.so.5 | 정상 작동 |
| **Ubuntu 24.04+** | 4.5+ | libtiff.so.6 | **심볼릭 링크 필요** |
| Debian 11 | 4.2 | libtiff.so.5 | 정상 작동 |
| **Debian 12+** | 4.5+ | libtiff.so.6 | **심볼릭 링크 필요** |
| RHEL/Rocky 8 | 4.0 | libtiff.so.5 | 정상 작동 |
| **RHEL/Rocky 9+** | 4.4+ | libtiff.so.6 | **심볼릭 링크 필요** |
| Fedora 38- | 4.4 | libtiff.so.5 | 정상 작동 |
| **Fedora 39+** | 4.5+ | libtiff.so.6 | **심볼릭 링크 필요** |

**해결 방법 (심볼릭 링크 생성)**:

```bash
# 1. libtiff.so.6 위치 확인
find /usr -name "libtiff.so.6*" 2>/dev/null
# 출력 예: /usr/lib/x86_64-linux-gnu/libtiff.so.6

# 2. 심볼릭 링크 생성
sudo ln -s /usr/lib/x86_64-linux-gnu/libtiff.so.6 /usr/lib/x86_64-linux-gnu/libtiff.so.5

# 3. 라이브러리 캐시 갱신
sudo ldconfig

# 4. 확인
ls -la /usr/lib/x86_64-linux-gnu/libtiff.so.5
# 출력: libtiff.so.5 -> libtiff.so.6
```

**ARM64 (aarch64) 시스템의 경우**:

```bash
sudo ln -s /usr/lib/aarch64-linux-gnu/libtiff.so.6 /usr/lib/aarch64-linux-gnu/libtiff.so.5
sudo ldconfig
```

> **참고**: libtiff.so.5와 libtiff.so.6은 이 패키지에서 사용하는 함수들에 대해 ABI 호환성이 있어 심볼릭 링크로 문제없이 작동합니다.

### 변환 실패 디버깅

```javascript
// verbose 모드 활성화
const result = await tiff2pdf.ConvertAsync(input, output, 1);
// 콘솔에 상세 로그 출력
```

---

## 라이선스

MPL 2.0 - 상용 프로젝트에서 자유롭게 사용 가능합니다.

---

**문의**: tangokorea@gmail.com
**이슈**: https://github.com/anthropics/janus-mcp/issues
