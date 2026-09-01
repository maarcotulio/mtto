# M.T.T.O — Hugo + Blowfish blog

Bilingual site (PT-BR and EN) built with [Hugo](https://gohugo.io) and the
[Blowfish](https://blowfish.page) theme.

## Requirements

You need Hugo installed (the **extended** build, 0.164.0 — the same version
the Netlify build uses):

```
# Linux (example)
wget https://github.com/gohugoio/hugo/releases/download/v0.164.0/hugo_extended_0.164.0_linux-amd64.tar.gz
tar -xzf hugo_extended_0.164.0_linux-amd64.tar.gz hugo
sudo mv hugo /usr/local/bin/hugo
```

On Mac: `brew install hugo`
On Windows: `choco install hugo-extended` or download the `.zip` from
https://github.com/gohugoio/hugo/releases

## Run locally (with live reload)

From the project root:

```
hugo server
```

Opens at `http://localhost:1313`. Any change to a `.md` file reloads the
page automatically.

## Structure

```
config/_default/       <- all site and theme configuration
  hugo.toml             (title, default language, theme)
  languages.pt-br.toml  (Portuguese-specific config)
  languages.en.toml     (English-specific config)
  menus.pt-br.toml / menus.en.toml   (navigation menu)
  params.toml           (theme options: homepage, article, etc.)
content/posts/          <- your posts live here
themes/blowfish/        <- the theme (don't touch this)
```

## How to write a new post

Each post needs **two files**, one per language, sharing the same base name:

```
content/posts/post-name.pt-br.md
content/posts/post-name.en.md
```

Front matter for each one:

```markdown
---
title: "Post title"
date: 2026-08-20
summary: "One-line summary, shown in the post list."
tags: ["ai", "market"]
---

Post body in plain markdown.
```

If for now you only want to write in Portuguese, you can leave out the
`.en.md` version. Hugo just won't generate that post on the English side of
the site, and everything else keeps working.

## Deploy

Deployment is already set up through `netlify.toml`: just connect this
repository to a site on Netlify. The build (`hugo --gc --minify`, version
0.164.0) runs automatically on every push, with no need to generate or
upload the `public/` folder by hand.

## Main customizations

| What to change | Where |
|---|---|
| Author name, bio, headline | `config/_default/languages.pt-br.toml` and `languages.en.toml`, `[params.author]` section |
| Theme color | `params.toml` → `colorScheme` (options: `blowfish`, `avocado`, `fire`, `ocean`, `forest`, `princess`, `neon`, `slate`) |
| Default light/dark mode | `params.toml` → `defaultAppearance` |
| Homepage layout | `params.toml` → `[homepage] layout` (`profile`, `page`, `hero`, `card`, `background`) |
| Top menu | `menus.pt-br.toml` / `menus.en.toml` |
| Real domain | `config/_default/hugo.toml` → `baseURL` |

Full theme documentation, with every option: https://blowfish.page/docs/
