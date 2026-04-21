# 🔊 Yodelog

**Write Markdown. Push to GitHub. Broadcast everywhere.**

Yodelog is a stateless, serverless microblogging pipeline. Write your micro-posts inside Markdown files, push to `main`, and a GitHub Action automatically broadcasts them to **Mastodon** and **BlueSky**. Keep your repository private, or make it public to also publish your content as a [Docsify](https://docsify.js.org/) website on **GitHub Pages**.

---

## ✨ Features

- **Append-Only** — Relies on `git diff`. Editing old posts to fix typos will *not* republish them.
- **Zero Config** — Use as a GitHub Template. If API keys are absent, the Action logs what *would* be posted without failing.
- **Smart Threading** — Long posts are automatically split into numbered threads at safe boundaries (paragraphs → sentences → words).
- **Image Support** — Use standard Markdown image syntax. Images stay attached to the thread chunk where you placed them.
- **Manual Threads** — Force a new thread reply with `---` horizontal rules.
- **Dry-Run Mode** — Name your file `*.dryrun.md` to test without broadcasting.
- **Scheduled Posts** — Add `{time: 2026-06-01T09:00Z}` to a heading and a cron job will broadcast it at the specified time, instead of immediately on commit.
- **No Dependencies** — Pure Node.js 22, zero npm packages. Fast, auditable, no supply-chain risk.

---

## 🚀 Quick Start

### 1. Use This Template

Click **"Use this template"** on GitHub to create your own repository.

### 2. Add Your API Keys

Go to **Settings → Secrets and variables → Actions** in your new repository.

#### For Mastodon
| Type | Name | Value |
|------|------|-------|
| Secret | `MASTODON_ACCESS_TOKEN` | Your Mastodon access token |
| Variable | `MASTODON_INSTANCE_URL` | e.g. `https://mastodon.social` |

#### For BlueSky
| Type | Name | Value |
|------|------|-------|
| Variable | `BLUESKY_HANDLE` | e.g. `you.bsky.social` |
| Secret | `BLUESKY_APP_PASSWORD` | An App Password from BlueSky settings |

> **Skip platforms you don't use.** If only Mastodon keys are set, only Mastodon gets posts (and vice versa).

### 3. Write Your First Post

Create any `.md` file (e.g. `journal/2026.md`) and add:

```markdown
---
yodelog: true
---

## My first broadcast
Hello from my git-powered microblog! 🎉
```

### 4. Push to `main`

```bash
git add journal/2026.md
git commit -m "first post"
git push
```

The GitHub Action will pick up the new `##` heading from the diff and broadcast it. Done!

---

## 🌐 Public Site (GitHub Pages)

Yodelog includes a pre-configured `index.html` file that lets you optionally host your microblog on the web using GitHub Pages and [Docsify](https://docsify.js.org/).

To enable this:
1. Go to your repository settings on GitHub.
2. Navigate to **Pages** (under the "Code and automation" section).
3. Under **Build and deployment**, select **Deploy from a branch**.
4. Set the branch to `main` (or whichever branch you push to) and the folder to `/root`. Click **Save**.

Within a few minutes, your repository contents will be accessible as a public website at `https://<your-username>.github.io/<repo-name>/`. You can preview the layout live by opening `index.html` locally using a simple HTTP server (e.g., `python -m http.server 8000` or an extension like Live Server) and navigating to `http://localhost:8000`.

**Customizing the Sidebar**

The site uses a sidebar for navigation. To list the files you want to display on your site, you need to create or edit a `_sidebar.md` file in the root of your repository with links to your markdown files (or adjust to an existing sidebar if you configure `docsify`). Please refer to the [Docsify documentation](https://docsify.js.org/) for details.

---

## 📝 Writing Guide

### File Structure

You can organize your files however you like:

```
posts/
├── 2026/
│   ├── january.md
│   └── february.md
├── ideas.md
└── daily.md
```

Every `.md` file with `yodelog: true` in its frontmatter will be scanned for new posts on push or schedule.

### Posts = `##` Headings

Each `##` heading marks the start of a new, independent post:

```markdown
## Morning thought
Coffee is essential.

## Evening reflection
Sleep is also essential.
```

Pushing this adds two separate posts.

> The heading text will not be posted on social media, but it will be used for titles in the [website version](#-public-site-github-pages).

**Empty headings** (`##`) are supported.

```markdown
##
This is a perfectly valid post, but will look a bit dry if you use GitHub Pages in your setup.
```

### Frontmatter

Each markdown file must start with a frontmatter block:

```yaml
---
yodelog: true                     # REQUIRED — identifies this file for broadcasting
prefix: "📝 "                     # OPTIONAL — prepended to the first post in a thread
suffix: "#journal #notes"         # OPTIONAL — appended to the last post
thread_style: "{current}/{total}" # OPTIONAL — numbering for auto-threaded posts
post_on: push_or_schedule         # OPTIONAL — push | schedule | push_or_schedule (default: push_or_schedule)
---
```

The `post_on` key controls which pipeline processes the file:

| Mode | Push triggers broadcast? | Cron triggers broadcast? |
|------|--------------------------|-------------------------|
| `push` | ✅ Yes | ❌ No |
| `schedule` | ❌ No | ✅ Yes (requires `{time:}` in heading) |
| `push_or_schedule` | ✅ Yes (only if no `{time:}` in heading) | ✅ Yes (only if `{time:}` in heading) |

## ⚡ Publishing

### Push Posts

Push posts are broadcast immediately when you push to the repository.

**How it works:**

1. You push a new markdown file to the repository.
2. The GitHub Action triggers and scans the file for new posts.
3. Posts are chunked into threads if necessary, images are reeoncoded.
4. The Action posts the new posts to Mastodon and/or BlueSky.

#### The Append-Only Rule

This system is an **append-only log**. It only cares about *new* lines in a push:

- ✅ Adding a new `##` heading → **broadcasts**
- ❌ Editing text in an old post → **ignored**
- ❌ Deleting a post → **ignored**
- ✅ Adding another `##` below existing posts → **broadcasts** only the new one

### Scheduled Posts

Add a `{time: ...}` tag to any `##` heading to schedule it for later (if `post_on` is set to `schedule` or `push_or_schedule`):

```markdown
## Universe expansion announcement {time: 2026-06-01T09:00Z}
We're thrilled to release the trailer for Yodelog The Movie!
```

> Time must be in ISO 8601 format (e.g. `2026-06-01T09:00Z`, `2026-06-01T14:30+02:00`).

**How it works:**

1. A cron GitHub Action runs every hour.
2. It reads a watermark tag (`yodelog-cron-watermark`) to know when it last ran.
3. It scans all markdown files for posts with `{time: ...}` tags.
4. Posts whose scheduled time falls between the watermark and now are broadcast using the same processing and threading logic as push posts.
5. The watermark is updated to the current time.

The `{time: ...}` tag is stripped with the rest of the heading before broadcasting, so your followers won't see it.

In files with `post_on: push_or_schedule`, pushed posts without a `{time: ...}` tag broadcast immediately as usual. Posts *with* a `{time: ...}` tag are skipped by the push trigger and deferred to the cron job.

### Manual Threads

Use `---` on its own line between content to force thread breaks:

```markdown
## My hot take
This is part 1 of my thread.

---
This is part 2 — a separate reply in the thread.

---
And this is part 3.
```

### Images

Use standard Markdown images:

```markdown
## Check out this diagram
Here's what the architecture looks like:

![Architecture diagram](./assets/arch.png)

Pretty clean, right?
```

The image is uploaded to the platform and attached to the specific thread chunk where it appears (proximity rule).

## 🔍 Dry-Run Mode

### Automatic Dry Run
If no API credentials are configured, the Action runs in dry-run mode automatically — it logs what would be posted without failing.

### Manual Dry Run
Name your file with a `.dryrun.md` suffix:

```
drafts/experiment.dryrun.md
```

When pushed, the Action will:
- ✅ Parse the file and validate frontmatter
- ✅ Run the diff and splitter
- ✅ Log the exact posts that *would* be sent
- ❌ Skip actual API calls

Rename the file (remove `.dryrun`) when you're ready to go live.

## 📁 Repository Structure

```
├── .github/workflows/
│   ├── broadcast.yml                — Push-triggered instant broadcasting
│   └── schedule.yml                 — Cron-triggered scheduled broadcasting
├── src/
│   ├── main.js                      — Pipeline orchestrator (instant + cron modes)
│   ├── diff.js                      — Git diff engine (append-only reader)
│   ├── parser.js                    — Frontmatter + content + schedule tag parser
│   ├── schedule.js                  — Watermark management + scheduled post scanner
│   ├── splitter.js                  — Smart thread splitting
│   ├── platforms/
│   │   ├── mastodon.js              — Mastodon API client
│   │   └── bluesky.js               — BlueSky AT Protocol client
│   └── utils.js                     — Shared helpers
├── example/posts.md                 — Example content file
└── package.json                     — Metadata (no dependencies)
```

---

## 📜 License

MIT — do whatever you want with it.
