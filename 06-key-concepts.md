# Key Concepts

This document explains important technical concepts used throughout the pathology viewer system.

## 1. Image Pyramids

### What is a Pyramid?

A multi-resolution pyramid stores the same image at multiple zoom levels:

- **Level 0** (base): Highest resolution (e.g., 0.25 mpp / "40X")
- **Level 1**: 2× smaller (e.g., 0.5 mpp / "20X")
- **Level 2**: 4× smaller (e.g., 1.0 mpp / "10X")
- **Level 3**: 8× smaller (e.g., 2.5 mpp / "4X")
- **Level 4**: 16× smaller (e.g., 5.0 mpp / "2X")

### Why Pyramids?

According to the [official DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/), pyramids enable:

- **Rapid zooming**: Switch between resolution levels without scaling large images
- **Efficient storage**: Pre-computed levels avoid runtime computation
- **Fast initial display**: Lower resolution loads quickly for overview

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

Even at a single pyramid level, loading the entire image is too large. Instead, images are divided into **tiles** (typically 256×256 or 512×512 pixels).

### Tile Grid

```
┌─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  4  │  Row 1
├─────┼─────┼─────┼─────┤
│  5  │  6  │  7  │  8  │  Row 2
├─────┼─────┼─────┤
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

### Pixel Coordinates

- **Origin**: Upper-left corner of image matrix
- **Row**: Vertical position (increasing downward)
- **Column**: Horizontal position (increasing rightward)
- **Units**: Pixels

### Physical Coordinates (Slide-Based)

According to the [DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/):

- **Origin**: Top-left corner of slide (label on left)
- **X-axis**: Horizontal, increasing left to right
- **Y-axis**: Vertical, increasing top to bottom
- **Z-axis**: Perpendicular to slide surface (height above slide)
- **Units**: Millimeters (mm) or micrometers (μm)

**Important**: Slide-based coordinates are rotated 180° from image matrix coordinates.

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

## 4. Optical Paths (Channels)

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

### Display Modes

For fluorescence images, multiple optical paths can be:

- **Displayed separately**: Switch between optical paths
- **Blended additively**: Overlay optical paths with colors
- **Used for analysis**: Combine optical paths for measurements

## 5. DICOMweb

DICOMweb is a RESTful API for accessing DICOM data over HTTP.

### Services

- **QIDO-RS** (Query): Search for studies/series/instances
- **WADO-RS** (Retrieve): Retrieve metadata and pixel data
- **STOW-RS** (Store): Store new DICOM objects

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

## 6. Annotations

According to the [DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/), annotations are stored separately from images:

### Types

- **ROI (Region of Interest)**: Polygons, circles, points
- **Segmentation**: Binary masks categorizing regions
- **Structured Reporting**: Measurements, observations, findings
- **Presentation States**: Display parameters (blending, pseudocoloring)

### Storage

- **Separate Series**: Different modality than images
- **Reference Images**: Link to images via `ReferencedSOPInstanceUID`
- **Physical Coordinates**: Stored in slide-based coordinates, not pixels

## 7. Compression

WSI images are compressed to reduce storage and transfer:

- **JPEG**: Lossy, good for color images
- **JPEG 2000**: Better compression, supports lossless
- **JPEG-LS**: Lossless compression

The viewer supports all formats and decodes them client-side.

## 8. Concatenation

Large images may be split across multiple DICOM files:

- **Linked**: Parts linked by `SOPInstanceUIDOfConcatenationSource`
- **Identified**: By `ConcatenationUID` and `InConcatenationNumber`
- **Offset**: Frame offset tracked by `ConcatenationFrameOffsetNumber`

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

ICC (International Color Consortium) profiles ensure accurate color display:

- **Color correction**: Adjusts colors to match original scanner output
- **Standardization**: Ensures consistent colors across displays
- **Optional**: Can be disabled for performance

## 10. Event Publishing

The viewer publishes events for integration:

- **Frame loading**: Started, completed, error
- **Viewport changes**: Pan, zoom, rotation
- **Selection**: ROI selection, measurement

Slim uses these events to update the UI (loading indicators, position displays, etc.).

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

## Next Steps

- [Common Use Cases](./07-common-use-cases.md) - Real-world scenarios
- [Troubleshooting](./08-troubleshooting.md) - Common issues and solutions

