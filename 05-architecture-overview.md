# Architecture Overview: How Everything Works Together

> **Note**: This document shows how all components fit together. Code examples demonstrate integration points but can be skipped - focus on the diagrams and conceptual flow.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Slim Application                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ CaseViewer   │  │ SlideViewer  │  │ Annotations  │    │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
│         │                 │                  │             │
│         └─────────────────┼──────────────────┘             │
│                           │                                │
│                           ▼                                │
│              ┌──────────────────────────┐                  │
│              │ dicom-microscopy-viewer  │                  │
│              │  VolumeImageViewer       │                  │
│              └────────────┬──────────────┘                  │
└───────────────────────────┼────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  DICOMweb API   │
                    │  (QIDO-RS,      │
                    │   WADO-RS)      │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  DICOM Server    │
                    │  (PACS/Archive)  │
                    └─────────────────┘
```

## Component Responsibilities

### Slim (Application Layer)

- **Study/Series Management**: Browse and select studies/series
- **Slide Organization**: Group images into slides
- **UI Components**: Buttons, menus, sidebars, modals
- **Annotation Tools**: Create, edit, save ROI annotations
- **Multi-Server**: Manage connections to multiple DICOMweb servers
- **Authentication**: Handle OAuth 2.0 / OIDC

### dicom-microscopy-viewer (Rendering Engine)

- **Metadata Parsing**: Parse and validate DICOM VL WSI metadata
- **Pyramid Computation**: Build multi-resolution pyramid structure
- **Tile Loading**: Request tiles via DICOMweb WADO-RS
- **Pixel Decoding**: Decompress JPEG/JPEG2000/JPEG-LS
- **Rendering**: Display tiles using OpenLayers
- **Coordinate Transformations**: Convert between pixel, physical, and screen coordinates
- **ICC Profiles**: Apply color correction

### DICOMweb (Communication Layer)

- **QIDO-RS**: Query for studies, series, instances
- **WADO-RS**: Retrieve metadata and pixel data
- **STOW-RS**: Store new DICOM objects

## Data Flow: Viewing a Slide

```
┌─────────────────────────────────────────────────────────┐
│           Data Flow: Viewing a Slide                    │
└─────────────────────────────────────────────────────────┘

User Action          System Component          DICOM Server
─────────────────────────────────────────────────────────────

1. Select study
   ┌──────────┐
   │CaseViewer│ ──QIDO-RS Query──→ ┌──────────────┐
   └──────────┘                    │ DICOM Server │
                                   └──────────────┘

2. Discover slides
   ┌──────────┐
   │   Slim   │ Groups by ContainerIdentifier
   └────┬─────┘         FrameOfReferenceUID

3. Select slide
   ┌───────────┐
   │SlideViewer│ Mounts
   └────┬──────┘

4. Create Slide object
   ┌──────────┐
   │   Slim   │ Organizes: VOLUME, LABEL, OVERVIEW, THUMBNAIL
   └────┬─────┘

5. Create viewer
   ┌──────────┐
   │   Slim   │ ──Instantiates──→ VolumeImageViewer
   └──────────┘

6. Compute pyramid
   ┌────────────────┐
   │dicom-microscopy│ Analyzes metadata
   │   -viewer      │ Builds pyramid structure
   └──────┬─────────┘

7. Setup tile loading
   ┌──────────────┐
   │dicom-microscopy│ Creates tile loaders
   │   -viewer    │
   └──────┬───────┘

8. User pans/zooms
   ┌──────────┐
   │OpenLayers│ Requests visible tiles
   └────┬─────┘
        │
9. Load tiles        │
   ┌────▼─────┐      │
   │  Viewer  │ ──WADO-RS──→ ┌──────────────┐
   └──────────┘              │ DICOM Server │
                             └──────────────┘

10. Decode pixels
    ┌────────────────┐
    │dicom-microscopy│ Decompresses (JPEG/JPEG2000)
    │   -viewer      │ Applies color correction
    └──────┬─────────┘

11. Render tiles
    ┌──────────┐
    │OpenLayers│ Displays tiles on screen
    └──────────┘
```

**Steps**:
1. **User selects study**: CaseViewer queries DICOMweb QIDO-RS for series
2. **Slim discovers slides**: Groups images by `ContainerIdentifier` and `FrameOfReferenceUID`
3. **User selects slide**: SlideViewer component mounts
4. **Slim creates Slide object**: Organizes images by type (VOLUME, LABEL, OVERVIEW, THUMBNAIL)
5. **Slim creates viewer**: Instantiates `VolumeImageViewer` with slide metadata
6. **Viewer computes pyramid**: Analyzes metadata to build multi-resolution structure
7. **Viewer sets up tile loading**: Creates tile loaders for each pyramid level
8. **User pans/zooms**: OpenLayers requests visible tiles
9. **Viewer loads tiles**: Requests frames via DICOMweb WADO-RS
10. **Viewer decodes pixels**: Decompresses and applies color correction
11. **Tiles render**: OpenLayers displays tiles on screen

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

### Separation of Concerns

- **Slim**: Application logic, UI, workflows
- **dicom-microscopy-viewer**: Rendering, tile management, DICOM parsing
- **DICOMweb**: Data access, storage

### Why OpenLayers?

- **Proven technology**: Battle-tested for tile-based rendering
- **Multi-resolution**: Built-in support for zoom levels
- **Performance**: Efficient tile caching and management
- **Extensibility**: Easy to customize for DICOM-specific needs

### Why DICOMweb?

- **Standard**: Official DICOM standard for web-based access
- **RESTful**: Simple HTTP requests, easy to debug
- **Interoperable**: Works with any DICOMweb-compliant server
- **Scalable**: Can distribute across multiple servers

### Why Separate Annotations?

According to [DICOM WSI standards](https://dicom.nema.org/dicom/dicomwsi/), annotations are stored separately because:

- **Different modality**: Annotations are created by different processes
- **Different timing**: Created after image acquisition
- **Different equipment**: May be created on different systems
- **Multiple annotations**: Multiple annotation objects can reference the same image

## Multi-Server Architecture

Slim can connect to multiple DICOMweb servers:

```
┌─────────────┐
│    Slim     │
└──────┬──────┘
       │
       ├──────────────┬──────────────┬──────────────┐
       │              │              │              │
       ▼              ▼              ▼              ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Image Server│ │Annotation   │ │Segmentation │ │  Archive    │
│             │ │   Server     │ │   Server    │ │             │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
```

**Benefits**:
- **Separation**: Different data types in different archives
- **Performance**: Optimize servers for specific workloads
- **Security**: Different access controls per server
- **Scalability**: Distribute load

## Error Handling

- **Slim**: Catches errors, displays user-friendly messages
- **dicom-microscopy-viewer**: Publishes error events
- **DICOMweb**: Returns HTTP error codes

Errors flow: DICOMweb → dicom-microscopy-viewer → Slim → User

## Performance Considerations

### Tile Caching

- **Memory cache**: Recently viewed tiles kept in memory (default: 1000 tiles)
- **LRU eviction**: Least recently used tiles evicted when cache full
- **Preloading**: Lower resolution levels can be preloaded

### Lazy Loading

- **On-demand**: Only visible tiles are loaded
- **Progressive**: Lower resolution loads first, then higher resolution
- **Cancellation**: Cancels requests for tiles that become invisible

### Compression

- **JPEG 2000**: Preferred for better compression
- **JPEG**: Fallback for compatibility
- **JPEG-LS**: Lossless option

## Next Steps

- [Key Concepts](./06-key-concepts.md) - Important technical concepts
- [Common Use Cases](./07-common-use-cases.md) - Real-world scenarios

