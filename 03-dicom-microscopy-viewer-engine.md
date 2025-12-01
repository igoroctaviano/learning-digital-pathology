# dicom-microscopy-viewer: The Rendering Engine

> **Note**: This document explains how the rendering engine works. Code examples illustrate concepts but can be skipped - focus on the "What" and "Why" sections for conceptual understanding.

## Overview

`dicom-microscopy-viewer` is a **vanilla JavaScript library** (no framework dependencies) that provides the core rendering capabilities for displaying DICOM WSI images. Think of it as the "engine" - it handles the low-level work of:

- Parsing DICOM metadata
- Computing image pyramids
- Loading and decoding tiles
- Rendering tiles on a map (using OpenLayers)
- Managing coordinate transformations

## Architecture

### Core Components

1. **Metadata Parsing** (`metadata.js`)

   - Parses DICOM JSON metadata
   - Creates `VLWholeSlideMicroscopyImage` objects
   - Groups images by type (VOLUME, THUMBNAIL, OVERVIEW, LABEL)

2. **Pyramid Computation** (`pyramid.js`)

   - Computes pyramid structure from multiple DICOM instances
   - Creates frame mappings (tile position → frame number)
   - Calculates resolutions and extents

3. **Viewers** (`viewer.js`)

   - `VolumeImageViewer`: Main viewer for VOLUME images
   - `OverviewImageViewer`: Static viewer for OVERVIEW images
   - `LabelImageViewer`: Static viewer for LABEL images

4. **Decoding** (`decode.js`)

   - Decodes compressed pixel data (JPEG, JPEG 2000, JPEG-LS)
   - Handles color space transformations
   - Applies ICC profiles for color correction

5. **Annotations** (`annotation.js`, `roi.js`)
   - Manages ROI (Region of Interest) annotations
   - Handles vector graphics (polygons, ellipses, etc.)
   - Supports DICOM SR and Bulk Annotations

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

**What happens during initialization**:
1. **Validates metadata**: Ensures all images are valid VOLUME instances
2. **Computes pyramid**: Analyzes metadata to build the multi-resolution structure
3. **Sets up DICOMweb client(s)**: Configures connection(s) to retrieve data
4. **Initializes OpenLayers map**: Creates the tile-based rendering surface
5. **Creates tile loaders**: Sets up functions to load tiles on-demand for each pyramid level

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

1. **Lazy Loading**: Tiles are only loaded when visible in the viewport
2. **Caching**: Recently loaded tiles are cached to avoid re-fetching
3. **Multi-resolution**: Automatically switches between pyramid levels based on zoom
4. **Multi-channel**: Supports multiple optical paths/channels
5. **Coordinate Transformation**: Converts between pixel coordinates and physical coordinates (millimeters)

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

- ❌ User interface (buttons, menus, etc.)
- ❌ Study/series selection
- ❌ Patient information display
- ❌ Annotation creation UI
- ❌ Authentication/authorization
- ❌ Routing/navigation

These are handled by the application layer (Slim).

## Next Steps

- [How Slim Viewer Works](./04-slim-viewer-application.md) - See how Slim uses this engine
- [Architecture Overview](./05-architecture-overview.md) - How everything fits together
