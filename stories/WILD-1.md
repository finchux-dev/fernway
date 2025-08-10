# Apple Vision Pro Immersive Player — v1 Offline Bundled Samples (WILD-1)

> **Progress:**
> Draft approved. Plan saved for v1 offline (bundled MV‑HEVC samples). Pending execution by development team.

## Overview

Build a minimal, high‑quality immersive video app for Apple Vision Pro (visionOS 2.0+) that ships fully offline with up to five bundled Apple Immersive Video (MV‑HEVC) samples. The player runs in an ImmersiveSpace with minimal controls and head‑tracked audio when present. No streaming or downloads in v1. v2.0 will add downloads and may add streaming.

## Requirements

- **Access:** Guest‑only. No accounts or paywalls in v1.
- **UI:**
  - Simple catalog grid of 5 bundled samples with poster, duration, resolution/fps badges.
  - Detail screen with Play and basic info.
  - Player controls: play/pause, timeline scrub/seek, recenter, environment selector (default black void).
- **Data Model:**
  - Local JSON manifest embedded in app bundle describing titles and variants (MV‑HEVC `.mov`), file paths, resolution, fps, audio type.
  - No reliance on embedded metadata for display; MV‑HEVC playback requires properly authored assets.
- **Navigation:**
  - Launch → Catalog → Detail → Play (ImmersiveSpace) → Exit → Detail/Catalog.
- **Testing:**
  - Unit tests (manifest parsing), integration tests (player init), on‑device performance tests with 8K assets.
- **Other:**
  - Platform: visionOS minimum 2.0, target latest Xcode SDK.
  - Content: Up to 5 videos, ≤1 GB each, total app size ≈5 GB.
  - Audio: Use best available; enable head‑tracked spatial audio if present, fallback to stereo.
  - Telemetry: Crash reporting and anonymous usage analytics with a privacy toggle.
  - No networking, no On‑Demand Resources in v1 (fully offline operation).

## User Journey

- **How does a user get there?**
  - Install and open the app to see a catalog of five immersive videos with posters and basic info.
- **What do they do when they are there?**
  - Select a title, review details, tap Play to enter the immersive player.
- **What happens if it goes right?**
  - Smooth playback in black void environment; correct stereoscopic rendering; head‑tracked audio when available; user can seek and recenter.
- **What happens if it goes wrong?**
  - Graceful error if an asset fails to load; suggestion to try another sample; optional crash report prompt.
- **Where does it take them when they are done?**
  - Exiting the player returns to the detail screen or catalog; session state is preserved for continue‑watching within the current app session.

## Implementation Phases

### Phase 0: Project Setup [ ]
- [ ] Create visionOS app (SwiftUI + RealityKit + ImmersiveVideo/AVFoundation), minOS 2.0.
- [ ] Define modules: `Core`, `Catalog`, `Player`, `Telemetry`.
- [ ] Configure Window + ImmersiveSpace scenes, app icons, and entitlements.
- [ ] Add telemetry (crash + anonymous analytics) behind a toggle in Settings.

### Phase 1: Local Manifest & Catalog [ ]
- [ ] Define manifest schema for bundled assets:
  - id, title, description, poster, duration, fps, resolution, color (SDR/HLG/HDR10), audioType, filePath, container=mov, codec=mv-hevc.
- [ ] Implement `ManifestService` to load/validate manifest from bundle.
- [ ] Build `CatalogView` grid and `DetailView` with metadata badges and Play CTA.

### Phase 2: Asset Packaging (Bundled Samples) [ ]
- [ ] Add up to 5 MV‑HEVC `.mov` samples to the app bundle; validate size constraints (≤1 GB/video).
- [ ] Include poster images and thumbnails; reference via manifest file paths.
- [ ] Build a lightweight content validation tool (during debug) to verify playable tracks on device.

### Phase 3: Immersive Player (MV‑HEVC) [ ]
- [ ] Implement `MVHEVCPlayer` using Immersive Video APIs for `.mov` playback.
- [ ] Player shell (`ImmersivePlayerView`): play/pause, scrub/seek, recenter, environment selector (default black void).
- [ ] Audio path: auto‑detect head‑tracked spatial audio; fallback to stereo; volume/mute.
- [ ] Handle error states: unsupported/invalid asset, decode failure, user‑friendly messaging.

### Phase 4: Performance, QA, and Submission Prep [ ]
- [ ] On‑device performance tests with 8K 180 content; frame pacing and thermals.
- [ ] Accessibility: VoiceOver labels, large comfortable hit targets.
- [ ] App review readiness: privacy labels, asset rights confirmation, TestFlight build.

### Phase 5: v2.0 Preview (Downloads + Streaming Design, No Implementation in v1) [ ]
- [ ] Outline download manager architecture (foreground/background, persistence).
- [ ] Choose CDN pipeline (Backblaze B2 + Cloudflare) and HLS/CMAF for streaming; define ABR ladder guidelines.
- [ ] Define manifest extensions for remote URLs, checksums, and optional DRM in future.

## Key Files Modified

- `App/App.swift`: Scene setup (Window + ImmersiveSpace).
- `Core/Models/VideoAsset.swift`: Title and variant models (v1: MV‑HEVC).
- `Core/Services/ManifestService.swift`: Load/validate local manifest.
- `Catalog/Views/CatalogView.swift`: Catalog grid UI.
- `Catalog/Views/DetailView.swift`: Detail page with Play.
- `Player/Views/ImmersivePlayerView.swift`: Player shell and controls.
- `Player/Playback/MVHEVCPlayer.swift`: Immersive Video playback pipeline.
- `Telemetry/TelemetryService.swift`: Crash + analytics integration.
- `Resources/manifest.local.json`: Embedded manifest for bundled assets.

## Current Status

- v1 scope finalized: fully offline, 5 bundled MV‑HEVC samples, minimal player, telemetry enabled.
- Plan approved; ready for implementation by development team.

## Recent Progress

- Re‑scoped to remove downloads/streaming from v1.
- Set bundle size and per‑video limits.
- Chosen MV‑HEVC only for v1 samples; spatial audio when available.
- Confirmed local manifest and privacy‑gated telemetry.

## Next Steps for Next Agent

> ### Handoff Note for Next Agent
> Prioritize asset packaging and on‑device validation early to derisk size/performance.

- [ ] Initialize project, scenes, and module structure.
- [ ] Implement manifest schema/loader and catalog/detail UI.
- [ ] Integrate MV‑HEVC playback and minimal controls.
- [ ] Package 5 sample assets; verify playability on device.
- [ ] Prepare TestFlight build.

## Technical Details & Decisions

- MV‑HEVC playback: Uses Immersive Video APIs; assets must be correctly authored (MV track/sample attachments). If an asset lacks required metadata, it may not initialize; manifest drives UI info but cannot “create” MV metadata.
- Projection scope v1: MV‑HEVC only. Equirect 180 SBS/TB support deferred.
- Audio: Prefer head‑tracked spatial audio (e.g., Atmos/ambisonic) when present; auto‑fallback to stereo.
- Asset packaging: No On‑Demand Resources (keeps app fully offline). Be mindful of overall app size and store limits.
- Privacy: Telemetry opt‑in toggle in Settings; no PII collection.

## Migration/Data Mapping (if applicable)

| Old Field/Model | New Field/Model | Notes |
|-----------------|-----------------|-------|
| —               | `VideoAsset`    | Bundled title with MV‑HEVC file path. |
| —               | `PlaybackConfig`| Environment default, recenter settings. |

## Success Criteria

1. App launches offline with a catalog of five bundled MV‑HEVC samples and posters.
2. Each sample plays in an ImmersiveSpace with play/pause, seek, recenter, and environment selector (black void default).
3. Stable 8K playback (effective 4K per eye) with head‑tracked audio when available.
4. Telemetry captures crashes and basic anonymous usage (toggle‑controlled).
5. No network required for any v1 functionality.

## Task Checklist
- [ ] Project setup (visionOS 2.0+, scenes, entitlements)
- [ ] Local manifest + loader
- [ ] Catalog and detail views
- [ ] Package 5 MV‑HEVC assets + posters
- [ ] MV‑HEVC immersive playback
- [ ] Minimal player controls + recenter + environment
- [ ] Telemetry integration
- [ ] On‑device performance validation
- [ ] TestFlight build

## Known Issues / Next Steps
- App size risk at ~5 GB; confirm store compliance and device storage experience.
- MV‑HEVC metadata correctness is required for playback; pre‑validate assets.
- Future expansion to downloads/streaming requires additional UX and infra.

## Future Considerations
- v2.0: Downloads (Backblaze B2 + Cloudflare), streaming (HLS/CMAF), manifest extensions, offline management.
- Captions/subtitles, continue‑watching, environment presets, passthrough scene.
- Subscriptions/premium gating and DRM (FairPlay) if needed later.

## Testing

### Model Specs
- [ ] Manifest parsing and validation for bundled assets.
- [ ] Poster/asset path resolution from bundle.

### Controller/Request Specs
- [ ] Player initialization for MV‑HEVC assets with and without spatial audio.
- [ ] Error surfaces for invalid/unplayable assets.

### Integration/Feature Specs
- [ ] End‑to‑end: Launch → Catalog → Detail → Play → Exit.
- [ ] Performance: 8K playback smoothness, memory/thermal checks.

---

## Progress Log
- 2025-08-10: Drafted and approved v1 offline plan (bundled MV‑HEVC samples). File created as `stories/WILD-1.md`. 
