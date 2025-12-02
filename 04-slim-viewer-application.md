# Slim: The Viewer Application

> **Note**: This document explains how Slim works as an application. Code examples show implementation details but can be skipped - the conceptual explanations provide the essential understanding.

## Overview

**Slim** is a single-page React application that provides a complete pathology viewer interface. It uses `dicom-microscopy-viewer` as its rendering engine and adds:

- **Study/Series management**: Browse and select studies and series
- **Slide organization**: Groups images into slides based on `ContainerIdentifier` and `FrameOfReferenceUID`
- **Annotation tools**: Create, edit, and save ROI annotations
- **Multi-server support**: Connect to multiple DICOMweb servers
- **Authentication**: OAuth 2.0 / OIDC support
- **UI components**: Complete user interface with controls, sidebars, and modals

## Component Hierarchy

```
CaseViewer (main container)
  ├── Sidebar
  │   ├── PatientInfo
  │   ├── StudyInfo
  │   └── SlideList
  └── Routes
      └── SlideViewer
          ├── VolumeImageViewer (from dicom-microscopy-viewer)
          ├── LabelImageViewer (from dicom-microscopy-viewer)
          ├── AnnotationTools
          ├── MeasurementTools
          └── ReportViewer
```

### CaseViewer

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

### SlideViewer

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

## Slide Discovery Process

Slim discovers slides by:

1. **Querying series**: Uses QIDO-RS to find all series in a study
2. **Retrieving metadata**: Gets metadata for all instances in each series
3. **Grouping by slide**: Groups images by `ContainerIdentifier` and `FrameOfReferenceUID`
4. **Filtering by type**: Separates VOLUME, LABEL, OVERVIEW, and THUMBNAIL images
5. **Creating Slide objects**: Instantiates `Slide` objects for each physical slide

**Result**: A list of slides, each containing all related images organized by type.

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

Slim provides comprehensive annotation capabilities:

### ROI Annotations

- **Create**: Draw polygons, circles, points on the slide
- **Edit**: Modify existing annotations
- **Delete**: Remove annotations
- **Save**: Store annotations as DICOM Comprehensive 3D SR objects
- **Load**: Retrieve and display existing annotations

### Annotation Types

Based on [DICOM WSI annotation standards](https://dicom.nema.org/dicom/dicomwsi/):

- **ROI (Region of Interest)**: Polygons, circles, points marking areas of interest
- **Segmentation**: Binary masks categorizing image regions
- **Structured Reporting**: Measurements, observations, findings with context
- **Presentation States**: Display parameters (blending, pseudocoloring for fluorescence)

### Annotation Storage

Annotations are stored as separate DICOM objects:

- **Series**: Annotations stored in separate series (different modality)
- **Reference**: Annotations reference the image via `ReferencedSOPInstanceUID`
- **Coordinates**: Stored in physical (slide) coordinates, not pixel coordinates
- **Format**: DICOM Comprehensive 3D SR (TID1500) for measurements and findings

## Authentication

Slim supports OAuth 2.0 / OIDC authentication:

- **Multiple providers**: Configurable OAuth providers
- **Token management**: Automatic token refresh
- **DICOMweb integration**: Tokens included in DICOMweb requests

## Configuration

Slim is configured via:

- **Environment variables**: Server URLs, OAuth settings
- **Configuration files**: Client mappings, UI preferences
- **Runtime settings**: User preferences, display options

## Key Differences from dicom-microscopy-viewer

| Feature            | dicom-microscopy-viewer    | Slim                    |
| ------------------ | -------------------------- | ----------------------- |
| **Purpose**        | Rendering engine           | Complete application    |
| **UI**             | Minimal built-in controls  | Full UI with React      |
| **Study management** | None                    | Browse studies/series   |
| **Annotations**    | Display only              | Create, edit, save       |
| **Multi-server**   | Basic support             | Full multi-server setup |
| **Authentication** | None                      | OAuth 2.0 / OIDC        |

## Data Flow

1. **User selects study**: CaseViewer queries DICOMweb for series
2. **Slim discovers slides**: Groups images by ContainerIdentifier
3. **User selects slide**: SlideViewer component mounts
4. **Slim creates Slide object**: Organizes images by type
5. **Slim creates viewer**: Instantiates `VolumeImageViewer` with slide metadata
6. **Viewer renders**: dicom-microscopy-viewer loads and displays tiles

## Next Steps

- [Architecture Overview](./05-architecture-overview.md) - How everything fits together
- [Key Concepts](./06-key-concepts.md) - Important technical concepts

