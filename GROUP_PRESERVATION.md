# SVG Group Structure Preservation

This document describes the SVG group structure preservation feature that has been implemented in SVG-to-PDFKit.

## Overview

SVG `<g>` (group) elements are now preserved in the output PDF using **PDF Marked Content** operators. This allows the SVG document structure to be maintained in the PDF, enabling better document organization, accessibility, and potential post-processing.

## How It Works

### Implementation Details

When an SVG `<g>` element is converted to PDF:

1. **Marked Content Wrapper**: The group content is wrapped with PDF marked content operators (`BDC`/`EMC`)
2. **Metadata Preservation**: Group attributes are stored in the marked content dictionary
3. **Unique Identifiers**: Each group receives a unique Marked Content ID (MCID)

### What Gets Preserved

The following SVG group attributes are preserved in the PDF:

| SVG Attribute | PDF Property | Description |
|--------------|--------------|-------------|
| `id` | `GroupID` | The SVG element's id attribute |
| `class` | `ClassName` | The SVG element's class attribute |
| `transform` | `Transform` | The transform attribute as a string |
| `data-*` | `data-*` | All custom data attributes |
| (automatic) | `MCID` | Unique Marked Content Identifier |

### Example

**SVG Input:**
```xml
<svg xmlns="http://www.w3.org/2000/svg">
  <g id="layer1" class="background" transform="translate(10, 20)" data-custom="value">
    <rect x="0" y="0" width="100" height="100" fill="red"/>
  </g>
</svg>
```

**PDF Content Stream Output:**
```pdf
/SVGGroup << /MCID 1 /GroupID (layer1) /ClassName (background) /Transform (translate(10, 20)) /data-custom (value) >> BDC
  q
  % ... rect drawing commands ...
  Q
EMC
```

## Usage

No changes are required to use this feature. It works automatically with existing code:

```javascript
const PDFDocument = require('pdfkit');
const SVGtoPDF = require('svg-to-pdfkit');

const doc = new PDFDocument();
SVGtoPDF(doc, svgElement, x, y);
```

## Benefits

### 1. Document Structure
- Preserves the logical organization of SVG content
- Makes it easier to understand complex SVGs in the PDF

### 2. Accessibility
- Marked content is the foundation for PDF accessibility features
- Can be extended to create tagged PDFs (PDF/UA)

### 3. Post-Processing
- Tools can identify and manipulate specific groups in the PDF
- Enables selective extraction or modification of content

### 4. Debugging
- Easier to trace which SVG elements correspond to PDF content
- Helpful when debugging conversion issues

## Technical Details

### When Marked Content Is Added

Marked content is added for `<g>` elements **except** when:
- The group is being used in a clipping path (`isClip = true`)
- The group is being used in a mask (`isMask = true`)

This ensures marked content only appears in the actual rendered content stream, not in auxiliary structures.

### Backward Compatibility

The implementation is **fully backward compatible**:
- ✅ Groups without any attributes work normally
- ✅ Groups with only transform work normally
- ✅ Nested groups work correctly
- ✅ Clipping and masking still function properly
- ✅ Existing functionality is unchanged

### Performance Impact

The performance impact is minimal:
- Metadata is only extracted once per group
- Only attributes that exist are included
- No impact on groups used in clip paths or masks

## Testing

Three test files are provided:

### 1. `test_group_preservation.js`
Creates a PDF with nested groups to verify the feature works end-to-end.

```bash
node test_group_preservation.js
```

### 2. `test_marked_content_debug.js`
Logs marked content calls to verify metadata is correctly captured.

```bash
node test_marked_content_debug.js
```

### 3. `test_backward_compatibility.js`
Runs comprehensive backward compatibility tests.

```bash
node test_backward_compatibility.js
```

## Inspecting Generated PDFs

To verify marked content in generated PDFs:

### Using qpdf (if available)
```bash
qpdf --qdf input.pdf output.txt
grep -A5 "SVGGroup" output.txt
```

### Using strings
```bash
strings output.pdf | grep -C3 "SVGGroup"
```

### Using Adobe Acrobat Pro
1. Open the PDF
2. View → Show/Hide → Navigation Panes → Content
3. Inspect the content tree for marked content sections

## Future Enhancements

Potential extensions to this feature:

1. **Structure Tree**: Create a PDF structure tree for full PDF/UA compliance
2. **Optional Content Groups (Layers)**: Convert SVG groups to PDF layers for visibility control
3. **Accessibility Tags**: Map SVG groups to PDF accessibility roles
4. **Configuration Options**: Allow users to customize which attributes are preserved
5. **Group Naming**: Automatically generate meaningful names for unnamed groups

## Code Location

The implementation is in `source.js`:
- **SvgElemGroup class**: Lines 2489-2557
- **getGroupMetadata method**: Lines 2520-2556
- **drawInDocument method**: Lines 2491-2518

## References

- [PDF Reference: Marked Content](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf#page=562)
- [PDFKit Documentation](http://pdfkit.org/)
- [SVG Specification: Grouping](https://www.w3.org/TR/SVG2/struct.html#Groups)