# DICOM WSI Format Deep Dive

> **Note**: This document explains DICOM concepts for WSI. Code examples are included but marked as optional - you can skip them and focus on the conceptual explanations if you're not coding.

## DICOM Structure Overview

DICOM files contain two main parts:

1. **Metadata**: Structured information about the image (patient ID, acquisition parameters, etc.)
2. **Pixel Data**: The actual image pixels

For WSI, pixel data is often stored externally and referenced via URIs (bulkdata).

## Key DICOM Attributes for WSI

### Identification Attributes

- **StudyInstanceUID**: Unique identifier for the study (collection of images)
- **SeriesInstanceUID**: Unique identifier for the series (related images)
- **SOPInstanceUID**: Unique identifier for the specific image instance
- **ContainerIdentifier**: Identifies the physical slide container

### Image Dimensions

- **TotalPixelMatrixColumns**: Total width of the entire slide image in pixels
- **TotalPixelMatrixRows**: Total height of the entire slide image in pixels
- **Columns**: Width of individual frames/tiles
- **Rows**: Height of individual frames/tiles
- **NumberOfFrames**: Number of tiles/frames in this DICOM instance

### Spatial Information

- **FrameOfReferenceUID**: Links images that share the same coordinate system
- **PixelSpacing**: Physical size of each pixel (e.g., 0.00025 mm = 0.25 microns)
- **PlanePositionSlideSequence**: Position of each tile in the total pixel matrix

### Image Type

The `ImageType` attribute is an array with three values:

- **Value 1**: `ORIGINAL` (from scanner) or `DERIVED` (processed)
- **Value 2**: `PRIMARY` (main) or `SECONDARY` (supplementary)
- **Value 3**: Image flavor - `VOLUME`, `THUMBNAIL`, `OVERVIEW`, or `LABEL`

Example: `["ORIGINAL", "PRIMARY", "VOLUME"]`

#### Image Flavors

- **VOLUME**: High-resolution image data that forms the pyramid (multiple resolution levels)
- **OVERVIEW**: Low-resolution image (1,000-5,000 px) for navigation
- **THUMBNAIL**: Small preview (200-500 px) for browsing slide lists
- **LABEL**: Image of the physical slide label (patient ID, etc.)

**Key point**: VOLUME images form the pyramid. Multiple VOLUME instances at different resolutions create the pyramid structure. THUMBNAIL images may be included as a fallback if no equivalent VOLUME image exists at that resolution. Other image types (OVERVIEW, LABEL) are standalone images, not part of the pyramid.

## Image Pyramid Structure

### How Pyramids Work

A pyramid consists of multiple DICOM instances, each representing a different resolution level:

```
Level 0 (highest resolution): 100,000 x 100,000 pixels
Level 1:                      50,000 x 50,000 pixels
Level 2:                      25,000 x 25,000 pixels
Level 3:                      12,500 x 12,500 pixels
```

All levels share:

- Same `FrameOfReferenceUID`
- Same `ContainerIdentifier`
- Different `TotalPixelMatrixColumns` and `TotalPixelMatrixRows`

### Computing Pyramid Levels

The viewer automatically computes pyramid structure from metadata. The process:

1. **Sort instances by size** (largest to smallest = highest to lowest resolution)
2. **Validate consistency**: All levels must share:
   - Same `FrameOfReferenceUID` (same coordinate system)
   - Same `ContainerIdentifier` (same physical slide)
   - Same number of optical paths/channels
3. **Calculate pyramid properties** for each level:
   - **Tile grid size**: How many tiles per row/column (e.g., 1000x1000 tiles)
   - **Resolution factor**: Zoom level relative to base (e.g., Level 1 = 2x zoom out)
   - **Pixel spacing**: Physical size per pixel (e.g., 0.25 microns)

**Result**: A pyramid structure with multiple resolution levels, each ready for tile-based rendering.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Simplified pyramid computation
function computePyramid(metadata) {
  // Sort by size (largest = highest resolution)
  metadata.sort((a, b) => 
    b.TotalPixelMatrixColumns - a.TotalPixelMatrixColumns
  );
  
  // Validate all levels share same identifiers
  const base = metadata[0];
  metadata.forEach(level => {
    if (level.FrameOfReferenceUID !== base.FrameOfReferenceUID) {
      throw new Error('Pyramid levels must share FrameOfReferenceUID');
    }
  });
  
  // Calculate properties for each level
  return metadata.map((level, index) => ({
    resolution: base.TotalPixelMatrixColumns / level.TotalPixelMatrixColumns,
    gridSize: [
      Math.ceil(level.TotalPixelMatrixColumns / level.Columns),
      Math.ceil(level.TotalPixelMatrixRows / level.Rows)
    ],
    pixelSpacing: getPixelSpacing(level)
  }));
}
```
</details>

## Frame Mapping: How Tiles are Organized

Each tile needs to be mapped to its position in the grid. The frame mapping creates a lookup table: `"row-column-opticalPath" → frame number`.

**Key concept**: Frame mapping depends on `DimensionOrganizationType`:

- **TILED_FULL**: Frames are organized in a predictable order (optical paths → rows → columns). Frame numbers follow a mathematical pattern.
- **TILED_SPARSE**: Each frame's position is explicitly stored in `PlanePositionSlideSequence`. More flexible but requires metadata lookup.

**Example mapping** (TILED_FULL):
- Tile at row 1, column 1, optical path "0" → Frame 1
- Tile at row 1, column 2, optical path "0" → Frame 2  
- Tile at row 1, column 1, optical path "1" → Frame 101 (if 100 tiles per optical path)

This mapping is used to construct WADO-RS URLs to retrieve specific frames.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Simplified frame mapping (TILED_FULL example)
function createFrameMapping(metadata) {
  const tileRows = Math.ceil(totalRows / tileHeight);
  const tileCols = Math.ceil(totalCols / tileWidth);
  const numOpticalPaths = metadata.OpticalPathSequence.length;
  const mapping = {};
  
  let frameNumber = 1;
  // Order: optical paths → rows → columns
  for (let opticalPathIdx = 0; opticalPathIdx < numOpticalPaths; opticalPathIdx++) {
    const opticalPathId = metadata.OpticalPathSequence[opticalPathIdx].OpticalPathIdentifier;
    for (let row = 0; row < tileRows; row++) {
      for (let col = 0; col < tileCols; col++) {
        const key = `${row + 1}-${col + 1}-${opticalPathId}`;
        mapping[key] = `${sopInstanceUID}/frames/${frameNumber}`;
        frameNumber++;
      }
    }
  }
  return mapping;
}
```
</details>

## Optical Paths (Channels)

WSI can have multiple **optical paths** (channels) representing different ways the slide was imaged:

- **Brightfield**: H&E staining (most common)
- **Fluorescence**: Multiple channels (DAPI, FITC, Rhodamine, etc.)
- **Multiplexed imaging**: Many channels (CycIF - Cyclic Immunofluorescence)

### How Optical Paths are Stored

**Important**: Optical paths are **NOT separate images or separate series**. They are stored as **separate frames within the same DICOM instance** (same `SOPInstanceUID` and `SeriesInstanceUID`).

- All optical paths share the same DICOM instance
- Each optical path is identified by an `OpticalPathIdentifier` in the `OpticalPathSequence` metadata
- Frames are organized by: tile position (row-column) + optical path identifier
- Frame mapping key format: `"row-column-opticalPath"` (e.g., `"1-2-0"` = row 1, column 2, optical path identifier "0")

**Example**: A fluorescence slide with 3 optical paths (DAPI, FITC, Rhodamine) stored in one DICOM instance:

- Same `SeriesInstanceUID` and `SOPInstanceUID` for all optical paths
- Each tile position has 3 frames (one per optical path)
- Frame mapping: `"1-1-0"` → DAPI (optical path "0"), `"1-1-1"` → FITC (optical path "1"), `"1-1-2"` → Rhodamine (optical path "2")
- All optical paths retrieved from the same instance: `/instances/{SOPInstanceUID}/frames/{FrameNumber}`

## DICOMweb: How Images are Retrieved

DICOMweb is a RESTful API for accessing DICOM data over HTTP. Key endpoints:

- **QIDO-RS** (Query): Search for studies/series/instances
- **WADO-RS** (Retrieve): Retrieve metadata and pixel data
- **STOW-RS** (Store): Store new DICOM objects

Example WADO-RS URL to retrieve a frame:

```
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/{FrameNumber}
```

## Compression

WSI images are typically compressed using:

- **JPEG**: Lossy compression for color images
- **JPEG 2000**: Better compression, supports lossless
- **JPEG-LS**: Lossless compression

The viewer supports all these formats and decodes them client-side.

## Next Steps

- [How dicom-microscopy-viewer Works](./03-dicom-microscopy-viewer-engine.md) - The rendering engine
- [How Slim Viewer Works](./04-slim-viewer-application.md) - The application layer
