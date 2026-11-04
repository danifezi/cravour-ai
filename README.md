# Cravour — Meta Global AI Developer Hackathon

Public record of the Cravour work built for the Meta Global AI Developer Hackathon (26 Oct 2026 – 4 Nov 2026).

This repository contains only that contribution: a unified diff. It does not contain the rest of the Cravour application, its history, environment files, or deployment workflows.

The patch was scanned before publish. It has no API keys, database URIs, private keys, or account tokens. Provider keys stay on the server.

This version is what cravour.com is deploying.

## What was built

Logged-in Cravour users connect WhatsApp, generate an image, a voice clip, or a video as that account, and send an Ad Maker file to the WhatsApp agent and on to Reels.

- **Account link.** Ad Maker mints a 6-character code. The sender texts `link CODE` to Savvy. The code expires in 15 minutes. The pairing stays, and later generations use that account's plan and Ad Maker history. Savvy never asks for a password.
- **Share link.** Only the owner can mint a link. It opens `https://cravour.com/ads?asset=<id>` in WhatsApp. The public card omits the prompt, user id, and email.
- **Agent.** A pasted Cravour asset link resolves the file. `post it` within 30 minutes publishes that same file to Reels. A missing file gets a direct reply instead of a blank post. Posters, video, and jingles come back in chat with the Ad Maker URL.
- **ElevenLabs.** WhatsApp voice notes transcribe with Scribe v2 (`scribe_v2`, opus-in-ogg) before any Muse fallback. Text-to-speech stops on auth, quota, and rate-limit errors. The voice cache key includes model and voice settings. A voice-note sender gets a short spoken reply.
- **Reels playback.** GridFS image, audio, and video ids (`gridfs:…`) play through `/public/reel/:id/media` instead of being treated as a literal URL.
- **Savvy prompts.** The voice prompt describes account linking and the share-to-Reels path. The checked-in agent config matches that prompt.

## Routes

- `POST /api/creative/whatsapp/link-code`
- `GET /api/creative/whatsapp/link-status`
- `POST /api/creative/whatsapp/share-link`
- `GET /api/public/asset/:id`

Server-only setting: `ELEVENLABS_STT_MODEL=scribe_v2`.

## Patch

[`patches/meta-hackathon-2026-11-03.patch`](patches/meta-hackathon-2026-11-03.patch)

Apply it from the parent of the shipped commit:

```bash
git apply --check patches/meta-hackathon-2026-11-03.patch
git apply patches/meta-hackathon-2026-11-03.patch
```

## Demo

1. Sign in on [cravour.com/ads](https://cravour.com/ads) and choose **Link WhatsApp**.
2. Make a code and open WhatsApp. The draft message is `link CODE`.
3. Ask Savvy for a poster, a video, or a jingle. The file arrives in chat and in Ad Maker history.
4. From history, choose **Send to Savvy**, then reply `post it`.
5. Open [cravour.com/reel](https://cravour.com/reel). GridFS media plays from the public media route.

Product: [cravour.com](https://cravour.com)
