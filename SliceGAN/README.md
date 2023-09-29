# SliceGAN training procedure

### Image processing
- Open an image by ImageJ.
- Image -> Type -> 8bit
- Measure scale bar to know the scale information of "um/pixel"
- Analyze -> Set Scale
- Select area by "Rectangle tool" as a square of 880x880 pixels (Use shift key), with left-top as origin.
- Image -> Crop

![crop](Threshold.png "Threshold")


- Process -> Filters -> Median -> radius 3.0 pixels (x1000), 1.3 pixels (x300) -> OK
- Image -> Adjust -> Threshold
    - [x] Dark background
    - [ ] Stack histogram
    - [ ] Don't reset range
    - [ ] Raw values          -> Threshold adjustment -> Apply
    - memo for threshold values (x300, x1000)
        - 58Bi: Auto, Auto
        - 40Bi: Auto, threshold=193
        - 20Bi: threshold=193, 193
- Save as -> Tiff

### Image analysis by ImageJ
- Analyze -> Set Measurements
    - Area
    - Centoid
    - Perimeter
    - Fit ellipse
    - Shape descriptors
        - Circularity (Circ.)
        - Aspect ratio (AR)
        - Roundness
    - Area fraction
- Analyze -> Analyze Particles
    - Size: 0-Infinity
    - Circularity:0.0-1.0
    - Show: Outlines
    - [x] Display results
    - [x] Clear results
    - [ ] Summarize
    - [ ] Add to Manager
    - [ ] Exclude on edges
    - [ ] Include holes
    - [ ] Overlay
    - [ ] Composite ROIs
- Save
    - Results (csv file)
    - Outline image (capture)

### Store the preprocessed image file
- Make a directory as "./SliceGAN_mod/Trained_Generators/Project_dirname/Project_name/"
    - Project_dir = "Trained_Generators/Project_dirname"
    - Project_name = "Project_name"
- Copy the tiff file at "./SliceGAN_mod/Examples/XX/"
    - data_path = "Examples/XX/XXXXX.tif"

### Train by SliceGAN
```
python run_slicegan_64model 1 # 通常解像度
python run_slicegan_128model 1 # 高解像度
```
