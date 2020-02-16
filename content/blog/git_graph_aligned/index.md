---
title: "Git Graph: Getting Aligned"
date: 2019-06-17T08:29:38-07:00
categories: [software]
slug: git-graph-aligned
resources:
- name: default
  src: defaultlog.png
  title: Output of default git log
markup: mark
draft: true
---

It can be tricky to get a feel for a git repository and where you are within it.

<!--more-->

## Getting started

We'll start with an oft-recommended command: `git log --all --decorate --oneline --graph`.

{{< figure src="default" class="mid" >}}

This is a good start, but we can do better.

## Pretty formats

`git` is happy to let us specify our own format with `--pretty`. Here's

<screenshot>

## Line it up

## One solution

`man git-log` points out `%<|(<N>)`, which will "make the next placeholder take
at least until Nth columns, padding spaces on the right if necessary".

There are similar formatting placeholders `%>|(N)`, `%>>|(N)`, and `%><|(N)`.


log --graph --abbrev=6 --date-order --all --format=tformat:'%C(auto)%>|(14)%h  %>|(28,trunc)%ar  %<|(41,trunc)%aN %d %s'
