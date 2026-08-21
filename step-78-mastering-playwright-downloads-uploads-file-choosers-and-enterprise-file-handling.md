# Playwright with TypeScript Mastery Bootcamp

# Step 78 — Mastering Playwright Downloads, Uploads, File Choosers & Enterprise File Handling

## 1. Introduction

File upload and download workflows are common in real-world applications.

Examples include:

- Uploading profile images
- Importing CSV files
- Uploading Excel files
- Downloading reports
- Exporting PDF files
- Bulk data import
- Document management
- Resume uploads
- Evidence and attachment workflows

Playwright provides reliable APIs for handling these scenarios without depending on manual operating-system interactions.

The most important APIs are:

```ts
setInputFiles()
waitForEvent('filechooser')
waitForEvent('download')
download.suggestedFilename()
download.path()
download.saveAs()
download.createReadStream()
```

---

## 2. File Upload Strategies

Playwright commonly handles uploads in three ways:

1. Directly using an `<input type="file">`
2. Using the `FileChooser` API
3. Uploading in-memory files

The preferred approach depends on how the application implements its upload control.

---

## 3. Basic File Upload with `setInputFiles()`

Suppose the page contains:

```html
<input type="file" />
```

The simplest Playwright implementation is:

```ts
await page.locator('input[type="file"]')
  .setInputFiles('test-data/sample.pdf');
```

Playwright sets the file directly on the input.

No operating-system file picker interaction is required.

---

## 4. Uploading by Test ID

If the application provides a stable test ID:

```html
<input type="file" data-testid="file-upload" />
```

Use:

```ts
await page
  .getByTestId('file-upload')
  .setInputFiles('test-data/sample.pdf');
```

This is usually preferable to fragile CSS selectors.

---

## 5. Uploading Multiple Files

For an input that supports multiple files:

```html
<input type="file" multiple />
```

Use:

```ts
await page.locator('input[type="file"]')
  .setInputFiles([
    'test-data/file1.pdf',
    'test-data/file2.pdf',
    'test-data/file3.pdf'
  ]);
```

---

## 6. Removing Selected Files

You can clear the selected files:

```ts
await page
  .locator('input[type="file"]')
  .setInputFiles([]);
```

This is useful when testing:

- Replace file
- Remove attachment
- Reset upload
- Validation after clearing a file

---

## 7. Uploading a File with the FileChooser API

Some applications trigger the file chooser through a button.

Example:

```ts
const fileChooserPromise = page.waitForEvent('filechooser');

await page.getByRole('button', { name: 'Upload File' }).click();

const fileChooser = await fileChooserPromise;

await fileChooser.setFiles('test-data/sample.pdf');
```

The important point is that `waitForEvent('filechooser')` must be registered before clicking the upload button.

---

## 8. Why `filechooser` Is Needed

Consider:

```html
<button>Choose File</button>
```

The button may internally trigger:

```ts
input.click();
```

There may not be a convenient visible `<input>` to interact with directly.

In such cases:

```ts
page.waitForEvent('filechooser')
```

allows Playwright to capture the file selection operation.

---

## 9. FileChooser with `Promise.all()`

A clean pattern is:

```ts
const [fileChooser] = await Promise.all([
  page.waitForEvent('filechooser'),
  page.getByRole('button', { name: 'Upload File' }).click()
]);

await fileChooser.setFiles('test-data/sample.pdf');
```

This prevents a race condition.

---

## 10. Uploading Multiple Files Through FileChooser

```ts
const [fileChooser] = await Promise.all([
  page.waitForEvent('filechooser'),
  page.getByRole('button', { name: 'Upload Files' }).click()
]);

await fileChooser.setFiles([
  'test-data/file1.pdf',
  'test-data/file2.pdf'
]);
```

---

## 11. Uploading an In-Memory File

Playwright can upload files without creating them first on disk.

Example:

```ts
await page.locator('input[type="file"]').setInputFiles({
  name: 'test.txt',
  mimeType: 'text/plain',
  buffer: Buffer.from('Hello Playwright')
});
```

This is useful for dynamically generated test data.

---

## 12. Creating a CSV File in Memory

```ts
const csvContent = `
id,name
1,Sandeep
2,John
3,Alice
`;

await page
  .locator('input[type="file"]')
  .setInputFiles({
    name: 'users.csv',
    mimeType: 'text/csv',
    buffer: Buffer.from(csvContent)
  });
```

This is useful for API-driven and data-driven testing.

---

## 13. Creating JSON Test Data in Memory

```ts
const data = {
  name: 'Sandeep',
  role: 'QA Engineer'
};

await page
  .locator('input[type="file"]')
  .setInputFiles({
    name: 'user.json',
    mimeType: 'application/json',
    buffer: Buffer.from(JSON.stringify(data))
  });
```

---

## 14. File Upload Validation

After uploading, validate the UI.

```ts
await page
  .locator('input[type="file"]')
  .setInputFiles('test-data/sample.pdf');

await expect(
  page.getByText('sample.pdf')
).toBeVisible();
```

You can also verify success messages:

```ts
await expect(
  page.getByText('File uploaded successfully')
).toBeVisible();
```

---

## 15. Testing Invalid File Types

Enterprise applications commonly restrict file extensions.

Example:

```ts
await page
  .locator('input[type="file"]')
  .setInputFiles('test-data/malicious.exe');
```

Then verify validation:

```ts
await expect(
  page.getByText('Invalid file type')
).toBeVisible();
```

Always test both valid and invalid files.

---

## 16. Testing File Size Validation

If the application limits file size:

```ts
const largeFile = Buffer.alloc(10 * 1024 * 1024);

await page
  .locator('input[type="file"]')
  .setInputFiles({
    name: 'large-file.pdf',
    mimeType: 'application/pdf',
    buffer: largeFile
  });
```

Then verify the expected validation message.

---

## 17. Download Handling

Playwright can wait for a download using:

```ts
page.waitForEvent('download')
```

Example:

```ts
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download Report' }).click();

const download = await downloadPromise;
```

---

## 18. Download with `Promise.all()`

A common pattern is:

```ts
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.getByRole('button', { name: 'Download Report' }).click()
]);
```

This is preferred because the download event listener is established before the action.

---

## 19. Getting the Suggested Filename

Use:

```ts
const filename = download.suggestedFilename();

console.log(filename);
```

Example:

```ts
expect(download.suggestedFilename()).toBe('report.pdf');
```

This is useful for validating application-generated filenames.

---

## 20. Saving the Download

Use:

```ts
await download.saveAs('downloads/report.pdf');
```

Example:

```ts
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.getByRole('button', { name: 'Download Report' }).click()
]);

await download.saveAs('downloads/report.pdf');
```

---

## 21. Downloading to a Test-Specific Directory

A useful framework pattern is to create a unique directory for each test.

Example:

```ts
const downloadPath = `downloads/${Date.now()}-report.pdf`;

await download.saveAs(downloadPath);
```

This reduces collisions between parallel tests.

---

## 22. Getting the Download Path

Playwright provides:

```ts
const path = await download.path();

console.log(path);
```

The path can be useful for temporary file processing.

However, enterprise tests should generally use `saveAs()` when they need a predictable persistent test artifact.

---

## 23. Reading Downloaded Files

After saving a file, Node.js can be used to inspect it.

```ts
import fs from 'fs/promises';

await download.saveAs('downloads/report.txt');

const content = await fs.readFile(
  'downloads/report.txt',
  'utf-8'
);

expect(content).toContain('Sandeep');
```

This allows UI-to-file validation.

---

## 24. Validating PDF Downloads

For a PDF download:

```ts
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.getByRole('button', { name: 'Export PDF' }).click()
]);

expect(download.suggestedFilename())
  .toMatch(/\.pdf$/);

await download.saveAs('downloads/report.pdf');
```

For actual PDF-content validation, use an appropriate PDF parsing library outside Playwright's core API.

---

## 25. Validating CSV Downloads

```ts
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.getByRole('button', { name: 'Export CSV' }).click()
]);

await download.saveAs('downloads/users.csv');
```

Then:

```ts
import fs from 'fs/promises';

const csv = await fs.readFile(
  'downloads/users.csv',
  'utf-8'
);

expect(csv).toContain('name');
expect(csv).toContain('Sandeep');
```

---

## 26. Download Failure Handling

Playwright provides:

```ts
download.failure()
```

Example:

```ts
const failure = await download.failure();

expect(failure).toBeNull();
```

This is useful for explicitly checking whether the download completed successfully.

---

## 27. Checking Download Filename

```ts
expect(download.suggestedFilename())
  .toBe('employee-report.xlsx');
```

Or use a pattern:

```ts
expect(download.suggestedFilename())
  .toMatch(/employee-report.*\.xlsx/);
```

---

## 28. Page Object for File Upload

```ts
import { Page } from '@playwright/test';

export class FileUploadPage {
  constructor(private readonly page: Page) {}

  private fileInput = this.page.locator(
    'input[type="file"]'
  );

  async uploadFile(filePath: string) {
    await this.fileInput.setInputFiles(filePath);
  }

  async clearFile() {
    await this.fileInput.setInputFiles([]);
  }
}
```

This keeps file-upload implementation details out of test files.

---

## 29. Page Object for Downloads

```ts
import { Page, Download } from '@playwright/test';

export class ReportsPage {
  constructor(private readonly page: Page) {}

  async downloadReport(): Promise<Download> {
    const [download] = await Promise.all([
      this.page.waitForEvent('download'),
      this.page.getByRole('button', {
        name: 'Download Report'
      }).click()
    ]);

    return download;
  }
}
```

Test:

```ts
const download = await reportsPage.downloadReport();

expect(download.suggestedFilename())
  .toMatch(/report/);
```

---

## 30. File Upload Utility

A reusable utility can centralize common upload behavior:

```ts
import { Page } from '@playwright/test';

export async function uploadFile(
  page: Page,
  selector: string,
  filePath: string
) {
  await page
    .locator(selector)
    .setInputFiles(filePath);
}
```

Usage:

```ts
await uploadFile(
  page,
  'input[type="file"]',
  'test-data/sample.pdf'
);
```

---

## 31. Download Utility

```ts
import { Page, Download } from '@playwright/test';

export async function downloadFile(
  page: Page,
  action: () => Promise<void>
): Promise<Download> {
  const [download] = await Promise.all([
    page.waitForEvent('download'),
    action()
  ]);

  return download;
}
```

Usage:

```ts
const download = await downloadFile(
  page,
  () => page.getByRole('button', {
    name: 'Download'
  }).click()
);
```

---

## 32. Organizing Test Files

A scalable project can use:

```text
playwright/
├── pages/
├── tests/
├── fixtures/
├── utils/
├── test-data/
│   ├── uploads/
│   │   ├── sample.pdf
│   │   ├── users.csv
│   │   └── image.png
│   └── downloads/
├── reports/
└── playwright.config.ts
```

Keep static upload files under `test-data`.

Generated downloads should normally be stored outside source-controlled test data.

---

## 33. Parallel Execution Considerations

When tests run in parallel, avoid sharing the same output filename.

Risky:

```ts
await download.saveAs('downloads/report.pdf');
```

Multiple workers may write to the same location.

Prefer unique paths:

```ts
const fileName =
  `downloads/report-${test.info().workerIndex}-${Date.now()}.pdf`;

await download.saveAs(fileName);
```

This helps prevent collisions.

---

## 34. Test Isolation

Each test should ideally have:

- Its own upload data
- Its own download directory
- Unique generated filenames
- Independent application state

This becomes especially important with:

```ts
fullyParallel: true
```

---

## 35. Upload and Download End-to-End Scenario

A realistic workflow:

```ts
test('upload and download document', async ({ page }) => {
  await page.goto('/documents');

  await page
    .locator('input[type="file"]')
    .setInputFiles('test-data/sample.pdf');

  await expect(
    page.getByText('sample.pdf')
  ).toBeVisible();

  const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.getByRole('button', {
      name: 'Download'
    }).click()
  ]);

  expect(download.suggestedFilename())
    .toMatch(/\.pdf$/);

  await download.saveAs(
    'downloads/downloaded-sample.pdf'
  );
});
```

---

## 36. Common Mistakes

### Mistake 1 — Clicking the OS file picker manually

Do not automate operating-system file dialogs with coordinate-based approaches.

Prefer:

```ts
setInputFiles()
```

or:

```ts
filechooser.setFiles()
```

### Mistake 2 — Registering download handling after clicking

Incorrect:

```ts
await page.getByText('Download').click();

const download =
  await page.waitForEvent('download');
```

Correct:

```ts
const downloadPromise =
  page.waitForEvent('download');

await page.getByText('Download').click();

const download = await downloadPromise;
```

### Mistake 3 — Using hard waits

Avoid:

```ts
await page.waitForTimeout(5000);
```

Wait for the actual file chooser or download event.

---

## 37. Upload Validation Matrix

A professional test suite can cover:

| Scenario | Expected Result |
|---|---|
| Valid PDF | Upload succeeds |
| Valid CSV | Upload succeeds |
| Invalid extension | Validation error |
| Empty file | Validation error or accepted based on requirements |
| Large file | Size validation |
| Multiple files | All valid files uploaded |
| Duplicate file | Business-rule validation |
| Missing file | Required-field validation |
| Corrupted file | Application rejects file |

---

## 38. Download Validation Matrix

Test:

| Scenario | Validation |
|---|---|
| PDF export | Correct extension |
| CSV export | Correct content |
| Excel export | Correct filename |
| Empty report | Business-rule behavior |
| Large report | Download completes |
| Unauthorized user | Download blocked |
| Failed export | Correct error |
| Parallel downloads | No filename collision |

---

## 39. Enterprise Best Practices

### Use stable selectors

```ts
getByTestId('upload-file')
```

### Keep test data organized

```text
test-data/uploads/
```

### Use unique download paths

Especially with parallel execution.

### Validate business results

Do not stop after checking that a file exists.

### Separate test data from generated artifacts

Static files belong in test data.

Generated downloads belong in test output directories.

### Avoid OS-level automation

Use Playwright's file APIs.

### Build reusable utilities

Centralize repeated upload and download logic.

---

## 40. Interview Questions

### Q1. How do you upload a file in Playwright?

```ts
await page
  .locator('input[type="file"]')
  .setInputFiles('test-data/file.pdf');
```

### Q2. How do you handle a file chooser?

```ts
const [fileChooser] = await Promise.all([
  page.waitForEvent('filechooser'),
  page.getByRole('button', { name: 'Upload' }).click()
]);

await fileChooser.setFiles('test-data/file.pdf');
```

### Q3. How do you handle downloads?

```ts
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.getByRole('button', { name: 'Download' }).click()
]);
```

### Q4. How do you get the downloaded filename?

```ts
download.suggestedFilename()
```

### Q5. How do you save a download?

```ts
await download.saveAs('downloads/report.pdf');
```

### Q6. Can Playwright upload a file without a physical file?

Yes. Use an object containing `name`, `mimeType`, and `buffer`.

### Q7. Why should download handling be registered before clicking Download?

Because the download event can occur immediately after the action. Registering the listener first prevents a race condition.

### Q8. How do you support parallel download tests?

Use unique download paths and filenames for each test or worker.

---

## 41. Hands-On Exercises

### Exercise 1 — Single File Upload

Create a test that uploads a PDF and verifies the uploaded filename.

### Exercise 2 — Multiple File Upload

Upload three files and verify all three are displayed.

### Exercise 3 — Invalid File

Upload an unsupported file type and verify the validation message.

### Exercise 4 — File Chooser

Implement an upload button using:

```ts
page.waitForEvent('filechooser')
```

### Exercise 5 — Download

Download a report and verify its filename.

### Exercise 6 — Download Content

Download a CSV file and verify that expected headers and data exist.

### Exercise 7 — Parallel Safety

Create a unique download directory for each test.

### Exercise 8 — Page Object

Build reusable upload and download methods inside Page Objects.

---

## 42. Key Takeaways

- Use `setInputFiles()` for standard file inputs.
- Use `filechooser` when an application opens the file chooser through an action.
- Use `waitForEvent('download')` for downloads.
- Register event waits before triggering actions.
- Use `download.suggestedFilename()` to validate filenames.
- Use `download.saveAs()` to save files to predictable locations.
- In-memory files can be uploaded with `name`, `mimeType`, and `buffer`.
- Avoid OS-level file-picker automation.
- Avoid arbitrary hard waits.
- Use unique paths for parallel downloads.
- Keep static upload files under test data.
- Keep generated downloads in test output directories.
- Validate file contents when business correctness matters.
- Encapsulate repeated file operations in utilities, Page Objects, or fixtures.

---

## 43. Next Step

**Step 79 — Mastering Playwright Network Interception, Routing, Mocking, Request Modification & API Simulation**
