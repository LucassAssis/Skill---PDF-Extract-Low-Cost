# PDF Exctract Low Cost

## Overview
Pipeline for extracting and structuring financial data from PDFs with focus on low cost, accuracy and traceability.

## Key Idea
Use the cheapest method first:
text extraction → fallback → OCR only when necessary.

## Workflow
1. File filtering (by name/date/type)
2. Check text layer
3. Extract:
   - pdfplumber (default)
   - pypdf (fallback)
   - OCR (last option)
4. Sample pages to understand layout
5. Parse with regex (structured rules)
6. Build structured dataset (Excel)
7. Validate (counts, totals, missing data)

## Stack
- pdfplumber
- pypdf
- pymupdf (fitz)
- openpyxl
- Tesseract OCR

## Output
- Structured Excel file
- One row per record
- Source tracking (file + page)
- Ready for analysis

## Key Insight
OCR should be the last option, not the default.

## Author
Lucas Assis
