# PDF Layers (OCG) Support for Adobe Illustrator

## Overview

SVG-to-PDFKit now supports **PDF Optional Content Groups (OCG)**, also known as **PDF Layers**. This means SVG groups with `id` attributes are converted into actual PDF layers that are **fully recognized by Adobe Illustrator**.

## What This Means

When you:
1. Create an SVG with groups that have `id` attributes
2. Convert it to PDF using SVG-to-PDFKit
3. Open the PDF in Adobe Illustrator

**The group IDs appear as named, editable layers in Illustrator's Layers panel!**

## How It Works

### SVG Groups with IDs → PDF Layers

**SVG groups with `id` attributes become PDF layers:**

```xml
<g id="background-layer">...</g>  →  PDF Layer named "background-layer"
<g id="shapes-layer">...</g>      →  PDF Layer named "shapes-layer"
<g id="text-layer">...</g>        →  PDF Layer named "text-layer"
```

**SVG groups WITHOUT `id` attributes use marked content** (structure preservation only, not layers):

```xml
<g class="my-class">...</g>  →  Marked content only (not a PDF layer)
```

### Layer Names

If a group has both `id` and `class`, the layer name includes both:

```xml
<g id="layer1" class="background">  →  PDF Layer named "layer1 (background)"
```

### Nested Layers

Nested groups with IDs create a hierarchy in the Layers panel:

```xml
<g id="parent">
  <g id="child1">...</g>
  <g id="child2">...</g>
</g>
```

Creates:
```
parent
  ├─ child1
  └─ child2
```

## Usage

### No Code Changes Required!

The feature works automatically - just ensure your SVG groups have `id` attributes:

```javascript
const PDFDocument = require('pdfkit');
const SVGtoPDF = require('svg-to-pdfkit');

const doc = new PDFDocument();
SVGtoPDF(doc, svgElement, x, y);
```

### Example SVG

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="600" height="400">
  <g id="background">
    <rect width="600" height="400" fill="#f0f0f0"/>
  </g>

  <g id="shapes">
    <rect x="50" y="50" width="100" height="100" fill="red"/>
    <circle cx="300" cy="100" r="50" fill="blue"/>
  </g>

  <g id="overlay">
    <circle cx="300" cy="200" r="60" fill="yellow" opacity="0.7"/>
  </g>
</svg>
```

This creates **3 PDF layers**: `background`, `shapes`, and `overlay`.

## Features

### In Adobe Illustrator

✅ **Named layers** - Group IDs appear as layer names
✅ **Layer hierarchy** - Nested groups preserved
✅ **Toggle visibility** - Eye icon to show/hide layers
✅ **Independent editing** - Each layer is fully editable
✅ **Reordering** - Drag layers to reorder
✅ **Layer selection** - Click to select entire layer

### In Adobe Acrobat

✅ **Layers panel** - View > Show/Hide > Navigation Panes > Layers
✅ **Toggle visibility** - Check/uncheck to show/hide
✅ **Layer list** - All layers displayed with names
✅ **Hierarchy** - Nested structure visible

### In Other PDF Viewers

Support varies:
- **PDF.js** (Firefox): Layer panel available
- **Preview (macOS)**: Limited OCG support
- **Chrome**: No layer panel (flattened view)

## Technical Details

### PDF Structure

For each SVG group with an ID, the library creates:

1. **OCG Dictionary**:
   ```
   << /Type /OCG /Name (layer-id) >>
   ```

2. **Content Marking**:
   ```
   /OC /OClayer_id BDC
     ... content ...
   EMC
   ```

3. **Catalog Entry**:
   ```
   /OCProperties <<
     /OCGs [ ... array of OCG references ... ]
     /D << /Order [ ... hierarchy ... ] /ON [ ... visible layers ... ] >>
   >>
   ```

### Behavior Rules

| SVG Element | PDF Output |
|-------------|-----------|
| `<g id="foo">` | PDF Layer named "foo" |
| `<g id="foo" class="bar">` | PDF Layer named "foo (bar)" |
| `<g class="foo">` | Marked content only (no layer) |
| `<g>` (no attributes) | Marked content only (no layer) |
| Nested `<g id="...">` | Nested PDF layers |

### Compatibility

**Works with:**
- ✅ Adobe Illustrator CC 2015+
- ✅ Adobe Acrobat Pro DC
- ✅ PDF Version 1.5+

**Backward Compatible:**
- ✅ Existing code works unchanged
- ✅ Groups without IDs still use marked content
- ✅ All previous features preserved

## Benefits

### For Designers

1. **Editable Layers**: Open PDFs in Illustrator and edit by layer
2. **Organized Content**: Logical layer structure preserved
3. **Non-Destructive**: Toggle layer visibility without deleting
4. **Reusable**: Save as Illustrator file with layers intact

### For Developers

1. **Automated Workflow**: SVG → PDF → Illustrator pipeline
2. **Layer Control**: Programmatically create layered PDFs
3. **Print Production**: Layers for different print versions
4. **Quality Control**: Review content layer-by-layer

### For Print Production

1. **Spot Colors**: Separate layers for different inks
2. **Overprint**: Control layer printing order
3. **Variants**: Multiple versions in one file
4. **Proofing**: Show/hide elements for client review

## Testing

### Test Files Provided

#### `test_ocg_layers.js`
Basic OCG functionality with 4 layers:
```bash
node test_ocg_layers.js
```

Creates `test_layers_output.pdf` with layers:
- background-layer
- shapes-layer
- text-layer
- overlay-layer

#### `test_ocg_hierarchy.js`
Tests nested layer hierarchy:
```bash
node test_ocg_hierarchy.js
```

Creates `test_layer_hierarchy.pdf` with:
- parent-layer
  - child-layer-1
  - child-layer-2
- separate-layer

### Verification Steps

**In Adobe Illustrator:**
1. Open the generated PDF
2. Window → Layers
3. Verify layer names match group IDs
4. Toggle eye icons to hide/show layers
5. Select and edit individual layers

**In Adobe Acrobat:**
1. Open the generated PDF
2. View → Show/Hide → Navigation Panes → Layers
3. Verify layer checkboxes work
4. Uncheck layers to hide content

**Technical Inspection:**
```bash
qpdf --qdf test_layers_output.pdf output.txt
grep -A20 "/OCProperties" output.txt
```

Should show:
- `/OCProperties` dictionary in catalog
- `/OCGs` array with OCG objects
- `/Type /OCG` for each layer
- `/Name` with layer names
- `/Order` array with hierarchy

## Limitations

### Current Limitations

1. **ID Required**: Groups without `id` attributes don't create layers
2. **Flat for Non-ID Groups**: Groups without IDs use marked content only
3. **No Layer Properties**: Advanced properties (locked, non-printing) not yet supported
4. **No Conditional Visibility**: No print/view/export conditions

### Workarounds

**For groups without IDs:**
```xml
<!-- This won't create a layer -->
<g class="my-group">

<!-- Add an ID to create a layer -->
<g id="my-group" class="my-group">
```

**For complex hierarchies:**
- Use meaningful IDs: `layer-background`, `layer-shapes`, etc.
- Keep nesting levels reasonable (2-3 levels max for clarity)
- Use class names for additional metadata

## Advanced Usage

### Dynamic Layer Names

Use descriptive IDs that make sense in Illustrator:

```xml
<g id="header-logo">...</g>
<g id="body-content">...</g>
<g id="footer-contact">...</g>
```

### Layer Organization

Group related elements under parent layers:

```xml
<g id="typography">
  <g id="headings">...</g>
  <g id="body-text">...</g>
  <g id="captions">...</g>
</g>
```

### Production Workflow

1. **Design in SVG**: Use groups with meaningful IDs
2. **Convert to PDF**: Run SVG-to-PDFKit
3. **Edit in Illustrator**: Open PDF, layers preserved
4. **Export**: Save as AI, PDF, or other formats

## Implementation Details

### Source Code Location

In `source.js`:

**OCG Helper Functions** (lines ~3720-3850):
- `createOCG()` - Creates OCG dictionary
- `beginOCG()` - Starts OCG content block
- `endOCG()` - Ends OCG content block
- `finalizeOCGs()` - Adds OCProperties to catalog

**Modified SvgElemGroup** (lines ~2489-2530):
- Detects groups with IDs
- Uses OCG for groups with IDs
- Falls back to marked content for groups without IDs

### Key Functions

```javascript
// Create an OCG for a layer
function createOCG(groupId, groupName)

// Begin OCG content (called when entering group)
function beginOCG(groupId, groupName)

// End OCG content (called when leaving group)
function endOCG()

// Finalize all OCGs and add to catalog
function finalizeOCGs()
```

## Migration Guide

### From Previous Versions

**No changes needed!** The feature is automatically enabled.

**Before (marked content only):**
```xml
<g id="layer1">...</g>  → Marked content only
```

**After (OCG layers):**
```xml
<g id="layer1">...</g>  → PDF Layer + Illustrator compatible
```

### Opting Out

If you want the old behavior (marked content only, no layers), remove the `id` attribute:

```xml
<!-- Creates layer -->
<g id="my-layer">...</g>

<!-- Uses marked content only -->
<g class="my-layer">...</g>
```

## See Also

- [GROUP_PRESERVATION.md](./GROUP_PRESERVATION.md) - Marked content structure preservation
- [NESTED_SVG_SUPPORT.md](./NESTED_SVG_SUPPORT.md) - Nested SVG element support
- [PDF Reference: Optional Content](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf#page=232)
- [Adobe Illustrator Layer Documentation](https://helpx.adobe.com/illustrator/using/layers.html)

## Troubleshooting

### Layers Not Appearing in Illustrator

**Problem**: PDF opens but no layers visible
**Solution**:
- Ensure groups have `id` attributes
- Check PDF was created with latest SVG-to-PDFKit version
- Try Window → Layers to open Layers panel

### Layer Names Incorrect

**Problem**: Layer names don't match group IDs
**Solution**:
- Verify `id` attribute in SVG
- Check for duplicate IDs (only first is used)
- Ensure IDs contain only alphanumeric characters

### Nested Layers Flat

**Problem**: Child layers not nested under parent
**Solution**:
- Verify SVG nesting structure
- Ensure all groups in hierarchy have `id` attributes
- Check parent group has `id` attribute

### PDF File Size Large

**Problem**: PDF file larger than expected
**Solution**:
- This is normal - OCG adds structure data
- Consider compressing PDF after generation
- Remove unused groups from SVG

---

**Created**: 2025-03-31
**Feature**: PDF Optional Content Groups (OCG)
**Compatibility**: PDF 1.5+, Illustrator CC 2015+