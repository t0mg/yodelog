# 🔊 Yodelog

**Write Markdown. Push to GitHub. Broadcast everywhere.**

Yodelog is a stateless, serverless microblogging pipeline. Write your micro-posts inside Markdown files, push to `main`, and a GitHub Action automatically broadcasts them to **Mastodon** and **BlueSky**.

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

The GitHub Action will pick up the new `## ` heading from the diff and broadcast it. Done!

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

Every `.md` file with `yodelog: true` in its frontmatter will be scanned for new posts on push.

### Posts = `## ` Headings

Each `## ` heading marks the start of a new, independent post:

```markdown
## Morning thought
Coffee is essential.

## Evening reflection
Sleep is also essential.
```

Pushing this adds two separate posts.

**Empty headings** (`##`) are supported if you don't want heading text in your post:

```markdown
##
Just the content, no heading line in the broadcast.
```

### Frontmatter

```yaml
---
yodelog: true               # REQUIRED — identifies this file for broadcasting
prefix: "📝 "                    # OPTIONAL — prepended to the first post in a thread
suffix: "#journal #notes"        # OPTIONAL — appended to the last post
thread_style: "{current}/{total}" # OPTIONAL — numbering for auto-threaded posts
post_on: push_or_schedule   # OPTIONAL — push | schedule | push_or_schedule (default: push_or_schedule)
---
```

The `post_on` key controls which pipeline processes the file:

| Mode | Push triggers broadcast? | Cron triggers broadcast? |
|------|--------------------------|-------------------------|
| `push` | ✅ Yes | ❌ No |
| `schedule` | ❌ No | ✅ Yes (requires `{time:}` in heading) |
| `push_or_schedule` | ✅ Yes (only if no `{time:}` in heading) | ✅ Yes (only if `{time:}` in heading) |

### Scheduled Posts

Add a `{time: ...}` tag to any `## ` heading to schedule it for later:

```markdown
## Product launch announcement {time: 2026-06-01T09:00Z}
We're thrilled to announce the release of...
```

**How it works:**

1. A cron GitHub Action runs every hour.
2. It reads a watermark tag (`yodelog-cron-watermark`) to know when it last ran.
3. It scans all markdown files for posts with `{time: ...}` tags.
4. Posts whose scheduled time falls between the watermark and now are broadcast.
5. The watermark is updated to the current time.

The `{time: ...}` tag is stripped from the heading before broadcasting — your followers won't see it.

**In `mode: both` files**, pushed posts without a `{time: ...}` tag broadcast immediately as usual. Posts *with* a `{time: ...}` tag are skipped by the push trigger and deferred to the cron job.

The time must be a valid ISO 8601 timestamp (e.g. `2026-06-01T09:00Z`, `2026-06-01T14:30+02:00`).

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

---

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

---

## ⚡ How It Works

```
Push to main
    │
    ▼
GitHub Action triggers
    │
    ▼
git diff extracts only NEW lines (lines starting with +)
    │
    ▼
Scans for files with yodelog: true frontmatter
    │
    ▼
Groups new content into posts (## headings)
    │
    ▼
Splits into threads if needed (manual --- or auto by char limit)
    │
    ▼
Uploads images, posts threads to Mastodon / BlueSky
```

### The Append-Only Rule

This system is an **append-only log**. It only cares about *new* lines in a push:

- ✅ Adding a new `## ` heading → **broadcasts**
- ❌ Editing text in an old post → **ignored** (not re-broadcast)
- ❌ Deleting a post → **ignored**
- ✅ Adding another `## ` below existing posts → **broadcasts** only the new one

This means you can safely fix typos in old posts without accidentally re-posting them.

---

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
