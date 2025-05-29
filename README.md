# Pre-Printed Forms Module for Odoo

> **Developer:** Ricardo Perico Jr  
> **Role:** FullStack Developer Intern 
> **Note:** This documentation is for internal reference only. Code uploads are restricted by company policy.

---

## Overview

The **Pre-Printed Forms Module** for Odoo enables the creation and management of pre-designed PDF forms with dynamic text overlays. It supports precise positioning, multiple font styles, and full control over page formats—ideal for automating government, legal, or custom company forms.

---

## Key Features

-  Upload and register PDF templates as static base forms.  
-  Add overlay text items with pixel-precise placement.  
-  Dynamically render content using Odoo model fields.  
-  Support for multiple page sizes: A4, A3, Letter, Legal, Half-sheet.  
-  Custom and built-in font support (TrueType and ReportLab).  
-  Font styling: **bold**, *italic*, underline, and combinations.  
-  Manual underline drawing for non-native font support.  
-  Generate downloadable, print-ready PDFs with overlays.  
-  Server actions integration for process automation.  

---

## Installation (Internal Use)

1. **Place the module folder** inside your local Odoo `addons` directory.  
2. **Add custom font files (`.ttf`)** under:  
   `your_module/static/fonts/`  
   Supported fonts:
   - **Arial**: `ARIAL.ttf`, `ARIAL-Bold.ttf`, `ARIAL-Italic.ttf`, `ARIAL-BoldItalic.ttf`
   - **Calibri**: `CALIBRI.ttf`, `CALIBRI-Bold.ttf`, `CALIBRI-Italic.ttf`, `CALIBRI-BoldItalic.ttf`
   - **AgencyFB**: `AGENCYFB.ttf`, `AGENCYFB-Bold.ttf`, `AGENCYFB-Italic.ttf`, `AGENCYFB-BoldItalic.ttf`

3. **Update the Apps list** and install the module through the Odoo interface.  
4. Upload PDF templates via the module UI and begin form setup.  

---

## How to Use

1. **Create a Form Record:** Navigate to the module interface and create a new pre-printed form.  
2. **Upload a PDF Template:** This acts as the static background.  
3. **Define Text Overlays:**
   - Set X/Y coordinates (in points).
   - Choose font, style (bold, italic, underline), and size.
   - Bind to static text or Odoo model fields dynamically.
4. **Generate Output:** Render and download the final PDF with overlays.  
5. **(Optional) Automate:** Add server actions to trigger generation from related models (e.g., Invoices, Orders).

---

## Fonts Supported

### Built-In Fonts (No TTF Needed)

- Times (Roman, Bold, Italic, BoldItalic)  
- Helvetica (Normal, Bold, Oblique, BoldOblique)  
- Courier (Normal, Bold, Oblique, BoldOblique)  

### Custom Fonts (TTF Required)

- **Arial**  
- **Calibri**  
- **AgencyFB**

---

## Implementation Details (by Developer)

This module was developed in **Odoo v17** using:

```python
from odoo import models, fields, api
from odoo.exceptions import UserError
from reportlab.lib.pagesizes import letter, legal, A3, A4
from reportlab.pdfgen import canvas
from reportlab.pdfbase.ttfonts import TTFont
from reportlab.pdfbase import pdfmetrics
import PyPDF2
from io import BytesIO
import base64
from odoo.modules import get_module_path
```

### Custom Font Registration

```python
font_path = get_module_path('your_module') + '/static/fonts/'
pdfmetrics.registerFont(TTFont('Calibri-BoldItalic', font_path + 'CALIBRI-BoldItalic.ttf'))
```

Registers a TrueType font for use in PDF generation. Font names must match their PostScript identifiers.

---

### Rendering Overlay Text on Canvas

```python
buffer = BytesIO()
pdf = canvas.Canvas(buffer, pagesize=A4)

pdf.setFont("Calibri-BoldItalic", 12)
pdf.drawString(100, 700, "Sample Text")
```

Creates a canvas and draws overlay text at a fixed position using the selected font and size.

---

### Underline Drawing (Manual)

```python
text = "Underlined Text"
x, y = 100, 680
pdf.drawString(x, y, text)
text_width = pdf.stringWidth(text, "Calibri-Bold", 12)
pdf.line(x, y - 2, x + text_width, y - 2)
```

Underline is not natively supported by fonts, so it is drawn manually using a line beneath the text.

---

### Merging with Base PDF (Using PyPDF2)

```python
# Save overlay
pdf.save()
buffer.seek(0)

# Load base and overlay
base_reader = PyPDF2.PdfReader(BytesIO(base64.b64decode(record.template_pdf)))
overlay_reader = PyPDF2.PdfReader(buffer)

# Merge pages
output = PyPDF2.PdfWriter()
for i in range(len(base_reader.pages)):
    base_page = base_reader.pages[i]
    overlay_page = overlay_reader.pages[0]
    base_page.merge_page(overlay_page)
    output.add_page(base_page)

# Output to binary field
output_buffer = BytesIO()
output.write(output_buffer)
record.generated_pdf = base64.b64encode(output_buffer.getvalue())
```

Merges the static base PDF with the dynamic overlay and stores the result as a binary PDF in Odoo.

---
