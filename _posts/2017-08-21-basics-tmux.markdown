---
layout: post
title: "tmux"
date:   2017-08-21 10:00:01 +0900
categories: shell
layout: post
---

## tmux

installation by brew
(for windows - tmux + cygwin)

(default) BIND-KEY : ctrl + b
 * ctrl + b + : -- command line mode

new window : ctrl + b + c
>                             
prev window : ctrl + b + p
>
prev window : ctrl + b + n
>
close window : ctrl + b + &

# change some of key mappings

``` unbind %
bind | split-window -h
bind - split-window -v
```

# pane synchronize option
``` setw synchronize-panes on
```

# pane synchronize key remapping
``` bind-key * set-window-option synchronize-panes
```

# Reference
http://blog.hawkhost.com/2010/06/28/tmux-the-terminal-multiplexer/
