---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
hidden: false                                   # seems to do nothing...
draft: true                                     # hides post from home
tags: []
keywords: []
description: ""
slug: ""
sitemap_exclude: false                          # don't publish in sitemap
noindex: false                                  # "true" tell google to not index
rss_unlisted:                                   # set to "true" to hide in rss feed, else leave blank.
---