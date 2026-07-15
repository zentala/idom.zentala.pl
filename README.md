# Intelligent Home Portal Project Documentation

## Overview

[This blog](https://idom.zentala.pl/) details my journey of transforming a 50m² apartment into a smart home using Bone.IO, a Polish smart home system. It covers the features implemented, preparations for integrating smart home technologies, and insights gained throughout the process. The aim is to guide others on their smart home adventures, discussing various topics from interior design projects, electrical choices, to selecting controllers, designing modern interiors, IoT design, components used, and considerations for apartment design, including recommended materials and their summaries.

## Technologies Used

Since the project transitioned from Jekyll to Hugo, here's an updated list of technologies:

| Category | Technology |
|---|---|
| Repo Conf | ![EditorConfig](https://img.shields.io/badge/-EditorConfig-FEFEFE?logo=editorconfig&logoColor=black) ![gitignore.io](https://img.shields.io/badge/-gitignore.io-204ECF?logo=gitignoredotio&logoColor=white) |
| Front-end     | ![Hugo](https://img.shields.io/badge/-Hugo-FF4088?logo=hugo&logoColor=white) ![Bootstrap](https://img.shields.io/badge/-Bootstrap-563D7C?logo=bootstrap&logoColor=white) ![SCSS](https://img.shields.io/badge/-SCSS-CC6699?logo=sass&logoColor=white) &nbsp;  |
| DevOps        | ![Cloudflare Workers](https://img.shields.io/badge/-CloudflareWorkers-F38020?logo=cloudflare&logoColor=white) ![SonarCloud](https://img.shields.io/badge/-SonarCloud-F3702A?logo=sonarcloud&logoColor=white) |
| Analytics     | ![Cloudflare](https://img.shields.io/badge/-CloudflareWebAnalytics-F38020?logo=cloudflare&logoColor=white) |
| IDE | ![Visual Studio Code](https://img.shields.io/badge/-VisualStudioCode-007ACC?logo=visualstudiocode&logoColor=white)                     |

Built with [Hugo](https://gohugo.io/), [Hyas](https://gethyas.com/), and [Doks](https://getdoks.org/) theme.

## Publishing

The site is a static-assets Cloudflare Worker on **[idom.zentala.pl](https://idom.zentala.pl/)**.
There is no CI — publish from your machine:

```bash
pnpm run deploy   # icons -> hugo (production) -> wrangler deploy
```

Needs `npx wrangler login` once. `baseurl` comes from `config/production/hugo.toml`.

**Math** is typeset in the browser by [KaTeX](https://katex.org/), bundled from `node_modules`
(mounted to `/katex/` in `config/_default/module.toml`). The theme's stock `math` shortcode fetches
pre-rendered images from `math.vercel.app` at *build* time; that API now returns HTTP 429 and took
the whole build down with it, so `layouts/shortcodes/math.html` and
`layouts/_default/_markup/render-codeblock-math.html` are overridden to emit plain LaTeX instead.
The build makes no remote calls.

**`sharp` on Windows:** `pnpm` skips the platform binaries by default. `pnpm.supportedArchitectures`
in `package.json` fixes it; if `pnpm run icons` ever fails with *"Could not load the sharp module"*,
run `pnpm install --force`.

## AI Development Ready

This project is configured for AI-assisted development:
- **Claude Code** - Development, deployment, debugging
- **Cursor IDE** - Content writing, blog posts, documentation
- Configurations: `.claude/CLAUDE.md` and `.cursor/rules/`

## Contributing

Want to contribute or set up a local development environment?

See **[CONTRIBUTING.md](CONTRIBUTING.md)** for:
- Development server setup
- Available commands
- VS Code configuration
- Deployment instructions
- Troubleshooting guide
