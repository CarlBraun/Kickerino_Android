# Privacy Policy — Kickerino

**Last updated: 17 August 2026**

Kickerino is an unofficial Android client for Kick.com. It is not
affiliated with, endorsed by or sponsored by Kick.com or 7TV.

## The short version

Kickerino has no servers. There is no account to create, nothing is
uploaded anywhere, and the developer cannot see what you do in the app.
Everything the app remembers stays on your phone. The app talks directly
to Kick and to a few other services, exactly as your browser would.

## What is stored on your device

All of it lives in the app's own private storage and is removed when you
uninstall the app.

- **Your Kick login.** When you sign in, the session cookie Kick gives
  your browser is kept so you don't have to sign in again. It is stored
  only on your phone and is sent only to Kick. Logging out deletes it.
- **Your settings** — text size, language, which features are switched
  on, the channels you have opened, your favourite emotes, red words, and
  the people you have blocked.
- **A cache of images** — emotes, badges and avatars, so they don't have
  to be downloaded again.

The app does not read your contacts, location, photos, files, microphone
or camera, and asks for no permissions beyond network access.

## What is sent, and to whom

Kickerino connects directly to these services on your behalf. It does not
route anything through the developer.

| Service | Why | What it receives |
|---|---|---|
| `kick.com`, `files.kick.com`, `player.kick.com` | chat, login, moderation, video | your Kick session, the messages you send, standard request data (IP address, device type) |
| `7tv.io` | third-party emotes, badges and name colours | the Kick user IDs whose emotes are being shown |
| `wsrv.nl` | resizing avatar images | the address of the image being resized |
| `raw.githubusercontent.com` | the list of supporter nicknames | nothing about you |
| `api.github.com`, `github.com` | checking for app updates (not in the Google Play version) | nothing about you |
| `peepebot.xyz` | the optional moderation panel, if you open it | whatever that site normally receives, as in a browser |

Opening a link from chat hands it to your normal browser, after which
this policy no longer applies.

Each of these services has its own privacy policy and its own handling of
your data. Kickerino has no control over them.

## Analytics

There are none. No analytics, no crash reporting, no advertising, no
tracking of any kind. The developer receives no information about you or
your usage at all.

## Purchases

The Google Play version offers an optional in-app purchase that unlocks
extra features. It is handled entirely by Google Play Billing. Kickerino
never sees your payment details. Google's own privacy policy covers that
transaction.

The supporter badge is matched by Kick nickname. If you ask for your
nickname to be added to the public supporters list, that nickname —
already public on Kick — becomes visible in a file in the app's public
source repository. This only happens if you ask for it.

## Children

Kickerino is not intended for children. It shows live chat from Kick.com,
which is not moderated by this app and may contain adult language.

## Your choices

- **Log out** in Settings → Account to delete the stored Kick session.
- **Uninstall** the app to remove everything else it has stored.
- There is nothing for the developer to delete on request, because
  nothing about you is ever collected.

## Changes

If this policy changes, the new version will be published at this address
and the date above will be updated.

## Contact

Questions about this policy: **carlbraun322@gmail.com**

Source code: <https://github.com/CarlBraun/Kickerino_Android>
