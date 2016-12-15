# Compilation FFMpeg / NVENC ~~+ NVRESIZE~~ + QSV + VAAPI + VDPAU + OpenCL

> nVidia `nvresize` patch is outdated and not more compatible to the latest version of FFmpeg, so it's not included in this documentation.

> *(even if I've passed a lot of time at trying to make it compile... without any success)*

> Please don't rely on this page: https://developer.nvidia.com/ffmpeg, the implementation is a hack and was never been added to the main FFmpeg tree.

> See:
* https://ffmpeg.org/pipermail/ffmpeg-devel/2015-November/182781.html
* https://ffmpeg.org/pipermail/ffmpeg-devel/2015-November/182784.html
* https://ffmpeg.org/pipermail/ffmpeg-devel/2015-November/182818.html

## Base documentation

* https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu
* https://trac.ffmpeg.org/wiki/HWAccelIntro
* [FFMPEG-with-NVIDIA-Acceleration-on-Ubuntu_UG_v01.pdf](http://developer.download.nvidia.com/compute/redist/ffmpeg/1511-patch/FFMPEG-with-NVIDIA-Acceleration-on-Ubuntu_UG_v01.pdf)

## Required softwares

* Linux Mint 18 / Ubuntu 16.04
* ~~[nVidia Video Codec SDK](https://developer.nvidia.com/nvidia-video-codec-sdk)~~ No more needed, now already included in FFMpeg source code
* ~~[nVidia CUDA SDK](https://developer.nvidia.com/cuda-downloads)~~ No more needed, now already included in FFMpeg source code

## Steps (respect the order)

###### FFmpeg dependencies

```shell
sudo apt-get install -y yasm autoconf automake build-essential libass-dev libfreetype6-dev libsdl2-dev libtool libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev
```

###### FFMpeg extra dependencies

```shell
sudo apt-get install -y ladspa-sdk ladspa-foo-plugins libbluray-dev libbluray-doc libbluray1 libbluray-bin
```

###### ~~NVENC Headers~~ No more needed, now already included in FFMpeg source code

```shell
cd Video_Codec_SDK_7.0.1
sudo cp -v Samples/common/inc/*.h /usr/local/include/
make -j $(nproc)
cd..
```

###### QSV `git clone https://github.com/lu-zero/mfx_dispatch.git`

> Require Intel graphics cards or minimum i915 chipset

```shell
cd mfx_dispatch
autoreconf -fiv
./configure
make -j $(nproc)
sudo make install
sudo ldconfig
cd ..
```

###### OpenCL `sudo apt-get install -y ocl-icd-opencl-dev opencl-headers`

###### VAAPI `sudo apt-get install -y libva-dev vainfo`

###### VDPAU `sudo apt-get install -y vdpau-driver-all mesa-vdpau-drivers vdpau-va-driver libvdpau-dev vdpauinfo`

###### x264 `git clone http://git.videolan.org/git/x264.git`

```shell
cd x264
./configure --disable-cli --enable-static --enable-shared --enable-strip
make -j $(nproc)
sudo make install
sudo ldconfig
cd ..
```

###### x265 `sudo apt-get install -y cmake mercurial`

```shell
hg clone http://hg.videolan.org/x265
cd x265/build
cmake ../source
make -j $(nproc)
sudo make install
sudo ldconfig
cd ..
```

###### fdk_aac `git clone https://github.com/mstorsjo/fdk-aac.git`

```shell
autoreconf -fiv
./configure
make -j $(nproc)
sudo make install
sudo ldconfig
cd ..
```

###### libmp3_lame `sudo apt-get install -y libmp3lame-dev`

###### libopus `sudo apt-get install -y libopus-dev`

###### libvpx `wget http://storage.googleapis.com/downloads.webmproject.org/releases/webm/libvpx-1.5.0.tar.bz2`

```shell
tar xjvf libvpx-1.5.0.tar.bz2
cd libvpx-1.5.0
./configure --disable-examples --disable-unit-tests
make -j $(nproc)
sudo make install
sudo ldconfig
cd ..
```

###### Sox `sudo apt-get install -y sox libsox-dev libsox-fmt-all libsox2 libsoxr-dev libsoxr-lsr0`

###### SSH / SSL `sudo apt-get install -y libssh-dev libssh-dbg libssh2-1-dev libssh2-1-dbg libssl-dev openssl`

###### Xvid / Theora / Vorbis / Wavpack / OpenAL / RTMPDump `sudo apt-get install -y libtheora-dev libwavpack-dev libwavpack1 libxvidcore-dev libxvidcore4 libvorbis-dev libopenal-dev librtmp-dev rtmpdump`

###### ffmpeg `git clone https://github.com/FFmpeg/FFmpeg.git ffmpeg`

```shell
mkdir -v ffmpeg_build
cd ffmpeg_build
../ffmpeg/configure --prefix="./" --disable-shared --extra-cflags=-I/usr/local/include --extra-cflags=-I/usr/local/cuda-8.0/targets/x86_64-linux/include --extra-cflags=-I../nvidia/cudautils --extra-ldflags=-L/usr/local/cuda-8.0/targets/x86_64-linux/lib --extra-ldflags=-L../nvidia/cudautils --enable-nonfree --enable-gpl --enable-version3 --enable-avresample --enable-avisynth --enable-openal --enable-opencl --enable-opengl --enable-x11grab --enable-libnpp --enable-libmfx --enable-nvenc --enable-cuda --enable-vaapi --enable-vdpau --enable-libx264 --enable-libx265 --enable-libxvid --enable-libass --enable-libwavpack --enable-libsoxr --enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libopus --enable-libtheora --enable-libvorbis --enable-libvpx --enable-librtmp --enable-libssh --enable-openssl
make -j $(nproc)
make install
```

###### Test install

```shell
./bin/ffmpeg -version
./bin/ffmpeg -encoders | grep -i 'nvidia'
./bin/ffmpeg -encoders | grep -i 'vaapi'
./bin/ffmpeg -encoders | grep -i 'vdpau'
./bin/ffmpeg -encoders | grep -i 'qsv'
```

## Some CUDA / NVENC Testing
### CPU Based encoding

```shell
time ./bin/ffmpeg -y -i ~/Vidéos/VTS.VOB -t 60 -r 25 -profile:v high -c:v libx264 -fpre ~/Projects/bjrencoder-pro/Presets/libx264-custom.ffpreset -vf "scale=1920:trunc(ow/a/2)*2" -pix_fmt yuv420p -b:v 6250k -maxrate:v 12500k -bufsize 12500k -x264-params threads=0:level=51:aq-mode=1:intra-refresh=0:b-pyramid=0:8x8dct=1 -movflags +faststart -map 0:1 -map 0:2 -metadata:s:a:0 language=eng -c:a ac3 -b:a 384k -ar 48000 -ac 2 ~/Vidéos/VOB_x264_25fps_12500_60s_cpu.mp4
```

	real	4m3.338s
	user	12m12.096s
	sys	0m7.448s

### GPU Based encoding

```shell
time ./bin/ffmpeg -y -i ~/Vidéos/VTS.VOB -t 60 -r 25 -profile:v high -c:v h264_nvenc -fpre ~/Projects/bjrencoder-pro/Presets/libx264-custom.ffpreset -vf "scale=1920:trunc(ow/a/2)*2" -pix_fmt yuv420p -b:v 6250k -maxrate:v 12500k -bufsize 12500k -x264-params threads=0:level=51:aq-mode=1:intra-refresh=0:b-pyramid=0:8x8dct=1 -movflags +faststart -map 0:1 -map 0:2 -metadata:s:a:0 language=eng -c:a ac3 -b:a 384k -ar 48000 -ac 2 ~/Vidéos/VOB_x264_25fps_12500_60s_gpu.mp4
```

	real	0m30.826s
	user	0m20.964s
	sys	0m1.084s

***

### CPU Based encoding (HEVC)

```shell
time ./bin/ffmpeg -y -i ~/Vidéos/BigBuckBunny/bbb_sunflower_native_60fps_normal.mp4 -t 30 -r 25 -c:v libx265 -vf "scale=1920:trunc(ow/a/2)*2" -pix_fmt yuv420p -b:v 6250k -maxrate:v 12500k -bufsize 12500k -x264-params threads=0:level=51:aq-mode=1:intra-refresh=0:b-pyramid=0 -movflags +faststart -map 0:0 -map 0:1 -metadata:s:a:0 language=eng -c:a ac3 -b:a 384k -ar 48000 -ac 2 ~/Vidéos/bbb_sunflower_native_60fps_normal_EN_x265_25fps_12500_cpu.mkv
```

	real	7m31.627s
	user	22m25.816s
	sys	0m11.992s

### GPU Based encoding (HEVC)

>_** Desktop: GPU HEVC works only for graphics cards Geforce GTX 950 series or higher graphics cards (GTX 950, GTX 960, GTX 970, GTX 980, GTX Titan X) **_

>_** Laptop: GTX 965M, 970M, 980M or higher graphics cards **_

```shell
time ./bin/ffmpeg -y -i ~/Vidéos/BigBuckBunny/bbb_sunflower_native_60fps_normal.mp4 -t 30 -r 25 -c:v hevc_nvenc -vf "scale=1920:trunc(ow/a/2)*2" -pix_fmt yuv420p -b:v 6250k -maxrate:v 12500k -bufsize 12500k -x264-params threads=0:level=51:aq-mode=1:intra-refresh=0:b-pyramid=0 -movflags +faststart -map 0:0 -map 0:1 -metadata:s:a:0 language=eng -c:a ac3 -b:a 384k -ar 48000 -ac 2 ~/Vidéos/bbb_sunflower_native_60fps_normal_EN_x265_25fps_12500_gpu.mkv
```

>[hevc_nvenc @ 0x30b7700] No NVENC capable devices found

***

### CPU Utilization

* `top`
* `vmstat -w -n 1`

### GPU Utilization

* `nvidia-smi`
* `nvidia-smi dmon -i 0`

***

### CUDA Tools binaries

```shell
cuda-install-samples-8.0.sh <dir>
cd nvidia/NVIDIA_CUDA-8.0_Samples
make -j $(nproc)
cd bin/x86_64/linux/release/
./deviceQuery
./bandwidthTest
```

Finished :)