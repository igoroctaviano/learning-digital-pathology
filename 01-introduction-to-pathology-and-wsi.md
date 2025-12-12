# Introduction to Pathology and Whole Slide Imaging

## What is Pathology?

Pathology is the medical specialty that studies the causes and effects of diseases by examining tissues, organs, and bodily fluids. Pathologists analyze samples under microscopes to diagnose diseases, determine treatment plans, and understand disease progression.

### Traditional Pathology Workflow

```
┌───────────────────┐
│ Sample Collection │
│  (Biopsy/Surgery) │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│   Processing      │
│ Fix → Embed → Cut │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    Staining       │
│  (H&E, IHC, etc.) │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│   Microscopy      │
│  Pathologist Exam │
└────────┬──────────┘
         │
         ▼
┌─────────────────┐
│  Documentation  │
│   (Report)      │
└─────────────────┘
```

**Steps**:
1. **Sample Collection**: A tissue sample (biopsy or surgical specimen) is collected
2. **Processing**: The sample is fixed, processed, embedded in paraffin, and sliced into thin sections
3. **Staining**: Sections are stained (commonly H&E - Hematoxylin and Eosin) to highlight different tissue structures
4. **Microscopy**: Pathologists examine slides under a microscope
5. **Documentation**: Findings are documented in reports

### The Digital Revolution: Whole Slide Imaging (WSI)

Whole Slide Imaging (WSI), also known as digital pathology, digitizes entire glass slides at high resolution. This enables:

```
┌─────────────────────────────────────────────────────┐
│         Benefits of Digital Pathology               │
└─────────────────────────────────────────────────────┘

✅ Remote Viewing      → Pathologists view slides from anywhere
✅ Collaboration       → Multiple pathologists review simultaneously  
✅ AI/ML Integration   → Computer algorithms analyze patterns
✅ Archiving           → Digital slides don't degrade over time
✅ Education           → Students access vast case libraries
```

**Key Capabilities**:
- **Remote viewing**: Pathologists can view slides from anywhere
- **Collaboration**: Multiple pathologists can review the same slide simultaneously
- **AI/ML integration**: Computer algorithms can analyze images for pattern recognition
- **Archiving**: Digital slides don't degrade over time
- **Education**: Students and trainees can access vast libraries of cases

## Whole Slide Scanning: How It Works

Whole slide imaging encompasses the digitization of entire histology slides or preselected areas. The process includes four sequential parts: image acquisition (scanning), storage, editing, and display of images.

### Scanner Components

Whole slide scanners consist of four main components:

```
┌─────────────────────────────────────────┐
│      Whole Slide Scanner                │
│                                         │
│  ┌──────────┐                           │
│  │  Light   │ ← Illumination            │
│  │  Source  │                           │
│  └────┬─────┘                           │
│       │                                 │
│       ▼                                 │
│  ┌──────────┐                           │
│  │  Slide   │ ← Holds glass slide       │
│  │  Stage   │                           │
│  └────┬─────┘                           │
│       │                                 │
│       ▼                                 │
│  ┌──────────┐                           │
│  │Objective │ ← Magnification (20X/40X) │
│  │  Lens    │                           │
│  └────┬─────┘                           │
│       │                                 │
│       ▼                                 │
│  ┌──────────┐                           │
│  │  Camera  │ ← Captures digital image  │
│  └──────────┘                           │
└─────────────────────────────────────────┘
```

**Components**:
1. **Light Source**: Provides illumination for image capture
2. **Slide Stage**: Holds and positions the slide during scanning
3. **Objective Lenses**: Provide magnification (typically 20X, 40X, or higher)
4. **High-Resolution Camera**: Captures the digital image

### Scanning Methods

Scanners capture images using one of two methods:

**Tile-by-Tile Scanning**:
```
┌─────┬─────┬─────┬─────┐
│ Tile│ Tile│ Tile│ Tile│  ← Captures overlapping tiles
├─────┼─────┼─────┼─────┤
│ Tile│ Tile│ Tile│ Tile│
├─────┼─────┼─────┼─────┤
│ Tile│ Tile│ Tile│ Tile│
└─────┴─────┴─────┴─────┘
         ↓
    [Stitching]
         ↓
┌────────────────────────┐
│   Complete Slide Image │
└────────────────────────┘
```
- Captures multiple overlapping images (tiles)
- Tiles are digitally assembled ("stitched") to create the complete slide image
- Used for both brightfield and fluorescent scanning
- Allows for individual tile focusing

**Line-Scanning**:
```
┌─────────────────────────┐
│ ─────────────────────── │  ← Continuous line capture
│ ─────────────────────── │
│ ─────────────────────── │
│ ─────────────────────── │
└─────────────────────────┘
         ↓
┌────────────────────────┐
│   Complete Slide Image │
└────────────────────────┘
```
- Captures images in continuous lines
- Faster than tile scanning for some applications
- Uses focus maps for focusing
- Typically used for brightfield imaging

### Focusing Strategies

Methods of focusing along the z-axis vary:

**Focusing Every Tile**:
- Most time-consuming but highest quality
- Each tile is individually focused
- Best for specimens with variable thickness

**Focusing Every Nth Tile**:
- Faster scanning with acceptable quality
- Focuses on selected tiles, interpolates for others
- Balance between speed and quality

**Focus Maps**:
- Network of focus points placed over the tissue surface
- Fastest scan times
- Higher potential error rate
- Can be manually set or automatically determined

**Continuous Automatic Refocusing**:
- Modern scanners incorporate continuous refocusing
- Further increases scan quality
- Balances speed and quality automatically

### Scan Times and Throughput

Scanning times vary based on several factors:

- **Per Slide**: Typically 30 seconds to several minutes per slide
- **High-Throughput Scanners**: Can hold up to 400 slides in loading units
- **Factors Affecting Time**:
  - Higher magnification (40X vs 20X) increases scan time
  - Larger tissue sections take longer
  - Fluorescent images take longer than brightfield
  - File size increases with scan time

### Magnification Selection

Different applications require different magnifications:

- **20X Magnification**: Usually acceptable for standard viewing and interpretation, including routine H&E and IHC slides
- **40X Magnification**: Required for digitization of in situ hybridization slides to resolve information separated by distances less than about 0.5 μm
- **60X/100X (Oil Immersion)**: Available on select scanners, typically only recommended for specific use cases such as blood smears

Some scanners can accommodate both dry scanning and oil immersion during the digitization process.

### Tissue Recognition

Most modern scanners incorporate tissue recognition features:

- **Automatic Detection**: Low-magnification overview scan detects histology specimen
- **Efficiency**: Greatly increases scanner efficiency by identifying regions to scan
- **Blank Region Removal**: Discards blank regions to reduce file size and scan time

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

| Magnification | Microns per Pixel | Use Case |
|--------------|-------------------|----------|
| **40X** | 0.25 mpp | Highest detail, diagnostic review |
| **20X** | 0.5 mpp | Standard viewing, routine H&E |
| **10X** | 1.0 mpp | Low magnification, overview |
| **4X** | 2.5 mpp | Navigation, tissue overview |
| **2X** | 5.0 mpp | Quick scanning, orientation |

**Extreme Example** (worst case):
- Sample size: 50mm × 25mm
- Resolution: 0.1 mpp (oil immersion, "100X")
- Multiple Z-planes: 10 focal planes
- Result: **500,000 × 250,000 pixels per plane** = **125 gigapixels per plane**
- Total dataset: **3.75 TB** (10 planes × 375 GB each)

### Why Pyramids?

**The Problem**: Loading a 15 GB image into memory is impossible. Even viewing a small region requires efficient access patterns.

**The Solution**: Store images as **multi-resolution pyramids**:

```
                    ┌─────────┐
                    │ Level 4 │  5.0 mpp  (2X)   - Fast overview
                    │  Small  │
                    └────┬────┘
                         │
                    ┌────▼────┐
                    │ Level 3 │  2.5 mpp  (4X)   - Navigation
                    │ Medium  │
                    └────┬────┘
                         │
                    ┌────▼────┐
                    │ Level 2 │  1.0 mpp  (10X)  - Low mag view
                    │  Large  │
                    └────┬────┘
                         │
                    ┌────▼────┐
                    │ Level 1 │  0.5 mpp  (20X)  - Medium mag
                    │ Larger  │
                    └────┬────┘
                         │
                    ┌────▼────┐
                    │ Level 0 │  0.25 mpp (40X)  - Full detail
                    │ Largest │  15 GB uncompressed
                    └─────────┘
```

**Pyramid Levels**:
- **Level 0** (base): Highest resolution, full detail (e.g., 0.25 mpp)
- **Level 1**: 2× smaller (e.g., 0.5 mpp)
- **Level 2**: 4× smaller (e.g., 1.0 mpp)
- **Level 3**: 8× smaller (e.g., 2.5 mpp)
- **Level 4**: 16× smaller (e.g., 5.0 mpp)

This allows rapid zooming without processing large amounts of data.

### Why Tiling?

Even at a single pyramid level, loading the entire image is impractical. Instead, images are divided into **tiles** (typically 256×256 or 512×512 pixels):

```
Full Slide Image (80,000 × 60,000 pixels)
┌───────────────────────────────────────┐
│ ┌─────┬─────┬─────┬─────┬─────┬─────┐ │
│ │  1  │  2  │  3  │  4  │  5  │  6  │ │  Row 1
│ ├─────┼─────┼─────┼─────┼─────┼─────┤ │
│ │  7  │  8  │  9  │ 10  │ 11  │ 12  │ │  Row 2
│ ├─────┼─────┼─────┼─────┼─────┼─────┤ │
│ │ 13  │ 14  │ 15  │ 16  │ 17  │ 18  │ │  Row 3
│ ├─────┼─────┼─────┼─────┼─────┼─────┤ │
│ │ ... │ ... │ ... │ ... │ ... │ ... │ │  ...
│ └─────┴─────┴─────┴─────┴─────┴─────┘ │
└───────────────────────────────────────┘
     ↑ Each tile: 512×512 pixels
     
Viewport (only loads visible tiles)
┌──────────┐
│  8  │  9 │  ← Only these tiles loaded
│ 14  │ 15 │
└──────────┘
```

**Benefits**:
- ✅ Random access to any subregion without loading the entire image
- ✅ Only visible tiles are loaded as the user pans and zooms
- ✅ Similar to how Google Maps works
- ✅ Efficient memory usage

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

### Scanning Modalities

When pairing scanners with slide staining techniques, WSI can be categorized as:

**Brightfield Scanning**:
- Emulates standard brightfield microscopy
- Most common and cost-effective approach
- Used for H&E and chromogenic IHC slides

**Fluorescent Scanning**:
- Akin to fluorescent microscopy
- Used to digitize fluorescently labeled slides (fluorescent IHC, FISH)
- Always captures images as tiles
- Requires different illumination and filters

**Multispectral Imaging**:
- Captures spectral information across the spectrum of light
- Can be applied to both brightfield and fluorescent settings
- Enables spectral unmixing and advanced analysis
- Some scanners can accommodate multiple modalities

### Multi-Spectral Imaging

Some scanners capture multiple spectral bands:

- Up to 10 spectral bands possible
- 16-bit per pixel resolution
- Each band stored as a separate optical path/channel
- Enables spectral unmixing for better separation of overlapping signals
- As storage becomes more affordable, routine storage of multispectral images becomes more feasible

### Optical Paths (Channels)

Different ways the slide was imaged:

- **Brightfield**: H&E staining (most common)
- **Fluorescence**: Multiple channels (DAPI, FITC, Rhodamine, etc.)
- **Multiplexed imaging**: Many channels (CycIF - Cyclic Immunofluorescence)

### Quality Control Considerations

When establishing a workflow for digital pathology, additional steps are needed beyond traditional histology:

- **Additional Equipment**: Scanners, viewing workstations, storage systems
- **Staff Training**: Proper training of personnel on scanning and viewing
- **Quality Control Steps**: Confirming scan quality, checking for artifacts
- **Equipment Maintenance**: Regular maintenance of scanners and software
- **IT Infrastructure**: Adequate network and storage infrastructure
- **Pathologist Workstation Setup**: Proper monitors and viewing environment

Scanning artifacts can affect downstream results and can be caused by:
- Improper cleaning of slides prior to scanning
- Poorly focused scans
- Compensation lines from improper stitching of lines or tiles
- Other technical factors

## Next Steps

Now that you understand the basics of pathology and WSI, let's dive into:

- [DICOM WSI Format Deep Dive](./02-dicom-wsi-format.md)
- [How dicom-microscopy-viewer Works](./03-dicom-microscopy-viewer-engine.md)
- [How Slim Viewer Works](./04-slim-viewer-application.md)

## References

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- DICOM Supplement 145 - Whole Slide Imaging in Pathology
- Zarella MD, Bowman D, Aeffner F, et al. A Practical Guide to Whole Slide Imaging: A White Paper From the Digital Pathology Association. Arch Pathol Lab Med. 2019;143(2):222-234. doi:10.5858/arpa.2018-0343-RA

