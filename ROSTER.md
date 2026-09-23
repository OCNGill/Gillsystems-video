# Gillsystems Video Production — Agent Roster

*Sovereign video pipeline agents.*

## Active Agents

| Agent Name | Role | Status |
|------------|------|--------|
| Commander Hermes | Primary interface, fleet orchestration | Active |
| Liara | Video model research (Seedance, MiniMax, Wan2.2, LTX) | Active |
| Coder | ComfyUI workflows, Python automation | Active |
| Geordi | Architecture (Main 7900 XTX pipeline) | Active |
| Scribe | 7D docs (CHANGELOG + conductor) | Active |
| Sentinel | Zero-trust (no cloud without auth) | Active |
| EDI | Comms backbone for asset sync | Active |
| Courier | YouTube upload notifications | Active |

## Infrastructure

| Component | Location | Specs |
|-----------|----------|-------|
| ComfyUI (Primary) | Main (10.0.0.164) | Radeon RX 7900 XTX 24GB, ComfyUI portable |
| ZFS Video Assets | HTPC (10.0.0.42) | gillsystems_zfs_pool/video_assets (50GB quota) |
| Seedance 2.5 | OpenRouter (cloud) | bytedance/seedance — requires Commander auth |
| MiniMax H3 | Main (local) | ComfyUI custom nodes |

## Folder Structure
```
Gillsystems-video/
├── images/
├── intro_video/
├── talking_head_video/
├── video/
├── conductor/
│   ├── tracks.md
│   ├── product.md
│   └── setup_state.json
├── scripts/
└── CHANGELOG.md
```

---

## Cross-Reference
- **Master ROSTER**: `../Agents/ROSTER.md`
- **MASTER_CHARTER**: `../Gillsystems-Commanders-Hermes/MASTER_CHARTER.md`
- **Fleet inference**: `../Gillsystems-Commanders-Hermes/conductor/tracks.md` (Track 8, 11)
