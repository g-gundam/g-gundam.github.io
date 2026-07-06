@def title = "Eshell and YASnippet: Making TAB Work"
@def tags = ["emacs"]
@def rss = "A small note on integrating yasnippet into eshell."
@def rss_pubdate = Date(2026, 7, 6)

# Eshell and YASnippet: Making TAB Work

2026-07-06

After reading the following blog posts, I wanted to try using [yasnippet](https://github.com/joaotavora/yasnippet) in eshell.

- [(irreal) YASnippet in Eshell ](https://irreal.org/blog/?p=9773) 2021-06-20
- [(xenodium) Blurring the lines between shell and editor](https://xenodium.com/yasnippet-in-emacs-eshell/) 2021-06-19

Unfortunately, in 2026, it wasn't a simple matter of just enabling `yas-global-mode` to get YASnippet to work.  I can manually `M-x yas-expand` but TAB only did eshell's default completion.  To make the TAB key perform `yas-expand` while in eshell, I had to add the following Elisp ot my config.

```lisp
(defun enable-yas-completion-at-point ()
  "Add `yas-expand' to `completion-at-point-functions'."
  (interactive)
  (add-to-list 'completion-at-point-functions #'yas-expand))

;; Make eshell try yas-expand first before 
;; falling back to eshell's default completion.
(add-to-list 'eshell-mode-hook #'enable-yas-completion-at-point)
```

Previously, I had bound `yas-expand` to a free keybinding that wasn't TAB, but that didn't feel right.  This new way feels better.  Now TAB in eshell will try to `yas-expand` first and then fall back to eshell's default completion behavior.

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
