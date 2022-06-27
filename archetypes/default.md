+++
title = "{{ replace .Name "-" " " | title }}"
date = {{ .Date }}
draft = false
author = ["{{ .Site.Author }}"]
categories = [""]
tags = [""]
archives = ["{{ dateFormat "2006" .Date }}", "{{ dateFormat "2006-01" .Date }}"]
comments = false
description = ""
url = "/{{ .Type }}/{{ .Name}}/"
+++
