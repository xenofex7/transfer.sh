Run the project's deploy script to release a new version. Optional version as argument, e.g. `v1.0.2`. Without an argument the patch version is auto-incremented from the latest tag (e.g. v1.0.1 -> v1.0.2).

## Steps

1. Check whether `$ARGUMENTS` contains a version (format: `vX.Y.Z`)
2. If no version is supplied, the script auto-increments the patch
3. Run the deploy script:

```bash
bash scripts/deploy.sh $ARGUMENTS
```

4. Wait for the output - the script does the following:
   - Validate the version and the working tree
   - Run `go vet ./...`
   - Run `go test -race ./...`
   - Generate a CHANGELOG entry from the commits since the last tag (uses Claude CLI if available, falls back to a raw commit list)
   - Commit `CHANGELOG.md` (only if it actually changed)
   - Create an annotated git tag containing the changelog
   - Push branch and tag to origin
   - Create a GitHub Release via `gh` (which triggers the GHCR build)
5. On success: report the version number and the GHCR + Release URLs

## Rules

- Only run when the user explicitly asks to deploy
- Version is optional; without an argument the patch is auto-incremented
- Never modify the script - only execute it

## Important

- **Dirty working tree**: commit or stash first, the script will refuse otherwise
- **Tests fail**: fix them before deploying
- **gh CLI not authenticated**: the push still works, but the GitHub Release must be created manually from the URL the script prints
- **Not on `main`**: the script warns and asks for confirmation
- **Watchtower** on the server picks up the new image automatically once the GHCR build is green
