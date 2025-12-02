# Introduction to Pathology and Whole Slide Imaging

## What is Pathology?

Pathology is the medical specialty that studies the causes and effects of diseases by examining tissues, organs, and bodily fluids. Pathologists analyze samples under microscopes to diagnose diseases, determine treatment plans, and understand disease progression.

### Traditional Pathology Workflow

1. **Sample Collection**: A tissue sample (biopsy or surgical specimen) is collected
2. **Processing**: The sample is fixed, processed, embedded in paraffin, and sliced into thin sections
3. **Staining**: Sections are stained (commonly H&E - Hematoxylin and Eosin) to highlight different tissue structures
4. **Microscopy**: Pathologists examine slides under a microscope
5. **Documentation**: Findings are documented in reports

### The Digital Revolution: Whole Slide Imaging (WSI)

Whole Slide Imaging (WSI), also known as digital pathology, digitizes entire glass slides at high resolution. This enables:

- **Remote viewing**: Pathologists can view slides from anywhere
- **Collaboration**: Multiple pathologists can review the same slide simultaneously
- **AI/ML integration**: Computer algorithms can analyze images for pattern recognition
- **Archiving**: Digital slides don't degrade over time
- **Education**: Students and trainees can access vast libraries of cases

## Characteristics of Whole Slide Images

Based on the [official DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/), WSI images have unique characteristics:

### Image Dimensions and Data Size

WSI images are **extremely large**. According to DICOM Supplement 145:

**Typical Example**:
- Sample size: 20mm × 15mm
- Resolution: 0.25 micrometers per pixel (0.25 mpp, conventionally called "40X")
- Resulting image: **80,000 × 60,000 pixels** = **4.8 gigapixels**
- With 24-bit color: **~15 GB uncompressed**

**Resolution Conventions**:
- 0.25 mpp = "40X" magnification
- 0.5 mpp = "20X" magnification
- 1.0 mpp = "10X" magnification
- 2.5 mpp = "4X" magnification
- 5.0 mpp = "2X" magnification

**Extreme Example** (worst case):
- Sample size: 50mm × 25mm
- Resolution: 0.1 mpp (oil immersion, "100X")
- Multiple Z-planes: 10 focal planes
- Result: **500,000 × 250,000 pixels per plane** = **125 gigapixels per plane**
- Total dataset: **3.75 TB** (10 planes × 375 GB each)

### Why Pyramids?

**The Problem**: Loading a 15 GB image into memory is impossible. Even viewing a small region requires efficient access patterns.

**The Solution**: Store images as **multi-resolution pyramids**:

- **Level 0** (base): Highest resolution, full detail (e.g., 0.25 mpp)
- **Level 1**: 2× smaller (e.g., 0.5 mpp)
- **Level 2**: 4× smaller (e.g., 1.0 mpp)
- **Level 3**: 8× smaller (e.g., 2.5 mpp)
- **Level 4**: 16× smaller (e.g., 5.0 mpp)

This allows rapid zooming without processing large amounts of data.

### Why Tiling?

Even at a single pyramid level, loading the entire image is impractical. Instead, images are divided into **tiles** (typically 256×256 or 512×512 pixels):

```
┌─────┬─────┬─────┐
│  1  │  2  │  3  │
├─────┼─────┼─────┤
│  4  │  5  │  6  │
├─────┼─────┼─────┤
│  7  │  8  │  9  │
└─────┴─────┴─────┘
```

**Benefits**:
- Random access to any subregion without loading the entire image
- Only visible tiles are loaded as the user pans and zooms
- Similar to how Google Maps works

### Access Patterns

Pathologists use specific viewing patterns:

1. **Panning**: Navigate at low resolution (typically 5 mpp / "2X" or 2.5 mpp / "4X")
2. **Zooming**: Zoom into regions of diagnostic interest at high resolution (0.25 mpp / "40X")
3. **Focusing**: Adjust focal plane when multiple Z-planes are available

The DICOM format supports these patterns through:
- **Tiled organization**: Enables random access to subregions
- **Multi-resolution pyramids**: Enables rapid zooming
- **Separate Z-plane images**: Enables rapid focal plane changes

## DICOM Standard for WSI

### Why DICOM?

DICOM (Digital Imaging and Communications in Medicine) is the international standard for medical imaging. Using DICOM for WSI provides:

- **Interoperability**: Works with existing medical imaging infrastructure (PACS)
- **Standardization**: Consistent format across vendors
- **Metadata**: Rich metadata about patient, study, acquisition parameters
- **Integration**: Pathology and radiology images can be managed together in PACS

### DICOM Supplement 145

The DICOM Standard supports WSI through **Supplement 145 - Whole Slide Imaging in Pathology**, which defines:

- **VL Whole Slide Microscopy Image** IOD (Information Object Definition)
- **SOP Class UID**: `1.2.840.10008.5.1.4.1.1.77.1.6`
- **Multi-frame organization**: Each tile stored as a "frame"
- **Spatial coordinates**: Tiles positioned using 3D spatial coordinates (SCOORD3D)
- **Optical paths**: Supports multiple channels (e.g., H&E brightfield, fluorescence)

### Key DICOM Concepts for WSI

1. **Multi-frame images**: Tiles stored as frames in a single DICOM instance
2. **Pyramid structure**: Multiple DICOM instances at different resolutions
3. **Tiled organization**: Frames organized in a grid pattern
4. **Slide-based coordinates**: Physical coordinates (X, Y, Z) vs. image matrix (row, column)
5. **Optical paths**: Different imaging modalities stored as separate frames

## Image Types in WSI

A single slide can have multiple image types:

1. **VOLUME**: The main high-resolution image data with full pyramid levels

2. **OVERVIEW**: Low-resolution image (1,000-5,000 pixels wide) used for **navigation** - shows your position on the slide in a minimap/navigation panel

3. **THUMBNAIL**: Small preview image (200-500 pixels wide) used for **quick identification** when browsing lists of slides

4. **LABEL**: Image of the physical slide label (patient ID, dates, etc.)

### OVERVIEW vs THUMBNAIL

- **OVERVIEW**: Larger, used for navigation within a single slide (like a map view)
- **THUMBNAIL**: Smaller, used for browsing multiple slides (like a preview icon)
- THUMBNAIL can serve as a fallback if OVERVIEW is not available

## Additional WSI Features

### Z-Planes (Multiple Focal Planes)

Some specimens are thicker than the depth of field, requiring multiple focal planes:

- Each Z-plane is stored as a separate DICOM instance
- Z-value represents nominal physical height (in μm) above the slide surface
- Enables viewing different depths of the specimen
- Z-planes can track curved surfaces (see [DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/))

### Multi-Spectral Imaging

Some scanners capture multiple spectral bands:

- Up to 10 spectral bands possible
- 16-bit per pixel resolution
- Each band stored as a separate optical path/channel

### Optical Paths (Channels)

Different ways the slide was imaged:

- **Brightfield**: H&E staining (most common)
- **Fluorescence**: Multiple channels (DAPI, FITC, Rhodamine, etc.)
- **Multiplexed imaging**: Many channels (CycIF - Cyclic Immunofluorescence)

## Next Steps

Now that you understand the basics of pathology and WSI, let's dive into:

- [DICOM WSI Format Deep Dive](./02-dicom-wsi-format.md)
- [How dicom-microscopy-viewer Works](./03-dicom-microscopy-viewer-engine.md)
- [How Slim Viewer Works](./04-slim-viewer-application.md)

## References

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- DICOM Supplement 145 - Whole Slide Imaging in Pathology

