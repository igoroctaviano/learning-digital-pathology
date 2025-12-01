# Learning Pathology: A Guide to Whole Slide Imaging and DICOM

This directory contains educational materials about pathology viewers, whole slide imaging (WSI), and how the dicom-microscopy-viewer and Slim applications work together.

## Purpose

These materials are designed to help you understand:

- What pathology and whole slide imaging are
- How DICOM WSI format works
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

2. **[DICOM WSI Format](./02-dicom-wsi-format.md)**

   - DICOM structure
   - Key attributes
   - Pyramid structure
   - Frame mapping
   - Optical paths

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

### Key Components

- **dicom-microscopy-viewer**: Rendering engine (library)
- **Slim**: Viewer application (React app)
- **DICOMweb**: RESTful API for DICOM data

### Key Concepts

- **Pyramids**: Multi-resolution image representation
- **Tiling**: Dividing images into small tiles
- **Optical Paths**: Different imaging channels
- **Annotations**: ROI markings and measurements

### Code Locations

- **dicom-microscopy-viewer**: `/dicom-microscopy-viewer/src/`
- **Slim**: `/slim/src/`
- **Key files**:
  - `dicom-microscopy-viewer/src/viewer.js` - Main viewer class
  - `dicom-microscopy-viewer/src/pyramid.js` - Pyramid computation
  - `slim/src/components/SlideViewer.tsx` - Main viewer component
  - `slim/src/components/CaseViewer.tsx` - Study/series navigation

## How to Use These Materials

### For Learning

1. Start with the introduction
2. Read through each document in order
3. Reference code examples in the actual codebase
4. Try the concepts in practice

### For Teaching

1. Use as presentation materials
2. Reference specific sections during discussions
3. Point to code examples for technical details
4. Use troubleshooting guide for support

### For Client Communication

1. Use introduction for high-level overview
2. Reference architecture overview for system design
3. Use use cases for feature demonstrations
4. Reference glossary for terminology

## Additional Resources

- [DICOM Standard](https://www.dicomstandard.org/)
- [DICOM WSI Supplement](http://dicom.nema.org/medical/dicom/current/output/chtml/part03/sect_A.32.8.html)
- [dicom-microscopy-viewer Documentation](https://imagingdatacommons.github.io/dicom-microscopy-viewer/)
- [Slim Repository](https://github.com/ImagingDataCommons/slim)
- [dicom-microscopy-viewer Repository](https://github.com/ImagingDataCommons/dicom-microscopy-viewer)

## Contributing

If you find errors or want to add content:

1. Edit the relevant markdown file
2. Keep code examples accurate
3. Update this README if adding new documents
4. Reference actual code from the codebase

## Questions?

- Check the [Troubleshooting](./08-troubleshooting.md) guide
- Review the [Glossary](./09-glossary.md) for terminology
- Look at code examples in the actual codebase
- Refer to DICOM standard documentation
