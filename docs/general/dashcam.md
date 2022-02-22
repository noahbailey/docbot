# Transcode AUKEY dashcam video

Dashcams record a raw MPEG-2 stream. This isn't typically playable using usual multimedia software. 

First, make sure ffmpeg is installed: 

    sudo apt install ffmpeg

    sudo yay -S ffmpeg

The memory card from the dashcam should be mounted:

    sudo mount /dev/mmc0 /mnt/mmc

To transcode to a standard H.264 encoded MPEG-4 file, use ffmpeg:

    ffmpeg -i /mnt/mmc/AUKEY/movie/input_file.TS -c:v libx264 -c:a copy ~/Desktop/output_file.mp4

The resulting file will work fine with VLC or any other regular video player. 
