# RAW Photos on Linux

## Previews in file manager

https://github.com/pop-os/pop/issues/1484
https://gist.github.com/h4cc/13450db3d4a7457f9b38?permalink_comment_id=3770740
https://support.system76.com/articles/fix-raw-image-previews/

    sudo apt install rawtherapee

Create a file in `/usr/share/thumbnailers/rawtherapee.thumbnailer ` with this:

```
[Thumbnailer Entry]
TryExec=/usr/bin/rawtherapee-cli
Exec=/usr/bin/rawtherapee-cli -s -n -Y -f -o %o -c %i 
MimeType=image/x-arw;image/x-bay;image/x-canon-cr2;image/x-canon-crw;image/x-cap;image/x-cr2;image/x-crw;image/x-dcr;image/x-dcraw;image/x-dcs;image/x-dng;image/x-drf;image/x-eip;image/x-erf;image/x-fff;image/x-fuji-raf;image/x-iiq;image/x-k25;image/x-kdc;image/x-mef;image/x-minolta-mrw;image/x-mos;image/x-mrw;image/x-nef;image/x-nikon-nef;image/x-nrw;image/x-olympus-orf;image/x-orf;image/x-panasonic-raw;image/x-panasonic-raw2;image/x-pef;image/x-pentax-pef;image/x-ptx;image/x-pxn;image/x-r3d;image/x-raf;image/x-raw;image/x-rw2;image/x-panasonic-rw2;image/x-rwl;image/x-rwz;image/x-samsung-srw;image/x-sigma-x3f;image/x-sony-arw;image/x-sony-sr2;image/x-sony-srf;image/x-sr2;image/x-srf;image/x-x3f;image/x-adobe-dng;image/x-portable-pixmap;image/tiff;
```

Clear the thumbnail cache:

    find ~/.cache/thumbnails/ -delete

Refresh the file browser.


## Processing RAW files

### Required packages

    sudo apt install imagemagick dcraw libimage-exiftool-perl

A bash function to automatically process a RAW photo with "sane defaults" on white balance, denoiseing, sharpening, and contrast, then add metadata to the jpeg file. 

```
autoraw () {
    FILE=$(basename $1 .RW2)
    dcraw -w -c -v -n 200 -b ${2:-1} -T $FILE.RW2 | convert - -verbose -contrast-stretch 0.5x1% -adaptive-sharpen '0x4.0' -quality '90' -auto-orient $FILE.raw.jpg
    exiftool -verbose -overwrite_original -TagsFromFile $FILE.RW2 $FILE.raw.jpg
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

