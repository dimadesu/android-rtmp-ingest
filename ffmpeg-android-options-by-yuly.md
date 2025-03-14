ffmpeg Android options
 
"-listen", "1",
"-hwaccel", "mediacodec",
"-i", "rtmp://0.0.0.0:1935",
"-listen_timeout", "1000",
"-c:v", "hevc_mediacodec",
//"-b:v 8M -maxrate 10M -bufsize 4M",  //can theoratically send out 8 to 10mbps h265 - which I think is overkill for just one mobile data streaming source
"-b:v 6M -maxrate 8M -bufsize 2M",
"-pix_fmt yuv420p",             // Essential for MediaCodec
"-level:v 4.1",               // Level for HD resolutions (adjust if needed)
"-g 30",                     // GOP size (adjust as needed)
"-bf 0",                      // No B-frames for minimal latency
"-qmin 18 -qmax 20",         // Tight QP range for very high quality
"-c:a", "copy",
"-f", "mpegts",
MysrtURL

References:
- By `Yulyp` from [IRL Pro Discord](https://discord.com/channels/996502486535901306/1056294460121690132/1324756719569731758).
- https://www.youtube.com/@YulyTravels
- https://www.twitch.tv/YulyTravels/
