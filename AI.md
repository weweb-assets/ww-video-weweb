---
name: ww-video-weweb
description: Video component for WeWeb projects enabling video playback with customization options
keywords:
  - video
  - player
  - weweb
  - autoplay
  - video controls
---

#### ww-video-weweb

Embeds video in WeWeb projects with customization options and local file support.

Properties:
- file: string - Video file URL. Default: ""
- previewImage: string - Preview image URL. Default: ""
- videoStartTime: number - Start time in seconds. Default: 0
- autoplay: boolean - Auto-start video. Default: false
- muted: boolean - Mute video. Default: false
- loop: boolean - Loop video. Default: false
- controls: boolean - Show player controls. Default: true
- preload: boolean - Preload video data. Default: true

Children: none

Events:
- play: {value: number} - Triggered when video starts playing. Value is current time
- pause: {value: number} - Triggered when video is paused. Value is current time
- end: {value: number} - Triggered when video ends. Value is current time

Variables:
- Is Playing: boolean - Tracks if video is currently playing
- Current time: number - Current video playback time in seconds
