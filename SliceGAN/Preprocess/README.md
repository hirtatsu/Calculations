# Preprocess for SliceGAN training

### Image processing
- Open an image by ImageJ.
- Image -> Type -> 8bit
- Select area by "Rectangle tool" as a square of 800x800 pixels (Use shift key).
- Image -> Crop
- Process -> Filters -> Median -> radius 3.0 pixels -> OK
- Image -> Adjust -> Threshold -> [x] Dark background, [ ] Stack histogram, [x] Don't reset range, [ ] Raw values -> Auto -> Apply
- Save as -> Tiff

