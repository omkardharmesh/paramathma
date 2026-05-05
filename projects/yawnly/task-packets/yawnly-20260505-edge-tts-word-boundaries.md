---
project: yawnly
id: yawnly-20260505-edge-tts-word-boundaries
title: "Edge TTS Word-Boundary Text Highlighting"
status: approved
mode: implement-local
owner: human
executor: hermes
created: 2026-05-05
updated: 2026-05-05
reviewed_by: opencode-go/kimi-k2.6
---

## Goal

Enable word-level text highlighting for Edge TTS playback so the UI can highlight individual words as they're spoken, instead of only full sentences. Currently Edge TTS word boundary metadata is supported by the protocol but disabled in the client config.

## Why

Edge TTS's WebSocket protocol returns `WordBoundary` events (offset + duration + text in 100ns ticks) identically to `SentenceBoundary` events. The infrastructure for parsing and routing them already exists — `"wordBoundaryEnabled"` is simply set to `"false"` in the config message. Sentence boundaries mostly work but have three gaps:
1. Not cached (cached Edge playback falls back to WPM estimation)
2. Not restored from cache (seek + replay loses boundary data)
3. No word-level timing (word boundary metadata is disabled)

## End Goal

- Word-level text highlighting works for Edge TTS playback (live + cached)
- Sentence-level highlighting continues to work (no regression)
- Both sentence and word boundaries are persisted to AND restored from `TtsAudioCacheManager`
- Kokoros/Chirp/Modal TTS paths are untouched
- WPM-based fallback still works when no boundaries are available
- `TtsTimingMetadata` schema extended to support word text (v1→v2)

## Non-Goals

- Not changing the Chirp/Modal/Kokoros TTS pipeline
- Not changing the UI layout or design
- Not changing the playback seek/skip logic
- Not adding SSML `<mark>` tag support (future enhancement)

## Files to Change (7 files)

### 1. EdgeTtsClient.kt
**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/EdgeTtsClient.kt`

Changes:
- **Line 95:** Add `onWordBoundary: (Long, String) -> Unit = { _, _ -> }` parameter to `synthesizeStream()`
- **Line 127:** Flip `"wordBoundaryEnabled":"false"` → `"wordBoundaryEnabled":"true"`
- **Parser refactor (lines 156-278):** Currently has 3 separate passes over `audio.metadata` JSON: `parseDurationHintMs` (line 282), `parseSentenceBoundaryHints` (line 232), and a new word parser. Edge TTS sends mixed SentenceBoundary + WordBoundary events in the same JSON array. Refactor to a single loop that extracts `offset`, `duration`, `text` once per event, then branches on `type`:
  - `"SentenceBoundary"` → dispatch via `onSentenceBoundary`
  - `"WordBoundary"` → dispatch via `onWordBoundary`
  - Both contribute to `maxEndTicks` for duration hint
  - This replaces `parseSentenceBoundaryHints()` and the planned separate `parseWordBoundaryHints()`
- **Add data class:** `WordBoundaryHint(endMs: Long, text: String)` — mirrors `SentenceBoundaryHint`
- **waitForFirstChunk() (line 313):** Calls `synthesizeStream()` — default parameter for `onWordBoundary` handles this automatically. No change needed.

### 2. TtsPlaybackEngine.kt
**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsPlaybackEngine.kt`

Changes:
- **Add data class:** `TtsWordBoundary(endMs: Long, text: String = "")` — identical structure to `TtsSentenceBoundary`
- **Add to interface:** `val wordBoundaries: StateFlow<List<TtsWordBoundary>>`

### 3. TtsEngine.kt
**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsEngine.kt`

Changes:
- **Add StateFlow (~line 109):** `_wordBoundaries` + `override val wordBoundaries`
- **playInternal() — line 646:** Also clear `_wordBoundaries`
- **playInternal() — lines 696-712:** Add `onWordBoundary` callback. Use `MutableStateFlow.update { }` for atomic append (instead of read-modify-write, which has a race window):
  ```kotlin
  onSentenceBoundary = { boundaryEndMs, boundaryText ->
      _sentenceBoundaries.update { existing ->
          val lastEndMs = existing.lastOrNull()?.endMs ?: Long.MIN_VALUE
          if (boundaryEndMs > lastEndMs) existing + TtsSentenceBoundary(boundaryEndMs, boundaryText.trim())
          else existing
      }
  },
  onWordBoundary = { boundaryEndMs, boundaryText ->
      _wordBoundaries.update { existing ->
          val lastEndMs = existing.lastOrNull()?.endMs ?: Long.MIN_VALUE
          if (boundaryEndMs > lastEndMs) existing + TtsWordBoundary(boundaryEndMs, boundaryText.trim())
          else existing
      }
  }
  ```
  Note: Also fix existing `_sentenceBoundaries` to use `.update { }` (currently uses read-modify-write at line 698-705).
  Note: Out-of-order boundary events are silently dropped. For sentences this is rare; for words (many small events) it's more likely under network jitter. Tracked as known limitation — acceptable for v1.

- **playInternal() — line 742-750:** After `player.finishFeeding()`, persist BOTH sentence and word boundaries to cache as `TtsTimingMetadata`:
  ```kotlin
  if (cacheAccumulator.size > 0) {
      val segments = buildTimingSegments(_sentenceBoundaries.value, _wordBoundaries.value)
      val metadata = TtsTimingMetadata(
          schemaVersion = 2,
          granularity = TtsTimingGranularity.WORD,
          source = TtsTimingSource.AUDIO_DURATION,
          segments = segments,
      )
      cacheManager.put(cacheKey, cacheAccumulator.toByteArray(), timingMetadata = metadata)
  }
  ```
  `buildTimingSegments()` is a new private helper that converts boundary lists into `List<TtsSegmentTiming>`. Only attached to final `complete=true` write — mid-stream partial writes stay metadata-free (line 724).

- **playCachedAudio() — line 431:** Restore `_sentenceBoundaries` and `_wordBoundaries` from cached `timingMetadata`:
  ```kotlin
  private suspend fun playCachedAudio(bytes: ByteArray, speed: Float): Boolean {
      // ... existing setup ...
      val metadata = _timingMetadata.value
      if (metadata != null) {
          _sentenceBoundaries.value = buildSentenceBoundariesFromMetadata(metadata)
          _wordBoundaries.value = buildWordBoundariesFromMetadata(metadata)
      }
      // ... rest of playback ...
  }
  ```
  Without this, cached replay has zero boundary data and falls back to WPM estimation.

- **seekTo() — line 389:** Already restores `_timingMetadata` from cache. Also restore `_sentenceBoundaries` and `_wordBoundaries`:
  ```kotlin
  cacheManager.getTimingMetadata(key)?.let {
      _timingMetadata.value = it
      _sentenceBoundaries.value = buildSentenceBoundariesFromMetadata(it)
      _wordBoundaries.value = buildWordBoundariesFromMetadata(it)
  }
  ```

- **stop() — line 949:** Also clear `_wordBoundaries`

### 4. TtsTimingMetadata.kt
**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/TtsTimingMetadata.kt`

Changes:
- **Add `text` field to `TtsSegmentTiming`:**
  ```kotlin
  data class TtsSegmentTiming(
      val segmentIndex: Int,
      val paragraphIndex: Int,
      val startMs: Long,
      val endMs: Long,
      val byteSize: Int? = null,
      val charCount: Int? = null,
      val text: String? = null,  // NEW — word boundary text
  )
  ```
- **Bump schema version:** Consumers check `schemaVersion` for forward compatibility. v1 segments have no `text`; v2 segments may have `text`.

### 5. StoryPlaybackCoordinator.kt
**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinator.kt`

Changes:
- **Add method:** `fun indexOfWordAtElapsed(elapsedMs: Long): Int` — returns word index for elapsed time
- **Add to SkipResult:** `val newWordIndex: Int = -1` (default -1, UI must handle gracefully)

### 6. StoryPlaybackCoordinatorImpl.kt
**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinatorImpl.kt`

Changes:
- **CRITICAL: `indexOfWordAtElapsed()` must be completely separate from `indexFromBoundaryTimeline()`.** Do NOT modify the existing sentence-boundary method — it's called by `skipBySeconds()` (line 160) and changing it could return word indices instead of sentence indices, breaking skip/snap-to-sentence behavior.
- **Add `indexOfWordAtElapsed()`:** Query `ttsEngine.wordBoundaries` directly:
  ```kotlin
  override fun indexOfWordAtElapsed(elapsedMs: Long): Int {
      val wordBoundaries = ttsEngine.wordBoundaries.value
      if (wordBoundaries.isNotEmpty()) {
          for (i in wordBoundaries.indices) {
              if (elapsedMs <= wordBoundaries[i].endMs) return i
          }
          return wordBoundaries.lastIndex
      }
      // Fall back to sentence-level
      val sentenceIndex = indexOfSentenceAtElapsed(elapsedMs)
      return if (sentenceIndex >= 0) sentenceIndex else -1
  }
  ```
- **No changes to `indexFromBoundaryTimeline()` or `indexOfSentenceAtElapsed()`.** These remain sentence-only.

### 7. StoryReaderViewModel.kt (optional enhancement — can be deferred)
**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/presentation/viewmodel/StoryReaderViewModel.kt`

Changes (can be deferred to separate task):
- Use `indexOfWordAtElapsed()` to drive per-word highlight instead of per-sentence
- Fall back to sentence-level when word boundaries aren't populated yet

## Kimi K2.6 Review Findings (Incorporated)

1. ✅ **Cache restoration gap fixed** — `playCachedAudio()` and `seekTo()` now restore boundaries from metadata
2. ✅ **TtsSegmentTiming.text field added** — word text can survive cache round-trips
3. ✅ **Parser refactored to single loop** — avoids re-parsing same JSON 3 times, correctly handles mixed arrays
4. ✅ **`.update { }` for atomic StateFlow writes** — eliminates race window on boundary append
5. ✅ **Coordinator isolation** — `indexOfWordAtElapsed()` is a separate method, `indexFromBoundaryTimeline()` untouched
6. ✅ **Schema version bumped v1→v2** — forward compatibility
7. ⚠️ **Out-of-order word boundaries** — silently dropped (same as sentence boundaries). Known limitation, acceptable for v1.
8. ⚠️ **`waitForFirstChunk()` compatibility** — handled by default parameter on `onWordBoundary`.

## Regression Prevention Checklist

- [ ] Sentence boundary highlighting still works (test with Edge TTS, verify sentence pills highlight correctly)
- [ ] Word boundary highlighting works (test with Edge TTS, verify word-level pills highlight)
- [ ] WPM-based fallback works when no metadata arrives (first 1-2 seconds of playback)
- [ ] Kokoros/Chirp TTS path untouched — verify by playing a story via Chirp provider
- [ ] Modal TTS path untouched — verify by playing via Modal provider
- [ ] Cached playback uses persisted boundaries (play Edge TTS story → stop → replay → sentence + word highlighting should work)
- [ ] Seek/skip still snaps to correct sentence (NOT affected by word boundaries — verified by coordinator isolation)
- [ ] `stop()` clears all boundary state (no stale data bleeding into next story)
- [ ] Speed changes don't break timing (boundaries are in absolute ms, independent of playback rate)
- [ ] Schema v1 cache entries don't crash (v1 segments have `text=null`, handled by `String?`)

## Allowed Paths

- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/EdgeTtsClient.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsPlaybackEngine.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsEngine.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/TtsTimingMetadata.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinator.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinatorImpl.kt`
- `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/presentation/viewmodel/StoryReaderViewModel.kt` (optional)

## Verification

```bash
# Android compile check
rtk ./gradlew :composeApp:compileAndroidMain
# Common metadata check (KMP — catches expect/actual mismatch, javaClass usage)
rtk ./gradlew :composeApp:compileCommonMainKotlinMetadata
# iOS target compile (catches platform-specific API misuse)
rtk ./gradlew :composeApp:compileKotlinIosSimulatorArm64
# Full APK build
rtk ./gradlew :androidApp:assembleDebug
```

## Stop Conditions

- Any Kotlin compilation error
- If more than the 7 allowed files need changes
- If sentence boundary highlighting breaks (test on device)
- If Chirp/Modal playback breaks
- If cached playback with old schema v1 entries crashes (test with existing cache entries)

## Output Required

- Task packet (this file) — plan approval
- Implementation after `GO` signal
- Verification build results (all 4 compile targets)
