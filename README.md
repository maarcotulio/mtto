# M.T.T.O — blog em Hugo + Blowfish

Site bilíngue (PT-BR e EN) montado com [Hugo](https://gohugo.io) e o tema
[Blowfish](https://blowfish.page).

## Requisitos

Você precisa do Hugo instalado (versão **extended**, 0.164.0 — a mesma usada no build da Netlify):

```
# Linux (exemplo)
wget https://github.com/gohugoio/hugo/releases/download/v0.164.0/hugo_extended_0.164.0_linux-amd64.tar.gz
tar -xzf hugo_extended_0.164.0_linux-amd64.tar.gz hugo
sudo mv hugo /usr/local/bin/hugo
```

Pra Mac: `brew install hugo`
Pra Windows: `choco install hugo-extended` ou baixe o `.zip` em
https://github.com/gohugoio/hugo/releases

## Rodar localmente (com live-reload)

Na raiz do projeto:

```
hugo server
```

Abre em `http://localhost:1313`. Qualquer alteração num arquivo `.md`
atualiza a página sozinha.

## Estrutura

```
config/_default/       <- todas as configurações do site e do tema
  hugo.toml             (título, idioma padrão, tema)
  languages.pt-br.toml  (config específica do português)
  languages.en.toml     (config específica do inglês)
  menus.pt-br.toml / menus.en.toml   (menu de navegação)
  params.toml           (opções do tema: homepage, artigo, etc.)
content/posts/          <- seus posts ficam aqui
themes/blowfish/        <- o tema (não mexer aqui)
```

## Como escrever um post novo

Cada post precisa de **dois arquivos**, um por idioma, com o mesmo nome base:

```
content/posts/nome-do-post.pt-br.md
content/posts/nome-do-post.en.md
```

Cabeçalho de cada um:

```markdown
---
title: "Título do post"
date: 2026-08-20
summary: "Resumo de uma linha, aparece na listagem."
tags: ["ia", "mercado"]
---

Corpo do post em markdown normal.
```

Se por enquanto você só quiser escrever em português, pode omitir a versão
`.en.md` — o Hugo simplesmente não vai gerar aquele post na versão em
inglês do site, o resto funciona normalmente.

## Deploy

O deploy já está configurado via `netlify.toml`: basta conectar este
repositório a um site na Netlify. O build (`hugo --gc --minify`, versão
0.164.0) roda automaticamente a cada push, sem precisar gerar ou subir a
pasta `public/` manualmente.

## Personalizações principais

| O que mudar | Onde |
|---|---|
| Nome, bio, headline do autor | `config/_default/languages.pt-br.toml` e `languages.en.toml`, seção `[params.author]` |
| Cor do tema | `params.toml` → `colorScheme` (opções: `blowfish`, `avocado`, `fire`, `ocean`, `forest`, `princess`, `neon`, `slate`) |
| Modo claro/escuro padrão | `params.toml` → `defaultAppearance` |
| Layout da home | `params.toml` → `[homepage] layout` (`profile`, `page`, `hero`, `card`, `background`) |
| Menu do topo | `menus.pt-br.toml` / `menus.en.toml` |
| Domínio real | `config/_default/hugo.toml` → `baseURL` |

A documentação completa do tema, com todas as opções: https://blowfish.page/docs/
