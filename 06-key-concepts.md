# Key Concepts

This document explains important technical concepts used throughout the pathology viewer system.

## 1. Image Pyramids

### What is a Pyramid?

A multi-resolution pyramid stores the same image at multiple zoom levels:

- **Level 0** (base): Highest resolution (e.g., 0.25 mpp / "40X")
- **Level 1**: 2× smaller (e.g., 0.5 mpp / "20X")
- **Level 2**: 4× smaller (e.g., 1.0 mpp / "10X")
- **Level 3**: 8× smaller (e.g., 2.5 mpp / "4X")
- **Level 4**: 16× smaller (e.g., 5.0 mpp / "2X")

### Why Pyramids?

According to the [official DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/), pyramids enable:

- **Rapid zooming**: Switch between resolution levels without scaling large images
- **Efficient storage**: Pre-computed levels avoid runtime computation
- **Fast initial display**: Lower resolution loads quickly for overview

### How It's Computed

The viewer processes multiple DICOM instances to build the pyramid:

1. **Sorts by size**: Arranges instances from largest (highest resolution) to smallest
2. **Validates**: Ensures all instances share the same `FrameOfReferenceUID` and `ContainerIdentifier`
3. **Calculates properties**: For each level, computes:
   - Resolution factor (how much smaller than base level)
   - Tile grid dimensions (how many tiles wide/tall)
   - Pixel spacing (physical size per pixel)

**Result**: A structured pyramid ready for multi-resolution rendering.

### Practical Pyramid Structure

In practice, whole slide images are stored at multiple resolutions to accommodate efficient loading. For example, a sample whole slide image acquired at 40X by the Aperio Scanscope whole slide scanner is accompanied by the same image downsampled at 10X, 2.5X, and 1.25X, as well as a thumbnail image that represents the entire tissue fit within a ~1-megapixel frame.

**Typical Pyramid Levels**:
- **Level 0**: Highest resolution (e.g., 40X / 0.25 mpp)
- **Level 1**: 10X / 1.0 mpp (4× downsampled)
- **Level 2**: 2.5X / 4.0 mpp (16× downsampled)
- **Level 3**: 1.25X / 8.0 mpp (32× downsampled)
- **Thumbnail**: Entire slide in ~1 megapixel frame

**Optimal Number of Levels**:
- Typically 3 or 4 levels provide good balance
- More levels result in incrementally larger file sizes but allow viewing software to more efficiently select the scale at which to load data
- Users are encouraged to select a value that balances the tradeoff between bandwidth and overall file size

**File Size Impact**:
The addition of downsampled images increases overall file size. The relationship between pyramid levels and file size can be estimated, with each additional level adding approximately 25-33% to the base level size (depending on compression).

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Pyramid computation sorts images by size
metadata.sort((a, b) => {
  const sizeDiff = a.TotalPixelMatrixColumns - b.TotalPixelMatrixColumns;
  return sizeDiff;
});

// Creates pyramid structure
const pyramid = {
  resolutions: [1, 2, 4, 8], // Zoom factors
  metadata: [level0, level1, level2, level3], // DICOM instances
  gridSizes: [
    [1000, 1000],
    [500, 500],
    [250, 250],
    [125, 125],
  ], // Tile grids
};
```
</details>

## 2. Tiling

### What is Tiling?

Even at a single pyramid level, loading the entire image is too large. Instead, images are divided into **tiles** (typically 256×256 or 512×512 pixels).

### Tile Grid

```
┌─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  4  │  Row 1
├─────┼─────┼─────┼─────┤
│  5  │  6  │  7  │  8  │  Row 2
├─────┼─────┼─────┤
│  9  │ 10  │ 11  │ 12  │  Row 3
└─────┴─────┴─────┴─────┘
```

Each tile is stored as a separate "frame" in the DICOM multi-frame object.

### Frame Mapping

The viewer creates a mapping: `"row-column-opticalPath" → frame number`

Example:

- `"1-2-0"` → Row 1, Column 2, Optical Path "0" → Frame 5
- Used to construct WADO-RS URLs

## 3. Coordinate Systems

### Pixel Coordinates

- **Origin**: Upper-left corner of image matrix
- **Row**: Vertical position (increasing downward)
- **Column**: Horizontal position (increasing rightward)
- **Units**: Pixels

### Physical Coordinates (Slide-Based)

According to the [DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/):

- **Origin**: Top-left corner of slide (label on left)
- **X-axis**: Horizontal, increasing left to right
- **Y-axis**: Vertical, increasing top to bottom
- **Z-axis**: Perpendicular to slide surface (height above slide)
- **Units**: Millimeters (mm) or micrometers (μm)

**Important**: Slide-based coordinates are rotated 180° from image matrix coordinates.

### Transformations

To convert between coordinate systems, multiply or divide by `PixelSpacing`:

- **Pixel → Physical**: Multiply pixel coordinates by pixel spacing
  - Example: Pixel (1000, 2000) × spacing (0.00025 mm) = Physical (0.25 mm, 0.5 mm)
- **Physical → Pixel**: Divide physical coordinates by pixel spacing
  - Example: Physical (0.25 mm, 0.5 mm) ÷ spacing (0.00025 mm) = Pixel (1000, 2000)

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Pixel → Physical
const physicalX = pixelX * pixelSpacing[0];
const physicalY = pixelY * pixelSpacing[1];

// Physical → Pixel
const pixelX = physicalX / pixelSpacing[0];
const pixelY = physicalY / pixelSpacing[1];
```
</details>

## 4. Optical Paths (Channels)

An optical path represents a different way the slide was imaged:

- **Brightfield**: Standard H&E staining
- **Fluorescence**: DAPI, FITC, Rhodamine optical paths
- **Multiplexed**: Many optical paths (CycIF)

### How They're Stored

**Important**: Optical paths are stored as **separate frames within the same DICOM instance** (same `SOPInstanceUID` and `SeriesInstanceUID`), not as separate images or series.

Each optical path is:

- Identified by `OpticalPathIdentifier` in the `OpticalPathSequence` metadata
- Stored as separate frames in the same multi-frame DICOM object
- Can be displayed independently or blended

### Display Modes

For fluorescence images, multiple optical paths can be:

- **Displayed separately**: Switch between optical paths
- **Blended additively**: Overlay optical paths with colors
- **Used for analysis**: Combine optical paths for measurements

## 5. DICOMweb

DICOMweb is a RESTful API for accessing DICOM data over HTTP.

### Services

- **QIDO-RS** (Query): Search for studies/series/instances
- **WADO-RS** (Retrieve): Retrieve metadata and pixel data
- **STOW-RS** (Store): Store new DICOM objects

### Example URLs

DICOMweb uses RESTful URLs:

- **Query studies**: `GET /studies?PatientID=12345` - Search for studies matching criteria
- **Retrieve series metadata**: `GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/metadata` - Get all DICOM attributes for instances in a series
- **Retrieve a frame**: `GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/1` - Get pixel data for a specific frame

These are standard HTTP GET requests - no special DICOM protocol needed.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Query for studies
GET /studies?PatientID=12345

// Retrieve series metadata
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/metadata

// Retrieve a frame
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/1
```
</details>

## 6. Annotations

According to the [DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/), annotations are stored separately from images:

### Types

- **ROI (Region of Interest)**: Polygons, circles, points
- **Segmentation**: Binary masks categorizing regions
- **Structured Reporting**: Measurements, observations, findings
- **Presentation States**: Display parameters (blending, pseudocoloring)

### Storage

- **Separate Series**: Different modality than images
- **Reference Images**: Link to images via `ReferencedSOPInstanceUID`
- **Physical Coordinates**: Stored in slide-based coordinates, not pixels

## 7. Compression

WSI images are compressed to reduce storage and transfer. Compression is ubiquitous in WSI, with many vendors using JPEG, JPEG 2000, or LZW compression to reduce file size to manageable levels, often resulting in a reduction of file size by a factor of 7 or more.

### Compression Formats

- **JPEG**: Lossy compression, good for color images
- **JPEG 2000**: Better compression, supports both lossless and lossy modes
- **JPEG-LS**: Lossless compression
- **LZW**: Lossless compression used by some vendors

The viewer supports all formats and decodes them client-side.

### JPEG Compression Quality

For systems that enable users to specify the level of JPEG compression (usually expressed as a quality factor between 0 and 1):

**Quality Factor 0.8**:
- Reduction in file size by a factor of ~15.9
- Generally acceptable quality for most diagnostic purposes
- Minimal visible artifacts

**Quality Factor 0.1**:
- Reduction in file size by a factor of ~108.3
- Visible artifacts become apparent
- Not recommended for diagnostic use

**Important Considerations**:
- Morphologic assessments appear to be less affected by compression
- Densitometric assessments are increasingly sensitive to loss of digital information
- Users are discouraged from applying JPEG compression successively on the same image, as each round of compression can result in degradation of image quality
- Excessive compression can introduce visible image artifacts

### Blank Region Removal

An additional method commonly used in WSI to reduce file size is to discard blank regions of the slide altogether. Vendors use this technique to:
- Reduce file sizes
- Reduce scan times by identifying regions in the initial macro snapshot that do not need to be scanned

## 8. Concatenation

Large images may be split across multiple DICOM files:

- **Linked**: Parts linked by `SOPInstanceUIDOfConcatenationSource`
- **Identified**: By `ConcatenationUID` and `InConcatenationNumber`
- **Offset**: Frame offset tracked by `ConcatenationFrameOffsetNumber`

### Reassembly

The viewer automatically detects and reassembles concatenations:

1. **Detects concatenation**: Checks for `SOPInstanceUIDOfConcatenationSource` attribute
2. **Groups parts**: Links parts using `ConcatenationUID` and `InConcatenationNumber`
3. **Reassembles metadata**: Combines metadata from all parts
4. **Combines frame mappings**: Merges frame mappings accounting for `ConcatenationFrameOffsetNumber`
5. **Creates unified instance**: Treats reassembled parts as a single instance

**Result**: The viewer sees one complete image, even though it was stored in multiple files.

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Detects concatenation parts
if (metadata.SOPInstanceUIDOfConcatenationSource) {
  // Reassemble metadata
  // Combine frame mappings
  // Create unified instance
}
```
</details>

## 9. ICC Profiles and Color Calibration

ICC (International Color Consortium) profiles ensure accurate color display and are critical for maintaining diagnostic quality in digital pathology.

### Why Color Matters

Differences in color can have a significant influence on diagnostic performance:

**Display Aging Effects**:
- Aging reduces color saturation and luminosity
- Produces a shift in the color point of white
- Research shows that aged displays increase pathologist scoring time (from 41 to 50 seconds)
- Ease of reading drops from 9.7 to 8.7 (on a scale of 1-10)
- Intersession percentage agreement of diagnostic scores is about 20% higher on non-aged displays

**Color Differences in the Pipeline**:
Every laboratory has its own protocols for processing samples, including tissue acquisition, fixation, processing, and staining. This is a first source of perceived color differences. WSI further highlights color differences as more components are added to the visualization chain:
- Different scanners may produce different color renditions of the same slide
- Different viewing applications may exhibit visible color differences
- Different displays can cause differences in color perception

### ICC Framework for Color Calibration

Absolute color calibration can be achieved using the International Color Consortium (ICC) framework, an open, vendor-neutral, cross-platform color management protocol.

**Calibration Process**:
1. **Characterize the Scanner**: Scan a color calibration slide containing semi-transparent colored patches with known color attributes
2. **Create ICC Profile**: Generate an ICC profile that maps scanner color space to a standard color space
3. **Characterize the Display**: Measure display color characteristics
4. **Apply Color Management**: Use ICC profiles to ensure consistent color from scanner to display

**Benefits**:
- **Color Correction**: Adjusts colors to match original scanner output
- **Standardization**: Ensures consistent colors across displays
- **Diagnostic Quality**: Maintains color accuracy critical for diagnosis
- **Reproducibility**: Enables consistent color perception across different systems

**Color Gamut Matching**:
When the color gamut of the whole slide scanner does not match the color gamut of the display, different color sensations may occur. For example, the most saturated red that can be acquired by the scanner may not correspond to the most saturated red of the display. ICC profiles help manage these differences.

**Pre-Imaging Standardization**:
Agreement on standardized tissue handling protocols and stain standardization may be useful at the pre-imaging stage to minimize color differences before digitization.

## 10. Viewing Software and Tools

Whole slide imaging offers an opportunity to expand the set of tools available to users, including digital annotation, rapid navigation/magnification, and computer-assisted viewing and analysis.

### Vendor-Provided Viewers

Many WSI systems include image viewing software:
- **Local Installation**: Software installed on user computers
- **Network-Based**: Software suite residing on network servers, enabling browser-based viewing
- **Integrated Analysis**: Some viewers include algorithms for cell detection, positive staining computation, regional segmentation, and nuclear segmentation

### Open-Source Viewing Tools

**ImageJ and Fiji**:
- **ImageJ**: Free software tool from the National Institutes of Health
- Includes common image analysis algorithms useful for histology image processing
- **Fiji**: Popular package that bundles features and enhancements to ImageJ
- Extends the suite of image analysis algorithms available to researchers
- **Limitation**: Loading entire whole slide images in these platforms remains difficult; special considerations must be made to efficiently load and process these images

**QuPath**:
- Freely available open-source whole slide image viewer
- Developed by Centre for Cancer Research & Cell Biology at Queen's University Belfast
- Extends ImageJ-like functionality to a platform designed specifically for whole slide images
- Offers whole slide viewing, annotations, image analysis, and automation
- Allows researchers access to advanced open-source software tools

### Libraries for Direct Image Access

For research applications where automated quantitative processing is used, direct access to image files is sometimes preferred:

**OpenSlide**:
- Vendor-neutral software foundation for digital pathology
- Provides support for more than 8 proprietary whole slide formats
- Enables users to use the same basic functionality for images across multiple vendors
- Facilitates multi-institution studies or analysis of public image repositories

**Bio-Formats**:
- Provides support for a greater number of image formats
- Supports integration with commonly used programming and scripting environments like Matlab and Python
- Supports more than 8 proprietary whole slide formats
- Allows users to use the same basic functionality for images across multiple vendors

**Matlab**:
- Built-in support for whole slide images using the `imread` function
- Optional input parameters allow users to specify the pyramid level to access
- Can restrict the region to load based on user-supplied coordinates
- Not all image formats and magnifications are supported

### Image Management Systems

Image management systems are software platforms that offer the ability to organize and access images using image metadata, patient information, or other characteristics:

**Common Features**:
- Organize slides in a hierarchy (patient → case → specimen → slides)
- Integrated image viewers and analysis routines
- Ability to save and recall slide annotations
- Integration with information systems (LIS)
- Storage of computed data (e.g., Her2 score)
- Authentication and user management
- Modules that provide reports of results

**Examples**:
- PathcoreFlow (Pathcore, Toronto, Ontario, Canada)
- Aperio eSlideManager (Leica Biosystems)

Some hospital PACS and LIS systems may support image organization, although they typically have only limited support for the unique requirements of whole slide images.

## 11. Event Publishing

The viewer publishes events for integration:

- **Frame loading**: Started, completed, error
- **Viewport changes**: Pan, zoom, rotation
- **Selection**: ROI selection, measurement

Slim uses these events to update the UI (loading indicators, position displays, etc.).

### Example Events

The viewer publishes events that Slim can listen to:

- **FRAME_LOADING_STARTED**: A tile is being requested from DICOMweb
- **FRAME_LOADING_ENDED**: Tile successfully loaded and decoded
- **FRAME_LOADING_ERROR**: Failed to load a tile
- **Viewport changes**: User panned or zoomed
- **Selection**: User selected an ROI

Slim uses these events to update the UI (show loading indicators, update position displays, etc.).

<details>
<summary><em>Code example (optional - skip if not coding)</em></summary>

```javascript
// Frame loading started
EVENT.FRAME_LOADING_STARTED;

// Frame loading completed
EVENT.FRAME_LOADING_ENDED;

// Frame loading error
EVENT.FRAME_LOADING_ERROR;
```
</details>

## Next Steps

- [Common Use Cases](./07-common-use-cases.md) - Real-world scenarios
- [Troubleshooting](./08-troubleshooting.md) - Common issues and solutions

## References

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- DICOM Supplement 145 - Whole Slide Imaging in Pathology
- Zarella MD, Bowman D, Aeffner F, et al. A Practical Guide to Whole Slide Imaging: A White Paper From the Digital Pathology Association. Arch Pathol Lab Med. 2019;143(2):222-234. doi:10.5858/arpa.2018-0343-RA
- [OpenSlide](https://openslide.org/)
- [Bio-Formats](https://www.openmicroscopy.org/bio-formats/)
- [QuPath](https://qupath.github.io/)
- [ImageJ](https://imagej.nih.gov/ij/)
- [Fiji](https://fiji.sc/)

