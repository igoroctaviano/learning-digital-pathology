# Learning Pathology: A Guide to Whole Slide Imaging and DICOM

This directory contains educational materials about pathology viewers, whole slide imaging (WSI), and how the dicom-microscopy-viewer and Slim applications work together.

## Purpose

These materials are designed to help you understand:

- What pathology and whole slide imaging are
- How DICOM WSI format works (based on [DICOM Supplement 145](https://dicom.nema.org/dicom/dicomwsi/))
- How dicom-microscopy-viewer (the rendering engine) works
- How Slim (the viewer application) works
- How they integrate together
- Key technical concepts
- Common use cases and troubleshooting

## Who Is This For?

- **Developers** new to pathology imaging
- **Product managers** needing to understand the technology
- **Sales/Support** teams communicating with clients
- **DICOM experts** wanting to understand WSI implementation
- **Anyone** wanting to understand how pathology viewers work

**Note for Non-Programmers**: Code examples are included to illustrate concepts but can be skipped. Focus on the conceptual explanations - they provide the essential understanding without needing to read JavaScript/TypeScript code.

## Learning Path

We recommend reading these documents in order:

1. **[Introduction to Pathology and WSI](./01-introduction-to-pathology-and-wsi.md)**

   - What is pathology?
   - What is whole slide imaging?
   - Why pyramids and tiling?
   - Official DICOM WSI characteristics

2. **[DICOM WSI Format](./02-dicom-wsi-format.md)**

   - DICOM structure
   - Key attributes
   - Pyramid structure
   - Frame mapping
   - Optical paths
   - Coordinate systems

3. **[dicom-microscopy-viewer Engine](./03-dicom-microscopy-viewer-engine.md)**

   - Architecture overview
   - VolumeImageViewer
   - Tile loading
   - How to use the library

4. **[Slim Viewer Application](./04-slim-viewer-application.md)**

   - Component hierarchy
   - Slide data model
   - DICOMweb client management
   - Annotation features
   - Configuration

5. **[Architecture Overview](./05-architecture-overview.md)**

   - How everything fits together
   - Data flow
   - Component responsibilities
   - Design decisions

6. **[Key Concepts](./06-key-concepts.md)**

   - Image pyramids
   - Tiling
   - Coordinate systems
   - Optical paths
   - DICOMweb
   - Annotations
   - Compression

7. **[Common Use Cases](./07-common-use-cases.md)**

   - Viewing H&E slides
   - Multiplexed fluorescence
   - Creating annotations
   - Viewing segmentations
   - Bulk annotations
   - Parametric maps

8. **[Troubleshooting](./08-troubleshooting.md)**

   - Common issues
   - Debugging tips
   - Solutions

9. **[Glossary](./09-glossary.md)**
   - Terminology reference

## Quick Reference

### Key DICOM Attributes

- `StudyInstanceUID`: Unique study identifier
- `SeriesInstanceUID`: Unique series identifier
- `SOPInstanceUID`: Unique instance identifier
- `ContainerIdentifier`: Links images from same physical slide
- `FrameOfReferenceUID`: Links images in same coordinate system
- `TotalPixelMatrixColumns/Rows`: Full image dimensions
- `PixelSpacing`: Physical size per pixel (e.g., 0.00025 mm = 0.25 microns)
- `ImageType`: Array indicating image flavor (VOLUME, THUMBNAIL, OVERVIEW, LABEL)

### Image Types

- **VOLUME**: High-resolution pyramid images
- **THUMBNAIL**: Small preview (200-500 px)
- **OVERVIEW**: Navigation aid (1,000-5,000 px)
- **LABEL**: Physical slide label image

### Official Resources

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- [DICOM Supplement 145 - Whole Slide Imaging](https://dicom.nema.org/dicom/dicomwsi/)

## Getting Help

- Check the [Troubleshooting](./08-troubleshooting.md) guide
- Review the [Glossary](./09-glossary.md) for terminology
- Consult the [official DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/)

