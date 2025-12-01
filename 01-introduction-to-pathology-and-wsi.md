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

## What Makes WSI Different from Regular Images?

### Scale and Resolution

A single whole slide image can be:

- **Gigapixel resolution**: 100,000 x 100,000 pixels or more
- **Multiple gigabytes**: A single slide can be 1-10 GB uncompressed
- **Multi-resolution**: Images are stored at multiple zoom levels (pyramid structure)

### Why Pyramids?

Imagine trying to view a map:

- When zoomed out, you need a low-resolution overview
- When zoomed in, you need high-resolution detail
- Loading the entire gigapixel image at once would be impossible

**Solution**: Store the image as a pyramid with multiple resolution levels:

- **Level 0** (base): Highest resolution, full detail
- **Level 1**: 2x smaller (half resolution)
- **Level 2**: 4x smaller
- **Level 3**: 8x smaller (thumbnail/overview)

### Tiling

Even at a single pyramid level, loading the entire image is impractical. Instead, images are divided into **tiles** (typically 256x256 or 512x512 pixels):

```
┌─────┬─────┬─────┐
│  1  │  2  │  3  │
├─────┼─────┼─────┤
│  4  │  5  │  6  │
├─────┼─────┼─────┤
│  7  │  8  │  9  │
└─────┴─────┴─────┘
```

Only visible tiles are loaded as the user pans and zooms, similar to Google Maps.

## DICOM Standard for WSI

### Why DICOM?

DICOM (Digital Imaging and Communications in Medicine) is the international standard for medical imaging. Using DICOM for WSI provides:

- **Interoperability**: Works with existing medical imaging infrastructure
- **Standardization**: Consistent format across vendors
- **Metadata**: Rich metadata about patient, study, acquisition parameters
- **Integration**: Works with PACS (Picture Archiving and Communication Systems)

### DICOM VL Whole Slide Microscopy Image

The specific DICOM standard for WSI is called **VL Whole Slide Microscopy Image** (VL WSI). Key characteristics:

- **SOP Class UID**: `1.2.840.10008.5.1.4.1.1.77.1.6`
- **Multi-frame**: Each tile is stored as a "frame" in a multi-frame DICOM object
- **Spatial coordinates**: Tiles are positioned using 3D spatial coordinates (SCOORD3D)
- **Optical paths**: Supports multiple optical paths (e.g., H&E brightfield, fluorescence channels)

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

## Next Steps

Now that you understand the basics of pathology and WSI, let's dive into:

- [DICOM WSI Format Deep Dive](./02-dicom-wsi-format.md)
- [How dicom-microscopy-viewer Works](./03-dicom-microscopy-viewer-engine.md)
- [How Slim Viewer Works](./04-slim-viewer-application.md)
