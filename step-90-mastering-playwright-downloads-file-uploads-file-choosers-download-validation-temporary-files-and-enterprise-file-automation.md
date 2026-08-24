# Playwright with TypeScript Mastery Bootcamp

# Step 90 — Mastering Playwright Downloads, File Uploads, File Choosers, Download Validation, Temporary Files & Enterprise File Automation

## 1. Introduction

File handling is a common requirement in enterprise applications.

Typical workflows include:

- Uploading profile images
- Uploading documents
- Importing CSV files
- Uploading Excel files
- Downloading reports
- Exporting PDF files
- Downloading invoices
- Validating generated files
- Handling file chooser dialogs
- Managing temporary test files

Playwright provides dedicated APIs for these scenarios.

Important APIs include:

```text
setInputFiles()
filechooser
waitForEvent('download')
Download
path()
saveAs()
suggestedFilename()
createReadStream()
```

---

## 2. File Automation Architecture

A typical workflow looks like:

```text
Test
 ↓
Page Object
 ↓
File Upload / Download
 ↓
File System
 ↓
Validation
```

For enterprise frameworks:

```text
Test Data
 ↓
File Utility
 ↓
Page Object
 ↓
Application
 ↓
Downloaded File
 ↓
Validation
```

---

## 3. Uploading a File

If the application contains:

```html
<input
  type="file"
  id="document">
```

Playwright can use:

```ts
await page
  .locator('#document')
  .setInputFiles(
    'test-data/document.pdf'
  );
```

---

## 4. Upload Using `getByLabel()`

If the file input has an accessible label:

```ts
await page
  .getByLabel('Upload document')
  .setInputFiles(
    'test-data/document.pdf'
  );
```

This is usually more readable than a CSS selector.

---

## 5. Upload Using Test ID

Example:

```html
<input
  type="file"
  data-testid="document-upload">
```

Then:

```ts
await page
  .getByTestId(
    'document-upload'
  )
  .setInputFiles(
    'test-data/document.pdf'
  );
```

---

## 6. Uploading Multiple Files

Playwright supports multiple files:

```ts
await page
  .getByLabel('Documents')
  .setInputFiles([
    'test-data/a.pdf',
    'test-data/b.pdf',
    'test-data/c.pdf'
  ]);
```

The application must support multiple file selection.

---

## 7. File Upload Validation

After uploading:

```ts
await expect(
  page.getByText(
    'document.pdf'
  )
).toBeVisible();
```

Or validate the application-specific upload status:

```ts
await expect(
  page.getByRole(
    'status'
  )
).toContainText(
  'Upload successful'
);
```

---

## 8. File Upload with Form Submission

Example:

```ts
await page
  .getByLabel('Resume')
  .setInputFiles(
    'test-data/resume.pdf'
  );

await page.getByRole(
  'button',
  { name: 'Submit' }
).click();

await expect(
  page.getByRole('status')
).toContainText(
  'Application submitted'
);
```

---

## 9. File Chooser

Some applications do not expose a simple file input interaction.

Instead, clicking a button triggers a file chooser.

Example:

```ts
const fileChooserPromise =
  page.waitForEvent(
    'filechooser'
  );

await page.getByRole(
  'button',
  { name: 'Choose File' }
).click();

const fileChooser =
  await fileChooserPromise;

await fileChooser.setFiles(
  'test-data/document.pdf'
);
```

---

## 10. Why Register the File Chooser Listener First?

Correct sequence:

```text
Start waiting
      ↓
Click upload button
      ↓
File chooser opens
      ↓
Set file
```

Bad:

```ts
await page.getByRole(
  'button',
  { name: 'Choose File' }
).click();

await page.waitForEvent(
  'filechooser'
);
```

The event may occur before Playwright starts waiting.

---

## 11. `setInputFiles()` vs `filechooser`

Use:

```ts
setInputFiles()
```

when you can directly locate the file input.

Use:

```ts
filechooser
```

when the application opens a native file chooser after an action.

---

## 12. Removing Selected Files

You can clear a file input:

```ts
await page
  .getByLabel('Document')
  .setInputFiles([]);
```

This resets the selected files.

---

## 13. Uploading a Buffer

Playwright can upload file data without requiring an existing physical file.

Example:

```ts
await page
  .getByLabel('Document')
  .setInputFiles({
    name: 'document.txt',
    mimeType: 'text/plain',
    buffer: Buffer.from(
      'Hello Playwright'
    )
  });
```

This is useful for generated test data.

---

## 14. Generated Test Files

Example:

```ts
const content =
  Buffer.from(
    'Employee Name,Sandeep'
  );

await page
  .getByLabel('CSV File')
  .setInputFiles({
    name: 'employees.csv',
    mimeType: 'text/csv',
    buffer: content
  });
```

This avoids maintaining many static files.

---

## 15. MIME Type

When uploading a buffer:

```ts
mimeType:
  'application/pdf'
```

Common MIME types:

```text
application/pdf
text/csv
application/json
text/plain
application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
```

Use the MIME type expected by the application.

---

## 16. Download Handling

To handle a download:

```ts
const downloadPromise =
  page.waitForEvent(
    'download'
  );

await page.getByRole(
  'button',
  { name: 'Download Report' }
).click();

const download =
  await downloadPromise;
```

Again, register the event before triggering the download.

---

## 17. Why Download Event Handling Matters

A download may happen immediately after clicking.

Bad:

```ts
await page.getByText(
  'Download'
).click();

const download =
  await page.waitForEvent(
    'download'
  );
```

Better:

```ts
const downloadPromise =
  page.waitForEvent(
    'download'
  );

await page.getByText(
  'Download'
).click();

const download =
  await downloadPromise;
```

---

## 18. Suggested Filename

Use:

```ts
const filename =
  download.suggestedFilename();

console.log(filename);
```

Example:

```text
expense-report-2026.pdf
```

You can validate:

```ts
expect(filename)
  .toMatch(
    /expense-report.*\.pdf/
  );
```

---

## 19. Saving a Download

Use:

```ts
await download.saveAs(
  'downloads/report.pdf'
);
```

This provides a controlled destination.

---

## 20. Download Path

You can access the temporary download path:

```ts
const filePath =
  await download.path();
```

The path is managed by Playwright.

For test validation, `saveAs()` is often convenient because it gives you a known destination.

---

## 21. Download Failure

Check:

```ts
const failure =
  await download.failure();
```

If the download succeeded:

```text
null
```

If it failed, a failure message may be returned.

Example:

```ts
expect(
  await download.failure()
).toBeNull();
```

---

## 22. Download Filename Validation

Example:

```ts
const downloadPromise =
  page.waitForEvent(
    'download'
  );

await page.getByRole(
  'button',
  { name: 'Export PDF' }
).click();

const download =
  await downloadPromise;

expect(
  download.suggestedFilename()
).toMatch(
  /report.*\.pdf/
);
```

---

## 23. Download Extension Validation

Example:

```ts
expect(
  download.suggestedFilename()
).toMatch(
  /\.pdf$/
);
```

For CSV:

```ts
expect(
  download.suggestedFilename()
).toMatch(
  /\.csv$/
);
```

---

## 24. Download Content Validation

After saving:

```ts
await download.saveAs(
  'downloads/report.pdf'
);
```

You can use Node.js file APIs to inspect the saved file.

Example:

```ts
import fs from 'node:fs';

expect(
  fs.existsSync(
    'downloads/report.pdf'
  )
).toBe(true);
```

---

## 25. File Size Validation

Example:

```ts
const stats =
  fs.statSync(
    'downloads/report.pdf'
  );

expect(
  stats.size
).toBeGreaterThan(0);
```

This confirms that a non-empty file was created.

---

## 26. Download Validation Strategy

A robust test can validate:

```text
Download triggered
       ↓
Download succeeded
       ↓
Filename correct
       ↓
Extension correct
       ↓
File exists
       ↓
File size > 0
       ↓
Content valid
```

---

## 27. PDF Validation

A basic validation can check:

```text
File exists
Extension = .pdf
Size > 0
```

For deeper validation, use a PDF parser in the test utilities.

Do not rely only on file existence when the business requirement includes document content.

---

## 28. CSV Validation

A CSV export can be validated by reading its content.

Example:

```ts
import fs from 'node:fs';

const content =
  fs.readFileSync(
    'downloads/report.csv',
    'utf-8'
  );

expect(content)
  .toContain(
    'Employee Name'
  );
```

---

## 29. JSON Download Validation

Example:

```ts
const content =
  fs.readFileSync(
    'downloads/data.json',
    'utf-8'
  );

const data =
  JSON.parse(content);

expect(data).toHaveProperty(
  'employees'
);
```

---

## 30. Excel File Validation

For `.xlsx` files, use an appropriate Node.js library such as `xlsx` in the test project.

Conceptually:

```ts
const workbook =
  XLSX.readFile(
    'downloads/report.xlsx'
  );

const sheet =
  workbook.Sheets[
    workbook.SheetNames[0]
  ];
```

Then validate expected values.

Keep spreadsheet parsing in a utility rather than directly inside every test.

---

## 31. File System Utilities

Create:

```text
utils/
└── fileUtils.ts
```

Possible methods:

```text
fileExists()
getFileSize()
readTextFile()
deleteFile()
createTempFile()
validateExtension()
```

This centralizes file-system operations.

---

## 32. Example File Utility

```ts
import fs from 'node:fs';

export function fileExists(
  path: string
): boolean {
  return fs.existsSync(path);
}
```

Then:

```ts
expect(
  fileExists(
    'downloads/report.pdf'
  )
).toBe(true);
```

---

## 33. Temporary Files

Tests should avoid polluting the repository with generated files.

Use a temporary directory such as:

```text
test-results/
tmp/
downloads/
```

depending on your project conventions.

Do not commit generated downloads to Git.

---

## 34. Unique Download Names

Parallel tests may download files with the same name.

Use unique paths.

Example:

```ts
const outputPath =
  `test-results/report-${Date.now()}.pdf`;
```

For more robust parallel execution, include a test-specific identifier rather than relying only on timestamps.

---

## 35. Test-Specific Download Directory

A scalable framework can use:

```text
test-results/
└── downloads/
    ├── test-1/
    ├── test-2/
    └── test-3/
```

This reduces file collisions.

---

## 36. Cleanup

Remove generated files after validation.

Example:

```ts
fs.rmSync(
  outputPath,
  { force: true }
);
```

For directories:

```ts
fs.rmSync(
  directoryPath,
  {
    recursive: true,
    force: true
  }
);
```

Cleanup should run even when tests fail.

---

## 37. Cleanup with `try/finally`

Example:

```ts
const outputPath =
  'test-results/report.pdf';

try {
  await download.saveAs(
    outputPath
  );

  // validation
} finally {
  fs.rmSync(
    outputPath,
    { force: true }
  );
}
```

This prevents leftover files.

---

## 38. File Upload Validation

After upload, verify:

```text
File name
File count
Upload status
Server response
Displayed file
```

Example:

```ts
await expect(
  page.getByText(
    'document.pdf'
  )
).toBeVisible();
```

---

## 39. File Type Validation

Test supported file types:

```text
PDF
CSV
XLSX
PNG
JPG
TXT
```

Also test unsupported formats when required:

```text
EXE
ZIP
Invalid MIME
Oversized file
```

---

## 40. File Size Validation

A UI may restrict file size.

Example test matrix:

```text
1 KB       → Accept
1 MB       → Accept
5 MB       → Accept
10 MB      → Boundary
10 MB + 1  → Reject
```

Use the application's actual limits.

---

## 41. Upload Error Validation

Example:

```ts
await page
  .getByLabel('Document')
  .setInputFiles(
    'test-data/unsupported.exe'
  );

await expect(
  page.getByRole(
    'alert'
  )
).toContainText(
  'Unsupported file type'
);
```

The exact message should match the application.

---

## 42. Upload Cancellation

For file chooser workflows, cancellation can be tested by triggering the chooser and handling the appropriate user interaction according to the application's implementation.

The test should verify that:

```text
No file selected
       ↓
Form remains unchanged
```

---

## 43. Drag-and-Drop Uploads

Some applications support drag-and-drop.

If the application exposes a file input behind the component, prefer `setInputFiles()` where possible.

For custom drag-and-drop implementations, the test may need to interact with the component according to its supported DOM behavior.

Avoid recreating browser internals unless required.

---

## 44. Upload Progress

For large uploads, verify user-visible progress:

```ts
await expect(
  page.getByRole(
    'progressbar'
  )
).toBeVisible();
```

Then verify completion:

```ts
await expect(
  page.getByRole(
    'status'
  )
).toContainText(
  'Upload complete'
);
```

Avoid fixed waits.

---

## 45. Multiple File Upload Validation

Example:

```ts
await page
  .getByLabel('Documents')
  .setInputFiles([
    'test-data/a.pdf',
    'test-data/b.pdf'
  ]);

await expect(
  page.getByText('a.pdf')
).toBeVisible();

await expect(
  page.getByText('b.pdf')
).toBeVisible();
```

---

## 46. File Download and Authentication

Downloads may require an authenticated session.

The Playwright page should use the correct browser context/storage state.

Example:

```ts
const context =
  await browser.newContext({
    storageState:
      '.auth/user.json'
  });

const page =
  await context.newPage();
```

Then perform the download.

---

## 47. Download from a New Tab

Some applications open a report before downloading.

Workflow:

```text
Click Report
 ↓
New Page
 ↓
Click Export
 ↓
Download
```

Handle both events independently:

```ts
const reportPromise =
  context.waitForEvent('page');

await page.getByText(
  'Open Report'
).click();

const report =
  await reportPromise;

const downloadPromise =
  report.waitForEvent('download');

await report.getByRole(
  'button',
  { name: 'Export' }
).click();

const download =
  await downloadPromise;
```

---

## 48. Download from Popup

Example:

```ts
const popupPromise =
  page.waitForEvent('popup');

await page.getByText(
  'Open Invoice'
).click();

const popup =
  await popupPromise;

const downloadPromise =
  popup.waitForEvent(
    'download'
  );

await popup.getByText(
  'Download PDF'
).click();

const download =
  await downloadPromise;
```

This combines Step 88 concepts with file automation.

---

## 49. Download Event Ordering

Always use:

```text
Wait for download
      ↓
Trigger download
      ↓
Receive Download
      ↓
Save
      ↓
Validate
```

Do not trigger the download first and then wait.

---

## 50. File Automation with Page Objects

Example:

```ts
export class ReportPage {
  constructor(
    private readonly page: Page
  ) {}

  async downloadReport() {
    const downloadPromise =
      this.page.waitForEvent(
        'download'
      );

    await this.page.getByRole(
      'button',
      { name: 'Download Report' }
    ).click();

    return await downloadPromise;
  }
}
```

The Page Object handles UI behavior while the test handles validation.

---

## 51. Download Utility

Example:

```ts
export async function
saveDownload(
  download: Download,
  path: string
) {
  await download.saveAs(path);
}
```

This keeps file operations reusable.

---

## 52. File Naming Convention

Use predictable names:

```text
expense-report.pdf
employee-data.csv
users.xlsx
invoice-1001.pdf
```

For parallel tests:

```text
expense-report-${testId}.pdf
```

Avoid ambiguous names such as:

```text
file.pdf
test.pdf
download.pdf
```

when many tests run concurrently.

---

## 53. File Data Management

Recommended project structure:

```text
playwright/
├── test-data/
│   ├── uploads/
│   │   ├── document.pdf
│   │   ├── employees.csv
│   │   └── image.png
│   │
│   └── expected/
│       ├── report.json
│       └── report.csv
│
├── utils/
│   └── fileUtils.ts
│
├── pages/
│   └── ReportPage.ts
│
└── tests/
    └── file-upload-download.spec.ts
```

---

## 54. File Test Data Best Practices

Keep test data:

- Small
- Deterministic
- Version controlled
- Non-sensitive
- Reusable

Never commit:

```text
Real customer documents
Real identity documents
Real credentials
Production exports
Private financial files
```

Use synthetic test data.

---

## 55. Download Security

Do not automatically execute downloaded files.

Especially avoid executing:

```text
.exe
.bat
.ps1
.sh
```

Download tests should normally validate files as data.

---

## 56. File Integrity

For critical exports, consider validating:

```text
File exists
File size
File extension
MIME/type
Expected headers
Expected records
Expected values
```

For binary files, checksum/hash validation can be used when the expected file is deterministic.

---

## 57. Hash Validation

Example using Node.js:

```ts
import crypto from 'node:crypto';
import fs from 'node:fs';

const hash =
  crypto
    .createHash('sha256')
    .update(
      fs.readFileSync(
        'downloads/report.pdf'
      )
    )
    .digest('hex');

console.log(hash);
```

Use hash validation only when the generated content is expected to be deterministic.

---

## 58. File Automation and CI

CI environments may have different:

```text
Paths
Operating systems
Permissions
Working directories
```

Prefer:

```ts
import path from 'node:path';

const outputPath =
  path.join(
    process.cwd(),
    'test-results',
    'report.pdf'
  );
```

Avoid hard-coded Windows paths such as:

```text
C:\Users\Sandeep\Desktop\
```

---

## 59. Cross-Platform Paths

Use:

```ts
path.join(
  'test-results',
  'downloads',
  'report.pdf'
);
```

instead of:

```text
test-results/downloads/report.pdf
```

when building paths programmatically.

This improves portability.

---

## 60. Parallel Test Safety

For parallel execution:

```text
Worker 1
 └── downloads/test-1/

Worker 2
 └── downloads/test-2/
```

Avoid sharing the same generated filename between workers.

Use unique directories or test-specific names.

---

## 61. File Test Isolation

Each test should ideally own:

```text
Upload data
Download directory
Temporary files
Cleanup
```

This prevents one test from affecting another.

---

## 62. Download Timeout

If a download can take longer than normal, configure an appropriate timeout rather than using arbitrary sleeps.

Example:

```ts
const download =
  await page.waitForEvent(
    'download',
    {
      timeout: 60_000
    }
  );
```

Use realistic limits based on application behavior and CI conditions.

---

## 63. Upload Timeout

For large uploads, the action may need an appropriate timeout.

Avoid simply increasing every timeout globally.

First determine whether the problem is:

```text
Network
Application
File size
Server processing
Incorrect locator
```

Then adjust the relevant configuration.

---

## 64. Download Failure Diagnostics

When a download fails, capture:

```text
URL
Suggested filename
Download failure
Screenshot
Trace
Console errors
Relevant API response
```

This makes CI failures easier to diagnose.

---

## 65. File Chooser Diagnostics

When `filechooser` does not fire:

```text
1. Verify the click actually triggers a file chooser.
2. Verify the listener is registered first.
3. Check whether the element is disabled.
4. Inspect the application implementation.
5. Check whether the input is already accessible.
```

If a normal file input exists, prefer `setInputFiles()`.

---

## 66. Common File Automation Mistakes

### Mistake 1

Waiting for download after clicking.

### Mistake 2

Using hard-coded local paths.

### Mistake 3

Committing generated downloads.

### Mistake 4

Using shared filenames in parallel tests.

### Mistake 5

Validating only that a file exists.

### Mistake 6

Ignoring file content.

### Mistake 7

Leaving temporary files after tests.

### Mistake 8

Uploading production or sensitive data.

### Mistake 9

Using arbitrary waits for upload completion.

### Mistake 10

Executing downloaded files during tests.

---

## 67. Production-Quality Upload Example

```ts
import {
  test,
  expect
} from '@playwright/test';

test('upload employee document', async ({
  page
}) => {
  await page.goto('/employees');

  await page
    .getByLabel('Document')
    .setInputFiles(
      'test-data/uploads/document.pdf'
    );

  await expect(
    page.getByText(
      'document.pdf'
    )
  ).toBeVisible();

  await page.getByRole(
    'button',
    { name: 'Upload' }
  ).click();

  await expect(
    page.getByRole('status')
  ).toContainText(
    'Upload successful'
  );
});
```

---

## 68. Production-Quality File Chooser Example

```ts
test('upload using file chooser', async ({
  page
}) => {
  await page.goto('/documents');

  const chooserPromise =
    page.waitForEvent(
      'filechooser'
    );

  await page.getByRole(
    'button',
    { name: 'Choose File' }
  ).click();

  const chooser =
    await chooserPromise;

  await chooser.setFiles(
    'test-data/uploads/document.pdf'
  );

  await expect(
    page.getByText(
      'document.pdf'
    )
  ).toBeVisible();
});
```

---

## 69. Production-Quality Download Example

```ts
import fs from 'node:fs';

test('download report', async ({
  page
}) => {
  await page.goto('/reports');

  const downloadPromise =
    page.waitForEvent(
      'download'
    );

  await page.getByRole(
    'button',
    { name: 'Download Report' }
  ).click();

  const download =
    await downloadPromise;

  expect(
    await download.failure()
  ).toBeNull();

  expect(
    download.suggestedFilename()
  ).toMatch(
    /\.pdf$/
  );

  const outputPath =
    'test-results/report.pdf';

  try {
    await download.saveAs(
      outputPath
    );

    expect(
      fs.existsSync(
        outputPath
      )
    ).toBe(true);

    expect(
      fs.statSync(
        outputPath
      ).size
    ).toBeGreaterThan(0);

  } finally {
    fs.rmSync(
      outputPath,
      { force: true }
    );
  }
});
```

---

## 70. Interview Questions

### Q1. How do you upload a file in Playwright?

```ts
await locator.setInputFiles(
  'file.pdf'
);
```

### Q2. How do you handle a native file chooser?

```ts
const chooserPromise =
  page.waitForEvent(
    'filechooser'
  );

await button.click();

const chooser =
  await chooserPromise;

await chooser.setFiles(
  'file.pdf'
);
```

### Q3. How do you handle downloads?

```ts
const downloadPromise =
  page.waitForEvent(
    'download'
  );

await button.click();

const download =
  await downloadPromise;
```

### Q4. How do you get the downloaded filename?

```ts
download.suggestedFilename();
```

### Q5. How do you save a download?

```ts
await download.saveAs(
  'downloads/report.pdf'
);
```

### Q6. How do you check whether a download failed?

```ts
await download.failure();
```

### Q7. How do you upload generated content?

Use a buffer:

```ts
await locator.setInputFiles({
  name: 'data.txt',
  mimeType: 'text/plain',
  buffer: Buffer.from('hello')
});
```

### Q8. Why should download listeners be registered before clicking?

To avoid missing the download event due to event-ordering races.

### Q9. How should file tests work in parallel?

Use isolated temporary directories or unique filenames.

### Q10. What should a robust download test validate?

At minimum:

```text
Download triggered
Download succeeded
Filename
Extension
File exists
File size
Content when required
```

---

## 71. Hands-On Exercises

### Exercise 1 — Single File Upload

Upload a PDF and verify the file appears in the UI.

### Exercise 2 — Multiple Files

Upload three files and verify all three.

### Exercise 3 — File Chooser

Automate a button that opens the native file chooser.

### Exercise 4 — Generated File

Create a CSV buffer dynamically and upload it.

### Exercise 5 — Download PDF

Download a report and verify its filename and extension.

### Exercise 6 — Download CSV

Download a CSV and validate its header row.

### Exercise 7 — Download Cleanup

Save a downloaded file and remove it in `finally`.

### Exercise 8 — Invalid File

Upload an unsupported file and verify the error message.

### Exercise 9 — Large File

Test the application's maximum file-size boundary.

### Exercise 10 — Enterprise File Utility

Create reusable utilities:

```text
uploadFile()
downloadFile()
fileExists()
fileSize()
readTextFile()
deleteFile()
validateExtension()
```

---

## 72. Key Takeaways

- Playwright provides dedicated APIs for file uploads and downloads.
- Use `setInputFiles()` for accessible file inputs.
- Use `filechooser` when a user action opens a native file chooser.
- Register file chooser listeners before triggering the action.
- Use `waitForEvent('download')` for downloads.
- Validate download failure status.
- Validate suggested filenames and extensions.
- Save downloads to controlled test directories.
- Validate file content when business requirements demand it.
- Use buffers for generated test files.
- Keep test files deterministic and non-sensitive.
- Isolate file paths during parallel execution.
- Clean up temporary files.
- Avoid hard-coded machine-specific paths.
- Do not execute downloaded files.
- Encapsulate repeated file operations in utilities or Page Objects.

---

## 73. Enterprise File Automation Challenge

Build a complete document-management automation framework:

```text
Employee Portal
      ↓
Upload Document
      ↓
Server Processing
      ↓
Document List
      ↓
Download Document
      ↓
File Validation
```

Requirements:

1. Upload PDF and CSV files.
2. Support file chooser workflows.
3. Validate upload status.
4. Test invalid file types.
5. Test file-size boundaries.
6. Download generated reports.
7. Validate filename and extension.
8. Validate file content.
9. Use isolated download directories.
10. Clean up generated files.
11. Support parallel test execution.
12. Keep all file operations reusable through utilities.
13. Use Page Objects for UI interactions.
14. Never use production or sensitive documents.
15. Make the solution CI-friendly and cross-platform.

---

## 74. Final Architecture

```text
Test
 │
 ├── Page Object
 │      ↓
 │   Upload / Download Action
 │
 ├── File Utility
 │      ↓
 │   File System
 │
 └── Validation
        ↓
   Business Result
```

For a larger enterprise framework:

```text
tests/
pages/
fixtures/
utils/
test-data/
test-results/
```

This structure keeps browser interactions, file-system operations, test data, and business assertions separated and maintainable.

---

## 75. Next Step

**Step 91 — Mastering Playwright Network Interception, Request Routing, API Mocking, Response Modification, Network Conditions & Enterprise Service Virtualization**
