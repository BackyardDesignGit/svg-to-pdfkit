# Nested SVG Support

## Overview

Nested `<svg>` elements are now fully supported with structure preservation in PDF output. Each nested SVG is wrapped with `/NestedSVG` marked content, preserving the complete document hierarchy.

## Features

### Full Hierarchy Preservation

The implementation preserves complex hierarchies including:
- **Groups within groups**: `<g>` → `<g>` → `<g>`
- **Nested SVGs**: `<svg>` → `<svg>` → `<svg>`
- **Mixed structures**: `<g>` → `<svg>` → `<g>` → `<svg>`
- **Sibling relationships**: Multiple elements at the same nesting level

### Metadata Preservation

Each nested SVG element preserves:
- `id` attribute → `SVGID` in PDF
- `class` attribute → `ClassName` in PDF
- `viewBox` attribute → `ViewBox` in PDF
- `width` and `height` → `Width` and `Height` in PDF
- `x` and `y` position → `X` and `Y` in PDF
- All `data-*` attributes → preserved as-is in PDF
- Unique `MCID` for each nested SVG

## Example

### SVG Input
```xml
<svg xmlns="http://www.w3.org/2000/svg">
  <g id="outer-group">
    <rect fill="red"/>
    <svg id="viewport" x="100" y="100" width="200" height="200" viewBox="0 0 100 100">
      <g id="inner-group">
        <circle fill="blue"/>
      </g>
    </svg>
  </g>
</svg>
```

### PDF Output Structure
```
/SVGGroup << MCID: 1, GroupID: "outer-group" >> BDC
  ... red rectangle ...
  /NestedSVG << MCID: 2, SVGID: "viewport", ViewBox: "0 0 100 100", Width: 200, Height: 200, X: 100, Y: 100 >> BDC
    /SVGGroup << MCID: 3, GroupID: "inner-group" >> BDC
      ... blue circle ...
    EMC
  EMC
EMC
```

## Use Cases

### 1. Complex Layouts
Create modular SVG layouts with independent viewports:
```xml
<svg>
  <svg id="header" viewBox="0 0 800 100">...</svg>
  <svg id="content" viewBox="0 0 800 600">...</svg>
  <svg id="footer" viewBox="0 0 800 100">...</svg>
</svg>
```

### 2. Scalable Components
Embed reusable components with independent coordinate systems:
```xml
<svg>
  <svg id="icon-1" width="50" height="50" viewBox="0 0 24 24">...</svg>
  <svg id="icon-2" width="50" height="50" viewBox="0 0 24 24">...</svg>
</svg>
```

### 3. Viewport Management
Different viewports for different parts of the document:
```xml
<svg>
  <svg id="diagram" viewBox="0 0 1000 1000">...</svg>
  <svg id="legend" viewBox="0 0 200 200">...</svg>
</svg>
```

## Technical Details

### When Marked Content is Added

Nested SVG elements get marked content **except**:
- When the SVG is the outer/root element (`isOuterElement = true`)
- When used in clipping paths (`isClip = true`)
- When used in masks (`isMask = true`)

This ensures:
- The document root is not wrapped (avoiding redundant wrapping)
- Auxiliary structures like clips/masks don't get marked content
- Only actual rendered content gets structure preservation

### Coordinate Systems

Nested SVGs establish their own coordinate systems via:
1. **Position**: `x` and `y` attributes set viewport position
2. **Size**: `width` and `height` set viewport dimensions
3. **Scaling**: `viewBox` establishes the internal coordinate system
4. **Transform**: Additional transformations can be applied

All of these are preserved in the PDF metadata, allowing reconstruction of the exact viewport configuration.

### Integration with Groups

Nested SVGs work seamlessly with group structure:
- Groups can contain nested SVGs
- Nested SVGs can contain groups
- Arbitrary nesting depth is supported
- Each level is properly marked with the appropriate tag (`/SVGGroup` or `/NestedSVG`)

## Testing

### Test Files

Three comprehensive test files are provided:

#### `test_nested_svg.js`
Tests basic nested SVG functionality:
- Simple nested SVG
- Multiple sibling nested SVGs
- Deep nesting with groups

#### `test_nested_svg_metadata.js`
Verifies all metadata is correctly preserved:
- SVGID, ClassName
- ViewBox, Width, Height
- X, Y position
- data-* attributes

#### `test_complete_structure.js`
Tests complex real-world structures:
- Mixed groups and nested SVGs
- Multiple nesting levels
- Sibling relationships
- Complete hierarchy preservation

### Running Tests

```bash
node test_nested_svg.js
node test_nested_svg_metadata.js
node test_complete_structure.js
```

## Backward Compatibility

✅ Fully backward compatible:
- Existing SVGs without nesting work unchanged
- Outer SVG element is not wrapped (as before)
- No breaking changes to API or behavior
- All existing tests continue to pass

## Benefits

### 1. Structure Preservation
The complete SVG hierarchy is preserved in the PDF, enabling:
- Document structure analysis
- Content identification and extraction
- Mapping between PDF and original SVG

### 2. Accessibility
Foundation for PDF accessibility features:
- Tagged PDF support (PDF/UA)
- Screen reader compatibility
- Logical reading order

### 3. Post-Processing
Enables advanced PDF manipulation:
- Selective content extraction
- Element identification
- Structure-aware transformations

### 4. Debugging
Easier debugging and inspection:
- Trace PDF content back to SVG elements
- Understand document structure
- Identify rendering issues

## Comparison: Groups vs Nested SVGs

| Feature | `<g>` Groups | Nested `<svg>` |
|---------|--------------|----------------|
| PDF Tag | `/SVGGroup` | `/NestedSVG` |
| Coordinate System | Inherits parent | Independent viewport |
| Metadata: ID | `GroupID` | `SVGID` |
| Metadata: Transform | ✅ `Transform` | ❌ (uses x, y, viewBox) |
| Metadata: ViewBox | ❌ | ✅ `ViewBox` |
| Metadata: Dimensions | ❌ | ✅ `Width`, `Height` |
| Metadata: Position | ❌ | ✅ `X`, `Y` |
| Primary Use | Grouping elements | Viewport management |

## Future Enhancements

Potential future additions:
1. **Viewport clipping**: Automatic clipping to viewport bounds
2. **Nested viewport references**: Cross-referencing between viewports
3. **Viewport metadata**: Additional viewport-specific properties
4. **Coordinate transformation maps**: Explicit coordinate system mappings

## Implementation

### Source Code Location

In `source.js`:
- **SvgElemSvg class**: Lines 2578-2704
- **getSvgMetadata method**: Lines 2641-2683
- **drawInDocument method**: Lines 2602-2640

### Key Changes

1. Added `getSvgMetadata()` method to extract SVG-specific attributes
2. Modified `drawInDocument()` to wrap content with `/NestedSVG` marked content
3. Added check for `isOuterElement` to avoid wrapping root SVG
4. Preserved all relevant attributes: id, class, viewBox, dimensions, position, data-*

## See Also

- [GROUP_PRESERVATION.md](./GROUP_PRESERVATION.md) - Documentation for group structure preservation
- [PDF Marked Content Specification](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf#page=562)
- [SVG Nested SVG Specification](https://www.w3.org/TR/SVG2/struct.html#NewDocument)