# Jarvis Notes

## 2026-04-28 LINE group media test

Jones asked whether Jarvis could find a recent Taiwanese/Chinese-language YouTube recommendation video about Tokyo ramen, capture a useful frame, and share it to the LINE Japan-trip group.

### Finding selected

- Creator: 雙人徐 / Double Xu
- Video: 【東京必吃】池袋美食特輯！精選5家日本人超愛的餐廳！飯店樓下就是唐吉訶德？米其林拉麵房客限定？台灣吃不到的胡麻生鮪魚飯？｜雙人徐✌️
- Upload date: 2025-08-28
- URL: https://www.youtube.com/watch?v=wsqMKJMg58M&t=159s
- Selected segment: about 2:39, 東京豚骨ラーメン 屯ちん 池袋西口店
- Why it fits: 2025 video, Chinese-language travel/food creator, Tokyo ramen, and directly relevant to the 5/3 Ikebukuro itinerary.

### Technical lesson

- `MEDIA:https://...` worked for remote images such as Azabudai Hills official OG image.
- Directly sending a locally generated image file to LINE was unreliable in this group.
- More reliable workflow: store generated images in this public GitHub repo, then send the raw HTTPS URL.

### Local generation pipeline used

1. Search YouTube candidates with `yt-dlp ytsearch` and metadata inspection.
2. Confirm upload date/title/channel with `yt-dlp --dump-json`.
3. Download a short video section with `yt-dlp --download-sections`.
4. Extract a still frame with `ffmpeg`.
5. Store the frame under this project folder and push to GitHub for a public URL.

## Current asset

- `assets/youtube-frames/doublexu-tonchin-ikebukuro-2025-frame.jpg`
