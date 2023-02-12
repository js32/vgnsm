---
date: 2021-03-08T10:19:29+01:00
title: "Impressum & Datenschutz"
hidden: false
draft: false
tags: []
keywords: []
description: ""
slug: "impressum-datenschutz"
noindex: false
sitemap_exclude: false
rss_unlisted: true
type: "datenschutz"
---

#### Datenschutz ist ein Menschenrecht – diese Seite respektiert deine Privatsphäre

Da die Seite nicht kommerziell betrieben wird, würde ich gern auch mein Recht auf Privatsphäre honorieren. Leider können die Posts nach § 5 TMG und § 55 RStV als redaktioneller Inhalt im Sinne des Rundfunkstaatsvertrages gesehen werden, daher folgt eine ladungsfähige Adresse:

> Jens Steger
> c/o Gloria Kison /
> TipTop Express
> Josefsstraße 36
> 55118 Mainz

Solltest du Wünsche, Anmerkungen oder Kritik loswerden wollen, so kannst du das per Mail an _infoATdieses-veganismusPUNKTde_ oder über das [Kontaktformular](/kontakt) tun.

#### Cookies & Tracking

Ich setze keine Cookies. ~~Zum Tracking verwende ich Google-Analytics, dabei wird deine Privatsphäre geschützt, indem IPs anonymisiert werden, DoNotTrack, sofern im Browser eingestellt, beachtet wird und keine Cookies gesetzt werden. Sobald ich eine andere Möglichkeit finde, kostenlos und ohne [EvilMegaCorp.Inc](https://analytics.google.com/) zu tracken, werde ich das entsprechend umbauen.~~

Neuerdings werte ich die Website statistisch mit [Plausible Analytics](https://plausible.io/privacy-focused-web-analytics) aus:

> By using Plausible Analytics, all the site measurement is carried out absolutely anonymously. Cookies are not set and no personal data is collected. All data is in aggregate only. The website owner gets some actionable data to help them learn and improve, while the visitor keeps having a nice and enjoyable experience.

Andere Dienste, wie z.B. Youtube- oder Vimeo-Videos werden sofern möglich ohne Javascript, also statisch, eingebunden und es werden mögliche Privatsphäre-Einstellungen gesetzt.

Die konkreten Einstellungen im Static Site Generator [hugo](https://gohugo.io/about/hugo-and-gdpr/) lauten:

```toml
[privacy]
  [privacy.disqus]
    disable = true
  [privacy.googleAnalytics]
    disable = false
    anonymizeIP = true
    respectDoNotTrack = true
    useSessionStorage = true
  [privacy.instagram]
    disable = true
    simple = true
  [privacy.twitter]
    disable = true
    enableDNT = true
    simple = true
    disableInlineCSS = true
  [privacy.vimeo]
    disable = false
    enableDNT = true
    simple = true
  [privacy.youtube]
    disable = false
    privacyEnhanced = true
```

Der Code meiner Website kann öffentlich auf [GitHub](https://github.com/js32/vgnsm-zozo) eingesehen werden.

#### Hosting

Mit jedem Zugriff auf diese Site werden durch den Hoster automatisch Informationen erfasst. Diese Informationen, auch als Server-Logfiles bezeichnet, sind allgemeiner Natur und erlauben keine persönlichen Rückschlüsse.

Die Website wird mittels [Netlify Inc.](https://www.netlify.com/gdpr-ccpa) gehostet:

> Netlify, Inc.
> San Francisco (HQ)
> 2325 3rd St #215
> San Francisco
> CA, United States

IP-Adressen werden für weniger als 30 Tage geloggt. Es werden keine anderen personenbezogenen Daten erfasst. 

<!-- Den entsprechenden AV-Vertrag / Data Processing Agreement gibt es [hier](https://dieses-veganismus.de/static/docs/netlify-dpa.pdf). -->

#### Kontaktformular

Wenn du das Kontaktformular nutzt, werden deine Daten per E-Mail an mich gesandt und bei mir gespeichert. E-Mails werden unverschlüsselt übertragen und können theoretisch von jedem vermittelnden Server automatisiert gescannt und gelesen werden. Ich gebe die Daten nicht weiter. Die Daten verbleiben bei mir, bis ich sie lösche – du kannst jederzeit das Löschen verlangen, indem du mich kontaktierst.

<!-- Diese Internetseite nutzt form.taxi, einen Webdienst der Webseite https://form.taxi (nachfolgend "form.taxi"). Um die Funktionalität des Formulars zur Verfügung zu stellen, sende ich die angegebenen Daten an form.taxi. Diese Daten werden dort verarbeitet und gespeichert. Außerdem werden von form.taxi unter anderem weitere Daten wie Ihre IP-Adresse,  Typ des Browsers, die Domain der Webseite, das Datum und die Zeit des Zugriffs erhoben, um die gewünschte Funktionalität des Formulars bereitzustellen. Rechtsgrundlage für die Nutzung von form.taxi ist Art. 6 Abs. 1 S. 1 lit. f DS-GVO (berechtigtes Interesse). Die Datenverarbeitung und Speicherung erfolgt innerhalb der Europäischen Union. Weitere Informationen findest du in der Datenschutzerklärung von form.taxi: https://form.taxi/de/privacy. -->

#### Die obligatorischen rechtlichen Hinweise

Alle Texte, Bilder und weitere hier veröffentlichte Informationen unterliegen meinem Urheberrecht, soweit nicht Urheberrechte Dritter bestehen. Meine Inhalte können gern vervielfältigt werden, du benötigst dafür keine Zustimmung von mir, sofern das Urheberrecht anderer nicht tangiert wird.

Für verlinkte Inhalte übernehme ich keine Verantwortung, da es sich nicht um meine Inhalte handelt – natürlich habe ich die Links zum Zeitpunkt des Schreibens der Artikel auf illegale und sonstige faschistoiden und menschenverachtenden Inhalte geprüft, spätere Änderungen kann ich jedoch nicht ausschließen. Sollte es dir oder mir auffallen, dass Rechtsverletzungen bestehen oder Links anstößig sind, werde ich die Links umgehend entfernen.

#### Und übrigens

{{< img src="https://ptrace.gafam.info/unofficial/img/color/lqdn-gafam-poster-en-color-5x1-2560x.png" class="img-100c" alt="The GAFAM poster campaign toolkit" title="Quelle: https://ptrace.gafam.info/">}}
