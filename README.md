# RTMP Ingest on Android. RTMP H264 to SRT H265 (aka HEVC)

The goal is to send RTMP stream from an action camera (or any RTMP feed really) to the server running on Android phone and use that a source for a video encoder (running on the same Android phone) to stream it as SRT HEVC.

To achieve this we need a server that can run on Android and a video encoder.

We'll be using [MediaMTX](https://github.com/bluenviron/mediamtx) as a server (running in Termux).

[`IRL Pro`](https://irlpro.app) is a great video encoder app. It can stream SRT HEVC with dynamic bitrate and bonding. Unfortunately, it cannot use audio from an RTMP stream, but it can use audio from a phone audio input or a Bluetooth mic connected to the phone.

Another option is to use `ffmpeg` in Termux. It can do SRT HEVC with no dynamic bitrate and no bonding. It uses audio from RTMP stream.

## Termux

MediaMTX will be running on [Termux](https://termux.dev/en/). Install Termux on your phone. Do not use Google Play version.

Start Termux and copy-paste or type the commands below.

To copy commands open this page in the browser on the phone and tap on commands.

To paste into Termux long tap anywhere on Termux terminal view to bring up context menu and choose `Paste`.

If at any time you need a new terminal swipe from the left edge of screen and tap `New session` button.

## MediaMTX

Install `wget` to allow downloading MediaMTX.

```
pkg install wget
```

We need to grab `linux_arm64v8` version from one of [MediaMTX releases](https://github.com/bluenviron/mediamtx/releases).
(Your phone can have a different architecture, but try this first.)

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

To stop MediaMTX tap `CTRL` button in Termux and type `c`.

### Test

#### Hotspot

Create a hotspot with your phone. Your camera will need to connect to your phone's hotspot (be on the same network).

#### New Termux terminal session

Keep MediaMTX running. Open new Termux terminal by swiping right from the left edge of the screen and tapping `New session` button.

#### Phone's IP address

Find out your phone's IP address.

```
ifconfig
```

Look for IP under `swlan0` > `inet`.

#### Get ready to stream RTMP to your phone

Configure your camera or video encoder to push to RTMP ingest of your phone.

(Replace `IP_OF_YOUR_PHONE` with an actual IP of your phone.)

```
rtmp://IP_OF_YOUR_PHONE:1935/mystream
```

#### Start the stream

Start streaming your camera to MediaMTX server. If it works it works.

## (Optional) Configure MediaMTX to auto start

There are different ways to do this. Creating a service is a little a bit involved. The lazy way is to add MediaMTX to `.bashrc` file. The server will start on new Termux terminal session and will be killed when you exit terminal.

```
nano .bashrc
```

Add the following line to the content of `.bashrc` file.

```
./mediamtx
```

In Nano:
- `CTRL` + `O`, `Enter` to write file.
- `CTRL` + `X` to quit.

Restart Termux app or create a new terminal session to see it work.

## IRL Pro

### Intro

As of March 2025 `IRL Pro` doesn't have RTMP ingest feature, so this is a workaround.

Big thanks to `Supairyacht` user from `IRL Pro` Discord server for this idea.

We'll be pushing into RTMP ingest of MediaMTX and pulling HLS from MediaMTX. MediaMTX can do this without any extra setup.

HLS is essentially an HTML page with a video in it.

We'll create an overlay to display HLS page in `IRL Pro`. It can cover the whole view area if you like.

### Audio

The issue with this idea is that overlays have no audio. `IRL Pro` lets you use audio via phone's audio input or Bluetooth mic connected to the phone.

There will be a delay of about 3 seconds between HLS video and phone audio, so you'll have to fix that in OBS (that has SRT media source).

### Setup `IRL Pro`

- Add an overlay in `IRL Pro`. Use HLS feed from MediaMTX as an overlay source URL:
  ```
  http://localhost:8888/mystream
  ```
- Setup your `IRL Pro` to stream SRT HEVC.
- Start your stream in `IRL Pro`.
- In OBS add audio delay of about 3000 ms. (Note when you test it in OBS delay is not applied to audio monitoring only to recording or stream.)
- Done!

## `ffmpeg`

Keep MediaMTX running. Open a new Termux terminal and run `ffmpeg` in it.

Install `ffmpeg`.

```
pkg install ffmpeg
```

To encode HEVC with hardware accelation use Mediacodec options. This works on my Samsung S20 FE for example:
```
ffmpeg -i rtmp://localhost:1935/mystream -c:v hevc_mediacodec -codec_name OMX.qcom.video.encoder.hevc -bitrate_mode 1 -b:v 4000K -g 250 -pix_fmt nv12 -c:a copy -f mpegts srt://au.srt.belabox.net:4000?streamid=YOUR_STREAM_ID
```

`-codec_name` setting you need to look up for your phone. Install [`Codec Info` app](https://play.google.com/store/apps/details?id=com.parseus.codecinfo) and use it to find a codec name on your phone that can do hardware accelation.

Tweak other `ffmpeg` options to your liking.

Get more details on Mediacodec options:

```
ffmpeg -help encoder=hevc_mediacodec
```

### Script that can restart `ffmpeg` if it exits or errors out

For convenience you can create a script and manually run it.

```sh
nano ffmpeg.sh
```

Paste script.

```sh
while true; do
ffmpeg -i rtmp://localhost:1935/mystream -c:v hevc_mediacodec -codec_name OMX.qcom.video.encoder.hevc -bitrate_mode 1 -b:v 4000K -g 250 -pix_fmt nv12 -c:a copy -f mpegts srt://au.srt.belabox.net:4000?streamid=YOUR_STREAM_ID
echo "FFmpeg exited. Restarting in 5 seconds."
sleep 5
done
```

In Nano:
- `CTRL` + `O`, `Enter` to write file.
- `CTRL` + `X` to quit.

Give executable permissions.

```sh
chmod +x ffmpeg.sh
```

Run script.

```sh
./ffmpeg.sh
```

## References

- `IRL Pro` Discord threads
  - [RTMP Ingest](https://discord.com/channels/996502486535901306/1191179335479087104)
  - [Built-in RTMP Server that can be used as a source](https://discord.com/channels/996502486535901306/1056294460121690132)
- This is sort of a continuation of this page that I wrote https://github.com/dimadesu/termux-nginx-rtmp (it's missing a bit with hardware accelarate options though, I'll update it someday).
- I got Mediacodec ideas from https://wiki.x266.mov/docs/encoders_hw/mediacodec
