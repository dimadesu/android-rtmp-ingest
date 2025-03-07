# RTMP Ingest for Android

The goal is to send RTMP stream from action camera to the server running on Android phone.

Then send SRT HEVC out to SRT ingest.

All we need is a server running on Android and some video encoder to encode video and to send it out.

For server with RTMP ingest I tested 2 options:
- [Nginx with RTMP module](https://github.com/dimadesu/termux-nginx-rtmp)
- [MediaMTX](https://github.com/bluenviron/mediamtx)

Here I'll be show how to setup MediaMTX.

For video encoder we'll be using `IRL Pro` app from Google Play. It can do SRT, HEVC, dynamic bitrate and bonding.

Another option is to use `ffmpeg` directly. To encode HEVC with hardware accelation use MediaCodec options.

Example:
```
ffmpeg -i rtmp://localhost:1935/publish/live -c:v hevc_mediacodec -codec_name OMX.qcom.video.encoder.hevc -bitrate_mode 1 -b:v 4000K -g 250 -pix_fmt nv12 -c:a copy -f mpegts srt://au.srt.belabox.net:4000?streamid=YOUR_STREAM_ID
```

## Termux

Server will be running on [Termux](https://termux.dev/en/). Install it on your phone. Do not use Google Play version.

## MediaMTX

Install `wget` to allow downloading MediaMTX

```
pkg install wget
```

We need to grab linux_arm64v8.tar.gz version from one of releases https://github.com/bluenviron/mediamtx/releases.

```
wget https://github.com/bluenviron/mediamtx/releases/download/v1.11.3/mediamtx_v1.11.3_linux_arm64v8.tar.gz
```

Unarchive it.

```
tar -xvzf mediamtx_v1.11.3_linux_arm64v8.tar.gz
```
