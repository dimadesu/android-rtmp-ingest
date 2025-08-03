_Check out the [video](https://www.youtube.com/watch?v=5H0AZca3nk4) with demo and explanation of how to use this guide._

# RTMP Ingest on Android. RTMP H264 to SRT H265 (aka HEVC)

The goal is to send RTMP stream from an action camera (or any RTMP feed really) to the server running on Android phone and use that as a source for a video encoder (running on the same Android phone) to stream it as SRT HEVC.

To achieve this we need a server that can run on Android and a video encoder.

We'll be using [MediaMTX](https://github.com/bluenviron/mediamtx) as a server (running in Termux).

[`IRL Pro`](https://irlpro.app) is a great video encoder app. It can stream SRT HEVC with dynamic bitrate and bonding.

With `IRL Pro` you'll have to be a little creative with pushing through RTMP audio.

Another option is to use `ffmpeg` in Termux. It can do SRT HEVC with no dynamic bitrate and no bonding.

Disclaimer. This was mostly tested on my phone Samsung S20 FE.

## Termux

MediaMTX server will be running in [Termux](https://termux.dev/en/). Install Termux on your phone. Do not use Google Play version.

Start Termux and copy-paste or type the commands below.

To copy commands open this page in the browser of your the phone and tap on commands.

To paste into Termux long tap anywhere in the Termux terminal view to bring up context menu and choose `Paste`.

If at any time you need a new terminal swipe from the left edge of screen and select `New session` button.

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

Start streaming your camera to MediaMTX server.

There are many ways to confirm that it works:
- Camera app would usually say if it was able to connect.
- MediaMTX will log a message in its console.
- You can open HLS url in the phone browser.
- Etc.

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

As of June 2025 `IRL Pro` doesn't have RTMP ingest feature, so this is a workaround.

Big thanks to `Supairyacht` user from `IRL Pro` Discord server for this idea.

We'll be pushing into RTMP ingest of MediaMTX and pulling HLS from MediaMTX. MediaMTX can do this without any extra setup.

Techinically, MediaMTX generates an HTML page with a video player in it, that plays HLS.

We'll create an overlay to display HLS in `IRL Pro`. It can cover the whole view area if you like.

### Audio

There are different ways to make `IRL Pro` use audio from RTMP/HLS feed.

`IRL Pro` can use only phone's "audio input". It can be built-in mic, USB audio device, mic connected via 3.5mm port, Bluetooth mic etc.

The options are:
- If all you need is a mic you can connect it directly to the phone. With this option there will be a delay of about 3 seconds between HLS video and mic, so you'll have to fix that in OBS (that has SRT media source).
- Make HLS audio play via phone's "audio output" and record it via "audio input". Video and audio should be in sync (maybe a tiny bit off, but it's a as good as it gets). I recommend getting a USB audio interface or mixer that can do this (and connect it to the phone via USB). These can be:
  - Rode AI Micro. It is small, light and relatively inexpesive. Connect headphones output to mic input with a cable.
    - Note about devices that use 3.5mm mic inputs. They usually expect very weak mic signal. In order to not overload the mic input set phone audio output to lowest volume first and test. Check audio levels in `IRL Pro`. Raise volume slowly and test until it's nice and loud without distorting.
  - Behringer Xenyx 302USB. I just happen to own one. It's relatively big, but records USB output very easily. I saw similar cheaper devices on Amazon.
  - I think Behringer UCA202 is a great option for this. It is light and cheap. I don't have one to test, but pretty sure it will work. Connect audio output to input with a cable.
  - IK Multimedia iRig Stream audio interface looks pretty cool. Loopback switch should do want we need.

### Setup `IRL Pro`

- Add an overlay in `IRL Pro`. Use HLS feed from MediaMTX as an overlay source. (By default it has audio muted.)
  ```
  http://localhost:8888/mystream
  ```
  If you want HLS audio to play via your phone's audio output use this instead:
  ```
  http://localhost:8888/mystream?muted=no
  ```
- Setup your `IRL Pro` to stream SRT HEVC.
- Start your stream in `IRL Pro`.
- If you connect mic directly to the phone add audio delay of about 3000 ms in OBS. (Note: when you test it in OBS, delay is not applied to audio monitoring, only to recording or a stream.)
- Done!

## `ffmpeg`

Alternative to using `IRL Pro` is to use `ffmpeg` in Termux.

I did a lot of tests with this option and I had a lot issues, so I'm not recommending it. Test details are at the bottom of the page.

Keep MediaMTX running. Open a new Termux terminal and run `ffmpeg` in it.

Install `ffmpeg`.

```
pkg install ffmpeg
```

To encode HEVC with hardware accelation use Mediacodec options. This works (* somewhat works, please see test results at the bottom) on my Samsung S20 FE for example.

Try this first (w/o `-codec_name` option):

```
ffmpeg -i srt://localhost:8890?streamid=read:mystream \
-c:v hevc_mediacodec \
-bitrate_mode 2 \
-b:v 3000K \
-g 250 \
-pix_fmt nv12 \
-c:a copy \
-f mpegts \
srt://YOUR_SRT_URL
```

Different phones can have different codecs, so if that doesn't work try setting `-codec_name` explicitly.
You need to look it up for your phone. Install [`Codec Info` app](https://play.google.com/store/apps/details?id=com.parseus.codecinfo) and use it to find a codec name on your phone that can do hardware accelation.

Here is what I use. `-codec_name` set to `OMX.qcom.video.encoder.hevc`:

```
ffmpeg -i srt://localhost:8890?streamid=read:mystream \
-c:v hevc_mediacodec \
-codec_name OMX.qcom.video.encoder.hevc \
-bitrate_mode 2 \
-b:v 3000K \
-g 250 \
-pix_fmt nv12 \
-c:a copy \
-f mpegts \
srt://YOUR_SRT_URL
```

Tweak other `ffmpeg` options to your liking.

(Optional) Get more details on Mediacodec options:

```
ffmpeg -help encoder=hevc_mediacodec
```

### (Optional) Script that can restart `ffmpeg` if it exits or errors out

For convenience you can create a script and manually run it.

```sh
nano ffmpeg.sh
```

Paste script.

```sh
while true; do
ffmpeg -i srt://localhost:8890?streamid=read:mystream \
-c:v hevc_mediacodec \
-bitrate_mode 2 \
-b:v 3000K \
-g 250 \
-pix_fmt nv12 \
-c:a copy \
-f mpegts \
srt://YOUR_SRT_URL
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

To stop running `CTRL` + `C` a couple of times.

## Test results (March 2025)

### IRL Pro

Tested HLS overlay in `IRL Pro` idea a bit more. Overall, it works fine.
The issue I've noticed is that it's skipping frames randomly here and there for me.
The same frame is rendered twice, so it has these micro stutters.
I had to step through frame by frame in YouTube player to see it. Untrained eyes might not notice it.
I tried to fix it by changing resolutions and bitrate in both `IRL Pro` and ingest stream - no luck.
I suspect that's just how overlays work, so I don't think it can be fixed.

### ffmpeg HEVC

As for hardware acceleration for encoding HEVC when ffmpeg and Medicodec - tested that thoroughly too.
It has issues with playback of stream in OBS and SRT players. It's a little flaky. It sometimes works alright for 1080p, but needs a bunch restarts. Weird.
With `libx265 -crf 23` I can get somewhat stable 480p stream, that at least is reliably playable in OBS.

This sums state of HEVC with ffmpeg in Termux on Android so far for me.

## References

- [Video guide](https://www.youtube.com/watch?v=5H0AZca3nk4) recorded in July 2025
- `IRL Pro` Discord threads
  - [RTMP Ingest](https://discord.com/channels/996502486535901306/1191179335479087104)
  - [Built-in RTMP Server that can be used as a source](https://discord.com/channels/996502486535901306/1056294460121690132)
- This is sort of a continuation of this page that I wrote https://github.com/dimadesu/termux-nginx-rtmp (it's missing a bit with hardware accelarate options though, I'll update it someday).
- I got Mediacodec ideas from https://wiki.x266.mov/docs/encoders_hw/mediacodec
