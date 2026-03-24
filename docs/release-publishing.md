# Release & Publishing

The lib package is published to npm as `@vojtechsimetka/swarm-id` using automated releases with npm Trusted Publishing (OIDC).

## How it works

1. **Conventional commits** (`feat:`, `fix:`, etc.) scoped to `lib/` are tracked by release-please
2. On push to `main`, **release-please** creates/updates a release PR with version bump + changelog
3. Merging the release PR creates a **GitHub Release** and triggers npm publish
4. Publishing uses **npm Trusted Publishing (OIDC)** — no npm tokens needed
5. If publish fails, re-run it manually: Actions → "Publish to npm" → "Run workflow"

## Key files

| File | Purpose |
|------|---------|
| `.github/workflows/release.yml` | Runs release-please, calls publish on release |
| `.github/workflows/publish.yml` | Builds and publishes to npm (reusable + manual trigger) |
| `release-please-config.json` | Configures which packages to release and npm package name |
| `.release-please-manifest.json` | Tracks current version |
| `lib/package.json` `repository` | Must match GitHub repo URL (required for OIDC) |
| `lib/package.json` `publishConfig` | Sets `access: public` and `provenance: true` |

## Important details

- Only commits touching files under `lib/` trigger a release PR
- The lib uses `@swarm-id/lib` as its workspace name internally; the publish workflow overrides the name to the npm package name before publishing
- The GitHub repo **must be public** for npm provenance/OIDC to work
- npm CLI is upgraded to latest at publish time for OIDC support

## Switching to `@snaha/swarm-id`

To publish under a different npm scope (e.g., `@snaha/swarm-id`):

1. **npm**: Create the `@snaha` org on npmjs.com (or ensure it exists)
2. **npm**: Manually publish once to create the package:
   ```bash
   cd lib
   npm pkg set name="@snaha/swarm-id"
   pnpm build
   npm publish --access public
   git checkout package.json  # revert name change
   ```
3. **npm**: Go to package Settings → Publishing access → Add Trusted Publisher:
   - Repository owner: `snaha`
   - Repository name: `swarm-id` (whatever the GitHub repo is)
   - Workflow filename: `publish.yml`
   - Environment: (leave blank)
4. **npm**: Set publishing access to "Require two-factor authentication or a granular access token with bypass 2fa enabled"
5. **`publish.yml`**: Change `@vojtechsimetka/swarm-id` → `@snaha/swarm-id` (two places: `npm pkg set` and `pnpm --filter`)
6. **`release-please-config.json`**: Change `package-name` to `@snaha/swarm-id`
7. **`lib/package.json`**: Update `repository.url` to the new GitHub repo URL
