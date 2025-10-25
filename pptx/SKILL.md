---
name: pptx
description: "Professional PowerPoint presentation creation, editing, and automation with support for layouts, templates, charts, images, and formatting. Use when working with .pptx files for: (1) Creating presentations from scratch, (2) Editing existing presentations, (3) Applying templates and themes, (4) Adding charts and visualizations, (5) Bulk slide generation, (6) Presentation automation"
---

# PowerPoint (PPTX) Skill

## Overview

This skill provides comprehensive PowerPoint presentation creation, editing, and automation capabilities using Python's `python-pptx` library. Create professional presentations programmatically with full control over layouts, themes, content, charts, and visualizations.

## Core Capabilities

### 1. Presentation Creation
- Create new presentations from scratch
- Clone existing presentations as templates
- Set presentation metadata (title, author, subject, keywords)
- Configure page size and orientation
- Save in multiple formats (.pptx, .ppt compatibility mode)

### 2. Slide Management
- Add slides with predefined layouts
- Duplicate existing slides
- Delete, reorder, and organize slides
- Section management for large presentations
- Slide notes and speaker notes

### 3. Content Types
- **Text**: Title, subtitle, body text, bullet points
- **Shapes**: Rectangles, circles, arrows, connectors
- **Images**: PNG, JPEG, GIF, BMP with sizing and positioning
- **Tables**: Formatted tables with cell styling
- **Charts**: Bar, line, pie, scatter, area charts
- **SmartArt**: Diagrams and process flows (limited support)
- **Hyperlinks**: Internal and external links

### 4. Design & Formatting
- Apply themes and color schemes
- Master slide customization
- Font management (typeface, size, color, bold, italic)
- Paragraph formatting (alignment, spacing, indentation)
- Fill types (solid, gradient, pattern, picture)
- Border and line styling
- Shadow and 3D effects

### 5. Advanced Features
- Slide transitions
- Animation effects
- Embedded objects
- Video and audio embedding
- Custom XML parts
- Comments and revision tracking

## Python Libraries

### Primary: python-pptx

**Installation:**
```bash
pip install python-pptx
# or with uv
uv pip install python-pptx
```

**Import:**
```python
from pptx import Presentation
from pptx.util import Inches, Pt, Cm
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR, MSO_AUTO_SIZE
from pptx.enum.chart import XL_CHART_TYPE, XL_LEGEND_POSITION
from pptx.chart.data import CategoryChartData
from pptx.dml.color import RGBColor
```

**Documentation:**
- Official Docs: https://python-pptx.readthedocs.io/
- GitHub: https://github.com/scanny/python-pptx

### Supporting Libraries

**Pillow (PIL)**: Image processing before insertion
```bash
pip install Pillow
```

**pandas**: Data preparation for charts and tables
```bash
pip install pandas
```

**matplotlib**: Chart generation (convert to images)
```bash
pip install matplotlib
```

## Detailed Workflows

### Workflow 1: Creating a Business Presentation from Scratch

**Goal:** Create a professional presentation with title slide, agenda, content slides, and conclusion.

**Steps:**

1. **Initialize Presentation**
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor

prs = Presentation()
prs.slide_width = Inches(10)  # Standard 16:9
prs.slide_height = Inches(7.5)

# Set metadata
core_props = prs.core_properties
core_props.title = "Q4 2025 Business Review"
core_props.author = "Jane Doe"
core_props.subject = "Quarterly Review"
core_props.keywords = "business, Q4, review, 2025"
```

2. **Add Title Slide**
```python
title_slide_layout = prs.slide_layouts[0]  # Title Slide layout
slide = prs.slides.add_slide(title_slide_layout)

title = slide.shapes.title
subtitle = slide.placeholders[1]

title.text = "Q4 2025 Business Review"
subtitle.text = "Prepared by: Jane Doe\nDate: October 25, 2025"

# Format title
title.text_frame.paragraphs[0].font.size = Pt(44)
title.text_frame.paragraphs[0].font.bold = True
title.text_frame.paragraphs[0].font.color.rgb = RGBColor(0, 51, 102)
```

3. **Add Agenda Slide**
```python
bullet_slide_layout = prs.slide_layouts[1]  # Title and Content
slide = prs.slides.add_slide(bullet_slide_layout)

title = slide.shapes.title
title.text = "Agenda"

body_shape = slide.placeholders[1]
tf = body_shape.text_frame
tf.clear()  # Clear default text

# Add agenda items
agenda_items = [
    "Executive Summary",
    "Financial Performance",
    "Market Analysis",
    "Product Updates",
    "Team Achievements",
    "Q1 2026 Goals"
]

for item in agenda_items:
    p = tf.add_paragraph()
    p.text = item
    p.level = 0
    p.font.size = Pt(24)
    p.space_before = Pt(12)
```

4. **Add Content Slide with Two Columns**
```python
blank_slide_layout = prs.slide_layouts[6]  # Blank layout
slide = prs.slides.add_slide(blank_slide_layout)

# Add title manually
left = Inches(0.5)
top = Inches(0.5)
width = Inches(9)
height = Inches(0.8)

title_shape = slide.shapes.add_textbox(left, top, width, height)
title_frame = title_shape.text_frame
title_frame.text = "Key Highlights"
title_frame.paragraphs[0].font.size = Pt(32)
title_frame.paragraphs[0].font.bold = True

# Left column
left_box = slide.shapes.add_textbox(Inches(0.5), Inches(1.5), Inches(4.5), Inches(5))
tf = left_box.text_frame
tf.text = "Revenue Growth"
p = tf.add_paragraph()
p.text = "• 25% increase YoY"
p.level = 1
p = tf.add_paragraph()
p.text = "• Exceeded targets by 15%"
p.level = 1

# Right column
right_box = slide.shapes.add_textbox(Inches(5.5), Inches(1.5), Inches(4), Inches(5))
tf = right_box.text_frame
tf.text = "Customer Satisfaction"
p = tf.add_paragraph()
p.text = "• NPS score: 82"
p.level = 1
p = tf.add_paragraph()
p.text = "• 95% retention rate"
p.level = 1
```

5. **Save Presentation**
```python
prs.save('Q4_Business_Review.pptx')
```

### Workflow 2: Adding Charts Programmatically

**Goal:** Create data visualizations with various chart types.

**Bar Chart:**
```python
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE
from pptx.util import Inches

# Prepare slide
slide = prs.slides.add_slide(prs.slide_layouts[5])  # Title Only
title = slide.shapes.title
title.text = "Quarterly Revenue Comparison"

# Define chart data
chart_data = CategoryChartData()
chart_data.categories = ['Q1', 'Q2', 'Q3', 'Q4']
chart_data.add_series('2024', (8.2, 9.1, 8.8, 10.5))
chart_data.add_series('2025', (9.5, 10.8, 11.2, 13.1))

# Add chart
x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(4.5)
chart = slide.shapes.add_chart(
    XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart_data
).chart

# Format chart
chart.has_legend = True
chart.legend.position = XL_LEGEND_POSITION.BOTTOM
chart.legend.include_in_layout = False

# Format value axis
value_axis = chart.value_axis
value_axis.has_major_gridlines = True
value_axis.maximum_scale = 15.0

# Format plot area
plot = chart.plots[0]
plot.has_data_labels = True
```

**Line Chart:**
```python
chart_data = CategoryChartData()
chart_data.categories = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun']
chart_data.add_series('Website Traffic', (25000, 28000, 31000, 29000, 33000, 36000))
chart_data.add_series('Conversions', (1250, 1400, 1550, 1450, 1650, 1800))

x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(4.5)
chart = slide.shapes.add_chart(
    XL_CHART_TYPE.LINE, x, y, cx, cy, chart_data
).chart

chart.has_legend = True
chart.legend.position = XL_LEGEND_POSITION.RIGHT
```

**Pie Chart:**
```python
chart_data = CategoryChartData()
chart_data.categories = ['North America', 'Europe', 'Asia Pacific', 'Latin America']
chart_data.add_series('Market Share', (38, 28, 25, 9))

x, y, cx, cy = Inches(2), Inches(2), Inches(6), Inches(4.5)
chart = slide.shapes.add_chart(
    XL_CHART_TYPE.PIE, x, y, cx, cy, chart_data
).chart

chart.has_legend = True
chart.legend.position = XL_LEGEND_POSITION.RIGHT
chart.plots[0].has_data_labels = True

# Format data labels
data_labels = chart.plots[0].data_labels
data_labels.show_percentage = True
data_labels.show_category_name = True
```

### Workflow 3: Working with Images

**Goal:** Add, position, and format images in slides.

**Basic Image Insertion:**
```python
from pptx.util import Inches

slide = prs.slides.add_slide(prs.slide_layouts[6])  # Blank

# Add image
img_path = 'company_logo.png'
left = Inches(1)
top = Inches(1)
height = Inches(2)  # Width will auto-scale to maintain aspect ratio

pic = slide.shapes.add_picture(img_path, left, top, height=height)

# Access image properties
print(f"Image size: {pic.width} x {pic.height}")
print(f"Image position: ({pic.left}, {pic.top})")
```

**Image with Specific Dimensions:**
```python
# Add with both width and height (may distort aspect ratio)
pic = slide.shapes.add_picture(
    'chart_screenshot.png',
    Inches(0.5),  # left
    Inches(2),    # top
    Inches(9),    # width
    Inches(5)     # height
)
```

**Center Image on Slide:**
```python
from pptx.util import Inches

img_path = 'hero_image.jpg'
slide = prs.slides.add_slide(prs.slide_layouts[6])

# Add image first
pic = slide.shapes.add_picture(img_path, Inches(0), Inches(0), height=Inches(4))

# Calculate centered position
slide_width = prs.slide_width
slide_height = prs.slide_height

pic.left = int((slide_width - pic.width) / 2)
pic.top = int((slide_height - pic.height) / 2)
```

**Image Processing with Pillow:**
```python
from PIL import Image
from io import BytesIO

# Resize image before adding to presentation
img = Image.open('large_photo.jpg')
img.thumbnail((1920, 1080))  # Resize to max 1920x1080

# Save to bytes
img_bytes = BytesIO()
img.save(img_bytes, format='PNG')
img_bytes.seek(0)

# Add to slide
pic = slide.shapes.add_picture(img_bytes, Inches(1), Inches(1), height=Inches(5))
```

### Workflow 4: Creating and Formatting Tables

**Goal:** Add structured data tables with formatting.

**Basic Table:**
```python
from pptx.util import Inches

slide = prs.slides.add_slide(prs.slide_layouts[5])  # Title Only
title = slide.shapes.title
title.text = "Product Comparison"

# Define table dimensions
rows, cols = 4, 3
left = Inches(1.5)
top = Inches(2)
width = Inches(7)
height = Inches(3)

# Add table
table = slide.shapes.add_table(rows, cols, left, top, width, height).table

# Set column widths
table.columns[0].width = Inches(3)
table.columns[1].width = Inches(2)
table.columns[2].width = Inches(2)

# Populate headers
headers = ['Product', 'Price', 'Sales']
for col_idx, header in enumerate(headers):
    cell = table.cell(0, col_idx)
    cell.text = header
    cell.text_frame.paragraphs[0].font.bold = True
    cell.text_frame.paragraphs[0].font.size = Pt(14)
    cell.fill.solid()
    cell.fill.fore_color.rgb = RGBColor(0, 51, 102)
    cell.text_frame.paragraphs[0].font.color.rgb = RGBColor(255, 255, 255)

# Populate data
data = [
    ['Widget A', '$299', '1,234'],
    ['Widget B', '$399', '2,456'],
    ['Widget C', '$499', '3,789']
]

for row_idx, row_data in enumerate(data, start=1):
    for col_idx, value in enumerate(row_data):
        cell = table.cell(row_idx, col_idx)
        cell.text = value
        cell.text_frame.paragraphs[0].font.size = Pt(12)
```

**Advanced Table Formatting:**
```python
# Merge cells
cell1 = table.cell(0, 0)
cell2 = table.cell(0, 1)
merged_cell = cell1.merge(cell2)
merged_cell.text = "Product Information"

# Cell alignment
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR

cell = table.cell(1, 1)
cell.text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER
cell.vertical_anchor = MSO_ANCHOR.MIDDLE

# Cell borders
from pptx.oxml.xmlchemy import OxmlElement

def set_cell_border(cell, border_color="000000", border_width='12700'):
    """
    Set cell border properties.
    border_width in EMUs (914400 EMUs = 1 inch)
    12700 = 1pt
    """
    tc = cell._tc
    tcPr = tc.get_or_add_tcPr()

    for border_name in ['lnL', 'lnR', 'lnT', 'lnB']:
        ln = OxmlElement(f'a:{border_name}')
        ln.set('w', border_width)
        ln.set('cap', 'flat')
        ln.set('cmpd', 'sng')
        ln.set('algn', 'ctr')

        solidFill = OxmlElement('a:solidFill')
        srgbClr = OxmlElement('a:srgbClr')
        srgbClr.set('val', border_color)
        solidFill.append(srgbClr)
        ln.append(solidFill)

        tcPr.append(ln)

# Apply border to all cells
for row in table.rows:
    for cell in row.cells:
        set_cell_border(cell, "000000", "12700")
```

### Workflow 5: Working with Existing Presentations

**Goal:** Edit and update existing PowerPoint files.

**Open and Modify:**
```python
from pptx import Presentation

# Open existing presentation
prs = Presentation('existing_presentation.pptx')

# Access slides
print(f"Total slides: {len(prs.slides)}")

# Iterate through slides
for idx, slide in enumerate(prs.slides):
    print(f"Slide {idx}: {slide.slide_layout.name}")

    # Access shapes
    for shape in slide.shapes:
        if hasattr(shape, "text"):
            print(f"  - {shape.text}")

# Modify specific slide
slide = prs.slides[2]  # Third slide (0-indexed)

# Find and update text
for shape in slide.shapes:
    if hasattr(shape, "text_frame"):
        if "Old Company Name" in shape.text:
            shape.text = shape.text.replace("Old Company Name", "New Company Name")

# Add new slide at specific position
new_slide = prs.slides.add_slide(prs.slide_layouts[1])
# Note: add_slide adds to end; to insert at position, use XML manipulation

# Save modified presentation
prs.save('updated_presentation.pptx')
```

**Copy Slide from Another Presentation:**
```python
from pptx import Presentation
import copy

source_prs = Presentation('source.pptx')
target_prs = Presentation('target.pptx')

# Get slide to copy
source_slide = source_prs.slides[0]

# Copy slide layout
slide_layout = target_prs.slide_layouts[source_slide.slide_layout.slide_layout_index]

# Add new slide
copied_slide = target_prs.slides.add_slide(slide_layout)

# Copy shapes (simplified; full copy requires deep cloning)
for shape in source_slide.shapes:
    el = shape.element
    newel = copy.deepcopy(el)
    copied_slide.shapes._spTree.insert_element_before(newel, 'p:extLst')

target_prs.save('target_with_copied_slide.pptx')
```

### Workflow 6: Applying Templates and Themes

**Goal:** Use master slides and templates for consistent branding.

**Using Template as Base:**
```python
# Start with template
prs = Presentation('corporate_template.pptx')

# Template already has master slides and layouts
print(f"Available layouts: {len(prs.slide_layouts)}")
for idx, layout in enumerate(prs.slide_layouts):
    print(f"{idx}: {layout.name}")

# Use specific layout
title_slide = prs.slides.add_slide(prs.slide_layouts[0])
content_slide = prs.slides.add_slide(prs.slide_layouts[1])

# Layouts inherit formatting from master
prs.save('presentation_from_template.pptx')
```

**Accessing Master Slides:**
```python
# Access slide master
slide_master = prs.slide_master

# Access master shapes (logo, footer, etc.)
for shape in slide_master.shapes:
    if shape.name == "Company Logo":
        # Update logo
        shape.image = 'new_logo.png'
```

**Creating Custom Color Scheme:**
```python
from pptx.dml.color import RGBColor

# Define brand colors
BRAND_COLORS = {
    'primary': RGBColor(0, 51, 102),      # Dark Blue
    'secondary': RGBColor(0, 153, 204),   # Light Blue
    'accent': RGBColor(255, 102, 0),      # Orange
    'text': RGBColor(51, 51, 51),         # Dark Gray
    'background': RGBColor(255, 255, 255) # White
}

# Apply to text
shape.text_frame.paragraphs[0].font.color.rgb = BRAND_COLORS['primary']

# Apply to fill
shape.fill.solid()
shape.fill.fore_color.rgb = BRAND_COLORS['secondary']
```

### Workflow 7: Bulk Slide Generation from Data

**Goal:** Generate multiple slides automatically from structured data.

**From Pandas DataFrame:**
```python
import pandas as pd
from pptx import Presentation
from pptx.util import Inches, Pt

# Load data
df = pd.read_csv('employee_data.csv')

# Create presentation
prs = Presentation()

# Add title slide
title_slide = prs.slides.add_slide(prs.slide_layouts[0])
title_slide.shapes.title.text = "Employee Directory"

# Create one slide per employee
for _, row in df.iterrows():
    slide = prs.slides.add_slide(prs.slide_layouts[1])

    # Title: Employee name
    title = slide.shapes.title
    title.text = row['Name']

    # Content: Employee details
    body_shape = slide.placeholders[1]
    tf = body_shape.text_frame
    tf.clear()

    details = [
        f"Position: {row['Position']}",
        f"Department: {row['Department']}",
        f"Email: {row['Email']}",
        f"Phone: {row['Phone']}"
    ]

    for detail in details:
        p = tf.add_paragraph()
        p.text = detail
        p.level = 0
        p.font.size = Pt(18)

prs.save('employee_directory.pptx')
```

**From JSON Data:**
```python
import json
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE

# Load JSON data
with open('sales_data.json', 'r') as f:
    data = json.load(f)

prs = Presentation()

# Generate slides for each region
for region_data in data['regions']:
    slide = prs.slides.add_slide(prs.slide_layouts[5])  # Title Only

    title = slide.shapes.title
    title.text = f"{region_data['name']} - Sales Performance"

    # Add chart
    chart_data = CategoryChartData()
    chart_data.categories = [m['month'] for m in region_data['monthly_sales']]
    chart_data.add_series('Sales', [m['amount'] for m in region_data['monthly_sales']])

    x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(4)
    chart = slide.shapes.add_chart(
        XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart_data
    ).chart

    # Add summary text
    text_box = slide.shapes.add_textbox(Inches(1), Inches(6.5), Inches(8), Inches(0.5))
    tf = text_box.text_frame
    tf.text = f"Total Sales: ${region_data['total']:,.2f} | Growth: {region_data['growth']}%"
    tf.paragraphs[0].font.size = Pt(14)
    tf.paragraphs[0].font.bold = True

prs.save('regional_sales.pptx')
```

## Code Examples

### Example 1: Complete Professional Presentation

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE, XL_LEGEND_POSITION

def create_professional_presentation():
    """Create a complete professional business presentation."""

    # Initialize
    prs = Presentation()
    prs.slide_width = Inches(10)
    prs.slide_height = Inches(7.5)

    # Brand colors
    PRIMARY = RGBColor(0, 51, 102)
    SECONDARY = RGBColor(0, 153, 204)
    ACCENT = RGBColor(255, 102, 0)

    # 1. Title Slide
    slide = prs.slides.add_slide(prs.slide_layouts[0])
    title = slide.shapes.title
    subtitle = slide.placeholders[1]

    title.text = "Annual Report 2025"
    title.text_frame.paragraphs[0].font.size = Pt(54)
    title.text_frame.paragraphs[0].font.bold = True
    title.text_frame.paragraphs[0].font.color.rgb = PRIMARY

    subtitle.text = "Innovation • Growth • Excellence\nPresented by: Leadership Team"

    # 2. Agenda Slide
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    title = slide.shapes.title
    title.text = "Agenda"

    body = slide.placeholders[1]
    tf = body.text_frame
    tf.clear()

    agenda_items = [
        "Executive Summary",
        "Financial Highlights",
        "Market Position",
        "Innovation & R&D",
        "2026 Strategic Goals"
    ]

    for item in agenda_items:
        p = tf.add_paragraph()
        p.text = item
        p.level = 0
        p.font.size = Pt(24)
        p.space_before = Pt(14)

    # 3. Executive Summary
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    title = slide.shapes.title
    title.text = "Executive Summary"

    body = slide.placeholders[1]
    tf = body.text_frame
    tf.clear()

    summary = [
        ("Record Revenue", "Achieved $500M in annual revenue, 35% YoY growth"),
        ("Market Expansion", "Entered 12 new markets across Asia and Europe"),
        ("Customer Growth", "2M+ active customers, 92% satisfaction rate"),
        ("Product Innovation", "Launched 5 major products, 15+ features")
    ]

    for heading, detail in summary:
        p = tf.add_paragraph()
        p.text = heading
        p.level = 0
        p.font.size = Pt(20)
        p.font.bold = True
        p.font.color.rgb = PRIMARY

        p = tf.add_paragraph()
        p.text = detail
        p.level = 1
        p.font.size = Pt(16)

    # 4. Financial Chart
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    title = slide.shapes.title
    title.text = "Revenue Growth (2020-2025)"

    chart_data = CategoryChartData()
    chart_data.categories = ['2020', '2021', '2022', '2023', '2024', '2025']
    chart_data.add_series('Revenue ($M)', (250, 285, 320, 370, 430, 500))

    x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(4.5)
    chart = slide.shapes.add_chart(
        XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart_data
    ).chart

    chart.has_legend = True
    chart.legend.position = XL_LEGEND_POSITION.BOTTOM
    chart.plots[0].has_data_labels = True

    # 5. Market Position - Pie Chart
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    title = slide.shapes.title
    title.text = "Market Share by Region"

    chart_data = CategoryChartData()
    chart_data.categories = ['North America', 'Europe', 'Asia Pacific', 'Other']
    chart_data.add_series('Market Share', (42, 28, 23, 7))

    x, y, cx, cy = Inches(2), Inches(2), Inches(6), Inches(4.5)
    chart = slide.shapes.add_chart(
        XL_CHART_TYPE.PIE, x, y, cx, cy, chart_data
    ).chart

    chart.has_legend = True
    chart.legend.position = XL_LEGEND_POSITION.RIGHT
    chart.plots[0].has_data_labels = True

    # 6. Thank You Slide
    slide = prs.slides.add_slide(prs.slide_layouts[6])  # Blank

    text_box = slide.shapes.add_textbox(
        Inches(2), Inches(2.5), Inches(6), Inches(2)
    )
    tf = text_box.text_frame
    tf.text = "Thank You"

    p = tf.paragraphs[0]
    p.alignment = PP_ALIGN.CENTER
    p.font.size = Pt(60)
    p.font.bold = True
    p.font.color.rgb = PRIMARY

    p = tf.add_paragraph()
    p.text = "Questions?"
    p.alignment = PP_ALIGN.CENTER
    p.font.size = Pt(32)
    p.font.color.rgb = SECONDARY

    # Save
    prs.save('annual_report_2025.pptx')
    print("✅ Presentation created: annual_report_2025.pptx")

if __name__ == "__main__":
    create_professional_presentation()
```

### Example 2: Image Gallery Presentation

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from PIL import Image
import os

def create_image_gallery(image_folder, output_file="image_gallery.pptx"):
    """Create a presentation with one image per slide."""

    prs = Presentation()
    prs.slide_width = Inches(10)
    prs.slide_height = Inches(7.5)

    # Get all image files
    image_extensions = ('.png', '.jpg', '.jpeg', '.gif', '.bmp')
    image_files = [
        f for f in os.listdir(image_folder)
        if f.lower().endswith(image_extensions)
    ]

    # Title slide
    title_slide = prs.slides.add_slide(prs.slide_layouts[0])
    title_slide.shapes.title.text = "Image Gallery"
    title_slide.placeholders[1].text = f"{len(image_files)} Images"

    # Add one slide per image
    for img_file in image_files:
        img_path = os.path.join(image_folder, img_file)

        # Get image dimensions
        with Image.open(img_path) as img:
            img_width, img_height = img.size
            aspect_ratio = img_width / img_height

        # Create slide
        slide = prs.slides.add_slide(prs.slide_layouts[6])  # Blank

        # Calculate dimensions to fit slide
        max_width = Inches(9)
        max_height = Inches(6.5)

        if aspect_ratio > max_width / max_height:
            # Width-constrained
            width = max_width
            height = width / aspect_ratio
        else:
            # Height-constrained
            height = max_height
            width = height * aspect_ratio

        # Center image
        left = (prs.slide_width - width) / 2
        top = (prs.slide_height - height) / 2

        # Add image
        slide.shapes.add_picture(img_path, left, top, width, height)

        # Add caption
        caption_box = slide.shapes.add_textbox(
            Inches(0.5), Inches(6.8), Inches(9), Inches(0.5)
        )
        tf = caption_box.text_frame
        tf.text = os.path.splitext(img_file)[0]  # Filename without extension
        tf.paragraphs[0].font.size = Pt(14)
        tf.paragraphs[0].alignment = PP_ALIGN.CENTER

    prs.save(output_file)
    print(f"✅ Gallery created: {output_file} ({len(image_files)} images)")

# Usage
# create_image_gallery('/path/to/images')
```

### Example 3: Data Report with Tables

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN
import pandas as pd

def create_data_report(csv_file, output_file="data_report.pptx"):
    """Create a presentation with data tables from CSV."""

    # Load data
    df = pd.read_csv(csv_file)

    prs = Presentation()

    # Title slide
    title_slide = prs.slides.add_slide(prs.slide_layouts[0])
    title_slide.shapes.title.text = "Data Report"
    title_slide.placeholders[1].text = f"Dataset: {csv_file}\n{len(df)} records"

    # Summary slide
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    title = slide.shapes.title
    title.text = "Dataset Overview"

    body = slide.placeholders[1]
    tf = body.text_frame
    tf.clear()

    summary_items = [
        f"Total Records: {len(df)}",
        f"Columns: {len(df.columns)}",
        f"Date Range: {df.iloc[0, 0]} to {df.iloc[-1, 0]}"  # Assumes first col is date
    ]

    for item in summary_items:
        p = tf.add_paragraph()
        p.text = item
        p.font.size = Pt(20)

    # Data table (first 10 rows)
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    title = slide.shapes.title
    title.text = "Sample Data (First 10 Rows)"

    # Determine table size
    rows = min(11, len(df) + 1)  # +1 for header, max 10 data rows
    cols = min(len(df.columns), 6)  # Max 6 columns

    # Add table
    left = Inches(0.5)
    top = Inches(2)
    width = Inches(9)
    height = Inches(4)

    table = slide.shapes.add_table(rows, cols, left, top, width, height).table

    # Set column widths
    col_width = width / cols
    for col in range(cols):
        table.columns[col].width = col_width

    # Headers
    for col_idx in range(cols):
        cell = table.cell(0, col_idx)
        cell.text = str(df.columns[col_idx])
        cell.text_frame.paragraphs[0].font.bold = True
        cell.text_frame.paragraphs[0].font.size = Pt(11)
        cell.fill.solid()
        cell.fill.fore_color.rgb = RGBColor(0, 51, 102)
        cell.text_frame.paragraphs[0].font.color.rgb = RGBColor(255, 255, 255)

    # Data
    for row_idx in range(min(10, len(df))):
        for col_idx in range(cols):
            cell = table.cell(row_idx + 1, col_idx)
            cell.text = str(df.iloc[row_idx, col_idx])
            cell.text_frame.paragraphs[0].font.size = Pt(10)

    prs.save(output_file)
    print(f"✅ Report created: {output_file}")

# Usage
# create_data_report('sales_data.csv')
```

## Design Best Practices

### 1. Color Palette Selection

**Rule of 60-30-10:**
- 60% primary color (backgrounds, large areas)
- 30% secondary color (supporting elements)
- 10% accent color (highlights, call-to-action)

**Professional Palettes:**

```python
# Corporate Blue
CORPORATE = {
    'primary': RGBColor(0, 51, 102),      # Navy
    'secondary': RGBColor(240, 243, 245), # Light Gray
    'accent': RGBColor(0, 153, 204)       # Sky Blue
}

# Tech Green
TECH = {
    'primary': RGBColor(34, 139, 34),     # Forest Green
    'secondary': RGBColor(245, 245, 245), # Off-white
    'accent': RGBColor(255, 165, 0)       # Orange
}

# Creative Purple
CREATIVE = {
    'primary': RGBColor(106, 27, 154),    # Deep Purple
    'secondary': RGBColor(238, 238, 238), # Light Gray
    'accent': RGBColor(255, 215, 0)       # Gold
}
```

**Accessibility:**
- Ensure sufficient contrast (WCAG AA: 4.5:1 for text)
- Avoid red-green combinations (colorblind-friendly)
- Test with grayscale preview

### 2. Typography Guidelines

**Font Hierarchy:**
```python
# Title
title_font_size = Pt(44)
title_font_bold = True

# Heading
heading_font_size = Pt(32)
heading_font_bold = True

# Subheading
subheading_font_size = Pt(24)
subheading_font_bold = False

# Body
body_font_size = Pt(18)
body_font_bold = False

# Caption
caption_font_size = Pt(12)
caption_font_bold = False
```

**Font Recommendations:**
- **Sans-serif**: Calibri, Arial, Helvetica, Segoe UI (screens)
- **Serif**: Georgia, Times New Roman (formal documents)
- **Limit to 2 fonts**: One for headings, one for body

**Readability:**
- Minimum font size: 18pt for body text
- Line spacing: 1.2-1.5x font size
- Character spacing: Normal (avoid tight tracking)

### 3. Layout Principles

**Rule of Thirds:**
```python
# Divide slide into 9 equal sections (3x3 grid)
slide_width = prs.slide_width
slide_height = prs.slide_height

third_width = slide_width / 3
third_height = slide_height / 3

# Place important elements at intersections
focal_points = [
    (third_width, third_height),      # Top-left
    (2 * third_width, third_height),  # Top-right
    (third_width, 2 * third_height),  # Bottom-left
    (2 * third_width, 2 * third_height)  # Bottom-right
]
```

**White Space:**
- Minimum margins: 0.5 inches on all sides
- Space between elements: 0.25-0.5 inches
- Don't overcrowd slides (5-7 elements maximum)

**Alignment:**
```python
# Align to grid
grid_size = Inches(0.25)

def snap_to_grid(value, grid_size):
    """Snap position to grid."""
    return round(value / grid_size) * grid_size

left = snap_to_grid(Inches(1.3), grid_size)
top = snap_to_grid(Inches(2.1), grid_size)
```

### 4. Visual Hierarchy

**Size & Scale:**
```python
# Most important element: Largest
title.text_frame.paragraphs[0].font.size = Pt(44)

# Secondary elements: Medium
heading.text_frame.paragraphs[0].font.size = Pt(28)

# Supporting details: Smallest
body.text_frame.paragraphs[0].font.size = Pt(18)
```

**Color Contrast:**
```python
# High contrast = high importance
important_text.font.color.rgb = RGBColor(0, 0, 0)  # Black on white

# Low contrast = low importance
caption.font.color.rgb = RGBColor(128, 128, 128)  # Gray on white
```

**Z-Pattern Layout:**
- Top-left: Logo/branding
- Top-right: Navigation/page number
- Middle: Main content
- Bottom-right: Call-to-action

### 5. Chart Design

**Best Practices:**
- Choose appropriate chart type (bar for comparison, line for trends, pie for parts of whole)
- Limit colors (3-5 maximum)
- Always label axes
- Include data labels for clarity
- Use gridlines sparingly

```python
# Clean chart formatting
chart.has_legend = True
chart.legend.position = XL_LEGEND_POSITION.RIGHT
chart.legend.font.size = Pt(12)

# Axis formatting
value_axis = chart.value_axis
value_axis.has_major_gridlines = True
value_axis.major_gridlines.format.line.color.rgb = RGBColor(200, 200, 200)
value_axis.tick_labels.font.size = Pt(11)

# Data labels
plot = chart.plots[0]
plot.has_data_labels = True
data_labels = plot.data_labels
data_labels.font.size = Pt(10)
data_labels.font.bold = True
```

### 6. Image Best Practices

**Resolution:**
- Screen presentations: 1920x1080 (1080p)
- Print presentations: 300 DPI minimum
- Photos: JPEG (smaller file size)
- Graphics/logos: PNG (transparency support)

**Optimization:**
```python
from PIL import Image

def optimize_image(input_path, output_path, max_size=(1920, 1080), quality=85):
    """Optimize image for presentation."""
    with Image.open(input_path) as img:
        # Resize if larger than max_size
        img.thumbnail(max_size, Image.Resampling.LANCZOS)

        # Save with compression
        img.save(output_path, optimize=True, quality=quality)

    return output_path

# Usage
optimized = optimize_image('large_photo.jpg', 'optimized.jpg')
pic = slide.shapes.add_picture(optimized, Inches(1), Inches(1))
```

**Aspect Ratios:**
- 16:9 (widescreen): Standard for modern presentations
- 4:3 (standard): Legacy format
- Match slide aspect ratio to avoid black bars

## Common Pitfalls

### 1. Layout Compatibility Issues

**Problem:** Layouts from templates don't match expected placeholders.

**Solution:**
```python
# Always check available placeholders
for shape in slide.placeholders:
    print(f"{shape.placeholder_format.idx} - {shape.name}")

# Use try-except when accessing placeholders
try:
    body = slide.placeholders[1]
except KeyError:
    # Placeholder doesn't exist, create text box instead
    body = slide.shapes.add_textbox(Inches(1), Inches(2), Inches(8), Inches(5))
```

### 2. Image Resolution Problems

**Problem:** Images appear pixelated or blurry.

**Solution:**
```python
from PIL import Image

# Check image DPI before adding
with Image.open('photo.jpg') as img:
    dpi = img.info.get('dpi', (72, 72))
    print(f"Image DPI: {dpi}")

    if dpi[0] < 150:
        print("⚠️  Warning: Low resolution image")
        # Resize or replace with higher quality version

# Calculate appropriate size
img_width_inches = img.width / dpi[0]
img_height_inches = img.height / dpi[1]

print(f"Image will be {img_width_inches:.2f}\" x {img_height_inches:.2f}\" at native resolution")
```

### 3. Font Embedding Issues

**Problem:** Fonts not displaying correctly on other computers.

**Solution:**
```python
# Use standard fonts that are widely available
SAFE_FONTS = [
    'Arial',
    'Calibri',
    'Georgia',
    'Times New Roman',
    'Verdana',
    'Tahoma'
]

# Or embed fonts (requires manual action in PowerPoint)
# File > Options > Save > "Embed fonts in the file"
```

**Manual Font Embedding:**
1. Open presentation in PowerPoint
2. File → Options → Save
3. Check "Embed fonts in the file"
4. Select "Embed all characters"

### 4. Chart Data Formatting

**Problem:** Chart data doesn't display as expected.

**Solution:**
```python
from pptx.chart.data import CategoryChartData

# Always validate data before creating chart
chart_data = CategoryChartData()

# Categories must be strings
categories = ['Q1', 'Q2', 'Q3', 'Q4']  # ✅ Good
# categories = [1, 2, 3, 4]  # ❌ Bad (numbers)

chart_data.categories = categories

# Series data must be numbers
values = [10, 20, 15, 25]  # ✅ Good
# values = ['10', '20', '15', '25']  # ❌ Bad (strings)

chart_data.add_series('Sales', values)

# Handle missing data
values_with_none = [10, 20, None, 25]  # None for missing
chart_data.add_series('Sales', values_with_none)
```

### 5. Text Overflow

**Problem:** Text doesn't fit in text boxes or placeholders.

**Solution:**
```python
from pptx.enum.text import MSO_AUTO_SIZE

# Enable auto-fit
text_frame = shape.text_frame
text_frame.auto_size = MSO_AUTO_SIZE.TEXT_TO_FIT_SHAPE  # Shrink text
# or
text_frame.auto_size = MSO_AUTO_SIZE.SHAPE_TO_FIT_TEXT  # Expand shape

# Check if text fits
text_frame.word_wrap = True

# Truncate long text
max_chars = 500
if len(long_text) > max_chars:
    display_text = long_text[:max_chars] + "..."
else:
    display_text = long_text
```

### 6. File Size Issues

**Problem:** Presentation file is too large.

**Solution:**
```python
import os

# Compress images before adding
from PIL import Image

def compress_image(img_path, max_size_mb=1):
    """Compress image to target file size."""
    img = Image.open(img_path)

    quality = 95
    while quality > 10:
        output = f"compressed_{os.path.basename(img_path)}"
        img.save(output, optimize=True, quality=quality)

        size_mb = os.path.getsize(output) / (1024 * 1024)
        if size_mb <= max_size_mb:
            return output

        quality -= 5

    return output

# Use compressed images
compressed = compress_image('large_photo.jpg')
slide.shapes.add_picture(compressed, Inches(1), Inches(1))
```

**Manual Compression:**
1. Open presentation in PowerPoint
2. File → Compress Pictures
3. Select resolution (220 ppi for print, 150 ppi for screen)
4. Check "Delete cropped areas of pictures"

### 7. Position and Size Calculations

**Problem:** Elements not positioned correctly.

**Solution:**
```python
from pptx.util import Inches, Pt, Cm

# Use consistent units
left = Inches(1)      # Not: left = 914400 (EMUs)
top = Inches(2)
width = Inches(8)
height = Inches(4)

# Center element horizontally
element_width = Inches(5)
slide_width = prs.slide_width
left = (slide_width - element_width) / 2

# Center element vertically
element_height = Inches(3)
slide_height = prs.slide_height
top = (slide_height - element_height) / 2

# Align multiple elements
spacing = Inches(0.5)
top = Inches(2)

for i, item in enumerate(items):
    shape = slide.shapes.add_textbox(Inches(1), top, Inches(8), Inches(0.5))
    shape.text_frame.text = item
    top += Inches(0.5) + spacing  # Move down for next item
```

## Advanced Techniques

### Custom Slide Layouts

```python
from pptx import Presentation
from pptx.util import Inches

# Start with blank presentation
prs = Presentation()

# Access slide master
slide_master = prs.slide_master

# Create custom layout (requires XML manipulation)
# Note: python-pptx has limited support for creating new layouts
# Recommended: Create template in PowerPoint, then use in python-pptx
```

### Speaker Notes

```python
slide = prs.slides.add_slide(prs.slide_layouts[1])

# Add speaker notes
notes_slide = slide.notes_slide
text_frame = notes_slide.notes_text_frame

text_frame.text = "Key talking points:\n"
text_frame.text += "- Emphasize 35% revenue growth\n"
text_frame.text += "- Mention customer testimonials\n"
text_frame.text += "- Time: 2 minutes"
```

### Hyperlinks

```python
# Add hyperlink to text
text_frame = shape.text_frame
p = text_frame.paragraphs[0]
run = p.add_run()
run.text = "Click here for more info"
run.hyperlink.address = "https://example.com"

# Link to another slide
run.hyperlink.address = f"slide{slide_number}"
```

### Animations (Limited Support)

```python
# python-pptx has limited animation support
# Recommended: Apply animations in PowerPoint after generation
# Or use VBA/COM for Windows automation
```

## Helper Script Reference

The `scripts/pptx_helper.py` module provides convenient utility functions. See the helper script for:

- `create_presentation()`: Initialize new presentation with defaults
- `add_title_slide()`: Add formatted title slide
- `add_bullet_slide()`: Add slide with bullet points
- `add_image_slide()`: Add slide with centered image
- `add_chart_slide()`: Add slide with chart
- `add_table_slide()`: Add slide with formatted table
- `apply_brand_colors()`: Apply consistent color scheme
- `optimize_images()`: Batch optimize images for presentation

**Usage Example:**
```python
from scripts.pptx_helper import create_presentation, add_title_slide, add_chart_slide

prs = create_presentation(title="My Presentation")
add_title_slide(prs, "Main Title", "Subtitle")
add_chart_slide(prs, "Sales Data", chart_type='bar',
                categories=['Q1', 'Q2', 'Q3', 'Q4'],
                values=[10, 20, 15, 25])
prs.save('output.pptx')
```

## Resources

### Documentation
- python-pptx: https://python-pptx.readthedocs.io/
- API Reference: https://python-pptx.readthedocs.io/en/latest/api/
- GitHub: https://github.com/scanny/python-pptx

### Tutorials
- Getting Started: https://python-pptx.readthedocs.io/en/latest/user/quickstart.html
- Working with Shapes: https://python-pptx.readthedocs.io/en/latest/user/shapes.html
- Charts: https://python-pptx.readthedocs.io/en/latest/user/charts.html

### Design Resources
- Microsoft Design Templates: https://templates.office.com/powerpoint
- Color Palette Tools: Coolors.co, Adobe Color
- Free Stock Images: Unsplash, Pexels

### Tools
- PowerPoint Online: Edit and test presentations
- LibreOffice Impress: Open-source alternative
- Google Slides: Cloud-based editing

## Troubleshooting

**Issue:** "ModuleNotFoundError: No module named 'pptx'"
```bash
pip install python-pptx
```

**Issue:** "AttributeError: 'NoneType' object has no attribute..."
- Check placeholder indices: `print([p.placeholder_format.idx for p in slide.placeholders])`
- Verify layout has expected placeholders

**Issue:** Charts not displaying correctly
- Ensure chart data types are correct (strings for categories, numbers for values)
- Check chart type compatibility with data

**Issue:** Images not found
- Use absolute paths: `os.path.abspath('image.png')`
- Verify file exists: `os.path.exists(img_path)`

**Issue:** File corrupted after generation
- Validate presentation: Open in PowerPoint and check for errors
- Check for invalid characters in text
- Ensure all shapes are properly closed

## Best Practices Summary

1. **Always use templates** for consistent branding
2. **Optimize images** before adding to presentation
3. **Limit text** on each slide (5-7 bullet points max)
4. **Use high contrast** for readability
5. **Test on target device** before presenting
6. **Keep file size manageable** (<20MB for email)
7. **Use speaker notes** for detailed talking points
8. **Follow 6x6 rule**: Max 6 bullets, max 6 words per bullet
9. **Validate data** before creating charts
10. **Use consistent spacing** and alignment

---

**When to Use This Skill:**
- Creating business presentations from data
- Automating report generation
- Bulk slide creation from databases
- Template-based presentations
- Educational content with charts/images
- Portfolio or showcase presentations
- Converting documents to slides

**Integration:**
- Works well with `pandas` for data analysis
- Combine with `matplotlib` for custom charts
- Use with `Pillow` for image processing
- Integrate with `jinja2` for template rendering
- Compatible with `openpyxl` for Excel data import
