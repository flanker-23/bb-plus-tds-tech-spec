# Razorpay Tech Spec DOCX Formatting Guide

This document explains the formatting standards used in `convert_summary_to_docx.py` based on the Razorpay benchmark document: **Tech Spec - Account Statements Read Cutoff**.

## 📋 Table of Contents
- [Font Standards](#font-standards)
- [Heading Hierarchy](#heading-hierarchy)
- [Spacing Guidelines](#spacing-guidelines)
- [Color Palette](#color-palette)
- [Table Formatting](#table-formatting)
- [Code Blocks](#code-blocks)
- [Callout Boxes](#callout-boxes)
- [Comparison: Old vs Enhanced](#comparison-old-vs-enhanced)

---

## 🔤 Font Standards

### Primary Font: **Arial**
The benchmark uses Arial throughout (not Calibri). This matches Google Docs export format and Razorpay's standard documentation.

```python
# All text elements
normal.font.name = "Arial"
heading.font.name = "Arial"
code.font.name = "Courier New"  # Exception for code blocks
```

### Font Sizes
| Element | Size | Usage |
|---------|------|-------|
| **Title** | 26pt | Main document title |
| **H1** | 18pt | Top-level sections |
| **H2** | 16pt | Sub-sections |
| **H3** | 14pt | Sub-sub-sections |
| **H4** | 12pt | Minor headings |
| **Body** | 11pt | Paragraphs, lists |
| **Table Content** | 10-11pt | Table cells |
| **Code** | 10pt | Code blocks |
| **TOC Note** | 9pt | Helper text |

---

## 📊 Heading Hierarchy

### H1 (18pt, Blue #1155cc)
- **Purpose**: Major sections (1., 2., 3., etc.)
- **Spacing**: 20pt before, 6pt after
- **Style**: Bold, Razorpay Blue
- **Line height**: 1.15

### H2 (16pt, Blue #1155cc)
- **Purpose**: Subsections (1.1, 1.2, etc.)
- **Spacing**: 18pt before, 6pt after
- **Style**: Bold, Razorpay Blue

### H3 (14pt, Blue #1155cc)
- **Purpose**: Sub-subsections (1.1.1, 1.1.2)
- **Spacing**: 16pt before, 4pt after
- **Style**: Bold, Razorpay Blue

### H4 (12pt, Blue #1155cc)
- **Purpose**: Minor headings
- **Spacing**: 14pt before, 4pt after
- **Style**: Bold, Razorpay Blue

```python
# Implementation example
heading_config = {
    "Heading 1": 18,  # Major sections
    "Heading 2": 16,  # Subsections
    "Heading 3": 14,  # Sub-subsections
    "Heading 4": 12,  # Minor headings
}

for style_name, size in heading_config.items():
    h = doc.styles[style_name]
    h.font.name = "Arial"
    h.font.size = Pt(size)
    h.font.bold = True
    h.font.color.rgb = RGBColor.from_string("1155cc")
```

---

## 📏 Spacing Guidelines

### Paragraph Spacing
- **After paragraphs**: 6pt
- **Line spacing**: 1.15 (matches Google Docs)
- **Keep headings with next paragraph**: Enabled

### Heading Spacing
```
H1: 20pt before, 6pt after
H2: 18pt before, 6pt after
H3: 16pt before, 4pt after
H4: 14pt before, 4pt after
```

This creates natural visual hierarchy while maintaining readability.

---

## 🎨 Color Palette

### Razorpay Standard Colors
```python
RAZORPAY_BLUE = "1155cc"    # Headings
TEXT_BLACK = "000000"        # Body text
LINK_BLUE = "0000ee"        # Hyperlinks (standard web blue)
TABLE_BORDER = "CCCCCC"     # Table borders (light gray)

# Callout box backgrounds
INFO_BG = "EFF6FF"          # Blue tint for info boxes
WARNING_BG = "FFFBEB"       # Amber tint for warnings
SUCCESS_BG = "F0FDF4"       # Green tint for recommendations
CODE_BG = "F8F9FA"          # Gray for code blocks
```

### Color Usage
| Color | Hex | Usage |
|-------|-----|-------|
| Razorpay Blue | #1155cc | All headings (H1-H4) |
| Black | #000000 | Body text, paragraphs |
| Link Blue | #0000ee | Hyperlinks |
| Light Gray | #CCCCCC | Table borders |
| Blue Tint | #EFF6FF | Info/note boxes |
| Amber Tint | #FFFBEB | Warning boxes |
| Green Tint | #F0FDF4 | Recommendation boxes |
| Code Gray | #F8F9FA | Code block backgrounds |

---

## 📋 Table Formatting

### Header Row
- **Background**: Razorpay Blue (#1155cc)
- **Text color**: White (#FFFFFF)
- **Font**: Arial 11pt Bold
- **Borders**: All sides, 4pt single line

### Data Rows
- **Background**: White or alternating (optional)
- **Text**: Arial 10-11pt
- **Borders**: All sides, light gray (#CCCCCC)

```python
# Header cell
set_cell_shading(header_cell, "1155cc")
run.font.color.rgb = RGBColor.from_string("FFFFFF")
run.font.bold = True

# Add borders to entire table
add_table_borders(table)
```

### Table Best Practices
- ✅ Use consistent column widths
- ✅ Bold header row with blue background
- ✅ Align numbers right, text left
- ✅ Add borders to all cells
- ✅ Keep table width within page margins

---

## 💻 Code Blocks

### Formatting
- **Font**: Courier New (monospace)
- **Size**: 10pt
- **Background**: Light gray (#F8F9FA)
- **Border**: Left accent bar (blue)
- **Indent**: 120 twips from left

```python
def format_code_block(para, code_text):
    run = para.add_run(code_text)
    run.font.name = "Courier New"
    run.font.size = Pt(10)
    run.font.color.rgb = RGBColor.from_string("000000")

    # Add background shading
    add_paragraph_border(para, "F8F9FA")
```

### Example
```json
{
  "contact_id": "cont_123",
  "gross_amount": 1000000,
  "category_id": "194J",
  "apply_ldc": false
}
```

---

## 📦 Callout Boxes

### Info Boxes (Blue)
- **Background**: #EFF6FF
- **Border**: Left blue accent (18pt)
- **Use**: General information, notes

### Warning Boxes (Amber)
- **Background**: #FFFBEB
- **Border**: Left blue accent (18pt)
- **Use**: Cautions, important notices

### Recommendation Boxes (Green)
- **Background**: #F0FDF4
- **Border**: Left blue accent (18pt)
- **Use**: Best practices, recommendations

```python
def add_paragraph_border(para, bg_color):
    # Shading
    shd = OxmlElement("w:shd")
    shd.set(qn("w:fill"), bg_color)

    # Left accent border (blue)
    left = OxmlElement("w:left")
    left.set(qn("w:val"), "single")
    left.set(qn("w:sz"), "18")
    left.set(qn("w:color"), "1155cc")
```

---

## 🔄 Comparison: Old vs Enhanced

### Font Changes
| Element | Old | Enhanced |
|---------|-----|----------|
| Primary Font | Calibri | **Arial** ✅ |
| Title Size | 24pt | **26pt** ✅ |
| H1 Size | 16pt | **18pt** ✅ |
| H2 Size | 14pt | **16pt** ✅ |
| H3 Size | 13pt | **14pt** ✅ |
| H4 Size | 12pt | **12pt** ✅ |

### Color Changes
| Element | Old | Enhanced |
|---------|-----|----------|
| Heading Color | #1A3C6E (dark blue) | **#1155cc (Razorpay blue)** ✅ |
| Link Color | #1A73E8 (custom) | **#0000ee (standard web)** ✅ |
| Body Text | Varied | **#000000 (consistent)** ✅ |

### Spacing Changes
| Element | Old | Enhanced |
|---------|-----|----------|
| H1 spacing | 12pt before | **20pt before** ✅ |
| H2 spacing | 12pt before | **18pt before** ✅ |
| Line height | 1.0 | **1.15** ✅ |

### Other Improvements
- ✅ **Better character encoding** (handles special chars)
- ✅ **Consistent Arial usage** throughout
- ✅ **Proper spacing** matching Google Docs
- ✅ **Cleaner table formatting** with standard borders
- ✅ **Professional callout boxes** with proper shading

---

## 🛠️ Usage

### Basic Usage
```bash
python3 convert_summary_to_docx.py
```

### Requirements
```bash
pip install python-docx beautifulsoup4
brew install poppler  # For PDF support (optional)
npm install -g @mermaid-js/mermaid-cli  # For diagrams
```

### Input
- HTML file: `TDS_Payout_Integration_Tech_Spec_Summary.html`
- Must have Mermaid diagrams in `<div class="mermaid">` blocks

### Output
- DOCX file: `TDS_Payout_Integration_Tech_Spec_Summary.docx`
- Format: Razorpay Standard
- Size: ~350KB (with 6 embedded diagrams)

---

## 📚 Reference Documents

### Benchmark Document
**Tech Spec - Account Statements Read Cutoff**
- Location: `/Users/prashant.chauhan/Desktop/PAYOUTS TDS SUPPORT DOC/`
- Format: PDF + HTML export from Google Docs
- Serves as the gold standard for Razorpay tech spec formatting

### Key Takeaways from Benchmark
1. **Arial is the standard font** (not Calibri)
2. **Headings are blue (#1155cc)**, not dark blue
3. **Font sizes** are larger than typical: H1=18pt, H2=16pt
4. **Spacing follows 1.15 line height** (Google Docs default)
5. **Tables use blue headers** with white text
6. **Code blocks use Courier New** 10pt

---

## 🚀 Future Enhancements

### Planned Improvements
- [ ] **PDF export** directly from Python
- [ ] **Custom templates** for different doc types
- [ ] **Automated table formatting** based on content
- [ ] **Image compression** for smaller file sizes
- [ ] **Style inheritance** for nested lists
- [ ] **Page headers/footers** with doc metadata
- [ ] **Version watermarks** for draft documents

### Potential Features
- [ ] **Dark mode support** (optional color scheme)
- [ ] **Multi-language** support
- [ ] **Accessibility** improvements (alt text, etc.)
- [ ] **Export to Google Docs** format
- [ ] **Template library** for different sections

---

## ✅ Quality Checklist

Before finalizing any DOCX output, verify:

- [ ] **Font**: Arial throughout (except code blocks)
- [ ] **Headings**: Blue (#1155cc), proper sizes (18, 16, 14, 12pt)
- [ ] **Body text**: 11pt black
- [ ] **Spacing**: 1.15 line height, proper paragraph spacing
- [ ] **Tables**: Blue headers, borders on all cells
- [ ] **Code blocks**: Courier New, gray background
- [ ] **Links**: Clickable, blue, underlined
- [ ] **TOC**: Updates correctly (Ctrl+A, F9 in Word)
- [ ] **Diagrams**: Clear, readable, properly sized
- [ ] **Page breaks**: Logical section boundaries
- [ ] **No orphans/widows**: Headings stay with content

---

## 📝 License & Credits

**Created by**: Claude (Anthropic)
**Based on**: Razorpay Tech Spec Template
**Benchmark**: Tech Spec - Account Statements Read Cutoff
**Version**: 2.0 (Enhanced)
**Date**: February 2026

---

**Note**: This formatting guide is continuously updated based on feedback and new benchmark documents. Always refer to the latest version in the repository.
