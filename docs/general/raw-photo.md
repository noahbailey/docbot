# Processing RAW photos

## Required packages

    sudo apt install imagemagick dcraw

## Processing

A bash function to automatically process a RAW photo with "sane defaults" on white balance, denoiseing, sharpening, and contrast. 

```
autoraw () {
    dcraw -w -c -v -n 200 -b ${2:-1} $1 | convert - -verbose -sigmoidal-contrast '4.00,50%' -sigmoidal-contrast '-5.00,0%' -adaptive-sharpen '0x4.0' -quality '90' $1.jpg
}
```

To use: 

    autoraw ./0123456.RW2

To increase exposure:

    autoraw ./0123456.RW2 2

To decrease exposure: 

    autoraw ./0123456.RW2 0.5

### Batch process raw files

```
autorawbatch () {
    for i in *.RW2; do autoraw $i; done
}
```

