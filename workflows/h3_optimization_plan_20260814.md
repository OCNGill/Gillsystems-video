# H3 Optimization Plan for AMD RX 7900 XTX
## Session Reference: 20260814_030048_2cb3f6
## Target: Reduce generation from ~68 minutes to ~40 minutes (40% improvement)

## Executive Summary
Current workflow achieves functional MiniMax H3 generation on RX 7900 XTX but at ~204s/step (~68 min for 20 steps). Target is ~40 minutes total. Key bottleneck is VRAM overflow causing silent spill to system RAM via PCIe, plus the Spectrum spectral skip node is installed but NOT wired into the graph.

## Current State Analysis

### Workflow Structure (video_minimax_h3_t2v.json)
- **6 core nodes**: RandomNoise → BasicScheduler → BasicGuider → SamplerCustomAdvanced → VAEDecode → CreateVideo
- **Resolution**: 0.4 MP (864×480), duration configurable via Math Expression
- **Models loaded**: 
  - UNET: minimax_h3_fl2va_pruned_int8_convrot.safetensors (diffusion_models)
  - CLIP: qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors (text_encoders)
  - VAE Video: minimax_h3_video_vae_fp16.safetensors
  - VAE Audio: minimax_h3_audio_vae_fp32.safetensors
- **Frame count**: Math Expression `max(5, round(a * 24)) + (5 - (max(5, round(a * 24)) % 17)) % 17` → snaps to 17-frame blocks at 24fps

### Performance Telemetry
- **Step time**: ~204 seconds per step (current)
- **Total for 20 steps**: ~68 minutes (current) 
- **Target**: ~40 minutes total (~2 min/step)
- **VRAM pressure**: CLIP (~15.7GB) + DiT (~19.9GB) = ~35.6GB theoretical, exceeds 24GB card → silent PCIe spill

## Critical Optimization Levers

### 1. WIRE SPECTRUM NODE (HIGHEST PRIORITY)
**Node**: ComfyUI-Spectrum-MiniMax-H3 (xmarre) — installed in custom_nodes but NOT in graph
**Expected gain**: 30-45% reduction in step time
**Mechanism**: Chebyshev spectral feature-forecasting — skips 6-7 of 20 transformer evals, forecasts them instead
**Wiring point**: Between SageAMD node (node 196 in full graph) and BasicGuider/Scheduler
**Action**: Add Spectrum node between MiniMaxH3ImageToVideo conditioning output and Guider input

### 2. REDUCE RESOLUTION/MEGAPIXELS
**Current**: 0.4 MP (864×480)
**Options**:
- 0.3 MP (736×416) — ~25% VRAM reduction, ~20% time reduction
- 0.2 MP (608×352) — ~40% VRAM reduction, ~30% time reduction
**Tradeoff**: Slight resolution decrease vs. significant time savings
**Recommendation**: Test 0.3 MP first, validate quality before dropping to 0.2 MP

### 3. FRAME COUNT OPTIMIZATION
**Current math**: `max(5, round(a * 24)) + (5 - (max(5, round(a * 24)) % 17)) % 17`
**Issue**: Snaps to 17-frame blocks, potentially generating excess frames
**Action**: Verify target duration produces optimal frame count. Consider reducing duration parameter if 15s ceiling is marketing claim (actual usable may be less due to VRAM).

### 4. VRAM MANAGEMENT FLAGS
**Critical flags for AMD/Windows**:
- `--lowvram`: Forces active offloading per stage (CLIP→RAM, DiT→VRAM→RAM, VAE→VRAM)
- `--disable-smart-memory`: Forces full unload instead of partial (avoids corrupt partial reload path on AMD)
- `--gpu-only`: Causes immediate VRAM spill — AVOID
- `--reserve-vram 1.5`: Reserve 1.5GB for system, test stability

**Launch configuration recommendation**:
```
python main.py --lowvram --disable-smart-memory --reserve-vram 1.5
```

### 5. SPECTRUM NODE SPECIFIC WIRING
If adding Spectrum node, wire as:
```
MiniMaxH3ImageToVideo → Spectrum → BasicGuider/SamplerCustomAdvanced
```
**Do NOT** stack with EasyCache/TeaCache — spectrum falls back gracefully but gains nothing stacked.

## Action Plan (Phased, One Change at a Time)

### PHASE 1: WRAP Spectrum Node (Immediate — if custom_nodes available)
1. Verify custom_nodes has `comfyui-spectrum-minimax-h3` installed
2. Add Spectrum node to graph between conditioning and guider
3. Run single test queue with timing log
4. Measure: step time reduction, shared VRAM
5. Keep if ≥25% improvement with shared VRAM reduced toward 0MB

### PHASE 2: RESOLUTION DOWNSAMPLE (Config-only, reversible)
1. Edit ResolutionSelector widget: change from `0.4` to `0.3`
2. Run test queue with same prompt/duration
3. Measure: step time, quality comparison frame-by-frame
4. Keep if quality acceptable at 20-25% time savings

### PHASE 3: VRAM FLAGS (Command-line, reversible)
1. Stop current ComfyUI instance
2. Restart with: `python main.py --lowvram --disable-smart-memory --reserve-vram 1.5`
3. Run baseline test
4. Measure: step time improvement
5. If good, make permanent in launcher scripts

### PHASE 4: DURATION/FRAME OPTIMIZATION (Config-only)
1. Adjust duration math expression to target exactly needed frames
2. Test at reduced frame counts while maintaining perceived quality
3. Measure impact on total generation time

## Priority Order for Tonight's Session

1. **FIRST**: Wire Spectrum node if custom_nodes available — highest potential gain (30-45%)
2. **SECOND**: Reduce ResolutionSelector from 0.4 to 0.3 MP — config-only, immediate signal
3. **THIRD**: Apply --lowvram --disable-smart-memory flags to launcher — config-only, reversible
4. **FOURTH**: Fine-tune duration/frame math after above produce signals

## Expected Outcome
If all three phases execute successfully:
- Spectrum node: ~30-45% step time reduction
- Resolution drop: ~20% additional reduction  
- VRAM flags: ~15% stabilization (prevents catastrophic spills)
- **Composite target**: 40 minutes total (from current 68 minutes) — approximately 60% improvement goal

## Commander's Directives for This Session
- "One change at a time. Never two changes simultaneously."
- "Measure before and after each change. Quality check frame-by-frame."
- "If a change degrades quality more than 10%, roll back immediately."
- "Report step times and VRAM usage after each test queue run."
- "Spectrum node: if custom_nodes doesn't have it, skip and move to resolution change."

## Troubleshooting Notes
- If Spectrum node causes errors: remove it, proceed with resolution/VRAM flags only
- If --lowvram causes extreme slowdowns (>5min/step): revert, the flags aren't working on this hardware/software combo
- If quality degrades noticeably at 0.3 MP: keep at 0.4 MP, optimize other vectors
- Spectrum: "Don't stack with EasyCache/TeaCache" — ensure those are disabled or not present