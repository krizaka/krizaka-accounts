# krizaka-accounts — Scope (agent-neutral)

> accounts.krizaka.com: Krizaka's own accounts for the krizaka.com properties (ADR-001 in krizaka/.github). The
> identity is **krizaka-users** (embedded: `krizaka-users-core`); this repository adds only the thin Krizaka layer.

## Rules of this repository

- **Identity changes go to krizaka-users**, never here: a provider, a sign-in rule, a token, linking. This repository
  holds the community (comments, saved posts, reports, reply e-mails) and the host's wiring.
- **Krizaka accounts are not product accounts** (House of Brands): no product calls this service for its users.
- **One database, one small pool** (`krizaka_accounts`, `pool-accounts` = 2): the cluster has 22 usable connections and
  they are all assigned. Schema: krizaka-users' `10-identity.sql`, then `infra/initdb/20-community.sql`.
- **Business values are rows of `community.settings`**, bounded in `CommunitySettings`; deployment values are
  environment variables named by bounded context (`DATABASE_`, `AUTH_`, `MAIL_`, `ABUSE_`, `APP_`).
- **Privacy (Loi 25)**: no IP address stored, no tracking; export and deletion stay complete when a table is added
  (add it to `AccountDataService` and to the context test).
- **Cost**: one `apps-s-1vcpu-0.5gb` instance. Anything more needs Oussama's written approval.

## Tests

- No test, not done; a fix starts with a failing test. Unit tests for the rules (`CommentServiceTest` on an in-memory
  store), `AccountsContextIT` (Testcontainers, `AbstractContainerIntegrationTest`) for the whole HTTP chain on the real
  schemas, `AccountsGovernanceTest` for the org's shared ArchUnit rules.
- `./mvnw verify` green before a push; CI must be green before merge (squash only).
