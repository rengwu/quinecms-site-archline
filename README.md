# Archline — independent QuineCMS site

This repository owns Archline's content and configuration. It pins the public
[QuineCMS engine](https://github.com/rengwu/quinecms) and
[design library](https://github.com/rengwu/quinecms-library) to independent Git
commits. It contains no engine, admin or library implementation copy.

## Develop and build

Node 22.12+ and pnpm 10.34.1:

```sh
pnpm install --frozen-lockfile
pnpm dev
```

Open `/admin/` using `admin@example.com` / `asdf1234` in development. Browser edits
are a recoverable local draft; they do not write this checkout. Edit JSON locally,
export a backup, or connect the editor to this repository to save to `draft` and
publish the saved snapshot to `main`.

For a production build, copy `.env.example` to ignored `.env` and supply the
admin email/password, or provide them as build environment secrets:

```sh
pnpm verify
pnpm preview
```

The engine assembles ignored `.quinecms/workspace/` and produces static `dist/`.
Deploy only `dist/` to any static host or bare-metal web server. Serve it at the
domain root with normal directory-index handling; no runtime Node server is needed.
Do not expose `.env`, source, node_modules or the build workspace. The repository
provider and hosting provider can differ. Publishing a Git snapshot does not itself
configure or confirm hosting.

## Deploy this pilot to Netlify

This is the maintainer's chosen pilot host; the engine still produces the same
portable static output for any host. The following setup has not yet been tested
on Netlify.

1. In Netlify, add a project from GitHub and select
   `rengwu/quinecms-site-archline`.
2. Set the production branch to `main`, leave the base/package directories empty
   (repository root), set the build command to `pnpm build`, and set the publish
   directory to `dist`. These are ordinary
   [Netlify build settings](https://docs.netlify.com/build/configure-builds/overview/).
3. Before building, add these environment variables in Netlify. If scope/context
   controls are offered, include **Builds** and **Production**. Store the two admin
   values as secrets; they become the deployed `/admin/` login.

| Variable | Value |
| --- | --- |
| `NODE_VERSION` | `22.19.0` (the locally verified version) |
| `ADMIN_EMAIL` | Your chosen admin email |
| `ADMIN_PASSWORD` | Your own long, unique password |
| `PUBLIC_GIT_PROVIDER` | `github` |
| `PUBLIC_GIT_NAMESPACE` | `rengwu` |
| `PUBLIC_GIT_REPO` | `quinecms-site-archline` |
| `PUBLIC_DRAFT_BRANCH` | `draft` |
| `PUBLIC_PUBLISH_BRANCH` | `main` |

Netlify reads `pnpm-lock.yaml` and the existing `packageManager` field to install
with pnpm 10.34.1 before running the build; no extra install command is needed.
See [dependency settings](https://docs.netlify.com/build/configure-builds/manage-dependencies/)
and [build variables](https://docs.netlify.com/build/configure-builds/environment-variables/).
The engine and library are public dependencies, so their installation needs no
GitHub token. Authorizing edits from the admin is a separate GitHub connection.

4. Deploy, then check `/`, open a project detail page, and unlock `/admin/` with
   the credentials you configured. The production build does not use the fixed
   development login. Disable branch/PR preview builds for this first setup;
   enabling them later needs suitable build credentials for those contexts.

With Git-linked deployment, a QuineCMS Publish that updates `main` triggers the
next Netlify build. Save alone updates `draft`. Missing `ADMIN_EMAIL` or
`ADMIN_PASSWORD` fails the build; add both and retry.

If you prefer a manual file upload, build locally using the production `.env`
setup above and upload only the resulting `dist/` folder. That workflow needs
a fresh local build and upload after later content publication.

## Optional R2 uploads

Use the commented R2 sections in [`.env.example`](.env.example). Set all five
`R2_ACCOUNT_ID`, `R2_BUCKET`, `R2_PUBLIC_URL`, `R2_ACCESS_KEY_ID` and
`R2_SECRET_ACCESS_KEY` values in the build environment to enable uploads; leave
all five blank to disable them. Tune size/variant defaults there as needed.
Configure the bucket's public access and CORS for the editor's actual origin,
then rebuild and redeploy. There is no storage setup screen in the admin.

Keys are included only in the encrypted admin payload. Anyone who unlocks the
editor can recover them for direct uploads; use bucket-scoped credentials and
test keys in development. The development login is fixed. See the engine's
[media setup and CORS guide](https://github.com/rengwu/quinecms/blob/engine-separation/docs/wiki/features/media.md#build-environment-setup).

## What clients edit

- **Pages:** homepage and project-index sections; generic project detail composition.
- **Content → Projects:** title, year, category, description, images, story chapters
  and gallery. Shared cards and detail routes use these records.
- **Site Layouts:** Archline's surrounding design/settings. Multiple independently
  configured layouts are supported, including the installed Intr design.
- **Media:** existing local image records, plus uploads when R2 is configured in the build environment.

`quinecms.config.json` configures subscriptions and the initial Archline layout
used for factory reset. These settings and dependency revisions are developer-owned,
outside the admin portal. `content/site.json` and each page's `layoutId` own saved
layout configurations and assignments.

## Upgrade and rollback

Save/export the latest browser draft before upgrading. Change one engine/library
Git SHA in package.json on a branch, run `pnpm install`, commit the lockfile, and
run `pnpm verify` plus desktop/mobile/editor checks. Migrate content or original
extensions manually if needed. Installed library changes go upstream.

Place the reviewed upgrade commit on the draft branch before connecting its matching
editor. Publish that exact upgraded draft, or fast-forward both branches to the same
reviewed commit when deploying through Git. Do not leave the connected draft on old
pins while building the editor from newer ones.

The old editor blocks Save and Publish after engine/library/config/source changes;
unsaved work is retained. Open an editor built from the matching branch to continue.
To roll back, restore the previous pins and lockfile and any content snapshot already
changed for the newer contract. Code rollback alone does not restore migrated data.
