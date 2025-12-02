# Troubleshooting

Common issues and solutions when working with pathology viewers.

## Images Not Loading

### Symptoms

- Blank viewer
- Tiles not appearing
- Loading spinner never stops

### Possible Causes

1. **Metadata issues**: Invalid or missing DICOM metadata
2. **DICOMweb connection**: Server unreachable or authentication failed
3. **Frame mapping errors**: Incorrect frame mapping
4. **Compression format**: Unsupported compression format

### Solutions

**Check metadata**:
```typescript
// Verify metadata structure
console.log('Metadata:', slide.volumeImages);
console.log('ImageType:', slide.volumeImages[0].ImageType);
console.log('FrameOfReferenceUID:', slide.volumeImages[0].FrameOfReferenceUID);
```

**Check DICOMweb connection**:
```typescript
// Test connection
const metadata = await client.retrieveSeriesMetadata({
  studyInstanceUID,
  seriesInstanceUID
});
console.log('Retrieved metadata:', metadata.length, 'instances');
```

**Check frame mapping**:
```typescript
// Verify frame mapping exists
const pyramid = _computeImagePyramid({ metadata: slide.volumeImages });
console.log('Frame mappings:', pyramid.frameMappings);
```

## Pyramid Not Building

### Symptoms

- Error: "Pyramid levels must share FrameOfReferenceUID"
- Error: "Pyramid levels must share ContainerIdentifier"
- Only one resolution level available

### Possible Causes

1. **Inconsistent UIDs**: Different `FrameOfReferenceUID` or `ContainerIdentifier` across levels
2. **Missing levels**: Not all pyramid levels present
3. **Wrong image type**: Non-VOLUME images included

### Solutions

**Verify UIDs**:
```typescript
// Check consistency
const frameOfReferenceUIDs = slide.volumeImages.map(img => img.FrameOfReferenceUID);
const containerIdentifiers = slide.volumeImages.map(img => img.ContainerIdentifier);

console.log('FrameOfReferenceUIDs:', [...new Set(frameOfReferenceUIDs)]);
console.log('ContainerIdentifiers:', [...new Set(containerIdentifiers)]);

// Should be single values
if (new Set(frameOfReferenceUIDs).size > 1) {
  console.error('Multiple FrameOfReferenceUIDs found!');
}
```

**Check image types**:
```typescript
// Filter VOLUME images only
const volumeImages = slide.volumeImages.filter(img => {
  return img.ImageType[2] === 'VOLUME';
});
console.log('VOLUME images:', volumeImages.length);
```

## Optical Paths Not Displaying

### Symptoms

- Only one channel visible
- Can't switch between channels
- Channel selector not appearing

### Possible Causes

1. **Missing OpticalPathSequence**: Metadata doesn't include optical path information
2. **Single optical path**: Only one optical path in image
3. **Frame mapping errors**: Incorrect frame mapping for optical paths

### Solutions

**Verify optical paths**:
```typescript
// Check optical paths
const opticalPaths = image.OpticalPathSequence;
console.log('Optical paths:', opticalPaths.length);
opticalPaths.forEach((path) => {
  console.log('Optical Path:', path.OpticalPathIdentifier);
});

// Set optical path visibility
viewer.setChannelVisibility(opticalPathIdentifier, true);

// Set optical path color
viewer.setChannelColor(opticalPathIdentifier, [255, 0, 0]); // Red
```

## Annotations Not Loading

### Symptoms

- Annotations don't appear
- Error loading annotations
- Annotations in wrong location

### Possible Causes

1. **Missing references**: Annotations don't reference correct image
2. **Coordinate mismatch**: Coordinates in wrong coordinate system
3. **Server issues**: Annotation server unreachable

### Solutions

**Check annotation references**:
```typescript
// Verify annotation references image
const referencedSOPInstanceUID = annotation.ReferencedSOPInstanceUID;
const imageSOPInstanceUID = slide.volumeImages[0].SOPInstanceUID;

if (referencedSOPInstanceUID !== imageSOPInstanceUID) {
  console.error('Annotation references wrong image!');
}
```

**Check coordinates**:
```typescript
// Verify coordinates are in physical (slide) coordinates
const coordinates = annotation.GraphicData; // Should be in mm
console.log('Coordinates:', coordinates);
```

## Performance Issues

### Symptoms

- Slow tile loading
- Laggy pan/zoom
- High memory usage

### Possible Causes

1. **Large cache**: Too many tiles cached
2. **Network issues**: Slow DICOMweb server
3. **Large images**: Very high resolution images
4. **Too many annotations**: Thousands of annotations

### Solutions

**Reduce cache size**:
```typescript
// Reduce tile cache
const viewer = new VolumeImageViewer({
  tilesCacheSize: 500, // Default: 1000
  // ...
});
```

**Preload lower resolutions**:
```typescript
// Preload lower resolution levels
const viewer = new VolumeImageViewer({
  preload: 2, // Preload 2 lowest resolution levels
  // ...
});
```

**Optimize annotations**:
```typescript
// Use bulk annotations for large numbers
// Instead of individual ROI annotations
```

## Color Issues

### Symptoms

- Colors look wrong
- Inconsistent colors across displays
- Color shifts

### Possible Causes

1. **Missing ICC profile**: ICC profile not applied
2. **Wrong ICC profile**: Incorrect profile for image
3. **Display calibration**: Monitor not calibrated

### Solutions

**Enable ICC profiles**:
```typescript
// Ensure ICC profiles are enabled
const viewer = new VolumeImageViewer({
  // ICC profiles enabled by default
  // ...
});
```

**Check ICC profile**:
```typescript
// Verify ICC profile exists
const iccProfile = image.ICCProfile;
if (!iccProfile) {
  console.warn('No ICC profile found');
}
```

## Coordinate Transformation Errors

### Symptoms

- Annotations in wrong location
- Measurements incorrect
- Coordinate conversion errors

### Possible Causes

1. **Wrong pixel spacing**: Incorrect `PixelSpacing` value
2. **Coordinate system mismatch**: Mixing pixel and physical coordinates
3. **Origin offset**: Incorrect `TotalPixelMatrixOriginSequence`

### Solutions

**Verify pixel spacing**:
```typescript
// Check pixel spacing
const pixelSpacing = getPixelSpacing(image);
console.log('Pixel spacing:', pixelSpacing); // Should be in mm

// Convert coordinates
const physicalX = pixelX * pixelSpacing[0];
const physicalY = pixelY * pixelSpacing[1];
```

**Check origin**:
```typescript
// Verify origin
const origin = image.TotalPixelMatrixOriginSequence[0];
console.log('Origin:', {
  x: origin.XOffsetInSlideCoordinateSystem,
  y: origin.YOffsetInSlideCoordinateSystem
});
```

## DICOMweb Connection Issues

### Symptoms

- Can't connect to server
- Authentication errors
- Timeout errors

### Possible Causes

1. **Wrong URL**: Incorrect DICOMweb endpoint
2. **Authentication**: Missing or invalid credentials
3. **CORS**: Cross-origin issues
4. **Network**: Server unreachable

### Solutions

**Check URL**:
```typescript
// Verify DICOMweb URL
const client = new DICOMwebClient({
  url: 'https://example.com/dicomweb', // Check this URL
  // ...
});
```

**Check authentication**:
```typescript
// Verify OAuth token
const token = getAuthToken();
if (!token) {
  console.error('No authentication token!');
}
```

**Check CORS**:
```typescript
// Verify CORS headers
// Server must allow requests from your domain
```

## Debugging Tips

### Enable Debug Mode

```typescript
// Enable debug features
const viewer = new VolumeImageViewer({
  debug: true, // Shows tile boundaries, etc.
  // ...
});
```

### Check Console Logs

```typescript
// Enable verbose logging
console.log('Pyramid:', pyramid);
console.log('Frame mappings:', frameMappings);
console.log('Tile requests:', tileRequests);
```

### Inspect Network Requests

- Open browser DevTools → Network tab
- Filter by "frames" or "metadata"
- Check request/response headers
- Verify WADO-RS URLs are correct

### Verify DICOM Metadata

```typescript
// Inspect metadata
console.log('Image metadata:', image.json);
console.log('Key attributes:', {
  SOPInstanceUID: image.SOPInstanceUID,
  TotalPixelMatrixColumns: image.TotalPixelMatrixColumns,
  TotalPixelMatrixRows: image.TotalPixelMatrixRows,
  PixelSpacing: getPixelSpacing(image),
  FrameOfReferenceUID: image.FrameOfReferenceUID,
  ContainerIdentifier: image.ContainerIdentifier
});
```

## Common Error Messages

### "Pyramid levels must share FrameOfReferenceUID"

**Cause**: Different `FrameOfReferenceUID` values across pyramid levels.

**Solution**: Ensure all VOLUME images in the same slide have the same `FrameOfReferenceUID`.

### "Pyramid levels must share ContainerIdentifier"

**Cause**: Different `ContainerIdentifier` values across pyramid levels.

**Solution**: Ensure all VOLUME images in the same slide have the same `ContainerIdentifier`.

### "No image metadata was provided"

**Cause**: Empty metadata array passed to viewer.

**Solution**: Verify metadata retrieval and filtering.

### "Failed to retrieve frame"

**Cause**: DICOMweb request failed (network, authentication, or server error).

**Solution**: Check DICOMweb connection, authentication, and server logs.

## Getting Help

- Check the [official DICOM WSI documentation](https://dicom.nema.org/dicom/dicomwsi/)
- Review [Key Concepts](./06-key-concepts.md) for technical details
- Consult [Glossary](./09-glossary.md) for terminology

