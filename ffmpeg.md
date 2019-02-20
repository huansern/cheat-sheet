### ffmpeg
#### Extract one frame at 10th minute mark
```bash
ffmpeg -ss 00:10:00.000 -i input.mp4 -frames:v 1 snapshot.jpg
```
#### Extract one frame every minute
```bash
ffmpeg -i input.mp4 -filter:v fps=fps=1/60 snapshot_%04d.jpg
```
Note: this method is slow since ffmpeg parses the entire video to extract the desired frames. Instead, we can instruct ffmpeg to seek for every frame we wanted to extract.

Assuming the video is 30 minutes long
```bash
for t in {0001..0030} ; \
do ffmpeg -accurate_seek -ss `echo $t*60.000 | bc` -i input.mp4 -frames:v 1 snapshot_$t.jpg ; \
done
```

#### Length of video in seconds
```bash
ffprobe -i input.mp4 -v fatal -show_entries format=duration -of csv="p=0"
```
Now that we know the length of the video in seconds, we can remove the hardcoded range.

```bash
INPUT='input.mp4'
INTERVAL=60
LENGTH=`ffprobe -i "$INPUT" -v quiet -show_entries format=duration -of csv="p=0"`
COUNT=`bc <<< $LENGTH/$INTERVAL`

for t in $(seq -f "%04g" 0 $COUNT) ; \
do ffmpeg -hide_banner -loglevel fatal -stats -accurate_seek -ss `echo $t*$INTERVAL | bc` -i "$INPUT" -qscale:v 2 -frames:v 1 snapshot_$t.jpg ; \
done
```

#### Draw text in the center of output frame
```bash
ffmpeg -accurate_seek -ss 10 -i input.mp4 -frames:v 1 -filter:v "drawtext=fontfile='/path/to/font.ttf':fontcolor=black:fontsize=48:x=(w-tw)/2:y=(h-th)/2:text='TEXT'" output.jpg
```
To make the text more readable, we can draw the text on top on a box.
```bash
ffmpeg -accurate_seek -ss 10 -i input.mp4 -frames:v 1 -filter:v "drawtext=fontfile='/path/to/font.tff':fontcolor=white:fontsize=16:x=(w-tw)/2:y=8:box=1:boxborderw=5:boxcolor=black@0.5:text='TEXT'" output.jpg
```

#### Extract frames and print timestamp on output frame
```bash
INPUT='/path/to/input.mp4'
INTERVAL=60
FONT='/path/to/font.ttf'

LENGTH=`ffprobe -i "$INPUT" -v quiet -show_entries format=duration -of csv="p=0"`
FRAMES=`bc <<< $LENGTH/$INTERVAL`

for t in $(seq -f "%04g" 0 $FRAMES) ; \
do SECONDS=`bc <<< $t*$INTERVAL`; \
TEXT=`printf %02d:%02d:%02d $(($SECONDS / 3600)) $(($SECONDS % 3600 / 60)) $(($SECONDS % 60))`; \
ffmpeg -hide_banner -loglevel quiet -stats -accurate_seek -ss "$SECONDS" -i "$INPUT" -qscale:v 2 -frames:v 1 -filter:v "drawtext=fontfile=\'$FONT\':fontcolor=white:fontsize=16:x=8:y=h-th-8:text=\'$TEXT\':box=1:boxborderw=5:boxcolor=black@0.4" output_$i.jpg ; \
done
```

#### Video resolution
```bash
ffprobe -i input.mp4 -v fatal -select_streams v:0 -show_entries stream=width,height -of csv="s=*:p=0"
```
