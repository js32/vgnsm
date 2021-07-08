---
title: "Womit ist diese Seite gebaut?"
date: 2021-03-07T16:20:28+01:00
draft: false
tags: ["tech", "hugo", "git"]
keywords:
  [
    "github",
    "static site generator",
    "hugo",
    "continuous deployment",
    "netlify",
  ]
description: "Ein Tutorial, wie dieser Blog mittels hugo, netflify und github gebaut wurde."
slug: "how-its-built"
sitemap_exclude: false
noindex: false
rss_unlisted:
---

Dieser Blog liegt in einem [öffentlichen Repository auf GitHub](https://github.com/js32/vgnsm) – warum nicht den ganzen Code zugänglich machen, wenn die Website sowieso öffentlich ist.

Als Static Site Generator benutze ich [hugo](https://gohugo.io) und als Theme kommt [varkais „Zozo“ Theme](https://github.com/varkai/hugo-theme-zozo) zum Einsatz. Hugo ist auf allen großen Plattformen verfügbar, ich nutze selbst einen Mac.

Wer mehr über die Vorzüge von Websites ohne Wordpress und Co. (also ohne Sicherheitslücken, ohne aufgeblähte Codebase und unüberschaubar viele Plugins) erfahren möchte, dem sei dieses [Youtube-Video](https://www.youtube.com/watch?v=YyRwMy59d4M&t=195s) vom Gründer von netlify ans Herz gelegt.

[netlify](https://www.netlify.com) kommt als Hoster zum Einsatz, dort wird auch SSL und DNS gemanagt.

Es folgt ein Tutorial für absolute Neulinge, wie ich es auch noch bin.

## Inhaltsverzeichnis

- [Voraussetzungen]({{< ref "posts/2021/tech/how-to.md#voraussetzungen" >}})
- [Installation]({{< ref "posts/2021/tech/how-to.md#installation" >}})
- [GitHub]({{< ref "posts/2021/tech/how-to.md#git" >}})
- [Neue hugo-Seite erstellen]({{< ref "posts/2021/tech/how-to.md#hugo-starten" >}})
- [Ein Theme installieren]({{< ref "posts/2021/tech/how-to.md#theme" >}})
- [Eine Beispiel-Website verwenden]({{< ref "posts/2021/tech/how-to.md#demo" >}})
- [hugo-Server starten]({{< ref "posts/2021/tech/how-to.md#init0" >}})
- [netlify mit git verbinden]({{< ref "posts/2021/tech/how-to.md#netlify" >}})
- [Repository updaten]({{< ref "posts/2021/tech/how-to.md#GitHub" >}})

### Wir benötigen {id="voraussetzungen"}

- ein Git-Repository (hier GitHub, es funktioniert aber auch mit GitLab oder Bitbucket)
- git-CLI
- einen netlify-Account
- einen Code-Editor, wie [Atom](https://github.com/atom/atom/releases/tag/v1.55.0) oder [VSCode](https://code.visualstudio.com/Download)
- [hugo](https://gohugo.io/getting-started/installing/)
- auf dem Mac zusätzlich [Homebrew](https://brew.sh/index_de)
- eine Domain – ich besorge diese über [INWX](https://www.inwx.com/en)

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
{{< img src="/static/images/git_1.png" width="738px">}}

Jetzt geben wir dem neuen Repository einen sinnvollen Namen, setzen noch den Haken bei README und klicken dann unten auf den Button „Create repository“:
{{< img src="/static/images/git_2.png" width="738px">}}

Im nächsten Screen können wir mit Klick auf den grünen Code-Button die URL für HTTPS sehen. Wir kopieren uns diese `https://github.com/js32/demo.git`:

{{< img src="/static/images/git_3.png" width="738px" title="">}}

Nun können wir wieder ins Terminal zurückwechseln und uns mittels folgendem Befehl einen neuen Ordner in unserem Benutzerverzeichnis erstellen – ich nennen ihn `src` für source/Quelle:

```sh
$ mkdir ~/src
```

Jetzt wechseln wir in den neu erstellten Ordner mittels

```bash
$ cd ~/src
```

und clonen das Repository, das wir vorher auf GitHub erstellt haben mit der kopierten URL (ihr setzt hier natürlich eure URL ein):

```bash
$ git clone https://github.com/js32/demo.git
```

Nun muss noch Benutzername und Passwort eingegeben werden und dann lädt das Repository runter. Wir prüfen, ob das Repo ordentlich runtergeladen wurde:

```bash
$ ls -alh
total 0
drwxr-xr-x   3 play  staff    96B 20 Mär 17:05 .
drwxr-xr-x@ 70 play  staff   2,2K 20 Mär 17:05 ..
drwxr-xr-x   4 play  staff   128B 20 Mär 17:05 demo
```

### Eine neue hugo-Seite erstellen {id="hugo-starten"}

Nun können wir ins Verzeichnis `demo` wechseln und eine neue Hugo-Seite erstellen:

```bash
$ cd ~/src/demo
```

```bash
$ hugo new site --force .
```

Ein erneutes

```bash
$ ls -alh
```

sollte uns folgendes zeigen:

```bash
total 16
drwxr-xr-x  11 play  staff   352B 20 Mär 17:08 .
drwxr-xr-x   3 play  staff    96B 20 Mär 17:05 ..
drwxr-xr-x  12 play  staff   384B 20 Mär 17:08 .git
-rw-r--r--   1 play  staff     6B 20 Mär 17:05 README.md
drwxr-xr-x   3 play  staff    96B 20 Mär 17:08 archetypes
-rw-r--r--   1 play  staff    82B 20 Mär 17:08 config.toml
drwxr-xr-x   2 play  staff    64B 20 Mär 17:08 content
drwxr-xr-x   2 play  staff    64B 20 Mär 17:08 data
drwxr-xr-x   2 play  staff    64B 20 Mär 17:08 layouts
drwxr-xr-x   2 play  staff    64B 20 Mär 17:08 static
drwxr-xr-x   2 play  staff    64B 20 Mär 17:08 themes
```

Gratulation – hugo ist installiert und unsere erste Seite kann konfiguriert und mit einem Theme ausgestattet werden!

### Ein Theme installieren {id="theme"}

Mittels

```bash
$ git clone https://github.com/varkai/hugo-theme-zozo themes/zozo
```

installieren wir den hier verwendeten Theme im Ordner `themes/zozo`. Es gibt noch viele anderen [interessante Themes](https://themes.gohugo.io), diese werden analog installiert.

### Beispiel-Website verwenden {id="demo"}

Im Ordner `~/src/demo/themes/zozo/exampleSite` befindet sich ein Beispiel-Inhalt einer Website. Wir können den gesamten Inhalt dieses Ordners im Finder in den Ordner `demo` kopieren und dabei die bestehenden Dateien und Ordner überschreiben.

{{< img src="/static/images/finder_1.png" width="738"  alt="" >}}

**Wichtig:** Wir müssen noch in der `config.toml` Datei in der ersten Zeile die Variable `baseURL` von `baseURL = "http://localhost:1313/"` auf `baseURL = "/"` ändern. Wenn diese URL falsch gesetzt ist, wird die Seite später nicht richtig gebaut.

### hugo server starten {id="init0"}

Wir können jetzt den Hugo-Server starten und uns unsere Website im Browser unter `localhost:1313` ansehen:

```bash
$ hugo server -D
```

### netlify mit git verbinden {id="netlify"}

Sofern noch nicht geschehen, machen wir uns einen [Account bei netlify](https://app.netlify.com/signup), am einfachsten ist es, wenn wir uns mit unserem GitHub-Account einloggen.

Über „New site from Git“ können wir netlify Zugriff auf unsere Repository gewähren.
Sobald er eingerichtet ist, lauscht netlify auf Veränderungen im Ordner und wird die Seite bauen, wenn wir Commits in das repository pushen.

Wir müssen noch eine weitere Datei im Ordner `demo` erzeugen:

```bash
$ touch netlify.toml
```

und fügen folgenden Inhalt ein:

```python
[build]
publish = "public"
command = "hugo --gc --minify"

[context.production.environment]
HUGO_VERSION = "0.81.0"
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"

[context.split1]
command = "hugo --gc --minify --enableGitInfo"

[context.split1.environment]
HUGO_VERSION = "0.80.0"
HUGO_ENV = "production"

[context.deploy-preview]
command = "hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL"

[context.deploy-preview.environment]
HUGO_VERSION = "0.80.0"

[context.branch-deploy]
command = "hugo --gc --minify -b $DEPLOY_PRIME_URL"

[context.branch-deploy.environment]
HUGO_VERSION = "0.80.0"

[context.next.environment]
HUGO_ENABLEGITINFO = "true"
```

### Repository updaten {id="GitHub"}

Zunächst fügen wir alle neuen Dateien zum Index hinzu:

```sh
$ git add *
```

Anschließend machen wir unser erstes Commit:

```bash
$ git commit -a -m "adding theme"
```

Jetzt pushen wir das Commit zu GitHub:

```bash
$ git push
```

netlify sollte nun anfangen, die Seit zu bauen und sie unter der zufällig generierten URL veröffentlichen.
Jetzt kann der Inhalt der hugo-Seite beliebig angepasst werden, netlify wird nach jedem Commit eine neue Variante bauen und veröffentlichen. Ich rate dennoch, die Website lokal im Browser zu testen, denn netlify bietet im kostenlosen Plan nur begrenzte Anzahl an Build-Minuten.
