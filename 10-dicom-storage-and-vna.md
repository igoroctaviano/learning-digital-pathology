# DICOM Storage & Vendor Neutral Archives (VNA)

This document explores storage solutions for digital pathology images, comparing radiology and pathology approaches, and examining how Vendor Neutral Archives (VNAs) can be adapted for whole slide imaging.

## Storage Solutions in Radiology vs. Pathology

### Radiology: Established DICOM Workflows

Radiology has utilized DICOM standards for image storage since the 1990s, establishing mature workflows:

- **PACS Systems**: Picture Archiving and Communication Systems store radiology images (CT, MRI, X-ray, etc.)
- **Vendor-Specific Solutions**: Many PACS systems are tailored to specific imaging equipment vendors
- **Vendor-Neutral Solutions**: VNAs provide vendor-neutral storage, decoupling storage from specific equipment
- **Standardized Format**: All radiology images use DICOM format, ensuring interoperability

### Pathology: Transition to Digital Imaging

Pathology is in a transition phase, moving from physical slides to digital whole slide imaging:

- **Recent Adoption**: Digital pathology adoption accelerated in the 2010s, with DICOM Supplement 145 (WSI standard) finalized in 2019
- **Proprietary Formats Still Common**: Many pathology labs still use proprietary formats from scanner manufacturers
- **Growing DICOM Adoption**: Increasing adoption of DICOM WSI format for standardization

### Size Differences

The scale difference between radiology and pathology images is dramatic:

**Radiology Images**:
- Typical size: Megapixels (e.g., 512×512 to 2048×2048 pixels)
- File size: Megabytes to low gigabytes per image
- Storage needs: Terabytes for large radiology departments

**Pathology WSI Images**:
- Typical size: Gigapixels (e.g., 80,000×60,000 pixels = 4.8 gigapixels)
- File size: Gigabytes per slide (15+ GB uncompressed)
- Storage needs: Terabytes to **petabytes** for pathology labs

### Format Differences

**Radiology**: Standardized DICOM format across all modalities

**Pathology**: Mix of proprietary and DICOM formats:
- Proprietary formats: `.svs` (Aperio), `.czi` (ZEISS), `.ndpi` (Hamamatsu), `.vms`/`.vmu` (Philips), `.mrxs` (3DHistech), `.scn` (Leica)
- Standardized format: DICOM WSI (DICOM Supplement 145)

### Storage Scale Requirements

- **Radiology**: Terabyte-scale storage sufficient for most departments
- **Pathology**: Petabyte-scale storage needed for large labs processing hundreds of slides per day

## How VNAs Work in Radiology and Potential Use in Pathology

### VNA Architecture: Brief Overview

Vendor Neutral Archives (VNAs) provide centralized storage that is independent of the originating imaging equipment:

- **Vendor-Neutral Storage**: Images stored in standard format (DICOM) regardless of source equipment
- **Standard Interface**: DICOM-based interfaces for storage and retrieval
- **Centralized Management**: Single repository for images from multiple departments/modalities
- **Long-Term Preservation**: Designed for long-term archiving and data migration

### Vendor-Neutral Storage

VNAs decouple storage from specific scanner vendors:

- **Interoperability**: Images from any vendor can be stored and retrieved
- **Future-Proofing**: Not locked into a single vendor's storage solution
- **Migration**: Easier to migrate between systems or vendors

### DICOM Standardization in VNAs

VNAs use DICOM standards for interoperability:

- **DICOM Storage**: All images stored in DICOM format
- **DICOMweb**: RESTful APIs (QIDO-RS, WADO-RS, STOW-RS) for access
- **Standard Metadata**: Consistent metadata structure across vendors
- **Query/Retrieve**: Standard DICOM query and retrieve mechanisms

### Benefits for Pathology

Implementing VNAs in pathology provides several advantages:

**Interoperability Across Scanner Vendors**:
- Store images from Aperio, Leica, Philips, Hamamatsu, ZEISS, and other scanners in one system
- View images from any scanner on any DICOM-compliant viewer
- Share images between institutions regardless of scanner vendor

**Long-Term Preservation**:
- Avoid vendor lock-in by using standard DICOM format
- Ensure images remain accessible even if scanner vendor discontinues support
- Facilitate data migration as technology evolves

**Centralized Management**:
- Single repository for all WSI images
- Unified access control and security policies
- Consistent backup and disaster recovery strategies

### Challenges Adapting VNAs for Pathology

While VNAs offer benefits, adapting them for pathology presents unique challenges:

**WSI File Sizes Require Different Storage Strategies**:
- Traditional VNAs designed for smaller radiology images
- Need for optimized storage architectures (tiered storage, compression strategies)
- Performance considerations for retrieving multi-gigabyte files

**DICOM Supplement 145 Support Requirements**:
- VNAs must properly support DICOM WSI format (Supplement 145)
- Handle multi-frame images, pyramids, and tiled organization
- Support WSI-specific metadata (optical paths, Z-planes, slide coordinates)

**Integration with LIS Workflows**:
- Laboratory Information Systems (LIS) integration requirements
- Workflow integration (order management, patient data linkage)
- Quality control and validation workflows

**Performance Considerations for Large Images**:
- Network bandwidth requirements for WSI transfer
- Retrieval performance for large files
- Caching strategies for frequently accessed slides

## Current Challenges in DICOM-Based Pathology Storage

### Large File Sizes

Whole slide images present unprecedented storage challenges:

**WSI Image Sizes**:
- Typical slide: 4.8+ gigapixels (80,000×60,000 pixels)
- Uncompressed size: 15+ GB per slide
- With compression (JPEG2000): 1-5 GB per slide

**Storage Requirements**:
- Small lab (100 slides/day): ~500 GB/day = ~180 TB/year
- Medium lab (500 slides/day): ~2.5 TB/day = ~900 TB/year
- Large lab (1000+ slides/day): 5+ TB/day = **petabytes per year**

**Network Transfer Challenges**:
- Transferring multi-gigabyte files over networks
- Bandwidth requirements for telepathology
- Cloud storage upload/download times

### Metadata Standardization Issues

Pathology images require rich, standardized metadata:

**Pathology-Specific Metadata Needs**:
- Staining protocols (H&E, IHC, special stains)
- Magnification levels and pixel spacing
- Scanner information and acquisition parameters
- Slide preparation details

**DICOM Supplement 145 Metadata Requirements**:
- Standardized metadata structure for WSI
- Optical path information (channels)
- Z-plane information (focal planes)
- Slide coordinates and transformations

**Consistency Across Scanner Vendors**:
- Different scanners capture different metadata
- Ensuring consistent metadata mapping during conversion
- Validating metadata completeness and accuracy

### Compression Requirements

Balancing diagnostic quality with storage efficiency:

**Lossless vs. Lossy Compression**:
- Lossless compression: Preserves all image data, larger file sizes
- Lossy compression: Smaller files, potential quality loss
- Diagnostic requirements: Some pathology use cases require lossless compression

**JPEG2000 Support for WSI**:
- JPEG2000 is commonly used for WSI compression
- Supports both lossless and lossy compression
- Efficient for large images with good quality retention

**Balancing Quality vs. Storage**:
- Diagnostic images may require higher quality (less compression)
- Research/archival images may tolerate more compression
- Need for configurable compression strategies

### Integration with LIS (Laboratory Information Systems)

Pathology workflows require tight integration with LIS:

**Workflow Integration Challenges**:
- Linking DICOM images to LIS orders and patient records
- Study/series organization matching LIS workflow
- Quality control checkpoints in workflow

**Patient Data Linkage**:
- Ensuring patient identifiers match between DICOM and LIS
- Maintaining data consistency across systems
- Handling patient data updates and corrections

**Study/Series Organization**:
- Organizing images into studies and series matching LIS structure
- Supporting multiple slides per case
- Managing related images (H&E, IHC, special stains)

**Quality Control Workflows**:
- Validating image quality before storage
- Tracking quality metrics and issues
- Supporting re-scan workflows

## Next Steps

- [DICOM Converters](./11-dicom-converters.md) - Converting proprietary formats to DICOM
- [Barriers to DICOM Adoption](./12-barriers-to-dicom-adoption.md) - Challenges in adopting DICOM for pathology
- [IHE DPIA Integration Profile](./13-ihe-dpia-integration-profile.md) - Standardized acquisition workflows

## References

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- DICOM Supplement 145 - Whole Slide Imaging in Pathology
- [Vendor Neutral Archive - Wikipedia](https://en.wikipedia.org/wiki/Vendor_Neutral_Archive)

