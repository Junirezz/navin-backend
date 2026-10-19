# Environment variable matrix

Single reference for every environment variable the Navin Backend reads or expects.
Cross-check source: `src/env.ts`, `.env.example`, and `process.env` / `env.*` usages under `src/`.

| Legend                 | Meaning                                                             |
| ---------------------- | ------------------------------------------------------------------- |
| **Validated**          | Present in the Zod schema in `src/env.ts`                           |
| **`.env.example`**     | Listed in `.env.example`                                            |
| **Used in code**       | Read via `env` / `config` / `process.env` outside the schema itself |
| **Required**           | Process will not start (or core flows fail) without a valid value   |
| **Implementing issue** | Tracking issue for wiring or hardening a gap                        |

---

## Core runtime

| Variable          | Validated | `.env.example` | Used in code | Required vs optional                        | Notes / implementing issue                    |
| ----------------- | :-------: | :------------: | :----------: | ------------------------------------------- | --------------------------------------------- |
| `ALLOWED_ORIGINS` |    yes    |      yes       |     yes      | Optional (default `''`)                     | Comma-separated CORS allowlist (`src/config`) |
| `CORS_ORIGIN`     |    yes    |      yes       |     yes      | Optional (default `*`)                      | Legacy single-origin CORS fallback            |
| `JWT_SECRET`      |    yes    |      yes       |     yes      | **Required**                                | Min 32 chars; auth tokens                     |
| `MONGO_URI`       |    yes    |      yes       |     yes      | **Required**                                | Mongo connection string                       |
| `NODE_ENV`        |    yes    |      yes       |     yes      | Optional (default `development`)            | `development` \| `test` \| `production`       |
| `PORT`            |    yes    |      yes       |     yes      | Optional (default `3000`)                   | HTTP listen port                              |
| `REDIS_URL`       |    yes    |      yes       |     yes      | Optional (default `redis://127.0.0.1:6379`) | BullMQ / SSE / caches                         |

## Stellar

| Variable                 | Validated | `.env.example` | Used in code | Required vs optional                                   | Notes / implementing issue                                                                                                                                                                                             |
| ------------------------ | :-------: | :------------: | :----------: | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `STELLAR_NETWORK`        |    yes    |      yes       |     yes      | Optional (default `testnet`)                           | `testnet` \| `public`; selects the default Horizon/Soroban URLs (`src/config/stellarNetwork.ts`, implementing [#360](https://github.com/Navin-xmr/navin-backend/issues/360)); an unrecognized value fails fast at boot |
| `HORIZON_URL`            |    yes    |       no       |     yes      | Optional — overrides the URL `STELLAR_NETWORK` derives | `src/services/stellar.service.ts` via `config.horizonUrl`                                                                                                                                                              |
| `STELLAR_SECRET_KEY`     |    yes    |      yes       |     yes      | Optional at boot; **required** for anchoring/signing   | Horizon/SDK workers (`src/services/stellar.service.ts`, `src/services/chain/simulated.adapter.ts`)                                                                                                                     |
| `STELLAR_WEBHOOK_SECRET` |    yes    |      yes       |     yes      | Optional at boot; **required** for webhook HMAC        | `verifyStellarSignature` middleware                                                                                                                                                                                    |

## Frontend & email

| Variable           | Validated | `.env.example` | Used in code | Required vs optional                       | Notes / implementing issue                                            |
| ------------------ | :-------: | :------------: | :----------: | ------------------------------------------ | --------------------------------------------------------------------- |
| `FRONTEND_URL`     |    yes    |      yes       |     yes      | Optional (default `http://localhost:3000`) | Invite / password-reset links                                         |
| `SENDGRID_API_KEY` |    yes    |      yes       |     yes      | Optional                                   | Prefer SendGrid SMTP relay when set (`src/services/email.service.ts`) |
| `SMTP_FROM`        |    yes    |      yes       |     yes      | Optional                                   | From-address for outbound mail                                        |
| `SMTP_HOST`        |    yes    |      yes       |     yes      | Optional                                   | Direct SMTP transport when SendGrid unset                             |
| `SMTP_PASS`        |    yes    |      yes       |     yes      | Optional                                   | SMTP password                                                         |
| `SMTP_PORT`        |    yes    |      yes       |     yes      | Optional (default `587`)                   | SMTP port                                                             |
| `SMTP_USER`        |    yes    |      yes       |     yes      | Optional                                   | SMTP username                                                         |

> **SMTP status:** Schema + transport are implemented. Delivery still depends on providing credentials in each environment. Notifications product work is [#362](https://github.com/Navin-xmr/navin-backend/issues/362) / [#363](https://github.com/Navin-xmr/navin-backend/issues/363).

## Twilio (SMS) — TODO gap

| Variable       | Validated | `.env.example` |   Used in code   | Required vs optional | Notes / implementing issue                                                                                                 |
| -------------- | :-------: | :------------: | :--------------: | -------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `TWILIO_FROM`  |    yes    |      yes       | no (config only) | Optional             | **TODO** — exposed on `config.twilio` but no SMS client yet. [#364](https://github.com/Navin-xmr/navin-backend/issues/364) |
| `TWILIO_SID`   |    yes    |      yes       | no (config only) | Optional             | **TODO** — same as above                                                                                                   |
| `TWILIO_TOKEN` |    yes    |      yes       | no (config only) | Optional             | **TODO** — same as above                                                                                                   |

## Storage

| Variable                | Validated | `.env.example` | Used in code | Required vs optional                        | Notes / implementing issue                                                                                           |
| ----------------------- | :-------: | :------------: | :----------: | ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `STORAGE_PROVIDER`      |    yes    |      yes       |     yes      | Optional (default `mock`)                   | Selects `mock`, `s3`, `r2`, or `cloudinary`; cloud-provider credentials are required when that provider is selected. |
| `S3_ACCESS_KEY`         |    yes    |      yes       |     yes      | Optional at boot; required for `s3` / `r2`  | S3-compatible access key.                                                                                            |
| `S3_BUCKET`             |    yes    |      yes       |     yes      | Optional at boot; required for `s3` / `r2`  | Bucket used by the selected S3-compatible provider.                                                                  |
| `S3_ENDPOINT`           |    yes    |      yes       |     yes      | Optional                                    | Set for custom S3-compatible endpoints such as Cloudflare R2; AWS S3 can use its default endpoint.                   |
| `S3_REGION`             |    yes    |      yes       |     yes      | Optional (default `us-east-1`)              | S3-compatible region.                                                                                                |
| `S3_SECRET_KEY`         |    yes    |      yes       |     yes      | Optional at boot; required for `s3` / `r2`  | S3-compatible secret key.                                                                                            |
| `CLOUDINARY_CLOUD_NAME` |    yes    |      yes       |     yes      | Optional at boot; required for `cloudinary` | Cloudinary account name.                                                                                             |
| `CLOUDINARY_API_KEY`    |    yes    |      yes       |     yes      | Optional at boot; required for `cloudinary` | Cloudinary API key.                                                                                                  |
| `CLOUDINARY_API_SECRET` |    yes    |      yes       |     yes      | Optional at boot; required for `cloudinary` | Cloudinary API secret.                                                                                               |

## Authentication / TOTP

| Variable              | Validated | `.env.example` | Used in code | Required vs optional                 | Notes / implementing issue                                                                                                                                                       |
| --------------------- | :-------: | :------------: | :----------: | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TOTP_ENCRYPTION_KEY` |    yes    |      yes       |     yes      | Optional (recommended in production) | Exactly 64 hex characters (32 bytes), used for AES-256-GCM encryption of TOTP secrets. If unset, the key is derived from `JWT_SECRET`; production should use an independent key. |

## Soroban / escrow

| Variable             | Validated | `.env.example` | Used in code | Required vs optional                               | Notes / implementing issue                                                                                                 |
| -------------------- | :-------: | :------------: | :----------: | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `ESCROW_CONTRACT_ID` |    yes    |      yes       |     yes      | Required when `SOROBAN_ADAPTER=soroban`            | Contract invoked by `SorobanAdapter`; validated conditionally at startup.                                                  |
| `SOROBAN_RPC_URL`    |    yes    |      yes       |     yes      | Required explicitly when `SOROBAN_ADAPTER=soroban` | Soroban RPC endpoint. Otherwise it defaults from `STELLAR_NETWORK` and can be overridden (`src/config/stellarNetwork.ts`). |
| `SOROBAN_ADAPTER`    |    yes    |      yes       |     yes      | Optional (default `simulated`)                     | Selects `simulated` or `soroban`; an unrecognized value or missing Soroban settings fails validation at startup.           |

### ChainAdapter env matrix

| `SOROBAN_ADAPTER`   | Required settings                                                 | Behaviour                                                                                                                                                                                                                                                         |
| ------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unset / `simulated` | `STELLAR_SECRET_KEY` for signing                                  | `SimulatedAdapter` uses Horizon manage-data anchors/releases; results carry `simulated: true` (no contract, no funds move). Submissions are serialized per signing account; a `tx_bad_seq` from another process is retried (up to 2 times) with a fresh sequence. |
| `soroban`           | `SOROBAN_RPC_URL`, `ESCROW_CONTRACT_ID`, and `STELLAR_SECRET_KEY` | `SorobanAdapter` submits contract calls through Soroban RPC; results carry `simulated: false`. The RPC URL and contract ID are required by environment validation.                                                                                                |
| anything else       | —                                                                 | Env validation fails; process exits at boot.                                                                                                                                                                                                                      |

> Several processes sharing one signing account are safe only up to the bounded `tx_bad_seq` retry; for sustained multi-process load give each worker process its own signing account.

## Observability

| Variable     | Validated | `.env.example` |   Used in code   | Required vs optional | Notes / implementing issue                                                        |
| ------------ | :-------: | :------------: | :--------------: | -------------------- | --------------------------------------------------------------------------------- |
| `LOG_LEVEL`  |    no     |      yes       |       yes        | Optional             | Read directly in `src/shared/logger/logger.ts` (not Zod-validated)                |
| `SENTRY_DSN` |    yes    |      yes       | no (config only) | Optional             | **TODO** — validated and on `config.sentryDsn`, but Sentry SDK is not initialized |

---

## Integration status

| Gap area        | Vars                                                       | Status                                                                                                | Implementing issue                                                              |
| --------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| SMTP / email    | `SMTP_*`, `SENDGRID_API_KEY`                               | Implemented (optional credentials)                                                                    | — / notifications [#362](https://github.com/Navin-xmr/navin-backend/issues/362) |
| Twilio SMS      | `TWILIO_*`                                                 | Schema + config only                                                                                  | [#364](https://github.com/Navin-xmr/navin-backend/issues/364)                   |
| Storage         | `STORAGE_PROVIDER`, `S3_*`, `CLOUDINARY_*`                 | Mock, S3, R2, and Cloudinary adapters are implemented; provider credentials are checked when selected | —                                                                               |
| Soroban escrow  | `SOROBAN_ADAPTER`, `SOROBAN_RPC_URL`, `ESCROW_CONTRACT_ID` | Simulated and Soroban adapters are implemented; Soroban settings are conditionally required           | —                                                                               |
| TOTP encryption | `TOTP_ENCRYPTION_KEY`                                      | Implemented; optional explicit key with a `JWT_SECRET`-derived fallback                               | —                                                                               |
| Sentry          | `SENTRY_DSN`                                               | Schema + config only                                                                                  | TBD (no dedicated issue yet)                                                    |

---

## Cross-check checklist

Every `env.*` / `process.env.*` name under `src/` should appear above:

- [x] `ALLOWED_ORIGINS`, `CORS_ORIGIN`, `JWT_SECRET`, `MONGO_URI`, `NODE_ENV`, `PORT`, `REDIS_URL`
- [x] `STELLAR_NETWORK`, `HORIZON_URL`, `STELLAR_SECRET_KEY`, `STELLAR_WEBHOOK_SECRET`
- [x] `FRONTEND_URL`, `SENDGRID_API_KEY`, `SMTP_*`
- [x] `TWILIO_*`, `STORAGE_PROVIDER`, `S3_*`, `CLOUDINARY_*`, `SOROBAN_ADAPTER`, `SOROBAN_RPC_URL`, `ESCROW_CONTRACT_ID`, `TOTP_ENCRYPTION_KEY`, `SENTRY_DSN`
- [x] `LOG_LEVEL` (unvalidated)
