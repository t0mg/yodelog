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
- **No Dependencies** — Pure Node.js 20, zero npm packages. Fast, auditable, no supply-chain risk.

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
---
```

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
├── .github/workflows/broadcast.yml  — The Action that runs the pipeline
├── src/
│   ├── main.js                      — Pipeline orchestrator
│   ├── diff.js                      — Git diff engine (append-only reader)
│   ├── parser.js                    — Frontmatter + content parser
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
