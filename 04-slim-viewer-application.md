# Slim: The Viewer Application

> **Note**: This document explains how Slim works as an application. Code examples show implementation details but can be skipped - the conceptual explanations provide the essential understanding.

## Overview

**Slim** is a single-page React application that provides a complete pathology viewer interface. It uses `dicom-microscopy-viewer` as its rendering engine and adds:

- User interface (UI components)
- Study/series navigation
- Annotation tools
- Patient information display
- Authentication/authorization
- DICOMweb client management

## Architecture

### Component Hierarchy

```
App
└── CaseViewer
    ├── SlideList (sidebar)
    └── SlideViewer
        ├── VolumeImageViewer (from dicom-microscopy-viewer)
        ├── LabelImageViewer (from dicom-microscopy-viewer)
        ├── Annotation Tools
        └── Sidebar (metadata, annotations)
```

### Key Components

#### CaseViewer

The main container component that manages study/series selection, slide discovery, and routing.

**What it does**:
- **Fetches slides**: Uses DICOMweb QIDO-RS to find all series in a study, then groups images by `ContainerIdentifier` to identify slides
- **Displays sidebar**: Shows patient info, study info, and a list of slides
- **Handles navigation**: When user selects a slide, updates the URL and displays the `SlideViewer` component
- **Manages routing**: Uses React Router to handle URL changes (e.g., `/studies/{UID}/series/{UID}`)

**Key concept**: This component is the "orchestrator" - it doesn't render images itself, but coordinates which slide to display and manages the overall application state.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
function CaseViewer({ clients, studyInstanceUID }) {
  // Fetches and organizes slides from DICOMweb
  const { slides } = useSlides({ clients, studyInstanceUID });
  
  // Handles series selection and updates URL
  const handleSeriesSelection = (seriesInstanceUID) => {
    navigate(`/studies/${studyInstanceUID}/series/${seriesInstanceUID}`);
  };
  
  return (
    <Layout>
      <Sidebar>
        <PatientInfo />
        <StudyInfo />
        <SlideList 
          slides={slides}
          onSelect={handleSeriesSelection}
        />
      </Sidebar>
      <Routes>
        <Route path="/series/:seriesInstanceUID" 
               element={<SlideViewer slides={slides} />} />
      </Routes>
    </Layout>
  );
}
```
</details>

#### SlideViewer

The main viewer component that displays a slide and provides interaction tools.

**What it does**:
- **Creates viewers**: Instantiates `VolumeImageViewer` (from dicom-microscopy-viewer) to render the main slide image, and optionally `LabelImageViewer` to show the slide label
- **Manages annotations**: Handles loading, displaying, creating, and saving ROI annotations
- **Provides UI controls**: Buttons for drawing tools, measurement tools, channel selection, etc.
- **Handles interactions**: Responds to user actions (pan, zoom, draw, select, etc.)

**Key concept**: This is where dicom-microscopy-viewer (the engine) meets Slim (the application). Slim provides the UI and workflow, while dicom-microscopy-viewer handles the actual rendering.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
function constructViewers({ clients, slide, preload }) {
  // Create main volume viewer
  const volumeViewer = new dmv.viewer.VolumeImageViewer({
    clientMapping: clients,
    metadata: slide.volumeImages,
    controls: ['overview', 'position'],
    preload
  });
  
  // Optionally create label viewer if available
  const labelViewer = slide.labelImages.length > 0
    ? new dmv.viewer.LabelImageViewer({
        client: clients[StorageClasses.VL_WHOLE_SLIDE_MICROSCOPY_IMAGE],
        metadata: slide.labelImages[0]
      })
    : undefined;
  
  return { volumeViewer, labelViewer };
}
```
</details>

## Slide Data Model

Slim organizes images into `Slide` objects. A slide represents a physical glass slide and contains all related DICOM images.

**Key attributes**:
- **ContainerIdentifier**: Links images from the same physical slide (all image types share this)
- **FrameOfReferenceUID**: Links images in the same coordinate system (important for annotations)
- **Image collections**: Separated by image type:
  - `volumeImages`: VOLUME images that form the pyramid
  - `labelImages`: LABEL images showing the slide label
  - `overviewImages`: OVERVIEW images for navigation
  - `thumbnailImages`: THUMBNAIL images for browsing

**Why this matters**: When a user selects a slide, Slim finds all related images (VOLUME, LABEL, OVERVIEW, THUMBNAIL) and organizes them into this structure. The viewer then uses the VOLUME images to build the pyramid.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
class Slide {
  containerIdentifier: string;        // Links images from same physical slide
  frameOfReferenceUID: string;        // Links images in same coordinate system
  volumeImages: VLWholeSlideMicroscopyImage[];  // Main pyramid images
  labelImages: VLWholeSlideMicroscopyImage[];   // Slide label images
  overviewImages: VLWholeSlideMicroscopyImage[]; // Navigation images
  thumbnailImages: VLWholeSlideMicroscopyImage[]; // Preview images
}
```
</details>

## DICOMweb Client Management

Slim can connect to multiple DICOMweb servers, allowing different DICOM object types to be stored in different archives.

**Why multiple clients?**
- **Separation of concerns**: Images might be in one archive, annotations in another
- **Performance**: Different servers optimized for different data types
- **Security**: Annotations might require different access controls
- **Scalability**: Distribute load across multiple servers

**How it works**:
- Slim maintains a mapping: SOP Class UID → DICOMweb client
- When retrieving data, Slim uses the appropriate client based on the DICOM object type
- If one client fails, Slim can try others (fallback behavior)

**Example**: Images stored in primary archive, annotations in secondary archive. Slim automatically uses the correct client for each.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
// Different DICOM types can be stored in different archives
const clients = {
  default: imageClient,                    // For images
  [StorageClasses.COMPREHENSIVE_3D_SR]: annotationClient,  // For annotations
  [StorageClasses.SEGMENTATION]: segmentationClient        // For segmentations
};

// Try each client to find metadata
for (const client of Object.values(clients)) {
  const metadata = await client.retrieveSeriesMetadata({...});
  if (metadata.length > 0) break;
}
```
</details>

## Annotation Features

### Creating Annotations

Slim provides tools for creating ROI (Region of Interest) annotations:

- Point
- Polygon
- Ellipse
- Freehand drawing

Annotations are stored as DICOM Comprehensive 3D SR instances.

### Displaying Annotations

Slim can display:

- **DICOM SR**: Structured reports with ROI annotations
- **DICOM Segmentation**: Binary masks (e.g., nuclei segmentation)
- **DICOM Parametric Map**: Heat maps (e.g., attention maps)
- **DICOM Bulk Annotations**: Large collections of annotations (e.g., cell annotations)

## Configuration

Slim is configured via JavaScript files in `public/config/`:

```javascript
window.config = {
  path: "/",
  servers: [
    {
      id: "local",
      url: "http://localhost:8008/dcm4chee-arc/aets/DCM4CHEE/rs",
      write: true,
    },
  ],
  annotations: [
    {
      finding: {
        value: "85756007",
        schemeDesignator: "SCT",
        meaning: "Tissue",
      },
      style: {
        stroke: {
          color: [251, 134, 4, 1],
          width: 2,
        },
        fill: {
          color: [255, 255, 255, 0.2],
        },
      },
    },
  ],
};
```

## Authentication

Slim supports OAuth 2.0 / OpenID Connect for authentication:

- Authorization code grant with PKCE
- Implicit grant (legacy)
- Google Cloud Healthcare API integration

## Deployment

Slim is a static web application:

1. Build: `yarn build`
2. Deploy: Copy `build/` folder to any static web server
3. Configure: Set `REACT_APP_CONFIG` environment variable

No server-side components required!

## Key Differences from dicom-microscopy-viewer

| Feature            | dicom-microscopy-viewer    | Slim                    |
| ------------------ | -------------------------- | ----------------------- |
| **Purpose**        | Rendering engine (library) | Complete application    |
| **Framework**      | Vanilla JS                 | React + TypeScript      |
| **UI**             | None                       | Full UI with Ant Design |
| **Navigation**     | None                       | Study/series navigation |
| **Annotations**    | Display only               | Create + Display        |
| **Authentication** | None                       | OAuth 2.0 / OIDC        |
| **Configuration**  | Programmatic               | Config files            |

## Workflow Example

1. **User opens Slim**: Navigates to study URL
2. **Slim fetches metadata**: Uses DICOMweb QIDO-RS to find series
3. **Slim organizes slides**: Groups images by ContainerIdentifier
4. **User selects slide**: Clicks on slide in sidebar
5. **Slim creates viewer**: Instantiates `VolumeImageViewer` with slide metadata
6. **Viewer renders**: dicom-microscopy-viewer loads and displays tiles
7. **User interacts**: Pans, zooms, creates annotations
8. **Slim saves annotations**: Stores as DICOM SR via STOW-RS

## Next Steps

- [Architecture Overview](./05-architecture-overview.md) - How everything fits together
- [Key Concepts](./06-key-concepts.md) - Important technical concepts
