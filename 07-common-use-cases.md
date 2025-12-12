# Common Use Cases

This document describes real-world scenarios and how the pathology viewer handles them.

## Use Case 1: Viewing H&E Stained Slides

### Scenario

A pathologist needs to review an H&E (Hematoxylin and Eosin) stained slide for diagnosis.

### Workflow

```
┌─────────────────────────────────────────────────────┐
│         H&E Slide Viewing Workflow                  │
└─────────────────────────────────────────────────────┘

1. User opens study
   ┌──────────────┐
   │ CaseViewer   │ ──Query──→ DICOMweb (QIDO-RS)
   └──────────────┘

2. Slim discovers slides
   ┌─────────────────────┐
   │ Groups by           │ ContainerIdentifier
   │ FrameOfReferenceUID │
   └─────────────────────┘

3. User selects slide
   ┌──────────────┐
   │ SlideViewer  │ Mounts
   └──────┬───────┘
          │
4. Viewer loads
   ┌──────▼───────┐
   │ Computes     │ Pyramid structure
   │ pyramid      │
   └──────┬───────┘
          │
5. User pans/zooms
   ┌──────▼───────┐
   │ Tiles load   │ On-demand via WADO-RS
   │ on-demand    │
   └──────────────┘
```

**Steps**:
1. **User opens study**: CaseViewer queries DICOMweb for series
2. **Slim discovers slides**: Groups images by `ContainerIdentifier`
3. **User selects slide**: SlideViewer mounts
4. **Viewer loads**: dicom-microscopy-viewer computes pyramid and displays slide
5. **User pans/zooms**: Tiles load on-demand as user navigates

### Technical Details

- **Image Type**: `["ORIGINAL", "PRIMARY", "VOLUME"]`
- **Optical Paths**: Single optical path (brightfield)
- **Compression**: Typically JPEG or JPEG 2000
- **Pyramid Levels**: 4-6 levels depending on resolution

### How It Works

**User workflow**:
1. User selects a slide from the list
2. Slim finds the slide's VOLUME images
3. Slim creates a `VolumeImageViewer` instance
4. dicom-microscopy-viewer computes the pyramid structure
5. dicom-microscopy-viewer sets up tile loading
6. User pans and zooms; tiles load on-demand

**Technical details**:
- Pyramid computation happens automatically from metadata
- Tiles are requested via DICOMweb WADO-RS as needed
- Lower resolution levels load first for quick initial display

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
// Slim: User selects slide
const slide = slides.find((s) =>
  s.seriesInstanceUIDs.includes(seriesInstanceUID)
);

// Slim: Creates viewer
const volumeViewer = new dmv.viewer.VolumeImageViewer({
  clientMapping: clients,
  metadata: slide.volumeImages,
  controls: ["overview", "position"],
});

// dicom-microscopy-viewer: Computes pyramid
const pyramid = _computeImagePyramid({ metadata: slide.volumeImages });

// dicom-microscopy-viewer: Loads tiles on demand
const tileLoader = _createTileLoadFunction({ pyramid, client, channel: "0" });
```
</details>

## Use Case 2: Viewing Multiplexed Fluorescence Images

### Scenario

A researcher needs to view a multiplexed fluorescence slide with 20+ channels (e.g., CycIF - Cyclic Immunofluorescence).

### Workflow

1. **User selects slide**: Same as H&E workflow
2. **Viewer detects channels**: Reads `OpticalPathSequence` from metadata
3. **User selects channels**: UI shows channel selector
4. **Viewer loads frames**: Requests frames for selected optical paths
5. **Channels blend**: Multiple channels displayed with pseudocoloring

### Technical Details

- **Image Type**: `["ORIGINAL", "PRIMARY", "VOLUME"]`
- **Optical Paths**: Multiple optical paths (20+ optical paths)
- **Storage**: Each optical path stored as separate frames in the same DICOM instance
- **Display**: Optical paths can be toggled on/off
- **Blending**: Additive blending for fluorescence

### How It Works

**Technical details**:
- All optical paths share the same `SOPInstanceUID` and `SeriesInstanceUID`
- Frame mapping includes optical path identifier: `"row-column-opticalPath"`
- Viewer loads frames for each selected optical path
- Channels can be blended additively with pseudocoloring

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
// dicom-microscopy-viewer: Detects multiple optical paths
const numberOfOpticalPaths = metadata.OpticalPathSequence.length;

// dicom-microscopy-viewer: Creates tile loader for each optical path
for (let i = 0; i < numberOfOpticalPaths; i++) {
  const opticalPathIdentifier =
    metadata.OpticalPathSequence[i].OpticalPathIdentifier;
  const tileLoader = _createTileLoadFunction({
    pyramid,
    client,
    channel: opticalPathIdentifier,
  });
}

// Slim: User selects optical paths to display
const visibleOpticalPaths = ["DAPI", "FITC", "Rhodamine"];
volumeViewer.setVisibleChannels(visibleOpticalPaths);
```
</details>

## Use Case 3: Creating ROI Annotations

### Scenario

A pathologist needs to mark regions of interest (ROI) on a slide for documentation or analysis.

### Workflow

1. **User selects drawing tool**: Polygon, circle, or point tool
2. **User draws on slide**: Clicks to create ROI
3. **Slim captures coordinates**: Stores in physical (slide) coordinates
4. **User saves annotation**: Slim creates DICOM Comprehensive 3D SR object
5. **Annotation stored**: Saved to DICOMweb server via STOW-RS

### Technical Details

- **Format**: DICOM Comprehensive 3D SR (TID1500)
- **Coordinates**: Physical (slide) coordinates, not pixels
- **Reference**: Links to image via `ReferencedSOPInstanceUID`
- **Series**: Stored in separate series (different modality)

### How It Works

**Technical details**:
- Annotations stored as separate DICOM objects (not part of image)
- Coordinates converted from screen → pixel → physical
- Stored in DICOM SR format with structured content
- Can reference multiple images (e.g., multiple Z-planes)

## Use Case 4: Viewing Segmentations

### Scenario

A researcher needs to view AI-generated segmentations (e.g., nuclei detection) overlaid on the slide.

### Workflow

1. **User selects slide**: Same as viewing workflow
2. **Slim queries for segmentations**: Searches for DICOM Segmentation objects
3. **Viewer loads segmentations**: Retrieves segmentation masks
4. **Masks overlay slide**: Displayed as colored overlays
5. **User toggles visibility**: Can show/hide different segments

### Technical Details

- **Format**: DICOM Segmentation IOD
- **Storage**: Separate series from images
- **Reference**: Links to image via `ReferencedSOPInstanceUID`
- **Segments**: Each segment (e.g., "Nuclei") is a separate layer/channel

### How It Works

**Technical details**:
- Segmentations stored as binary masks
- Each segment has a category (e.g., "Nuclei", "Cytoplasm")
- Masks aligned to image using spatial coordinates
- Displayed as colored overlays with transparency

## Use Case 5: Bulk Annotations

### Scenario

A machine learning algorithm generates thousands of annotations (e.g., cell detections) that need to be displayed.

### Workflow

1. **Algorithm generates annotations**: Creates DICOM Microscopy Bulk Simple Annotations object
2. **Annotations stored**: Saved to DICOMweb server
3. **User opens slide**: SlideViewer loads slide
4. **Slim queries annotations**: Searches for bulk annotation objects
5. **Viewer displays annotations**: Renders as overlay

### Technical Details

- **Format**: DICOM Microscopy Bulk Simple Annotations IOD
- **Storage**: Optimized for large numbers of annotations
- **Rendering**: Efficiently renders thousands of annotations
- **Categories**: Supports coded categories and classes

### How It Works

**Technical details**:
- Bulk annotations optimized for performance
- Supports 2D/3D points, polygons, polylines
- Coded categories for classification
- Efficient rendering for large datasets

## Use Case 6: Parametric Maps

### Scenario

A researcher needs to view parametric maps (heat maps) showing quantitative measurements across the slide.

### Workflow

1. **Analysis generates map**: Creates DICOM Parametric Map object
2. **Map stored**: Saved to DICOMweb server
3. **User opens slide**: SlideViewer loads slide
4. **Slim queries maps**: Searches for parametric map objects
5. **Viewer displays map**: Overlays heat map on slide

### Technical Details

- **Format**: DICOM Parametric Map IOD
- **Storage**: Separate series from images
- **Reference**: Links to image via `ReferencedSOPInstanceUID`
- **Values**: Real-world values mapped to colors

### How It Works

**Technical details**:
- Parametric maps store quantitative measurements
- Values mapped to colors (heat map)
- Can overlay on original image
- Supports multiple measurement types

## Use Case 7: Multi-Server Configuration

### Scenario

An organization stores images in one archive and annotations in another archive.

### Workflow

1. **Slim configured**: Multiple DICOMweb clients configured
2. **User opens study**: CaseViewer queries image server
3. **User selects slide**: SlideViewer loads images from image server
4. **User views annotations**: SlideViewer queries annotation server
5. **Annotations overlay**: Displayed on slide

### Technical Details

- **Client Mapping**: SOP Class UID → DICOMweb client
- **Images**: Retrieved from image server
- **Annotations**: Retrieved from annotation server
- **Fallback**: Can try multiple servers if one fails

### How It Works

**Technical details**:
- Slim maintains mapping of SOP Class UIDs to clients
- Automatically uses correct client for each object type
- Supports failover if primary server unavailable

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```typescript
// Slim: Configure multiple clients
const clients = {
  default: imageClient,
  [StorageClasses.COMPREHENSIVE_3D_SR]: annotationClient,
  [StorageClasses.SEGMENTATION]: segmentationClient
};

// dicom-microscopy-viewer: Uses client mapping
const volumeViewer = new dmv.viewer.VolumeImageViewer({
  clientMapping: clients,
  metadata: slide.volumeImages
});
```
</details>

## Next Steps

- [Troubleshooting](./08-troubleshooting.md) - Common issues and solutions
- [Glossary](./09-glossary.md) - Terminology reference

