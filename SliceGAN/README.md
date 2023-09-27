# Preprocess for SliceGAN training

### Image processing
- Open an image by ImageJ.
- Image -> Type -> 8bit
- Select area by "Rectangle tool" as a square of 800x800 pixels (Use shift key).
- Image -> Crop
- Process -> Filters -> Median -> radius 3.0 pixels -> OK
- Image -> Adjust -> Threshold -> [x] Dark background, [ ] Stack histogram, [x] Don't reset range, [ ] Raw values -> Auto -> Apply
- Save as -> Tiff

### Store the preprocessed image file
- Make a directory as "./SliceGAN_mod/Trained_Generators/Project_dirname/Project_name/"
    - Project_dir = "Trained_Generators/Project_dirname"
    - Project_name = "Project_name"
- Copy the tiff file at "./SliceGAN_mod/Examples/XX/"
    - data_path = "Examples/XX/XXXXX.tif"

### Train by SliceGAN
```
python run_slicegan_64model.py 1
```
