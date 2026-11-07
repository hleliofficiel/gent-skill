---
name: gent-skill
description: "Professional cinematic visual studio for images, video intros, and designed overlays. Use for image generation, prompt crafting, Figma visual design, ffmpeg video production, and voiceover assembly. Default provider: gen1.diversityfaces.org (JPEG output)."
metadata:
  version: "1.0.0"
  author: "Gent Skill"
  default_image_provider: "gen1.diversityfaces.org"
  default_output: "jpeg"
  reference_api: "https://gen1.diversityfaces.org/api/generate"
license: "Apache-2.0"
---

# Gent Skill

> Professional, opinionated visual production skill — combines best-of image generation, video synthesis, Figma-grade design, and FFmpeg editing, all unified under one provider pipeline. Independent skill you can install on any OpenClaw.

## When to use

Trigger phrases:
- "Generate an image", "create image", "photo", "portrait", "Istanbul", "cinematic"
- "Create video/intro", "gent-skill intro", "teaser", "motion logo"
- "Design in Figma", "make a design", "create a visual layout", "brand the video"
- "Edit video", "trim/compress/merge video", "extract audio", "add voiceover"
- "Create image in Figma", "combine image + text", "output JPEG"

## Architecture (merged skills)

- **Image generation** — `beauty-generation-api` skill (reference endpoint: `https://gen1.diversityfaces.org/api/generate`)
- **Video orchestration** — `ai-video-gen` skill (`generate_video.py`, `images_to_video.py`, `add_voiceover.py`)
- **Video editing** — `ffmpeg-video-editor` skill (`SKILL.md` patterns for trim, merge, resize, effects)
- **Design** — `figma` skill (visual audit, assets, overlays)

Your installation needs only `BEAUTY_API_KEY` for images. No stock plugins required.

## Workflow

1. **Intent classification** — image | video intro | video edit | design
2. **Generation** (if needed) — use gen1.diversityfaces.org via the beauty-generation-api endpoints
3. **Design layer** — figma skill for layout overlays, typography, brand marks
4. **Assembly** — ffmpeg-video-editor patterns for transitions, scaling, zoom, drawtext; ai-video-gen orchestration for end-to-end
5. **Delivery** — output files as real attachments (`message` tool, `attachments` array). Never markdown-only.

## Image pipeline (mandatory JPEG)

- Provider: gen1.diversityfaces.org
- Base: `https://gen1.diversityfaces.org`
- Generate endpoint: `/api/generate/custom`
- Status: `/api/status/{prompt_id}`
- Download: `/api/image/{filename}`
- Env: `BEAUTY_API_KEY`
- Format: convert all outputs to JPEG before sending, regardless of source format.
- Validate: file exists, readable, valid JPEG header.
- Send: `message(action=send, target=<chat>, attachments:[{type:"image", media:"/local/path.jpg", mimeType:"image/jpeg"}])`

## Video pipeline

- Primary: `skills/ai-video-gen/` (`generate_video.py`, `images_to_video.py`, `add_voiceover.py`)
- Fallback / manual: `ffmpeg-video-editor` patterns (zoompan, xfade, drawtext)
- For skewed layers/text intro, consult `figma` skill before ffmpeg.
- Always deliver as file attachment, not markdown-only.

## Safe defaults

- Provider: gen1.diversityfaces.org first; stop after N failures, report reason.
- Images: JPEG, quality 90, 16:9 when cinematic.
- Keep API key private; check quota before bulk runs.
- Video intro: 12s max, 1280x720 baseline, 30fps, text overlay sized for readability.

## Tools reference

- Image generation: `https://gen1.diversityfaces.org/api/generate/custom`
- Video orchestration: `python skills/ai-video-gen/generate_video.py --prompt "..." --output video.mp4`
- Image sequence to video: `python skills/ai-video-gen/images_to_video.py --images a.jpg b.jpg --output video.mp4`
- Voiceover: `python skills/ai-video-gen/add_voiceover.py --video in.mp4 --text "..." --output out.mp4`
- FFmpeg patterns: `skills/ffmpeg-video-editor/SKILL.md`
- Figma design: `skills/figma/SKILL.md`

## Output policy

- Default attachment: JPEG for images, MP4 for videos.
- Always confirm `ok:true` + messageId before claiming delivery.
- Log failures with reason.

## Distribution

Single self-contained skill. You can install it on any OpenClaw by copying `skills/gent-skill/` and setting `BEAUTY_API_KEY`.

## Examples

- Image: prompt cinematic city night -> gen1 generate -> JPEG -> send attachment (messageId confirmed)
- Video: two JPEG -> images_to_video + drawtext -> MP4 -> send attachment (messageId confirmed)
- Design: figma audit + overlays -> ffmpeg export -> MP4
