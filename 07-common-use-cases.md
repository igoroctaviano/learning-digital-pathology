# Common Use Cases and Scenarios

## Use Case 1: Viewing an H&E Slide

### Scenario

A pathologist needs to review an H&E (Hematoxylin and Eosin) stained slide for diagnosis.

### What Happens

1. **User opens Slim**: Navigates to study URL or searches for patient
2. **Slim queries DICOMweb**: Uses QIDO-RS to find studies/series
3. **Slim organizes slides**: Groups images by ContainerIdentifier
4. **User selects slide**: Clicks on slide in sidebar
5. **Viewer loads**: dicom-microscopy-viewer computes pyramid
6. **Initial view**: Low-resolution overview loads first
7. **User zooms in**: High-resolution tiles load on demand
8. **User pans**: New tiles load as they become visible

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

A researcher needs to view a CycIF (Cyclic Immunofluorescence) slide with 20+ channels.

### What Happens

1. **Multiple optical paths**: Each channel is a separate optical path
2. **Channel selection**: User can select which channels to display
3. **Channel blending**: Multiple channels can be overlaid with colors
4. **Additive blending**: Channels are combined additively
5. **Color mapping**: Each channel gets a color (DAPI=blue, FITC=green, etc.)

### Technical Details

- **Image Type**: `["ORIGINAL", "PRIMARY", "VOLUME"]`
- **Optical Paths**: Multiple optical paths (20+ optical paths)
- **Storage**: Each optical path stored as separate frames in the same DICOM instance
- **Display**: Optical paths can be toggled on/off
- **Blending**: Additive blending for fluorescence

### Code Flow

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

## Use Case 3: Creating ROI Annotations

### Scenario

A pathologist wants to annotate a region of interest (tumor area) on a slide.

### What Happens

1. **User activates tool**: Clicks "Draw Polygon" button
2. **User draws**: Clicks points to create polygon
3. **Slim captures geometry**: Stores polygon coordinates
4. **User adds label**: Selects finding (e.g., "Neoplasm")
5. **Slim creates DICOM SR**: Structures annotation according to TID 1500
6. **Slim stores**: Sends DICOM SR to server via STOW-RS
7. **Viewer displays**: Annotation appears as overlay

### Technical Details

- **Storage Format**: DICOM Comprehensive 3D SR
- **Template**: TID 1500 (Measurement Report)
- **ROI Template**: TID 1410 (Planar ROI Measurements)
- **Coordinates**: Stored in physical coordinates (mm)
- **Geometry**: SCOORD3D (3D spatial coordinates)

### Code Flow

```typescript
// Slim: User draws polygon
const polygon = new dmv.scoord3d.Polygon({
  coordinates: [[x1, y1], [x2, y2], [x3, y3], ...]
})

// Slim: Creates ROI
const roi = new dmv.roi.ROI({
  scoord3d: polygon,
  finding: {
    value: "108369006",  // SNOMED CT: Neoplasm
    schemeDesignator: "SCT",
    meaning: "Neoplasm"
  }
})

// Slim: Creates DICOM SR
const report = buildReport({
  rois: [roi],
  referencedImage: slide.volumeImages[0]
})

// Slim: Stores via DICOMweb
await client.storeInstances({
  studyInstanceUID,
  seriesInstanceUID,
  instances: [report]
})
```

## Use Case 4: Viewing Segmentation Results

### Scenario

An AI algorithm has segmented nuclei in a slide. The user wants to view the results.

### What Happens

1. **Segmentation exists**: DICOM Segmentation instance references the slide
2. **Slim detects**: Finds segmentation series linked to slide
3. **Viewer loads**: dicom-microscopy-viewer loads segmentation
4. **Pyramid fitting**: Segmentation pyramid is fitted to image pyramid
5. **Overlay display**: Segmentation displayed as colored overlay
6. **User toggles**: Can show/hide segmentation

### Technical Details

- **Storage Format**: DICOM Segmentation
- **Segments**: Each segment (e.g., "Nuclei") is a separate layer/channel
- **Pyramid Fitting**: Segmentation must match image pyramid levels
- **Display**: Colored overlay on top of image
- **Interpolation**: Can interpolate between pyramid levels

### Code Flow

```typescript
// Slim: Finds segmentation
const segmentation = await findSegmentation({
  studyInstanceUID,
  referencedSeriesInstanceUID: slide.seriesInstanceUIDs[0],
});

// dicom-microscopy-viewer: Loads segmentation
const segmentationPyramid = _computeImagePyramid({
  metadata: segmentation.metadata,
});

// dicom-microscopy-viewer: Fits to image pyramid
const [fittedPyramid, minZoom, maxZoom] = _fitImagePyramid(
  segmentationPyramid,
  imagePyramid
);

// dicom-microscopy-viewer: Creates overlay layer
const segmentLayer = new TileLayer({
  source: new TileSource({
    tileLoadFunction: _createTileLoadFunction({
      pyramid: fittedPyramid,
      client,
      channel: segmentNumber,
    }),
  }),
});

// Viewer: Adds overlay
volumeViewer.addOverlay(segmentLayer);
```

## Use Case 5: Viewing Bulk Annotations

### Scenario

An algorithm has annotated thousands of cells. The user wants to view them.

### What Happens

1. **Bulk annotations exist**: DICOM Microscopy Bulk Simple Annotations instance
2. **Slim detects**: Finds annotation series
3. **Viewer loads**: dicom-microscopy-viewer loads annotation groups
4. **Clustering**: Annotations are clustered for performance
5. **Display**: Annotations shown as points/circles
6. **Filtering**: User can filter by annotation properties

### Technical Details

- **Storage Format**: DICOM Microscopy Bulk Simple Annotations
- **Annotation Groups**: Annotations grouped by properties
- **Clustering**: Many annotations clustered for performance
- **Display**: Points or circles at annotation locations
- **Properties**: Each annotation has properties (e.g., cell type)

### Code Flow

```typescript
// Slim: Finds bulk annotations
const bulkAnnotations = await findBulkAnnotations({
  studyInstanceUID,
  referencedSeriesInstanceUID: slide.seriesInstanceUIDs[0],
});

// dicom-microscopy-viewer: Loads annotation groups
const annotationGroups = bulkAnnotations.AnnotationGroupSequence.map(
  (group) => new AnnotationGroup(group)
);

// dicom-microscopy-viewer: Clusters annotations
const clusters = clusterAnnotations(annotationGroups, {
  maxZoom: 10,
  clusterDistance: 50,
});

// dicom-microscopy-viewer: Creates feature layer
const features = new Collection();
annotationGroups.forEach((group) => {
  group.annotations.forEach((annotation) => {
    const feature = new Feature({
      geometry: new Point(annotation.coordinates),
    });
    features.push(feature);
  });
});

// Viewer: Adds feature layer
volumeViewer.addFeatures(features);
```

## Use Case 6: Viewing Parametric Maps

### Scenario

An AI model has generated an attention map showing where it "looks" when making predictions.

### What Happens

1. **Parametric map exists**: DICOM Parametric Map instance
2. **Slim detects**: Finds parametric map series
3. **Viewer loads**: dicom-microscopy-viewer loads parametric map
4. **Color mapping**: Float values mapped to colors (heat map)
5. **Overlay display**: Heat map overlaid on image
6. **User adjusts**: Can adjust opacity, colormap

### Technical Details

- **Storage Format**: DICOM Parametric Map
- **Data Type**: Float arrays (attention scores)
- **Color Mapping**: Colormap applied (e.g., jet, hot, cool)
- **Interpolation**: Can interpolate between pyramid levels
- **Overlay**: Semi-transparent overlay

### Code Flow

```typescript
// Slim: Finds parametric map
const parametricMap = await findParametricMap({
  studyInstanceUID,
  referencedSeriesInstanceUID: slide.seriesInstanceUIDs[0],
});

// dicom-microscopy-viewer: Loads parametric map
const parametricMapPyramid = _computeImagePyramid({
  metadata: parametricMap.metadata,
});

// dicom-microscopy-viewer: Fits to image pyramid
const [fittedPyramid] = _fitImagePyramid(parametricMapPyramid, imagePyramid);

// dicom-microscopy-viewer: Creates colormap
const colormap = createColormap({
  name: "jet",
  min: 0,
  max: 1,
});

// dicom-microscopy-viewer: Creates overlay layer
const mapLayer = new TileLayer({
  source: new TileSource({
    tileLoadFunction: async (z, y, x) => {
      const tile = await loadTile(z, y, x);
      const coloredTile = applyColormap(tile, colormap);
      return coloredTile;
    },
  }),
  opacity: 0.5,
});

// Viewer: Adds overlay
volumeViewer.addOverlay(mapLayer);
```

## Use Case 7: Multi-Server Configuration

### Scenario

Images are stored in one archive, annotations in another.

### What Happens

1. **Slim configured**: Multiple DICOMweb servers configured
2. **Client mapping**: Different SOP classes map to different servers
3. **Image retrieval**: Images loaded from primary server
4. **Annotation retrieval**: Annotations loaded from secondary server
5. **Seamless display**: User doesn't notice difference

### Technical Details

- **Client Mapping**: `clientMapping` option in VolumeImageViewer
- **SOP Class UIDs**: Different UIDs map to different clients
- **Fallback**: Can try multiple clients if one fails
- **Transparency**: User doesn't need to know about multiple servers

### Code Flow

```typescript
// Slim: Configures multiple clients
const clients = {
  default: new DICOMwebClient({ url: "https://images.example.com/dicomweb" }),
  [StorageClasses.COMPREHENSIVE_3D_SR]: new DICOMwebClient({
    url: "https://annotations.example.com/dicomweb",
  }),
};

// Slim: Creates viewer with client mapping
const volumeViewer = new dmv.viewer.VolumeImageViewer({
  clientMapping: clients,
  metadata: slide.volumeImages,
});

// dicom-microscopy-viewer: Uses appropriate client
// Images → default client
// Annotations → SR client
```

## Summary

These use cases demonstrate:

- **Flexibility**: Viewer handles many scenarios
- **Integration**: Works with various DICOM types
- **Performance**: Optimized for large datasets
- **Usability**: Simple interface for complex operations

Understanding these use cases helps explain why certain features exist and how they're used in practice.

## Next Steps

- [Troubleshooting](./08-troubleshooting.md) - Common issues and solutions
- [Glossary](./09-glossary.md) - Terminology reference
