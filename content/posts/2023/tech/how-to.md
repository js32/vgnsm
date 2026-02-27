---
title: "Kontaktformulare versenden mit MailyGo bei Hugo (Static) Sites"
date: 2023-02-12T13:10:28+01:00
draft: false
tags: ["tech", "hugo", "tutorial"]
keywords:
  [
    "formspree",
    "static site generator",
    "hugo",
    "forms",
    "mailygo",
    "self hosted",
    "JAMstack",
    "SSG",
    "Static Site Generator",
  ]
description: "Erfahre, wie du Kontaktformulare DSGVO-konform mit MailyGo versendest."
slug: "kontaktform-submissions-mit-mailygo"
sitemap_exclude: false
noindex: false
rss_unlisted:
---

## Fragestellung {id="fragestellung"}

Wie Kontaktformulare datenschutzkonform per E-Mail übermitteln?

## Problem {id="problem"}

Statisch gehostete Seiten unterstützen in der Regel keine serverseitigen Sprachen, wie PHP, mit denen sich ein Mailer für Kontaktformulare realisieren ließe (like [so](https://stackoverflow.com/questions/18379238/send-email-with-php-from-html-form-on-submit-with-the-same-script)).

Daher sind wir auf Dienste, wie z.B. formspree.io (eine gute Liste gibt es [hier](https://css-tricks.com/a-comparison-of-static-form-providers/)), angewiesen. Die meisten dieser Dienste sind allerdings nicht DSGVO-konform, da die Daten in unsichere Drittstaaten (looking at you, US of A) übertragen werden. Ich selbst nutze Netlify zum Hosten dieser Seite und habe auch in der Vergangenheit deren an sich ganz tollen und für meine Zwecke kostenlosen [Dienst für Kontaktformulare](https://www.netlify.com/products/forms/) eingesetzt. Netlify sitzt leider aber auch außerhalb der EU und hat daher mit den Formularübermittlungen besser nichts zu schaffen. Bliebe noch der in Österreich sitzende Dienst [form.taxi](https://form.taxi/de), der für maximal 40 übermittelte Formulare auch kostenlos ist. Mir ist form.taxi sehr sympathisch und ich komme eventuell für Kundenprojekte darauf zurück.

Ich ziehe es jedoch vor, bei übermittelten Daten so wenige Mittelsmenschen wie möglich zu haben und da ich sowieso einen VServer bei [Contabo](https://contabo.com/de/vps/) angemietet habe, war die Überlegung, ob sich das auch darüber realisieren ließe.

## Lösung {id="loesung"}

Stellt sich raus: ließe sich! Wie der Zufall so will, bin ich beim Recherchieren nach Lösungen auf [MailyGo](https://codeberg.org/jlelse/MailyGo) von Jan-Lukas Else gestoßen. Es handelt sich um eine Go-binary, die als Endpunkt für POST-Requests aus Formularen dient. Das Programm versendet übermittelte Daten per Mail, ohne dass die Daten noch irgendwo zwischengespeichert werden – perfekt für meine Anforderungen.

## Umsetzung {id="Umsetzung"}

Im folgenden beschreibe ich euch, wie ihr MailyGo installiert und startet. Außerdem beschreibe ich mein Setup, bei dem ich MailyGo mehrmals mit unterschiedlichen Benutzern per systemd-Unit starte und unter verschiedenen Domain-Unterverzeichnissen zur Verfügung stelle, damit ich mehrerer Formulare auf unterschiedlichen Seiten realisiert bekomme.

Ganz unten gibt es eine aktualisierte, deitlich einfachere Version mittels [Docker]({{< ref "#docker" >}}).

### Inhalt

- [Voraussetzungen]({{< ref "#voraussetzungen" >}})
- [Installation von Golang]({{< ref "#golanginstallation" >}})
- [Installation von MailyGo]({{< ref "#mailygoinstallation" >}})
- [Test]({{< ref "#test" >}})
- [Reverse Proxy mit nginx]({{< ref "#nginx" >}})
- [SSL mit Certbot]({{< ref "#certbot" >}})
- [Starten von mailygo mittels systemd]({{< ref "#systemd" >}})
- [Environment Variables über env-Datei]({{< ref "#env" >}})
- [Das Kontaktformular]({{< ref "#form" >}})
- [Mehrere Prozesse für unterschiedliche Domains]({{< ref "#multi" >}})
- [Docker]({{< ref "#docker" >}})

## Voraussetzungen {id="voraussetzungen"}

- eine freie (Sub-)Domain, die ihr kontrolliert und bei der ihr DNS-Einträge setzen könnt
- einen Linux-VServer mit Root-Zugriff, bei mir eine Ubuntu 20.04.5 LTS Installation
- Einen E-Mail-Account, idealerweise richtet ihr euch einen ein, der nur zum Versand der Kontaktformulare dient
- eine Website mit Kontaktformular ;)

## Installation von Golang {id="golanginstallation"}

Die Installation über Apt hat bei mir nicht funktioniert, weil keine aktuelle Version von Golang zur Verfügung stand und MailyGo damit nicht lief, daher folgt die manuelle Installation – ich folge im Wesentlichen einem Tutorial von [Digitalocean](https://www.digitalocean.com/community/tutorials/how-to-install-go-on-ubuntu-20-04).

Zunächst loggen wir uns auf dem Server mittels SSH ein:

```
ssh benutzer@ip_deines_vservers
```

Wir wechseln ins Homeverzeichnis:

```
cd ~
```

und laden uns die neueste Version von Golang runter:

```
curl -OL https://go.dev/dl/go1.20.linux-amd64.tar.gz
```

und entpacken das Archiv in `/usr/local`:

```
sudo tar -C /usr/local -xvf go1.20.linux-amd64.tar.gz
```

### Pfade setzen

Damit go gefunden werden kann, geben wir noch den Pfad der binary bekannt und fügen ihn dafür in unsere `.profile`-Datei hinzu:

```
sudo nano ~/.profile
```

Hier setzen wir folgende Zeile am Ende der Datei ein:

`export PATH=$PATH:/usr/local/go/bin` und schließen/speichern mit Ctrl+x, y.

Jetzt müssen wir die aktualisierte .profile-Datei noch neu einlesen mit:

```
source ~/.profile
```

Das war's – mit `$ go version` können wir prüfen, ob alles geklappt hat.

Tada:

`go version go1.20 linux/amd64`

## Installation von MailyGo {id="mailygoinstallation"}

In meinem Setup nutze ich nicht die Originalversion von MailyGo, sondern einen etwas erweiterten [Fork](https://codeberg.org/mzch/mailygo) von Koichi Matsumoto.

### Kurze, aktuelle Version

```
go install codeberg.org/js32/mailygo@v1.0.0
```

Die Binary liegt dann unter ~/go/bin/mailygo

### lange, alte Version

Ich bevorzuge manuell installierte Pakete in `/opt/` zu installieren, daher wechseln wir zunächst in dieses Verzeichnis:

```
cd /opt/
```

Jetzt klonen wir das Git-Verzeichnis mittels:

```
git clone https://codeberg.org/mzch/mailygo.git
```

und wechseln in das neue Verzeichnis:

```
cd mailygo/
```

Ein:

```
go build ./
```

kompiliert uns die eigentliche Application `mailygo`. Ich belasse sie in diesem Verzeichnis und start sie im Folgenden über den vollen Pfad `/opt/mailygo/mailygo`.

### Erster Test {id="test"}

mailygo müsst ihr bei jedem Start **environment variables** mitgeben, derer da wären:

| Name                           | Type     | Default value     | Usage                                                                                                  |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------------------------------------------------ |
| **`SMTP_USER`**                | required | -                 | The SMTP user                                                                                          |
| **`SMTP_PASS`**                | required | -                 | The SMTP password                                                                                      |
| **`SMTP_HOST`**                | required | -                 | The SMTP host                                                                                          |
| **`SMTP_PORT`**                | optional | 587               | The SMTP port                                                                                          |
| **`USE_STARTTLS`**             | optional | `true`            | SMTP connection with STARTTLS                                                                          |
| **`SMTP_HELO`**                | optional | `localhost`       | SMTP HELO hostname                                                                                     |
| **`EMAIL_FROM`**               | required | -                 | The sender mail address                                                                                |
| **`EMAIL_TO`**                 | required | -                 | Default recipient                                                                                      |
| **`ALLOWED_TO`**               | required | -                 | All allowed recipients (separated by `,`)                                                              |
| **`PORT`**                     | optional | `8080`            | The port on which the server should listen                                                             |
| **`HONEYPOTS`**                | optional | `_t_email`        | Honeypot form fields (separated by `,`)                                                                |
| **`GOOGLE_API_KEY`**           | optional | -                 | Google API Key for the [Google Safe Browsing API](https://developers.google.com/safe-browsing/v4/)     |
| **`SPAMLIST`**                 | optional | `gambling,casino` | List of spam words                                                                                     |
| **`DENYLIST`**                 | optional | `submit`          | List of fields names to deny                                                                           |
| **`TOKEN`**                    | optional | -                 | A token to identify the origin of the submission                                                       |
| **`MESSAGE_HEADER`**           | optional | -                 | Text to appear at the beginning of the email message, before the list of fields                        |
| **`MESSAGE_FOOTER`**           | optional | -                 | Text to appear at the end of the email message, after the list of fields                               |
| **`MESSAGE_SUBMITTER`**        | optional | `false`           | If set to `true` and the form submitter provide an email address, a copy of the message is sent to him |
| **`MESSAGE_SUBMITTER_HEADER`** | optional | -                 | Text to appear at the beginning of the email message sent to submitter, before the list of fields      |
| **`MESSAGE_SUBMITTER_FOOTER`** | optional | -                 | Text to appear at the end of the email message sent to submitter, after the list of fields             |

Die komplette Readme-Datei findet ihr [hier](https://codeberg.org/mzch/mailygo/src/branch/main/README.md).

Probieren wir das gleich mal aus. Da wir uns aktuell im richtigen Verzeichnis `/opt/mailygo/` befinden, können wir die binary mit folgendem Befehl starten (setzt dabei natürlich eure Daten für euren verwendeten Mail-Account ein):

```
SMTP_HOST=mail.host.com PORT=587 SMTP_PASS=yourpassword SMTP_USER=user@host.com USE_STARTTLS=true EMAIL_FROM=user@host.com EMAIL_TO=otheruser@otherhost.com ALLOWED_TO=otheruser@otherhost.com PORT=8000 ./mailygo
```

Obiger Befehl startet mailygo - der Dienst lauscht wie unter `PORT` angegeben auf Port 8000. Sofern ihr keine Firewall einsetzt (living on the edge!), ist der Dienst jetzt schon unter der öffentlichen IP eures Servers erreichbar.

Falls ihr doch eine Firewall laufen habt, was ich hoffe, müsst ihr den Port zunächst öffnen. Ich setze ufw ein, da geht das so:

```
sudo ufw allow 8000
```

Prüfen wir das mal. Mit:

`$ curl icanhazip.com` erfahren wir unsere Public IP-Adresse. Wenn wir diese im Browser, gefolgt von `:8000`, einsetzen, sollte er uns ein lakonisches **MailyGo works!** zurückmelden.

Außerdem könnt ihr prüfen, ob mailygo ordentlich lauscht:

```
lsof -i TCP| fgrep mailygo
```

Was etwas wie

```
mailygo   762851           euerUser    3u  IPv6 3049673      0t0  TCP *:8000 (LISTEN)
```

zurückwerfen sollte. Perfekt!

Jetzt löschen wir wieder die Portfreigabe der Firewall, da wir einen Reverse Proxy (nginx) vor mailygo setzen, der den internen Port unter unserer Domain weiterleitet:

```
sudo ufw delete allow 8000
```

## Reverse Proxy mit nginx {id="nginx"}

Damit wir in unserem Kontaktformular nicht die IP:Port-Kombination unserer mailygo-Installation angeben müssen und damit in den Genuss von SSL für unsere POST-Operation kommen, müssen wir einen virtuellen Proxy vor den Dienst setzen. Ich gehe hier davon aus, dass ihr nginx bereits laufen habt – sonst würde das den Rahmen sprengen.

Ich habe mich dafür entschieden, jeweils Unterverzeichnisse meiner Domain für die unterschiedlichen mailygo-Dienste zu verwenden. So erspare ich mir, für jede neue Website eine eigene Subdomain anlegen zu müssen und kann stattdessen einen beliebigen Unterordner meiner Domain setzen. mailygo läuft dabei unter verschiedenen Benutzern und öffnet pro Instanz einen anderen Port, den ich dann dort zur Verfügung stelle. Das klingt jetzt vielleicht komplizierter, als es ist. Und das war auch genau das Problem bei meinen Google-Suchen. Ihr habt dafür jetzt diesen Post, falls ihr ein ähnliches Problem habt ;)

Ihr erstellt euch also zunächst bei eurem Domain-Anbieter im DNS-Panel eine Subdomain (A-Record) für euren Dienst, der auf euren Server zeigt also etwas wie `mailygo.eure-domain.de`.

Dann wechselt ihr auf eurem Sever in das Verzeichnis `/etc/nginx/sites-available/` und erzeugt gleichzeitig eine neue nginx-Konfiguration:

```
cd /etc/nginx/sites-available/ && nano mailygo
```

Im neuen Editor-Fenster fügt ihr folgendes für eure neue Domain `mailygo.eure-domain.de` ein:

```
server {
    server_name mailygo.eure-domain.de;
        location /eurewebsite {
                proxy_pass http://localhost:8000;
                proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```

`eurewebsite` kann beliebig heißen, ich nehme dafür eine Kurzbezeichnung meiner Websites, hier wäre das `vgnsm`. Achtet drauf, dass ihr den richtigen Port in Zeile 4 eingebt (wir starten mailygo ja mit Port 8000).

Jetzt erzeugen wir noch einen Symlink, damit die neue Konfig auch von nginx ausgeführt wird:

```
ln -s /etc/nginx/sites-available/mailygo /etc/nginx/sites-enabled/mailygo
```

Startet den Service neu mit:

```
sudo systemctl restart nginx
```

### SSL mit Certbot {id="certbot"}

Ich empfehle ja immer den Einsatz von letsencrypt für euer SSL. Wenn ihr das benutzen wollt, könnt ihr euch ein Zertifikat holen und das installieren:

```
sudo certbot --nginx
```

Danach sieht eure nginx-Konfig so aus:

```
server {
    server_name mailygo.eure-domain.de;
        location /eurewebsite {
                proxy_pass http://localhost:8000;
                proxy_set_header X-Forwarded-For $remote_addr;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/mailygo.eure-domain.de/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/mailygo.eure-domain.de/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {

    if ($host = mailygo.eure-domain.de) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    server_name mailygo.eure-domain.de;
    listen 80;
    return 404; # managed by Certbot

}
```

Das war's zunächst für nginx. Startet den Service nochmals neu mit:

```
sudo systemctl restart nginx
```

## Starten von mailygo mittels systemd {id="systemd"}

Damit wir mailygo automatisiert laufen lassen können, macht es Sinn, diese Aufgabe systemd zu übergeben. Aber zunächst erzeugen wir uns einen neuen Benutzer, unter dem mailygo für diese Webiste laufen soll:

```
sudo useradd eurewebsite
```

Ich wähle als Benutzername wiederum den Kurznamen für eure Website, den wir weiter oben in den nginx-Konfig schon verwendet haben, aber ihr könnt auch einfach „Thomas“ nehmen :D

Jetzt erzeugen wir uns eine neue systemd-Unit.

```
sudo nano /etc/systemd/system/mailygo.eure-domain.de
```

und fügen dort folgendes ein:

```
[Unit]
Description = runs mailygo for mailygo.eure-domain.de

[Service]
Type = simple
User = eurewebsite
Group = eurewebsite
ExecStart = /opt/mailygo/mailygo
Restart=always
StandardOutput=syslog
EnvironmentFile=/opt/mailygo/env/eurewebsite

[Install]
WantedBy=multi-user.target
```

Bei User, Group und EnvironmentFile müsst ihr noch eure passenden Namen einsetzen. Das EnvironmentFile (env) dient uns dazu, die Environment Variables übersichtlich beim Startbefehl mitgeben zu können. Auch diese Datei können wir pro laufenden Dienst hinterlegen und entsprechend anpassen.

## Environment Variables über env-Datei {id="env"}

Zunächst wechseln wir wieder ins Verzeichnis der mailygo-Installation und erzeugen dort einen neuen Ordner `env`:

```
cd /opt/mailygo/ && mkdir ./env
```

Im neuen Oder `env` erstellen wir mit nano eine neue Datei `eurewebsite` mit folgendem Inhalt:

```
SMTP_HOST=euer_Mailserver
USE_STARTTLS=true (oder false)
PORT=587 (euer Port)
SMTP_USER=Benutzername
SMTP_PASS=2Passwort
EMAIL_FROM=absender@mailserver.com
EMAIL_TO=empfänger@mailserver.com
ALLOWED_TO=empfänger@mailserver.com
PORT=8000 (Port, auf dem mailygo lauschen soll)
MESSAGE_HEADER="Ein Besucher hat folgende Nachricht hinterlassen"
MESSAGE_FOOTER="Diese Mail wurde mittels mailygo gesandt"
TOKEN=qdhiofh4t5d2lkehwc323 (Beliebige Zeichenkette)
SPAMLIST=gambling,casino,lottery
HONEYPOTS=_t_email
```

Schaut euch die [Readme](https://codeberg.org/mzch/mailygo/src/branch/main/README.md) zur Erläuterung an.

Die Unit ist ready, sobald wir folgende beiden Befehle ausgeführt haben:

```
sudo systemctl daemon-reload && sudo systemctl restart mailygo.eure-domain.de.service

```

Wenn ihr nun auf mailygo.eure-domain.de/eurewebsite surft, sollte wieder das bekannte lakonische „MailyGo works!“ zurückkommen. Super!

## Das Kontaktformular {id="form"}

Das Repository enthält eine [Beispiel-Datei](https://codeberg.org/mzch/mailygo/src/branch/main/form.html) für ein Kontaktformular, das ihr beliebig anpassen könnt:

```
<form action="//localhost:8080" method="post">
  <!-- <input type="hidden" name="_to" value="test@example.com" /> -->
  <input type="hidden" name="_formName" value="Test Form" />
  <input type="hidden" name="_redirectTo" value="https://example.com" />
  <input type="hidden" name="_token" value="a3B2c1" />
  <label for="TEmail" style="display: none;">Test</label>
  <input id="TEmail" type="email" name="_t_email" style="display: none;" />
  <label for="Email">Email</label>
  <br />
  <input id="Email" type="email" name="_replyTo" required />
  <br />
  <label for="Name">Name</label>
  <br />
  <input id="Name" type="text" name="_name" required />
  <br />
  <label for="Subject">Subject</label>
  <br />
  <input id="Subject" type="text" name="_subject" required />
  <br />
  <label for="Message">Message</label>
  <br />
  <textarea id="Message" name="_message"></textarea>
  <br />
  <label for="Website">Website</label>
  <br />
  <input id="Website" type="text" name="Website" required />
  <br />
  <input type="submit" value="Send" />
</form>
```

Ihr müsst nur die erste Zeile wie folgt anpassen:

````
<form action="https://mailygo.eure-domain.de/eurewebsite" method="post">
````

Die zweite Zeile `<input type="hidden" name="_to" value="test@example.com" />` habe ich bei meinen Formularen rausgeschmissen, damit meine Mail-Adresse nicht von Spambots gefressen wird. Wir haben den Empfänger ja in unserem env-File bereits hinterlegt:

```
EMAIL_TO=empfänger@mailserver.com
ALLOWED_TO=empfänger@mailserver.com
```

Das war's – mailygo läuft nun mit env-File unter dem User „eurewebsite“ und der Port 8000 wird mittels nginx unter einem Unterverzeichnis eurer Subdomain vermittelt.

## Mehrere Prozesse für unterschiedliche Domains {id="multi"}

Wir haben jetzt die Voraussetzungen geschaffen, mailygo für verschiedene Domains/Websites zur Verfügung zu stellen.

Um eine zweite Instanz von mailygo laufen zu lassen, müssen wir nur einen weiteren Benutzer erzeugen (siehe „Starten von mailygo mittels systemd“) und für diesen eine neue Unit anlegen mit den entsprechenden Einträgen für User, Group und EnvironmentFile.

Der neue env-File muss einen anderen Port enthalten, zum Beispiel 8001, sonst kann mailygo nicht noch einmal gestartet werden, da der Port schon belegt ist.

Anschließend wechseln wir wieder in unserer nginx-Konfig und fügen dort einen neuen Eintrag für das neue Unterverzeichnis `eureanderewebsite` (Zeilen 8 und 10) ein:

````
server {
    server_name mailygo.eure-domain.de;
        location /eurewebsite {
                proxy_pass http://localhost:8000;
                proxy_set_header X-Forwarded-For $remote_addr;
    }
<!-- zweiter Eintrag für weiteren mailygo-Prozess! -->
        location /eureanderewebsite {
<!-- neuer Port 8001! -->
                proxy_pass http://localhost:8001;
                proxy_set_header X-Forwarded-For $remote_addr;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/mailygo.eure-domain.de/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/mailygo.eure-domain.de/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {

    if ($host = mailygo.eure-domain.de) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    server_name mailygo.eure-domain.de;
    listen 80;
    return 404; # managed by Certbot

}

````

In eurem Formular auf der anderen Website ändert ihr wieder die Zeile:

`<form action="//localhost:8080" method="post">`

zu

`<form action="https://mailygo.eure-domain.de/eureanderewebsite" method="post">`

und voilà.

## Alternative: Docker-Deployment {id="docker"}

  Wer Go nicht manuell installieren und keine systemd-Units verwalten möchte, kann mailygo auch per Docker betreiben. Das hat noch einen weiteren Vorteil: **Eine einzige Instanz
  genügt für beliebig viele Websites** – kein separater Prozess und kein separater Port pro Site.

### Inhalt

- [Voraussetzungen](#docker-voraussetzungen)
- [Repository klonen und Image bauen](#docker-build)
- [env-Datei anlegen](#docker-env)
- [Container starten](#docker-start)
- [nginx konfigurieren](#docker-nginx)
- [Mehrere Domains mit einer Instanz](#docker-multi)

### Voraussetzungen {id="docker-voraussetzungen"}

- Docker und Docker Compose auf dem Server installiert
- nginx läuft bereits (wie oben beschrieben)
- eine oder mehrere (Sub-)Domains, die ihr kontrolliert

### Repository klonen und Image bauen {id="docker-build"}

  ```
  cd /opt/
  git clone https://codeberg.org/js32/mailygo.git
  cd mailygo/

  Das mitgelieferte Dockerfile nutzt einen zweistufigen Build: Zunächst wird die Go-Binary kompiliert, anschließend landet nur diese in einem schlanken Alpine-Image. Kein Go auf
  dem Server notwendig.

  env-Datei anlegen {id="docker-env"}

  Im Verzeichnis /opt/mailygo/ liegt eine .env.example-Datei als Vorlage. Kopiert sie und befüllt sie mit euren Daten:

  cp .env.example .env
  nano .env

  Eine ausgefüllte .env sieht etwa so aus:

  PORT=8080

  EMAIL_TO=kontakt@meinewebsite.de
  ALLOWED_TO=kontakt@meinewebsite.de,info@anderewebsite.de

  EMAIL_FROM=forms@meinserver.de

  SMTP_HOST=mail.meinserver.de
  SMTP_PORT=587
  SMTP_USER=forms@meinserver.de
  SMTP_PASS=geheimespasswort
  USE_STARTTLS=true

  TOKEN=zufaellige-zeichenkette-hier
  HONEYPOTS=_t_email
  SPAMLIST=gambling,casino
  DENYLIST=submit

  Der entscheidende Unterschied zur systemd-Variante: In ALLOWED_TO tragt ihr alle Empfängeradressen aller eurer Websites ein, kommagetrennt. Die Weiterleitung an die richtige
  Adresse regelt dann das Formular (siehe #docker-multi).

  Container starten {id="docker-start"}

  Das mitgelieferte docker-compose.yml baut das Image lokal und startet den Container:

  services:
    mailygo:
      build: .
      image: mailygo:latest
      restart: always
      env_file: .env
      ports:
        - "127.0.0.1:8080:8080"

  Der Port ist bewusst nur an 127.0.0.1 gebunden – von außen nicht direkt erreichbar, nginx leitet weiter.

  Starten:

  docker compose up -d

  Prüfen, ob alles läuft:

  docker compose ps
  docker compose logs

  Updates sind denkbar einfach:

  git pull
  docker compose up -d --build

  nginx konfigurieren {id="docker-nginx"}

  Für Docker empfiehlt sich, jede Domain direkt auf den root-Pfad zu mappen, statt Unterverzeichnisse zu verwenden. Erstellt oder erweitert eure nginx-Konfiguration in
  /etc/nginx/sites-available/mailygo:

  server {
      server_name forms.eure-domain.de;

      location / {
          proxy_pass http://127.0.0.1:8080;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }

  SSL mit Certbot holen und nginx neustarten:

  sudo certbot --nginx
  sudo systemctl restart nginx

  Wenn ihr jetzt https://forms.eure-domain.de aufruft, sollte wieder das vertraute MailyGo works! erscheinen.

  Mehrere Domains mit einer Instanz {id="docker-multi"}

  Hier liegt der größte Unterschied zur systemd-Variante: Ihr müsst keine zweite mailygo-Instanz starten. Stattdessen verwendet ihr das versteckte Formularfeld _to, um den
  Empfänger pro Website festzulegen:

  Website 1:

  <form action="https://forms.eure-domain.de" method="post">
    <input type="hidden" name="_to" value="kontakt@meinewebsite.de" />
    <input type="hidden" name="_token" value="zufaellige-zeichenkette-hier" />
    <!-- weitere Felder ... -->
  </form>

  Website 2:

  <form action="https://forms.eure-domain.de" method="post">
    <input type="hidden" name="_to" value="info@anderewebsite.de" />
    <input type="hidden" name="_token" value="zufaellige-zeichenkette-hier" />
    <!-- weitere Felder ... -->
  </form>

  Voraussetzung: Die jeweilige Adresse muss in ALLOWED_TO eurer .env eingetragen sein. Ist das nicht der Fall, lehnt mailygo die Übermittlung ab.

  Für eine weitere nginx-Domain legt ihr einfach einen weiteren server-Block an – alle zeigen auf denselben Port 8080:

  server {
      server_name forms.andere-domain.de;
      location / {
          proxy_pass http://127.0.0.1:8080;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```

  Fertig – ein Prozess, beliebig viele Domains. Subba!
