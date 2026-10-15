# Datalater maintenance branch

Upstream: https://github.com/Sarrius/pi-multi-account

Branch: `datalater/codex-credits`, based on upstream tag `v1.22.0`. The package keeps upstream identity/version for compatibility; use `git rev-parse HEAD` and `/multi-account status`'s source fingerprint to identify the actual fork revision. No npm publication or release tag is created by this migration.

## Install from a maintained checkout

```sh
git clone --branch datalater/codex-credits https://github.com/datalater/pi-multi-account.git
cd pi-multi-account
npm ci --ignore-scripts
npm run check
npm test
npm run pack:check
pi install "$PWD"
```

Remove/replace the existing `npm:pi-multi-account@...` declaration in Pi settings first; do not load both copies. Local sources are not replaced by `pi update --extensions`. Dependencies belong to this checkout. Pi AI is declared as a host-provided peer, with a development copy for standalone tests. Extensions run with the same privileges as Pi.

Set `"allowCodexCredits": true` in the existing user `provider-failover.json` to opt in; never commit auth, state, live logs, or balances. Completely quit and resume Pi to load changed code. The README documents credit permission and real-refusal safeguards.

## Updating

1. Start from a clean checkout and keep the last working commit ID.
2. `git fetch upstream --tags` (add `upstream` pointing to the URL above if needed).
3. Merge a reviewed upstream tag/commit into a separate upgrade branch based on this branch. Do not overwrite fork commits with a forced sync.
4. Run the gates above, especially the Codex credit policy/preflight cases in `test/failover.test.ts`. Tests use synthetic credentials and do not spend credits.
5. Review OAuth refresh/proxy identity, response headers versus usage-body parsing, actual refusal handling, persisted state, Pi host API compatibility, and dependency security updates.
6. Push only verified changes, advance the maintenance branch, then restart Pi. Do not automatically publish under the upstream npm name or run its release workflow.

For rollback, checkout the last known-good commit, reinstall its locked dependencies, and restart Pi; preserve the user configuration and auth/state files. Do not run a blanket cooldown reset as part of an upgrade.

## What maintenance actually involves

A new model release does **not** normally require adding its ID to this extension. The recurring work is maintaining personal selection preferences and adapting to provider/host contract changes. Keep those responsibilities separate.

### Automatic discovery versus manual configuration

With `autoDiscoverModels` enabled (the default), the account-specific Codex catalog supplies model IDs, display names, reasoning levels, input modalities, context/output limits and server priorities. The current implementation caches that catalog for five minutes and refreshes it at lifecycle/metadata-refresh boundaries. Pi's registry and the extension's static definitions are offline fallbacks, not the primary live catalog. Availability still depends on the account and successful catalog retrieval.

| Item | Who owns it? | When to act |
| --- | --- | --- |
| Newly available Codex model IDs | Live account catalog, then Pi registry/static fallback | Normally check discovery rather than edit code; `/multi-account rediscover` explicitly refreshes discovery. |
| `defaultProvider` / `defaultModel` | Pi user/project settings and explicit session choices | Change only when you want a new default. Discovery does not imply user consent to change defaults. |
| `enabledModels` | Pi settings | Add an ID/pattern if you want it in startup selection/model cycling. This is not a universal ban on other models in `/model`. |
| `modelThinkingLevels` | Pi settings | Add or revise a per-model default when your preference changes; the model still must support the requested level. |
| `preferredModels` | `provider-failover.json` | Review any explicit order when introducing or retiring models. An old preference may remain intentional. |
| Custom `models.json` entries | The user or custom provider owner | Maintain endpoint/API compatibility and manually overridden capability/context/cost information that is not supplied by discovery. Do not hand-edit proxy-generated credential/publication fields. |
| New model families and fallback quality bands | `index.ts` (`modelQualityBand`) and `model-catalog.ts` | New suffixes/product tiers can require code and tests even if discovery already lists the model. Existing naming patterns can continue to work automatically. |
| Offline fallback definitions | `index.ts` model tables | Update when needed for offline/catalog-failure behavior, not as the normal procedure for every new Codex release. |

**Discovery and safe routing are different contracts.** A model can appear in a catalog yet remain unclassified for automatic cross-model fallback. Do not guess that an unfamiliar model is equivalent to a known tier. Other families have different discovery strategies; Codex's account catalog is not a guarantee of automatic discovery for every provider.

### Recurring work categories

| Trigger | Maintenance work | Configuration only? |
| --- | --- | --- |
| New models or changed personal workflow | Review defaults, cycling scope, thinking preferences and routing preferences | Usually |
| New product tier, renamed capabilities or catalog schema | Review quality-band matching, reasoning-level mapping, context limits and catalog parsing | Sometimes code |
| Pi/SDK upgrade | Check OAuth/provider interfaces, transcript/tool shapes, lifecycle events and proxy publication | Often code/dependencies |
| Provider authentication change | Check token refresh, cross-process leases, sidecar identity and parent/child routing | Code; never solved by blindly clearing auth/state |
| Usage endpoint/header change | Reconcile full-body versus partial-header data, credit availability and actual refusal evidence | Code/tests |
| New HTTP/stream failure behavior | Review retry classification and cooldown/continuation semantics | Code/tests |
| Dependency/security advisory | Review locked versions, runtime versus development dependencies and tested host versions | Dependency/toolchain update |
| Upstream merge | Preserve our credit opt-in and independent footer visibility; rebuild and run regression gates | Merge/test work |

### What upstream was changing — observed 2026-10-15

This is a dated investigation snapshot, not a statement that these versions remain latest:

- Our fork baseline is `v1.22.0`. The latest published release observed was [`v1.24.0`](https://github.com/Sarrius/pi-multi-account/releases/tag/v1.24.0); upstream main described `1.25.0` as unreleased.
- September updates repaired numbered Codex loopback routing, managed-install OAuth bridge resolution, opaque-token expiry refresh and multi-session proxy ownership.
- [`v1.24.0`](https://github.com/Sarrius/pi-multi-account/blob/v1.24.0/CHANGELOG.md) added model pins, Pi 1 host compatibility, numbered Anthropic credential shaping, Codex image-route normalization and dependency updates. Model pins are **not** available merely because this document mentions them: they have not been merged into our baseline.
- [Upstream main at `f5819b3`](https://github.com/Sarrius/pi-multi-account/blob/f5819b3/CHANGELOG.md) included native OpenAI ChatGPT OAuth, persistent actual-refusal evidence, cross-process refresh/lease fixes, a new model-tier classification, billing-header version updates and a development-dependency security remediation.

The first upgrade review should therefore prioritize **authentication, Pi compatibility and refusal/cooldown fixes**, rather than manually refreshing model name lists. Reviewing an unreleased main change is not approval to deploy all of main. Our credit exception must be reconciled carefully with upstream's newer refusal-evidence rules.

### Trigger-based review routine

- **New Codex model:** confirm account discovery, then update personal preferences if desired; inspect quality-band handling only when the naming/tier is new. Use `pi update --models` for Pi's own catalog and `/multi-account rediscover` for this extension's account discovery; neither command replaces user preferences.
- **Before upgrading Pi:** check upstream compatibility notes and run the gates above on the intended host version.
- **Login or usage starts failing:** inspect the exact error and relevant upstream fixes first. A missing usage field is not proof of exhausted credit or invalid credentials.
- **Periodically (for example every two weeks):** review releases and security/auth changes. Do not merge every feature or dependency bump automatically.
- **Before deployment:** verify credit-positive/zero-quota routing across two consecutive messages, actual refusal failover, account identity isolation, and `showUsageStatusline: false` with polling still enabled. The pi-usage fork owns the visible footer.

Read-only upstream review commands:

```sh
git fetch upstream --tags
git log --oneline HEAD..upstream/main
git diff HEAD...upstream/main -- index.ts model-catalog.ts usage.ts slot-proxy.ts slot-proxy-auth.ts package.json package-lock.json
```

The three-dot diff shows upstream-side changes since the common ancestor. Review our own changes separately before merging; fetching alone does not modify the deployed checkout.

## Migration verification (historical)

- `npm run check` and `npm run pack:check` passed; full `npm test`: 616 passed.
- New native event-flow tests cover base and numbered Codex accounts, first and second messages across response-header usage updates, opt-out, depleted/unknown credits, stale same-account balance, mismatched identity, pre-existing cooldown, and real refusal failover. Disabling the credit-readiness branch makes the regression tests fail.
- The upstream hanging-compaction fixture also cancelled on a pristine checkout because its bare Promise did not keep the event loop alive for the unref'ed production watchdog. The fork fixes only that fixture's pending-I/O lifetime, released on abort; production timeout policy is unchanged.
- A fresh installed-Pi RPC process registered this fork and the built pi-usage fork in an isolated agent directory. No paid inference was sent as part of migration.
