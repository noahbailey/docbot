# Transcode dashcam video

Dashcams record a raw MPEG-2 stream. This isn't typically playable using usual multimedia software. 
Transcoding down to a regular compressed h264 file also saves a massive amount of storage space. 

### Install ffmpeg tools

First, make sure ffmpeg is installed: 

    sudo apt install ffmpeg

    sudo yay -S ffmpeg

### Mount the card

The memory card from the dashcam should be mounted:

    sudo mount /dev/mmc0 /mnt/mmc

### Transcode the video

To transcode to a standard H.264 encoded MPEG-4 file, use ffmpeg:

    ffmpeg -i /mnt/mmc/AUKEY/movie/input_file.TS -c:v libx264 -c:a copy ~/Desktop/output_file.mp4

The resulting file will work fine with VLC or any other regular video player. 

### Transcode and trim video

* START is the timestamp in the video to begin the clip, example 1:23
* DURATION is the amount of time to clip, example 0:45

    ffmpeg -ss START -i ./input_file.TS -t DURATION -c:v libx264 -c:a copy ./output_file.mp4
