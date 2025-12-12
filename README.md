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
- DICOM storage and Vendor Neutral Archives (VNA) for pathology
- DICOM converters for pathology images
- Barriers to DICOM adoption in digital pathology
- IHE DPIA integration profile for standardized workflows
- Future trends including AI and telepathology

## Who Is This For?

- **Developers** new to pathology imaging
- **Product managers** needing to understand the technology
- **Sales/Support** teams communicating with clients
- **DICOM experts** wanting to understand WSI implementation
- **Anyone** wanting to understand how pathology viewers work

**Note for Non-Programmers**: Code examples are included to illustrate concepts but can be skipped. Focus on the conceptual explanations - they provide the essential understanding without needing to read JavaScript/TypeScript code.

## Learning Path

We recommend reading these documents in order:

```
┌─────────────────────────────────────────────────────┐
│              Recommended Learning Path              │
└─────────────────────────────────────────────────────┘

Foundation (Start Here)
├─ 01. Introduction to Pathology and WSI
├─ 02. DICOM WSI Format
└─ 03. dicom-microscopy-viewer Engine

Application Layer
├─ 04. Slim Viewer Application
└─ 05. Architecture Overview

Core Concepts
├─ 06. Key Concepts
└─ 07. Common Use Cases

Reference & Troubleshooting
├─ 08. Troubleshooting
└─ 09. Glossary

Advanced Topics
├─ 10. DICOM Storage & VNA
├─ 11. DICOM Converters
├─ 12. Barriers to DICOM Adoption
├─ 13. IHE DPIA Integration Profile
└─ 14. Future Trends & Next Steps (AI)
```

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

10. **[DICOM Storage & Vendor Neutral Archives (VNA)](./10-dicom-storage-and-vna.md)**

    - Storage solutions in radiology vs. pathology
    - How VNAs work and their potential use in pathology
    - Current challenges in DICOM-based pathology storage

11. **[DICOM Converters](./11-dicom-converters.md)**

    - Why pathology images need conversion to DICOM
    - Overview of DICOM converters for digital pathology
    - Pathology-specific conversion challenges

12. **[Barriers to DICOM Adoption](./12-barriers-to-dicom-adoption.md)**

    - Lack of standardization in pathology compared to radiology
    - Challenges with image size, metadata handling, and compression
    - Legal, regulatory, and IT barriers

13. **[IHE DPIA Integration Profile](./13-ihe-dpia-integration-profile.md)**

    - Overview of DPIA and its relevance to digital pathology
    - How DPIA facilitates standardized image acquisition workflows

14. **[Future Trends & Next Steps (AI)](./14-future-trends-and-next-steps.md)**

    - AI & machine learning applications using DICOM-based pathology images
    - Emerging technologies and DICOM standards development
    - Telepathology as facilitated by DICOM

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

