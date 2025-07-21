# SREC Utility 🔧

A powerful, browser-based tool for bidirectional conversion between Binary (.bin) and Motorola S-Record (.srec) files. Built as a single HTML file with no external dependencies.

![SREC Utility Screenshot](https://img.shields.io/badge/Format-SREC%20%E2%86%94%20BIN-blue?style=for-the-badge)
![No Dependencies](https://img.shields.io/badge/Dependencies-None-green?style=for-the-badge)
![Browser Compatible](https://img.shields.io/badge/Browser-Compatible-orange?style=for-the-badge)

## ✨ Features

- **🔄 Bidirectional Conversion**: BIN ↔ SREC with automatic file type detection
- **📁 Drag & Drop Interface**: Simply drop files to convert
- **📊 Real-time Progress**: Smooth progress indicators for all file sizes
- **🎯 Smart Detection**: Automatic file type recognition beyond just extensions
- **⚡ Streaming Processing**: Handles large files (up to 300MB+) efficiently
- **🔧 Configurable Options**: Customizable address offsets, record types, and byte counts
- **📋 File Analysis**: Detailed SREC file information display
- **🎨 Modern UI**: Clean, responsive design with smooth animations
- **💾 No Installation**: Single HTML file - works offline

## 🚀 Quick Start

1. Download `srec-util.html`
2. Open in any modern web browser
3. Drag and drop your .bin or .srec file
4. Configure options if needed
5. Click "Convert" and download the result



## 📖 How to Use


### Converting BIN to SREC

<p align="left">
  <img src="https://github.com/user-attachments/assets/ff90f40f-e9c8-4e75-bace-3302437e6188" alt="bin to srec" width="55%">
</p>

1. **Drop a .bin file** into the interface
2. **Configure conversion settings**:
   - **Start Address**: Memory address where data begins (default: 0x01600000)
   - **Address Offset**: Additional offset to add (default: 0x00000000)  
   - **Record Type**: S1 (16-bit), S2 (24-bit), or S3 (32-bit addresses)
   - **Bytes per Record**: Data bytes per SREC line (1-255, default: 32)
   - **Hex Case**: Uppercase or lowercase hex output
3. **Click "Convert to SREC"**
4. **Download** the generated .srec file

### Converting SREC to BIN

<p align="left">
  <img src="https://github.com/user-attachments/assets/a3b6fcef-0a90-48ac-b8f7-b85e0cad3dcd" alt="srec to bin" width="55%">
</p>

1. **Drop a .srec file** into the interface
2. **View detected file information**:
   - Number of data records
   - Record types present
   - Start address and address range
   - Total data size
3. **Click "Convert to BIN"**
4. **Download** the extracted binary data

### Smart Features
- **Automatic Type Detection**: Files are analyzed by content, not just extension
- **Termination Preservation**: SREC→BIN→SREC roundtrips preserve original termination behavior
- **Progress Tracking**: Real-time progress for all conversions
- **Error Handling**: Clear error messages for invalid files

## 🔧 Technical Details

### Supported SREC Record Types
| Record | Address Width | Usage |
|--------|---------------|--------|
| S0     | -             | Header (optional) |
| S1     | 16-bit        | Data with 2-byte addresses |
| S2     | 24-bit        | Data with 3-byte addresses |
| S3     | 32-bit        | Data with 4-byte addresses |
| S7/S8/S9 | 32/24/16-bit | Termination (optional) |

### File Processing Architecture

#### BIN → SREC Conversion
```
Binary File → Stream Reader → Chunk Processor → SREC Generator → Output
     ↓              ↓              ↓              ↓              ↓
  File API    32KB Chunks    Progress Update   Checksum       Download
```

#### SREC → BIN Conversion  
```
SREC File → Two-Pass Parser → Address Mapping → Binary Builder → Output
    ↓            ↓               ↓               ↓              ↓
Text Lines   Range Analysis   Memory Layout   Uint8Array     Download
```

### Memory Optimization

**Problem**: Large SREC files can cause "Maximum call stack size exceeded" errors when using JavaScript Maps.

**Solution**: Two-pass processing approach:
1. **Pass 1**: Analyze address range without storing data
2. **Pass 2**: Direct write to pre-allocated Uint8Array

This allows processing of 300MB+ files without memory issues.

### Streaming Implementation

All file processing uses streaming with 1ms yields to maintain UI responsiveness:

```javascript
// Yield control every 100 records for smooth progress
if (i % (byteCount * 100) === 0) {
    const progress = (i / data.length) * 90;
    this.updateProgress(progress, 'Converting to SREC');
    await new Promise(resolve => setTimeout(resolve, 1));
}
```

## 📋 SREC Format Details

### Record Structure
```
S<type><length><address><data><checksum>
│   │      │        │       │       │
│   │      │        │       │       └─ 1-byte checksum
│   │      │        │       └─ Data bytes (0-255)
│   │      │        └─ Address (2-4 bytes)
│   │      └─ Record length (address + data + checksum)
│   └─ Record type (0-9)
└─ Start character
```

### Example S3 Record
```
S325003BFFE00FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF3A
│││  │       │                                                             │
│││  │       └─ 32 bytes of data (all 0xFF)                                │
│││  └─ Start address: 0x003BFFE0                                          │
││└─ Length: 0x25 (37 bytes = 4 addr + 32 data + 1 checksum)              │
│└─ Type: S3 (32-bit address)                                              │
└─ Checksum: 0x3A
```

### Checksum Calculation
```javascript
checksum = (~sum(length + address_bytes + data_bytes)) & 0xFF
```

## 🌐 Browser Compatibility

- ✅ **Chrome/Chromium** 60+
- ✅ **Firefox** 55+  
- ✅ **Safari** 12+
- ✅ **Edge** 79+

**Required APIs**: File API, FileReader, Drag & Drop API, ES6+ JavaScript

## 🛠️ Technical Implementation

### Core Technologies
- **Vanilla JavaScript**: No external dependencies
- **File API**: For reading local files
- **Streaming**: Chunk-based processing for large files
- **Web Workers**: Not used - single-threaded with yielding
- **CSS3**: Modern animations and transitions

### Key Algorithms
- **Checksum**: Two's complement verification
- **Address Mapping**: Sparse memory layout handling  
- **Progress Calculation**: Real-time percentage tracking
- **Type Detection**: Content-based file analysis

### Performance Optimizations
- **Chunked Reading**: 32KB file chunks
- **Efficient Yielding**: 1ms setTimeout for UI updates
- **Memory Pre-allocation**: Avoid dynamic resizing
- **Progress Batching**: Update every 100 iterations

## 📁 File Structure

```
srec-util.html          # Complete application (single file)
├── HTML Structure      # UI layout and components  
├── CSS Styling        # Modern design with animations
└── JavaScript Logic   # Core conversion algorithms
    ├── File Handling  # Drag/drop and file I/O
    ├── SREC Parser    # Two-pass parsing algorithm
    ├── BIN Converter  # Streaming SREC generation
    ├── UI Management  # Progress and state updates
    └── Utilities      # Checksum, formatting, etc.
```

## 🚀 Advanced Usage

### Batch Processing
While the UI handles single files, the core functions support batch processing:

```javascript
// Example: Process multiple files programmatically
const converter = new SrecConverter();
for (const file of files) {
    const result = await converter.handleFile(file);
    // Handle result
}
```

### Custom Integration
The conversion functions can be extracted for use in other applications:

```javascript
// Core conversion functions
binToSrecWithProgress(data, options)    // Returns SREC string
srecToBinWithProgress(text)             // Returns Uint8Array
```

## 🐛 Troubleshooting

### Common Issues

**File not recognized**
- Ensure file contains valid SREC records (S0-S9)
- Check for proper hex formatting
- Verify checksum integrity

**Large file processing slow**
- Expected for 100MB+ files
- Progress bar shows real-time status
- Browser may become unresponsive temporarily

**Conversion errors**
- Invalid hex characters in SREC
- Corrupted file structure
- Unsupported record types

### Error Messages
- `"Invalid SREC format"` → Check file structure
- `"Address range too large"` → File exceeds 512MB limit  
- `"Maximum call stack exceeded"` → Should not occur (fixed)

## 📝 License

This project is open source and available under the MIT License.

## 🤝 Contributing

Contributions are welcome! Areas for improvement:
- Additional record type support
- Batch file processing UI
- Export format options
- Performance optimizations

## 🔗 Related Projects

- [SRecord Tools](http://srecord.sourceforge.net/) - Command-line utilities
- [Motorola S-Record Specification](https://en.wikipedia.org/wiki/SREC_(file_format)) - Format documentation

---
