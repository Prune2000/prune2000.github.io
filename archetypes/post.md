{{/*
Copyright (C) 2019  Josh Habdas <jhabdas@protonmail.com>

This file is part of After Dark.

After Dark is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

After Dark is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
*/ -}}

+++
title = "{{ replace .TranslationBaseName "-" " " | title }}"
date = {{ .Date }}
description = "Hi."
draft = true
toc = false
categories = ["hacking"]
tags = ["after", "dark"]
images = [
  "http://pre14.deviantart.net/dc3a/th/pre/i/2014/190/6/b/cyberpunk_2077_guy_by_biogear-d7pxlf7.jpg"
] # overrides site-wide open graph image
[[copyright]]
  owner = "{{ .Site.Params.author | default .Site.Title }}"
  date = "{{ now.Format "2006" }}"
  license = "cc-by-nc-sa-4.0"
+++

This is a test
