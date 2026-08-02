---
name: Stream live captions over Voiceitt WebSockets
description: >-
  Open a Socket.IO session and stream PCM or compressed audio for real-time
  recognition of non-standard speech, consuming partial and final results.
api: asyncapi/voiceitt-websockets-asyncapi.yml
operations:
- PublicApiController_loginUserId
- sendSetOptions
- sendStreamAudioSamples
- sendStreamCompressedAudio
- receiveRecognitionEvents
- sendDisconnectRequest
---

# Stream live captions over Voiceitt WebSockets

Endpoint: `https://web.voiceitt.com/socket.io` (Socket.IO). Docs:
https://voiceitt-si-api.readme.io/reference/getting-started-with-your-api-1.

## Steps

1. **Get a JWT** via the HTTP API (`PublicApiController_loginUserId`,
   POST `https://api2.voiceitt.com/v1/auth/login/user_id`).
2. **Connect** with Socket.IO, passing `{token, refresh_token}` in the `auth`
   option. Wait for `model_loading` → `model_ready` → `connection_ready`
   before sending audio; handle `model_missing`, `model_loading_error`,
   `critical_error` as fatal.
3. **Configure** — emit `set_options` (repeatable any time): `run_itn`,
   `run_capitalization`, `run_spoken_command_conversion`,
   `filter_profanities`, `streaming_vad_min_silence_length` (silence needed to
   close a streaming segment).
4. **Stream audio** — emit `stream_audio_samples` with raw PCM bytes plus the
   sample-type string (`int16` | `int32` | `float32` | `float64`). PCM must be
   single-channel 16,000 Hz; send ~3200-sample chunks. For compressed audio
   (mp4/ogg) emit `stream_compressed_audio` with the bytes + MIME type
   (browsers: prefer MediaRecorder — less distortion).
5. **Consume results** — `partial_recognition` events carry `text` (stable),
   `unstable_text` (may still change), `segment_id`; final `recognition`
   events carry `text`, `segment_id`, `start_time`, `end_time` (seconds from
   stream start). Render partials, replace with finals per `segment_id`.
6. **Housekeeping** — refresh the JWT with the `refresh_token` event before
   expiry; on `pre_shutdown`/`shutdown` reconnect; end with
   `disconnect_request`.

## Rules

- Errors arrive as `error` events with `message` (+ `request_id` when tied to
  a request); see errors/voiceitt-problem-types.yml.
- For one-shot pre-segmented utterances use `recognize_audio_samples` instead:
  ack is `request_received` (`request_id`), result is a `recognition` event
  whose `text` may be null if no speech was detected.
