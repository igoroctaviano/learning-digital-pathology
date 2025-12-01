# Glossary of Terms

## A

**Annotation**: A mark or note added to an image, typically indicating a region of interest (ROI) or measurement.

**Additive Blending**: A method of combining multiple image channels where pixel values are added together, commonly used for fluorescence imaging.

## B

**Brightfield**: A type of microscopy illumination where light passes through the specimen, commonly used for H&E stained slides.

**Bulk Annotations**: A DICOM format (Microscopy Bulk Simple Annotations) optimized for storing large numbers of annotations efficiently.

## C

**Channel**: See Optical Path. In code, optical paths are often referred to as "channels" for brevity.

**Container Identifier**: A DICOM attribute that identifies the physical slide container, used to group images from the same slide.

**Concatenation**: A method of splitting a large DICOM instance across multiple files, linked by a Concatenation UID.

**Coordinate System**: A system for specifying positions. In WSI, three systems are used: pixel coordinates, physical coordinates (millimeters), and screen coordinates.

## D

**DICOM**: Digital Imaging and Communications in Medicine - the international standard for medical imaging.

**DICOMweb**: A RESTful API for accessing DICOM data over HTTP, providing QIDO-RS (query), WADO-RS (retrieve), and STOW-RS (store) services.

**DICOM SR**: DICOM Structured Report - a format for storing structured data, used for annotations and measurements.

## F

**Frame**: A single image within a multi-frame DICOM object. In WSI, each tile is stored as a frame.

**Frame Mapping**: A mapping from tile position (row-column-channel) to frame number in a DICOM instance.

**FrameOfReferenceUID**: A DICOM attribute that links images sharing the same coordinate system.

**Fluorescence**: A type of microscopy where specimens emit light when excited by specific wavelengths.

## H

**H&E**: Hematoxylin and Eosin - the most common staining method in pathology, producing pink and purple colors.

## I

**ICC Profile**: International Color Consortium profile - defines color spaces to ensure consistent color display across devices.

**Image Pyramid**: A multi-resolution representation of an image, with multiple levels at different resolutions.

**Image Type**: A DICOM attribute indicating the type of image: VOLUME, THUMBNAIL, OVERVIEW, or LABEL.

## M

**Metadata**: Structured information about an image, stored in DICOM format.

**Multi-frame**: A DICOM object containing multiple frames (images), used for storing tiled WSI images.

**Multiplexed Imaging**: Imaging techniques that capture multiple channels simultaneously, such as CycIF (Cyclic Immunofluorescence).

## O

**Optical Path**: A way the slide was imaged, such as brightfield or a specific fluorescence channel. Identified by OpticalPathIdentifier.

**OpenLayers**: A JavaScript mapping library used by dicom-microscopy-viewer for tile-based rendering.

**Overview Image**: A low-resolution image providing an overview of the entire slide.

## P

**PACS**: Picture Archiving and Communication System - a system for storing and retrieving medical images.

**Parametric Map**: A DICOM format for storing quantitative data as images (e.g., attention maps, saliency maps).

**Pixel Spacing**: The physical size of each pixel, typically in millimeters (e.g., 0.00025 mm = 0.25 microns).

**Pyramid Level**: A single resolution level in an image pyramid, with Level 0 being the highest resolution.

## Q

**QIDO-RS**: Query based on ID for DICOM Objects - RESTful Service. A DICOMweb service for searching/querying DICOM objects.

## R

**ROI**: Region of Interest - an annotated area on an image, stored as vector graphics (polygon, ellipse, etc.).

## S

**SCOORD3D**: 3D Spatial Coordinates - a DICOM format for storing 3D coordinates, used for ROI annotations.

**Segmentation**: A DICOM format for storing binary masks, indicating which pixels belong to which segments (e.g., nuclei, tissue).

**Series**: A collection of related DICOM instances, identified by SeriesInstanceUID.

**Slim**: The viewer application built on top of dicom-microscopy-viewer.

**SOP Instance**: A Service-Object Pair instance - a single DICOM object, identified by SOPInstanceUID.

**Study**: A collection of related series, identified by StudyInstanceUID.

**STOW-RS**: Store Over the Web - RESTful Service. A DICOMweb service for storing DICOM objects.

## T

**Tile**: A small rectangular portion of an image, typically 256x256 or 512x512 pixels. Tiles are loaded on-demand as the user pans and zooms.

**Tiling**: The process of dividing an image into tiles for efficient storage and retrieval.

**Total Pixel Matrix**: The entire image dimensions (TotalPixelMatrixColumns x TotalPixelMatrixRows), before tiling.

**Transfer Syntax**: A DICOM attribute indicating how pixel data is encoded (e.g., JPEG, JPEG 2000, uncompressed).

## V

**VNA**: Vendor Neutral Archive - a system for storing medical images in a vendor-independent format.

**VOLUME**: The main high-resolution image data in a WSI dataset.

**VL Whole Slide Microscopy Image**: The DICOM standard (SOP Class) for whole slide imaging.

## W

**WADO-RS**: Web Access to DICOM Objects - RESTful Service. A DICOMweb service for retrieving DICOM objects and pixel data.

**Whole Slide Imaging (WSI)**: The process of digitizing entire glass slides at high resolution, also known as digital pathology.

**WSI**: See Whole Slide Imaging.
