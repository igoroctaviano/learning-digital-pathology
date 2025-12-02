# Glossary

Terminology reference for pathology imaging and DICOM WSI.

## A

**Annotation**: A mark or note added to an image. In DICOM WSI, annotations are stored separately from images as DICOM SR (Structured Reporting) objects.

## C

**Channel**: See Optical Path. In code, optical paths are often referred to as "channels" for brevity.

**ContainerIdentifier**: DICOM attribute that identifies the physical slide container. Images from the same physical slide share the same `ContainerIdentifier`.

**Concatenation**: The process of splitting a large DICOM instance across multiple files. The viewer automatically reassembles concatenated instances.

## D

**DICOM**: Digital Imaging and Communications in Medicine - the international standard for medical imaging.

**DICOMweb**: RESTful API for accessing DICOM data over HTTP. Includes QIDO-RS (Query), WADO-RS (Retrieve), and STOW-RS (Store).

**DimensionOrganizationType**: DICOM attribute indicating how frames are organized. Values include `TILED_FULL` (predictable order) and `TILED_SPARSE` (explicit positions).

## F

**Frame**: A single tile in a multi-frame DICOM object. Each frame represents one tile of the image.

**Frame Mapping**: A lookup table mapping tile positions (row-column-opticalPath) to frame numbers. Used to construct WADO-RS URLs.

**FrameOfReferenceUID**: DICOM attribute linking images that share the same coordinate system. All pyramid levels for a slide share the same `FrameOfReferenceUID`.

## I

**ICC Profile**: International Color Consortium profile for color correction. Ensures accurate color display across different monitors.

**Image Type**: DICOM attribute array indicating image characteristics. Format: `[ORIGINAL/DERIVED, PRIMARY/SECONDARY, VOLUME/THUMBNAIL/OVERVIEW/LABEL]`.

**IOD**: Information Object Definition - a DICOM data structure definition.

## L

**LABEL**: Image type for the physical slide label (patient ID, dates, etc.). Stored as a separate DICOM instance.

## O

**Optical Path**: A different way the slide was imaged (e.g., brightfield, fluorescence channel). Stored as separate frames within the same DICOM instance, identified by `OpticalPathIdentifier`.

**OpticalPathIdentifier**: Unique identifier for an optical path within a DICOM instance.

**OVERVIEW**: Image type for low-resolution navigation images (1,000-5,000 pixels wide). Used as a minimap within the viewer.

## P

**Pixel Spacing**: Physical size of each pixel (e.g., 0.00025 mm = 0.25 microns). Used to convert between pixel and physical coordinates.

**Pyramid**: Multi-resolution image structure with multiple levels at different zoom factors. Enables rapid zooming without scaling large images.

**PyramidUID**: Optional DICOM attribute identifying images that belong to the same pyramid.

## Q

**QIDO-RS**: Query based on ID for DICOM Objects - RESTful Service. DICOMweb service for searching studies, series, and instances.

## R

**ROI**: Region of Interest - an annotation marking an area of interest on an image (polygon, circle, point).

## S

**SCOORD3D**: 3D Spatial Coordinates - DICOM data structure for storing 3D coordinates.

**SOP Class UID**: Service-Object Pair Class Unique Identifier. Identifies the type of DICOM object (e.g., `1.2.840.10008.5.1.4.1.1.77.1.6` for VL Whole Slide Microscopy Image).

**SOPInstanceUID**: Service-Object Pair Instance Unique Identifier. Unique identifier for a specific DICOM instance.

**STOW-RS**: Store Over the Web - RESTful Service. DICOMweb service for storing new DICOM objects.

**SeriesInstanceUID**: Unique identifier for a DICOM series (collection of related instances).

**StudyInstanceUID**: Unique identifier for a DICOM study (collection of related series).

## T

**THUMBNAIL**: Image type for small preview images (200-500 pixels wide). Used for browsing slide lists.

**Tile**: A small rectangular region of an image (typically 256×256 or 512×512 pixels). Stored as a separate frame in a multi-frame DICOM object.

**TILED_FULL**: Dimension organization type where frames are organized in a predictable order (optical paths → rows → columns).

**TILED_SPARSE**: Dimension organization type where each frame's position is explicitly stored in `PlanePositionSlideSequence`.

**TotalPixelMatrixColumns**: Total width of the entire slide image in pixels.

**TotalPixelMatrixRows**: Total height of the entire slide image in pixels.

## V

**VL Whole Slide Microscopy Image**: DICOM Information Object Definition (IOD) for whole slide imaging. Defined in DICOM Supplement 145.

**VOLUME**: Image type for high-resolution pyramid images. Multiple VOLUME instances at different resolutions form the pyramid structure.

## W

**WADO-RS**: Web Access to DICOM Objects - RESTful Service. DICOMweb service for retrieving metadata and pixel data.

**WSI**: Whole Slide Imaging - the process of digitizing entire microscope slides at high resolution.

## Z

**Z-Plane**: A focal plane at a specific depth. Multiple Z-planes can be captured for thick specimens. Each Z-plane is stored as a separate DICOM instance.

## References

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- DICOM Supplement 145 - Whole Slide Imaging in Pathology

