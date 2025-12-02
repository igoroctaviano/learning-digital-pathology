# dicom-microscopy-viewer: The Rendering Engine

> **Note**: This document explains how the rendering engine works. Code examples illustrate concepts but can be skipped - focus on the "What" and "Why" sections for conceptual understanding.

## Overview

`dicom-microscopy-viewer` is a **vanilla JavaScript library** (no framework dependencies) that provides the core rendering capabilities for displaying DICOM WSI images. Think of it as the "engine" - it handles the low-level work of:

- Parsing DICOM metadata
- Computing pyramid structures
- Loading tiles on-demand via DICOMweb
- Decoding compressed pixel data
- Rendering tiles using OpenLayers
- Managing coordinate transformations
- Handling ICC profiles for color correction

## Architecture

The library is organized into key modules:

- **Metadata**: Parses and validates DICOM VL Whole Slide Microscopy Image metadata
- **Pyramid**: Computes multi-resolution pyramid structure
- **Viewer**: Main rendering components
   - Groups images by type (VOLUME, THUMBNAIL, OVERVIEW, LABEL)
   - `VolumeImageViewer`: Main viewer for VOLUME images
   - `LabelImageViewer`: Viewer for LABEL images
   - `OverviewImageViewer`: Viewer for OVERVIEW images
   - `ThumbnailImageViewer`: Viewer for THUMBNAIL images
- **Utils**: Helper functions for coordinate transformations, frame mapping, etc.

## VolumeImageViewer: The Main Viewer

The `VolumeImageViewer` class is the heart of the rendering engine. Let's examine how it works:

### Initialization

When creating a `VolumeImageViewer`, you provide:

**Required inputs**:
- **Metadata**: Array of DICOM VL Whole Slide Microscopy Image instances (VOLUME images)
- **DICOMweb client**: Connection to the DICOMweb server (or multiple clients for different storage classes)

**Optional configuration**:
- **Preload**: Whether to preload lower resolution levels for faster initial zoom
- **Cache size**: How many tiles to keep in memory (default: 1000)
- **Skip thumbnails**: Whether to exclude THUMBNAIL images from the pyramid
- **Controls**: UI controls to include (e.g., "overview", "position")
- **Error interceptor**: Callback for handling errors

**What happens during initialization**:
1. **Validates metadata**: Ensures all images are valid VOLUME instances
2. **Groups by optical path**: Separates monochrome and color images by optical path identifier
3. **Computes pyramid**: Analyzes metadata to build the multi-resolution structure (see below)
4. **Sets up DICOMweb client(s)**: Configures connection(s) to retrieve data
5. **Initializes OpenLayers map**: Creates the tile-based rendering surface
6. **Creates tile loaders**: Sets up functions to load tiles on-demand for each pyramid level

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
class VolumeImageViewer {
  constructor(options) {
    // Required: metadata array and DICOMweb client
    this.metadata = options.metadata; // Array of VLWholeSlideMicroscopyImage
    this.client = options.client || options.clientMapping.default;
    
    // Optional: performance and behavior
    this.preload = options.preload || false;
    this.tilesCacheSize = options.tilesCacheSize || 1000;
    this.skipThumbnails = options.skipThumbnails || false;
    
    // Computes pyramid structure from metadata
    this.pyramid = computeImagePyramid({ metadata: options.metadata });
    
    // Sets up OpenLayers map for tile rendering
    this.map = new Map({ ... });
  }
}
```
</details>

### Pyramid Computation

The pyramid computation process (implemented in `_computeImagePyramid`):

1. **Sorts instances by size**: Arranges from smallest to largest (lowest to highest resolution)
2. **Validates consistency**: Ensures all instances share:
   - Same `FrameOfReferenceUID` (same coordinate system)
   - Same `ContainerIdentifier` (same physical slide)
   - Same number of optical paths/channels
3. **Handles concatenation**: Detects and reassembles instances split across multiple files
4. **Computes frame mappings**: Creates tile-to-frame mappings for each pyramid level
5. **Calculates properties**: For each level, computes:
   - Resolution factors (zoom levels)
   - Tile grid dimensions
   - Pixel spacings
   - Physical sizes
   - Extents (bounding boxes)

**Result**: A structured pyramid object ready for multi-resolution rendering.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Simplified pyramid computation
function _computeImagePyramid({ metadata }) {
  // Sort instances by size (smallest to largest)
  metadata.sort((a, b) => {
    const sizeDiff = a.TotalPixelMatrixColumns - b.TotalPixelMatrixColumns;
    if (sizeDiff === 0) {
      // Handle concatenation parts
      if (a.ConcatenationFrameOffsetNumber !== undefined) {
        return a.ConcatenationFrameOffsetNumber - b.ConcatenationFrameOffsetNumber;
      }
    }
    return sizeDiff;
  });
  
  // Validate consistency
  const base = metadata[0];
  metadata.forEach(level => {
    if (level.FrameOfReferenceUID !== base.FrameOfReferenceUID) {
      throw new Error('Pyramid levels must share FrameOfReferenceUID');
    }
    if (level.ContainerIdentifier !== base.ContainerIdentifier) {
      throw new Error('Pyramid levels must share ContainerIdentifier');
    }
  });
  
  // Compute properties for each level
  return {
    metadata: pyramidMetadata,      // DICOM instances for each level
    frameMappings: pyramidFrameMappings, // Tile-to-frame mappings
    resolutions: pyramidResolutions,     // Zoom factors
    gridSizes: pyramidGridSizes,         // Tile grid dimensions
    pixelSpacings: pyramidPixelSpacings, // Physical pixel sizes
    numberOfChannels: numberOfChannels   // Number of optical paths
  };
}
```
</details>

### How Tiles are Loaded

When the viewer needs to display a tile, here's what happens:

1. **Frame lookup**: Uses the frame mapping to find which frame number corresponds to the tile position (row-column-opticalPath)
2. **Construct WADO-RS URL**: Builds the DICOMweb URL to retrieve that specific frame
   - Format: `/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/{FrameNumber}`
3. **Request frame**: Sends HTTP request to DICOMweb server with preferred compression formats (JPEG 2000, JPEG, JPEG-LS)
4. **Decode pixel data**: Decompresses the frame using the appropriate decoder (JPEG/JPEG2000/JPEG-LS)
5. **Apply color correction**: If it's a color image, applies ICC profile to ensure accurate colors
6. **Return pixel data**: Provides the decoded pixels to OpenLayers for rendering

**Result**: The tile appears on screen. This happens automatically for each visible tile as the user pans and zooms.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Simplified tile loading flow
async function loadTile(z, y, x, opticalPathId) {
  // 1. Look up frame mapping: "row-column-opticalPath" → frame number
  const key = `${y + 1}-${x + 1}-${opticalPathId}`;
  const framePath = pyramid.frameMappings[z][key];
  
  // 2. Construct WADO-RS URL
  const url = `/studies/${studyUID}/series/${seriesUID}/instances/${framePath}`;
  
  // 3. Request frame with preferred compression formats
  const frame = await client.retrieveInstanceFrames({
    studyInstanceUID,
    seriesInstanceUID,
    sopInstanceUID,
    frameNumbers: [frameNumber],
    mediaTypes: ['image/jp2', 'image/jpeg', 'image/jls'] // Preferred formats
  });
  
  // 4. Decode compressed pixel data
  const pixelArray = decodeFrame(frame[0], {
    compression: 'JPEG2000', // or JPEG, JPEG-LS
    bitsAllocated: 8,
    samplesPerPixel: 3
  });
  
  // 5. Apply ICC profile if color image
  if (samplesPerPixel === 3 && iccProfile) {
    return applyICCProfile(pixelArray, iccProfile);
  }
  
  return pixelArray;
}
```
</details>

### Key Features

1. **Multi-resolution rendering**: Automatically switches between pyramid levels based on zoom
2. **On-demand tile loading**: Only loads visible tiles
3. **Tile caching**: Caches recently viewed tiles to avoid re-downloading
4. **Multi-channel**: Supports multiple optical paths/channels
5. **Coordinate transformations**: Converts between pixel, physical (slide), and screen coordinates
6. **ICC profiles**: Applies color correction for accurate color display
7. **Event publishing**: Publishes events for frame loading, viewport changes, etc.

## Using the Library

### Basic Workflow

To use dicom-microscopy-viewer, you follow these steps:

1. **Connect to DICOMweb server**: Create a client pointing to your DICOMweb endpoint
2. **Retrieve metadata**: Use QIDO-RS to get series metadata (all DICOM instances in the series)
3. **Filter VOLUME images**: Extract only VOLUME image instances (they form the pyramid)
4. **Create viewer**: Instantiate `VolumeImageViewer` with the metadata and client
5. **Render**: Call `render()` to display the viewer in an HTML element

The viewer then automatically:
- Computes the pyramid structure
- Sets up tile loading
- Handles pan/zoom interactions
- Manages multi-resolution switching

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
import * as DICOMMicroscopyViewer from "dicom-microscopy-viewer";
import * as DICOMwebClient from "dicomweb-client";

// 1. Create a DICOMweb client
const client = new DICOMwebClient.api.DICOMwebClient({
  url: "http://localhost:8080/dicomweb",
});

// 2. Retrieve metadata
const metadata = await client.retrieveSeriesMetadata({
  studyInstanceUID: "1.2.3.4",
  seriesInstanceUID: "1.2.3.5",
});

// 3. Parse and filter metadata (keep only VOLUME images for pyramid)
const volumeImages = [];
metadata.forEach((m) => {
  const image = new DICOMMicroscopyViewer.metadata.VLWholeSlideMicroscopyImage({
    metadata: m,
  });
  const imageFlavor = image.ImageType[2];
  if (imageFlavor === "VOLUME") {
    volumeImages.push(image);
  }
});

// 4. Create viewer instance
const viewer = new DICOMMicroscopyViewer.viewer.VolumeImageViewer({
  client,
  metadata: volumeImages,
});

// 5. Render in HTML element
viewer.render({ container: "viewport" });
```
</details>

## What dicom-microscopy-viewer Does NOT Do

The library is **purposefully minimal** and does NOT include:

- **UI components**: No buttons, menus, or controls (except basic built-in controls)
- **Study/series management**: Doesn't browse studies or series
- **Authentication**: Doesn't handle OAuth or other auth mechanisms
- **Annotation creation**: Can display annotations but doesn't create them
- **Multi-server management**: Basic support but not full multi-server workflows
- **Application framework**: Not a complete application

This is intentional - it's a **rendering engine**, not a complete viewer application.

## OpenLayers Integration

dicom-microscopy-viewer uses [OpenLayers](https://openlayers.org/) for tile-based rendering:

- **Why OpenLayers?**: Provides robust tile management, multi-resolution support, and pan/zoom capabilities
- **Tile sources**: Creates custom tile sources that load tiles via DICOMweb
- **Coordinate systems**: Uses DICOM projection for accurate spatial representation
- **Interactions**: Built-in pan, zoom, and selection tools

## Next Steps

- [How Slim Viewer Works](./04-slim-viewer-application.md) - See how Slim uses this engine
- [Architecture Overview](./05-architecture-overview.md) - How everything fits together

