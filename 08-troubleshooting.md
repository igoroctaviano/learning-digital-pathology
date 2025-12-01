# Troubleshooting Common Issues

## Issue: Images Not Loading

### Symptoms

- Blank viewer
- Error messages about failed frame retrieval
- Tiles not appearing

### Possible Causes

1. **DICOMweb Server Unreachable**

   - Check server URL in configuration
   - Verify network connectivity
   - Check CORS settings

2. **Authentication Issues**

   - Verify OAuth tokens are valid
   - Check token expiration
   - Verify scopes/permissions

3. **Metadata Issues**
   - Verify DICOM metadata is valid
   - Check for required attributes
   - Verify FrameOfReferenceUID matches

### Solutions

```javascript
// Check DICOMweb client configuration
const client = new DICOMwebClient({
  url: "https://your-server.com/dicomweb",
  headers: {
    Authorization: "Bearer " + token,
  },
});

// Verify metadata retrieval
const metadata = await client.retrieveSeriesMetadata({
  studyInstanceUID: "...",
  seriesInstanceUID: "...",
});
console.log("Metadata:", metadata);
```

## Issue: Pyramid Levels Not Matching

### Symptoms

- Segmentation/annotations don't align with image
- Overlays appear in wrong location
- Zoom levels don't match

### Possible Causes

1. **Different Pixel Spacings**

   - Image and overlay have different pixel spacings
   - Pyramid levels don't align

2. **Missing Pyramid Levels**
   - Overlay has fewer pyramid levels than image
   - Some zoom levels missing

### Solutions

```javascript
// Verify pixel spacings match
const imagePixelSpacing = getPixelSpacing(imageMetadata);
const overlayPixelSpacing = getPixelSpacing(overlayMetadata);

if (!areArraysEqual(imagePixelSpacing, overlayPixelSpacing)) {
  console.warn("Pixel spacings do not match");
}

// Use pyramid fitting
const [fittedPyramid] = _fitImagePyramid(overlayPyramid, imagePyramid);
```

## Issue: Annotations Not Displaying

### Symptoms

- Annotations don't appear
- ROI overlays missing
- Error messages about annotation loading

### Possible Causes

1. **Coordinate System Mismatch**

   - Annotations in wrong coordinate system
   - FrameOfReferenceUID mismatch

2. **Annotation Format Issues**

   - Invalid DICOM SR structure
   - Missing required attributes

3. **Client Mapping Issues**
   - Annotations stored in different server
   - Client mapping not configured

### Solutions

```typescript
// Verify FrameOfReferenceUID matches
const imageFOR = slide.volumeImages[0].FrameOfReferenceUID;
const annotationFOR = annotation.ReferencedFrameOfReferenceUID;

if (imageFOR !== annotationFOR) {
  console.error("FrameOfReferenceUID mismatch");
}

// Check client mapping
const clients = {
  default: imageClient,
  [StorageClasses.COMPREHENSIVE_3D_SR]: annotationClient,
};
```

## Issue: Performance Problems

### Symptoms

- Slow tile loading
- Laggy pan/zoom
- High memory usage

### Possible Causes

1. **Too Many Tiles Cached**

   - Cache size too large
   - Not clearing cache

2. **Preloading Too Many Levels**

   - Preloading all pyramid levels
   - Loading unnecessary tiles

3. **Network Issues**
   - Slow DICOMweb server
   - Large tile sizes

### Solutions

```javascript
// Reduce cache size
const viewer = new VolumeImageViewer({
  tilesCacheSize: 500, // Reduce from default 1000
  metadata: images,
});

// Disable preloading
const viewer = new VolumeImageViewer({
  preload: false, // Don't preload
  metadata: images,
});

// Use smaller tiles (if possible)
// Configure scanner to use 256x256 instead of 512x512
```

## Issue: Color Display Issues

### Symptoms

- Colors look wrong
- Images appear washed out
- Inconsistent colors across slides

### Possible Causes

1. **Missing ICC Profiles**

   - ICC profiles not applied
   - Color space not corrected

2. **Display Calibration**
   - Monitor not calibrated
   - Color profile issues

### Solutions

```javascript
// Enable ICC profiles
const viewer = new VolumeImageViewer({
  metadata: images,
  // ICC profiles enabled by default
});

// Verify ICC profile exists
const iccProfile = image.OpticalPathSequence[0].ICCProfile;
if (!iccProfile) {
  console.warn("ICC profile missing");
}
```

## Issue: Multi-Channel Display Problems

### Symptoms

- Channels not displaying
- Wrong colors for channels
- Channels not blending correctly

### Possible Causes

1. **Channel Identifier Mismatch**

   - Wrong channel identifier used
   - Channel not found

2. **Blending Mode Issues**
   - Additive blending not working
   - Color mapping incorrect

### Solutions

```typescript
// Verify optical path identifiers
const opticalPaths = image.OpticalPathSequence;
opticalPaths.forEach((path) => {
  console.log("Optical Path:", path.OpticalPathIdentifier);
});

// Set optical path visibility
viewer.setChannelVisibility(opticalPathIdentifier, true);

// Set optical path color
viewer.setChannelColor(opticalPathIdentifier, [255, 0, 0]); // Red
```

## Issue: Concatenation Problems

### Symptoms

- Missing tiles
- Incomplete image display
- Frame mapping errors

### Possible Causes

1. **Concatenation Not Reassembled**

   - Parts not properly linked
   - Missing concatenation metadata

2. **Frame Offset Issues**
   - Frame numbers don't match
   - Offset calculation wrong

### Solutions

```javascript
// Verify concatenation metadata
if (metadata.SOPInstanceUIDOfConcatenationSource) {
  const concatenationUID = metadata.ConcatenationUID;
  const partNumber = metadata.InConcatenationNumber;
  const frameOffset = metadata.ConcatenationFrameOffsetNumber;

  console.log("Concatenation:", {
    uid: concatenationUID,
    part: partNumber,
    offset: frameOffset,
  });
}
```

## Issue: Authentication Failures

### Symptoms

- 401 Unauthorized errors
- Token expiration errors
- OAuth flow failures

### Possible Causes

1. **Token Expired**

   - Access token expired
   - Refresh token invalid

2. **Scope Issues**

   - Missing required scopes
   - Insufficient permissions

3. **OAuth Configuration**
   - Wrong client ID
   - Incorrect redirect URI

### Solutions

```javascript
// Check token expiration
const token = getAccessToken();
if (isTokenExpired(token)) {
  await refreshToken();
}

// Verify scopes
const requiredScopes = ["https://www.googleapis.com/auth/cloud-healthcare"];
verifyScopes(token, requiredScopes);

// Check OAuth configuration
const oidcConfig = {
  authority: "https://accounts.google.com",
  clientId: "your-client-id",
  scope:
    "email profile openid https://www.googleapis.com/auth/cloud-healthcare",
};
```

## Debugging Tips

### Enable Debug Mode

```javascript
const viewer = new VolumeImageViewer({
  debug: true, // Shows tile boundaries
  metadata: images,
});
```

### Check Console Logs

```javascript
// Enable verbose logging
console.info("Loading tile:", { z, y, x, channel });
console.info("Pyramid:", pyramid);
console.info("Frame mapping:", frameMapping);
```

### Verify Metadata

```javascript
// Check image metadata
const image = new VLWholeSlideMicroscopyImage({ metadata });
console.log("Image type:", image.ImageType);
console.log("Dimensions:", {
  total: [image.TotalPixelMatrixColumns, image.TotalPixelMatrixRows],
  tile: [image.Columns, image.Rows],
});
console.log("Pixel spacing:", getPixelSpacing(image));
```

## Getting Help

1. **Check Logs**: Browser console and network tab
2. **Verify Metadata**: Use DICOM viewer to check DICOM files
3. **Test DICOMweb**: Use curl/Postman to test endpoints
4. **Check Documentation**: Review DICOM standard and library docs
5. **Report Issues**: Include error messages, metadata samples, steps to reproduce
