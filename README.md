# Pdfium Android binding with Bitmap rendering
Uses pdfium library [from AOSP](https://android.googlesource.com/platform/external/pdfium/)

Forked for use with [AndroidPdfViewer](https://github.com/bmericc/AndroidPdfViewer) project.

API is highly compatible with original version, only additional methods were created.

## What's new in 1.9.11?
* `libjniPdfium.so` now ships with embedded native debug symbols
  (built with GN's `symbol_level=2`, `use_debug_fission=false`) so apps
  that set `android.buildTypes.release.ndk.debugSymbolLevel = 'FULL'`
  get proper Play Console crash symbolication instead of the "missing
  native debug symbols" warning. AGP strips the symbols out of the
  packaged `.so` automatically and bundles them separately — this does
  not increase app size.

## What's new in 1.9.10?
* Native libraries rebuilt from upstream PDFium source (pdfium.googlesource.com)
  via the standard depot_tools/GN/ninja toolchain, replacing the old prebuilt
  binaries that shipped with no source in this repo
* Everything (PDFium, FreeType, libpng, zlib, abseil) is now statically linked
  into a single `libjniPdfium.so` per ABI instead of four separate libraries
* All 64-bit ABIs (`arm64-v8a`, `x86_64`) are 16 KB page-size aligned
* `libc++_shared.so` is bundled alongside `libjniPdfium.so` per ABI

## Installation
Add to _build.gradle_:

`implementation 'com.github.bmericc:PdfiumAndroid:v1.9.11'`

Resolved via [JitPack](https://jitpack.io/#bmericc/PdfiumAndroid) — add
`maven { url 'https://jitpack.io' }` to your repositories.

## Usage example
``` java
import com.github.bmericc.pdfium.PdfDocument;
import com.github.bmericc.pdfium.PdfiumCore;

void openPdf() {
    ImageView iv = (ImageView) findViewById(R.id.imageView);
    ParcelFileDescriptor fd = ...;
    int pageNum = 0;
    PdfiumCore pdfiumCore = new PdfiumCore(context);
    try {
        PdfDocument pdfDocument = pdfiumCore.newDocument(fd);

        pdfiumCore.openPage(pdfDocument, pageNum);

        int width = pdfiumCore.getPageWidthPoint(pdfDocument, pageNum);
        int height = pdfiumCore.getPageHeightPoint(pdfDocument, pageNum);

        // ARGB_8888 - best quality, high memory usage, higher possibility of OutOfMemoryError
        // RGB_565 - little worse quality, twice less memory usage
        Bitmap bitmap = Bitmap.createBitmap(width, height,
                Bitmap.Config.RGB_565);
        pdfiumCore.renderPageBitmap(pdfDocument, bitmap, pageNum, 0, 0,
                width, height);

        iv.setImageBitmap(bitmap);

        pdfiumCore.closeDocument(pdfDocument); // important!
    } catch(IOException ex) {
        ex.printStackTrace();
    }
}
```

