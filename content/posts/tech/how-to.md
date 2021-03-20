---
title: "Womit ist diese Seite gebaut?"
date: 2021-03-07T16:20:28+01:00
hidden: true
draft: true
tags: ["tech", "hugo", "git"]
keywords: ["github", "static site generator", "hugo", "continuous deployment", "netlify"]
description: "Ein Tutorial, wie dieser Blog mittels hugo, netflify und github gebaut wurde."
slug: "how-its-built"
sitemap_exclude: false
noindex: false
---

Dieser Blog liegt in einem [öffentlichen Repository auf GitHub](https://github.com/js32/vgnsm) – warum nicht den ganzen Code zugänglich machen, wenn die Website sowieso öffentlich ist.

Als Static Site Generator benutze ich [hugo](https://gohugo.io) und als Theme kommt [varkais „Zozo“ Theme](https://github.com/varkai/hugo-theme-zozo) zum Einsatz. Hugo ist auf allen großen Plattformen verfügbar, ich nutze selbst einen Mac.

Wer mehr über die Vorzüge von Websites ohne Wordpress und Co. (also ohne Sicherheitslücken, ohne aufgeblähte Codebase und unüberschaubar viele Plugins) erfahren möchte, dem sei dieses [Youtube-Video](https://www.youtube.com/watch?v=YyRwMy59d4M&t=195s) vom Gründer von netlify ans Herz gelegt.

[netlify](https://www.netlify.com) kommt als Hoster zum Einsatz, dort wird auch SSL und DNS gemanagt.

## Inhaltsverzeichnis
- [Voraussetzungen]({{< ref "posts/tech/how-to.md#voraussetzungen" >}})
- [Installation]({{< ref "posts/tech/how-to.md#installation" >}})
- [GitHub]({{< ref "posts/tech/how-to.md#git" >}})



### Wir benötigen {id="voraussetzungen"}

* ein Git-Repository (hier GitHub, es funktioniert aber auch mit GitLab oder Bitbucket)
* git-CLI
* einen netlify-Account
* einen Code-Editor, wie [Atom](https://github.com/atom/atom/releases/tag/v1.55.0) oder [VSCode](https://code.visualstudio.com/Download)
* [hugo](https://gohugo.io/getting-started/installing/)
* auf dem Mac zusätzlich [Homebrew](https://brew.sh/index_de)
* eine Domain – ich besorge diese über [INWX](https://www.inwx.com/en)


### Installation {id="installation"}

Zunächst laden wir uns den Code-Editor (für das tutorial hier Atom) runter und installieren hugo, das geht auf dem Mac am besten mittels Homebrew.

Also zunächste Homebrew - wir wechseln ins Terminal und geben folgende Zeile ohne das vorangestellte `$`-Symbol ein – das `$` signalisiert nur, dass die Kommandos mit normalen Benutzerrechten ausgeführt werden:


```sh
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Über folgende Zeile installieren wir hugo – hugo steht danach in unserem `$PATH` zur Verfügung.

```sh
$ brew install hugo
```

Jetzt installieren wir uns noch git für die Kommandozeile – das geht am einfachsten, indem wir uns die Installationsdatei für den Mac einfach [runterladen](https://git-scm.com/download/mac) und installieren.

### Git-Repository {id="git"}

Jetzt müssen wir noch die Voraussetzungen schaffen, dass netlify unsere Seite hosten kann. netlify unterstützt zwar auch das manuelle Hochladen von Seiten, der Workflow über GitHub ist aber deutlich eleganter.

Wir loggen uns auf GitHub ein und generieren ein neues Repository, indem wir links auf den grünen Button „New“ klicken:
![Local Image](/static/images/git_1.png)
