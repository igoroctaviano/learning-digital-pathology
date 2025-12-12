# Future Trends & Next Steps (AI)

This document explores emerging trends in digital pathology, focusing on AI and machine learning applications, evolving DICOM standards, and the role of DICOM in telepathology.

## AI & Machine Learning Applications Using DICOM-Based Pathology Images

Artificial intelligence and machine learning are transforming digital pathology, and DICOM standardization plays a crucial role in enabling these applications.

### Diagnostic Assistance

AI algorithms can assist pathologists in diagnosis:

**Pattern Recognition in Pathology Images**:
- Automated detection of abnormal patterns
- Identification of specific tissue structures
- Detection of pathological features
- Pattern classification

**Cancer Detection and Grading**:
- Automated cancer detection (e.g., breast cancer, prostate cancer)
- Cancer grading (e.g., Gleason scoring for prostate cancer)
- Tumor classification
- Margin assessment

**Automated Detection of Specific Features**:
- Detection of specific cell types
- Identification of biomarkers
- Quantification of features
- Feature extraction for analysis

### Prognostic Assessments

AI can predict patient outcomes from pathology images:

**Predicting Patient Outcomes**:
- Survival prediction
- Recurrence risk assessment
- Treatment response prediction
- Prognostic biomarker identification

**Biomarker Quantification**:
- Automated biomarker quantification
- Expression level measurement
- Biomarker pattern analysis
- Multi-marker analysis

**Treatment Response Prediction**:
- Predicting response to specific treatments
- Treatment selection guidance
- Personalized treatment recommendations
- Response monitoring

### Personalized Treatment Planning

AI enables personalized medicine approaches:

**AI-Guided Treatment Decisions**:
- Treatment recommendation based on image analysis
- Personalized treatment plans
- Treatment optimization
- Risk-benefit analysis

**Patient Stratification**:
- Patient grouping based on image features
- Risk stratification
- Treatment response prediction
- Outcome prediction

### Quality Control Automation

AI can automate quality control processes:

**Automated Quality Checks for Pathology Slides**:
- Focus quality assessment
- Artifact detection
- Image quality validation
- Quality metrics calculation

**Focus Quality Assessment**:
- Automated focus quality checking
- Out-of-focus detection
- Focus quality scoring
- Re-scan recommendations

**Artifact Detection**:
- Detection of artifacts (folds, bubbles, etc.)
- Quality issue identification
- Automated quality reporting
- Quality improvement recommendations

### Research Applications

AI enables large-scale research applications:

**Large-Scale Pathology Image Analysis**:
- Analysis of large image datasets
- Population-level studies
- Cohort analysis
- Comparative studies

**Biomarker Discovery**:
- Identification of new biomarkers
- Biomarker pattern discovery
- Predictive biomarker identification
- Biomarker validation

**Population Studies**:
- Epidemiological studies
- Disease prevalence studies
- Outcome studies
- Comparative effectiveness research

### DICOM's Role in AI Workflows

DICOM standardization is essential for AI applications:

**Standardized Input Format Enables AI Model Training**:
- Consistent data format for training
- Easier dataset preparation
- Reproducible training processes
- Standardized preprocessing

**Reproducibility Across Institutions**:
- Same format across institutions
- Reproducible results
- Comparable studies
- Standardized evaluation

**Interoperability with AI Platforms**:
- AI platforms can work with DICOM format
- Standardized interfaces
- Easier integration
- Vendor-neutral AI solutions

**Pathology-Specific AI Use Cases**:
- H&E analysis (hematoxylin and eosin staining)
- IHC quantification (immunohistochemistry)
- Special stain analysis
- Multi-stain analysis

### Image Analysis Workflows

A typical image analysis workflow involves two key steps:

**Step 1: Region of Interest Identification**:
Most tissue samples contain a mix of host tissue, target tissue, blood vessels, stroma, tumor, organ compartments, and so forth. It is important to delineate the target compartment in the most relevant region, as the biomarker of interest may exist in other tissue compartments that are not relevant to the current study.

**Step 2: Cellular Analysis**:
Once the region of interest is defined, the analysis algorithm is configured and tested to optimize accurate segmentation of cells and measurement. Common user-configurable parameters include:
- **Color**: Thresholds for color-based segmentation
- **Threshold**: Intensity thresholds for detection
- **Categorical Thresholds**: Multiple thresholds for classification
- **Object Size**: To split adjacent or overlapping nuclei if the algorithm has an expectation of nuclear size

**Optimization Process**:
Image analysis algorithm optimization is often an iterative process:
1. User adjusts the parameters
2. Runs the algorithm on representative images
3. Assesses results
4. Re-optimizes if necessary

**Algorithm Development**:
Image analysis algorithms are often developed for specific applications or stains:
- In situ hybridization (chromogenic or fluorescent)
- Nuclear/cytoplasmic/membrane biomarkers
- Neurite outgrowth
- Other specific applications

**Quality Control**:
Image analysis results can vary depending on a number of factors:
- Staining quality
- Image capture quality
- Tissue quality

It is important to develop appropriate quality control measures to assess potential artifacts throughout the IHC and image analysis workflow. Some of the most common image analysis artifacts include:
- **Segmentation Errors**: Oversegmentation or undersegmentation of nuclei
- **Classification Errors**: Classifying tumor as stroma and vice versa

**Pathologist Involvement**:
It is important that a pathologist be involved in the image-analysis workflow, which should include at least one quality control step where the pathologist reviews all aspects that can influence image analysis results.

## Emerging Technologies and DICOM Standards Development

DICOM standards continue to evolve to support emerging technologies in digital pathology.

### DICOM Supplement 145 Updates

Ongoing improvements to the WSI standard:

**Ongoing Improvements for WSI**:
- Enhanced metadata support
- Better compression support
- Improved performance
- Extended capabilities

**Enhanced Metadata Support**:
- Richer metadata structures
- More metadata fields
- Better metadata organization
- Enhanced metadata validation

**Better Compression Support**:
- Improved compression algorithms
- Better compression quality
- More compression options
- Compression optimization

### Multi-Dimensional Imaging Support

Support for complex imaging scenarios:

**Z-Stacks (Multiple Focal Planes)**:
- Support for multiple focal planes
- Z-coordinate handling
- Multi-plane viewing
- Focus stacking

**Time-Lapse Imaging**:
- Support for time-series images
- Temporal metadata
- Time-lapse viewing
- Temporal analysis

**Multiplexed Imaging (Many Channels)**:
- Support for many channels (20+)
- Channel organization
- Channel blending
- Multi-channel analysis

### Multispectral Imaging

Multispectral imaging captures spectral information across the spectrum of light and can be applied to both brightfield and fluorescent settings.

**Technical Capabilities**:
- Captures spectral information at multiple wavelengths
- Enables spectral unmixing for better separation of overlapping signals
- Can resolve signals that appear similar in standard imaging
- Provides richer data for analysis

**Applications**:
- **Brightfield Pathology**: Spectral unmixing of chromogenic stains
- **Fluorescent Imaging**: Better separation of overlapping fluorophores
- **Biomarker Analysis**: Enhanced detection and quantification
- **Research Applications**: Advanced analysis capabilities

**Storage Considerations**:
- Multispectral images require more storage than standard images
- As storage continues to become more affordable, routine storage of multispectral whole slide images becomes a more realizable goal
- Vendors must place additional emphasis on improving whole slide scan times to accommodate the multiple passes often necessary to collect images at different wavelengths

**Future Developments**:
Some groups have begun to develop robust tools to rapidly acquire multispectral images without requiring multiple passes, which could be harnessed in WSI systems to improve scan time and optical throughput. With continued development, multispectral acquisition and storage may become ubiquitous in WSI, resulting in an overall expansion of analysis capabilities available to pathologists.

### Enhanced Metadata Capabilities

Richer metadata for pathology images:

**Rich Pathology-Specific Metadata**:
- Staining protocol information
- Scanner-specific metadata
- Acquisition parameters
- Quality metrics

**Staining Protocol Information**:
- Staining method details
- Reagent information
- Protocol parameters
- Quality control information

**Scanner-Specific Metadata Preservation**:
- Preserve vendor-specific metadata
- Scanner calibration information
- Acquisition settings
- Equipment information

### Cloud Storage Integration

Cloud-based solutions for pathology images:

**Cloud-Based DICOM Archives for Pathology**:
- Cloud storage for WSI images
- Scalable storage solutions
- Cost-effective storage
- Global accessibility

**Scalability Benefits**:
- Elastic scaling
- Pay-as-you-go pricing
- No upfront infrastructure costs
- Global distribution

**Cost Considerations**:
- Storage costs
- Data transfer costs
- Compute costs
- Cost optimization strategies

### Real-Time Streaming

Live pathology image streaming:

**Live Pathology Image Streaming**:
- Real-time image streaming
- Live viewing capabilities
- Streaming protocols
- Quality of service

**Telepathology Applications**:
- Real-time telepathology
- Live consultation
- Streaming workflows
- Interactive viewing

### Integration with AI/ML Platforms

DICOM as input format for AI systems:

**DICOM as Input Format for AI Systems**:
- Standardized input for AI models
- Easier AI integration
- Consistent data format
- Reproducible AI workflows

**Standardized Interfaces**:
- Standard APIs for AI integration
- Consistent interfaces
- Easier development
- Vendor-neutral solutions

**Research Reproducibility**:
- Reproducible research workflows
- Standardized data formats
- Comparable studies
- Reproducible results

### 3D Reconstruction of Whole Slide Images

Historically, WSI has been marketed and used mainly as a tool for 2-dimensional analysis, mimicking existing histopathology workflows. However, examination of 3-D structures in 2 dimensions generates a well-known information gap between recorded observations and the true state of the original tissue.

**Clinical Value**:
3-D reconstruction of whole slide histologic data is becoming more relevant and has demonstrated value in:
- **Tissue Visualization**: Better understanding of 3D tissue structure
- **Clinical Diagnosis**: Improved diagnostic accuracy
- **Correlation with Imaging Modalities**: Better correlation with MRI, CT, and other imaging modalities
- **Pattern Discovery**: Improved discovery of diagnostic patterns

**Process Overview**:

**Step 1: Serial Sectioning**:
- Generate serial, ultrathin tissue sections
- Sections mounted to glass slides
- Can be obtained manually or using automated robotic microtomes

**Automated Sectioning Benefits**:
- Near-uniform thickness of sections
- Uniform alignment of sections
- Fewer sectioning artifacts
- Critical for facilitating interpolation of structures between sections

**Step 2: Staining and Scanning**:
- Serial sections stained using routine histologic and/or IHC techniques
- Stained serial sections scanned using WSI
- Generates a z-stack of digital images, each corresponding to a different scanned level of the tissue block

**Step 3: 3D Reconstruction**:
Serial whole slide images are run through commercially available or custom software to generate 3-D models. Most 3-D reconstruction software involves several image processing steps:
- **Image Registration**: Aligning serial sections
- **Segmentation**: Identifying structures of interest
- **Interpolation**: Filling gaps between sections
- **Volumetric Rendering**: Creating 3D visualization

**Use Cases**:
- Tumor visualization and analysis
- Vascular structure analysis
- Organ structure analysis
- Research applications requiring 3D understanding

## Telepathology as Facilitated by DICOM: Expanding Access to Pathology Expertise

DICOM standards enable telepathology, expanding access to pathology expertise globally.

### Remote Consultation Workflows

DICOM enables various remote consultation scenarios:

**Pathologist-to-Pathologist Consultation**:
- Second opinion consultations
- Expert consultations
- Collaborative diagnosis
- Case discussion

**Second Opinion Workflows**:
- Requesting second opinions
- Expert review workflows
- Consultation management
- Opinion documentation

**Expert Consultation for Rare Cases**:
- Access to rare disease experts
- Specialized expertise access
- Expert network utilization
- Knowledge sharing

### DICOMweb for Telepathology

Web-based pathology image sharing:

**Web-Based Pathology Image Sharing**:
- Browser-based image sharing
- Web viewer integration
- Easy access from anywhere
- No special software required

**DICOMweb RESTful APIs**:
- Standard REST APIs
- Easy integration
- Web-friendly protocols
- Standardized interfaces

**Browser-Based Viewing**:
- View images in web browsers
- No installation required
- Cross-platform support
- Easy access

### Quality of Service Considerations

Network requirements for WSI transmission:

**Network Requirements for WSI Transmission**:
- High bandwidth requirements
- Low latency needs
- Reliable connections
- Quality of service guarantees

**Bandwidth Requirements**:
- Multi-gigabyte file transfers
- Streaming requirements
- Concurrent users
- Bandwidth planning

**Latency Considerations**:
- Real-time viewing requirements
- Interactive viewing needs
- Latency optimization
- User experience considerations

**Image Quality Preservation**:
- Maintaining diagnostic quality
- Compression considerations
- Quality vs. bandwidth trade-offs
- Quality assurance

### Geographic Access Expansion

DICOM enables geographic expansion of pathology services:

**Rural/Underserved Area Access to Pathology Expertise**:
- Bringing expertise to underserved areas
- Remote pathology services
- Telepathology networks
- Access expansion

**International Collaboration**:
- Cross-border consultations
- International expert networks
- Global pathology services
- Knowledge sharing

**Time Zone Advantages**:
- 24/7 pathology services
- Time zone coverage
- Continuous service
- Global coverage

### Collaborative Diagnosis

Multi-pathologist collaboration:

**Multi-Pathologist Review Workflows**:
- Multiple pathologists reviewing same case
- Collaborative diagnosis
- Consensus building
- Case discussion

**Annotation Sharing**:
- Sharing annotations between pathologists
- Collaborative annotation
- Annotation discussion
- Consensus annotations

**Consensus Building**:
- Building diagnostic consensus
- Discussion workflows
- Consensus documentation
- Quality improvement

### Pathology-Specific Telepathology Challenges

Telepathology faces unique challenges in pathology:

**Large File Sizes (Gigabytes per Slide)**:
- Transferring large files
- Storage requirements
- Bandwidth needs
- Cost considerations

**Diagnostic Quality Requirements**:
- Maintaining diagnostic quality
- Quality assurance
- Quality validation
- Quality metrics

**Regulatory Considerations**:
- Cross-border regulations
- Licensing requirements
- Legal considerations
- Compliance requirements

**Workflow Integration**:
- Integrating with existing workflows
- LIS integration
- Workflow optimization
- User adoption

## Next Steps

- [DICOM Storage & VNA](./10-dicom-storage-and-vna.md) - Storage solutions for pathology images
- [DICOM Converters](./11-dicom-converters.md) - Converting proprietary formats to DICOM
- [Barriers to DICOM Adoption](./12-barriers-to-dicom-adoption.md) - Challenges in adopting DICOM
- [IHE DPIA Integration Profile](./13-ihe-dpia-integration-profile.md) - Standardized acquisition workflows

## References

- [DICOM WSI Official Documentation](https://dicom.nema.org/dicom/dicomwsi/)
- [DICOM Standard](http://www.dicomstandard.org/current)
- DICOM Supplement 145 - Whole Slide Imaging in Pathology
- [AI in Pathology Research](https://www.nature.com/articles/s41591-021-01614-0)
- [Telepathology Guidelines](https://www.cap.org/member-resources/articles/telepathology-guidelines)
- Zarella MD, Bowman D, Aeffner F, et al. A Practical Guide to Whole Slide Imaging: A White Paper From the Digital Pathology Association. Arch Pathol Lab Med. 2019;143(2):222-234. doi:10.5858/arpa.2018-0343-RA

