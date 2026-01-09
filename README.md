# vk-youtube-downloader-bot
VK bot for searching and downloading YouTube videos (yt-dlp based)
# VK YouTube Downloader Bot

A personal VK bot that allows searching and downloading YouTube videos
using **yt-dlp** and uploading them directly to VK messages.

⚠️ This project is intended for **personal and educational use**.

---

## Features

- 🔍 YouTube search via official YouTube Data API
- 🎬 Direct download via YouTube links
- 🚫 Automatic filtering of Shorts (< 60s)
- ⏱ Duration and view count display
- 📤 Uploads videos directly to VK messages
- 🧹 Automatic cleanup of temporary video files
- 🔒 Restricted to a single VK user ID
- ♻️ Safe error handling and recovery

---

## How It Works

1. User sends a command or YouTube link in VK
2. Bot searches YouTube or processes the link
3. Video is downloaded using yt-dlp
4. Video is uploaded to VK as a document
5. Temporary files are removed automatically

The bot runs continuously as a **systemd service**.

---

## Commands

- `поиск <запрос>` — search for YouTube videos  
- Send a number (`1`, `2`, `3`...) to choose a video  
- Send a YouTube link to download directly  
- `отмена` — cancel current operation  
- `помощь` — show help message  

---

## Requirements

- Python 3.9+
- VK group with bot token
- YouTube Data API key

Python libraries:
- `vk_api`
- `yt-dlp`
- `requests`

---

## Setup

1. Clone the repository
2. Install dependencies
3. Edit the configuration section in `bot.py`:

```python
VK_TOKEN = 'your_vk_bot_token'
GROUP_ID = 'your_vk_group_id'
YOUTUBE_API_KEY = 'your_youtube_api_key'
MY_USER_ID = your_vk_user_id

4.Run the bot or configure it as a systemd service

Security Notes

API keys are stored directly in the code

Do NOT publish real tokens in public repositories

If tokens are compromised, revoke and regenerate them immediately

Limitations

Not intended for public or multi-user access

No rate limiting beyond API limits

Depends on YouTube availability and yt-dlp updates

License

MIT License

Author

Developed as a personal automation and learning project.
Implementation includes AI-assisted coding.
