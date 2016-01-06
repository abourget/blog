---
tags:
- golang
- hugo
date: 2015-12-30T02:25:29-05:00
slug: hugo-multilingual-site-support
title: Hugo multilingual site support
draft: true

---

I wanted to build a multilingual site with [Hugo](http://gohugo.io).

There we three downsides:

1. You need to have two separate domains, one for each language
2. You couldn't cross-link articles between languages
3. You needed to have two separate `config.yaml`

So I dug into the `hugo` codebase, which is very readable and well
laid out, and started hacking a _Multilingual_ mode.

<!--more-->

My goal:

1. Only dealing with a single domain

2. Having a "Francais" and "English" switcher, but that would bring
   you to the corresponding article/post in the other language.

3. Understand why you need two config files.

For now, I left no 3. as-is. I was afraid of that at start, but it does make
a lot of sense.


New configuration options
-------------------------

Here are the new config options you can put in your `config`:

```
Multilingual: true
RenderLanguage: en
DefaultContentLang: en
LinkLanguages:
 - en
 - fr
```

`Multilingual` means you enable the Multilingual mode of Hugo.  It will now render everything
under `/en` for English material.

`RenderLanguage` is the key you change between `config.en.yaml` and `config.fr.yaml`. It
determines which language this run is going to render. You will need to run `hugo` twice
(well, once for each language you want to support).

`LinkLanguages` lists all the languages you want to render links to.  This is to help
you out in the templates.  See below.

`DefaultContentLang` simply means that if you have `post/welcome.md`, it will assume
it is in `en`.  Whereas if you have `post/welcome.es.md`, it will be considered Spanish.



The language switcher
---------------------

You can implement a language switcher like this:

```
{{if .Site.Multilingual}}
  {{if .IsPage}}
    {{$en := index .Translations "en"}}
    {{$fr := index .Translations "fr"}}
    {{if eq $lang "fr"}}
      {{with $en}}<a href="{{$en.Permalink}}">English</a>{{end}}
    {{else if eq $lang "en"}}
      {{with $fr}}<a href="{{$fr.Permalink}}">Francais</a>{{end}}
    {{end}}
  {{end}}
  {{if .IsNode}}
    {{if eq $lang "fr"}}
      <a href="/en">English</a>
    {{else if eq $lang "en"}}
      <a href="/fr">Francais</a>
    {{end}}
  {{end}}
{{end}}
```

See the special statement for _Node_ pages (lists, sections, taxonomy
pages, etc..): they do not link to a corresponding translation, since
they might not always exist, but still, we want to link to other
languages, so we point to the root.


Sitemaps
--------

Genering the site for each language means generating a `sitemap.xml`
for each language.

If you do that, you'll still want to have a `sitemap.xml` at the
complete root of your site, so best thing to do is to hard-code one in
your `static/` directory, pointing to all the languages you have.

For example:

```
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
   <sitemap>
      <loc>http://www.example.com/en/sitemap.xml</loc>
   </sitemap>
   <sitemap>
      <loc>http://www.example.com/fr/sitemap.xml</loc>
   </sitemap>
</sitemapindex>
```


Root URL of your site
---------------------

If we generate one site per language, then what happens to the
complete root, http://example.com ? There's nothing there anymore!

You should now add some sort of manual redirection, something that
knows which languages exist. The easiest way is to hard-code something
like this in `static/index.html`:

```
<html><head>
<meta http-equiv="refresh" content="1;url=/en" /><!-- just in case JS doesn't work -->
<script>
lang = window.navigator.language.substr(0, 2);
if (lang == "fr") {
    window.location = "/fr";
} else {
    window.location = "/en";
}
</script></head><body></body></html>
```
