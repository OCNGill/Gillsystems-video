<p align="center">
  <img src="https://raw.githubusercontent.com/OCNGill/Gillsystems-AMD-Radeon-llama-cpp-Update-AI-Stack/main/Gillsystems_logo_stuff/Gill%20Systems%20Logo.png" alt="Gill Systems Logo" width="800">
</p>

# **Gillsystems-video**

> *Production pipeline for Gillsystems video content — scripts and workflows in Git, media on sovereign ZFS.*

[![Storage](https://img.shields.io/badge/Storage-ZFS%2050GB%20quota-003B57)](#50gb-zfs-storage-limit)
[![Media](https://img.shields.io/badge/Media-not%20in%20Git-orange)](#whats-gitignored-lives-on-zfs)
[![Pipeline](https://img.shields.io/badge/Pipeline-ComfyUI%20%7C%20Resolve-blue)](#repo-structure)
[![YouTube](https://img.shields.io/badge/YouTube-cleanup%20day%207-c4302b?logo=youtube&logoColor=white)](#youtube-delete-policy)
[![PayPal](https://img.shields.io/badge/PayPal-Donate-009cde?logo=paypal&logoColor=white)](https://paypal.me/gillsystems)

Production pipeline for Gillsystems video content — scripts, configs, workflows, and documentation. **This repo does NOT store media assets.** All video, audio, image, and 3D source files are gitignored and live on the ZFS storage pool.

---

## **Table of Contents**

- [Repo Structure](#repo-structure)
- [50GB ZFS Storage Limit](#50gb-zfs-storage-limit)
- [YouTube-Delete Policy](#youtube-delete-policy)
- [Quick Start](#quick-start)
- [Conductor](#conductor)
- [Support / Donate](#support--donate)

---

## **Repo Structure**

```
Gillsystems-video/
├── conductor/               # Pipeline orchestration and state tracking
│   ├── product.md           # What this project is, goals, and scope
│   ├── tracks.md            # Active, next, and future work tracks
│   └── setup_state.json     # Current state of all assets and infrastructure
├── scripts/                 # Automation scripts (Python, shell, batch)
├── comfyui_workflows/       # ComfyUI workflow JSONs for image/video generation
├── docs/                    # Extended documentation, guides, and references
├── config/                  # Configuration files (env templates, tool configs)
├── audio/scripts/           # Audio processing scripts only (audio assets are gitignored)
├── .gitignore               # Media asset exclusion rules
└── README.md                # This file
```

### **What's Tracked in Git**

| Type | Examples | Rationale |
|------|----------|-----------|
| Markdown docs | `*.md` | Documentation, planning, trackers |
| Scripts | `*.py`, `*.bat`, `*.sh` | Automation and pipeline code |
| Configs | `*.json`, `*.yaml`, `*.yml`, `*.toml`, `*.cfg` | Tool and pipeline configuration |
| Workflows | `comfyui_workflows/*.json` | ComfyUI node graphs |
| Resolve projects | `*.drp` | DaVinci Resolve project files (no media) |
| Text | `*.txt` | Notes, logs, manifests |

### **What's Gitignored (Lives on ZFS)**

| Category | Extensions | Location |
|----------|-----------|----------|
| Video | `*.mp4`, `*.mov`, `*.avi`, `*.mkv`, `*.webm`, `*.mxf`, `*.prores`, etc. | `renders/`, `exports/`, `raw_footage/` |
| Audio | `*.wav`, `*.mp3`, `*.flac`, `*.aac`, `*.ogg`, etc. | `audio/` (except `audio/scripts/`) |
| Images | `*.png`, `*.jpg`, `*.webp`, `*.psd`, `*.exr`, `*.tif`, etc. | `renders/`, `exports/` |
| 3D / VFX | `*.blend`, `*.fbx`, `*.glb`, `*.usd`, `*.abc`, `*.vdb`, etc. | Various |
| Archives | `*.zip`, `*.tar.gz`, `*.rar`, `*.7z` | Anywhere |
| Cache / Temp | N/A | `cache/`, `tmp/`, `previews/`, `proxies/` |

---

## **50GB ZFS Storage Limit**

All media assets live on a dedicated ZFS dataset with a **hard 50GB quota**. This is enforced at the filesystem level:

```
zfs set quota=50G tank/gillsystems-video
```

### **Rules**

1. **No media in Git.** If a file is blocked by `.gitignore`, it belongs on ZFS. Do not force-add it.
2. **Check usage before rendering.** Run `zfs get used,quota tank/gillsystems-video` before starting a render job. If usage exceeds 45GB, free space or defer the render.
3. **Cleanup is mandatory.** After a video is published to YouTube, all intermediate renders and raw exports for that project must be removed from ZFS within 7 days (see YouTube-delete policy below).
4. **Compression is on.** The ZFS dataset uses `compression=lz4` to maximize effective capacity. Do not pre-compress assets before storing — let ZFS handle it.
5. **No personal files.** This storage is for production assets only. No backups, personal media, or unrelated data.

### **Monitoring**

```bash
# Check current usage
zfs get used,quota,available tank/gillsystems-video

# List largest directories
du -sh /tank/gillsystems-video/*/ | sort -rh | head -20
```

---

## **YouTube-Delete Policy**

### **Purpose**

Once a video is published to YouTube, the source files serve no further purpose on local storage. YouTube is the canonical distribution platform. Keeping raw renders and project exports after publication wastes the limited 50GB ZFS quota.

### **Policy**

| Stage | Timeline | Action |
|-------|----------|--------|
| **Pre-publish** | Before upload | All assets retained on ZFS. |
| **Published** | Day 0 | Video is live on YouTube. |
| **Verification window** | Days 0–7 | Keep source files for re-upload if needed (errors, corrections, takedowns). |
| **Cleanup deadline** | Day 7 | **Delete all project renders and exports from ZFS.** |
| **Retained** | Indefinite | Keep only: scripts, `.drp` project files, ComfyUI workflows, and documentation — all of which live in Git. |

### **What Gets Deleted (Day 7)**

- All rendered video files (`renders/`, `exports/`)
- Raw footage (`raw_footage/`)
- Intermediate renders and proxies
- Audio exports and stems
- Project-specific cache files
- Exported images and thumbnails

### **What Stays**

- DaVinci Resolve project files (`.drp`) — these are tiny and stored in Git
- ComfyUI workflow JSONs — stored in Git
- All scripts and automation code — stored in Git
- Documentation and planning — stored in Git
- Source assets that are reused across multiple projects (e.g., logo, intro template, brand assets) — these stay on ZFS in a `shared/` directory

### **Exceptions**

If a video is taken down or demonetized and requires re-editing, the cleanup deadline resets to 7 days after the corrected version is published. Mark the track in `conductor/tracks.md` as `status: rework` and note the reason.

### **Enforcement**

1. The `conductor/setup_state.json` file tracks the `published_at` date for each video.
2. A cleanup script (`scripts/cleanup_published.sh`) scans for projects past the 7-day window and reports what's eligible for deletion.
3. Run the cleanup script weekly. Do not let stale renders accumulate.

---

## **Quick Start**

```bash
# Clone the repo (no media — lightweight)
git clone https://github.com/OCNGill/Gillsystems-video.git

# Check ZFS storage status
zfs get used,quota,available tank/gillsystems-video

# See current pipeline state
cat conductor/setup_state.json | python -m json.tool

# Run cleanup audit (dry-run)
bash scripts/cleanup_published.sh --dry-run
```

---

## **Conductor**

The `conductor/` directory is the single source of truth for pipeline state. See the individual files:

- **`product.md`** — Project identity, goals, and scope. Read this first.
- **`tracks.md`** — Kanban-style work tracks: active, next, and future.
- **`setup_state.json`** — Machine-readable state of all assets, infrastructure, and project timelines.

---

## **Support / Donate**

If you find this project useful, supporting Gillsystems keeps the pipeline running:

**Donate:**

- [![PayPal](https://img.shields.io/badge/PayPal-Donate-009cde?logo=paypal&logoColor=white)](https://paypal.me/gillsystems) https://paypal.me/gillsystems

---

*Last updated: 2026-07-11*
