@def title = "Paredit and Evil: Improving Navigation"
@def tags = ["emacs"]
@def rss = "Show a paredit configuration for evil users that makes structural navigation more ergonomic."
@def rss_pubdate = Date(2025, 11, 30)

# Paredit and Evil: Improving Navigation

2025-11-30

## If...

- ...you are an [Emacs](https://www.gnu.org/software/emacs/) user
- who also uses [evil](https://github.com/noctuid/evil-guide) 
- and is having a hard time getting [paredit](https://paredit.org/)  navigation to work,
- _**this post is for you**_.

## The Most Important Setting

First make sure `evil-move-beyond-eol` is true.

```lisp
;; Many paredit navigation commands won't work without this!
(setq evil-move-beyond-eol t)
```

Unfortunately, it is a departure from traditional vi/vim behavior that some of you may not like.  To limit this behavior to only when you're using paredit-mode, you could do something like this.

```lisp
(defun enable-evil-move-beyond-eol ()
  "Enable `evil-move-beyond-eol' in a buffer-local way."
  (setq-local evil-move-beyond-eol t))
  
;; My preference is to used named functions in hooks,
;; because it makes removal easier.
(add-hook 'paredit-mode-hook #'enable-evil-move-beyond-eol)
```

I personally don't mind and do it the first way, but you have options.

## Improving Navigation Keybindings

Paredit provides the following navigation commands.

| command              | key   | key   | command               |
|----------------------|-------|-------|-----------------------|
| paredit-forward      | C-M-f | C-M-b | paredit-backward      |
| paredit-forward-up   | C-M-n | C-M-p | paredit-backward-down |
| paredit-forward-down | C-M-d | C-M-u | paredit-backward-up   |

There are two dimensions of movement here:
- forward and backward and 
- up and down.



For the commands that move in two dimensions, the mnemonics used by the existing keybindings struggle to capture both dimensions at once.  However, vi-style editing has a long tradtion of using `([{}])` for forward and backward movement, so forward and backward directionality is already solved.  The only thing left to solve is encoding up and down.

- `[]` will be used for downard movement, because I think it's more common to move down into a sexp.
- `{}` will be used for upward movement.  This requires the SHIFT key.

| command              | key | key | command               |
|----------------------|-----|-----|-----------------------|
| paredit-forward      | (   | )   | paredit-backward      |
| paredit-forward-up   | [   | ]   | paredit-backward-down |
| paredit-forward-down | {   | }   | paredit-backward-up   |

~~~
<span class="marginnote">
<a href="https://github.com/meow-edit/meow">Meow</a> users could try something similar and gain similar benefits.
</span>
~~~

```lisp
;; Integrate paredit navigation commands in a vi style.
(evil-define-key '(normal visual) paredit-mode-map
  ;;  paredit function                    overriden evil function
  ;;  ----------------                    -----------------------
  ")" #'paredit-forward                 ; evil-forward-sentence-begin
  "(" #'paredit-backward                ; evil-backward-sentence-begin
  "]" #'paredit-forward-down            ; evil-forward-section-begin
  "[" #'paredit-backward-down           ; evil-backward-section-begin
  "}" #'paredit-forward-up              ; evil-forward-paragraph
  "{" #'paredit-backward-up             ; evil-backward-paragraph
  )
```

The trade-off is that some traditional evil movement commands had to be unbound, but I think it's a sacrifice worth making in a Lisp editing context.  After all, you're working with (sexps) not sentences and paragraphs.

I would like evil users to try this out, and see how it feels.  I think you'll find it very intuitive and compatible with vi-style movement.  It should make lisp programming a little more enjoyable too.

Happy Lisp Hacking

