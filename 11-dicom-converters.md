# DICOM Converters

This document explains why pathology images need conversion to DICOM format and provides an overview of converters specifically designed for digital pathology workflows.

## What are DICOM Converters?

DICOM converters are tools that transform medical images from various formats into the DICOM standard, ensuring compatibility across different medical imaging systems. These tools handle both format conversion (e.g., proprietary formats to DICOM) and metadata mapping (e.g., adding required DICOM attributes).

For general information about DICOM conversion, see:
- [DICOM Standard - Part 10: Media Storage and File Format](http://dicom.nema.org/dicom/2013/output/chtml/part10/chapter_7.html)
- [DICOM Standard Documentation](http://www.dicomstandard.org/current)

The focus of this document is on **pathology-specific conversion needs** and tools designed for whole slide imaging workflows.

## Why Pathology Images Need Conversion (JPEG2000, TIFF to DICOM)

Pathology images often originate in proprietary formats from scanner manufacturers. Converting these to DICOM is essential for achieving interoperability, standardization, and long-term preservation in digital pathology workflows.

### Proprietary Formats from Pathology Scanners

Different scanner manufacturers use proprietary file formats:

- **Aperio (Leica)**: `.svs` (Aperio ScanScope), `.svslide` (Aperio ScanScope XT), `.scn` (Leica)
- **Philips**: `.vms`, `.vmu` (Philips IntelliSite)
- **Hamamatsu**: `.ndpi`, `.vms` (NanoZoomer)
- **ZEISS**: `.czi` (ZEISS Axio Scan)
- **3DHistech**: `.mrxs` (Pannoramic)
- **Other formats**: Various TIFF-based formats, JPEG2000 files

These proprietary formats create interoperability challenges:
- Images from one scanner cannot be viewed on another vendor's viewer
- Sharing images between institutions requires vendor-specific software
- Long-term archiving risks format obsolescence

### Pathology-Specific Challenges

Converting pathology images presents unique challenges compared to other medical imaging:

**Large File Sizes**:
- Gigapixel images (4.8+ gigapixels per slide)
- Terabytes of data per pathology lab
- Conversion processes must handle very large files efficiently
- Memory and processing requirements are substantial

**Multi-Channel Images**:
- Brightfield images (H&E staining)
- Fluorescence images (multiple channels: DAPI, FITC, Rhodamine, etc.)
- Multiplexed imaging (20+ channels in CycIF)
- Each channel must be properly mapped to DICOM optical paths

**Z-Stacks (Multiple Focal Planes)**:
- Some specimens require multiple focal planes
- Each Z-plane stored as separate DICOM instance
- Z-coordinate information must be preserved
- Conversion must handle multi-plane datasets

**Multi-Resolution Pyramids**:
- WSI images stored as pyramids (multiple resolution levels)
- Each pyramid level must be converted to separate DICOM instance
- Pyramid structure must be preserved (same FrameOfReferenceUID, ContainerIdentifier)
- Resolution relationships between levels must be maintained

### Integration with PACS/VNA for Pathology Workflows

Converting to DICOM enables integration with existing medical imaging infrastructure:

**DICOM Supplement 145 Compliance Requirements**:
- Images must conform to DICOM Supplement 145 (WSI standard)
- Proper IOD (Information Object Definition) structure
- Required metadata attributes must be populated
- Multi-frame organization for tiles

**WSI-Specific Storage Considerations**:
- Tiled organization for efficient access
- Pyramid structure for multi-resolution viewing
- Optical path organization for multi-channel images
- External bulkdata references for large pixel data

### Standardization Benefits for Pathology Labs

Converting to DICOM provides significant benefits:

**Vendor Interoperability**:
- View images from any scanner on any DICOM-compliant viewer
- No longer locked into vendor-specific viewing software
- Choose best viewer for specific use case

**Long-Term Archiving**:
- DICOM is an international standard, reducing risk of format obsolescence
- Avoid vendor lock-in for long-term storage
- Easier data migration as technology evolves

**Consistent Metadata Structure**:
- Standardized metadata across all images
- Consistent patient, study, and series organization
- Easier querying and retrieval

### Workflow Integration with LIS (Laboratory Information Systems)

DICOM conversion enables integration with laboratory workflows:

**Patient Data Linkage**:
- DICOM metadata links images to patient records in LIS
- Patient identifiers (MRN, Accession Number) preserved
- Study/series organization matches LIS workflow

**Study/Series Organization**:
- Images organized into studies (cases) and series (slides)
- Supports multiple slides per case
- Related images (H&E, IHC, special stains) grouped appropriately

### Interoperability Between Different Scanner Vendors

DICOM conversion breaks down vendor barriers:

**Viewing Images from Any Scanner on Any Viewer**:
- Aperio images viewable on Leica viewers
- Philips images viewable on Hamamatsu viewers
- Any DICOM-compliant viewer can display any DICOM WSI image

**Sharing Images Between Institutions**:
- Standard format enables easy sharing
- No need for vendor-specific software at receiving institution
- Supports telepathology and consultation workflows

### Long-Term Archival Requirements

DICOM provides a preservation format for pathology images:

**DICOM as Preservation Format**:
- International standard with long-term support
- Well-documented format specification
- Multiple implementations available

**Avoiding Proprietary Format Obsolescence**:
- Proprietary formats may become unsupported
- Vendor may discontinue software or support
- DICOM ensures long-term accessibility

### AI/ML Pipeline Compatibility

DICOM conversion enables AI and machine learning workflows:

**Standardized Input Format for AI Models**:
- AI models can be trained on standardized DICOM format
- No need for format-specific preprocessing
- Consistent data structure across training datasets

**Research Reproducibility**:
- Standard format enables reproducible research
- Datasets can be shared in standard format
- Results can be compared across institutions

## Overview of Existing DICOM Converters for Digital Pathology

The following converters are specifically designed for or commonly used in digital pathology workflows:

### ZEISS DICOM Converter

**Purpose**: Converts ZEISS microscope images to DICOM WSI format

**Capabilities**:
- Converts ZEISS `.czi` files to DICOM WSI
- Supports brightfield images
- Supports fluorescence images (multiple channels)
- Supports multi-channel images
- Supports Z-stacks (multiple focal planes)
- Preserves metadata from ZEISS format

**Use Cases**:
- ZEISS scanner users converting to DICOM for PACS/VNA storage
- Integrating ZEISS images into DICOM-based workflows
- Sharing ZEISS images with other institutions

**Availability**: Official ZEISS tool

### Google Cloud Healthcare API Transformation Pipeline

**Purpose**: Open-source tool for converting pathology images to DICOM for cloud storage

**Capabilities**:
- Converts proprietary pathology formats to DICOM
- Designed for ingestion into Google Cloud Healthcare API DICOM store
- Supports various proprietary formats
- Handles metadata mapping and augmentation
- Cloud-optimized conversion pipeline

**Use Cases**:
- Pathology labs migrating to cloud storage
- Integrating pathology images into Google Cloud Healthcare API
- Large-scale batch conversion workflows

**Availability**: Open-source, part of Google Cloud Healthcare API

### Orthanc Dicomizer

**Purpose**: Command-line tool for converting various image formats to DICOM

**Capabilities**:
- Converts pathology images to DICOM format
- Supports various formats including WSI formats
- Command-line interface for batch processing
- Integrates with Orthanc DICOM server

**Use Cases**:
- Batch conversion of pathology images
- Integration with Orthanc DICOM server
- Automated conversion workflows

**Availability**: Open-source solution

### Meditecs DICOMPath

**Purpose**: Modular DICOM integration solution for digital pathology

**Capabilities**:
- Converts proprietary formats to DICOM
- IHE DPIA (Digital Pathology Image Acquisition) compliant
- Modular architecture for integration
- Enterprise-grade solution

**Use Cases**:
- Enterprise pathology labs requiring IHE DPIA compliance
- Complex integration requirements
- Production workflows requiring reliability

**Availability**: Enterprise solution from Meditecs

## Conversion Considerations

When converting pathology images to DICOM, consider:

**Metadata Mapping**:
- Map proprietary format metadata to DICOM attributes
- Ensure required DICOM attributes are populated
- Preserve pathology-specific information

**Image Quality**:
- Maintain diagnostic quality during conversion
- Preserve color accuracy (ICC profiles)
- Handle compression appropriately

**Performance**:
- Large file sizes require efficient conversion processes
- Batch conversion capabilities for high-volume labs
- Progress tracking and error handling

**Validation**:
- Validate DICOM compliance after conversion
- Verify metadata completeness
- Test image viewing in DICOM viewers

## Next Steps

- [Barriers to DICOM Adoption](./12-barriers-to-dicom-adoption.md) - Challenges in adopting DICOM for pathology
- [IHE DPIA Integration Profile](./13-ihe-dpia-integration-profile.md) - Standardized acquisition workflows
- [DICOM Storage & VNA](./10-dicom-storage-and-vna.md) - Storage solutions for pathology images

## References

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- DICOM Supplement 145 - Whole Slide Imaging in Pathology
- [ZEISS DICOM Converter](https://www.zeiss.com/microscopy/en/products/software/zeiss-dicom-converter.html)
- [Google Cloud Healthcare API - Digital Pathology](https://cloud.google.com/healthcare-api/docs/how-tos/dicom-digital-pathology)
- [Orthanc Project](https://www.orthanc-server.com/)
- [Meditecs DICOMPath](https://www.meditecs.com/dicompath-digital-pathology-integration/)

