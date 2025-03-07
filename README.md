# RTMP Ingest for Android. RTMP H264 to SRT H265 (aka HEVC)

The goal is to send RTMP stream from an action camera (any RTMP feed really) to the server running on Android phone and then send SRT HEVC out.

To achieve that we need a server that can run on Android and a video encoder.

We'll be using [MediaMTX](https://github.com/bluenviron/mediamtx) as a server.

For the video encoder we'll be using [`IRL Pro` app from Google Play](https://play.google.com/store/apps/details?id=app.irlpro.android). It can do SRT, HEVC, dynamic bitrate and bonding.

Another option is to use `ffmpeg` in command line. It will be able to do SRT HEVC with no dynamic bitrate and no bonding.

## Termux

Server will be running on [Termux](https://termux.dev/en/). Install it on your phone. Do not use Google Play version.

Start Termux and run commands below.

## MediaMTX

Install `wget` to allow downloading MediaMTX.

```
pkg install wget
```

We need to grab `linux_arm64v8` version from one of releases https://github.com/bluenviron/mediamtx/releases.

```
wget https://github.com/bluenviron/mediamtx/releases/download/v1.11.3/mediamtx_v1.11.3_linux_arm64v8.tar.gz
```

Unarchive it.

```
tar -xvzf mediamtx_v1.11.3_linux_arm64v8.tar.gz
```

Run MediaMTX.

```
./mediamtx
```

### (Optional) MediaMTX conflicts with Nginx

I am switching back and forth between MediaMTX and Nginx. If MediaMTX is failing to start that usually means port `1935` is taken by Nginx, so stop Nginx:

```
sv down nginx
```

Check if that Nginx is down.

```
sv status nginx
```

### Test

Create hotspot with your phone.

#### Find out your phone's IP address

```
ifconfig
```

Look for IP under `swlan0` > `inet`.

#### Configure your camera / video encoder to push to RTMP ingest

```
rtmp://IP_OF_YOUR_PHONE:1935/mystream
```

## TODO: The following section needs to be re-written for Termux. Skip it for now
## (Optional) Configure MediaMTX to auto start as a service

Please refer to documention in MediaMTX README https://github.com/bluenviron/mediamtx?tab=readme-ov-file#start-on-boot

`systemd` is in charge of managing services and starting them on boot.

Copy:

```
cp mediamtx /usr/local/bin/
cp mediamtx.yml /usr/local/etc/
```

Create a systemd service:

```
sudo tee /etc/systemd/system/mediamtx.service >/dev/null << EOF
[Unit]
Wants=network.target
[Service]
ExecStart=/usr/local/bin/mediamtx /usr/local/etc/mediamtx.yml
[Install]
WantedBy=multi-user.target
EOF
```

Enable and start the service:

```
systemctl daemon-reload
systemctl enable mediamtx
systemctl start mediamtx
```

## IRL Pro

### Intro

As of March 2025 `IRL Pro` doesn't have RTMP ingest feature, so this is a workaround.

Big thanks goes to `Supairyacht` user from `IRL Pro`'s Discord server for this idea.

We'll be pushing into RTMP ingest of MediaMTX and pulling HLS from MediaMTX. It can do this by default. No setup needed.

HLS is essentially an HTML page with a video in it.

We'll create an overlay to display HLS page in `IRL Pro`. It can cover the whole phone view area if you like.

### Audio

The only major issue with this idea is that overlays have no audio. You can get audio via phone's audio input or Bluetooth mic connected to the phone.

There will be a delay of about 3 seconds between HLS video and audio, so you'll have to fix that in OBS (that has SRT media source).

### Setup

- Add an overlay in `IRL Pro`. Use HLS feed from MediaMTX as an overlay URL source:
  ```
  http://localhost:8888/mystream
  ```
- Set you `IRL Pro` to send SRT HEVC.
- Go live in `IRL Pro`.
- In OBS add audio delay of about 3000 ms. Note that in OBS delay is not applied to audio monitoring only to recording or a stream.

## ffmpeg

I usually use `ffmpeg` with Nginx that is running as a service. I think you can run another Termux tab if you want to make it work with MediaMTX.

Please refer to the Nginx and ffmpeg instructions in another repo for details https://github.com/dimadesu/termux-nginx-rtmp (it's missing a bit with hardware accelarate options though).

Install

```
pkg install ffmpeg
```

To encode HEVC with hardware accelation use MediaCodec options. This works on my Samsung S20 FE for example:
```
ffmpeg -i rtmp://localhost:1935/mystream -c:v hevc_mediacodec -codec_name OMX.qcom.video.encoder.hevc -bitrate_mode 1 -b:v 4000K -g 250 -pix_fmt nv12 -c:a copy -f mpegts srt://au.srt.belabox.net:4000?streamid=YOUR_STREAM_ID
```

Note: `-codec_name` you'll need to look up for your phone. Use [`Codec Info` app](https://play.google.com/store/apps/details?id=com.parseus.codecinfo) from Play Store and pick a codec that can do hardware accelation.

## References / Links

IRL Pro Discord threads
- [RTMP Ingest](https://discord.com/channels/996502486535901306/1191179335479087104)
- [Built-in RTMP Server that can be used as a source](https://discord.com/channels/996502486535901306/1056294460121690132)
- This is sort of a continuation of this repo: https://github.com/dimadesu/termux-nginx-rtmp


Tweak other ffmpeg options to your liking.
