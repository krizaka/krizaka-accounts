<!-- krizaka-header -->
<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/krizaka/.github/main/profile/assets/krizaka-dark.svg">
  <img src="https://raw.githubusercontent.com/krizaka/.github/main/profile/assets/krizaka-light.svg" alt="Krizaka" width="96">
</picture>

# Krizaka Accounts

**One Krizaka account for the krizaka.com properties: sign in, comment, keep what you read.**

[![CI](https://github.com/krizaka/krizaka-accounts/actions/workflows/ci.yml/badge.svg)](https://github.com/krizaka/krizaka-accounts/actions/workflows/ci.yml)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

[Website](https://www.krizaka.com) · [Blog](https://blog.krizaka.com) · [Krizaka on GitHub](https://github.com/krizaka)

</div>
<!-- /krizaka-header -->

`accounts.krizaka.com` — [krizaka-users](https://github.com/krizaka/krizaka-users) (sign-in with GitHub, Google,
Apple, Facebook; sessions; roles; export and deletion) plus the thin Krizaka layer: the **community** of the
krizaka.com properties (comments with moderation, saved posts, reply e-mails). Decided in
[ADR-001](https://github.com/krizaka/.github/blob/main/docs/adr/ADR-001-krizaka-accounts.md). A Krizaka account is
not a product account: Orazaka, Yomizaka… keep their own users (House of Brands).

**Status: built and tested, not deployed.** The service needs one App Platform instance ($5/month); it waits on
Oussama's approval (ADR-001, *Cost*). The database `krizaka_accounts` and its pool already exist.

## What it serves

| | Endpoint | Who |
|:--|:--|:--|
| Sign in (browser) | `GET /api/v1/auth/web/{github,google,apple,facebook}/start?return_to=` → provider → `/callback` | anyone |
| Who am I · providers · sign out | `GET /api/v1/auth/web/session` · `GET …/providers` · `POST …/logout` | the allowed sites |
| Read a thread | `GET /api/v1/community/{site}/threads/{post}/comments` | anyone |
| Comment · edit · delete · report | `POST …/comments` · `PUT/DELETE /api/v1/community/comments/{id}` · `POST …/{id}/report` | signed in |
| Moderation queue · decide | `GET /api/v1/community/moderation?status=PENDING\|HIDDEN` · `POST …/moderation/{id}/{approve\|reject\|hide\|restore}` | `ROLE_ADMIN` |
| Saved posts | `GET/PUT /api/v1/me/saved?site=blog` · `DELETE /api/v1/me/saved/{site}/{slug}` | signed in |
| Reply e-mails · mute a thread | `GET/PUT /api/v1/me/notifications` · `POST /api/v1/me/notifications/mute` | signed in |
| Export · delete the account (Loi 25) | `GET /api/v1/me/export` · `DELETE /api/v1/me` `{"confirm":"DELETE"}` | signed in |
| One-click unsubscribe | `GET/POST /api/v1/community/unsubscribe?token=` | the e-mail's link |

The session is an HttpOnly, Secure, SameSite=Lax cookie on accounts.krizaka.com; every state-changing request that
carries it must come from an allowed origin. An account's first comment waits for a moderator; the parameters
(hold first, rate limit, length, edit window, report threshold) are rows of `community.settings`.

## Configuration

| Variable | Default | |
|:--|:--|:--|
| `DATABASE_URL` / `DATABASE_USERNAME` / `DATABASE_PASSWORD` | — / `krizaka_accounts` / — | `krizaka_accounts` through `pool-accounts` (`prepareThreshold=0`) |
| `DATABASE_POOL_SIZE` | `2` | client connections (PgBouncer holds 2 server connections) |
| `IDENTITY_JWT_SECRET` · `IDENTITY_JWT_TTL` | — · `P30D` | session signing, lifetime |
| `AUTH_PUBLIC_URL` · `AUTH_ALLOWED_ORIGINS` | `https://accounts.krizaka.com` · blog, www | callback base, the sites |
| `AUTH_{GITHUB,GOOGLE,FACEBOOK}_CLIENT_ID/_SECRET`, `AUTH_APPLE_{CLIENT_ID,TEAM_ID,KEY_ID,PRIVATE_KEY}` | off | a provider is on when its id is set |
| `AUTH_SIGNUP_ENABLED` | `true` | `false`: existing accounts only |
| `ABUSE_CAPTCHA_SECRET_KEY` | off | Turnstile on every comment |
| `MAIL_API_KEY` · `MAIL_DOMAIN` · `MAIL_API_URL` · `MAIL_FROM` | off · `mg.krizaka.com` | reply e-mails (Mailgun) |
| `APP_BLOG_URL` | `https://blog.krizaka.com` | links in e-mails |

Schema: krizaka-users' `infra/initdb/10-identity.sql`, then [`infra/initdb/20-community.sql`](infra/initdb/20-community.sql).

## Build

```bash
./mvnw verify   # unit tests + the Testcontainers context test (Docker); krizaka-users is read from ../krizaka-users
```

Built on [krizaka-build](https://github.com/krizaka/krizaka-build), [krizaka-platform-kit](https://github.com/krizaka/krizaka-platform-kit)
and krizaka-users, from their default branches (CI and the Docker image clone them).

## License

[Apache License 2.0](LICENSE) © 2026 Krizaka

---

<sub>A Krizaka open-source project · [krizaka.com](https://www.krizaka.com) · Apache-2.0</sub>
