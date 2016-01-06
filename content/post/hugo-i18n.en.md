---
tags:
 - golang
 - hugo
date: 2015-12-31T01:03:12-05:00
slug: hugo-internationalization-i18n-and-multilingual-navigation
title: Hugo internationalization and multilingual navigation
draft: true

---

While improving the _Multilingual_ mode of [Hugo](http://gohugo.io)
(see my previous post on the subject), I was interested in a more
complete solution for internationalization (i18n).


TODO: take the i18n configuration, and use that instead.. we have real i18n now if
 my PR is merged!


#### Multilingual navigation


This is how you'd have articles with links to the other available translations:

Inside `data/i18n/langlinks.yml`, write:

```
en: English
fr: Francais
```

so you can use this template to allow navigation between languages:

<!--more-->

```
        {{if .Site.Multilingual}}
          {{if .IsPage}}
            {{ range $txLang := .Site.LinkLanguages }}
              {{if ne $lang $txLang}}
                {{if isset $.Translations $txLang}}
                  <a href="{{ (index $.Translations $txLang).Permalink }}">{{ index $.Site.Data.i18n "langlinks" $txLang }}</a>
                {{else}}
                  <a href="/{{$txLang}}">{{ index $.Site.Data.i18n "langlinks" $txLang }}</a>
                {{end}}
              {{end}}
            {{end}}
          {{end}}

          {{if .IsNode}}
            {{ range $txLang := .Site.LinkLanguages }}
              {{if ne $lang $txLang}}
                <a href="/{{$txLang}}">{{ index $.Site.Data.i18n "langlinks" $txLang }}</a>
              {{end}}
            {{end}}
          {{end}}
        {{end}}
```

I want to bring your attention to this piece:

```
{{ index $.Site.Data.i18n "langlinks" $txLang }}
```

Here we're reading the `data/i18n/langlinks.yaml` file and fetching
the key for `$txLang`, the language for which the content was
translated.


#### Other strings

With a file named `data/i18n/fr.yaml` containing:

```
nav:
  prev: Précédent
  next: Suivant
```

and `data/i18n/en.yaml` containing something similar, I can write a template such as:

```
{{ $lang := $.Site.RenderLanguage }}

{{if .HasPrev}}
  <a href="{{.Prev.URL}}">{{ index $.Site.Data.i18n $lang "nav" "prev" }}</a>
{{end}}

  <div class="flex"></div>

{{if .HasNext}}
  <a href="{{.Next.URL}}">{{ index $.Site.Data.i18n $lang "nav" "next" }}</a>
{{end}}
```

This is all good, but quite cumbersome in templates. Also, because of
how `partials` are isolated, it is sometimes difficult to pass on the
`$.Site.RenderLanguage` everywhere. So a function was needed!

[INSERT details about how we can handle that]


#### Full blown i18n

Now from that point, we're really close to having a full-blown i18n
package, with support for 200 languages, plurals and all
(https://github.com/nicksnyder/go-i18n). This package has all the
tooling necessary to help with your translation workflows. It's a
great way to integrate with your Hugo development.

[INSERT: how we use that to support i18n in Hugo]

TODO: we have that now, let's document that.. even better: in the hugo docs!
