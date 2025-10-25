---
name: xlsx
description: "Comprehensive Excel spreadsheet creation, editing, and analysis with support for formulas, formatting, charts, data analysis, and visualization. Use when working with .xlsx, .xlsm, .csv files for: (1) Creating spreadsheets with formulas and formatting, (2) Reading/analyzing data, (3) Modifying existing spreadsheets while preserving formulas, (4) Creating charts and visualizations, (5) Data transformation and analysis, (6) Multi-worksheet operations"
---

# Excel (XLSX) Skill

## Overview

This skill provides comprehensive capabilities for working with Excel spreadsheets programmatically using Python. It covers everything from basic file operations to advanced data analysis, formula management, chart creation, and formatting.

## Core Capabilities

### 1. File Operations
- **Reading**: Load .xlsx, .xlsm, and .csv files
- **Writing**: Create new Excel workbooks from scratch
- **Editing**: Modify existing workbooks while preserving formulas, formatting, and charts
- **Converting**: Transform between CSV, Excel, and other formats

### 2. Data Management
- **Cell Operations**: Read, write, and modify individual cells or ranges
- **Formulas**: Create and manage Excel formulas (SUM, VLOOKUP, INDEX/MATCH, etc.)
- **Data Validation**: Set dropdown lists, numeric ranges, date constraints
- **Named Ranges**: Define and use named cell ranges for easier formula management

### 3. Formatting
- **Cell Styling**: Fonts, colors, borders, alignment, number formats
- **Conditional Formatting**: Apply rules-based formatting
- **Row/Column Sizing**: Set widths, heights, auto-fit
- **Merge Cells**: Combine cells for headers and labels

### 4. Charts & Visualizations
- **Chart Types**: Line, bar, column, pie, scatter, area, combo charts
- **Chart Customization**: Titles, legends, data labels, colors
- **Multiple Series**: Multi-dataset charts with secondary axes
- **Chart Positioning**: Place charts in specific locations

### 5. Multi-Worksheet Operations
- **Sheet Management**: Create, rename, delete, reorder worksheets
- **Cross-Sheet Formulas**: Reference data across multiple sheets
- **Sheet Copying**: Duplicate sheets with formatting intact
- **Sheet Protection**: Lock/unlock sheets and ranges

### 6. Data Analysis
- **Filtering**: Auto-filter data ranges
- **Sorting**: Multi-level sorting
- **Pivot Tables**: Programmatic pivot table creation
- **Statistical Functions**: Built-in and custom calculations

## Python Libraries

### Primary: openpyxl
The main library for reading and writing Excel 2010 xlsx/xlsm files.

```bash
# Installation
pip install openpyxl

# Or with uv
uv pip install openpyxl
```

**Key Features:**
- Full read/write support for .xlsx files
- Formula preservation and creation
- Chart creation and editing
- Formatting and styling
- Multiple worksheet support

```python
# Basic imports
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Font, PatternFill, Border, Side, Alignment
from openpyxl.chart import LineChart, BarChart, PieChart, Reference
from openpyxl.utils import get_column_letter
```

### Secondary: pandas
Excellent for data analysis and CSV operations.

```bash
pip install pandas openpyxl
```

**Key Features:**
- High-performance data manipulation
- Easy CSV to Excel conversion
- DataFrame to Excel export
- Excel to DataFrame import with formulas evaluated

```python
import pandas as pd

# Read Excel file into DataFrame
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# Write DataFrame to Excel
df.to_excel('output.xlsx', sheet_name='Results', index=False)
```

### Alternative: xlsxwriter
Focused on writing Excel files with rich formatting.

```bash
pip install xlsxwriter
```

**Note**: xlsxwriter cannot read or edit existing files, only create new ones.

```python
import xlsxwriter

# Create workbook
workbook = xlsxwriter.Workbook('output.xlsx')
worksheet = workbook.add_worksheet()

# Write with formatting
bold = workbook.add_format({'bold': True})
worksheet.write('A1', 'Hello', bold)
workbook.close()
```

## Detailed Workflows

### Workflow 1: Creating a New Workbook from Scratch

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

# Create new workbook
wb = Workbook()
ws = wb.active
ws.title = "Sales Report"

# Add headers with formatting
headers = ["Product", "Q1", "Q2", "Q3", "Q4", "Total"]
header_fill = PatternFill(start_color="4472C4", end_color="4472C4", fill_type="solid")
header_font = Font(color="FFFFFF", bold=True)

for col, header in enumerate(headers, start=1):
    cell = ws.cell(row=1, column=col, value=header)
    cell.fill = header_fill
    cell.font = header_font
    cell.alignment = Alignment(horizontal="center")

# Add data
data = [
    ["Product A", 1000, 1200, 1100, 1300],
    ["Product B", 800, 900, 950, 1000],
    ["Product C", 1500, 1400, 1600, 1700]
]

for row_idx, row_data in enumerate(data, start=2):
    for col_idx, value in enumerate(row_data, start=1):
        ws.cell(row=row_idx, column=col_idx, value=value)

# Add formulas for totals
for row in range(2, len(data) + 2):
    formula = f"=SUM(B{row}:E{row})"
    ws.cell(row=row, column=6, value=formula)

# Adjust column widths
for col in range(1, 7):
    ws.column_dimensions[get_column_letter(col)].width = 12

# Save workbook
wb.save("sales_report.xlsx")
print("Workbook created successfully!")
```

### Workflow 2: Reading and Analyzing Existing Workbooks

```python
from openpyxl import load_workbook
from openpyxl.utils import get_column_letter

# Load existing workbook
wb = load_workbook('data.xlsx', data_only=True)  # data_only=True evaluates formulas
ws = wb.active

# Method 1: Iterate through all rows
print("All data:")
for row in ws.iter_rows(min_row=2, values_only=True):
    print(row)

# Method 2: Read specific cells
print(f"\nCell A1 value: {ws['A1'].value}")
print(f"Cell B2 value: {ws.cell(row=2, column=2).value}")

# Method 3: Read entire column
print("\nColumn A values:")
for cell in ws['A']:
    print(cell.value)

# Method 4: Read range
print("\nRange B2:D5:")
for row in ws['B2':'D5']:
    for cell in row:
        print(cell.value, end=' ')
    print()

# Get dimensions
print(f"\nWorksheet dimensions: {ws.dimensions}")
print(f"Max row: {ws.max_row}, Max column: {ws.max_column}")

# Calculate statistics
values = [cell.value for cell in ws['B'][1:] if isinstance(cell.value, (int, float))]
if values:
    print(f"\nColumn B statistics:")
    print(f"Sum: {sum(values)}")
    print(f"Average: {sum(values) / len(values):.2f}")
    print(f"Min: {min(values)}, Max: {max(values)}")

wb.close()
```

### Workflow 3: Editing Workbooks While Preserving Formulas

```python
from openpyxl import load_workbook
from openpyxl.styles import Font, PatternFill

# Load workbook WITHOUT data_only to preserve formulas
wb = load_workbook('existing_report.xlsx')
ws = wb['Sales']

# Update values (formulas will recalculate when opened in Excel)
ws['B2'] = 1500  # Update Q1 sales for Product A
ws['C2'] = 1650  # Update Q2 sales for Product A

# Add new row with data and formulas
new_row = ws.max_row + 1
ws[f'A{new_row}'] = "Product D"
ws[f'B{new_row}'] = 900
ws[f'C{new_row}'] = 1000
ws[f'D{new_row}'] = 1100
ws[f'E{new_row}'] = 1200
ws[f'F{new_row}'] = f"=SUM(B{new_row}:E{new_row})"  # Add formula

# Apply formatting to new row
for col in range(1, 7):
    cell = ws.cell(row=new_row, column=col)
    if col == 1:
        cell.font = Font(bold=True)
    elif col == 6:
        cell.fill = PatternFill(start_color="E7E6E6", end_color="E7E6E6", fill_type="solid")

# Add a summary row
summary_row = new_row + 1
ws[f'A{summary_row}'] = "Grand Total"
ws[f'A{summary_row}'].font = Font(bold=True, size=12)

# Create formula that sums all quarterly totals
ws[f'F{summary_row}'] = f"=SUM(F2:F{new_row})"
ws[f'F{summary_row}'].font = Font(bold=True, size=12)

# Save changes
wb.save('existing_report.xlsx')
print("Workbook updated successfully with formulas preserved!")
```

### Workflow 4: Creating Charts

```python
from openpyxl import Workbook
from openpyxl.chart import LineChart, BarChart, PieChart, Reference
from openpyxl.chart.series import DataPoint

# Create workbook with sample data
wb = Workbook()
ws = wb.active
ws.title = "Sales Data"

# Add data
ws.append(["Month", "Product A", "Product B", "Product C"])
months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
sales_a = [120, 135, 150, 140, 160, 175]
sales_b = [80, 90, 85, 95, 100, 110]
sales_c = [150, 145, 160, 155, 170, 180]

for i, month in enumerate(months):
    ws.append([month, sales_a[i], sales_b[i], sales_c[i]])

# Create Line Chart
line_chart = LineChart()
line_chart.title = "Monthly Sales Trend"
line_chart.style = 10
line_chart.y_axis.title = "Sales ($)"
line_chart.x_axis.title = "Month"

# Define data range
data = Reference(ws, min_col=2, min_row=1, max_col=4, max_row=7)
categories = Reference(ws, min_col=1, min_row=2, max_row=7)

line_chart.add_data(data, titles_from_data=True)
line_chart.set_categories(categories)

# Add chart to worksheet
ws.add_chart(line_chart, "F2")

# Create Bar Chart on same sheet
bar_chart = BarChart()
bar_chart.type = "col"
bar_chart.title = "Sales Comparison"
bar_chart.y_axis.title = "Sales ($)"
bar_chart.x_axis.title = "Month"

bar_chart.add_data(data, titles_from_data=True)
bar_chart.set_categories(categories)

ws.add_chart(bar_chart, "F18")

# Create Pie Chart (last month data)
pie_chart = PieChart()
pie_chart.title = "June Sales Distribution"

# Use only June data (last row)
pie_data = Reference(ws, min_col=2, min_row=7, max_col=4, max_row=7)
pie_categories = Reference(ws, min_col=2, min_row=1, max_col=4, max_row=1)

pie_chart.add_data(pie_data)
pie_chart.set_categories(pie_categories)

ws.add_chart(pie_chart, "N2")

wb.save("sales_charts.xlsx")
print("Charts created successfully!")
```

### Workflow 5: Applying Conditional Formatting

```python
from openpyxl import Workbook
from openpyxl.styles import PatternFill
from openpyxl.formatting.rule import ColorScaleRule, CellIsRule, IconSetRule

wb = Workbook()
ws = wb.active

# Add sample data
ws.append(["Product", "Sales", "Target", "Achievement %"])
products_data = [
    ["Product A", 1200, 1000, 120],
    ["Product B", 850, 1000, 85],
    ["Product C", 1500, 1200, 125],
    ["Product D", 950, 1000, 95],
    ["Product E", 800, 1000, 80],
]

for row in products_data:
    ws.append(row)

# Conditional Formatting 1: Color Scale (Sales column)
color_scale = ColorScaleRule(
    start_type='min',
    start_color='F8696B',  # Red for low values
    mid_type='percentile',
    mid_value=50,
    mid_color='FFEB84',    # Yellow for medium
    end_type='max',
    end_color='63BE7B'     # Green for high values
)
ws.conditional_formatting.add('B2:B6', color_scale)

# Conditional Formatting 2: Highlight cells above target (Achievement % > 100)
green_fill = PatternFill(start_color='C6EFCE', end_color='C6EFCE', fill_type='solid')
above_target = CellIsRule(
    operator='greaterThan',
    formula=['100'],
    fill=green_fill
)
ws.conditional_formatting.add('D2:D6', above_target)

# Conditional Formatting 3: Highlight cells below 90% achievement
red_fill = PatternFill(start_color='FFC7CE', end_color='FFC7CE', fill_type='solid')
below_target = CellIsRule(
    operator='lessThan',
    formula=['90'],
    fill=red_fill
)
ws.conditional_formatting.add('D2:D6', below_target)

# Conditional Formatting 4: Icon Sets
icon_set = IconSetRule('3Arrows', 'num', [0, 90, 100])
ws.conditional_formatting.add('D2:D6', icon_set)

wb.save('conditional_formatting.xlsx')
print("Conditional formatting applied!")
```

### Workflow 6: Data Validation

```python
from openpyxl import Workbook
from openpyxl.data_validation import DataValidation

wb = Workbook()
ws = wb.active

# Setup headers
ws.append(["Employee", "Department", "Status", "Rating", "Start Date"])

# Data Validation 1: Dropdown list for Department
dept_validation = DataValidation(
    type="list",
    formula1='"Sales,Marketing,Engineering,HR,Finance"',
    allow_blank=False
)
dept_validation.error = 'Please select a department from the list'
dept_validation.errorTitle = 'Invalid Department'
dept_validation.prompt = 'Select a department'
dept_validation.promptTitle = 'Department Selection'

ws.add_data_validation(dept_validation)
dept_validation.add('B2:B100')  # Apply to column B

# Data Validation 2: Dropdown list for Status
status_validation = DataValidation(
    type="list",
    formula1='"Active,On Leave,Terminated"',
    allow_blank=False
)
status_validation.error = 'Please select a valid status'
status_validation.errorTitle = 'Invalid Status'

ws.add_data_validation(status_validation)
status_validation.add('C2:C100')

# Data Validation 3: Numeric range for Rating (1-5)
rating_validation = DataValidation(
    type="whole",
    operator="between",
    formula1=1,
    formula2=5,
    allow_blank=True
)
rating_validation.error = 'Rating must be between 1 and 5'
rating_validation.errorTitle = 'Invalid Rating'
rating_validation.prompt = 'Enter a rating from 1 to 5'
rating_validation.promptTitle = 'Rating'

ws.add_data_validation(rating_validation)
rating_validation.add('D2:D100')

# Data Validation 4: Date validation (past dates only)
date_validation = DataValidation(
    type="date",
    operator="lessThanOrEqual",
    formula1="TODAY()",
    allow_blank=False
)
date_validation.error = 'Start date cannot be in the future'
date_validation.errorTitle = 'Invalid Date'

ws.add_data_validation(date_validation)
date_validation.add('E2:E100')

# Add sample data
ws.append(["John Doe", "Sales", "Active", 4, "2023-01-15"])
ws.append(["Jane Smith", "Engineering", "Active", 5, "2022-06-01"])

wb.save('data_validation.xlsx')
print("Data validation rules applied!")
```

### Workflow 7: Working with Multiple Worksheets

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill
from copy import copy

wb = Workbook()

# Remove default sheet
default_sheet = wb.active
wb.remove(default_sheet)

# Create multiple sheets
sheet_names = ["Q1 Sales", "Q2 Sales", "Q3 Sales", "Q4 Sales", "Annual Summary"]
for name in sheet_names:
    wb.create_sheet(title=name)

# Add data to each quarterly sheet
quarters = ["Q1 Sales", "Q2 Sales", "Q3 Sales", "Q4 Sales"]
quarter_data = [
    [15000, 12000, 18000],  # Q1
    [16000, 13000, 19000],  # Q2
    [15500, 12500, 18500],  # Q3
    [17000, 14000, 20000],  # Q4
]

for i, quarter in enumerate(quarters):
    ws = wb[quarter]

    # Headers
    ws.append(["Product", "Sales"])
    ws['A1'].font = Font(bold=True)
    ws['B1'].font = Font(bold=True)

    # Data
    products = ["Product A", "Product B", "Product C"]
    for j, product in enumerate(products):
        ws.append([product, quarter_data[i][j]])

# Create summary sheet with cross-sheet formulas
summary = wb["Annual Summary"]
summary.append(["Product", "Q1", "Q2", "Q3", "Q4", "Annual Total"])

# Format header
header_fill = PatternFill(start_color="4472C4", end_color="4472C4", fill_type="solid")
header_font = Font(color="FFFFFF", bold=True)
for cell in summary[1]:
    cell.fill = header_fill
    cell.font = header_font

products = ["Product A", "Product B", "Product C"]
for i, product in enumerate(products, start=2):
    summary[f'A{i}'] = product

    # Cross-sheet formulas
    summary[f'B{i}'] = f"='Q1 Sales'!B{i}"
    summary[f'C{i}'] = f"='Q2 Sales'!B{i}"
    summary[f'D{i}'] = f"='Q3 Sales'!B{i}"
    summary[f'E{i}'] = f"='Q4 Sales'!B{i}"
    summary[f'F{i}'] = f"=SUM(B{i}:E{i})"

# Add grand total row
summary['A5'] = "Grand Total"
summary['A5'].font = Font(bold=True)
for col in ['B', 'C', 'D', 'E', 'F']:
    summary[f'{col}5'] = f"=SUM({col}2:{col}4)"
    summary[f'{col}5'].font = Font(bold=True)

# Adjust column widths
for sheet_name in wb.sheetnames:
    ws = wb[sheet_name]
    for col in range(1, ws.max_column + 1):
        ws.column_dimensions[get_column_letter(col)].width = 15

wb.save('multi_sheet_workbook.xlsx')
print("Multi-sheet workbook created!")
```

### Workflow 8: Advanced Data Analysis with Pandas Integration

```python
import pandas as pd
from openpyxl import load_workbook
from openpyxl.styles import Font, PatternFill
from openpyxl.chart import BarChart, Reference

# Step 1: Read and analyze data with pandas
df = pd.read_excel('sales_data.xlsx')

# Perform analysis
summary = df.groupby('Product').agg({
    'Sales': ['sum', 'mean', 'count'],
    'Profit': 'sum'
}).round(2)

summary.columns = ['Total Sales', 'Avg Sales', 'Transactions', 'Total Profit']
summary['Profit Margin %'] = ((summary['Total Profit'] / summary['Total Sales']) * 100).round(2)

# Calculate top performers
top_products = summary.nlargest(5, 'Total Sales')

# Step 2: Write results to new Excel file
with pd.ExcelWriter('sales_analysis.xlsx', engine='openpyxl') as writer:
    # Write multiple sheets
    df.to_excel(writer, sheet_name='Raw Data', index=False)
    summary.to_excel(writer, sheet_name='Summary')
    top_products.to_excel(writer, sheet_name='Top 5 Products')

# Step 3: Enhance with openpyxl formatting
wb = load_workbook('sales_analysis.xlsx')

# Format Summary sheet
ws = wb['Summary']
header_fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
header_font = Font(color="FFFFFF", bold=True)

for cell in ws[1]:
    cell.fill = header_fill
    cell.font = header_font

# Add chart to Summary sheet
chart = BarChart()
chart.title = "Total Sales by Product"
chart.y_axis.title = "Sales ($)"

data = Reference(ws, min_col=2, min_row=1, max_row=ws.max_row)
categories = Reference(ws, min_col=1, min_row=2, max_row=ws.max_row)

chart.add_data(data, titles_from_data=True)
chart.set_categories(categories)

ws.add_chart(chart, "H2")

wb.save('sales_analysis.xlsx')
print("Analysis complete! Results saved to sales_analysis.xlsx")
```

## Common Use Cases with Complete Examples

### Use Case 1: Financial Report with Formulas

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Border, Side, Alignment, numbers
from openpyxl.utils import get_column_letter

def create_financial_report():
    wb = Workbook()
    ws = wb.active
    ws.title = "Income Statement"

    # Title
    ws.merge_cells('A1:D1')
    ws['A1'] = "COMPANY NAME - Income Statement"
    ws['A1'].font = Font(size=14, bold=True)
    ws['A1'].alignment = Alignment(horizontal='center')

    # Period headers
    ws['A2'] = "Account"
    ws['B2'] = "Q1"
    ws['C2'] = "Q2"
    ws['D2'] = "Total"

    # Format headers
    header_font = Font(bold=True)
    header_fill = PatternFill(start_color="D9E1F2", end_color="D9E1F2", fill_type="solid")
    for cell in ws[2]:
        cell.font = header_font
        cell.fill = header_fill

    # Revenue section
    ws['A3'] = "Revenue"
    ws['A3'].font = Font(bold=True)

    revenue_items = [
        ["Product Sales", 50000, 55000],
        ["Service Revenue", 30000, 35000],
        ["Other Income", 5000, 7000]
    ]

    row = 4
    for item in revenue_items:
        ws[f'A{row}'] = item[0]
        ws[f'B{row}'] = item[1]
        ws[f'C{row}'] = item[2]
        ws[f'D{row}'] = f"=SUM(B{row}:C{row})"
        row += 1

    # Total Revenue
    ws[f'A{row}'] = "Total Revenue"
    ws[f'A{row}'].font = Font(bold=True)
    ws[f'B{row}'] = f"=SUM(B4:B{row-1})"
    ws[f'C{row}'] = f"=SUM(C4:C{row-1})"
    ws[f'D{row}'] = f"=SUM(D4:D{row-1})"
    total_revenue_row = row
    row += 1

    # Expenses section
    ws[f'A{row}'] = "Expenses"
    ws[f'A{row}'].font = Font(bold=True)
    row += 1

    expense_items = [
        ["Salaries", 25000, 26000],
        ["Rent", 8000, 8000],
        ["Marketing", 5000, 7000],
        ["Utilities", 2000, 2200],
        ["Supplies", 3000, 3500]
    ]

    expense_start = row
    for item in expense_items:
        ws[f'A{row}'] = item[0]
        ws[f'B{row}'] = item[1]
        ws[f'C{row}'] = item[2]
        ws[f'D{row}'] = f"=SUM(B{row}:C{row})"
        row += 1

    # Total Expenses
    ws[f'A{row}'] = "Total Expenses"
    ws[f'A{row}'].font = Font(bold=True)
    ws[f'B{row}'] = f"=SUM(B{expense_start}:B{row-1})"
    ws[f'C{row}'] = f"=SUM(C{expense_start}:C{row-1})"
    ws[f'D{row}'] = f"=SUM(D{expense_start}:D{row-1})"
    total_expense_row = row
    row += 1

    # Net Income
    row += 1
    ws[f'A{row}'] = "Net Income"
    ws[f'A{row}'].font = Font(size=12, bold=True)
    ws[f'B{row}'] = f"=B{total_revenue_row}-B{total_expense_row}"
    ws[f'C{row}'] = f"=C{total_revenue_row}-C{total_expense_row}"
    ws[f'D{row}'] = f"=D{total_revenue_row}-D{total_expense_row}"

    # Format Net Income row
    net_income_fill = PatternFill(start_color="FFE699", end_color="FFE699", fill_type="solid")
    for col in ['A', 'B', 'C', 'D']:
        ws[f'{col}{row}'].fill = net_income_fill
        ws[f'{col}{row}'].font = Font(bold=True)

    # Apply currency format to all numeric cells
    for row_cells in ws[f'B3:D{row}']:
        for cell in row_cells:
            if cell.value and cell.value != "Account":
                cell.number_format = numbers.FORMAT_CURRENCY_USD_SIMPLE

    # Add borders
    thin_border = Border(
        left=Side(style='thin'),
        right=Side(style='thin'),
        top=Side(style='thin'),
        bottom=Side(style='thin')
    )

    for row in ws[f'A2:D{row}']:
        for cell in row:
            cell.border = thin_border

    # Adjust column widths
    ws.column_dimensions['A'].width = 25
    ws.column_dimensions['B'].width = 15
    ws.column_dimensions['C'].width = 15
    ws.column_dimensions['D'].width = 15

    wb.save('financial_report.xlsx')
    print("Financial report created successfully!")

create_financial_report()
```

### Use Case 2: Data Import and Transformation

```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill
from openpyxl.chart import LineChart, Reference
from datetime import datetime

def transform_csv_to_excel():
    # Read CSV file
    df = pd.read_csv('sales_raw.csv')

    # Data cleaning and transformation
    df['Date'] = pd.to_datetime(df['Date'])
    df['Month'] = df['Date'].dt.strftime('%B %Y')
    df['Revenue'] = df['Quantity'] * df['Price']

    # Create pivot table
    pivot = df.pivot_table(
        values='Revenue',
        index='Product',
        columns='Month',
        aggfunc='sum',
        fill_value=0
    )

    # Create Excel workbook
    wb = Workbook()
    ws = wb.active
    ws.title = "Sales Summary"

    # Write pivot table to Excel
    ws.append(['Product'] + list(pivot.columns))

    for product, row_data in pivot.iterrows():
        ws.append([product] + list(row_data))

    # Format headers
    header_fill = PatternFill(start_color="4472C4", end_color="4472C4", fill_type="solid")
    header_font = Font(color="FFFFFF", bold=True)

    for cell in ws[1]:
        cell.fill = header_fill
        cell.font = header_font

    # Add totals column
    col_letter = get_column_letter(ws.max_column + 1)
    ws.cell(row=1, column=ws.max_column + 1, value="Total")
    ws[f'{col_letter}1'].fill = header_fill
    ws[f'{col_letter}1'].font = header_font

    for row in range(2, ws.max_row + 1):
        formula = f"=SUM(B{row}:{get_column_letter(ws.max_column - 1)}{row})"
        ws[f'{col_letter}{row}'] = formula

    # Add chart
    chart = LineChart()
    chart.title = "Monthly Revenue Trend"
    chart.y_axis.title = "Revenue ($)"
    chart.x_axis.title = "Month"

    data = Reference(ws, min_col=2, min_row=1, max_col=ws.max_column - 1, max_row=ws.max_row)
    categories = Reference(ws, min_col=1, min_row=2, max_row=ws.max_row)

    chart.add_data(data, titles_from_data=True)
    chart.set_categories(categories)

    ws.add_chart(chart, f"A{ws.max_row + 3}")

    wb.save('sales_transformed.xlsx')
    print("CSV transformed and saved to Excel!")

# Example usage
# transform_csv_to_excel()
```

### Use Case 3: Dynamic Dashboard with Multiple Charts

```python
from openpyxl import Workbook
from openpyxl.chart import LineChart, BarChart, PieChart, Reference
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.utils import get_column_letter

def create_dashboard():
    wb = Workbook()
    ws = wb.active
    ws.title = "Dashboard"

    # Title
    ws.merge_cells('A1:H1')
    ws['A1'] = "Sales Dashboard - 2024"
    ws['A1'].font = Font(size=18, bold=True, color="FFFFFF")
    ws['A1'].fill = PatternFill(start_color="1F4E78", end_color="1F4E78", fill_type="solid")
    ws['A1'].alignment = Alignment(horizontal='center', vertical='center')
    ws.row_dimensions[1].height = 30

    # Data section
    ws['A3'] = "Month"
    ws['B3'] = "Sales"
    ws['C3'] = "Costs"
    ws['D3'] = "Profit"

    months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
    sales = [45000, 52000, 48000, 61000, 58000, 65000]
    costs = [30000, 32000, 31000, 38000, 36000, 40000]

    for i, month in enumerate(months, start=4):
        ws[f'A{i}'] = month
        ws[f'B{i}'] = sales[i-4]
        ws[f'C{i}'] = costs[i-4]
        ws[f'D{i}'] = f"=B{i}-C{i}"

    # Summary statistics
    ws['F3'] = "Metric"
    ws['G3'] = "Value"

    metrics = [
        ["Total Sales", "=SUM(B4:B9)"],
        ["Total Costs", "=SUM(C4:C9)"],
        ["Total Profit", "=SUM(D4:D9)"],
        ["Avg Monthly Sales", "=AVERAGE(B4:B9)"],
        ["Profit Margin %", "=(G6/G4)*100"]
    ]

    for i, (metric, formula) in enumerate(metrics, start=4):
        ws[f'F{i}'] = metric
        ws[f'F{i}'].font = Font(bold=True)
        ws[f'G{i}'] = formula

    # Format header rows
    header_fill = PatternFill(start_color="4472C4", end_color="4472C4", fill_type="solid")
    header_font = Font(color="FFFFFF", bold=True)

    for cell in ws[3]:
        if cell.value:
            cell.fill = header_fill
            cell.font = header_font

    # Chart 1: Line chart for Sales and Costs
    line_chart = LineChart()
    line_chart.title = "Sales vs Costs Trend"
    line_chart.y_axis.title = "Amount ($)"
    line_chart.x_axis.title = "Month"
    line_chart.height = 10
    line_chart.width = 20

    data = Reference(ws, min_col=2, min_row=3, max_col=3, max_row=9)
    categories = Reference(ws, min_col=1, min_row=4, max_row=9)

    line_chart.add_data(data, titles_from_data=True)
    line_chart.set_categories(categories)

    ws.add_chart(line_chart, "A11")

    # Chart 2: Bar chart for Profit
    bar_chart = BarChart()
    bar_chart.type = "col"
    bar_chart.title = "Monthly Profit"
    bar_chart.y_axis.title = "Profit ($)"
    bar_chart.x_axis.title = "Month"
    bar_chart.height = 10
    bar_chart.width = 20

    profit_data = Reference(ws, min_col=4, min_row=3, max_row=9)

    bar_chart.add_data(profit_data, titles_from_data=True)
    bar_chart.set_categories(categories)

    ws.add_chart(bar_chart, "K11")

    # Chart 3: Pie chart for Sales distribution
    pie_chart = PieChart()
    pie_chart.title = "Sales Distribution by Month"
    pie_chart.height = 10
    pie_chart.width = 12

    pie_data = Reference(ws, min_col=2, min_row=4, max_row=9)
    pie_categories = Reference(ws, min_col=1, min_row=4, max_row=9)

    pie_chart.add_data(pie_data)
    pie_chart.set_categories(pie_categories)

    ws.add_chart(pie_chart, "A27")

    # Adjust column widths
    for col in range(1, 8):
        ws.column_dimensions[get_column_letter(col)].width = 14

    wb.save('dashboard.xlsx')
    print("Dashboard created successfully!")

create_dashboard()
```

## Best Practices

### 1. Formula Management

**DO:**
```python
# Use formulas for calculations
ws['D2'] = "=B2*C2"  # Quantity * Price
ws['E10'] = "=SUM(E2:E9)"  # Total

# Use named ranges for clarity
wb.define_name('TotalSales', f'{ws.title}!$E$10')
ws['G2'] = "=TotalSales*0.1"  # 10% commission
```

**DON'T:**
```python
# Don't calculate in Python when Excel can do it
total = sum([cell.value for cell in ws['E2:E9']])  # Bad
ws['E10'] = total  # This won't update if source values change
```

### 2. Performance Optimization

**DO:**
```python
# For large datasets, write rows in bulk
data = [[f'Row {i}', i*100] for i in range(1000)]
for row_data in data:
    ws.append(row_data)

# Or use pandas for very large datasets
df = pd.DataFrame(data)
df.to_excel('large_file.xlsx', index=False)
```

**DON'T:**
```python
# Don't read cell-by-cell in loops
for row in range(1, 10000):
    for col in range(1, 50):
        value = ws.cell(row, col).value  # Very slow!
```

### 3. Memory Management

**DO:**
```python
# Use read_only mode for reading large files
wb = load_workbook('large_file.xlsx', read_only=True)
for row in ws.rows:
    # Process row
    pass
wb.close()

# Use write_only mode for writing large files
wb = Workbook(write_only=True)
ws = wb.create_sheet()
for row_data in large_dataset:
    ws.append(row_data)
wb.save('output.xlsx')
```

**DON'T:**
```python
# Don't load entire large files into memory
wb = load_workbook('huge_file.xlsx')  # May cause memory error
all_data = list(ws.values)  # Loads everything into RAM
```

### 4. Preserving Existing Formatting

**DO:**
```python
# Load workbook without data_only to preserve formulas
wb = load_workbook('report.xlsx')

# When copying styles, use copy()
from copy import copy
new_cell.font = copy(old_cell.font)
new_cell.fill = copy(old_cell.fill)
```

**DON'T:**
```python
# Don't use data_only if you need to preserve formulas
wb = load_workbook('report.xlsx', data_only=True)
ws['A1'] = ws['A1'].value  # This will replace formula with value!
```

### 5. Error Handling

**DO:**
```python
from openpyxl import load_workbook
from openpyxl.utils.exceptions import InvalidFileException

try:
    wb = load_workbook('data.xlsx')
    ws = wb.active

    # Validate data exists
    if ws.max_row < 2:
        raise ValueError("File contains no data rows")

    # Process data
    for row in ws.iter_rows(min_row=2, values_only=True):
        if row[0] is not None:  # Check for empty cells
            # Process row
            pass

    wb.save('data.xlsx')

except FileNotFoundError:
    print("Error: File not found")
except InvalidFileException:
    print("Error: Invalid Excel file format")
except Exception as e:
    print(f"Unexpected error: {e}")
finally:
    if 'wb' in locals():
        wb.close()
```

### 6. Date and Time Handling

**DO:**
```python
from datetime import datetime
from openpyxl.styles import numbers

# Write date with proper format
ws['A1'] = datetime(2024, 1, 15)
ws['A1'].number_format = numbers.FORMAT_DATE_XLSX14  # mm-dd-yy

# Or use string format
ws['A1'].number_format = 'dd/mm/yyyy'

# Read dates correctly
wb = load_workbook('file.xlsx')
date_value = ws['A1'].value  # Returns datetime object
```

**DON'T:**
```python
# Don't write dates as strings
ws['A1'] = "2024-01-15"  # This is text, not a date
```

### 7. Column Width Auto-adjustment

**DO:**
```python
from openpyxl.utils import get_column_letter

def auto_adjust_column_width(ws):
    """Auto-adjust column widths based on content."""
    for column in ws.columns:
        max_length = 0
        column_letter = get_column_letter(column[0].column)

        for cell in column:
            try:
                if cell.value:
                    max_length = max(max_length, len(str(cell.value)))
            except:
                pass

        adjusted_width = min(max_length + 2, 50)  # Cap at 50
        ws.column_dimensions[column_letter].width = adjusted_width

# Usage
auto_adjust_column_width(ws)
```

## Common Pitfalls and Solutions

### Pitfall 1: Formula Reference Errors

**Problem:**
```python
# This creates broken formula when copied
ws['B2'] = "=A2*10"
ws['B3'] = "=A2*10"  # Still references A2, not A3!
```

**Solution:**
```python
# Let openpyxl handle relative references
for row in range(2, 10):
    ws[f'B{row}'] = f"=A{row}*10"  # Correct relative reference
```

### Pitfall 2: Lost Formulas When Using data_only

**Problem:**
```python
wb = load_workbook('file.xlsx', data_only=True)
ws['A1'] = ws['A1'].value  # Replaces formula with calculated value
wb.save('file.xlsx')  # Formulas are lost!
```

**Solution:**
```python
# Don't use data_only if you need to preserve formulas
wb = load_workbook('file.xlsx')  # Keep formulas intact

# Or, read values separately if needed
wb_data = load_workbook('file.xlsx', data_only=True)
value = wb_data['Sheet1']['A1'].value

wb_formula = load_workbook('file.xlsx')
formula = wb_formula['Sheet1']['A1'].value
```

### Pitfall 3: Chart Data Range Errors

**Problem:**
```python
# Wrong: Includes header in categories
categories = Reference(ws, min_col=1, min_row=1, max_row=10)
```

**Solution:**
```python
# Correct: Separate headers from data
data = Reference(ws, min_col=2, min_row=1, max_col=4, max_row=10)
categories = Reference(ws, min_col=1, min_row=2, max_row=10)  # Start at row 2

chart.add_data(data, titles_from_data=True)
chart.set_categories(categories)
```

### Pitfall 4: Date Format Issues

**Problem:**
```python
# Dates appear as numbers (e.g., 44927)
ws['A1'] = datetime(2024, 1, 15)
# Excel shows: 44927
```

**Solution:**
```python
from openpyxl.styles import numbers

ws['A1'] = datetime(2024, 1, 15)
ws['A1'].number_format = numbers.FORMAT_DATE_XLSX14
# Excel now shows: 01-15-24
```

### Pitfall 5: Memory Issues with Large Files

**Problem:**
```python
wb = load_workbook('huge_file.xlsx')  # Loads entire file into memory
data = list(ws.values)  # Memory error!
```

**Solution:**
```python
# Use read_only mode
wb = load_workbook('huge_file.xlsx', read_only=True)
for row in ws.iter_rows(values_only=True):
    # Process one row at a time
    pass
wb.close()
```

### Pitfall 6: Merged Cells Confusion

**Problem:**
```python
ws.merge_cells('A1:C1')
print(ws['B1'].value)  # Returns None!
print(ws['C1'].value)  # Returns None!
```

**Solution:**
```python
ws.merge_cells('A1:C1')
ws['A1'] = "Merged Title"  # Only set value in top-left cell
print(ws['A1'].value)  # Returns "Merged Title"

# To unmerge
ws.unmerge_cells('A1:C1')
```

## Helper Script Reference

The included `scripts/excel_helper.py` provides utility functions:

```python
from scripts.excel_helper import (
    create_workbook,
    read_excel_data,
    add_chart,
    apply_formatting,
    add_formula,
    auto_fit_columns
)

# Create new workbook with data
wb, ws = create_workbook("Sales Report", headers=["Product", "Q1", "Q2"])

# Read data from existing file
data = read_excel_data("data.xlsx", sheet_name="Sheet1")

# Add chart to worksheet
add_chart(ws, chart_type="line", data_range="B2:D10", title="Sales Trend")

# Apply formatting
apply_formatting(ws, cell_range="A1:D1", bold=True, bg_color="4472C4")

# Add formula to range
add_formula(ws, cell="E2", formula="=SUM(B2:D2)", copy_down=10)

# Auto-fit all columns
auto_fit_columns(ws)

wb.save("output.xlsx")
```

## Quick Reference Commands

```python
# Installation
pip install openpyxl pandas

# Create workbook
from openpyxl import Workbook
wb = Workbook()
ws = wb.active

# Load workbook
from openpyxl import load_workbook
wb = load_workbook('file.xlsx')
ws = wb.active

# Read cell
value = ws['A1'].value
value = ws.cell(row=1, column=1).value

# Write cell
ws['A1'] = "Hello"
ws.cell(row=1, column=1, value="Hello")

# Write formula
ws['C1'] = "=A1+B1"

# Add row
ws.append([1, 2, 3])

# Save
wb.save('output.xlsx')

# Close
wb.close()
```

## Troubleshooting Guide

### Issue: "ModuleNotFoundError: No module named 'openpyxl'"
```bash
pip install openpyxl
# Or
uv pip install openpyxl
```

### Issue: "InvalidFileException: openpyxl does not support .xls files"
Solution: Convert .xls to .xlsx first, or use `xlrd` library for old format.

### Issue: Formulas showing as text
Solution: Don't prefix with quotes. Use `ws['A1'] = "=SUM(B1:B10)"` not `ws['A1'] = "'=SUM(B1:B10)"`

### Issue: Charts not appearing
Solution: Ensure data references are correct and save file after adding chart.

### Issue: Date showing as numbers
Solution: Apply date format: `ws['A1'].number_format = 'mm/dd/yyyy'`

## Additional Resources

- **openpyxl Documentation**: https://openpyxl.readthedocs.io/
- **pandas Excel Support**: https://pandas.pydata.org/docs/reference/io.html#excel
- **Excel Formula Reference**: https://support.microsoft.com/en-us/excel
- **Chart Examples**: https://openpyxl.readthedocs.io/en/stable/charts/introduction.html

## Summary

This skill enables comprehensive Excel automation including:
- ✅ Creating complex spreadsheets with formulas and formatting
- ✅ Reading and analyzing existing workbooks
- ✅ Editing files while preserving formulas and styles
- ✅ Creating professional charts and visualizations
- ✅ Applying conditional formatting and data validation
- ✅ Working with multiple worksheets and cross-sheet formulas
- ✅ Integrating with pandas for advanced data analysis
- ✅ Handling large datasets efficiently

Use this skill for any task involving Excel files, from simple data entry to complex financial reports and dashboards.
