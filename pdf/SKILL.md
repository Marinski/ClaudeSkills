---
name: pdf
description: "Comprehensive PDF manipulation, extraction, and generation with support for text extraction, form filling, merging, splitting, annotations, and creation. Use when working with .pdf files for: (1) Extracting text and tables, (2) Filling PDF forms, (3) Merging/splitting PDFs, (4) Creating PDFs programmatically, (5) Adding watermarks/annotations, (6) PDF metadata management"
---

# PDF Manipulation Skill

A comprehensive guide for working with PDF files in Python, covering extraction, manipulation, creation, and advanced operations.

## Table of Contents

1. [Core Capabilities](#core-capabilities)
2. [Python Libraries](#python-libraries)
3. [Installation](#installation)
4. [Text Extraction](#text-extraction)
5. [Table Extraction](#table-extraction)
6. [PDF Form Operations](#pdf-form-operations)
7. [Merging PDFs](#merging-pdfs)
8. [Splitting PDFs](#splitting-pdfs)
9. [Creating PDFs](#creating-pdfs)
10. [Watermarks and Annotations](#watermarks-and-annotations)
11. [Metadata Management](#metadata-management)
12. [Security and Encryption](#security-and-encryption)
13. [OCR for Scanned Documents](#ocr-for-scanned-documents)
14. [Best Practices](#best-practices)
15. [Common Pitfalls](#common-pitfalls)

---

## Core Capabilities

This skill enables you to:

- **Extract text** with layout preservation
- **Extract tables** and parse structured data
- **Fill PDF forms** programmatically
- **Merge multiple PDFs** into a single document
- **Split PDFs** by pages or ranges
- **Create PDFs from scratch** with text, images, and graphics
- **Add watermarks** and annotations
- **Extract and modify metadata** (author, title, keywords, etc.)
- **Add password protection** and encryption
- **Perform OCR** on scanned documents
- **Convert images to PDF**
- **Compress and optimize** PDF files
- **Extract images** from PDFs
- **Rotate and reorder pages**

---

## Python Libraries

### 1. pypdf (PyPDF2)
**Purpose**: Basic PDF operations (merging, splitting, rotation)

```bash
pip install pypdf
```

```python
from pypdf import PdfReader, PdfWriter, PdfMerger
```

**Use for**: Merging, splitting, rotating pages, extracting metadata

---

### 2. pdfplumber
**Purpose**: Advanced text and table extraction with layout awareness

```bash
pip install pdfplumber
```

```python
import pdfplumber
```

**Use for**: Extracting text with positioning, table detection, precise layout analysis

---

### 3. reportlab
**Purpose**: Creating PDFs from scratch

```bash
pip install reportlab
```

```python
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter, A4
from reportlab.lib.units import inch
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph
from reportlab.lib.styles import getSampleStyleSheet
```

**Use for**: Generating reports, invoices, certificates, custom PDFs

---

### 4. PyMuPDF (fitz)
**Purpose**: Advanced PDF manipulation, rendering, and conversion

```bash
pip install PyMuPDF
```

```python
import fitz  # PyMuPDF
```

**Use for**: Advanced operations, image extraction, annotations, rendering, compression

---

### 5. pdf2image
**Purpose**: Converting PDF pages to images

```bash
pip install pdf2image
# Also requires poppler:
# macOS: brew install poppler
# Ubuntu: sudo apt-get install poppler-utils
# Windows: Download from https://github.com/oschwartz10612/poppler-windows/releases/
```

```python
from pdf2image import convert_from_path
```

---

### 6. pytesseract (for OCR)
**Purpose**: OCR for scanned documents

```bash
pip install pytesseract
# Also requires tesseract:
# macOS: brew install tesseract
# Ubuntu: sudo apt-get install tesseract-ocr
# Windows: Download from https://github.com/UB-Mannheim/tesseract/wiki
```

```python
import pytesseract
from PIL import Image
```

---

## Installation

**Quick Start - Install All Libraries:**

```bash
pip install pypdf pdfplumber reportlab PyMuPDF pdf2image pytesseract pillow
```

**System Dependencies:**
```bash
# macOS
brew install poppler tesseract

# Ubuntu/Debian
sudo apt-get install poppler-utils tesseract-ocr

# Windows (use package managers or download installers)
```

---

## Text Extraction

### Basic Text Extraction (pypdf)

```python
from pypdf import PdfReader

def extract_text_basic(pdf_path):
    """Extract all text from a PDF using pypdf."""
    reader = PdfReader(pdf_path)
    text = ""

    for page_num, page in enumerate(reader.pages, start=1):
        page_text = page.extract_text()
        text += f"--- Page {page_num} ---\n{page_text}\n\n"

    return text

# Usage
text = extract_text_basic("document.pdf")
print(text)
```

---

### Advanced Text Extraction with Layout (pdfplumber)

```python
import pdfplumber

def extract_text_with_layout(pdf_path):
    """Extract text preserving layout information."""
    results = []

    with pdfplumber.open(pdf_path) as pdf:
        for page_num, page in enumerate(pdf.pages, start=1):
            # Extract text
            text = page.extract_text()

            # Extract words with positioning
            words = page.extract_words()

            results.append({
                'page': page_num,
                'text': text,
                'words': words,
                'width': page.width,
                'height': page.height
            })

    return results

# Usage
pages = extract_text_with_layout("document.pdf")
for page_data in pages:
    print(f"Page {page_data['page']}:")
    print(page_data['text'])
    print(f"Total words: {len(page_data['words'])}")
```

---

### Extract Text from Specific Regions

```python
import pdfplumber

def extract_text_from_region(pdf_path, page_num, bbox):
    """
    Extract text from a specific region.

    Args:
        pdf_path: Path to PDF
        page_num: Page number (0-indexed)
        bbox: Tuple (x0, y0, x1, y1) defining the region
    """
    with pdfplumber.open(pdf_path) as pdf:
        page = pdf.pages[page_num]

        # Crop to specific region
        cropped = page.crop(bbox)
        text = cropped.extract_text()

        return text

# Usage - Extract header region
header_text = extract_text_from_region(
    "document.pdf",
    page_num=0,
    bbox=(0, 0, 612, 100)  # Top 100 points
)
```

---

## Table Extraction

### Basic Table Extraction (pdfplumber)

```python
import pdfplumber
import pandas as pd

def extract_tables(pdf_path):
    """Extract all tables from a PDF."""
    all_tables = []

    with pdfplumber.open(pdf_path) as pdf:
        for page_num, page in enumerate(pdf.pages, start=1):
            tables = page.extract_tables()

            for table_num, table in enumerate(tables, start=1):
                # Convert to DataFrame
                if table:
                    df = pd.DataFrame(table[1:], columns=table[0])
                    all_tables.append({
                        'page': page_num,
                        'table': table_num,
                        'data': df
                    })

    return all_tables

# Usage
tables = extract_tables("report.pdf")
for t in tables:
    print(f"Page {t['page']}, Table {t['table']}:")
    print(t['data'])
    print("\n")
```

---

### Advanced Table Extraction with Settings

```python
import pdfplumber

def extract_tables_advanced(pdf_path):
    """Extract tables with custom settings for better accuracy."""
    tables = []

    table_settings = {
        "vertical_strategy": "lines",
        "horizontal_strategy": "lines",
        "explicit_vertical_lines": [],
        "explicit_horizontal_lines": [],
        "snap_tolerance": 3,
        "join_tolerance": 3,
        "edge_min_length": 3,
        "min_words_vertical": 3,
        "min_words_horizontal": 1,
    }

    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            page_tables = page.extract_tables(table_settings=table_settings)
            tables.extend(page_tables)

    return tables
```

---

### Find and Extract Specific Tables

```python
import pdfplumber
import pandas as pd

def find_table_with_keyword(pdf_path, keyword):
    """Find and extract tables containing a specific keyword."""
    matching_tables = []

    with pdfplumber.open(pdf_path) as pdf:
        for page_num, page in enumerate(pdf.pages, start=1):
            tables = page.extract_tables()

            for table in tables:
                # Check if keyword exists in table
                table_text = str(table).lower()
                if keyword.lower() in table_text:
                    df = pd.DataFrame(table[1:], columns=table[0])
                    matching_tables.append({
                        'page': page_num,
                        'data': df
                    })

    return matching_tables

# Usage
sales_tables = find_table_with_keyword("report.pdf", "revenue")
```

---

## PDF Form Operations

### Fill PDF Forms (PyMuPDF)

```python
import fitz

def fill_pdf_form(input_pdf, output_pdf, field_values):
    """
    Fill PDF form fields.

    Args:
        input_pdf: Path to input PDF with form fields
        output_pdf: Path to save filled PDF
        field_values: Dictionary of {field_name: value}
    """
    doc = fitz.open(input_pdf)

    for page_num in range(len(doc)):
        page = doc[page_num]

        for widget in page.widgets():
            if widget.field_name in field_values:
                widget.field_value = field_values[widget.field_name]
                widget.update()

    doc.save(output_pdf)
    doc.close()

# Usage
form_data = {
    "name": "John Doe",
    "email": "john@example.com",
    "date": "2025-10-25",
    "signature": "John Doe"
}
fill_pdf_form("form_template.pdf", "filled_form.pdf", form_data)
```

---

### Extract Form Field Names

```python
import fitz

def get_form_fields(pdf_path):
    """Extract all form field names and their current values."""
    doc = fitz.open(pdf_path)
    fields = []

    for page_num in range(len(doc)):
        page = doc[page_num]

        for widget in page.widgets():
            fields.append({
                'page': page_num + 1,
                'name': widget.field_name,
                'type': widget.field_type_string,
                'value': widget.field_value
            })

    doc.close()
    return fields

# Usage
fields = get_form_fields("form.pdf")
for field in fields:
    print(f"{field['name']}: {field['type']} = {field['value']}")
```

---

### Flatten PDF Forms (Make Non-Editable)

```python
import fitz

def flatten_pdf_form(input_pdf, output_pdf):
    """Flatten form fields to make them non-editable."""
    doc = fitz.open(input_pdf)

    for page_num in range(len(doc)):
        page = doc[page_num]

        # Get all widgets (form fields)
        for widget in page.widgets():
            # This makes the field non-editable
            widget.update()

    # Save with form fields flattened
    doc.save(output_pdf, garbage=4, deflate=True)
    doc.close()
```

---

## Merging PDFs

### Basic Merge (pypdf)

```python
from pypdf import PdfMerger

def merge_pdfs(pdf_list, output_path):
    """Merge multiple PDFs into one."""
    merger = PdfMerger()

    for pdf in pdf_list:
        merger.append(pdf)

    merger.write(output_path)
    merger.close()

# Usage
pdfs = ["file1.pdf", "file2.pdf", "file3.pdf"]
merge_pdfs(pdfs, "merged_output.pdf")
```

---

### Merge with Page Ranges

```python
from pypdf import PdfMerger

def merge_pdfs_with_ranges(pdf_configs, output_path):
    """
    Merge PDFs with specific page ranges.

    Args:
        pdf_configs: List of dicts with 'path', 'pages' keys
        output_path: Output file path

    Example:
        configs = [
            {'path': 'doc1.pdf', 'pages': (0, 3)},  # First 3 pages
            {'path': 'doc2.pdf', 'pages': (5, 10)}, # Pages 6-10
        ]
    """
    merger = PdfMerger()

    for config in pdf_configs:
        path = config['path']
        pages = config.get('pages')

        if pages:
            merger.append(path, pages=pages)
        else:
            merger.append(path)

    merger.write(output_path)
    merger.close()

# Usage
configs = [
    {'path': 'intro.pdf', 'pages': (0, 2)},
    {'path': 'content.pdf'},  # All pages
    {'path': 'appendix.pdf', 'pages': (10, 15)}
]
merge_pdfs_with_ranges(configs, "compiled.pdf")
```

---

### Merge with Bookmarks

```python
from pypdf import PdfMerger

def merge_with_bookmarks(pdf_list, output_path, bookmark_names=None):
    """Merge PDFs and add bookmarks for each document."""
    merger = PdfMerger()

    if bookmark_names is None:
        bookmark_names = [f"Document {i+1}" for i in range(len(pdf_list))]

    for pdf, bookmark in zip(pdf_list, bookmark_names):
        merger.append(pdf, outline_item=bookmark)

    merger.write(output_path)
    merger.close()

# Usage
pdfs = ["chapter1.pdf", "chapter2.pdf", "chapter3.pdf"]
bookmarks = ["Introduction", "Methods", "Results"]
merge_with_bookmarks(pdfs, "thesis.pdf", bookmarks)
```

---

## Splitting PDFs

### Split into Individual Pages

```python
from pypdf import PdfReader, PdfWriter
import os

def split_pdf_pages(input_pdf, output_dir):
    """Split PDF into individual pages."""
    reader = PdfReader(input_pdf)

    os.makedirs(output_dir, exist_ok=True)

    for page_num, page in enumerate(reader.pages):
        writer = PdfWriter()
        writer.add_page(page)

        output_path = os.path.join(output_dir, f"page_{page_num + 1}.pdf")
        with open(output_path, 'wb') as output_file:
            writer.write(output_file)

    print(f"Split {len(reader.pages)} pages into {output_dir}")

# Usage
split_pdf_pages("document.pdf", "split_pages/")
```

---

### Split by Page Ranges

```python
from pypdf import PdfReader, PdfWriter

def split_pdf_ranges(input_pdf, ranges, output_paths):
    """
    Split PDF into multiple files by page ranges.

    Args:
        input_pdf: Input PDF path
        ranges: List of tuples (start, end) - pages are 0-indexed
        output_paths: List of output file paths
    """
    reader = PdfReader(input_pdf)

    for (start, end), output_path in zip(ranges, output_paths):
        writer = PdfWriter()

        for page_num in range(start, end):
            writer.add_page(reader.pages[page_num])

        with open(output_path, 'wb') as output_file:
            writer.write(output_file)

# Usage
ranges = [(0, 5), (5, 10), (10, 15)]
outputs = ["part1.pdf", "part2.pdf", "part3.pdf"]
split_pdf_ranges("document.pdf", ranges, outputs)
```

---

### Split by Size

```python
from pypdf import PdfReader, PdfWriter
import os

def split_pdf_by_size(input_pdf, max_size_mb, output_dir):
    """Split PDF into chunks not exceeding max size."""
    reader = PdfReader(input_pdf)

    os.makedirs(output_dir, exist_ok=True)

    current_writer = PdfWriter()
    current_size = 0
    file_count = 1

    for page in reader.pages:
        current_writer.add_page(page)

        # Estimate size (approximate)
        temp_path = f"/tmp/temp_check.pdf"
        with open(temp_path, 'wb') as f:
            current_writer.write(f)

        current_size = os.path.getsize(temp_path) / (1024 * 1024)  # MB

        if current_size >= max_size_mb:
            output_path = os.path.join(output_dir, f"part_{file_count}.pdf")
            with open(output_path, 'wb') as f:
                current_writer.write(f)

            current_writer = PdfWriter()
            current_size = 0
            file_count += 1

        os.remove(temp_path)

    # Write remaining pages
    if len(current_writer.pages) > 0:
        output_path = os.path.join(output_dir, f"part_{file_count}.pdf")
        with open(output_path, 'wb') as f:
            current_writer.write(f)
```

---

## Creating PDFs

### Basic PDF Creation (reportlab)

```python
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter

def create_simple_pdf(output_path, content):
    """Create a simple PDF with text content."""
    c = canvas.Canvas(output_path, pagesize=letter)
    width, height = letter

    # Set font
    c.setFont("Helvetica", 12)

    # Add text
    y_position = height - 50
    for line in content:
        c.drawString(50, y_position, line)
        y_position -= 20

    c.save()

# Usage
content = [
    "This is the first line",
    "This is the second line",
    "This is the third line"
]
create_simple_pdf("output.pdf", content)
```

---

### Create PDF Report with Styling

```python
from reportlab.lib.pagesizes import letter
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import inch
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.enums import TA_CENTER, TA_JUSTIFY

def create_styled_report(output_path, title, sections):
    """
    Create a professionally styled PDF report.

    Args:
        output_path: Output file path
        title: Document title
        sections: List of dicts with 'heading' and 'content' keys
    """
    doc = SimpleDocTemplate(output_path, pagesize=letter)
    story = []
    styles = getSampleStyleSheet()

    # Custom styles
    title_style = ParagraphStyle(
        'CustomTitle',
        parent=styles['Heading1'],
        fontSize=24,
        textColor='darkblue',
        alignment=TA_CENTER,
        spaceAfter=30
    )

    # Add title
    story.append(Paragraph(title, title_style))
    story.append(Spacer(1, 0.5*inch))

    # Add sections
    for section in sections:
        # Section heading
        story.append(Paragraph(section['heading'], styles['Heading2']))
        story.append(Spacer(1, 0.2*inch))

        # Section content
        for paragraph in section['content']:
            p = Paragraph(paragraph, styles['BodyText'])
            story.append(p)
            story.append(Spacer(1, 0.1*inch))

        story.append(Spacer(1, 0.3*inch))

    doc.build(story)

# Usage
sections = [
    {
        'heading': 'Introduction',
        'content': [
            'This is the introduction paragraph.',
            'It contains important information about the topic.'
        ]
    },
    {
        'heading': 'Methods',
        'content': [
            'We used various methods to conduct this research.',
            'The methodology was carefully designed.'
        ]
    }
]
create_styled_report("report.pdf", "Research Report", sections)
```

---

### Create PDF with Tables

```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph
from reportlab.lib import colors
from reportlab.lib.styles import getSampleStyleSheet

def create_pdf_with_table(output_path, title, table_data):
    """Create PDF with formatted table."""
    doc = SimpleDocTemplate(output_path, pagesize=letter)
    elements = []
    styles = getSampleStyleSheet()

    # Add title
    elements.append(Paragraph(title, styles['Title']))
    elements.append(Paragraph("<br/><br/>", styles['Normal']))

    # Create table
    table = Table(table_data)

    # Add style to table
    table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
        ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
        ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('FONTSIZE', (0, 0), (-1, 0), 14),
        ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
        ('BACKGROUND', (0, 1), (-1, -1), colors.beige),
        ('GRID', (0, 0), (-1, -1), 1, colors.black)
    ]))

    elements.append(table)
    doc.build(elements)

# Usage
data = [
    ['Product', 'Quantity', 'Price'],
    ['Widget A', '10', '$50'],
    ['Widget B', '5', '$75'],
    ['Widget C', '20', '$30']
]
create_pdf_with_table("invoice.pdf", "Sales Invoice", data)
```

---

### Create PDF with Images

```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Image, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib.units import inch

def create_pdf_with_images(output_path, title, image_paths, captions=None):
    """Create PDF with images and captions."""
    doc = SimpleDocTemplate(output_path, pagesize=letter)
    elements = []
    styles = getSampleStyleSheet()

    # Add title
    elements.append(Paragraph(title, styles['Title']))
    elements.append(Spacer(1, 0.5*inch))

    if captions is None:
        captions = [f"Image {i+1}" for i in range(len(image_paths))]

    for img_path, caption in zip(image_paths, captions):
        # Add image
        img = Image(img_path, width=4*inch, height=3*inch)
        elements.append(img)

        # Add caption
        elements.append(Spacer(1, 0.1*inch))
        elements.append(Paragraph(caption, styles['Italic']))
        elements.append(Spacer(1, 0.3*inch))

    doc.build(elements)

# Usage
images = ["chart1.png", "chart2.png", "chart3.png"]
captions = ["Sales Chart", "Revenue Chart", "Growth Chart"]
create_pdf_with_images("visual_report.pdf", "Analytics Report", images, captions)
```

---

## Watermarks and Annotations

### Add Text Watermark

```python
import fitz

def add_text_watermark(input_pdf, output_pdf, watermark_text, opacity=0.3):
    """Add text watermark to all pages."""
    doc = fitz.open(input_pdf)

    for page in doc:
        # Get page dimensions
        rect = page.rect

        # Add watermark
        text_writer = fitz.TextWriter(rect)
        text_writer.append(
            (rect.width / 2, rect.height / 2),
            watermark_text,
            fontsize=50,
            rotate=45
        )

        # Apply with opacity
        page.insert_textbox(
            rect,
            watermark_text,
            fontsize=50,
            align=fitz.TEXT_ALIGN_CENTER,
            rotate=45,
            opacity=opacity,
            color=(0.7, 0.7, 0.7)
        )

    doc.save(output_pdf)
    doc.close()

# Usage
add_text_watermark("document.pdf", "watermarked.pdf", "CONFIDENTIAL")
```

---

### Add Image Watermark

```python
import fitz

def add_image_watermark(input_pdf, output_pdf, watermark_image, opacity=0.3):
    """Add image watermark to all pages."""
    doc = fitz.open(input_pdf)

    for page in doc:
        rect = page.rect

        # Calculate watermark position (center)
        img_rect = fitz.Rect(
            rect.width / 4,
            rect.height / 4,
            3 * rect.width / 4,
            3 * rect.height / 4
        )

        # Insert image
        page.insert_image(img_rect, filename=watermark_image, overlay=True)

        # Set opacity (requires additional processing)
        # Note: Direct opacity control may require more complex operations

    doc.save(output_pdf)
    doc.close()
```

---

### Add Annotations

```python
import fitz

def add_annotations(input_pdf, output_pdf, annotations):
    """
    Add various annotations to PDF.

    Args:
        annotations: List of dicts with 'page', 'type', 'rect', 'content'
    """
    doc = fitz.open(input_pdf)

    for annot in annotations:
        page = doc[annot['page']]
        rect = fitz.Rect(annot['rect'])

        if annot['type'] == 'highlight':
            highlight = page.add_highlight_annot(rect)
            highlight.update()

        elif annot['type'] == 'text':
            text_annot = page.add_text_annot(
                rect.top_left,
                annot['content']
            )
            text_annot.update()

        elif annot['type'] == 'underline':
            underline = page.add_underline_annot(rect)
            underline.update()

        elif annot['type'] == 'strikeout':
            strike = page.add_strikeout_annot(rect)
            strike.update()

    doc.save(output_pdf)
    doc.close()

# Usage
annotations = [
    {
        'page': 0,
        'type': 'highlight',
        'rect': (100, 100, 300, 120)
    },
    {
        'page': 0,
        'type': 'text',
        'rect': (400, 400, 450, 450),
        'content': 'Important note here'
    }
]
add_annotations("document.pdf", "annotated.pdf", annotations)
```

---

### Add Stamps

```python
import fitz

def add_stamp(input_pdf, output_pdf, stamp_text, position="top-right"):
    """Add a stamp (e.g., 'APPROVED', 'DRAFT') to all pages."""
    doc = fitz.open(input_pdf)

    for page in doc:
        rect = page.rect

        # Determine stamp position
        if position == "top-right":
            stamp_rect = fitz.Rect(rect.width - 150, 20, rect.width - 20, 60)
        elif position == "top-left":
            stamp_rect = fitz.Rect(20, 20, 150, 60)
        elif position == "bottom-right":
            stamp_rect = fitz.Rect(rect.width - 150, rect.height - 60, rect.width - 20, rect.height - 20)
        else:  # center
            stamp_rect = fitz.Rect(rect.width/2 - 75, rect.height/2 - 20, rect.width/2 + 75, rect.height/2 + 20)

        # Add stamp
        page.draw_rect(stamp_rect, color=(1, 0, 0), width=2)
        page.insert_textbox(
            stamp_rect,
            stamp_text,
            fontsize=20,
            align=fitz.TEXT_ALIGN_CENTER,
            color=(1, 0, 0)
        )

    doc.save(output_pdf)
    doc.close()

# Usage
add_stamp("document.pdf", "stamped.pdf", "APPROVED", position="top-right")
```

---

## Metadata Management

### Extract Metadata

```python
from pypdf import PdfReader

def extract_metadata(pdf_path):
    """Extract PDF metadata."""
    reader = PdfReader(pdf_path)
    metadata = reader.metadata

    info = {
        'title': metadata.get('/Title', ''),
        'author': metadata.get('/Author', ''),
        'subject': metadata.get('/Subject', ''),
        'creator': metadata.get('/Creator', ''),
        'producer': metadata.get('/Producer', ''),
        'creation_date': metadata.get('/CreationDate', ''),
        'modification_date': metadata.get('/ModDate', ''),
        'pages': len(reader.pages)
    }

    return info

# Usage
metadata = extract_metadata("document.pdf")
for key, value in metadata.items():
    print(f"{key}: {value}")
```

---

### Modify Metadata

```python
from pypdf import PdfReader, PdfWriter

def modify_metadata(input_pdf, output_pdf, metadata):
    """
    Modify PDF metadata.

    Args:
        metadata: Dict with keys like '/Title', '/Author', '/Subject', etc.
    """
    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    # Copy all pages
    for page in reader.pages:
        writer.add_page(page)

    # Update metadata
    writer.add_metadata(metadata)

    with open(output_pdf, 'wb') as output_file:
        writer.write(output_file)

# Usage
new_metadata = {
    '/Title': 'Updated Title',
    '/Author': 'John Doe',
    '/Subject': 'Research Paper',
    '/Keywords': 'PDF, Python, Automation'
}
modify_metadata("document.pdf", "updated.pdf", new_metadata)
```

---

### Extract All PDF Information

```python
import fitz

def get_pdf_info(pdf_path):
    """Get comprehensive PDF information."""
    doc = fitz.open(pdf_path)

    info = {
        'metadata': doc.metadata,
        'page_count': doc.page_count,
        'is_encrypted': doc.is_encrypted,
        'is_pdf': doc.is_pdf,
        'page_sizes': []
    }

    # Get page sizes
    for page_num in range(doc.page_count):
        page = doc[page_num]
        info['page_sizes'].append({
            'page': page_num + 1,
            'width': page.rect.width,
            'height': page.rect.height
        })

    doc.close()
    return info

# Usage
info = get_pdf_info("document.pdf")
print(f"Pages: {info['page_count']}")
print(f"Title: {info['metadata'].get('title', 'N/A')}")
print(f"Encrypted: {info['is_encrypted']}")
```

---

## Security and Encryption

### Add Password Protection

```python
from pypdf import PdfReader, PdfWriter

def encrypt_pdf(input_pdf, output_pdf, user_password, owner_password=None):
    """
    Add password protection to PDF.

    Args:
        user_password: Password to open the document
        owner_password: Password for full permissions (optional)
    """
    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    # Copy all pages
    for page in reader.pages:
        writer.add_page(page)

    # Encrypt with password
    if owner_password is None:
        owner_password = user_password

    writer.encrypt(
        user_password=user_password,
        owner_password=owner_password,
        algorithm="AES-256"
    )

    with open(output_pdf, 'wb') as output_file:
        writer.write(output_file)

# Usage
encrypt_pdf("document.pdf", "encrypted.pdf", user_password="user123", owner_password="owner456")
```

---

### Decrypt PDF

```python
from pypdf import PdfReader, PdfWriter

def decrypt_pdf(input_pdf, output_pdf, password):
    """Remove password protection from PDF."""
    reader = PdfReader(input_pdf)

    # Decrypt
    if reader.is_encrypted:
        reader.decrypt(password)

    writer = PdfWriter()

    # Copy all pages
    for page in reader.pages:
        writer.add_page(page)

    with open(output_pdf, 'wb') as output_file:
        writer.write(output_file)

# Usage
decrypt_pdf("encrypted.pdf", "decrypted.pdf", password="user123")
```

---

### Set Permissions

```python
from pypdf import PdfWriter

def set_pdf_permissions(input_pdf, output_pdf, password, allow_printing=True, allow_copying=False):
    """Set specific permissions on PDF."""
    from pypdf import PdfReader

    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    for page in reader.pages:
        writer.add_page(page)

    # Set permissions
    writer.encrypt(
        user_password=password,
        owner_password=password + "_owner",
        permissions_flag=(
            (0b100 if allow_printing else 0) |
            (0b10000 if allow_copying else 0)
        )
    )

    with open(output_pdf, 'wb') as f:
        writer.write(f)
```

---

## OCR for Scanned Documents

### Basic OCR with pytesseract

```python
from pdf2image import convert_from_path
import pytesseract
from PIL import Image

def ocr_pdf(pdf_path, output_txt_path=None):
    """
    Perform OCR on scanned PDF.

    Returns extracted text from all pages.
    """
    # Convert PDF to images
    images = convert_from_path(pdf_path)

    all_text = []

    for page_num, image in enumerate(images, start=1):
        # Perform OCR
        text = pytesseract.image_to_string(image)
        all_text.append(f"--- Page {page_num} ---\n{text}\n")

    full_text = "\n".join(all_text)

    # Optionally save to file
    if output_txt_path:
        with open(output_txt_path, 'w', encoding='utf-8') as f:
            f.write(full_text)

    return full_text

# Usage
text = ocr_pdf("scanned_document.pdf", "extracted_text.txt")
print(text)
```

---

### OCR with Language Support

```python
from pdf2image import convert_from_path
import pytesseract

def ocr_pdf_multilang(pdf_path, languages='eng'):
    """
    Perform OCR with multiple language support.

    Args:
        languages: Language codes separated by '+' (e.g., 'eng+fra+deu')
    """
    images = convert_from_path(pdf_path)
    all_text = []

    for image in images:
        text = pytesseract.image_to_string(image, lang=languages)
        all_text.append(text)

    return "\n\n".join(all_text)

# Usage
text = ocr_pdf_multilang("french_document.pdf", languages='fra')
```

---

### Create Searchable PDF from Scanned PDF

```python
from pdf2image import convert_from_path
import pytesseract
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter
import fitz

def create_searchable_pdf(input_pdf, output_pdf):
    """Convert scanned PDF to searchable PDF with OCR layer."""
    # Convert to images
    images = convert_from_path(input_pdf)

    # Create new PDF with OCR text
    temp_pdfs = []

    for i, image in enumerate(images):
        # Perform OCR
        text = pytesseract.image_to_string(image)

        # Create PDF page with invisible text layer
        temp_pdf = f"/tmp/page_{i}.pdf"
        c = canvas.Canvas(temp_pdf, pagesize=letter)

        # Add invisible OCR text
        c.setFillColorRGB(1, 1, 1, alpha=0)  # Transparent
        text_obj = c.beginText(40, 750)
        text_obj.setFont("Helvetica", 10)

        for line in text.split('\n'):
            text_obj.textLine(line)

        c.drawText(text_obj)
        c.save()

        temp_pdfs.append(temp_pdf)

    # Merge all pages
    from pypdf import PdfMerger
    merger = PdfMerger()
    for pdf in temp_pdfs:
        merger.append(pdf)
    merger.write(output_pdf)
    merger.close()

    # Clean up
    import os
    for pdf in temp_pdfs:
        os.remove(pdf)

# Usage
create_searchable_pdf("scanned.pdf", "searchable.pdf")
```

---

## Best Practices

### 1. Memory Management for Large PDFs

```python
from pypdf import PdfReader
import gc

def process_large_pdf_in_chunks(pdf_path, chunk_size=10):
    """Process large PDFs in chunks to manage memory."""
    reader = PdfReader(pdf_path)
    total_pages = len(reader.pages)

    for start in range(0, total_pages, chunk_size):
        end = min(start + chunk_size, total_pages)

        # Process chunk
        for page_num in range(start, end):
            page = reader.pages[page_num]
            text = page.extract_text()

            # Process text here
            yield page_num, text

        # Force garbage collection
        gc.collect()

# Usage
for page_num, text in process_large_pdf_in_chunks("large_file.pdf"):
    print(f"Processing page {page_num}")
```

---

### 2. Handle Text Encoding Issues

```python
import pdfplumber

def extract_text_safe(pdf_path):
    """Extract text with proper encoding handling."""
    with pdfplumber.open(pdf_path) as pdf:
        all_text = []

        for page in pdf.pages:
            text = page.extract_text()

            if text:
                # Handle encoding issues
                text = text.encode('utf-8', errors='ignore').decode('utf-8')
                all_text.append(text)

        return "\n\n".join(all_text)
```

---

### 3. Preserve Document Structure

```python
import pdfplumber

def extract_structured_content(pdf_path):
    """Extract content while preserving structure."""
    with pdfplumber.open(pdf_path) as pdf:
        structured_data = []

        for page in pdf.pages:
            page_data = {
                'page_number': page.page_number,
                'text': page.extract_text(),
                'tables': page.extract_tables(),
                'images': len(page.images),
                'width': page.width,
                'height': page.height
            }

            structured_data.append(page_data)

        return structured_data
```

---

### 4. Error Handling Template

```python
from pypdf import PdfReader
import logging

def safe_pdf_operation(pdf_path):
    """Template for safe PDF operations with error handling."""
    try:
        reader = PdfReader(pdf_path)

        # Check if encrypted
        if reader.is_encrypted:
            logging.warning(f"PDF {pdf_path} is encrypted")
            return None

        # Perform operations
        result = []
        for page in reader.pages:
            try:
                text = page.extract_text()
                result.append(text)
            except Exception as e:
                logging.error(f"Error extracting page: {e}")
                result.append("")

        return result

    except FileNotFoundError:
        logging.error(f"File not found: {pdf_path}")
        return None
    except Exception as e:
        logging.error(f"Error processing PDF: {e}")
        return None
```

---

### 5. Optimize PDF Size

```python
import fitz

def optimize_pdf(input_pdf, output_pdf, image_quality=50):
    """Compress and optimize PDF file size."""
    doc = fitz.open(input_pdf)

    # Compress with optimization
    doc.save(
        output_pdf,
        garbage=4,  # Maximum garbage collection
        deflate=True,  # Compress streams
        clean=True,  # Clean up content
        pretty=False  # No pretty-printing
    )

    doc.close()

    import os
    original_size = os.path.getsize(input_pdf) / (1024 * 1024)
    optimized_size = os.path.getsize(output_pdf) / (1024 * 1024)

    print(f"Original: {original_size:.2f} MB")
    print(f"Optimized: {optimized_size:.2f} MB")
    print(f"Reduction: {((original_size - optimized_size) / original_size * 100):.1f}%")

# Usage
optimize_pdf("large_file.pdf", "optimized.pdf")
```

---

## Common Pitfalls

### 1. Scanned Documents Without OCR

**Problem**: Text extraction returns empty strings for scanned PDFs.

**Solution**: Use OCR (pytesseract + pdf2image)

```python
import fitz

def extract_text_with_ocr_fallback(pdf_path):
    """Try text extraction, fall back to OCR if needed."""
    doc = fitz.open(pdf_path)
    page = doc[0]
    text = page.get_text()

    if not text.strip():
        print("No text found, using OCR...")
        from pdf2image import convert_from_path
        import pytesseract

        images = convert_from_path(pdf_path)
        text = pytesseract.image_to_string(images[0])

    return text
```

---

### 2. Table Detection Accuracy

**Problem**: Tables not detected or extracted incorrectly.

**Solution**: Adjust table detection settings

```python
import pdfplumber

def extract_tables_robust(pdf_path):
    """Extract tables with multiple strategies."""
    with pdfplumber.open(pdf_path) as pdf:
        page = pdf.pages[0]

        # Try different strategies
        strategies = [
            {"vertical_strategy": "lines", "horizontal_strategy": "lines"},
            {"vertical_strategy": "text", "horizontal_strategy": "text"},
            {"vertical_strategy": "lines", "horizontal_strategy": "text"}
        ]

        for strategy in strategies:
            tables = page.extract_tables(table_settings=strategy)
            if tables:
                return tables

        return []
```

---

### 3. Form Field Identification

**Problem**: Can't find form field names.

**Solution**: Inspect and list all fields first

```python
import fitz

def debug_form_fields(pdf_path):
    """Debug helper to see all form fields."""
    doc = fitz.open(pdf_path)

    print("=== Form Fields ===")
    for page_num in range(len(doc)):
        page = doc[page_num]
        widgets = page.widgets()

        if widgets:
            print(f"\nPage {page_num + 1}:")
            for widget in widgets:
                print(f"  Name: {widget.field_name}")
                print(f"  Type: {widget.field_type_string}")
                print(f"  Value: {widget.field_value}")
                print(f"  Rect: {widget.rect}")
                print("  ---")

    doc.close()
```

---

### 4. Encrypted PDFs

**Problem**: Operations fail on encrypted PDFs.

**Solution**: Check and handle encryption

```python
from pypdf import PdfReader

def handle_encrypted_pdf(pdf_path, password=None):
    """Safely handle encrypted PDFs."""
    reader = PdfReader(pdf_path)

    if reader.is_encrypted:
        if password:
            success = reader.decrypt(password)
            if success == 0:
                print("Incorrect password")
                return None
        else:
            print("PDF is encrypted, password required")
            return None

    # Now safe to process
    return reader
```

---

### 5. Page Rotation Issues

**Problem**: Extracted text appears rotated or out of order.

**Solution**: Check and handle page rotation

```python
import fitz

def extract_text_handle_rotation(pdf_path):
    """Extract text accounting for page rotation."""
    doc = fitz.open(pdf_path)

    for page in doc:
        # Check rotation
        rotation = page.rotation

        if rotation != 0:
            # Rotate page to 0 degrees
            page.set_rotation(0)

        text = page.get_text()
        print(text)

    doc.close()
```

---

## Additional Utilities

### Extract Images from PDF

```python
import fitz
import os

def extract_images(pdf_path, output_dir):
    """Extract all images from PDF."""
    doc = fitz.open(pdf_path)
    os.makedirs(output_dir, exist_ok=True)

    image_count = 0

    for page_num in range(len(doc)):
        page = doc[page_num]
        images = page.get_images()

        for img_index, img in enumerate(images):
            xref = img[0]
            base_image = doc.extract_image(xref)

            image_bytes = base_image["image"]
            image_ext = base_image["ext"]

            image_filename = os.path.join(
                output_dir,
                f"page{page_num + 1}_img{img_index + 1}.{image_ext}"
            )

            with open(image_filename, "wb") as img_file:
                img_file.write(image_bytes)

            image_count += 1

    print(f"Extracted {image_count} images to {output_dir}")
    doc.close()

# Usage
extract_images("document.pdf", "extracted_images/")
```

---

### Convert Images to PDF

```python
from PIL import Image
from reportlab.pdfgen import canvas
from reportlab.lib.utils import ImageReader

def images_to_pdf(image_paths, output_pdf):
    """Convert multiple images to a single PDF."""
    c = canvas.Canvas(output_pdf)

    for img_path in image_paths:
        img = Image.open(img_path)
        width, height = img.size

        # Set page size to image size
        c.setPageSize((width, height))

        # Draw image
        c.drawImage(img_path, 0, 0, width=width, height=height)
        c.showPage()

    c.save()

# Usage
images = ["scan1.jpg", "scan2.jpg", "scan3.jpg"]
images_to_pdf(images, "scanned_document.pdf")
```

---

### Rotate Pages

```python
from pypdf import PdfReader, PdfWriter

def rotate_pages(input_pdf, output_pdf, rotation=90, pages=None):
    """
    Rotate specific pages in PDF.

    Args:
        rotation: Degrees to rotate (90, 180, 270)
        pages: List of page numbers (0-indexed), or None for all pages
    """
    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    for page_num, page in enumerate(reader.pages):
        if pages is None or page_num in pages:
            page.rotate(rotation)
        writer.add_page(page)

    with open(output_pdf, 'wb') as output_file:
        writer.write(output_file)

# Usage
rotate_pages("document.pdf", "rotated.pdf", rotation=90, pages=[0, 2, 4])
```

---

### Count Words in PDF

```python
import pdfplumber

def count_words_in_pdf(pdf_path):
    """Count total words in PDF."""
    total_words = 0

    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            text = page.extract_text()
            if text:
                words = text.split()
                total_words += len(words)

    return total_words

# Usage
word_count = count_words_in_pdf("document.pdf")
print(f"Total words: {word_count}")
```

---

## Summary

This skill provides comprehensive PDF manipulation capabilities:

- **Extract**: Text, tables, images, metadata
- **Create**: Reports, invoices, documents with styling
- **Modify**: Merge, split, rotate, compress
- **Secure**: Encrypt, decrypt, set permissions
- **Enhance**: Watermarks, annotations, stamps
- **Process**: OCR for scanned documents

Use the helper script (`scripts/pdf_helper.py`) for common operations, or refer to specific sections above for detailed implementations.

**Quick Reference:**
- Text extraction → `pdfplumber` or `pypdf`
- Table extraction → `pdfplumber`
- PDF creation → `reportlab`
- Advanced manipulation → `PyMuPDF (fitz)`
- OCR → `pytesseract` + `pdf2image`
- Forms → `PyMuPDF`
- Merging/Splitting → `pypdf`

For production use, always implement proper error handling, validate inputs, and test with various PDF types and versions.
