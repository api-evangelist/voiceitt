---
name: Transcribe an audio file with Voiceitt
description: >-
  Authenticate a Voiceitt app and transcribe an audio file of non-standard
  speech using the JWT-authenticated HTTP API.
api: openapi/voiceitt-rest-api-openapi-original.json
operations:
- PublicApiController_loginUserId
- PublicApiController_transcribe
- PublicApiController_refreshToken
---

# Transcribe an audio file with Voiceitt

Base URL: `https://api2.voiceitt.com` (docs: https://voiceitt-si-api.readme.io/).

## Steps

1. **Authenticate** — `PublicApiController_loginUserId`: POST `/v1/auth/login/user_id`
   with JSON `{app_id, api_key}` (both issued in the Voiceitt developer portal,
   https://developer.voiceitt.com/). Optionally include `user_id` (max 36 chars)
   to bind the session to a specific end user. For personalized models use
   `PublicApiController_loginEmail` (POST `/v1/auth/login/email` with
   `{app_id, api_key, email, password}`) — the user must first enroll and train
   at https://web.voiceitt.com/. The response carries `token`,
   `token_expires_at`, `refresh_token`, `refresh_token_expires_at`
   (epoch milliseconds).
2. **Transcribe** — `PublicApiController_transcribe`: POST `/v1/rec/transcribe`
   as `multipart/form-data` with `Authorization: Bearer <token>`. Required
   parts: `audio` (the audio file) and `options` (a JSON file), e.g.
   `{"run_itn": true, "run_capitalization": true,
   "run_spoken_command_conversion": false, "filter_profanities": true,
   "save_audio": false}`. The response is `{"text": "..."}`.
3. **Refresh before expiry** — `PublicApiController_refreshToken`: POST
   `/v1/auth/refresh_token` with `{refresh_token}` when `token_expires_at`
   approaches; replaces both tokens.

## Rules

- Auth failures return `403` with plain descriptions ("Invalid App ID or API
  key") — there is no problem+json envelope (see errors/voiceitt-problem-types.yml).
- No idempotency-key mechanism exists; retry a failed transcribe by
  resubmitting (see conventions/voiceitt-conventions.yml).
- No rate limits are documented; back off on repeated 4xx/5xx.
