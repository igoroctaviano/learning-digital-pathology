# Architecture Overview: How Everything Works Together

> **Note**: This document shows how all components fit together. Code examples demonstrate integration points but can be skipped - focus on the diagrams and conceptual flow.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         User Browser                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Slim (React Application)                │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌───────────--┐ │   │
│  │  │ CaseViewer   │  │ SlideViewer  │  │ UI Controls │ │   │
│  │  └──────┬───────┘  └──────┬───────┘  └───────────--┘ │   │
│  │         │                 │                          │   │
│  │         └──────────┬──────┘                          │   │
│  │                    │                                 │   │
│  │  ┌─────────────────▼──────────────────────────────┐  │   │
│  │  │    dicom-microscopy-viewer (Library)           │  │   │
│  │  │  ┌──────────────────────────────────────────┐  │  │   │
│  │  │  │ VolumeImageViewer                        │  │  │   │
│  │  │  │  - Pyramid computation                   │  │  │   │
│  │  │  │  - Tile loading                          │  │  │   │
│  │  │  │  - OpenLayers rendering                  │  │  │   │
│  │  │  └──────────────────────────────────────────┘  │  │   │
│  │  └────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                    │                                        │
│                    │ HTTP (DICOMweb)                        │
│                    ▼                                        │
└─────────────────────────────────────────────────────────────┘
                    │
                    ▼
┌───────────────────────────────────────────────────────────────┐
│              DICOMweb Server (PACS/VNA)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   QIDO-RS    │  │   WADO-RS    │  │   STOW-RS    │         │
│  │   (Query)    │  │  (Retrieve)  │  │   (Store)    │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└───────────────────────────────────────────────────────────────┘
```

## Data Flow

### 1. Initial Load

```
User → Slim → DICOMweb (QIDO-RS) → Metadata
                ↓
         Slim organizes slides
                ↓
         User selects slide
                ↓
         Slim → dicom-microscopy-viewer
                ↓
         Viewer computes pyramid
                ↓
         Viewer renders initial view
```

### 2. Tile Loading (On-Demand)

```
User pans/zooms
    ↓
OpenLayers detects visible tiles
    ↓
dicom-microscopy-viewer requests tiles
    ↓
DICOMweb (WADO-RS) retrieves frames
    ↓
Tiles decoded (JPEG/JPEG2000)
    ↓
Tiles rendered on map
```

### 3. Annotation Creation

```
User draws annotation
    ↓
Slim captures geometry
    ↓
Slim creates DICOM SR structure
    ↓
Slim → DICOMweb (STOW-RS)
    ↓
Annotation stored in archive
    ↓
Annotation displayed on viewer
```

## Component Responsibilities

### Slim Responsibilities

- **Study/Series Navigation**: Finding and organizing slides
- **User Interface**: Buttons, menus, sidebars
- **Annotation Tools**: Drawing tools, measurement tools
- **Authentication**: OAuth/OIDC flow
- **Configuration**: Server URLs, annotation styles
- **Error Handling**: User-friendly error messages
- **State Management**: React state, routing

### dicom-microscopy-viewer Responsibilities

- **Metadata Parsing**: Converting DICOM JSON to objects
- **Pyramid Computation**: Building multi-resolution structure
- **Tile Loading**: Fetching frames from DICOMweb
- **Decoding**: JPEG/JPEG2000/JPEG-LS decompression
- **Rendering**: OpenLayers map rendering
- **Coordinate Transformation**: Pixel ↔ Physical coordinates
- **Annotation Display**: Rendering ROI overlays

### DICOMweb Server Responsibilities

- **Storage**: Storing DICOM objects
- **Query**: Finding studies/series/instances (QIDO-RS)
- **Retrieve**: Returning metadata and pixel data (WADO-RS)
- **Store**: Accepting new DICOM objects (STOW-RS)

## Integration Points

### How Slim Uses dicom-microscopy-viewer

Slim integrates with dicom-microscopy-viewer in three simple steps:

1. **Import the library**: Loads the dicom-microscopy-viewer JavaScript library
2. **Create viewer instance**: Instantiates `VolumeImageViewer` with:
   - Slide metadata (VOLUME images)
   - DICOMweb client(s) for data retrieval
   - Configuration options (preload, cache size, etc.)
3. **Render**: Calls `render()` to display the viewer in an HTML element

**Key point**: Slim provides the UI and workflow; dicom-microscopy-viewer handles all the rendering complexity.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
// 1. Import the library
import * as dmv from "dicom-microscopy-viewer";

// 2. Create viewer instance
const volumeViewer = new dmv.viewer.VolumeImageViewer({
  clientMapping: clients,
  metadata: slide.volumeImages,
  controls: ["overview", "position"],
  skipThumbnails: true,
  preload,
  errorInterceptor: (error: CustomError) => {
    NotificationMiddleware.onError(NotificationMiddlewareContext.DMV, error);
  },
});

// 3. Render in DOM element
volumeViewer.render({
  container: viewportRef.current,
});
```
</details>

### How dicom-microscopy-viewer Uses DICOMweb

When dicom-microscopy-viewer needs a tile, it:

1. **Constructs request**: Builds a DICOMweb WADO-RS request with:
   - Study/Series/Instance UIDs
   - Frame number(s)
   - Preferred media types (JPEG 2000, JPEG, JPEG-LS)
2. **Sends HTTP request**: Uses the dicomweb-client library to make the request
3. **Receives response**: Gets compressed pixel data
4. **Decodes**: Decompresses the frame using appropriate decoder

**Key point**: All communication uses standard DICOMweb RESTful APIs - no custom protocols.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Inside dicom-microscopy-viewer
const retrieveOptions = {
  studyInstanceUID,
  seriesInstanceUID,
  sopInstanceUID,
  frameNumbers,
  mediaTypes
}

// Uses dicomweb-client library
return client.retrieveInstanceFrames(retrieveOptions)
  .then((rawFrames) => {
    // Decode frames
    return _decodeAndTransformFrame({...})
  })
```
</details>

## Key Design Decisions

### Why Separate Library and Application?

1. **Reusability**: Other applications can use dicom-microscopy-viewer
2. **Separation of Concerns**: Rendering logic separate from UI logic
3. **Testing**: Library can be tested independently
4. **Maintenance**: Changes to UI don't affect rendering engine

### Why OpenLayers?

OpenLayers is a mapping library that provides:

- **Tile-based rendering**: Perfect for WSI tiles
- **Multi-resolution support**: Built-in pyramid handling
- **Pan/zoom interactions**: Out-of-the-box
- **Coordinate systems**: Handles transformations
- **Performance**: Optimized for large datasets

### Why DICOMweb?

- **Standard**: Works with existing medical imaging infrastructure
- **RESTful**: Easy to use over HTTP
- **Scalable**: Can handle large datasets
- **Interoperable**: Works with any DICOMweb-compliant server

## Performance Optimizations

### 1. Lazy Loading

Tiles are only loaded when visible:

- OpenLayers calculates visible tiles
- Only those tiles are requested
- Tiles outside viewport are not loaded

### 2. Caching

Recently loaded tiles are cached:

- Avoids re-fetching same tiles
- Configurable cache size (default: 1000 tiles)
- Cache eviction when limit reached

### 3. Preloading

Optional preloading of pyramid levels:

- Can preload lower resolution levels
- Speeds up initial zoom operations
- Configurable via `preload` option

### 4. Multi-resolution

Automatic level switching:

- High zoom → High resolution tiles
- Low zoom → Low resolution tiles
- Smooth transitions between levels

## Error Handling

### Three Layers

1. **dicom-microscopy-viewer**: Throws errors, publishes events
2. **Slim**: Catches errors, shows user-friendly messages
3. **DICOMweb**: Returns HTTP error codes

### Error Flow

```
DICOMweb error
    ↓
dicom-microscopy-viewer catches
    ↓
Publishes error event
    ↓
Slim error interceptor receives
    ↓
Notification middleware displays
    ↓
User sees friendly error message
```

## Extension Points

### Adding New Features

1. **New Image Type**: Extend `metadata.js` in dicom-microscopy-viewer
2. **New Annotation Type**: Extend `annotation.js` in dicom-microscopy-viewer
3. **New UI Component**: Add React component in Slim
4. **New DICOM Type**: Add support in both libraries

## Next Steps

- [Key Concepts](./06-key-concepts.md) - Important technical concepts
- [Common Use Cases](./07-common-use-cases.md) - Real-world scenarios
