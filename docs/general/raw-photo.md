# Convert RAW to JPG

Install prereq's

    sudo apt install imagemagick dcraw

Convert a single image from canon CR2 raw format to JPG:

    dcraw -c -a -h ./IMG_123456.CR2 | ppmtojpeg > IMG_123456.jpg

Convert many canon CR2 raw format photos to JPG: 

    for i in *.CR2; do 
        dcraw -c -a -h $i | ppmtojpeg > $i.jpg;
        echo $i
    done

## Batch process RAW files with adjustments

```
for i in *.RW2; do dcraw -w -c -v -n 200 $i | convert - -verbose -sigmoidal-contrast '4.00,50%' -sigmoidal-contrast '-5.00,0%' -adaptive-sharpen '0x4.0' -quality '90' $i.jpg; done
```

