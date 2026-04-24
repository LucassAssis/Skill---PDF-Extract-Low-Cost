---
name: pdf-extract-low-cost
description: Extract requested data from PDFs or image documents into structured outputs such as Excel/CSV while minimizing context and tool cost. Use when Codex is asked to read folders of PDFs, bank statements, invoices, NFs/NFe/NFS-e, scanned documents, OCR images, or to find specific fields/transactions/payments and organize them into spreadsheets with validation.
---

# PDF Extract Low Cost

## Goal

Use the cheapest reliable path first: inspect filenames and text layers, extract only the needed pages/fields, OCR only when text extraction fails or is insufficient, then validate the output with deterministic checks.

Do not store document-specific layouts in this skill. Learn the current layout from a few samples each time.

## Low-Cost Workflow

1. List candidate files first.
   - Use `rg --files`, `Get-ChildItem`, or equivalent.
   - Filter by requested month, year, name, extension, or folder before opening files.
   - Exclude noisy/generated paths unless the user points to them.

2. Check whether PDFs have a text layer.
   - Try `pdfplumber` first for tables/line-preserving text.
   - If weak or empty, try `pypdf`.
   - If still weak or empty, use OCR.
   - Use `PyMuPDF`/`fitz` for page counts, rendering, dimensions, clips, and image generation.

3. Sample before bulk extraction.
   - Inspect 1-3 pages from representative files.
   - Print only short snippets or search hits, not full documents.
   - Identify repeated line patterns, continuation lines, field labels, date/value formats, and page-to-record assumptions.

4. Prefer deterministic parsers.
   - For text PDFs, parse lines with regex and structured rules.
   - For scanned PDFs, render pages/clips with `fitz` and OCR with Tesseract.
   - For fixed visual fields, OCR clipped regions instead of whole pages when cheaper and more accurate.
   - For images, run Tesseract directly on the image.

5. Generate an audit-friendly spreadsheet.
   - Use `openpyxl` for Excel.
   - Include one row per logical record.
   - Preserve provenance columns when useful: source file, page, original line/text, extraction method, warnings.
   - Add separate tabs for requested groups/months/categories.
   - Add a summary tab when it helps understanding.

6. Validate before final.
   - Count files, pages, records, tabs, and blank critical fields.
   - Check totals/counts for filtered tabs.
   - Search for footer/noise text accidentally joined to records.
   - Report output path, counts, and any residual risk.

## Tool Order

For PDFs:

1. `pdfplumber` for text and table-like lines.
2. `pypdf` if `pdfplumber` is empty or malformed.
3. OCR only when text is insufficient.
4. For OCR, render with `fitz`/PyMuPDF to temporary PNGs, then run:

```powershell
tesseract <image> stdout -l por --psm 6
```

If the Portuguese language data is not installed globally, use:

```powershell
tesseract <image> stdout --tessdata-dir <path-with-por.traineddata> -l por --psm 6
```

For images:

```powershell
tesseract <image-file> stdout -l por --psm 6
```

Use OCR on clipped regions when fields are visually fixed. Use whole-page OCR only when the layout is unknown or continuation text matters.

## Finding Requested Information

When the user asks for specific fields:

- Start by searching labels and nearby examples in extracted text.
- If labels are unreliable, infer by stable patterns: dates, document numbers, CNPJ/CPF, money values, credit/debit markers, section headings.
- For fields split across lines, attach continuation lines until the next transaction/date/header.
- Keep both parsed columns and original/provenance text if ambiguity exists.
- For bank statements, parse each dated line as a transaction and attach the following non-date line as `Contraparte/Complemento` when it names payer/payee/CNPJ.
- For invoice-like documents, determine whether each page is one record before assuming it; validate by page count vs numbering or repeated headers.

When the user asks for a filtered subset:

- Build the full structured dataset first if cheap.
- Filter deterministically by exact identifiers first, then normalized names.
- For company/person filters, search both the transaction line and continuation/complement lines.
- Put filtered results in a separate tab and keep source columns.

## Cost Controls

- Do not load all document text into chat.
- Do not OCR a whole folder before checking text layers.
- Do not use model reasoning to parse large repetitive text when regex/local code is enough.
- Cache or write a local extraction script for multi-file jobs.
- Print progress as counts per file, not full content.
- Increase sampling only when validation fails or formats vary.

## Suggested Python Packages

Install or reuse:

```powershell
py -3 -m pip install --upgrade pdfplumber pypdf pymupdf openpyxl
```

Use Tesseract outside Python for OCR. Confirm Portuguese support with:

```powershell
tesseract --list-langs
```

Expect `por` in the language list, or provide `--tessdata-dir`.
