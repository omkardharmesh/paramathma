---
title: Kokoros Quantized Model Handoff
summary: Handoff note for the 2026-05-03 attempt to run Kokoros with the community quantized ONNX model.
type: project-note
status: draft
updated: 2026-05-03
---

# Kokoros Quantized Model Handoff — 2026-05-03

## Goal

Make Kokoros usable for Yawnly by reducing model size and improving response time with the community quantized Kokoro ONNX model.

The target model is the ONNX Community Kokoro v1.0 quantized model:

- Hugging Face repo: `onnx-community/Kokoro-82M-v1.0-ONNX`
- Target file: `onnx/model_quantized.onnx`
- Expected size: about 92 MB

## Important User Preference

Do local work first.

- Do not use the VPS for build/test/debug unless the user explicitly says deploy.
- Do not copy files from the VPS when the same artifact can be downloaded directly.
- The VPS should only be touched after the patched image is locally built and locally smoke-tested.

## What Was Diagnosed

The current upstream Kokoros Docker image works with the full model but fails with the community quantized model.

Observed production behavior with `model_quantized.onnx` mounted over `/app/checkpoints/kokoro-v1.0.onnx`:

- Kokoros starts successfully.
- ONNX Runtime loads the model.
- Streaming output returns zero bytes.
- MP3/non-streaming requests can return `500`.

The root cause is a tensor schema mismatch.

The community quantized model uses:

- input tensor: `input_ids`
- input tensor: `style`
- input tensor: `speed`
- output tensor: `waveform`

The Kokoros Rust code path for one-output models assumed the older schema:

- input tensor: `tokens`
- output tensor: `audio` or fallback `waveforms`

Because the quantized model has one output, Kokoros classified it as `Standard`, then sent the wrong input tensor name and looked for the wrong output tensor name.

## Community Check

Web search confirmed the community ONNX model expects `input_ids`, `style`, and `speed`, with waveform audio as the model output. This matches the local/VPS ONNX inspection and supports the patch direction.

Related references found:

- `onnx-community/Kokoro-82M-v1.0-ONNX` Hugging Face README.
- `onnx-community/Kokoro-82M-ONNX` model card / commits.
- `tts-rs` has Kokoro ONNX support, but switching libraries is a larger change and was not pursued.

## Local Code Patch

Patched file:

- `/Users/eloelo/Downloads/Kokoros/kokoros/src/onn/ort_koko.rs`

Patch intent:

- Add a `StandardWaveform` strategy for one-output models that use `input_ids -> waveform`.
- Detect actual model schema from `Session::inputs()` and `Session::outputs()`.
- Treat models with `durations` output as timestamped.
- Treat models with `input_ids` input and `waveform` output as standard waveform models.
- Keep legacy `tokens -> audio` support for the original model.
- In standard inference, accept `waveform`, `waveforms`, or `audio` as the audio tensor.

High-level diff:

```text
ModelStrategy:
  Standard(Session)
  StandardWaveform(Session)
  Timestamped(Session)

Detection:
  has_input_ids = input name == "input_ids"
  has_waveform = output name == "waveform"
  has_durations = output name == "durations"

Selection:
  if has_durations -> Timestamped
  else if has_input_ids && has_waveform -> StandardWaveform
  else -> Standard
```

## Build Results

Local macOS `cargo build --release` / `cargo check -p kokoros` did not validate the patch because native dependencies failed before reaching meaningful Rust validation:

- `audiopus_sys` failed during CMake/pkg-config setup.
- `espeak-rs-sys` also failed in the local macOS build path.

This was a local native toolchain issue, not evidence that the patch failed.

Local Docker was then used and succeeded:

```shell
cd /Users/eloelo/Downloads/Kokoros
docker build -t kokoros:quantized-schema-fix .
```

Result:

- Image built successfully.
- Image tag: `kokoros:quantized-schema-fix`
- Image ID observed: `b0ce9a0151f1`
- Image size observed: about `1.4GB`
- Rust release build completed inside Docker.

## Local Runtime Smoke Test Results

Local quantized runtime smoke test completed successfully after downloading the community quantized model directly from Hugging Face.

Downloaded local model:

```shell
cd /Users/eloelo/Downloads/Kokoros/checkpoints
curl -L --fail -o model_quantized.onnx \
  "https://huggingface.co/onnx-community/Kokoro-82M-v1.0-ONNX/resolve/main/onnx/model_quantized.onnx"
```

Observed local model size:

```text
model_quantized.onnx  88.1M
```

Started local patched container:

```shell
docker rm -f kokoros-quantized-test 2>/dev/null || true

docker run -d --name kokoros-quantized-test \
  -p 127.0.0.1:3001:3000 \
  -v /Users/eloelo/Downloads/Kokoros/checkpoints/model_quantized.onnx:/app/checkpoints/kokoro-v1.0.onnx:ro \
  -e RUST_LOG=info \
  kokoros:quantized-schema-fix \
  --instances 1 openai
```

Container logs confirmed the intended patched schema path:

```text
OrtKoko: Standard waveform backend activated (input_ids -> waveform, 1 output)
```

MP3 smoke test succeeded:

```text
HTTP 200 total=1.275394s
/tmp/kokoros-quantized-test.mp3: Audio file with ID3 version 2.3.0, contains:
- MPEG ADTS, layer III, v2, 160 kbps, 24 kHz, Monaural
/tmp/kokoros-quantized-test.mp3  59.7K
```

Streaming PCM smoke test succeeded:

```text
HTTP 200 starttransfer=0.018060s total=1.504066s size=168000
/tmp/kokoros-quantized-stream.pcm: data
/tmp/kokoros-quantized-stream.pcm  164.1K
```

Container logs for streaming:

```text
TTS session started - 2 chunks streaming
TTS session completed - 2 chunks, 168000 bytes, 3.5s audio, PCM format
```

## What Not To Do Next

Do not continue the earlier VPS build/deploy path unless explicitly approved.

The user objected to:

- Building on the VPS before testing locally.
- Copying the quantized model from the VPS.
- Touching the VPS while local Docker is available.

Honor that.

## Immediate Next Steps

1. Confirm image architecture before deployment.
2. If the local image is `arm64`, build an amd64 image for Hetzner.
3. Deploy only after explicit user approval.
4. Benchmark the same story-sized text locally and then on VPS after deploy.

## Deployment Result

Deployment completed 2026-05-03 after local validation.

VPS cleanup before deployment:

- Removed temporary `/opt/kokoros-src`.
- Pruned Docker build cache.
- Removed the accidental VPS-built image tag `kokoros:quantized-schema-fix`.
- Left the live `/opt/kokoros` deployment intact until the amd64 local image was ready.

Because the first local image was `arm64`, an amd64 image was built locally for Hetzner:

```shell
cd /Users/eloelo/Downloads/Kokoros
docker buildx build --platform linux/amd64 \
  -t kokoros:quantized-schema-fix-amd64 \
  --load .
```

Verified local image:

```text
architecture: amd64
image id: sha256:f12ffd54a67b6b746533afe43b2689a8c61252876ea340bd8230c81707f17fc5
size: 579MB
```

The image was exported, uploaded, loaded on the VPS, then the archive was deleted after deployment.

The active VPS compose now uses:

```yaml
image: kokoros:quantized-schema-fix-amd64
```

and retains the quantized model mount:

```yaml
volumes:
  - /opt/kokoros/models/model_quantized.onnx:/app/checkpoints/kokoro-v1.0.onnx:ro
```

VPS startup logs confirmed both instances use the patched schema path:

```text
OrtKoko: Standard waveform backend activated (input_ids -> waveform, 1 output)
Starting TTS server with 2 instances
Starting OpenAI-compatible HTTP server on 0.0.0.0:3000
```

VPS local health check passed:

```text
GET http://127.0.0.1:3000/ -> HTTP 200 OK
GET http://127.0.0.1:3000/v1/models -> model list returned
```

VPS local MP3 generation passed:

```text
HTTP 200 total=6.942337s size=73156
/tmp/kokoros-quantized-vps.mp3: Audio file with ID3 version 2.3.0, contains:
- MPEG ADTS, layer III, v2, 160 kbps, 24 kHz, Monaural
```

VPS local PCM streaming passed:

```text
HTTP 200 starttransfer=0.131747s total=7.293287s size=171600
/tmp/kokoros-quantized-vps-stream.pcm: data
TTS session completed - 2 chunks, 171600 bytes, 3.6s audio, PCM format
```

Public Cloudflare route was not re-tested in this pass because the raw WAF header secret must not be stored in wiki or chat. The service is healthy behind Caddy locally on the VPS; public testing should be done with the secret available from the secure operator environment or via Supabase Edge Function.

## Previous Deployment Plan

This section is retained as historical reference; the deployment above has already been performed.

## Deployment Plan After Local Success

Only after local smoke tests pass:

1. Export the local Docker image.

```shell
docker save kokoros:quantized-schema-fix | gzip > /tmp/kokoros-quantized-schema-fix.tar.gz
```

2. Upload the image to the VPS.

```shell
scp /tmp/kokoros-quantized-schema-fix.tar.gz kokoros-hetzner:/opt/kokoros/
```

3. On VPS, load the image.

```shell
ssh kokoros-hetzner
cd /opt/kokoros
gunzip -c kokoros-quantized-schema-fix.tar.gz | docker load
```

4. Update `/opt/kokoros/docker-compose.yml` to use:

```yaml
image: kokoros:quantized-schema-fix
```

and keep the quantized model mount:

```yaml
volumes:
  - /opt/kokoros/models/model_quantized.onnx:/app/checkpoints/kokoro-v1.0.onnx:ro
```

5. Restart and verify.

```shell
cd /opt/kokoros
docker compose up -d
docker compose logs --tail=100 kokoros
curl -I http://127.0.0.1:3000/
```

6. Only then test through Cloudflare with the secret header.

Do not write the raw secret into docs, source, or chat.

## Open Risks

- The patch has passed Docker compilation and local runtime inference with the quantized model.
- Story-sized benchmark is still pending.
- The local Docker build created an ARM image on Apple Silicon. The Hetzner VPS is likely amd64. If `docker save` uploads an ARM image, it will not run on the VPS. Before deployment, check:

```shell
docker image inspect kokoros:quantized-schema-fix --format '{{.Architecture}}'
```

If it says `arm64`, build an amd64 image locally using:

```shell
docker buildx build --platform linux/amd64 -t kokoros:quantized-schema-fix-amd64 --load .
```

Use the amd64 image for the VPS.

## Current Stop Point

As of this handoff:

- Code patch exists locally.
- Local Docker image build succeeded.
- Quantized model was downloaded locally from Hugging Face.
- Local MP3 and streaming PCM smoke tests succeeded.
- Do not touch the VPS unless the user explicitly approves deployment.
