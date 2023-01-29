---
title: "One Second Diary v1.5 was released"
date: 2023-01-28
draft: false
categories: ['News']
---

## Creating the movie of your life

My open source, [minimalist video diary app](https://play.google.com/store/apps/details?id=com.kylekun.one_second_diary) just got a huge update. See the [changelog](https://github.com/KyleKun/one_second_diary/blob/main/CHANGELOG.md) for more info.

I would like to share some technical details about this update:

### 1. FFmpeg is the real deal

The videos recorded with the previous version of the app had a different configuration than the ones recorded with the new version. Also, the new feature to add videos from the gallery could bring any kind of video, so I needed to make sure that all videos would have the same configuration. That's because I wanted to be able to concatenate them with the `concat` method, which is by far the fastest way to do it without losing quality. All this was only possible with the help of [ffmpeg-kit](https://github.com/arthenica/ffmpeg-kit).

Here are some related commands I used and might be useful if you are trying to do something similar:

```bash
# Add metadata to a video
ffmpeg -i input.mp4 -metadata artist="$artist" -metadata album="$album" -metadata comment="$comment" -c copy output.mp4

# Scale a video 1920x1080 and automatically add black bars to keep the aspect ratio
ffmpeg -i input.mp4 -vf scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black -c:a copy -c:s copy output.mp4

# Trim a video
ffmpeg -i input.mp4 -ss 00:00:00 -to 00:00:10 -c copy output.mp4

# Add subtitles to a video as default subtitles using a .srt file
ffmpeg -i input.mp4 -c:v copy -c:a copy -map 0:v -map 0:a? -map 1 -disposition:s:0 default output.mp4

# Change framerate to 30, audio codec to aac and video codec to h264
ffmpeg -i input.mp4 -r 30 -c:a aac -c:v h264 output.mp4

# Change audio stream to mono, 44100Hz, 256kbit/s
ffmpeg -i input.mp4 -ac 1 -ar 44100 -b:a 256k output.mp4

# Add an empty audio stream to a video that doesn't have one, matching the video's duration
ffmpeg -i input.mp4 -f lavfi -i anullsrc=channel_layout=mono:sample_rate=44100 -shortest -c:a aac -c:v copy output.mp4

# Concatenate multiple videos into one using a .txt file
ffmpeg -f concat -safe 0 -i input.txt -c copy output.mp4
```

Notes:
- `c:a` and `c:v` are the same as `codec:a` and `codec:v`.
- `c:{a/v/s} copy` means that the codec will be copied from the input file to the output file.
- The `?` in `-map 0:a?` means that the command will ignore errors if the input video doesn't have an audio track.
- Add `-y` to the end of the command to overwrite the output file if it already exists.
- If you want to concatenate videos with different frame rates, you can use `-vsync vfr`.

How did I learn all this? That brings us to the next point.

### 2. ChatGPT and GitHub Copilot

While FFmpeg is amazing, its documentation is kinda... disappointing. Even StackOverflow was not so helpful. So, I turned to [ChatGPT](https://chat.openai.com/chat) and it magically brought most of those commands that achieved what I wanted. After copying them, [GitHub Copilot](https://github.com/features/copilot) would complete another good portion of the work, like error handling and UI changes. This duo is a very powerful combination and if you haven't yet, I highly recommend you to try it out. I don't think I can live without it anymore lol.

### 3. Fun Fact: Google rejected my first attempt to publish the update

In my first attempt to publish the update, Google rejected it because I was requesting the `MANAGE_EXTERNAL_STORAGE` permission, which is now a very restricted permission that few apps can make use of. This allowed me to save videos in the root path of the device and I didn't have to care about the permission errors some devices were reportedly having in the previous version even when storage permission was granted. I was able to fix it by using the `MediaStore API` instead, thanks to the [media_store_plus](https://github.com/SNNafi/media_store_plus) package. I had to change a lot of code to make it work, but at the end of the day, it was a good thing since I was able to implement more features along with the requested storage migration 😅.

### 4. A huge thanks to everyone who supported me

I wouldn't have been able to finish this update - at least not so quickly and stable like it was - if I hadn't received that much help from the community and friends. So, thank you to everyone who supported me in any way: with code, testing, suggestions and donations. I am very, very grateful. This update is dedicated to you.

## What's next?

In the short term, bug fixes and minor improvements based on the feedback I get from users.

In the long term, I am planning to give the app a brand new UI and redo its architecure (currently, it's a complete mess to be honest, but everything works 🤣). There's no ETA since I am making a game right now, but this project is meaningful to me and I am definitely willing to work hard on it again in the future.
