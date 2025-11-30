@def title = "Paredit and Evil: Improving Navigation"
@def tags = ["emacs"]
@def rss = "Show a paredit configuration for evil users that makes structural navigation more ergonomic."
@def rss_pubdate = Date(2025, 11, 30)

# Paredit and Evil: Improving Navigation

2025-11-30

## The Most Important Setting

If you're an evil user who is having trouble getting paredit's navigation commands to work, make sure `evil-move-beyond-eol` is `t`.

```lisp
;; Many paredit navigation commands won't work without this!
(setq evil-move-beyond-eol t)
```

Without this setting, many paredit navigation commands just don't work.  Unfortunately, it is a departure from traditional vi/vim behavior that some of you may not like.  To limit this behavior to only when you're using paredit-mode, you could do something like this.

```lisp
(defun enable-evil-move-beyond-eol ()
  "Enable `evil-move-beyond-eol' in a buffer-local way."
  (setq-local evil-move-beyond-eol t))
  
;; My preference is to used named functions in hooks,
;; because it makes removal easier.
(add-hook 'paredit-mode-hook #'enable-evil-move-beyond-eol)
```

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

For the commands that move in two dimensions, the mnemonics used by the existing keybindings struggle to capture both dimensions at once.  However, vi-style editing has a long tradtion of using `([{}])` for forwarnd and backward movement, so forward and backward directionality is already solved.  The only thing left to solve is encoding up and down.  My intuition was that downard movement into a sexp would be more common than upward movement out of a sexp so I opted to give downward movement to `[]` and upward movment to `{}`.  This is the end result.

```lisp
(evil-define-key '(normal visual) paredit-mode-map
  ")" #'paredit-forward                 ; evil-forward-sentence-begin
  "(" #'paredit-backward                ; evil-backward-sentence-begin
  "]" #'paredit-forward-down            ; evil-forward-section-begin
  "[" #'paredit-backward-down           ; evil-backward-section-begin
  "}" #'paredit-forward-up              ; evil-forward-paragraph
  "{" #'paredit-backward-up             ; evil-backward-paragraph
  )
```

| command              | key | key | command               |
|----------------------|-----|-----|-----------------------|
| paredit-forward      | (   | )   | paredit-backward      |
| paredit-forward-up   | [   | ]   | paredit-backward-down |
| paredit-forward-down | {   | }   | paredit-backward-up   |

The trade-off is that some traditional evil movement commands had to be unbound, but I think it's a sacrifice worth making in a Lisp editing context.  You're working with sexps not sentences and paragraphs.
