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

Remove/replace the existing `npm:pi-multi-account@...` declaration in Pi settings first; do not load both copies. Local sources are not replaced by `pi update --extensions`. Dependencies belong to this checkout. Extensions run with the same privileges as Pi.

Set `"allowCodexCredits": true` in the existing user `provider-failover.json` to opt in; never commit auth, state, live logs, or balances. Completely quit and resume Pi to load changed code. The README documents credit permission and real-refusal safeguards.

## Updating

1. Start from a clean checkout and keep the last working commit ID.
2. `git fetch upstream --tags` (add `upstream` pointing to the URL above if needed).
3. Merge a reviewed upstream tag/commit into a separate upgrade branch based on this branch. Do not overwrite fork commits with a forced sync.
4. Run the gates above, especially the Codex credit policy/preflight cases in `test/failover.test.ts`. Tests use synthetic credentials and do not spend credits.
5. Review OAuth refresh/proxy identity, response headers versus usage-body parsing, actual refusal handling, persisted state, Pi host API compatibility, and dependency security updates.
6. Push only verified changes, advance the maintenance branch, then restart Pi. Do not automatically publish under the upstream npm name or run its release workflow.

For rollback, checkout the last known-good commit, reinstall its locked dependencies, and restart Pi; preserve the user configuration and auth/state files. Do not run a blanket cooldown reset as part of an upgrade.

## Migration verification

- `npm run check` and `npm run pack:check` passed; full `npm test`: 616 passed.
- New native event-flow tests cover base and numbered Codex accounts, first and second messages across response-header usage updates, opt-out, depleted/unknown credits, stale same-account balance, mismatched identity, pre-existing cooldown, and real refusal failover. Disabling the credit-readiness branch makes the regression tests fail.
- The upstream hanging-compaction fixture also cancelled on a pristine checkout because its bare Promise did not keep the event loop alive for the unref'ed production watchdog. The fork fixes only that fixture's pending-I/O lifetime, released on abort; production timeout policy is unchanged.
- A fresh installed-Pi RPC process registered this fork and the built pi-usage fork in an isolated agent directory. No paid inference was sent as part of migration.
