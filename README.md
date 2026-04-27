# Pdfium Android binding with Bitmap rendering
Uses pdfium library [from AOSP](https://android.googlesource.com/platform/external/pdfium/)

Forked for use with [AndroidPdfViewer](https://github.com/bmericc/AndroidPdfViewer) project.

API is highly compatible with original version, only additional methods were created.

## What's new in 1.9.2?
* Updated version to 1.9.2

## Installation
Add to _build.gradle_:

`implementation 'com.github.bmericc:pdfium-android:1.9.2'`

Library is available in Maven Central repository.

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

