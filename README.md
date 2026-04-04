# pretiosusdata.com

Personal working notes on data platforms, architecture, and strategy.

## Setup

```bash
brew install hugo
git clone <this-repo>
cd pretiosusdata
hugo server -D
```

Open http://localhost:1313

## Publishing a note

```bash
hugo new notes/my-topic.md
# edit content/notes/my-topic.md
# set draft: false when ready
git add . && git commit -m "note: my topic" && git push
```

## Publishing a reference

```bash
hugo new references/my-reference.md
# edit content/references/my-reference.md
# set draft: false when ready
git add . && git commit -m "ref: my reference" && git push
```

## Deploy

Connected to Cloudflare Pages. Push to `main` triggers auto-deploy.

Build command: `hugo`
Publish directory: `public`
