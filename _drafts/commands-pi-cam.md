LD_PRELOAD=/usr/lib/uv4l/uv4lext/armv6l/libuv4lext.so uv4l -nopreview --auto-video_nr --driver raspicam --encoding jpeg --quality 40 --metering matrix --drc low --width 640 --height 480 --framerate 30 --server-option '--port=9090' --server-option '--max-queued-connections=10' --server-option '--max-streams=5' --server-option '--max-threads=15'

raspistill -o image.jpg

raspivid -o video.h264

v4l2-ctl -c sharpness=30,compression_quality=100,video_bitrate_mode=1,video_bitrate=25000000

v4l2-ctl --list-ctrls

sudo apt-get install v4l-utils
v4l2-ctl --list-devices
ls -ltr /dev/video*

cvlc --no-audio v4l2:///dev/video0 --v4l2-width 1920 --v4l2-height 1080 --v4l2-chroma MJPG --sout '#standard{access=http{mime=multipart/x-mixed-replace;boundary=--7b3cc56e5f51db803f790dad720ed50a},mux=mpjpeg,dst=:8554/}' -I dummy

ffmpeg('/dev/video0').fps(30).timeout(1).size('640x480').output('~/c.jpg');

raspivid -n -t 1000000 -vf -b 2000000 -fps 25 -o - | gst-launch-1.0 fdsrc ! video/x-h264,framerate=25/1,stream-format=byte-stream ! decodebin ! videorate ! video/x-raw,framerate=10/1 ! videoconvert ! jpegenc ! multifilesink location=img_%04d.jpg

streamer -d -p 10 -o /tmp/pic0000000.jpeg -c /dev/video0 -r 25 -s 1024x768 -t 00:0:05 -j 100

1280x720

streamer -d -p 10 -o /tmp/pic0000000.jpeg -c /dev/video0 -r 25 -s 1024x768 -t 00:00:01 -j 75
average image size 65KB
1 second = 25 images = 1.3MB
60 seconds = 81MB
1 hour = 4.8GB

streamer -d -p 10 -o /tmp/pic0000000.jpeg -c /dev/video0 -r 25 -s 640x480 -t 00:00:01 -j 75
average size 30KB

at -q 25
16KB

at q5 and 1024x768
17KB
1.5GB

at q75 and 1024x768
77KB
6.9GB


for i in {1..100}
do
   kill -USR1 2306
   echo "done"
   sleep 0.1
done


ffmpeg -i /dev/video0 -t 5 ~/code/testing/test.mp4

ffmpeg -i /dev/video0 -t 5 ~/code/testing/test.mp4

ffmpeg -f v4l2 -i /dev/video0 -t 5 ~/code/testing/test_1920.mp4

ffmpeg -f v4l2 -video_size 1920x1080p -i /dev/video0 -t 5 ~/code/testing/test_1920.mp4


v4l2-ctl --set-fmt-video=width=1920,height=1080
v4l2-ctl --set-fmt-video=width=1280,height=720
v4l2-ctl --set-fmt-video=width=640,height=480
v4l2-ctl --all

# get all information of device 0
v4l2-ctl --all -d 0


# load the module
sudo modprobe bcm2835-v4l2

# viewfinder
v4l2-ctl --overlay=1 # enable viewfinder
v4l2-ctl --overlay=0 # disable viewfinder

# record video
v4l2-ctl --set-fmt-video=width=1920,height=1088,pixelformat=4
v4l2-ctl --stream-mmap=3 --stream-count=100 --stream-to=somefile.264

# capture jpeg
v4l2-ctl --set-fmt-video=width=2592,height=1944,pixelformat=3
v4l2-ctl --stream-mmap=3 --stream-count=1 --stream-to=somefile.jpg

# set bitrate
v4l2-ctl --set-ctrl video_bitrate=10000000

# list supported formats
v4l2-ctl --list-formats

# downloads a video of the web
ffmpeg -i http://vjs.zencdn.net/v/oceans.mp4 out2.mp4

# converts a video in to smaller segments which can be played on the browser
ffmpeg -i out2.mp4 -f segment -segment_time 5 -vcodec h264 out4_%d.mp4

ffmpeg -f v4l2 -i /dev/video0 -f alsa -i plughw:1,0 -t 5 ~/code/testing/money.mp4
ffmpeg -f v4l2 -i /dev/video0 -f alsa -i plughw:1,0 -t 5 -vcodec h264 ~/code/testing/money.mp4

ffmpeg -f alsa -i plughw:1,0 -f v4l2  -i /dev/video0 -vcodec h264 -t 5 ~/code/testing/money4.mp4

ffmpeg -f v4l2 -thread_queue_size 5000 -framerate 25 -video_size 640x480 -i /dev/video0 -thread_queue_size 5000 -f alsa -i plughw:1,0 -c:v libx264 -ac 1 -ar 44100 -c:a aac -y -t 5 ~/code/testing/money5.mp4

ffmpeg \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-f alsa -thread_queue_size 5000 -i plughw:1,0 
-vcodec h264 -ac 1 -ar 44100 -acodec aac -y -t 5 ~/code/testing/money5.mp4

ffmpeg -f alsa -ac 1 -i hw:1 -f video4linux2 -i /dev/video0 /tmp/out.mpg

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -q:a  -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-t 5 ~/code/testing/out.mpg

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-t 5 -vcodec h264 -b:a 1K ~/code/testing/out_h264.mp4

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-t 5 -c:v libx264 -crf 22 -ac 1 -c:a aac \
~/code/testing/out_h264.mp4


ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-t 5 -c:v libx264 -crf 60 -ac 1 -c:a aac \
~/code/testing/out_h264_crf_60.mp4

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-t 5 -c:v libx264 -crf 60 -ac 1 -c:a aac -vbr 5 \
~/code/testing/out_h264_crf_60_vbr_5.mp4

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-c:v libx264 -crf 60 -ac 1 -c:a aac -vbr 1 \
-t 15 ~/code/testing/out_h264_crf_60_vbr_1.mp4

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-crf 60 -vbr 5 \
-t 15 ~/code/testing/out_h264_crf_60_vbr_5.mp4

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-crf 23 -vbr 1 -c:v libx264 -c:a aac \
-t 15 -preset veryfast ~/code/testing/out_h264_crf_23_vbr_1.mp4


# For Images

mkdir -p /tmp/mjpeg
cd /tmp/mjpeg
ffmpeg \
-i /dev/video0 \
-vf fps=2 \
/tmp/mjpeg/pic%d.jpg 

rm ~/code/testing/pic_*.jpg

ffmpeg \
-i /dev/video0 \
-vf fps=5 \
-strftime 1 "pic_%Y-%m-%d_%H-%M-%S.jpg"

ffmpeg -i /dev/video0 -vf fps=5 -start_number 100 "pic_%d.jpg"

# WINNER - FOR JPEG IMAGES
ffmpeg -i /dev/video0 -vf fps=5 -start_number 1 "/tmp/mjpeg/pic_%d.jpg"

ffmpeg -f v4l2 -video_size 1920x1080 -framerate 5 -i /dev/video0 -vf fps=5 -start_number 1 "/tmp/mjpeg/pic_%d.jpg"

# numbering is an every incremenrting number starting at 100 and always starts at 100 and will override.

#WINNER - FOR VIDEO

# for audio using driver alsa, specifying audion channel to 1 as our audio is mono
# we specified input to be at card 1 device 0
# and also specified thread queque size as default is too less

# for video using driver v4l2 from device at video0
# thread queue size is higher

# for output options we have bits for audion (-b:a) of 64K which is what we need per channel
# we have video quality at 23 which gives good file size to CPU
# we are converting video (-c:v) to libx264
# we are converting audio (-c:a) to aac


# then we are recording for 15 seconds
# using presetn of veryfast which gives us reasonable filesize and CPU usage

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-b:a 64k -crf 23 -c:v libx264 -c:a aac \
-t 10 -preset veryfast ~/code/testing/out_h264_crf_23_cbr_64k_veryfast_reg_mic.mp4

ffmpeg \
-f alsa -ac 1 -thread_queue_size 5000 -i hw:1,0 \
-f v4l2 -thread_queue_size 5000 -i /dev/video0 \
-b:a 64k -crf 23 -c:v libx264 -c:a aac \
-t 15 -preset ultrafast ~/code/testing/out_h264_crf_23_cbr_64k_ultrafast.mp4

ffmpeg -i input.mp4 -c:v libx264 -crf 22 -preset:v veryfast \
-ac 2 -c:a libfdk_aac -vbr 3 output.mp4

https://trac.ffmpeg.org/wiki/Encode/AAC

arecord -D plughw:1,0 | ffmpeg -f v4l2 -i /dev/video0 -i - -t 5 -vcodec h264 ~/code/testing/money.mp4

arecord -D plughw:1,0 -d 3 test.wav && aplay test.wav

The audio recording device is represented by hw:card_number,device_numer. So to use the second device in the example, use hw:3,0 as the device in gst-launch-1.0 command.
arecord -l
arecord -l (or arecord --list-devices)


ffmpeg -f alsa -i plughw:1,0 -t 30 out.wav

sudo ./configure --arch=arm --target-os=linux --enable-gpl --enable-omx --enable-omx-rpi --enable-nonfree

# compiled ffmpeg by following isntructions here - https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
# left out libaom
# left out h265

cd ~/ffmpeg_sources && \
wget -O ffmpeg-snapshot.tar.bz2 https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 && \
tar xjvf ffmpeg-snapshot.tar.bz2 && \
cd ffmpeg && \
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --extra-libs="-lpthread -lm" \
  --bindir="$HOME/bin" \
  --enable-gpl \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libx264 \
  --enable-nonfree && \
PATH="$HOME/bin:$PATH" make -j4 && \
make -j4 install && \
hash -r

source ~/.profile

# list usb devices
lsusb

arecord -l

aplay -l

scp -r ~/code_mistersingh179/video-playback/src pi@raspberrypi:~/code/video-playback
scp -r ~/code_mistersingh179/video-playback/public pi@raspberrypi:~/code/video-playback
scp -r ~/code_mistersingh179/video-playback/package.json pi@raspberrypi:~/code/video-playback
scp -r ~/code_mistersingh179/video-playback/.env pi@raspberrypi:~/code/video-playback
scp -r ~/code_mistersingh179/video-playback/.env.production pi@raspberrypi:~/code/video-playback
scp -r ~/code_mistersingh179/video-playback/.env.production.local pi@raspberrypi:~/code/video-playback
cd ~/video-playback/
npm install
npm start
mkdir -p /tmp/mjpeg
delete unused files (resync)

scp -r -p pi@raspberrypi:/tmp/mjpeg /Users/sandeeparneja/code_mistersingh179/testing/mjpeg

sudo vim /boot/config.txt
gpu_mem=256
value over 512 broke audio


sudo netstat -ano -p tcp

netstat -vanp tcp | grep 5000

sudo GST_DEBUG=4 AWS_ACCESS_KEY_ID=XXXXXXX AWS_SECRET_ACCESS_KEY=YYYYYY AWS_KVS_AUDIO_DEVICE=hw:1,0 ./kinesis_video_gstreamer_audio_video_sample_app test

AWS_ACCESS_KEY_ID=XXXXXXX AWS_SECRET_ACCESS_KEY=YYYYYY AWS_KVS_AUDIO_DEVICE=hw:1,0 gst-launch-1.0 -v v4l2src ! videoconvert ! video/x-raw,format=I420 ! omxh264enc periodicty-idr=45 inline-header=FALSE ! h264parse ! video/x-h264,stream-format=avc,alignment=au ! queue ! matroskamux name=mux ! fakesink name=sink silent=FALSE alsasrc device=hw:1,0 ! avenc_aac ! aacparse ! audio/mpeg,stream-format=raw ! queue ! mux.


./kinesis_video_gstreamer_sample_app test -w 640 -h 480 -f 15
./kinesis_video_gstreamer_sample_app AmazonRekognitionVideoPlayback -w 640 -h 480 -f 15

add GST_DEBUG=4 for more debugging

gst-device-monitor-1.0

export LD_LIBRARY_PATH=/home/pi/code/amazon-kinesis-video-streams-producer-sdk-cpp/kinesis-video-native-build
export GST_PLUGIN_PATH=/home/pi/code/amazon-kinesis-video-streams-producer-sdk-cpp/kinesis-video-native-build/downloads/local/lib

./gstreamer-plugin-install-script
gst-inspect-1.0 kvssink
./gstreamer-plugin-install-script

gst-launch-1.0 -v v4l2src device=/dev/video0 ! videoconvert ! video/x-raw,format=I420,width=640,height=480,framerate=30/1 ! x264enc  bframes=0 key-int-max=45 bitrate=500 tune=zerolatency ! video/x-h264,stream-format=avc,alignment=au ! kvssink stream-name=test storage-size=128 access-key="XXXXXXX" secret-key="YYYYYY"

gst-launch-1.0 -v v4l2src device=/dev/video0 ! h264parse ! video/x-h264,stream-format=avc,alignment=au ! kvssink stream-name=test storage-size=128 access-key="XXXXXXX" secret-key="YYYYYY"

gst-launch-1.0 -v v4l2src do-timestamp=TRUE device=/dev/video0 ! videoconvert ! video/x-raw,format=I420,width=640,height=480,framerate=30/1 ! omxh264enc periodicty-idr=45 inline-header=FALSE ! h264parse ! video/x-h264,stream-format=avc,alignment=au ! kvssink stream-name=test access-key="XXXXXXX" secret-key="YYYYYY"

gst-launch-1.0 -v v4l2src device=/dev/video0 ! videoconvert ! video/x-raw,width=640,height=480,framerate=30/1,format=I420 ! omxh264enc periodicty-idr=45 inline-header=FALSE ! h264parse ! video/x-h264,stream-format=avc,alignment=au,profile=baseline ! kvssink name=sink stream-name="test" access-key="XXXXXXX" secret-key="YYYYYY" alsasrc device=hw:1,0 ! audioconvert ! avenc_aac ! queue ! sink.




ffmpeg -i /dev/video0 \
-an -c:v h264 -profile:v baseline -preset ultrafast -tune zerolatency -vf "fps=20" -bsf:v h264_mp4toannexb  -f rtp rtp://localhost:8004 

# extract 1 frame every 10 seconds from a video
ffmpeg -i video1.mov -vf fps=1/10 ./frames/thumb%04d.png

# The first -ss seeks fast to (approximately) 8min0sec, and then the second -ss seeks accurately to 9min0sec, and the -t 00:01:00 takes out a 1min0sec clip.
ffmpeg -ss 00:08:00 -i Video.mp4 -ss 00:01:00 -t 00:01:00 -c copy VideoClip.mp4

ffmpeg -i video1.mov -vcodec h264 -acodec mp2 video2.mp4

ffmpeg -i video2.mp4 -c copy -map 0 -segment_time 00:00:01 -f segment -reset_timestamps 1 ./clips/output%03d.mp4

ffmpeg -hide_banner  -err_detect ignore_err -i video2.mp4 -r 24 -codec:v libx264  -vsync 1  -codec:a aac  -ac 2  -ar 48k  -f segment   -preset fast  -segment_format mpegts  -segment_time 0.5 -force_key_frames  "expr: gte(t, n_forced * 0.5)" ./clips/out%d.mkv

# mkv to mp4
ffmpeg -i LostInTranslation.mkv -codec copy LostInTranslation.mp4

for i in *.mkv; do
    ffmpeg -i "$i" -codec copy "${i%.*}.mp4"
done

get devices on network
step 1. know the ip range of the netwwork
`ifconfig` -> inet 192.168.86.20 netmask 0xffffff00 broadcast 192.168.86.255
`sudo nmap -sn 192.168.86.0/24`







