---
id: 110
title: Configure vim with Python PEP8 check
date: 2015-06-03T11:20:05+00:00
author: Zamir
layout: post
guid: https://i-zsun.rhcloud.com/?p=110
permalink: /configure-vim-with-python-pep8-check/
categories:
  - Uncategorized
---
1. install pep8 for Python

> sudo pip install  pep8

2.  Download the PEP8.vim plugin from http://www.vim.org/scripts/script.php?script_id=2914

3. Put the plugin to ~/.vim/ftplugin/python/pep8.vim

4.  add the following line to ~/.vimrc

> source ~/.vim/ftplugin/python/pep8.vim