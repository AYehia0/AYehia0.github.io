---
title: "How to setup tmux for advanced users ?"
date: 2022-08-10T01:10:44+02:00
draft: true
toc: 
    enable: true
    auto: false
images:
tags: [tmux, linux, rice]
categories: [Linux]
featuredImagePreview: "featured-image.png"
featuredImage: "featured-image.png"
---

## Why you should use tmux ?
Tmux is a terminal multiplexer, it lets you switch easily between several programs in one terminal, detach them (they keep running in the background) and re-attach them to a different terminal.

It also allows you to have multiple panes open at the same time, each with their own shell running, with a single SSH connection (yes, it supports SSH). Not only that, you can also have multiple windows open at the same time, a bit like tabs with more panes in them.

{{< image src="/img/tmux.gif" alt="stats" position="center" style="border-radius: 8px;" >}}

## How to install ?

Many platforms provide prebuilt packages of tmux, although these are often out
of date. Details of the commands to discover and install these can be found in
the documentation for the platform package management tools, for example:

Platform|Install Command
---|---
Arch Linux|`pacman -S tmux`
Debian or Ubuntu|`apt install tmux`
Fedora|`dnf install tmux`

## Configuration
First create a config file as `~/.tmux.conf`, this file holds all the configurations.

### Sourcing
The first thing to do is sourcing the config file by adding this line : `bind r source-file ~/.tmux.conf`, this makes sure the source file is sourced by pressing `prefix + r`

### Prefix
What is the prefix, you ask ? Prefix is similar to vim's leader key and it's so important in tmux as almost any command is set behind a prefix.
Tmux uses `C-b`. Let's change it to something nice. 

```bash
# remap prefix from 'C-b' to 'C-w'
unbind C-b
set-option -g prefix C-w
bind-key C-w send-prefix
```

### Mouse
By default, the mouse is disabled in tmux. We can use the mouse to scroll and resize panes. To enable it:
```bash
# pane scrolling
set -g mouse on
```

### Colors
After launching tmux for the first time, you will notice that the colors are messed up, to fix it:

```bash
set-option -ga terminal-overrides ",xterm-256color:Tc"
set -g default-terminal "screen-256color"
```

### Splitting
One of the great features you use tmux for is : window splitting.
To split use `Prefix + h` for horizontal split, and `Prefix + v` for vertical split.

```bash
# window splitting
bind v split-window -h
bind h split-window -v
unbind '"'
unbind %
```

### Switching
You can switch either between panes or windows in the same tmux session, to easily switch panes by pressing `Prefix + w` while holding `Ctrl` :
```bash
# pane switching
bind ^w select-pane -t :.+
```
To switch between windows : 

```bash
# window switching 
bind -r C-h select-window -t :-
```

### Plugins

Installing TPM (Tmux Plugin Manager)

To install TPM, you need to:

1. Git clone TPM to a HOME directory (~/.tmux/plugins/tpm)
2. Add any plugins with `set -g @plugin 'YOUR/PLUGIN'` in your tmux config file.
3. Point the run command to the TPM repository location (by default it points to `~/.tmux/tpm/tpm`).

### Themes
So we installed TPM, now we can add some plugins for nicer themes, I like the dracula theme :D :

```bash
# dracula plugin
# only show these
set -g @dracula-plugins "cpu-usage ram-usage"
set -g @dracula-show-left-icon session

# plugins
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'dracula/tmux'
```

### Extra
Above are the basic customizations, however there are still more configurations we haven't gone through. Feel free to check the [tmux wiki page](https://github.com/tmux/tmux/wiki) for more advanced configurations.

```bash
# speed it up in neovim
set -sg escape-time 0

# don't rename windows automatically
set-option -g allow-rename off

# index start with 1
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# increase history : default = 2000 lines
set-option -g history-limit 5000

# jump to marked location
bind \` switch-client -t'{marked}'
```
