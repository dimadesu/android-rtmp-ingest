# `ffmpeg` Android options by `Yuly`

```
ffmpeg \
-listen 1 \
-hwaccel mediacodec \
-i rtmp://0.0.0.0:1935 \
-listen_timeout 1000 \
-c:v hevc_mediacodec \
# can theoratically send out 8 to 10mbps h265 - which I think is overkill for just one mobile data streaming source
# -b:v 8M -maxrate 10M -bufsize 4M \
-b:v 6M -maxrate 8M -bufsize 2M \
# Essential for MediaCodec \
-pix_fmt yuv420p \
# Level for HD resolutions (adjust if needed)
-level:v 4.1 \
# GOP size (adjust as needed)
-g 30 \
# No B-frames for minimal latency
-bf 0 \
# Tight QP range for very high quality
-qmin 18 -qmax 20 \
-c:a copy \
-f mpegts \
srt://URL
```

References:
- By `Yulyp` from [IRL Pro Discord](https://discord.com/channels/996502486535901306/1056294460121690132/1324756719569731758).
- https://www.youtube.com/@YulyTravels
- https://www.twitch.tv/YulyTravels/
