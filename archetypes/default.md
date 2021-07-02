---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true                                    # hides post from home
tags: []
keywords: []
description: ""
slug: ""
sitemap_exclude: true                          # don't publish in sitemap
noindex: true                                  # "true" tell google to not index
rss_unlisted:                                  # set to "true" to hide in rss feed, else leave blank.
---