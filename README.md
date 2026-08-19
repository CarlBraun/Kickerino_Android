# <img src="https://i.imgur.com/L5Vvsx6.png" width="28"> Kickerino for Android

An Android chat client for [Kick.com](https://kick.com) with 7TV support — an
Android port of the [Desktop Kickerino](https://github.com/CarlBraun/Kickerino).

Watch the stream and read chat in one screen, with the emotes, badges and
nickname paints you actually expect: Kick emotes, 7TV channel/global/personal
emotes (animated ones included), sub badges, level badges, and 7TV paints.


<p align="center">
  <img src="https://i.imgur.com/tDRuOzU.png" width="49%">
  <img src="https://i.imgur.com/ttxDLn4.png" width="49%">
</p>

---

## Install

Grab the latest APK from [Releases](https://github.com/CarlBraun/Kickerino_Android/releases)
and open it on your phone.

Android will ask you to allow installing from unknown sources — that's normal
for any app not from the Play Store. If Play Protect warns you, tap
**More details → Install anyway**.

**Requirements:** Android 7.0+ (API 24), 64-bit ARM (`arm64-v8a`) — that's
essentially every phone from 2017 onward.

Updates install straight over the top and keep your login, as long as they come
from the same source (all official releases are signed with the same key).

---

## Features

**Chat**
- Live chat over Kick's Pusher WebSocket, with channel history on join
- Kick emotes (channel + sub) and 7TV emotes (channel, global, personal)
- Animated WebP/GIF emotes play natively
- Badges: moderator, VIP, subscriber (by months), gifter, plus **level badges**
- 7TV cosmetics: nickname paints (gradients, images, shadows) and 7TV badges
- Replies, mentions with highlight and vibration, red/green keyword highlighting
- Channel tabs — keep several channels open and switch between them

**Typing**
- `:` — emote autocomplete, as a swipeable strip above the keyboard
- `@` — nickname autocomplete from people currently in chat
- Optional prefix-free mode: typing `car` suggests nicknames *and* emotes together
- Favourite emotes bar
- Emote picker with search

**Moderation** (when you have rights on the channel)
- `/ban`, `/timeout`, `/mute`, `/unban`, `/user` — with `/` autocomplete
- Per-message delete / timeout / ban icons
- Pin and unpin messages
- User card with message logs (moderator logs where available, plus what this
  chat has seen), account age and follow date
- System messages show **who** banned or timed out whom, and for how long

**Player & layout**
- Kick player embedded above the chat
- Portrait: player on top, chat below. Landscape: player left, chat right —
  with system bars hidden for a full-screen view
- Collapse the chat entirely for fullscreen viewing
- Reload button for when a stream stalls

**Extras**
- Optional [PeepeBot](https://peepebot.xyz) integration — a swipe-out moderation
  panel along the edge of the chat
- English and Russian interface
- Timestamps, text/emote size, and other display settings

---

## Building from source

Builds run in Docker, so you don't need an Android SDK/NDK locally — only Docker
Desktop.

```bash
git clone https://github.com/CarlBraun/Kickerino_Android
cd Kickerino_Android
```

**Debug build** (signed with a throwaway key, fine for testing):

```bash
docker run --rm -it \
  -v "${PWD}:/home/user/hostcwd" \
  -v buildozer-global-cache:/home/user/.buildozer \
  -v gradle-cache:/root/.gradle \
  kivy/buildozer:latest android debug
```

**Release build** — needs your own signing keystore. Create one once:

```bash
docker run --rm -it -v "C:/keys:/keys" --entrypoint keytool \
  kivy/buildozer:latest -genkey -v -keystore /keys/kickerino.keystore \
  -alias kickerino -keyalg RSA -keysize 2048 -validity 10000
```

> Back this file up. Lose it and you can never ship an update to anyone who
> already installed the app — Android identifies apps by their signing key.

Then build, passing the passwords through the environment so they never land in
your shell history:

```bash
docker run --rm -it \
  -v "${PWD}:/home/user/hostcwd" \
  -v buildozer-global-cache:/home/user/.buildozer \
  -v gradle-cache:/root/.gradle \
  -v "C:/keys:/keys" \
  -e P4A_RELEASE_KEYSTORE=/keys/kickerino.keystore \
  -e P4A_RELEASE_KEYALIAS=kickerino \
  -e P4A_RELEASE_KEYSTORE_PASSWD -e P4A_RELEASE_KEYALIAS_PASSWD \
  kivy/buildozer:latest android release
```

The APK lands in `bin/`. **If the filename ends in `-unsigned`, the passwords
didn't reach the container** — an unsigned APK won't install, and Android gives
no useful error when you try.

The first build downloads the SDK/NDK and compiles Python, OpenSSL and friends
from source: expect 30–40 minutes. Later builds take a couple of minutes.

### Running on desktop for development

The UI is a web page, so you can develop it without an Android device:

```bash
pip install flask pysher requests
python main.py
# open http://127.0.0.1:5111
```

Narrow the window to phone width for the `stack` layout, widen it for `split`.
Android-only features (login, PeepeBot panel, system bars) degrade gracefully.

```bash
python tests/test_engine.py
python tests/test_server.py
```

---

## How it works

The app is **not Kivy**. Python runs a local Flask server, and the interface is
HTML/CSS/JS rendered by the system WebView (`p4a.bootstrap = webview`).

```
┌─────────────────── Android APK ───────────────────┐
│                                                   │
│  WebView  ──HTTP──►  Flask (127.0.0.1:5111)       │
│  webui/              server.py                    │
│    ▲                    │                         │
│    └────SSE─────────────┘                         │
│                         │                         │
│                      engine/  (no UI at all)      │
│                         ├── chat_worker   Pusher  │
│                         ├── assets        Kick+7TV│
│                         ├── cosmetics_…   7TV     │
│                         ├── render        HTML    │
│                         ├── webview_bridge        │
│                         └── api           Kick API│
└───────────────────────────────────────────────────┘
```

**Why WebView rather than Kivy**

1. **Animated emotes.** 7TV serves animated WebP. Kivy can't display it; WebView
   plays it natively.
2. **Message layout.** On desktop this is ~1200 lines of manual text/image
   layout. In HTML it's ordinary inline flow.
3. **7TV paints.** Hand-drawn with `QPainter` on desktop; here it's
   `background-clip: text` and a CSS gradient.
4. **Rotation and safe areas** come free with media queries and
   `env(safe-area-inset-*)`.

**Getting past Cloudflare.** Kick sits behind Cloudflare, which blocks Python's
HTTP stack outright — the desktop app dodges this with `curl_cffi` TLS
impersonation, which has no Android build. So API calls are proxied through a
second, hidden WebView (`engine/webview_bridge.py`): it's genuine Chrome, so
Cloudflare lets it through, and it shares the app's cookie jar. Requests are
issued as `fetch()` inside it and the results come back through
`evaluateJavascript`.

**Auth.** Login happens on Kick's real page inside the WebView. Kick issues a
bearer token in a (deliberately readable) `session_token` cookie; the app reads
it from the shared `CookieManager` and refreshes it before each request, since
it rotates.

---

## Project layout

```
engine/               logic, zero UI imports
  state.py            CURRENT_STATE and config
  paths.py            cross-platform config/cache paths
  session.py          HTTP transport + Cloudflare bridge routing
  webview_bridge.py   hidden WebView used as an HTTP client
  chat_worker.py      Pusher WebSocket + auto-reconnect
  assets.py           3-phase Kick/7TV asset loading, on-disk cache
  cosmetics_worker.py per-user 7TV paints and badges
  render.py           message -> HTML
  mentions.py         mentions, red/green words
  api.py              channels, sending, moderation, history, user logs
  image_size.py       image dimensions without Pillow

main.py               entry point + all Android-native glue
server.py             Flask: SSE stream, REST, asset serving
webui/                index.html, app.css, app.js
tests/                engine and HTTP-layer tests
buildozer.spec        APK build config
```

---

## Links

- **Telegram:** [@Kickerino](https://t.me/Kickerino)
- **Desktop version:** [CarlBraun/Kickerino](https://github.com/CarlBraun/Kickerino)

If the app is useful to you, you can support development:

- [DeStream](https://destream.net/live/CarlBraun/donate) — card, Apple/Google Pay, crypto
- [DonationAlerts](https://www.donationalerts.com/r/carlbraun) — RU cards
- [Crypto](https://linktr.ee/CarlBraun) — USDT / USDC / BTC / TON

---

Made by CarlBraun

*Not affiliated with Kick.com or 7TV. Uses Kick's private web API, which can
change without notice.*
