# pretiosusdata.com — Local Setup

## Install Hugo

```bash
brew install hugo
```

Verify:

```bash
hugo version
```

## Run locally

```bash
cd ~/Developer/pretiosusdata
hugo server -D
```

Open http://localhost:1313

The `-D` flag includes draft posts. Remove it to see only published posts.

The server watches for changes — edit a markdown file, save it, and the browser refreshes automatically.

## Create a new note

```bash
hugo new notes/my-topic.md
```

This creates `content/notes/my-topic.md` with front matter pre-filled.
Edit the file, write your content, set `draft: false` when ready.

## Create a new reference

```bash
hugo new references/my-topic.md
```

Same process, longer format.

## Publish

```bash
git add .
git commit -m "note: my topic"
git push
```

Cloudflare Pages auto-deploys on push.

## Front matter format

Every post starts with:

```yaml
---
title: "Your title here"
date: 2026-04-02
tags: ["aws", "airbyte"]
draft: false
---
```

- `draft: true` — only visible locally with `-D` flag
- `draft: false` — published to pretiosusdata.com on push
- `tags` — optional, used for grouping
