# Product: Gillsystems-video

## What This Is

Gillsystems-video is the video production pipeline for Gillsystems content. It covers the full lifecycle: ideation → scripting → asset generation → editing → rendering → YouTube publishing → archival.

This repo is the **control plane** — scripts, configs, workflows, and documentation. Media assets (video, audio, images, 3D) live on the ZFS storage pool and are explicitly gitignored.

## Goals

- [ ] **Goal 1:** [Define — e.g., produce 2 high-quality videos per month]
- [ ] **Goal 2:** [Define — e.g., automate ComfyUI → DaVinci Resolve round-trip]
- [ ] **Goal 3:** [Define — e.g., maintain sub-24hr turnaround from script to publish]

## Scope

### In Scope
- Script writing and editing
- AI-assisted asset generation (ComfyUI, image/video models)
- Video editing and compositing (DaVinci Resolve)
- Audio production and mixing
- YouTube publishing workflow
- Thumbnail and metadata creation
- Pipeline automation and scripting

### Out of Scope
- Live streaming
- 3D animation (beyond what ComfyUI/Blender can do in-pipeline)
- Social media clip extraction (for now)
- Sponsorship/ads management

## Stack

| Component | Tool | Notes |
|-----------|------|-------|
| Editing | DaVinci Resolve | `.drp` files in Git |
| AI Generation | ComfyUI | Workflows in `comfyui_workflows/` |
| Image Gen | FLUX, SDXL, etc. | Via ComfyUI |
| Audio | [TBD] | DAW, stem separation |
| Storage | ZFS (50GB quota) | `tank/gillsystems-video` |
| Publishing | YouTube | Manual upload for now |
| Automation | Python, shell scripts | `scripts/` directory |

## Constraints

- **50GB ZFS hard limit** — all assets must fit; cleanup enforced via YouTube-delete policy
- **No media in Git** — repo stays lightweight and cloneable without LFS
- **7-day YouTube-delete window** — renders deleted 7 days after publish

## Non-Goals

- This is not a general-purpose media library
- This is not a backup system for raw footage
- This is not a collaboration hub for external editors (yet)

---

*Last updated: 2026-07-11*