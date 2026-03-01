# GitHub Utils Actions

A set of local composite actions for deploying a static site to GitHub Pages.

## What is included

- `.github/actions/prepare-repo` - checkout, setup Node.js, and `npm ci`.
- `.github/actions/configure-git-push` - configures git and `origin` with an auth token.
- `.github/actions/ensure-cname` - creates a `CNAME` file in the build output (if `site_url` is set).
- `.github/actions/prepare-and-build-for-gh-pages` - combines all of the above into one flow: prepare -> build -> CNAME -> deploy.

## Quick start

Create a workflow that calls the main action:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Prepare, build and deploy
        uses: ./.github/actions/prepare-and-build-for-gh-pages
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          site_url: ${{ secrets.NEXT_PUBLIC_SITE_URL }}
          # optional:
          # node-version: "20"
          # github-repository: "owner/repo"
          # output_path: "dist"
          # build-env-vars: |
          #   NODE_ENV=production
          #   NEXT_PUBLIC_API_URL=${{ vars.NEXT_PUBLIC_API_URL }}
          # build-env-file-path: ".env.production"
          # build-env-file-content: ${{ secrets.ENV_FILE_PRODUCTION }}
          # build-command: "npm run build"
          # deploy-command: "npm run deploy"
```

## Main action inputs

`./.github/actions/prepare-and-build-for-gh-pages`

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `github-token` | yes | - | Token used for pushing to the repository. Usually `${{ secrets.GITHUB_TOKEN }}`. |
| `node-version` | no | `20` | Node.js version used for setup. |
| `github-repository` | no | `""` | Repository in `owner/name` format. If empty, `GITHUB_REPOSITORY` is used. |
| `site_url` | no | `""` | Site URL used to generate `CNAME` (for example, `https://example.com`). |
| `output_path` | no | `dist` | Build output directory where `CNAME` is written. |
| `build-env-vars` | no | `""` | Multiline list in `KEY=VALUE` format, exported before build/deploy. |
| `build-env-file-path` | no | `""` | Path to an env file that should be created before build (for example, `.env.production`). |
| `build-env-file-content` | no | `""` | Content written to `build-env-file-path` (convenient to store in `secrets`). |
| `build-command` | no | `npm run build` | Build command. |
| `deploy-command` | no | `npm run deploy` | Deploy command. |

## How to pass a different env

There are 2 generic options:

1. Through variables (`build-env-vars`) - when you only need to inject several environment variables.
2. Through a file (`build-env-file-path` + `build-env-file-content`) - when your framework expects a specific env file.

Staging example:

```yaml
- name: Prepare, build and deploy
  uses: ./.github/actions/prepare-and-build-for-gh-pages
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    site_url: ${{ vars.SITE_URL_STAGING }}
    build-env-vars: |
      APP_ENV=staging
      NEXT_PUBLIC_API_URL=${{ vars.API_URL_STAGING }}
    build-env-file-path: ".env.production"
    build-env-file-content: ${{ secrets.ENV_FILE_STAGING }}
    build-command: "npm run build"
    deploy-command: "npm run deploy"
```

## Important notes

- `site_url` is optional. If it is empty, `CNAME` is not created.
- Only the domain is written to `CNAME` (without `https://` and without path).
- Your workflow must have `permissions.contents: write`, otherwise push will fail.
- It is assumed that your `package.json` has `build` and `deploy` scripts (or pass custom commands via inputs).
- Use `secrets` (not `vars`) for sensitive environment values.
