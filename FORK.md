# aberrantCode fork of paperless-ngx-mcp

This is a personal fork of [cubinet-code/paperless-ngx-mcp](https://github.com/cubinet-code/paperless-ngx-mcp),
kept at <https://github.com/aberrantCode/paperless-ngx-mcp>. It tracks upstream `main` and carries a
small set of security-hardening patches, each developed on its own branch and merged into the fork's
`main` with a merge commit (`git merge --no-ff`) so every patch stays identifiable and can be offered
upstream or dropped once upstream has an equivalent.

The fork is used to build a Claude Desktop extension (`.mcpb`) from a pinned commit. It does **not**
publish to npm, the MCP Registry or GHCR: `release.yml` only runs its publish jobs in
`cubinet-code/paperless-ngx-mcp`.

## Patches

| Branch | What it does | Upstream PR |
| --- | --- | --- |
| `fix/audit-deps` | `npm audit fix` for all production advisories; raises minimum versions of the affected direct dependencies. | [cubinet-code#7](https://github.com/cubinet-code/paperless-ngx-mcp/pull/7) |
| `fix/http-bind-localhost` | `--http` binds `127.0.0.1` by default (`--host` / `PAPERLESS_MCP_HOST` to change; Docker image sets `0.0.0.0`); DNS-rebinding `Host` check on loopback. | [cubinet-code#8](https://github.com/cubinet-code/paperless-ngx-mcp/pull/8) |
| `feat/upload-path-allowlist` | `PAPERLESS_UPLOAD_DIRS` / `--uploadDirs` limits which server-local paths the upload tools may read (`none` disables path reads; unset keeps upstream behaviour). | [cubinet-code#9](https://github.com/cubinet-code/paperless-ngx-mcp/pull/9) |
| `fix/windows-test-glob` | `test` / `test:e2e` scripts double-quote their globs so `npm test` runs the tests on Windows (single quotes reached `node --test` literally via `cmd.exe`, matching 0 files and exiting 0). | _not yet opened_ |
| `chore/fork-dependabot` | Dependabot config, `npm audit` step in CI, and a `github.repository` guard so the release workflow never publishes from the fork. | fork-only |

Fork-only commits directly on `main` (not intended for upstream):

- `manifest.json`: display name `Paperless-ngx (aberrantCode fork)`, version `<upstream>-fork.N`,
  repository/homepage/support pointing at this fork, and an optional `upload_dirs` setting
  (default `${HOME}/PaperlessUpload`) mapped to `PAPERLESS_UPLOAD_DIRS`.
- This file.

`package.json` keeps the upstream version; only `manifest.json` carries the `-fork.N` suffix.

## Syncing with upstream

```bash
git fetch upstream
git checkout main
git merge --no-ff upstream/main      # resolve conflicts; CHANGELOG keeps a single [Unreleased] section
npm ci
npm run build
npm test                             # must report the full test count, not "tests 0"
npm audit --omit=dev                 # must report 0 vulnerabilities
git push origin main
```

After the merge:

1. If upstream bumped its version, set `manifest.json` `version` to `<new upstream version>-fork.1`
   (otherwise bump the `-fork.N` counter when the fork's own changes change).
2. If upstream merged one of the patches above (or an equivalent), update the table and drop the
   fork's copy when resolving conflicts.
3. Rebuild the extension from the new `main` SHA with the pinned build script
   (`scripts/build-mcpb.ps1` in the paperless-claude project, `-Ref <new sha>`), and update its
   pinned default SHA.

To refresh a patch branch for an upstream PR, rebase it onto `upstream/main` in its own worktree;
`main` itself is never rebased.
