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

## 9. Anti-slop (ADR-015)

What a model tends to write and the code does not need. Each line points at the rule that refuses it
([ADR-015](https://github.com/krizaka/.github/blob/main/docs/adr/ADR-015-ai-code-audit-measured-and-enforced.md), [docs/standards/guards.md](https://github.com/krizaka/.github/blob/main/docs/standards/guards.md)).

1. **No defensive null on `@NullMarked` code.** No `Objects.requireNonNull` or `if (x == null)` on a value the type already
   guarantees. Enforced by NullAway at compile time; a new package is `@NullMarked` (ArchUnit).
2. **A closed set is a type.** An enum, a sealed interface or an `as const` object, never a string compared by hand;
   strategies are a `Map<Kind, Strategy>` built once at startup. Enforced by PMD / ArchUnit (Java) and `no-enum-string-compare` (ast-grep).
3. **HTTP in Java is a DTO record, `@Valid` and `ProblemDetail`.** No `Map.of("error", …)`, no entity in a response, one
   `@RestControllerAdvice` per service. Enforced by ArchUnit.
4. **No `*Utils`, `*Helper` or `*Manager`.** Name the thing by what it does (§4). Enforced by ArchUnit (ADR-015 §5).
5. **`?.` and `??` in TypeScript**, not a chain of `&&` or a ternary on null. The target rules are `@typescript-eslint/prefer-optional-chain` and `prefer-nullish-coalescing`; they are not in `@krizaka/config` yet.
6. **The build is the judge.** Compile, lint and test before every push; `krizaka-edit-check` runs on each edit (below) and
   NullAway and ArchUnit run at pre-push and in CI. A rule is never silenced to get green.

**The edit hook.** Every repository adds this to `.claude/settings.json`. After each `Edit` or `Write`, the one file is
checked in under 10 s: ESLint with the repository's config for TypeScript, PMD with the Krizaka rules for Java (never Maven).
A finding comes back to the agent (exit 2); a timeout is silent.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "npx --yes -p @krizaka/config krizaka-edit-check", "timeout": 10 }]
      }
    ]
  }
}
```

In a TypeScript repository that already depends on `@krizaka/config`, use `npx --no-install krizaka-edit-check` (no download).
