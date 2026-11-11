# Enterprise Architecture Blueprint: Custom Sounds & Audio Asset Management

## 1. Executive Summary & Domain Boundaries

This document defines the production-grade, enterprise architecture for **Custom Notification Sounds & Call Ringtones** in the Setu platform.

To maintain architectural integrity across microservices, sound functionality is strictly decoupled into three bounded contexts:
1. **Media & Storage Domain (`core-media-service`):** Responsible for raw binary upload, audio format validation, loudness normalization, and immutable cloud storage (S3/CDN).
2. **User Identity & Personalization Domain (`idp-user-service`):** Responsible for the **Sound Asset Catalog** and **User Sound Preferences** mapping. It maintains user ownership, tenant policies, and cross-device sync.
3. **Communication & VoIP Runtime (`setu-webapp`):** Responsible for the client-side Web Audio graph, buffer caching, volume attenuation, burst deduplication, and priority interruption.

---

## 2. Audio Safety & Ingestion Guardrails

Allowing arbitrary user uploads in an enterprise communication suite introduces severe risks (audio bombing, digital clipping, network congestion, and storage bloat). The platform enforces strict guardrails:

| Constraint | Notification Sound | Call Ringtone | Enforcement Point |
|---|---|---|---|
| **Max Duration** | 3.0 seconds | 15.0 seconds | Client (pre-upload) + Server |
| **Max File Size** | 350 KB | 1.0 MB | Client (pre-upload) + Server |
| **Allowed Formats** | `.mp3`, `.ogg`, `.wav`, `.aac` | `.mp3`, `.ogg`, `.wav`, `.aac` | Client MIME check + Server magic bytes |
| **Loudness Normalization** | -16 LUFS (Broadcast standard) | -16 LUFS | Audio processing pipeline |
| **Peak Limiting** | -1.0 dBFS true peak | -1.0 dBFS true peak | Compressor/Limiter node |

### Client-Side Pre-Validation
Before any bytes leave the browser, `AudioContext.decodeAudioData()` inspects the file:
```ts
async function validateAudioFile(file: File, maxDurationSec: number, maxBytes: number): Promise<AudioBuffer> {
  if (file.size > maxBytes) {
    throw new Error(`File size (${Math.round(file.size / 1024)} KB) exceeds limit of ${Math.round(maxBytes / 1024)} KB`);
  }
  const arrayBuffer = await file.arrayBuffer();
  const audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
  const buffer = await audioCtx.decodeAudioData(arrayBuffer);
  if (buffer.duration > maxDurationSec) {
    throw new Error(`Audio length (${buffer.duration.toFixed(1)}s) exceeds max allowed length of ${maxDurationSec}s`);
  }
  return buffer;
}
```

---

## 3. Database Schema (`idp-user-service` / PostgreSQL)

### A. Sound Asset Registry (`sound_assets`)
```sql
CREATE TABLE IF NOT EXISTS sound_assets (
    id VARCHAR(64) PRIMARY KEY,                    -- e.g. "preset:soft-chime", "custom:a8f9-4b21"
    organization_id UUID,                          -- NULL for system presets; UUID for org-wide tones
    user_id UUID,                                  -- NULL for presets/org; UUID for personal tones
    name VARCHAR(100) NOT NULL,                    -- Friendly display label (e.g. "Executive Chime")
    asset_type VARCHAR(20) NOT NULL,               -- 'NOTIFICATION' or 'RINGTONE'
    file_url VARCHAR(512) NOT NULL,                -- CDN/Streaming URL (/api/core-media-service/get-file/...)
    duration_ms INT NOT NULL,                      -- Duration in milliseconds
    size_bytes INT NOT NULL,                       -- Exact file size
    mime_type VARCHAR(50) NOT NULL,                -- 'audio/mpeg', 'audio/ogg', etc.
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_sound_assets_owner ON sound_assets (organization_id, user_id, asset_type, is_active);
```

### B. User Sound Preferences (`user_sound_preferences`)
```sql
CREATE TABLE IF NOT EXISTS user_sound_preferences (
    user_id UUID PRIMARY KEY,
    organization_id UUID NOT NULL,
    master_sound_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    volume NUMERIC(3, 2) NOT NULL DEFAULT 0.85 CHECK (volume >= 0.0 AND volume <= 1.0),
    direct_message_sound_id VARCHAR(64) NOT NULL DEFAULT 'preset:soft-chime',
    direct_message_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    mention_sound_id VARCHAR(64) NOT NULL DEFAULT 'preset:clear-ping',
    mention_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    group_chat_sound_id VARCHAR(64) NOT NULL DEFAULT 'preset:gentle-bell',
    group_chat_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    channel_message_sound_id VARCHAR(64) NOT NULL DEFAULT 'preset:subtle-pop',
    channel_message_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    bot_message_sound_id VARCHAR(64) NOT NULL DEFAULT 'preset:digital-blip',
    bot_message_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    incoming_call_sound_id VARCHAR(64) NOT NULL DEFAULT 'preset:rising-tone',
    incoming_call_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 4. REST API Specification

### 1. Sound Catalog API
- `GET /api/idp-user-service/users/me/sound-catalog`
  - Returns system presets + user's uploaded custom sounds.
  - Response:
    ```json
    {
      "presets": [
        { "id": "preset:soft-chime", "name": "Soft Chime", "type": "NOTIFICATION", "durationMs": 1200 },
        { "id": "preset:rising-tone", "name": "Rising Tone", "type": "RINGTONE", "durationMs": 2400 }
      ],
      "customSounds": [
        { "id": "custom:8a2f-901c", "name": "My Custom Ringtone", "type": "RINGTONE", "durationMs": 4200, "fileUrl": "/api/core-media-service/get-file/8a2f.mp3" }
      ]
    }
    ```

- `POST /api/idp-user-service/users/me/sound-assets`
  - Multipart upload (`file`, `name`, `assetType`).
  - Stores binary in S3 via `core-media-service`, records row in `sound_assets`.
  - Returns created `SoundAssetDto`.

- `DELETE /api/idp-user-service/users/me/sound-assets/{id}`
  - Soft-deletes the asset. If any category in `user_sound_preferences` points to it, automatically falls back to default preset.

### 2. User Sound Preferences API
- `GET /api/idp-user-service/users/me/sound-preferences`
  - Fetches the user's sound preferences.
- `PUT /api/idp-user-service/users/me/sound-preferences`
  - Updates per-event sound mappings with optimistic UI rollback.

---

## 5. Client Web Audio Architecture (`setu-webapp`)

### The Web Audio Graph
All sounds pass through a single, shared `AudioContext` graph initialized on the first user interaction:
```
[ AudioBufferSourceNode ] 
           │
           ▼
     [ GainNode ] (Master volume attenuation: 0.0 to 1.0)
           │
           ▼
[ DynamicsCompressorNode ] (Soft limiter: prevents digital clipping)
           │
           ▼
[ AudioDestinationNode ] (Speakers / Headphones)
```

### Burst Protection & Call Ringing Engine
1. **Burst Gate:** When multiple chat messages arrive within 1.2s, only the highest priority event sounds; secondary events are dropped silently.
2. **Priority Preemption:** An incoming call (`RINGTONE`, priority 3) instantly stops any currently playing notification chime. While an incoming call is ringing, message chimes are muted.
3. **Looping Ringer:** The incoming call ringtone repeats cleanly on a loop until answered, rejected, or timed out.

