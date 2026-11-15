# pretiosusdata.com

Personal working notes on data platforms, architecture, and strategy.

## Setup

```bash
brew install hugo
git clone git@github-personal:ymdrobj/pretiosusdata.git
cd pretiosusdata
hugo server -D
```

Open http://localhost:1313

## Publishing a note

```bash
hugo new notes/my-topic.md
# edit content/notes/my-topic.md
# set draft: false when ready
hugo --minify
git add content/notes/my-topic.md
git diff --cached
git commit -m "note: my topic"
git push origin main
```

## Publishing a reference

```bash
hugo new references/my-reference.md
# edit content/references/my-reference.md
# set draft: false when ready
hugo --minify
git add content/references/my-reference.md
git diff --cached
git commit -m "ref: my reference"
git push origin main
```

## Deploy

Hosted on GitHub Pages at https://pretiosusdata.com/. Push to `main` triggers
`.github/workflows/hugo-deploy.yml`, which builds Hugo and deploys the result.

Build command: `hugo --minify`. Publish directory: `public` (not committed).

Before publishing, confirm that you are on `main`, the checkout is current,
and the article is approved. Preserve the approved text and source links.
Set `draft: false` and use the intended publication date; drafts and future-dated
articles are excluded from the production build. Stage only the intended source
files and any article assets.

After pushing, wait for the matching commit's run in
[GitHub Actions](https://github.com/ymdrobj/pretiosusdata/actions/workflows/hugo-deploy.yml)
to succeed, then open the article's live URL and check its content. A successful
push alone does not confirm publication. The same workflow also supports
**Run workflow** on `main` for a manual redeployment.

The Git remote uses the `github-personal` SSH alias for the `ymdrobj` account.
See `github-multi-account-setup.md` if that alias is not configured. Do not switch
to a client GitHub account when publishing this blog.
