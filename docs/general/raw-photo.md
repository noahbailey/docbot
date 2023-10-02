# Processing RAW photos

## Required packages

    sudo apt install imagemagick dcraw libimage-exiftool-perl

## Processing

A bash function to automatically process a RAW photo with "sane defaults" on white balance, denoiseing, sharpening, and contrast. 

```
autoraw () {
    FILE=$(basename $1 .RW2)
    dcraw -w -c -v -n 200 -b ${2:-1} -T $FILE.RW2 | convert - -verbose -sigmoidal-contrast '4.00,50%' -sigmoidal-contrast '-5.00,0%' -adaptive-sharpen '0x4.0' -quality '90' $FILE.raw.jpg
    exiftool -verbose -overwrite_original -TagsFromFile $FILE.JPG $FILE.raw.jpg
}
```

To use: 

    autoraw ./0123456.RW2

To increase exposure:

    autoraw ./0123456.RW2 2

To decrease exposure: 

    autoraw ./0123456.RW2 0.5

Example: 

```
% autoraw P1190192.RW2 2.5

Loading Panasonic DMC-GX85 image from P1190192.RW2 ...
Wavelet denoising...
Scaling with darkness 143, saturation 4095, and
multipliers 2.496094 1.000000 1.503906 1.000000
AHD interpolation...
Converting to sRGB colorspace...
Writing data to standard output ...
-=>P1190192.RW2.jpg PPM 4608x3464 4608x3464+0+0 8-bit sRGB 5.43557MiB 35.120u 0:06.218
```

### Batch process raw files

```
autorawbatch () {
    for i in *.RW2; do autoraw $i; done
}
```

