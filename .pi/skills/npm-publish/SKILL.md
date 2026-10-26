---
name: npm-publish
description: Publish an npm package from scratch. Use when creating a new scoped package, bootstrapping a package on npm, or performing a manual npm publish with --access public.
---

# npm Publish

Publish an npm package, including first-time bootstrap for scoped packages.

## Bootstrap a new scoped package

Required when the package does not exist on npm yet. After this, steady-state releases use CI-driven trusted publishing.

### 1. npm login

```bash
npm login
npm whoami
```

npm requires 2FA (Authenticator App recommended).

### 2. Publish

```bash
npm publish --access public
```

`--access public` is required for scoped packages (`@scope/name`). Without it, the package is published as private.

### 3. Verify

```bash
npm view @scope/name
npm install -g @scope/name
```

## Version management

Use semantic versioning. `npm version` auto-updates `package.json` and creates a git tag.

| Increment | When | Example |
|-----------|------|---------|
| `patch` | Bug fixes, no API change | `1.0.0` → `1.0.1` |
| `minor` | New features, backward compatible | `1.0.0` → `1.1.0` |
| `major` | Breaking changes | `1.0.0` → `2.0.0` |

```bash
npm version patch   # auto-updates version + creates tag
npm publish         # publishes
```

Or skip the tag:

```bash
npm version patch --no-git-tag-version
git add package.json package-lock.json CHANGELOG.md
git commit -m "chore: release v$(node -p "require('./package.json').version")"
git push origin main  # triggers CI release workflow
```

## Pre-publish checklist

- `package.json` name, version, description correct
- `files` whitelist only contains required directories (safer than `.npmignore` blacklist)
- `engines` declares minimum Node.js version
- `repository` and `homepage` point to GitHub
- `npm pack --dry-run` shows only expected files, no `.env` or sensitive data
- README has a 30-second quickstart
- `npm login` authenticated

## package.json essentials

```json
{
  "name": "@scope/package-name",
  "version": "1.0.0",
  "type": "module",
  "bin": { "command": "bin/cli.mjs" },
  "files": ["bin", "src", "templates"],
  "engines": { "node": ">=18" },
  "description": "One-line description for npm search",
  "keywords": ["keyword1", "keyword2"],
  "repository": { "type": "git", "url": "https://github.com/owner/repo" },
  "homepage": "https://github.com/owner/repo#readme",
  "license": "MIT",
  "author": "Author Name"
}
```

## CI publishing (trusted publishing)

After the package exists on npm, configure GitHub Actions trusted publishing:

1. npmjs.com → package settings → trusted publishers
2. Add GitHub Actions: org/user, repo, workflow filename, allowed action `npm publish`
3. Workflow job needs `permissions.id-token: write` on a GitHub-hosted runner
4. Publish with: `npm publish --access public --provenance`

CI also needs `npm token create --cidr=0.0.0.0/0` for the NPM_TOKEN secret, or use tokenless OIDC.

## Common pitfalls

1. **Missing build step** — add `prepublishOnly` script
2. **Sensitive files in tarball** — always run `npm pack --dry-run`
3. **Package name taken** — add scope prefix: `@yourname/tool-name`
4. **Version already published** — npm versions are immutable; bump and republish
5. **2FA in CI** — use automation token: `npm token create --cidr=0.0.0.0/0`
