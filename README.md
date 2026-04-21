# 🏔️ Yodelog

**Write Markdown. Push to GitHub. Broadcast everywhere.**

Yodelog is a stateless, serverless microblogging pipeline. Write your micro-posts inside Markdown files, push to `main`, and a GitHub Action automatically broadcasts them to Mastodon and BlueSky. You can keep your repository private for publishing only, or make it public to also host your content as a Docsify website on GitHub Pages. Note that making the repo public slightly spoils the scheduling feature, as raw markdown files will be visible to everyone (though future posts are filtered out in the web view).

---

## Features

- **Append-Only** — "Push" mode relies on `git diff`. Editing old posts to fix typos won't republish them.
- **Multiple Outlets** — Broadcast your thoughts to Mastodon, BlueSky, or publish as a Docsify web view.
- **Zero Config** — Works instantly as a GitHub Template. Missing API keys will just log a dry-run.
- **Smart Threading** — Long posts automatically split into numbered threads at safe boundaries.
- **Manual Threads** — Force a new thread reply with a horizontal rule (`---`).
- **Scheduled Posts** — Add a `{time: YYYY-MM-DDTHH:MMZ}` tag to broadcast later.
- **Image Support** — Standard Markdown images stay attached to the relevant thread chunk.
- **No Dependencies** — Pure Node.js 22, meaning fast execution and zero supply-chain risk. Note that broadcasting relies entirely on GitHub Actions.

---

## Quick Start

### 1. Use This Template
Click **"Use this template"** on GitHub to create your own repository.

### 2. Add API Keys
Go to **Settings → Secrets and variables → Actions** in your repository. Add the keys for the platforms you want to use.

**Mastodon:**
- Secret: `MASTODON_ACCESS_TOKEN`
- Variable: `MASTODON_INSTANCE_URL` (e.g., `https://mastodon.social`)

**BlueSky:**
- Variable: `BLUESKY_HANDLE` (e.g., `you.bsky.social`)
- Secret: `BLUESKY_APP_PASSWORD` (App Password from settings)

### 3. Write & Push
Create any `.md` file (e.g., `journal.md`) and add:

```markdown
---
yodelog: true
---

## My first broadcast 🪵
Hello from my git-powered microblog! 🏔️
```

Commit and push to `main`. The GitHub Action detects the new `##` heading and broadcasts it. Done!

---

## Writing & Publishing

### File Structure & Frontmatter
Organize your markdown files however you like. Any `.md` file with the following frontmatter will be scanned:

```yaml
---
yodelog: true                     # REQUIRED
prefix: "Memo: "                  # OPTIONAL — prepended to the first post
suffix: "#notes"                  # OPTIONAL — appended to the last post
thread_style: "{current}/{total}" # OPTIONAL — numbering style
post_on: push_or_schedule         # OPTIONAL — push | schedule | push_or_schedule
---
```

### Posts = Headings
Each `##` heading marks a new post. Empty headings are supported. The heading titles are only visible in the Docsify web version and are **not** broadcasted to social media.

Yodelog is an **append-only log**—only new headings added in a push trigger a broadcast. Edits or deletions of old posts are ignored.

### Scheduled Posts
To schedule a post, add a `{time: ...}` tag in ISO 8601 format to your heading. A cron job checks every hour and broadcasts it when the time arrives.

```markdown
## Mountain retreat plans 🏔️ {time: 2026-06-01T09:00Z}
Packing up the cabin for the winter. Don't forget the firewood! 🪵
```

If your file uses `post_on: push_or_schedule` (the default), posts without a `{time: ...}` tag will broadcast immediately on push, while those with a tag are skipped by the push trigger and deferred to the cron job.

### Manual Threads & Images
Use `---` on its own line to force thread breaks within a single post. Standard Markdown images are supported and will stay attached to the thread chunk where they appear.

---

## Public Web View

Yodelog includes an `index.html` file to host your microblog on GitHub Pages using [Docsify](https://docsify.js.org/).

1. In GitHub, go to **Settings → Pages**.
2. Select **Deploy from a branch**, set it to `main`, and folder to `/root`. Click **Save**.
3. Create a `_sidebar.md` to link to your files for site navigation.

Within minutes, your markdown files will be visible as a public website.

---

## Dry-Run Mode

Want to test without posting? Name your file with `.dryrun.md` (e.g., `draft.dryrun.md`). The Action will run the diff, parse content, split threads, and log what *would* be sent, skipping actual API calls. Missing API credentials will also trigger dry-run mode automatically.

---

## Repository Structure

```text
├── .github/workflows/  — GitHub Actions (instant & scheduled broadcasts)
├── example/            — Example content files
├── src/                — Core Node.js logic (diffing, parsing, APIs)
├── tests/              — Unit tests
├── .gitignore          — Ignored files
├── index.html          — Docsify web entry point
├── LICENSE             — MIT License
├── package.json        — Metadata & scripts
└── README.md           — This file
```

---

## License

MIT — do whatever you want with it.
