# Cravour — Meta Global AI Developer Hackathon

Public record of the Cravour work built for the Meta Global AI Developer Hackathon (26 Oct 2026 – 4 Nov 2026).

This repository contains **only that contribution**: a unified diff of the files changed on 3 Nov 2026. It does not contain the rest of the Cravour application, its git history, environment files, or deployment workflows.

The patch was scanned before publish. It has no API keys, database URIs, private keys, or account tokens. Provider keys stay in the deployment environment.

## What was built

Logged-in Cravour users can connect WhatsApp, generate image, voice, and video from that linked account, and share an Ad Maker file into the WhatsApp agent and on to Reels.

- **Account link.** Ad Maker mints a short code. The sender texts `link CODE` to the WhatsApp agent. The phone is paired to that Cravour user for 15 minutes per code, then generation uses their plan and their Ad Maker history. The agent never asks for a password.
- **Share link.** An owned Ad Maker asset gets a public card at `https://cravour.com/ads?asset=<id>` and a `wa.me` link. The public payload omits the prompt, user id, and email. The owner is the only person who can mint the share link.
- **Agent.** A Cravour asset link in chat resolves the file. `post it` publishes it to Reels. Posters, video, and jingles reply with the in-chat file plus the Ad Maker URL.
- **ElevenLabs.** WhatsApp voice notes transcribe with Scribe v2 (`scribe_v2`, opus-in-ogg) before any Muse fallback. Text-to-speech stops on auth, quota, and rate-limit errors instead of trying every voice and model. The voice cache key includes model and voice settings. A voice-note sender gets a short spoken reply.
- **Reels playback.** GridFS image and audio ids (`gridfs:…`) play through the public reel media route instead of being treated as a literal URL.
- **Savvy prompts.** The voice prompts describe account linking and the share-to-Reels path.

New server routes:

- `POST /creative/whatsapp/link-code`
- `GET /creative/whatsapp/link-status`
- `POST /creative/whatsapp/share-link`
- `GET /public/asset/:id`

One new environment variable, set on the server only: `ELEVENLABS_STT_MODEL=scribe_v2`.

## Patch

[`patches/meta-hackathon-2026-11-03.patch`](patches/meta-hackathon-2026-11-03.patch)

17 files, +722 / −58. Source commit on the private working branch: `af0b88b17c` (`hack/meta-reels-admaker-whatsapp`), 3 Nov 2026.

| File | Change |
| --- | --- |
| `functions/src/__tests__/whatsappAssetLink.test.ts` | New. Parses shared asset ids from ad and reel links. |
| `functions/src/services/adMakerAssetStore.ts` | Link codes, claim, link status, Ad Maker URLs on saved media. |
| `functions/src/services/whatsappAgentService.ts` | `link CODE`, shared-asset resolution, post to Reels, Scribe-first voice notes. |
| `functions/src/services/unifiedVoiceService.ts` | Scribe transcription, auth-error stop, cache key. |
| `functions/src/services/agentToolHandlers.ts` | Share and media replies include the Ad Maker link. |
| `functions/src/services/whatsappCreativeIntent.ts` | `extractSharedAssetId`. |
| `functions/src/routes/creativeRoutes.ts` | Link code, link status, share link. |
| `functions/src/routes/publicRoutes.ts` | Public asset card. |
| `functions/src/services/aiService.ts` | Voice path uses the updated ElevenLabs behaviour. |
| `functions/src/config/savvyVoicePrompt.ts` | Linking and share instructions. |
| `functions/src/config/costGuard.ts` | Comment aligned with the Muse → Grok → Gemini → OpenAI → ElevenLabs cascade. |
| `src/api.ts` | Client calls for link code, link status, and share link. |
| `src/api-handlers/ai.ts` | Small voice-handler alignment. |
| `src/components/creative/AdsPage.tsx` | Ad Maker WhatsApp link and share actions. |
| `src/components/creative/ContentReelPage.tsx` | GridFS image and audio playback. |
| `src/components/creative/partnerReel.ts` | Reel share helpers for partner clips. |
| `src/config/savvyVoicePrompt.ts` | Same prompt update as the server copy. |

Apply it from a checkout of the Cravour app at the parent of that commit:

```bash
git apply --check patches/meta-hackathon-2026-11-03.patch
git apply patches/meta-hackathon-2026-11-03.patch
```

## Demo path

1. Sign in on Cravour and open Ad Maker.
2. Create a poster, video, or jingle.
3. Mint a WhatsApp link code and send `link CODE` to the Cravour WhatsApp agent.
4. Ask for an image, a voice note, or a video. The file is delivered in chat and saved on the linked account.
5. Open the share link (or paste `https://cravour.com/ads?asset=<id>`) and reply `post it` to publish on Reels.

Product: [cravour.com](https://cravour.com)
