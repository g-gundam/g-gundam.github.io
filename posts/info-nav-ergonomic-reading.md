@def title = "`info-nav`: An Ergonomic Info Reading Experience"
@def tags = ["emacs"]
@def rss = "Present my attempt making info documents easier to read."
@def rss_pubdate = Date(2026, 3, 4)

# info-nav: An Ergonomic Info Reading Experience

2026-03-04

A few months ago, I released [info-nav](https://codeberg.org/ggxx/info-nav)
on MELPA.  It's a small package that remixes existing functionality in a
novel way to make info documents easier to read in Emacs.

## The Idea is Simple

When viewing an info document:

1. display the table of contents on the left;
2. display the actual contents on the right.
3. Make actions in the table of contents window drive navigation on the contents window.

## Screencast

~~~
<div>
<video width="100%" id="video05923777870110751" poster="https://world2ch.net/imgboard/thumb/1771949009300s.jpg" src="https://world2ch.net/imgboard/src/1771949009300.webm" autoplay="" controls="" loop="" class="iichan-video-player"></video>
</div>
~~~

The end result is that you don't need to know any keybindings to navigate an
info document.  Navigation can be 100% mouse driven (even in the terminal).

## Installation

```lisp
(use-package info-nav
  :ensure t
  :bind (("C-h 3" . info-nav)))
```

## Usage

### C-h 3

The recommended binding is `C-h 3`.  The reason I chose this was because it reminds me of how `C-x 3` splits a window in half.  It's also likely to be free and unbound.

### M-x info-nav

If you don't want to bind a key, you can just run it via `M-x info-nav`.

## A Special Message for Unix Philosophers

~~~
<span class="marginnote">
I really want to see Emacs used more as a platform for delivering Elisp applications to people who don't normally use Emacs.  With the addition of <a href="https://emacsredux.com/blog/2024/02/23/changing-the-emacs-configuration-directory/" target="_blank">--init-directory</a> in Emacs 29, this has become easier than ever.
</span>
~~~

- What if you love the command line and don't use Emacs, but you still want to read info documents?
- Check out [nfo](https://codeberg.org/ggxx/nfo).
  + It is a CLI tool that uses Emacs as a user-friendly replacement for the terminal `info` program.
  + I made for people who don't even like Emacs but want to read some docs.

~~~
<script src="https://giscus.app/client.js"
        data-repo="g-gundam/g-gundam.github.io"
        data-repo-id="R_kgDONjA3jw"
        data-category="Announcements"
        data-category-id="DIC_kwDONjA3j84CzOYG"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="catppuccin_macchiato"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
~~~
