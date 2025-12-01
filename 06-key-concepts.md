# Key Concepts Explained

## 1. Image Pyramids

### What is a Pyramid?

An image pyramid is a multi-resolution representation of an image. Think of it like a pyramid:

- **Base (Level 0)**: Highest resolution, most detail
- **Level 1**: 2x smaller (half resolution)
- **Level 2**: 4x smaller
- **Level 3**: 8x smaller (thumbnail)

### Why Pyramids?

**Problem**: A whole slide image can be 100,000 x 100,000 pixels = 10 billion pixels. Loading this all at once is impossible.

**Solution**: Store multiple resolutions:

- When zoomed out: Load low-resolution overview
- When zoomed in: Load high-resolution tiles
- Only load what's visible

### How It's Computed

The viewer processes multiple DICOM instances to build the pyramid:

1. **Sorts by size**: Arranges instances from largest (highest resolution) to smallest
2. **Validates**: Ensures all instances share the same `FrameOfReferenceUID` and `ContainerIdentifier`
3. **Calculates properties**: For each level, computes:
   - Resolution factor (how much smaller than base level)
   - Tile grid dimensions (how many tiles wide/tall)
   - Pixel spacing (physical size per pixel)

**Result**: A structured pyramid ready for multi-resolution rendering.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Pyramid computation sorts images by size
metadata.sort((a, b) => {
  const sizeDiff = a.TotalPixelMatrixColumns - b.TotalPixelMatrixColumns;
  return sizeDiff;
});

// Creates pyramid structure
const pyramid = {
  resolutions: [1, 2, 4, 8], // Zoom factors
  metadata: [level0, level1, level2, level3], // DICOM instances
  gridSizes: [
    [1000, 1000],
    [500, 500],
    [250, 250],
    [125, 125],
  ], // Tile grids
};
```
</details>

## 2. Tiling

### What is Tiling?

Even at a single pyramid level, loading the entire image is too large. Instead, images are divided into **tiles** (typically 256x256 or 512x512 pixels).

### Tile Grid

```
┌─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  4  │  Row 1
├─────┼─────┼─────┼─────┤
│  5  │  6  │  7  │  8  │  Row 2
├─────┼─────┼─────┼─────┤
│  9  │ 10  │ 11  │ 12  │  Row 3
└─────┴─────┴─────┴─────┘
```

Each tile is stored as a separate "frame" in the DICOM multi-frame object.

### Frame Mapping

The viewer creates a mapping: `"row-column-opticalPath" → frame number`

Example:

- `"1-2-0"` → Row 1, Column 2, Optical Path "0" → Frame 5
- Used to construct WADO-RS URLs

## 3. Coordinate Systems

### Three Coordinate Systems

1. **Pixel Coordinates**: `(x, y)` in pixels (0 to TotalPixelMatrixColumns)
2. **Physical Coordinates**: `(x, y)` in millimeters on the slide
3. **Screen Coordinates**: `(x, y)` in pixels on the screen

### Transformations

To convert between coordinate systems, multiply or divide by `PixelSpacing`:

- **Pixel → Physical**: Multiply pixel coordinates by pixel spacing
  - Example: Pixel (1000, 2000) × spacing (0.00025 mm) = Physical (0.25 mm, 0.5 mm)
- **Physical → Pixel**: Divide physical coordinates by pixel spacing
  - Example: Physical (0.25 mm, 0.5 mm) ÷ spacing (0.00025 mm) = Pixel (1000, 2000)

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Pixel → Physical
const physicalX = pixelX * pixelSpacing[0];
const physicalY = pixelY * pixelSpacing[1];

// Physical → Pixel
const pixelX = physicalX / pixelSpacing[0];
const pixelY = physicalY / pixelSpacing[1];
```
</details>

### Why This Matters

- **Annotations**: Stored in physical coordinates (millimeters)
- **Display**: Converted to pixel coordinates for rendering
- **Precision**: Physical coordinates are device-independent

## 4. Optical Paths (Channels)

### What are Optical Paths?

An optical path represents a different way the slide was imaged:

- **Brightfield**: Standard H&E staining
- **Fluorescence**: DAPI, FITC, Rhodamine optical paths
- **Multiplexed**: Many optical paths (CycIF)

### How They're Stored

**Important**: Optical paths are stored as **separate frames within the same DICOM instance** (same `SOPInstanceUID` and `SeriesInstanceUID`), not as separate images or series.

Each optical path is:

- Identified by `OpticalPathIdentifier` in the `OpticalPathSequence` metadata
- Stored as separate frames in the same multi-frame DICOM object
- Can be displayed independently or blended

### Channel Blending

For fluorescence images, multiple optical paths can be:

- **Displayed separately**: Switch between optical paths
- **Blended additively**: Overlay optical paths with colors
- **Used for analysis**: Combine optical paths for measurements

## 5. DICOMweb

### What is DICOMweb?

DICOMweb is a RESTful API for accessing DICOM data over HTTP. It provides three main services:

1. **QIDO-RS** (Query): Search for studies/series/instances
2. **WADO-RS** (Retrieve): Get metadata and pixel data
3. **STOW-RS** (Store): Store new DICOM objects

### Example URLs

DICOMweb uses RESTful URLs:

- **Query studies**: `GET /studies?PatientID=12345` - Search for studies matching criteria
- **Retrieve series metadata**: `GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/metadata` - Get all DICOM attributes for instances in a series
- **Retrieve a frame**: `GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/1` - Get pixel data for a specific frame

These are standard HTTP GET requests - no special DICOM protocol needed.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Query for studies
GET /studies?PatientID=12345

// Retrieve series metadata
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/metadata

// Retrieve a frame
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/1
```
</details>

### Why DICOMweb?

- **Standard**: Works with existing PACS/VNA systems
- **HTTP-based**: Easy to use, firewall-friendly
- **Scalable**: Can handle large datasets
- **Interoperable**: Works across vendors

## 6. Annotations

### Types of Annotations

1. **ROI (Region of Interest)**: Vector graphics

   - Point
   - Polygon
   - Ellipse
   - Polyline

2. **Segmentation**: Binary masks

   - Pixels marked as foreground/background
   - Used for nuclei segmentation, tissue segmentation

3. **Parametric Maps**: Heat maps

   - Attention maps
   - Saliency maps
   - Class activation maps

4. **Bulk Annotations**: Large collections
   - Thousands of cell annotations
   - Efficient storage format

### Storage Formats

- **DICOM SR**: Structured reports (TID 1500, TID 1410)
- **DICOM Segmentation**: Binary masks
- **DICOM Parametric Map**: Float arrays
- **DICOM Bulk Annotations**: Optimized for many annotations

### Coordinate Storage

Annotations are stored in **physical coordinates** (millimeters):

- Device-independent
- Precise measurements
- Converted to pixel coordinates for display

## 7. Compression

### Why Compress?

Uncompressed WSI images are huge:

- 100,000 x 100,000 x 3 bytes = 30 GB per slide
- Compression reduces to ~1-5 GB

### Compression Formats

1. **JPEG**: Lossy, good for color images

   - Transfer Syntax: `1.2.840.10008.1.2.4.50`
   - Good compression ratio
   - Some quality loss

2. **JPEG 2000**: Better compression

   - Transfer Syntax: `1.2.840.10008.1.2.4.90` (lossless)
   - Transfer Syntax: `1.2.840.10008.1.2.4.91` (lossy)
   - Better quality than JPEG
   - Supports lossless

3. **JPEG-LS**: Lossless compression
   - Transfer Syntax: `1.2.840.10008.1.2.4.80` (lossless)
   - Transfer Syntax: `1.2.840.10008.1.2.4.81` (near-lossless)
   - No quality loss
   - Good for medical images

### Decoding

The viewer decodes compressed frames client-side:

- Uses WebAssembly for performance
- Supports all three formats
- Handles color space transformations

## 8. Concatenations

### What are Concatenations?

Sometimes a single DICOM instance is split across multiple files:

- **Concatenation UID**: Links related parts
- **InConcatenationNumber**: Part number (1, 2, 3, ...)
- **ConcatenationFrameOffsetNumber**: Frame offset

### Why Split?

- **File size limits**: Some systems limit file sizes
- **Network efficiency**: Can transfer parts in parallel
- **Storage**: Easier to manage smaller files

### Reassembly

The viewer automatically detects and reassembles concatenations:

1. **Detects concatenation**: Checks for `SOPInstanceUIDOfConcatenationSource` attribute
2. **Groups parts**: Links parts using `ConcatenationUID` and `InConcatenationNumber`
3. **Reassembles metadata**: Combines metadata from all parts
4. **Combines frame mappings**: Merges frame mappings accounting for `ConcatenationFrameOffsetNumber`
5. **Creates unified instance**: Treats reassembled parts as a single instance

**Result**: The viewer sees one complete image, even though it was stored in multiple files.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Detects concatenation parts
if (metadata.SOPInstanceUIDOfConcatenationSource) {
  // Reassemble metadata
  // Combine frame mappings
  // Create unified instance
}
```
</details>

## 9. ICC Profiles

### What are ICC Profiles?

ICC (International Color Consortium) profiles define color spaces:

- Ensures consistent color display
- Corrects for scanner variations
- Important for accurate diagnosis

### How They're Used

1. **Stored in DICOM**: In `OpticalPathSequence.ICCProfile`
2. **Retrieved**: Fetched from DICOMweb if needed
3. **Applied**: Used to transform pixel data
4. **Display**: Ensures accurate colors

### Why Important?

- **Diagnosis**: Color accuracy matters for pathology
- **Consistency**: Same slide looks same on different displays
- **Standards**: Required for some use cases

## 10. Events and Interactions

### Viewer Events

The viewer publishes events for:

- **Frame loading**: Started, ended, error
- **Viewport changes**: Pan, zoom
- **Selection**: ROI selected
- **Annotations**: Added, removed, modified

### Event Flow

```
User interaction
    ↓
OpenLayers event
    ↓
dicom-microscopy-viewer processes
    ↓
Publishes custom event
    ↓
Slim listens and updates UI
```

### Example Events

The viewer publishes events that Slim can listen to:

- **FRAME_LOADING_STARTED**: A tile is being requested from DICOMweb
- **FRAME_LOADING_ENDED**: Tile successfully loaded and decoded
- **FRAME_LOADING_ERROR**: Failed to load a tile
- **Viewport changes**: User panned or zoomed
- **Selection**: User selected an ROI

Slim uses these events to update the UI (show loading indicators, update position displays, etc.).

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Frame loading started
EVENT.FRAME_LOADING_STARTED;

// Frame loading completed
EVENT.FRAME_LOADING_ENDED;

// Frame loading error
EVENT.FRAME_LOADING_ERROR;
```
</details>

## Summary

These concepts work together:

- **Pyramids** enable multi-resolution viewing
- **Tiling** enables efficient loading
- **Coordinates** enable precise annotations
- **DICOMweb** enables data access
- **Compression** enables storage efficiency
- **Annotations** enable analysis and documentation

Understanding these concepts helps explain how the viewer works and why certain design decisions were made.

## Next Steps

- [Common Use Cases](./07-common-use-cases.md) - Real-world scenarios
- [Troubleshooting](./08-troubleshooting.md) - Common issues and solutions
