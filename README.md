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

## What clients edit

- **Pages:** homepage and project-index sections; generic project detail composition.
- **Content → Projects:** title, year, category, description, images, story chapters
  and gallery. Shared cards and detail routes use these records.
- **Site Layouts:** Archline's surrounding design/settings. Multiple independently
  configured layouts are supported, including the installed Intr design.
- **Media:** existing local image records or uploads after configuring storage.

`quinecms.config.json` configures subscriptions and the initial Archline layout
used for factory reset. These settings and dependency revisions are developer-owned,
outside the admin portal. `content/site.json` and each page's `layoutId` own saved
layout configurations and assignments.

## Upgrade and rollback

Save/export the latest browser draft before upgrading. Change one engine/library
Git SHA in package.json on a branch, run `pnpm install`, commit the lockfile, and
run `pnpm verify` plus desktop/mobile/editor checks. Migrate content or original
extensions manually if needed. Installed library changes go upstream.

The old editor blocks Save and Publish after engine/library/config/source changes;
unsaved work is retained. Open an editor built from the matching branch to continue.
To roll back, restore the previous pins and lockfile and any content snapshot already
changed for the newer contract. Code rollback alone does not restore migrated data.
