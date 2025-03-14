# `ffmpeg` Android options by `Yuly`

```
ffmpeg \
-i srt://localhost:8890?streamid=read:mystream \
-c:v hevc_mediacodec \
-codec_name OMX.qcom.video.encoder.hevc \
-bitrate_mode 2 \
-b:v 6M \
-pix_fmt nv12 \
-level:v 4.1 \
-g 30 \
-bf 0 \
-qmin 18 -qmax 20 \
-c:a copy \
-f mpegts \
srt://URL
```

References:
- By `Yulyp` from [IRL Pro Discord](https://discord.com/channels/996502486535901306/1056294460121690132/1324756719569731758).
- https://www.youtube.com/@YulyTravels
- https://www.twitch.tv/YulyTravels/
