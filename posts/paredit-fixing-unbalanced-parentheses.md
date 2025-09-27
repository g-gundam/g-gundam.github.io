@def title = "Paredit: Fixing Unbalanced Parentheses"
@def tags = ["emacs"]
@def rss = "Talk about how to recover when you're using paredit, and your parentheses become unbalanced."

# Paredit: Fixing Unbalanced Parentheses

2025-09-27

A very important Emacs skill that not enough people talk about is how to fix your code when you're using paredit and your parentheses become unbalanced.  I remember when I first started using paredit, this was a problem I'd run into occasionally, and I did not know how to fix it.  Instead of dealing with a malfunctioning paredit, I decided to not use paredit for a long time which was a shame, because structural editing in Lisp is great.  Here are a few techniques I've come up with to recover from this unbalanced state so that you can continue to use paredit happily.

## Temporarily Disable paredit

- Hit `M-x paredit` to temporarily disable paredit.
- Find where the missing parenthesis belongs and insert it.
- Using `M-x check-parens` can be helpful in this situation.
- Hit `M-x paredit` to reenable paredit.

## Evil Users, use `C-v` in insert mode.

If you're an evil user, you don't have to temporarily disable paredit.  Just find where the missing parenthesis needs to go and hit `C-v )` while in insert mode to force insertion of the ")" character.

## Balance Restored

Once all the parentheses are balanced, paredit should resume functioning properly again.
