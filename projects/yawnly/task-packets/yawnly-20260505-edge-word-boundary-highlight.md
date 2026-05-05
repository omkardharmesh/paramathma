---
id: yawnly-20260505-edge-word-boundary-highlight
project: yawnly
repo_path: /Users/eloelo/Downloads/Yawnly
status: draft
mode: plan-only
risk: medium
created: 2026-05-05
owner: human
executor: hermes
---

# Plan: Edge TTS Word-Boundary Text Highlighting

## Goal

Enable per-word text highlighting during Edge TTS playback by flipping `wordBoundaryEnabled` to `true` in the Edge TTS WebSocket config, parsing the resulting WordBoundary metadata from `audio.metadata` frames, threading it through the engine and coordinator, persisting it to cache, and exposing it to the UI. Sentence-boundary highlighting remains intact and is the fallback when word boundaries are unavailable (Kokoros/Chirp paths, cached playback without word metadata).

## Why

Current sentence-level highlighting is coarse — it lights up entire sentences, which can be 15–20+ words long. Word-level highlighting provides tighter visual sync between the spoken word and the text on screen, improving readability and engagement for young readers. Edge TTS already supports word-boundary metadata via its WebSocket protocol; the feature is one config flag away from being usable. This is a high-impact, low-risk enhancement.

## End Goal

A `plan-only` task packet with exact diff-level code changes for all 5 files, ready for human review and `GO` approval. Every code change specified at the line/function level. No regressions on sentence-boundary path, Chirp/Kokoros path, WPM fallback, or cached playback.

## Output Format

This document — `plan-only` task packet with per-file code change specifications. When approved, the same packet serves as the implementation guide.

## Recommended Approach

1. **EdgeTtsClient.kt** — Flip the flag, add `WordBoundaryHint` data class + parser, add `onWordBoundary` callback alongside existing `onSentenceBoundary`.
2. **TtsPlaybackEngine.kt** — Add `TtsWordBoundary` domain data class and `wordBoundaries` StateFlow to the interface.
3. **TtsEngine.kt** — Add `_wordBoundaries` flow, wire the callback, persist word boundaries into cache as `TtsTimingMetadata` after stream completes, clear on stop.
4. **TtsTimingMetadata.kt** — Add optional `sentenceBoundaries` and `wordBoundaries` lists so Edge TTS timing data survives cache round-trips.
5. **StoryPlaybackCoordinator.kt** — Add word-level span computation methods to the interface.
6. **StoryPlaybackCoordinatorImpl.kt** — Add word-span data structures, word-level active-span computation, fall-back to sentence-level.
7. **StoryReaderViewModel.kt** — In `observePlaybackPosition`, prefer word-level spans when available; existing `activeSentenceStart`/`activeSentenceEnd` fields carry word ranges transparently.
8. **StoryTextCard.kt** — No changes needed. `HighlightableParagraph` already renders arbitrary `activeStart`/`activeEnd` character ranges.

## Common Failure Handling

- **Word boundaries arrive before sentence boundaries**: The Edge TTS protocol can emit WordBoundary events interleaved with SentenceBoundary events. Accumulate both independently. Never assume ordering.
- **Word boundary text doesn't match original text**: Edge TTS may apply SSML normalization, pronunciation overrides, or text cleanup. The word boundary text is advisory for display but MUST NOT be used for character-offset computation. Compute word character spans from the original paragraph text independently.
- **Cached playback without word metadata**: Existing cached audio won't have word boundaries. The `TtsTimingMetadata.schemaVersion` (currently 1) can be bumped to 2 to distinguish old cache from new. Cache queries with `schemaVersion < 2` → no word boundaries available → fall back to sentence-level.
- **Chirp/Kokoros path emits word boundaries**: Chirp generates `TtsTimingMetadata` with `PARAGRAPH` granularity. That path sets `_timingMetadata.value` directly. Word boundaries from Edge must NOT clobber Chirp's metadata. The word-boundary path is Edge-only.
- **Empty word boundaries from Edge**: The `onWordBoundary` callback may fire zero times (e.g., very short text, Edge server regression). The coordinator must gracefully handle empty word boundaries and fall back to sentence-level.
- **WPM fallback when no boundaries are available**: The existing WPM-based sentence timing must still work identically. Word boundaries are additive.

## Allowed Paths

- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/EdgeTtsClient.kt`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsPlaybackEngine.kt`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsEngine.kt`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/TtsTimingMetadata.kt`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/TtsAudioCacheManager.kt`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinator.kt`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinatorImpl.kt`
- `/Users/eloelo/Downloads/Yawnly/composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/presentation/viewmodel/StoryReaderViewModel.kt`
- `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/`

## Required Context

- `Yawnly/AGENTS.md` — architecture gotchas, review gates, layer contracts
- `projects/yawnly/context.md` — project context, stack, hard constraints
- `projects/yawnly/decisions.md` — D-series decisions
- Source files listed in **Reference Files** below

## Reference Files In Repo

All files below have been read and analyzed in full:

| # | File | Lines | Purpose |
|---|------|-------|---------|
| 1 | `EdgeTtsClient.kt` | 418 | Edge TTS WebSocket client; needs flag flip + WordBoundary parser |
| 2 | `TtsPlaybackEngine.kt` | 48 | Engine interface; needs `wordBoundaries` StateFlow |
| 3 | `TtsEngine.kt` | 992 | Engine implementation; wires callbacks, persists to cache |
| 4 | `TtsTimingMetadata.kt` | 27 | Timing metadata model; needs boundary list fields |
| 5 | `TtsAudioCacheManager.kt` | 52 | Cache; already supports `TtsTimingMetadata` storage |
| 6 | `StoryPlaybackCoordinator.kt` | 122 | Coordinator interface; needs word-span methods |
| 7 | `StoryPlaybackCoordinatorImpl.kt` | 627 | Coordinator impl; word-level span computation |
| 8 | `StoryReaderViewModel.kt` | 716 | VM; maps coordinator → screen state |
| 9 | `StoryReaderScreenState.kt` | 63 | Screen state model |
| 10 | `StoryTextCard.kt` | 280 | UI rendering; NO CHANGES NEEDED |

## Non-Goals

- No changes to `StoryTextCard.kt` or any UI composable
- No changes to Chirp/Kokoros TTS paths
- No changes to WPM-based duration estimation
- No changes to the `HighlightableParagraph` composable
- No new streaming protocol — word boundaries arrive on the same WebSocket connection
- No remote config toggle — `wordBoundaryEnabled` is always `true` after this change (Edge TTS always supports it)
- No privacy/data implications — word boundary metadata is transient and only cached locally in `TtsAudioCacheManager`

## Stop Conditions

- Any file outside Allowed Paths needs editing
- Schema migration needed in Supabase
- Koin module registration needed (word boundaries are internal to existing singletons — no new DI)
- Build fails after changes

## Verification

After implementation, run:
```bash
rtk ./gradlew :composeApp:compileKotlinAndroid
```
Pass criteria: zero compilation errors. No runtime test suite exists for TTS code; manual testing on-device validates word highlighting.

---

# Implementation Plan — Per-File Code Changes

## File 1: `EdgeTtsClient.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/EdgeTtsClient.kt`

### Change 1.1 — Flip wordBoundaryEnabled flag (line 127)

**Current (line 127):**
```kotlin
"""{"context":{"synthesis":{"audio":{"metadataoptions":{"sentenceBoundaryEnabled":"true","wordBoundaryEnabled":"false"},"outputFormat":"$outputFormat"}}}}"""
```

**Replace with:**
```kotlin
"""{"context":{"synthesis":{"audio":{"metadataoptions":{"sentenceBoundaryEnabled":"true","wordBoundaryEnabled":"true"},"outputFormat":"$outputFormat"}}}}"""
```

### Change 1.2 — Add WordBoundaryHint data class (after line 230, after SentenceBoundaryHint)

**Current (lines 227–230):**
```kotlin
    private data class SentenceBoundaryHint(
        val endMs: Long,
        val text: String,
    )
```

**Add after line 230:**
```kotlin
    private data class WordBoundaryHint(
        val endMs: Long,
        val text: String,
        val durationMs: Long,
    )
```

### Change 1.3 — Add onWordBoundary callback parameter to synthesizeStream (line 95)

**Current (lines 86–96):**
```kotlin
    fun synthesizeStream(
        text: String,
        voice: String = "en-US-EmmaMultilingualNeural",
        rate: String = "-20%",
        pitch: String = "+0Hz",
        volume: String = "+0%",
        outputFormat: String = OUTPUT_FORMAT_MP3_24KHZ_48KBPS,
        pronunciationOverrides: Map<String, String> = emptyMap(),
        onDurationHintMs: (Long) -> Unit = {},
        onSentenceBoundary: (Long, String) -> Unit = { _, _ -> },
    ): Flow<ByteArray> = channelFlow {
```

**Replace with:**
```kotlin
    fun synthesizeStream(
        text: String,
        voice: String = "en-US-EmmaMultilingualNeural",
        rate: String = "-20%",
        pitch: String = "+0Hz",
        volume: String = "+0%",
        outputFormat: String = OUTPUT_FORMAT_MP3_24KHZ_48KBPS,
        pronunciationOverrides: Map<String, String> = emptyMap(),
        onDurationHintMs: (Long) -> Unit = {},
        onSentenceBoundary: (Long, String) -> Unit = { _, _ -> },
        onWordBoundary: (Long, String, Long) -> Unit = { _, _, _ -> },
    ): Flow<ByteArray> = channelFlow {
```

### Change 1.4 — Parse word boundaries in audio.metadata frame handler (insert ~line 192, after sentence boundary loop closes)

**Current (lines 176–192):**
```kotlin
                            val sentenceBoundaries = parseSentenceBoundaryHints(body)
                            if (sentenceBoundaries.isNotEmpty()) {
                                Napier.d(
                                    tag = TAG,
                                    message = "◀ audio.metadata sentenceBoundaries=${sentenceBoundaries.size}",
                                )
                                sentenceBoundaries.forEach { boundary ->
                                    try {
                                        onSentenceBoundary(boundary.endMs, boundary.text)
                                    } catch (callbackError: Exception) {
                                        Napier.w(
                                            tag = TAG,
                                            message = "◀ sentence callback failed: ${callbackError.message}",
                                        )
                                    }
                                }
                            }
                        }
```

**Replace with (insert word boundary parsing after the sentence boundary block, before the closing `}` of the `if (path == "audio.metadata")` block):**
```kotlin
                            val sentenceBoundaries = parseSentenceBoundaryHints(body)
                            if (sentenceBoundaries.isNotEmpty()) {
                                Napier.d(
                                    tag = TAG,
                                    message = "◀ audio.metadata sentenceBoundaries=${sentenceBoundaries.size}",
                                )
                                sentenceBoundaries.forEach { boundary ->
                                    try {
                                        onSentenceBoundary(boundary.endMs, boundary.text)
                                    } catch (callbackError: Exception) {
                                        Napier.w(
                                            tag = TAG,
                                            message = "◀ sentence callback failed: ${callbackError.message}",
                                        )
                                    }
                                }
                            }

                            val wordBoundaries = parseWordBoundaryHints(body)
                            if (wordBoundaries.isNotEmpty()) {
                                Napier.d(
                                    tag = TAG,
                                    message = "◀ audio.metadata wordBoundaries=${wordBoundaries.size}",
                                )
                                wordBoundaries.forEach { boundary ->
                                    try {
                                        onWordBoundary(boundary.endMs, boundary.text, boundary.durationMs)
                                    } catch (callbackError: Exception) {
                                        Napier.w(
                                            tag = TAG,
                                            message = "◀ word callback failed: ${callbackError.message}",
                                        )
                                    }
                                }
                            }
                        }
```

### Change 1.5 — Add parseWordBoundaryHints method (after parseSentenceBoundaryHints, before parseDurationHintMs, ~line 278)

**Insert after line 278 (the closing `}` of `parseSentenceBoundaryHints`):**
```kotlin
    private fun parseWordBoundaryHints(metadataBody: String): List<WordBoundaryHint> {
        if (metadataBody.isBlank()) return emptyList()
        return try {
            val root = json.parseToJsonElement(metadataBody).jsonObject
            val events = root["Metadata"]?.jsonArray ?: root["metadata"]?.jsonArray ?: return emptyList()
            val wordBoundaries = mutableListOf<WordBoundaryHint>()

            for (event in events) {
                val eventObj = event.jsonObject
                val type = eventObj["Type"]?.jsonPrimitive?.contentOrNull
                    ?: eventObj["type"]?.jsonPrimitive?.contentOrNull
                val data = eventObj["Data"]?.jsonObject ?: eventObj["data"]?.jsonObject ?: continue
                val offset = data["Offset"]?.jsonPrimitive?.longOrNull
                    ?: data["offset"]?.jsonPrimitive?.longOrNull
                    ?: continue
                val duration = data["Duration"]?.jsonPrimitive?.longOrNull
                    ?: data["duration"]?.jsonPrimitive?.longOrNull
                    ?: 0L
                val safeDuration = duration.coerceAtLeast(0L)
                val endTicks = if (Long.MAX_VALUE - offset < safeDuration) {
                    Long.MAX_VALUE
                } else {
                    offset + safeDuration
                }
                if (type.equals("WordBoundary", ignoreCase = true)) {
                    // Edge TTS WordBoundary event: Data.text.Text contains the word,
                    // Data.text.Length contains character count.
                    val textData = data["text"]?.jsonObject
                        ?: data["Text"]?.jsonObject
                    val wordText = textData?.get("Text")?.jsonPrimitive?.contentOrNull
                        ?: textData?.get("text")?.jsonPrimitive?.contentOrNull
                        ?: ""
                    val endMs = (endTicks / TICKS_PER_MS).coerceAtLeast(0L)
                    val durationMs = (safeDuration / TICKS_PER_MS).coerceAtLeast(0L)
                    if (endMs > 0L && wordText.isNotBlank()) {
                        wordBoundaries.add(
                            WordBoundaryHint(
                                endMs = endMs,
                                text = wordText.trim(),
                                durationMs = durationMs,
                            )
                        )
                    }
                }
            }

            wordBoundaries
        } catch (_: Exception) {
            emptyList()
        }
    }
```

---

## File 2: `TtsPlaybackEngine.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsPlaybackEngine.kt`

### Change 2.1 — Add TtsWordBoundary data class (after TtsSentenceBoundary, before interface, ~line 10)

**Current (lines 7–10):**
```kotlin
data class TtsSentenceBoundary(
    val endMs: Long,
    val text: String = "",
)
```

**Add after line 10:**
```kotlin
data class TtsWordBoundary(
    val endMs: Long,
    val text: String = "",
    val durationMs: Long = 0L,
)
```

### Change 2.2 — Add wordBoundaries StateFlow to interface (after sentenceBoundaries, ~line 22)

**Current (lines 21–22):**
```kotlin
    val sentenceBoundaries: StateFlow<List<TtsSentenceBoundary>>
    val timingMetadata: StateFlow<TtsTimingMetadata?>
```

**Replace with:**
```kotlin
    val sentenceBoundaries: StateFlow<List<TtsSentenceBoundary>>
    val wordBoundaries: StateFlow<List<TtsWordBoundary>>
    val timingMetadata: StateFlow<TtsTimingMetadata?>
```

---

## File 3: `TtsEngine.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/domain/TtsEngine.kt`

### Change 3.1 — Add _wordBoundaries MutableStateFlow (after _sentenceBoundaries, ~line 109)

**Current (lines 109–110):**
```kotlin
    private val _sentenceBoundaries = MutableStateFlow<List<TtsSentenceBoundary>>(emptyList())
    override val sentenceBoundaries: StateFlow<List<TtsSentenceBoundary>> = _sentenceBoundaries.asStateFlow()
```

**Replace with:**
```kotlin
    private val _sentenceBoundaries = MutableStateFlow<List<TtsSentenceBoundary>>(emptyList())
    override val sentenceBoundaries: StateFlow<List<TtsSentenceBoundary>> = _sentenceBoundaries.asStateFlow()

    private val _wordBoundaries = MutableStateFlow<List<TtsWordBoundary>>(emptyList())
    override val wordBoundaries: StateFlow<List<TtsWordBoundary>> = _wordBoundaries.asStateFlow()
```

### Change 3.2 — Clear word boundaries in playInternal() (~line 646)

**Current (line 646):**
```kotlin
        _sentenceBoundaries.value = emptyList()
        _timingMetadata.value = null
```

**Replace with:**
```kotlin
        _sentenceBoundaries.value = emptyList()
        _wordBoundaries.value = emptyList()
        _timingMetadata.value = null
```

### Change 3.3 — Wire onWordBoundary callback in playInternal() (after onSentenceBoundary callback block, ~line 712)

**Current (lines 696–712):**
```kotlin
            onSentenceBoundary = { boundaryEndMs, boundaryText ->
                if (boundaryEndMs > 0L) {
                    val existing = _sentenceBoundaries.value
                    val lastKnownEndMs = existing.lastOrNull()?.endMs ?: Long.MIN_VALUE
                    if (boundaryEndMs > lastKnownEndMs) {
                        val next = TtsSentenceBoundary(
                            endMs = boundaryEndMs,
                            text = boundaryText.trim(),
                        )
                        _sentenceBoundaries.value = existing + next
                        Napier.d(
                            tag = TAG,
                            message = "edge sentence boundary added endMs=$boundaryEndMs count=${existing.size + 1} text='${next.text.take(32)}'",
                        )
                    }
                }
            },
        ).collect { chunk ->
```

**Replace with:**
```kotlin
            onSentenceBoundary = { boundaryEndMs, boundaryText ->
                if (boundaryEndMs > 0L) {
                    val existing = _sentenceBoundaries.value
                    val lastKnownEndMs = existing.lastOrNull()?.endMs ?: Long.MIN_VALUE
                    if (boundaryEndMs > lastKnownEndMs) {
                        val next = TtsSentenceBoundary(
                            endMs = boundaryEndMs,
                            text = boundaryText.trim(),
                        )
                        _sentenceBoundaries.value = existing + next
                        Napier.d(
                            tag = TAG,
                            message = "edge sentence boundary added endMs=$boundaryEndMs count=${existing.size + 1} text='${next.text.take(32)}'",
                        )
                    }
                }
            },
            onWordBoundary = { boundaryEndMs, boundaryText, boundaryDurationMs ->
                if (boundaryEndMs > 0L) {
                    val existing = _wordBoundaries.value
                    val lastKnownEndMs = existing.lastOrNull()?.endMs ?: Long.MIN_VALUE
                    if (boundaryEndMs > lastKnownEndMs) {
                        val next = TtsWordBoundary(
                            endMs = boundaryEndMs,
                            text = boundaryText.trim(),
                            durationMs = boundaryDurationMs,
                        )
                        _wordBoundaries.value = existing + next
                        Napier.d(
                            tag = TAG,
                            message = "edge word boundary added endMs=$boundaryEndMs count=${existing.size + 1} text='${next.text.take(32)}'",
                        )
                    }
                }
            },
        ).collect { chunk ->
```

### Change 3.4 — Persist sentence + word boundaries as TtsTimingMetadata after stream completes (after cache write, ~line 745)

**Current (lines 742–750):**
```kotlin
        if (!progressiveMode) {
            setState(TtsState.PLAYING)
        }
        if (cacheAccumulator.size > 0) {
            cacheManager.put(cacheKey, cacheAccumulator.toByteArray())
        }
        player.finishFeeding()
        finalizePlaybackAfterNaturalEnd(player)
        return true
```

**Replace with:**
```kotlin
        if (!progressiveMode) {
            setState(TtsState.PLAYING)
        }
        if (cacheAccumulator.size > 0) {
            // Persist sentence + word boundary metadata alongside audio bytes
            // so cached replays have timing data for highlighting.
            val sentenceBoundariesForCache = _sentenceBoundaries.value.map {
                TtsTimingSentenceBoundary(endMs = it.endMs, text = it.text)
            }
            val wordBoundariesForCache = _wordBoundaries.value.map {
                TtsTimingWordBoundary(endMs = it.endMs, text = it.text, durationMs = it.durationMs)
            }
            val edgeTimingMetadata = TtsTimingMetadata(
                schemaVersion = 2,
                granularity = TtsTimingGranularity.WORD,
                source = TtsTimingSource.AUDIO_DURATION,
                segments = emptyList(),
                sentenceBoundaries = sentenceBoundariesForCache.ifEmpty { null },
                wordBoundaries = wordBoundariesForCache.ifEmpty { null },
            )
            _timingMetadata.value = edgeTimingMetadata
            cacheManager.put(cacheKey, cacheAccumulator.toByteArray(), timingMetadata = edgeTimingMetadata)
        }
        player.finishFeeding()
        finalizePlaybackAfterNaturalEnd(player)
        return true
```

### Change 3.5 — Add import for new data types (top of file, ~line 8)

**Current (line 8):**
```kotlin
import com.techmo.yawnly.core.feature.tts.data.TtsTimingGranularity
```

**Replace with:**
```kotlin
import com.techmo.yawnly.core.feature.tts.data.TtsTimingGranularity
import com.techmo.yawnly.core.feature.tts.data.TtsTimingSentenceBoundary
import com.techmo.yawnly.core.feature.tts.data.TtsTimingWordBoundary
```

### Change 3.6 — Clear word boundaries in stop() (~line 946)

**Current (line 946):**
```kotlin
        _sentenceBoundaries.value = emptyList()
        _timingMetadata.value = null
```

**Replace with:**
```kotlin
        _sentenceBoundaries.value = emptyList()
        _wordBoundaries.value = emptyList()
        _timingMetadata.value = null
```

---

## File 4: `TtsTimingMetadata.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/TtsTimingMetadata.kt`

### Change 4.1 — Add sentence + word boundary data classes (after imports, before TtsTimingMetadata)

**Add after line 11 (after `TtsTimingSource`):**
```kotlin
/**
 * Persistable sentence-boundary snapshot for Edge TTS cache entries.
 * Mirrors [com.techmo.yawnly.core.feature.tts.domain.TtsSentenceBoundary]
 * but lives in the data layer so [TtsTimingMetadata] stays serializable
 * without importing domain types.
 */
data class TtsTimingSentenceBoundary(
    val endMs: Long,
    val text: String = "",
)

/**
 * Persistable word-boundary snapshot for Edge TTS cache entries.
 * Mirrors [com.techmo.yawnly.core.feature.tts.domain.TtsWordBoundary]
 * but lives in the data layer.
 */
data class TtsTimingWordBoundary(
    val endMs: Long,
    val text: String = "",
    val durationMs: Long = 0L,
)
```

### Change 4.2 — Add boundary fields to TtsTimingMetadata

**Current (lines 13–18):**
```kotlin
data class TtsTimingMetadata(
    val schemaVersion: Int = 1,
    val granularity: TtsTimingGranularity,
    val source: TtsTimingSource,
    val segments: List<TtsSegmentTiming>,
)
```

**Replace with:**
```kotlin
data class TtsTimingMetadata(
    val schemaVersion: Int = 1,
    val granularity: TtsTimingGranularity,
    val source: TtsTimingSource,
    val segments: List<TtsSegmentTiming>,
    /** Edge TTS sentence boundaries (null when not applicable, e.g., Chirp/MODAL paths). */
    val sentenceBoundaries: List<TtsTimingSentenceBoundary>? = null,
    /** Edge TTS word boundaries (null when not applicable or when schema < 2). */
    val wordBoundaries: List<TtsTimingWordBoundary>? = null,
)
```

---

## File 5: `TtsAudioCacheManager.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/core/feature/tts/data/TtsAudioCacheManager.kt`

### No changes needed.

The cache already accepts `TtsTimingMetadata?` in its `put()` method (line 39–47) and returns it via `getTimingMetadata()` (lines 34–37). The new optional fields in `TtsTimingMetadata` are transparent to the cache manager.

However, note: `getTimingMetadata()` already guards on `snapshot.complete == true`. The Edge TTS path now calls `cacheManager.put()` with `complete = true` and `timingMetadata` set — this matches the existing contract exactly.

---

## File 6: `StoryPlaybackCoordinator.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinator.kt`

### Change 6.1 — Add WordSpan data class and word-level methods

**Add after `SentenceSpan` data class (after line 121):**
```kotlin
    data class WordSpan(
        val paragraphIndex: Int,
        val sentenceIndex: Int,
        val wordIndex: Int,
        val start: Int,
        val endExclusive: Int,
    )

    /** Look up the word index for a given elapsed time, when word boundaries are available. */
    fun indexOfWordAtElapsed(elapsedMs: Long): Int

    /** Get the char-range span for a word within its sentence. */
    fun getActiveWordSpan(sentenceIndex: Int, wordIndex: Int): WordSpan?
```

---

## File 7: `StoryPlaybackCoordinatorImpl.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/domain/playback/StoryPlaybackCoordinatorImpl.kt`

### Change 7.1 — Add word-level data structures (after sentence structures, ~line 41)

**Current (lines 38–41):**
```kotlin
    private var sentences: List<StoryPlaybackCoordinator.SentenceSpan> = emptyList()
    private var sentenceNormalizedTexts: List<String> = emptyList()
    private var sentenceDurationsMs: List<Long> = emptyList()
    private var sentenceCumulativeMs: List<Long> = emptyList()
```

**Add after line 41:**
```kotlin
    // Word-level timing data. Populated from Edge TTS word boundaries
    // when available; empty otherwise (falls back to sentence-level).
    private var wordsBySentence: List<List<StoryPlaybackCoordinator.WordSpan>> = emptyList()
    private var wordEndMsBySentence: List<List<Long>> = emptyList()
```

### Change 7.2 — Add import for TtsWordBoundary (top of file, ~line 5)

**Current (line 5):**
```kotlin
import com.techmo.yawnly.core.feature.tts.domain.TtsSentenceBoundary
```

**Add after:**
```kotlin
import com.techmo.yawnly.core.feature.tts.domain.TtsWordBoundary
```

### Change 7.3 — Populate word spans in computeSentenceTimings() (inside the paragraph loop, ~line 576)

**Current (lines 566–586):**
```kotlin
    private fun computeSentenceTimings(paragraphs: List<String>, displaySpeed: Float) {
        val spans = mutableListOf<StoryPlaybackCoordinator.SentenceSpan>()
        val normalizedTexts = mutableListOf<String>()
        paragraphs.forEachIndexed { pIdx, paragraph ->
            sentenceRangesInParagraph(paragraph).forEach { range ->
                val text = paragraph.substring(range.first, range.last + 1)
                val wordCount = text.split(Regex("\\s+"))
                    .count { it.isNotBlank() }
                    .coerceAtLeast(1)
                normalizedTexts.add(normalizeForAlignment(text))
                spans.add(
                    StoryPlaybackCoordinator.SentenceSpan(
                        paragraphIndex = pIdx,
                        start = range.first,
                        endExclusive = range.last + 1,
                        wordCount = wordCount,
                    )
                )
            }
        }
        sentences = spans
        boundaryAlignmentMode = "unknown"
```

**Replace with:**
```kotlin
    private fun computeSentenceTimings(paragraphs: List<String>, displaySpeed: Float) {
        val spans = mutableListOf<StoryPlaybackCoordinator.SentenceSpan>()
        val normalizedTexts = mutableListOf<String>()
        val allWordSpans = mutableListOf<List<StoryPlaybackCoordinator.WordSpan>>()
        paragraphs.forEachIndexed { pIdx, paragraph ->
            sentenceRangesInParagraph(paragraph).forEachIndexed { sIdx, range ->
                val text = paragraph.substring(range.first, range.last + 1)
                val wordCount = text.split(Regex("\\s+"))
                    .count { it.isNotBlank() }
                    .coerceAtLeast(1)
                normalizedTexts.add(normalizeForAlignment(text))
                spans.add(
                    StoryPlaybackCoordinator.SentenceSpan(
                        paragraphIndex = pIdx,
                        start = range.first,
                        endExclusive = range.last + 1,
                        wordCount = wordCount,
                    )
                )
                // Compute word-level character spans within this sentence.
                val wordSpans = computeWordSpansInSentence(
                    paragraph = paragraph,
                    sentenceStart = range.first,
                    sentenceEnd = range.last + 1,
                    paragraphIndex = pIdx,
                    sentenceIndex = sIdx,
                )
                allWordSpans.add(wordSpans)
            }
        }
        sentences = spans
        wordsBySentence = allWordSpans
        boundaryAlignmentMode = "unknown"
```

### Change 7.4 — Reset word data when sentences are empty (after line 587)

**Current (lines 587–593):**
```kotlin
        if (spans.isEmpty()) {
            sentenceNormalizedTexts = emptyList()
            sentenceDurationsMs = emptyList()
            sentenceCumulativeMs = emptyList()
            return
        }
```

**Replace with:**
```kotlin
        if (spans.isEmpty()) {
            sentenceNormalizedTexts = emptyList()
            sentenceDurationsMs = emptyList()
            sentenceCumulativeMs = emptyList()
            wordsBySentence = emptyList()
            wordEndMsBySentence = emptyList()
            return
        }
```

### Change 7.5 — Add computeWordSpansInSentence helper (~before computeSentenceTimings, ~line 565)

**Insert before computeSentenceTimings (after the sentenceRangesInParagraph method, ~line 510):**
```kotlin
    /**
     * Splits a sentence into word character-offset spans.
     * Words are delimited by whitespace; each span records its
     * absolute position within the paragraph for highlighting.
     */
    private fun computeWordSpansInSentence(
        paragraph: String,
        sentenceStart: Int,
        sentenceEnd: Int,
        paragraphIndex: Int,
        sentenceIndex: Int,
    ): List<StoryPlaybackCoordinator.WordSpan> {
        if (sentenceStart >= sentenceEnd || sentenceEnd > paragraph.length) return emptyList()
        val sentenceText = paragraph.substring(sentenceStart, sentenceEnd)
        val words = mutableListOf<StoryPlaybackCoordinator.WordSpan>()
        var pos = sentenceStart
        val wordPattern = Regex("\\S+")
        wordPattern.findAll(sentenceText).forEachIndexed { wIdx, match ->
            val wordStart = sentenceStart + match.range.first
            val wordEnd = sentenceStart + match.range.last + 1
            words.add(
                StoryPlaybackCoordinator.WordSpan(
                    paragraphIndex = paragraphIndex,
                    sentenceIndex = sentenceIndex,
                    wordIndex = wIdx,
                    start = wordStart,
                    endExclusive = wordEnd,
                )
            )
        }
        return words
    }
```

### Change 7.6 — Add indexOfWordAtElapsed method (after indexOfSentenceAtElapsed, ~line 280)

**Insert after line 280 (after the closing `}` of `indexOfSentenceAtElapsed`):**
```kotlin
    /**
     * Returns the word index within the active sentence at [elapsedMs],
     * or -1 if word boundaries are unavailable.
     *
     * When real Edge word boundaries exist in the engine, use them for
     * precise mapping. Otherwise return -1 so callers fall back to
     * sentence-level highlighting.
     */
    override fun indexOfWordAtElapsed(elapsedMs: Long): Int {
        val boundaries = ttsEngine.wordBoundaries.value
        val metadata = ttsEngine.timingMetadata.value
        // Only trust word boundaries from Edge TTS (schema >= 2, WORD granularity).
        val hasEdgeWordBoundaries = metadata != null &&
            metadata.schemaVersion >= 2 &&
            metadata.granularity == TtsTimingGranularity.WORD &&
            metadata.wordBoundaries != null &&
            metadata.wordBoundaries.isNotEmpty()

        if (!hasEdgeWordBoundaries || boundaries.isEmpty()) return -1

        // Find the word whose endMs contains elapsedMs.
        for (i in boundaries.indices) {
            if (elapsedMs <= boundaries[i].endMs) return i
        }
        // Past all word boundaries — return last word.
        return boundaries.lastIndex
    }
```

### Change 7.7 — Add getActiveWordSpan method (after getActiveSpan, ~line 409)

**Insert after line 409 (after the closing `}` of `getActiveSpan`):**
```kotlin
    override fun getActiveWordSpan(sentenceIndex: Int, wordIndex: Int): StoryPlaybackCoordinator.WordSpan? {
        if (sentenceIndex < 0 || wordIndex < 0) return null
        val sentenceWords = wordsBySentence.getOrNull(sentenceIndex) ?: return null
        return sentenceWords.getOrNull(wordIndex)
    }
```

### Change 7.8 — Import TtsTimingGranularity (already imported at line 3, verify)

Line 3 already has:
```kotlin
import com.techmo.yawnly.core.feature.tts.data.TtsTimingGranularity
```
No change needed.

---

## File 8: `StoryReaderViewModel.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/presentation/viewmodel/StoryReaderViewModel.kt`

### Change 8.1 — Modify observePlaybackPosition() to use word-level spans when available (~lines 587–630)

**Current (lines 587–629):**
```kotlin
    private fun observePlaybackPosition() {
        suspendCall {
            ttsEngine.positionMs.collect { position ->
                // Drive Determinate/Indeterminate off the engine's real duration
                // (0L until the streaming player reports actual length). VM's
                // ttsDurationMs may hold a word-count estimate used for skip
                // bounds — don't leak it into UI numbers.
                val engineDuration = ttsEngine.durationMs.value
                val paragraphTimingActive =
                    ttsEngine.timingMetadata.value?.granularity == TtsTimingGranularity.PARAGRAPH
                val timedParagraphIndex = if (paragraphTimingActive) {
                    coordinator.activeParagraphIndexAtElapsed(position)
                } else {
                    null
                }
                val sentenceIndex = if (paragraphTimingActive) -1 else coordinator.indexOfSentenceAtElapsed(position)
                val activeSpan = if (paragraphTimingActive) null else coordinator.getActiveSpan(sentenceIndex)
                val progress = if (engineDuration > 0L) {
                    PlaybackProgress.Determinate(position, engineDuration)
                } else {
                    PlaybackProgress.Indeterminate(position)
                }
                val canSeek = ttsEngine.canSeek()
                updateState { current ->
                    val activeParagraphIndex = when {
                        timedParagraphIndex != null -> timedParagraphIndex
                        paragraphTimingActive -> current.playback.activeParagraphIndex
                        else -> activeSpan?.paragraphIndex ?: -1
                    }
                    current.copy(
                        playback = current.playback.copy(
                            ttsElapsedMs = position,
                            canSeek = canSeek,
                            currentSentenceIndex = sentenceIndex,
                            activeParagraphIndex = activeParagraphIndex,
                            activeSentenceStart = if (paragraphTimingActive) -1 else activeSpan?.start ?: -1,
                            activeSentenceEnd = if (paragraphTimingActive) -1 else activeSpan?.endExclusive ?: -1,
                            playbackProgress = progress,
                        )
                    )
                }
            }
        }
    }
```

**Replace with:**
```kotlin
    private fun observePlaybackPosition() {
        suspendCall {
            ttsEngine.positionMs.collect { position ->
                val engineDuration = ttsEngine.durationMs.value
                val paragraphTimingActive =
                    ttsEngine.timingMetadata.value?.granularity == TtsTimingGranularity.PARAGRAPH
                val timedParagraphIndex = if (paragraphTimingActive) {
                    coordinator.activeParagraphIndexAtElapsed(position)
                } else {
                    null
                }
                val sentenceIndex = if (paragraphTimingActive) -1 else coordinator.indexOfSentenceAtElapsed(position)
                val activeSpan = if (paragraphTimingActive) null else coordinator.getActiveSpan(sentenceIndex)

                // Word-level highlighting: prefer word spans when available.
                // Falls back to sentence-level spans when word boundaries are absent
                // (Chirp, cached Edge without word metadata, WPM-estimate mode).
                val wordIndex = if (!paragraphTimingActive) coordinator.indexOfWordAtElapsed(position) else -1
                val wordSpan = if (wordIndex >= 0 && sentenceIndex >= 0) {
                    coordinator.getActiveWordSpan(sentenceIndex, wordIndex)
                } else null

                val highlightStart: Int
                val highlightEnd: Int
                if (wordSpan != null) {
                    highlightStart = wordSpan.start
                    highlightEnd = wordSpan.endExclusive
                } else if (!paragraphTimingActive && activeSpan != null) {
                    highlightStart = activeSpan.start
                    highlightEnd = activeSpan.endExclusive
                } else {
                    highlightStart = -1
                    highlightEnd = -1
                }

                val progress = if (engineDuration > 0L) {
                    PlaybackProgress.Determinate(position, engineDuration)
                } else {
                    PlaybackProgress.Indeterminate(position)
                }
                val canSeek = ttsEngine.canSeek()
                updateState { current ->
                    val activeParagraphIndex = when {
                        timedParagraphIndex != null -> timedParagraphIndex
                        paragraphTimingActive -> current.playback.activeParagraphIndex
                        else -> wordSpan?.paragraphIndex ?: activeSpan?.paragraphIndex ?: -1
                    }
                    current.copy(
                        playback = current.playback.copy(
                            ttsElapsedMs = position,
                            canSeek = canSeek,
                            currentSentenceIndex = sentenceIndex,
                            activeParagraphIndex = activeParagraphIndex,
                            activeSentenceStart = highlightStart,
                            activeSentenceEnd = highlightEnd,
                            playbackProgress = progress,
                        )
                    )
                }
            }
        }
    }
```

### Change 8.2 — Apply same word-level logic in performSkip() (~lines 486–506)

**Current (lines 486–506):**
```kotlin
        updateState {
            val activeParagraphIndex = when {
                timedParagraphIndex != null -> timedParagraphIndex
                paragraphTimingActive -> it.playback.activeParagraphIndex
                else -> result.newActiveSpan?.paragraphIndex ?: -1
            }
            it.copy(
                playback = it.playback.copy(
                    ttsElapsedMs = result.newElapsedMs,
                    currentSentenceIndex = if (paragraphTimingActive) -1 else result.newSentenceIndex,
                    activeParagraphIndex = activeParagraphIndex,
                    activeSentenceStart = if (paragraphTimingActive) -1 else result.newActiveSpan?.start ?: -1,
                    activeSentenceEnd = if (paragraphTimingActive) -1 else result.newActiveSpan?.endExclusive ?: -1,
                    scrollToTopSignal = if (!paragraphTimingActive && result.newSentenceIndex <= 0) {
                        it.playback.scrollToTopSignal + 1
                    } else {
                        it.playback.scrollToTopSignal
                    },
                )
            )
        }
```

**Replace with:**
```kotlin
        updateState {
            // After skip, try word-level spans first.
            val wordIndexAfterSkip = if (!paragraphTimingActive) {
                coordinator.indexOfWordAtElapsed(result.newElapsedMs)
            } else -1
            val wordSpanAfterSkip = if (wordIndexAfterSkip >= 0 && result.newSentenceIndex >= 0) {
                coordinator.getActiveWordSpan(result.newSentenceIndex, wordIndexAfterSkip)
            } else null

            val highlightStart: Int
            val highlightEnd: Int
            if (wordSpanAfterSkip != null) {
                highlightStart = wordSpanAfterSkip.start
                highlightEnd = wordSpanAfterSkip.endExclusive
            } else if (!paragraphTimingActive && result.newActiveSpan != null) {
                highlightStart = result.newActiveSpan.start
                highlightEnd = result.newActiveSpan.endExclusive
            } else {
                highlightStart = -1
                highlightEnd = -1
            }

            val activeParagraphIndex = when {
                timedParagraphIndex != null -> timedParagraphIndex
                paragraphTimingActive -> it.playback.activeParagraphIndex
                else -> wordSpanAfterSkip?.paragraphIndex ?: result.newActiveSpan?.paragraphIndex ?: -1
            }
            it.copy(
                playback = it.playback.copy(
                    ttsElapsedMs = result.newElapsedMs,
                    currentSentenceIndex = if (paragraphTimingActive) -1 else result.newSentenceIndex,
                    activeParagraphIndex = activeParagraphIndex,
                    activeSentenceStart = highlightStart,
                    activeSentenceEnd = highlightEnd,
                    scrollToTopSignal = if (!paragraphTimingActive && result.newSentenceIndex <= 0) {
                        it.playback.scrollToTopSignal + 1
                    } else {
                        it.playback.scrollToTopSignal
                    },
                )
            )
        }
```

---

## File 9: `StoryTextCard.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/presentation/ui/StoryTextCard.kt`

### No changes needed.

`HighlightableParagraph` (lines 216–280) already accepts `activeStart: Int` and `activeEnd: Int` character offsets and renders a `SpanStyle` background on that range. Whether these offsets span a whole sentence or a single word is transparent to the UI. The `StoryReaderViewModel` now feeds word-level character ranges when available and sentence-level ranges otherwise. The same `activeSentenceStart`/`activeSentenceEnd` fields in `StoryPlaybackState` carry the values.

---

## File 10: `StoryReaderScreenState.kt`

**Path:** `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature/storyreader/presentation/state/StoryReaderScreenState.kt`

### No changes needed.

`StoryPlaybackState.activeSentenceStart` and `activeSentenceEnd` (lines 39–40) already carry `Int` character offset ranges. Their semantics change from "sentence range" to "word or sentence range" transparently. No new fields required.

---

# Diagram: Data Flow

```
Edge TTS WebSocket
  │
  ├─ audio.metadata frame with WordBoundary events
  │
  ▼
EdgeTtsClient.parseWordBoundaryHints()
  │ WordBoundaryHint(endMs, text, durationMs)
  ▼
onWordBoundary callback
  │
  ▼
TtsEngine._wordBoundaries.add(TtsWordBoundary(endMs, text, durationMs))
  │
  ├──▶ _wordBoundaries StateFlow ──▶ StoryPlaybackCoordinatorImpl.indexOfWordAtElapsed()
  │                                       │
  │                                       ▼
  │                                   wordSpan = getActiveWordSpan(sIdx, wIdx)
  │                                       │
  │                                       ▼
  └──▶ (after stream) ──▶ TtsTimingMetadata(sentenceBoundaries, wordBoundaries)
         │                        │
         ▼                        ▼
    cacheManager.put()    _timingMetadata StateFlow
         │
         ▼
    Future cache hits include word boundaries
```

---

# Regression Safety Checklist

| Scenario | Expected Behavior | Verified By |
|----------|------------------|-------------|
| Edge TTS with word boundaries | Word-level highlights appear; sentence boundaries still parsed | Manual test |
| Edge TTS without word boundaries (server doesn't send them) | Falls back to sentence-level highlights; `indexOfWordAtElapsed` returns -1 | `parseWordBoundaryHints` returns empty list → coordinator returns -1 |
| Chirp/Kokoros TTS path | `PARAGRAPH` granularity path unchanged; word boundaries never populated | `_wordBoundaries` not set in Chirp path |
| Cached Edge playback (old cache, schema=1) | No word boundaries in cached metadata → sentence-level fallback | `schemaVersion >= 2` check in `indexOfWordAtElapsed` |
| Cached Edge playback (new cache, schema=2) | Word boundaries restored from cache → word-level highlighting | `getTimingMetadata()` returns metadata with wordBoundaries |
| WPM fallback (no boundaries at all) | `sentenceCumulativeMs` used; word boundaries empty → sentence-level | `boundaries.isEmpty()` check |
| Skip forward/backward | Word-level spans computed for new position, fallback to sentence-level | `performSkip()` uses `indexOfWordAtElapsed` |
| Speed change mid-playback | Word boundaries unchanged (audio-relative timing); highlighting adjusts | No timing recomputation needed |
| Story restart / stop | Both `_sentenceBoundaries` and `_wordBoundaries` cleared | `stop()` clears both |

---

# Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Edge TTS word-boundary text mismatches original text (SSML/stripping) | Medium | Low | Word boundary *text* is never used for position mapping. Word *character spans* are computed from the original paragraph text. The word index from Edge is used only for temporal lookup. |
| Word boundary timestamps drift from audio playback | Low | Medium | Edge TTS word boundaries are in audio ticks (100ns units), same as sentence boundaries. Same conversion math. If drift exists, it would affect sentence boundaries too — already a known limitation. |
| Memory pressure from large word boundary lists | Low | Low | A 10-minute bedtime story at ~150 WPM is ~1500 words. Each `TtsWordBoundary` is ~60 bytes → ~90 KB. `TtsTimingWordBoundary` in cache is similar. Negligible. |
| Edge TTS changes WordBoundary JSON shape | Low | High | The `parseWordBoundaryHints` parser uses the same defensive `?.` pattern as the existing sentence boundary parser. `catch (_: Exception)` returns empty list. Worst case: word boundaries silently unavailable, sentence highlighting continues. |

---

# Suggested Wiki Updates

After implementation and manual verification, propose:
- `inbox/hermes-proposals/yawnly-word-boundary-report-20260505.md` — implementation report
